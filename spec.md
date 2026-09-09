# Специфікація моделі даних: Система бронювання авіаквитків

## 1. Намір і межі домену
Метою є проєктування реляційної моделі даних для підсистеми пошуку, бронювання авіаквитків та проведення оплат на основі ER діаграми.

### Основні процеси:
1. **Пошук та вибір рейсів**: перегляд доступних вильотів, розкладу та актуальних маршрутів за заданими параметрами.
2. **Вибір крісел та додаткових послуг**: бронювання конкретних місць у літаку та замовлення додаткових послуг.
3. **Оформлення бронювання**: фіксація даних пасажирів, генерація унікального pnr-коду та блокування місця на 15 хвилин до сплати.
4. **Проведення платежів**: взаємодія з платіжним сервісом, підтримка повторних спроб оплати, фіксація транзакції та випуск електронних квитків.

## 2. Сутності та їх атрибути (v4, нормалізація до 3НФ)
* **User** (Користувач системи):
  * `id` (UUID, PK) — унікальний ідентифікатор.
  * `email` (VARCHAR(255), UNIQUE, NOT NULL) — логін / пошта.
  * `password_hash` (VARCHAR(255), NOT NULL) — хеш пароля.
  * `role` (VARCHAR(20), NOT NULL) — роль (TOURIST, AGENT, ADMIN).
  * `full_name` (VARCHAR(100), NOT NULL) — ім'я користувача.
  * `phone` (VARCHAR(20), NULL) — номер телефону.
  * `created_at` (TIMESTAMP, NOT NULL) — дата реєстрації.

* **Flight** (Розклад / Маршрут рейсу):
  * `id` (UUID, PK) — унікальний ідентифікатор рейсу.
  * `flight_number` (VARCHAR(10), NOT NULL) — номер рейсу (напр. LH-1492).
  * `airline_name` (VARCHAR(100), NOT NULL) — авіакомпанія.
  * `origin_airport` (VARCHAR(3), NOT NULL) — IATA-код вильоту (напр. KBP, WAW).
  * `destination_airport` (VARCHAR(3), NOT NULL) — IATA-код прибуття.
  * `departure_time` (TIMESTAMP, NOT NULL) — запланований час вильоту.
  * `arrival_time` (TIMESTAMP, NOT NULL) — запланований час посадки.

* **FlightSeat** (Фізичні місця на рейсі з підтримкою блокування):
  * `id` (UUID, PK) — ідентифікатор місця.
  * `flight_id` (UUID, FK -> Flight.id, NOT NULL) — прив'язка до вильоту.
  * `seat_code` (VARCHAR(4), NOT NULL) — номер крісла (наприклад, "12A", "14C").
  * `seat_class` (VARCHAR(20), NOT NULL) — клас (ECONOMY, BUSINESS, FIRST).
  * `base_price` (DECIMAL(10,2), NOT NULL) — базова ціна крісла.
  * `status` (VARCHAR(20), NOT NULL) — статус (AVAILABLE, RESERVED, BOOKED).

* **Booking** (Бронювання):
  * `id` (UUID, PK) — ідентифікатор замовлення.
  * `pnr_code` (VARCHAR(6), UNIQUE, NOT NULL) — унікальний код бронювання.
  * `user_id` (UUID, FK -> User.id, NOT NULL) — замовник.
  * `status` (VARCHAR(20), NOT NULL) — статус (PENDING, PAID, CANCELLED).
  * `total_price` (DECIMAL(10,2), NOT NULL) — загальна вартість.
  * `created_at` (TIMESTAMP, NOT NULL) — час створення.
  * `expires_at` (TIMESTAMP, NOT NULL) — час завершення 15-ти хвилинного холду.

* **Passenger** (Дані пасажира):
  * `id` (UUID, PK) — ідентифікатор пасажира.
  * `first_name` (VARCHAR(50), NOT NULL) — ім'я латиницею.
  * `last_name` (VARCHAR(50), NOT NULL) — прізвище латиницею.
  * `passport_number` (VARCHAR(30), NOT NULL) — серія та номер паспорта.
  * `dob` (DATE, NOT NULL) — дата народження.

