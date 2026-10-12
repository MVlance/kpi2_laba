# Система бронювання авіаквитків - Взаємодія (Sequence Diagram)

```mermaid
sequenceDiagram
    autonumber
    actor Tourist as Турист
    participant UI as WebClient (Frontend)
    participant API as BookingService (Backend)
    participant DB as Database (PostgreSQL)
    participant Bank as PaymentGateway
    participant Email as NotificationService

    Tourist->>UI: selectSeat(seatId: "12A")
    UI->>API: createBooking(flightId, seatId)
    API->>DB: checkSeatStatus(seatId)
    DB-->>API: SeatStatus (AVAILABLE)
    API->>DB: holdSeat(seatId)
    API->>DB: saveBooking()
    DB-->>API: bookingId: 1042
    API-->>UI: bookingCreated(bookingId)

    loop Повторні спроби оплати (максимум 3 спроби до закінчення 15 хв)
        Tourist->>UI: submitPayment(cardDetails)
        UI->>API: processPayment(bookingId)
        API->>Bank: authorizePayment(amount)
        
        alt Успішна оплата (Bank Status: SUCCESS)
            Bank-->>API: paymentResponse(SUCCESS)
            API->>DB: savePayment()
            API->>DB: saveTicket()
            
            par Асинхронне сповіщення
                API->>Email: sendTicketEmail(passengerEmail, PNR)
                Email-->>API: deliveryStatus(QUEUED)
            end
            
            API-->>UI: paymentSuccess()

        else Помилка оплати або вичерпання таймауту (Status FAILED)
            Bank-->>API: paymentResponse(FAILED)
            API->>DB: updateBookingStatus(CANCELLED)
            API->>DB: releaseSeat(seatId, status: "AVAILABLE")
            API-->>UI: paymentError("Payment failed. Seat released.")
        end
    end
```