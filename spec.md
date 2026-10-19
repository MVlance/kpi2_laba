# Специфікація машини станів (State Machine)

## 1. Об'єкт моделювання
Життєвий цикл замовлення `Booking` (поля `status`, `expires_at`) та прив'язаного місця `FlightSeat` (поле `status`).

## 2. Перелік станів та підстанів
| Стан                              | Опис                                        | Відповідність БД                |
|:----------------------------------|:--------------------------------------------|:--------------------------------|
| **PENDING**                       | Замовлення утримується на 15 хвилин         | `Booking.status = 'PENDING'`    |
| **PENDING.ExtraServicesAttached** | Додано багаж/харчування                     | Записи у `TicketExtraService`   |
| **ProcessingPayment**             | Транзакція в обробці                        | Запис у `Payment`               |
| **OrderFulfillment**              | Паралельне виконання генерації та відправки | Створення `Ticket` + SMTP Queue |
| **PAID**                          | Квиток виписано                             | `Booking.status = 'PAID'`       |
| **CANCELLED**                     | Замовлення скасовано, місце звільнено       | `Booking.status = 'CANCELLED'`  |

## 3. Правила переходів
1. `PENDING` -> `CANCELLED`: за умовою `timeout > 15 min` викликається `releaseSeat(status="AVAILABLE")`.
2. `ProcessingPayment` -> `OrderFulfillment`: за умови успішної відповіді платіжного шлюзу.
3. `OrderFulfillment`: здійснює паралельну виписку квитка та відправку email.