* **Ticket** (Асоціативна сутність квитка: Booking - Passenger - Seat):
  * `id` (UUID, PK) — унікальний квиток.
  * `ticket_number` (VARCHAR(20), UNIQUE, NOT NULL) — номер квитка.
  * `booking_id` (UUID, FK -> Booking.id, NOT NULL) — прив'язка до замовлення.
  * `passenger_id` (UUID, FK -> Passenger.id, NOT NULL) — на кого виписаний.
  * `flight_seat_id` (UUID, FK -> FlightSeat.id, UNIQUE, NOT NULL) — виділене місце.
  * `price_at_booking` (DECIMAL(10,2), NOT NULL) — фіксована вартість на момент бронювання.

* **ExtraService** (Довідник додаткових послуг):
  * `id` (UUID, PK) — ідентифікатор послуги.
  * `code` (VARCHAR(30), UNIQUE, NOT NULL) — код (EXTRA_BAGGAGE, PRIORITY_BOARDING).
  * `name` (VARCHAR(100), NOT NULL) — назва.
  * `current_price` (DECIMAL(10,2), NOT NULL) — актуальна вартість у каталозі.

* **TicketExtraService** (Асоціативна сутність замовлених послуг):
  * `id` (UUID, PK) — зовнішній PK.
  * `ticket_id` (UUID, FK -> Ticket.id, NOT NULL) — до якого квитка додано.
  * `service_id` (UUID, FK -> ExtraService.id, NOT NULL) — послуга.
  * `purchased_price` (DECIMAL(10,2), NOT NULL) — фіксація вартості на момент додавання.
  * `quantity` (INT, NOT NULL) — кількість (за замовчуванням 1).

* **Payment** (Фінансова транзакція):
  * `id` (UUID, PK) — транзакція оплати.
  * `booking_id` (UUID, FK -> Booking.id, NOT NULL) — за яке бронювання.
  * `amount` (DECIMAL(10,2), NOT NULL) — сума транзакції.
  * `currency` (VARCHAR(3), NOT NULL) — валюта (UAH, EUR, USD).
  * `status` (VARCHAR(20), NOT NULL) — статус (INITIATED, SUCCESS, FAILED, REFUNDED).
  * `payment_provider_ref` (VARCHAR(100), NULL) — ID транзакції.
  * `created_at` (TIMESTAMP, NOT NULL) — час проведення.

## 3. Кардинальності та правила зв'язків
* `User ||--o{ Booking`: один користувач може не мати або мати безліч бронювань; кожне бронювання належить строго одному клієнту.
* `Booking ||--|{ Ticket`: у бронюванні має бути як мінімум 1 квиток (немає порожніх бронювань), один квиток належить рівно 1 бронюванню.
* `Passenger ||--o{ Ticket`: пасажир може фігурувати у багатьох квитках (за історію поїздок), один квиток виписується строго на 1 пасажира.
* `Flight ||--|{ FlightSeat`: на рейсі обов'язково є конфігурація місць, одне фізичне крісло належить строго одному рейсу.
* `FlightSeat ||--o| Ticket`: конкретне місце на рейс може бути ще не продано (0), або продано строго по одному квитку (1) (вимога UNIQUE на `flight_seat_id` усуває овербукінг).
* `Ticket ||--o{ TicketExtraService`: квиток може не мати або мати кілька додаткових послуг.
* `ExtraService ||--o{ TicketExtraService`: послуга з каталогу може додаватись у багато квитків.
* `Booking ||--o{ Payment`: на одне замовлення може бути 0 спроб оплати, або декілька (у разі повторної спроби після збою чи повернення коштів).

## 4. Критерії прийнятності (Acceptance Criteria)
1. Відсутні нерозв'язані відношення many-to-many.
2. Всі сутності перебувають у 3NF.
3. Фінансові значення фіксуються в момент транзакції (атрибути `price_at_booking`, `purchased_price`), що ізолює історію замовлень від змін на серверній стороні.
4. Модель запобігає double booking на рівні констрейнтів (1:1 між активним квитком та місцем).