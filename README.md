## Цей файл містить вміст readme.md з усіх попередніх робіт. Текст для ЛР №3 починається з рядка *211*

---
# ЛР №1. Система бронювання авіаквитків - домен і модель даних (ER)

Даний репозиторій містить опис даних обраного домену як моделі сутностей і зв'язків. 

## 1. Опис предметної області
Система автоматизує взаємодію між туристами, агентами та авіакомпаніями (пошук рейсів, вибір додаткових послуг, бронювання, можливість зворотного зв'язку та оплата.).

## 2. Логічна ER-модель (v4, нормалізація до 3НФ)
```mermaid
erDiagram
    USER ||--o{ BOOKING : places
    BOOKING ||--|{ TICKET : contains
    PASSENGER ||--o{ TICKET : assigned_to
    FLIGHT ||--|{ FLIGHT_SEAT : has
    FLIGHT_SEAT ||--o| TICKET : allocated_to
    TICKET ||--o{ TICKET_EXTRA_SERVICE : includes
    EXTRA_SERVICE ||--o{ TICKET_EXTRA_SERVICE : referenced_by
    BOOKING ||--o{ PAYMENT : settles

    USER {
        uuid id PK
        varchar email UK
        varchar password_hash
        varchar role
        varchar full_name
        varchar phone
        timestamp created_at
    }

    FLIGHT {
        uuid id PK
        varchar flight_number
        varchar airline_name
        varchar origin_airport
        varchar destination_airport
        timestamp departure_time
        timestamp arrival_time
    }

    FLIGHT_SEAT {
        uuid id PK
        uuid flight_id FK
        varchar seat_code
        varchar seat_class
        decimal base_price
        varchar status
    }

    BOOKING {
        uuid id PK
        varchar pnr_code UK
        uuid user_id FK
        varchar status
        decimal total_price
        timestamp created_at
        timestamp expires_at
    }

    PASSENGER {
        uuid id PK
        varchar first_name
        varchar last_name
        varchar passport_number
        date dob
    }

    TICKET {
        uuid id PK
        varchar ticket_number UK
        uuid booking_id FK
        uuid passenger_id FK
        uuid flight_seat_id FK,UK
        decimal price_at_booking
    }

    EXTRA_SERVICE {
        uuid id PK
        varchar code UK
        varchar name
        decimal current_price
    }

    TICKET_EXTRA_SERVICE {
        uuid id PK
        uuid ticket_id FK
        uuid service_id FK
        decimal purchased_price
        int quantity
    }

    PAYMENT {
        uuid id PK
        uuid booking_id FK
        decimal amount
        varchar currency
        varchar status
        varchar payment_provider_ref
        timestamp created_at
    }
```

---


### 2. `README.md` (Огляд та Use Case діаграма)

# ЛР №2. Система бронювання авіаквитків - Вимоги та Use Cases

Цей репозиторій містить вимоги, специфіковані за синтаксисом EARS, User Stories із Gherkin-сценаріями, матрицю трасування та Use Case діаграму в межах виконання Завдання 2.

---

## 1. Діаграми прецедентів (Use Case Diagrams)

Було розроблено дві окремі діаграми (для Tourist та Agent).
### 1.1. Прецеденти Туриста (Tourist)
Демонструє персональне бронювання квитків, вибір місць, замовлення послуг та оплату банківською карткою.

```mermaid
flowchart LR
    subgraph ActorsB2C ["Користувачі B2C"]
        direction TB
        User((Користувач))
        Tourist((Турист))
        Tourist -->|узагальнення| User
    end

    subgraph TouristSystem ["Підсистема бронювання (Турист)"]
        direction TB
        UC1(["UC-01: Пошук рейсів"])
        UC2A(["UC-02a: Оформити персональне бронювання"])
        UC3(["UC-03: Вибрати місце"])
        UC8(["UC-08: Замовити послуги"])
        UC6(["UC-06: Автоскасування холду"])
        UC4A(["UC-04a: Оплатити карткою онлайн"])
        UC5(["UC-05: Обробити транзакцію"])
        UC7(["UC-07: Надіслати e-ticket"])

        UC2A -.->|include| UC3
        UC8 -.->|extend| UC2A
        UC6 -.->|extend| UC2A
        UC2A --> UC4A
        UC4A -.->|include| UC5
        UC4A -.->|include| UC7
    end

    subgraph ExternalServices1 ["Зовнішні сервіси"]
        direction TB
        PaymentGateway["external
        Платіжний шлюз"]
        NotificationService["external
        Email-сервіс"]
    end

    User --> UC1
    Tourist --> UC2A
    UC5 --- PaymentGateway
    UC7 --- NotificationService
```

### 1.2. Прецеденти Турагента (Agent)

Демонструє агентське бронювання від імені клієнта, доступ до спецтарифів, проведення розрахунків через агентський баланс та управління замовленнями клієнтів.

```mermaid
flowchart LR
    subgraph ActorsB2B ["Користувачі B2B"]
        direction TB
        User((Користувач))
        Agent((Турагент))
        Agent -->|узагальнення| User
    end

    subgraph AgentSystem ["Підсистема бронювання (Турагент)"]
        direction TB
        UC1(["UC-01: Пошук рейсів та спецтарифів"])
        UC2B(["UC-02b: Бронювати квиток клієнту"])
        UC3(["UC-03: Вибрати місце"])
        UC9(["UC-09: Оформити за агентським депозитом / Invoice"])
        UC10(["UC-10: Управління замовленнями клієнтів"])
        UC5B(["UC-05b: Списати кошти з депозиту"])
        UC7(["UC-07: Надіслати e-ticket клієнту"])

        UC2B -.->|include| UC3
        UC2B --> UC9
        UC9 -.->|include| UC5B
        UC9 -.->|include| UC7
        UC10 -.->|extend| UC2B
    end

    subgraph ExternalServices2 ["Зовнішні сервіси"]
        direction TB
        AgentBilling["external
        Агентська білінг-система"]
        NotificationService2["external
        Email-сервіс"]
    end

    User --> UC1
    Agent --> UC2B
    Agent --> UC10
    UC5B --- AgentBilling
    UC7 --- NotificationService2
```

---

## ЛР №3. Діаграми послідовності (Sequence Diagrams) 

### 1. Сценарій 1: Успішна оплата замовлення (US-04 сценарій 1)
```mermaid
sequenceDiagram
    autonumber
    actor Tourist as Турист (Client)
    participant API as BookingController
    participant Service as BookingService
    participant DB as PostgreSQL
    participant PG as PaymentGateway
    participant Email as NotificationService

    Tourist->>+API: POST /bookings/{id}/pay (card_data)
    API->>+Service: processPayment(booking_id, card_data)
    
    opt Перевірка дійсності холду (NOW() < expires_at)
        Service->>+DB: SELECT status, expires_at FROM Booking WHERE id = booking_id
        DB-->>-Service: status='PENDING', expires_at
    end

    Service->>+PG: authorizeAndCharge(amount, card_data)
    PG-->>-Service: 200 OK (provider_ref="TX_12345")

    Service->>+DB: savePayment(booking_id, amount, status='SUCCESS', provider_ref)
    DB-->>-Service: payment_id

    Service->>+DB: updateBookingStatus(booking_id, status='PAID')
    DB-->>-Service: OK

    Service->>+DB: updateSeatStatus(flight_seat_id, status='BOOKED')
    DB-->>-Service: OK

    Service->>+DB: saveTicket(booking_id, passenger_id, flight_seat_id, price_at_booking)
    DB-->>-Service: ticket_number="TKT-778899"

    Service->>+DB: SELECT email FROM User WHERE id = user_id
    DB-->>-Service: email="tourist@example.com"
    
    par Асинхронне сповіщення
        Service-)Email: sendTicketEmailAsync(ticket_number, pnr_code)
        activate Email
        Email-->>Service: ACK (Event Queued)
        deactivate Email
    end

    Service-->>-API: PaymentSuccessResponse(ticket_number, pnr_code)
    API-->>-Tourist: 200 OK (Квиток успішно оформлено)
```

### 2. Сценарій 2: Відмова платіжного gateway та повтор оплати (US-04 сценарій 2)
```mermaid
sequenceDiagram
    autonumber
    actor Tourist as Турист (Client)
    participant API as BookingController
    participant Service as BookingService
    participant DB as PostgreSQL
    participant PG as PaymentGateway

    Tourist->>+API: POST /bookings/{id}/pay (card_data)
    API->>+Service: processPaymentWithRetry(booking_id, card_data)

    loop Повторні спроби оплати [attempt < 3 AND NOW() < expires_at]
        Service->>+PG: authorizeAndCharge(amount, card_data)
        alt Платіж відхилено шлюзом
            PG-->>Service: 400 PaymentDeclined (Insufficient Funds)
            Service->>+DB: savePayment(booking_id, amount, status='FAILED', provider_ref=NULL)
            DB-->>-Service: payment_id
            
            Note over Service,DB: Замовлення залишається в статусі PENDING, місце в RESERVED
            Service-->>API: PaymentFailedException("Картку відхилено. Спробуйте ще раз")
            API-->>Tourist: 400 Bad Request (Замовлення PENDING, повторіть спробу)
        else Успішна повторна оплата
            PG-->>-Service: 200 OK (provider_ref="TX_99999")
            Service->>+DB: savePayment(booking_id, amount, status='SUCCESS', provider_ref)
            DB-->>-Service: payment_id
            Service->>+DB: updateBookingStatus(booking_id, status='PAID')
            DB-->>-Service: OK
            Service-->>API: PaymentSuccessResponse(...)
            API-->>Tourist: 200 OK (Оплачено успішно)
        end
    end
    deactivate Service
    deactivate API
```

### 3. Сценарій 3: Автоматичне скасування замовлення за таймаутом (US-05)
```mermaid
sequenceDiagram
    autonumber
    participant Worker as HoldTimeoutWorker (System)
    participant Service as BookingService
    participant DB as PostgreSQL

    Worker->>+Service: checkExpiredHoldings()
    Service->>+DB: SELECT id, flight_seat_id FROM Booking WHERE status='PENDING' AND expires_at < NOW()
    DB-->>-Service: expired_bookings_list

    loop Для кожного простроченого замовлення
        Service->>+DB: updateBookingStatus(booking_id, status='CANCELLED')
        DB-->>-Service: OK
        Service->>+DB: updateSeatStatus(flight_seat_id, status='AVAILABLE')
        DB-->>-Service: OK
    end

    Service-->>-Worker: Completed (Released)
```