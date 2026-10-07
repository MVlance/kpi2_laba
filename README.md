### 2. `README.md` (Огляд та Use Case діаграма)

# Система бронювання авіаквитків — Вимоги та Use Cases

Цей репозиторій містить вимоги, специфіковані за синтаксисом EARS, User Stories із Gherkin-сценаріями, матрицю трасування та Use Case діаграму в межах виконання Завдання 2.

---

## 1. Діаграма прецедентів (Use Case Diagram)

Діаграма демонструє:
* **Генералізацію акторів**: `Tourist` та `Agent` є нащадками загального `User`.
* **Вторинних акторів**: `Payment Gateway` та `Notification Service`.
* **Обов'язкові зв'язки `<<include>>`**: `Бронювати квиток` обов'язково включає `Вибрати місце`.
* **Умовні зв'язки `<<extend>>`**: `Замовити додаткові послуги` опціонально розширює базовий процес бронювання; `Автоскасування за таймаутом` розширює замовлення при вичерпанні 15 хв.

```mermaid
flowchart LR
    subgraph Actors ["Користувачі"]
        direction TB
        User((Користувач))
        Tourist((Турист))
        Agent((Турагент))

        Tourist -->|узагальнення| User
        Agent -->|узагальнення| User
    end

    subgraph SystemBoundary ["Підсистема бронювання авіаквитків"]
        direction TB
        UC1(["UC-01: Пошук рейсів"])
        UC2(["UC-02: Бронювати квиток"])
        UC3(["UC-03: Вибрати місце"])
        UC8(["UC-08: Замовити послуги"])
        UC6(["UC-06: Автоскасування холду"])
        UC4(["UC-04: Оплатити замовлення"])
        UC5(["UC-05: Обробити транзакцію"])
        UC7(["UC-07: Надіслати e-ticket"])

        UC2 -.->|include| UC3
        UC8 -.->|extend| UC2
        UC6 -.->|extend| UC2
        UC2 --> UC4
        UC4 -.->|include| UC5
        UC4 -.->|include| UC7
    end

    subgraph ExternalServices ["Зовнішні сервіси"]
        direction TB
        PaymentGateway["external
        Платіжний шлюз"]
        NotificationService["external
        Email-сервіс"]
    end

    User --> UC1
    Tourist --> UC2
    Agent --> UC2

    UC5 --- PaymentGateway
    UC7 --- NotificationService
```
