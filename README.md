# Система бронювання авіаквитків - Машина станів

## 1. Опис станів та переходів

### Основні стани:
* **`PENDING`**: замовлення створене, місце зарезервоване на 15 хвилин (`expires_at`).
* **`ProcessingPayment`**: транзакцію відправлено на обробку в `PaymentGateway`.
* **`OrderFulfillment`**: паралельний стан виконання замовлення після успішного списання коштів.
* **`PAID`**: фінальний успішний стан (квиток згенеровано та сповіщення відправлено).
* **`CANCELLED`**: фінальний стан скасування (таймаут або відхилення платежу).


## 2. Діаграма станів (State Diagram)

```mermaid
stateDiagram-v2
    [*] --> PENDING: selectSeat() / holdSeat(status="RESERVED")

    state PENDING {
        [*] --> SeatReserved
        SeatReserved --> ExtraServicesAttached: addExtraService()
        ExtraServicesAttached --> SeatReserved: removeExtraService()
    }

    PENDING --> CANCELLED: [timeout > 15 min] / releaseSeat(status="AVAILABLE")
    PENDING --> ProcessingPayment: submitPayment()

    state ProcessingPayment {
        [*] --> Authorizing
        Authorizing --> PaymentFailed: [card declined]
    }

    ProcessingPayment --> PENDING: retryPayment() [attempts < 3]
    ProcessingPayment --> CANCELLED: [max retries reached] / releaseSeat(status="AVAILABLE")

    ProcessingPayment --> OrderFulfillment: [payment SUCCESS]

    state OrderFulfillment {
        state "Генерація квитка" as TicketGen {
            [*] --> GeneratingTicket
            GeneratingTicket --> TicketActive: UNIQUE(flight_seat_id) verified
        }
        --
        state "Асинхронне сповіщення" as EmailNotify {
            [*] --> QueuingEmail
            QueuingEmail --> EmailSent
        }
    }

    OrderFulfillment --> PAID: fulfillmentComplete
    PAID --> [*]
    CANCELLED --> [*]
```