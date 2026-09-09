# Специфікація моделі даних: Система бронювання авіаквитків

## 1. Намір і межі домену
Метою є проєктування реляційної моделі даних для підсистеми пошуку, бронювання авіаквитків та проведення оплат на основі діаграми класів 1-го курсу.

## 2. Сутності та їх атрибути
* **User**:
  * `id` (UUID, PK) — ідентифікатор користувача.
  * `email` (VARCHAR(255), UNIQUE, NOT NULL) — email.
  * `password_hash` (VARCHAR(255), NOT NULL) — хеш пароля.
  * `role` (VARCHAR(20), NOT NULL) — роль (TOURIST, AGENT).
  * `full_name` (VARCHAR(100), NOT NULL) — повне ім'я.
  * `created_at` (TIMESTAMP, NOT NULL) — дата реєстрації.

* **Flight**:
  * `id` (UUID, PK) — ідентифікатор рейсу.
  * `flight_number` (VARCHAR(10), NOT NULL) — номер рейсу.
  * `airline_name` (VARCHAR(100), NOT NULL) — авіакомпанія.
  * `origin_airport` (VARCHAR(3), NOT NULL) — код аеропорту вильоту.
  * `destination_airport` (VARCHAR(3), NOT NULL) — код аеропорту прибуття.
  * `departure_time` (TIMESTAMP, NOT NULL) — час вильоту.
  * `arrival_time` (TIMESTAMP, NOT NULL) — час посадки.
  * `available_seats` (INT, NOT NULL) — кількість вільних місць на рейсі.

* **Booking**:
  * `id` (UUID, PK) — ідентифікатор бронювання.
  * `pnr_code` (VARCHAR(6), UNIQUE, NOT NULL) — PNR код.
  * `user_id` (UUID, FK -> User.id, NOT NULL) — клієнт.
  * `status` (VARCHAR(20), NOT NULL) — статус бронювання.
  * `total_price` (DECIMAL(10,2), NOT NULL) — сума.
  * `created_at` (TIMESTAMP, NOT NULL) — дата створення.

* **Passenger**:
  * `id` (UUID, PK) — ідентифікатор пасажира.
  * `first_name` (VARCHAR(50), NOT NULL) — ім'я.
  * `last_name` (VARCHAR(50), NOT NULL) — прізвище.
  * `passport_number` (VARCHAR(30), NOT NULL) — паспорт.

* **Ticket**:
  * `id` (UUID, PK) — ідентифікатор квитка.
  * `ticket_number` (VARCHAR(20), UNIQUE, NOT NULL) — номер квитка.
  * `booking_id` (UUID, FK -> Booking.id, NOT NULL) — бронювання.
  * `passenger_id` (UUID, FK -> Passenger.id, NOT NULL) — пасажир.
  * `flight_id` (UUID, FK -> Flight.id, NOT NULL) — рейс.

* **ExtraService**:
  * `id` (UUID, PK) — ідентифікатор послуги.
  * `code` (VARCHAR(30), UNIQUE, NOT NULL) — код послуги.
  * `name` (VARCHAR(100), NOT NULL) — назва.
  * `current_price` (DECIMAL(10,2), NOT NULL) — ціна послуги.

* **Payment**:
  * `id` (UUID, PK) — ідентифікатор транзакції.
  * `booking_id` (UUID, FK -> Booking.id, NOT NULL) — замовлення.
  * `amount` (DECIMAL(10,2), NOT NULL) — сума.
  * `status` (VARCHAR(20), NOT NULL) — статус платежу.
  * `created_at` (TIMESTAMP, NOT NULL) — час.

## 3. Кардинальності та правила зв'язків
* `User ||--o{ Booking`: клієнт має від 0 до багатьох замовлень.
* `Booking ||--|{ Ticket`: замовлення містить від 1 до багатьох квитків.
* `Passenger ||--o{ Ticket`: пасажир прив'язаний до квитків.
* `Flight ||--o{ Ticket`: рейс містить заброньовані квитки.
* `Ticket }o--o{ ExtraService`: квиток може містити додаткові послуги, а послуга додається до квитків.
* `Booking ||--|| Payment`: кожне замовлення має рівно одну оплату.