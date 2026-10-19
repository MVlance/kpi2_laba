# Система бронювання авіаквитків - Машина станів

```mermaid
stateDiagram-v2
    [*] --> PENDING: selectSeat() / holdSeat()
    
    PENDING --> CANCELLED: [timeout > 15 min] / releaseSeat()
    PENDING --> ProcessingPayment: submitPayment()
    
    ProcessingPayment --> PAID: paymentSuccess()
    ProcessingPayment --> CANCELLED: paymentFailed() / releaseSeat()
    
    PAID --> [*]
    CANCELLED --> [*]
```