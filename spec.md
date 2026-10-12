# Специфікація динаміки взаємодії (Sequence Diagram)

## 1. Обраний Use Case
* Сценарій: UC-02: Бронювання квитка та UC-04: Оплата замовлення (повний транзакційний цикл з 15-ти хвилинним холдом місця).

## 2. Учасники взаємодії (Participants & Mappings)
| Компонент / Учасник | Тип             | Відповідність моделлю даних                                 |
|:--------------------|:----------------|:------------------------------------------------------------|
| Tourist             | Actor           | Актор Tourist                                               |
| WebClient           | Client UI       | Інтерфейс користувача                                       |
| BookingService      | Backend API     | Сервіс управління бізнес-логікою та холдуванням             |
| Database            | Database        | Реляційна БД (таблиці FlightSeat, Booking, Ticket, Payment) |
| PaymentGateway      | External System | Зовнішній актор Payment Gateway                             |
| NotificationService | External System | Зовнішній актор Notification Service                        |

## 3. Кроки та методи взаємодії
1. Вибір крісла та запуск холду: Tourist -> WebClient.selectSeat(seatId)
2. Перевірка та блокування у БД: BookingService -> Database.holdSeat(seatId)
3. Оплата замовлення: Tourist -> PaymentGateway.authorizePayment(cardDetails)
4. Асинхронне сповіщення: BookingService -> NotificationService.sendTicketEmail() (у блоці par).