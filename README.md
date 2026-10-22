### 2. `README.md` (Огляд та Use Case діаграма)

# Система бронювання авіаквитків - Вимоги та Use Cases

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

