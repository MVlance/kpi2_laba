# Специфікація моделі даних: Система бронювання авіаквитків

## 1. Намір і межі домену
Метою є проєктування реляційної моделі даних для підсистеми пошуку, бронювання авіаквитків та проведення оплат на основі ER діаграми.

## 2. Сутності та їх атрибути (v2, виправлення many-to-many між Ticket та Extra_Service)
* **User**:
  * `id` (UUID, PK) — унікальний ідентифікатор.
  * `email` (VARCHAR(255), UNIQUE, NOT NULL) — логін / пошта.
  * `password_hash` (VARCHAR(255), NOT NULL) — хеш пароля.
  * `role` (VARCHAR(20), NOT NULL) — роль (TOURIST, AGENT).
  * `full_name` (VARCHAR(100), NOT NULL) — повне ім'я.
  * `created_at` (TIMESTAMP, NOT NULL) — дата реєстрації.

* **Flight**:
  * `id` (UUID, PK) — унікальний ідентифікатор рейсу.
  * `flight_number` (VARCHAR(10), NOT NULL) — номер рейсу.
  * `airline_name` (VARCHAR(100), NOT NULL) — авіакомпанія.
  * `origin_airport` (VARCHAR(3), NOT NULL) — IATA-код вильоту.
  * `destination_airport` (VARCHAR(3), NOT NULL) — IATA-код прибуття.
  * `departure_time` (TIMESTAMP, NOT NULL) — час вильоту.
  * `arrival_time` (TIMESTAMP, NOT NULL) — час посадки.
  * `available_seats` (INT, NOT NULL) — лічильник вільних місць.

* **Booking**:
  * `id` (UUID, PK) — ідентифікатор замовлення.
  * `pnr_code` (VARCHAR(6), UNIQUE, NOT NULL) — PNR код.
  * `user_id` (UUID, FK -> User.id, NOT NULL) — замовник.
  * `status` (VARCHAR(20), NOT NULL) — статус (PENDING, PAID, CANCELLED).
  * `total_price` (DECIMAL(10,2), NOT NULL) — загальна вартість.
  * `created_at` (TIMESTAMP, NOT NULL) — час створення.

* **Passenger**:
  * `id` (UUID, PK) — ідентифікатор пасажира.
  * `first_name` (VARCHAR(50), NOT NULL) — ім'я.
  * `last_name` (VARCHAR(50), NOT NULL) — прізвище.
  * `passport_number` (VARCHAR(30), NOT NULL) — серія та номер паспорта.

* **Ticket**:
  * `id` (UUID, PK) — ідентифікатор квитка.
  * `ticket_number` (VARCHAR(20), UNIQUE, NOT NULL) — унікальний номер квитка.
  * `booking_id` (UUID, FK -> Booking.id, NOT NULL) — замовлення.
  * `passenger_id` (UUID, FK -> Passenger.id, NOT NULL) — пасажир.
  * `flight_id` (UUID, FK -> Flight.id, NOT NULL) — рейс.

* **ExtraService** (Довідник додаткових послуг):
  * `id` (UUID, PK) — ідентифікатор послуги.
  * `code` (VARCHAR(30), UNIQUE, NOT NULL) — код (BAGGAGE, MEAL).
  * `name` (VARCHAR(100), NOT NULL) — назва послуги.
  * `current_price` (DECIMAL(10,2), NOT NULL) — ціна в каталозі.

* **TicketExtraService** (Асоціативна сутність замовлених послуг):
  * `id` (UUID, PK) — сурогатний PK.
  * `ticket_id` (UUID, FK -> Ticket.id, NOT NULL) — до якого квитка додано.
  * `service_id` (UUID, FK -> ExtraService.id, NOT NULL) — послуга з довідника.
  * `purchased_price` (DECIMAL(10,2), NOT NULL) — зафіксована вартість послуги.
  * `quantity` (INT, NOT NULL) — кількість (наприклад, кількість місць багажу).

* **Payment**:
  * `id` (UUID, PK) — транзакція оплати.
  * `booking_id` (UUID, FK -> Booking.id, NOT NULL) — замовлення.
  * `amount` (DECIMAL(10,2), NOT NULL) — сума.
  * `status` (VARCHAR(20), NOT NULL) — статус транзакції.
  * `created_at` (TIMESTAMP, NOT NULL) — час проведення.

## 3. Кардинальності та правила зв'язків
* `User ||--o{ Booking`: один користувач може мати від 0 до багатьох замовлень.
* `Booking ||--|{ Ticket`: замовлення містить від 1 до багатьох квитків.
* `Passenger ||--o{ Ticket`: пасажир прив'язаний до квитків.
* `Flight ||--o{ Ticket`: рейс містить квитки.
* `Ticket ||--o{ TicketExtraService`: до квитка може бути додано від 0 до багатьох додаткових послуг.
* `ExtraService ||--o{ TicketExtraService`: конкретна додаткова послуга фігурує у багатьох квитках.
* `Booking ||--|| Payment`: кожне замовлення має рівно одну оплату.