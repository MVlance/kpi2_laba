# Система бронювання авіаквитків - Машина станів

```mermaid
stateDiagram-v2
    [*] --> PENDING: selectSeat() / holdSeat()
    
    PENDING --> CANCELLED: [timeout > 15 min] / releaseSeat()
    PENDING --> ProcessingPayment: submitPayment()
    
    ProcessingPayment --> OrderFulfillment: paymentSuccess()
    
    state OrderFulfillment {
        state "Генерація квитка" as TicketGen {
            [*] --> GeneratingTicket
            GeneratingTicket --> TicketActive
        }
        --
        state "Асинхронне сповіщення" as EmailNotify {
            [*] --> QueuingEmail
            QueuingEmail --> EmailSent
        }
    }

    OrderFulfillment --> PAID: fulfillmentComplete
    ProcessingPayment --> CANCELLED: paymentFailed() / releaseSeat()
    
    PAID --> [*]
    CANCELLED --> [*]
```