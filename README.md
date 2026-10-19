# Система бронювання авіаквитків - Машина станів

```mermaid
stateDiagram-v2
    [*] --> PENDING: selectSeat()
    PENDING --> ProcessingPayment: submitPayment()
    ProcessingPayment --> PAID: paymentSuccess()
    ProcessingPayment --> CANCELLED: paymentFailed()
    PAID --> [*]
    CANCELLED --> [*]
```