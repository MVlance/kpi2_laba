# Специфікація машини станів (State Machine)

## 1. Об'єкт моделювання
Життєвий цикл замовлення `Booking` та зв'язаного місця `FlightSeat`.

## 2. Перелік станів
* `PENDING`: Початковий стан після створення замовлення.
* `ProcessingPayment`: Очікування відповіді від банку.
* `PAID`: Успішна оплата.
* `CANCELLED`: Скасоване замовлення.

## 3. Події та переходи
* `selectSeat()` -> перехід у PENDING.
* `submitPayment()` -> перехід у ProcessingPayment.
* `paymentSuccess()` -> перехід у PAID.