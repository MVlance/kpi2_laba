# Система бронювання авіаквитків - домен і модель даних (ER)

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