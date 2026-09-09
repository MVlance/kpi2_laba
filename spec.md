# Специфікація моделі даних: Система бронювання авіаквитків

## 1. Намір і межі домену
Метою є проєктування реляційної моделі даних для підсистеми пошуку, бронювання авіаквитків та проведення оплат на основі ER діаграми.

## 2. Сутності та їх атрибути (v3, додано FlightSeat замість абстрактного лічильнику available_seats)
* **User**:
  * `id` (UUID, PK) — унікальний ідентифікатор.
  * `email` (VARCHAR(255), UNIQUE, NOT NULL) — пошта.
  * `password_hash` (VARCHAR(255), NOT NULL) — хеш пароля.
  * `role` (VARCHAR(20), NOT NULL) — роль (TOURIST, AGENT).
  * `full_name` (VARCHAR(100), NOT NULL) — повне ім'я.
  * `created_at` (TIMESTAMP, NOT NULL) — дата реєстрації.

* **Flight**:
  * `id` (UUID, PK) — унікальний ідентифікатор рейсу.
  * `flight_number` (VARCHAR(10), NOT NULL) — номер рейсу.
  * `airline_name` (VARCHAR(100), NOT NULL) — авіакомпанія.
  * `origin_airport` (VARCHAR(3), NOT NULL) — аеропорт вильоту.
  * `destination_airport` (VARCHAR(3), NOT NULL) — аеропорт прибуття.
  * `departure_time` (TIMESTAMP, NOT NULL) — запланований виліт.
  * `arrival_time` (TIMESTAMP, NOT NULL) — запланована посадка.

* **FlightSeat** (Фізичні місця на рейсі):
  * `id` (UUID, PK) — ідентифікатор крісла.
  * `flight_id` (UUID, FK -> Flight.id, NOT NULL) — рейс.
  * `seat_code` (VARCHAR(4), NOT NULL) — номер крісла (напр. "12A").
  * `seat_class` (VARCHAR(20), NOT NULL) — клас обслуговування.
  * `base_price` (DECIMAL(10,2), NOT NULL) — базова ціна місця.
  * `status` (VARCHAR(20), NOT NULL) — статус (AVAILABLE, RESERVED, BOOKED).

* **Booking**:
  * `id` (UUID, PK) — ідентифікатор замовлення.
  * `pnr_code` (VARCHAR(6), UNIQUE, NOT NULL) — PNR код.
  * `user_id` (UUID, FK -> User.id, NOT NULL) — замовник.
  * `status` (VARCHAR(20), NOT NULL) — статус замовлення.
  * `total_price` (DECIMAL(10,2), NOT NULL) — загальна вартість.
  * `created_at` (TIMESTAMP, NOT NULL) — час створення.

* **Passenger**:
  * `id` (UUID, PK) — ідентифікатор пасажира.
  * `first_name` (VARCHAR(50), NOT NULL) — ім'я.
  * `last_name` (VARCHAR(50), NOT NULL) — прізвище.
  * `passport_number` (VARCHAR(30), NOT NULL) — серія/номер паспорта.

* **Ticket**:
  * `id` (UUID, PK) — унікальний квиток.
  * `ticket_number` (VARCHAR(20), UNIQUE, NOT NULL) — номер квитка.
  * `booking_id` (UUID, FK -> Booking.id, NOT NULL) — бронювання.
  * `passenger_id` (UUID, FK -> Passenger.id, NOT NULL) — пасажир.
  * `flight_seat_id` (UUID, FK -> FlightSeat.id, UNIQUE, NOT NULL) — заброньоване крісло.

* **ExtraService**:
  * `id` (UUID, PK) — ідентифікатор послуги.
  * `code` (VARCHAR(30), UNIQUE, NOT NULL) — код послуги.
  * `name` (VARCHAR(100), NOT NULL) — назва.
  * `current_price` (DECIMAL(10,2), NOT NULL) — ціна в каталозі.

* **TicketExtraService**:
  * `id` (UUID, PK) — сурогатний PK.
  * `ticket_id` (UUID, FK -> Ticket.id, NOT NULL) — квиток.
  * `service_id` (UUID, FK -> ExtraService.id, NOT NULL) — послуга.
  * `purchased_price` (DECIMAL(10,2), NOT NULL) — фіксована ціна покупки.
  * `quantity` (INT, NOT NULL) — кількість.

* **Payment**:
  * `id` (UUID, PK) — транзакція оплати.
  * `booking_id` (UUID, FK -> Booking.id, NOT NULL) — бронювання.
  * `amount` (DECIMAL(10,2), NOT NULL) — сума платежу.
  * `status` (VARCHAR(20), NOT NULL) — статус транзакції.
  * `created_at` (TIMESTAMP, NOT NULL) — час створення.

## 3. Кардинальності та правила зв'язків
* `User ||--o{ Booking`: клієнт має від 0 до багатьох замовлень.
* `Booking ||--|{ Ticket`: бронювання містить як мінімум 1 квиток.
* `Passenger ||--o{ Ticket`: пасажир прив'язаний до квитків.
* `Flight ||--|{ FlightSeat`: рейс складається з фізичних місць.
* `FlightSeat ||--o| Ticket`: місце або ще не викуплене (0), або належить рівно 1 квитку (1) завдяки UNIQUE FK.
* `Ticket ||--o{ TicketExtraService`: квиток має 0..N додаткових послуг.
* `ExtraService ||--o{ TicketExtraService`: послуга може додаватися в багато квитків.
* `Booking ||--|| Payment`: кожне замовлення має рівно одну оплату.