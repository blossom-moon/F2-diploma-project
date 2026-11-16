# Thesis Proposal

| Параметр | Значення |
|---|---|
| Студент | Костецька Христина, група 443-2 |
| Спеціальність | F2 Інженерія програмного забезпечення |
| Кафедра | Кафедра комп'ютерних наук, ЧНУ ім. Юрія Федьковича |
| Науковий керівник | _ПІБ керівника_ |
| Напрям (Додаток 1) | Напрям 3 — Application Security & DevSecOps (з елементами Напряму 1) |
| Статус | Чернетка, очікує рецензії керівника |

---

## 1. Робоча назва

- **UA:** Розробка захищеної веб-платформи для перепродажу квитків
- **EN:** Development of a Secure Web Platform for Ticket Resale

## 2. Актуальність та проблематика

Вторинний ринок квитків на концерти, спортивні та культурні події функціонує переважно через соціальні мережі, месенджери та дошки оголошень. Такі канали не забезпечують перевірки автентичності квитка, гарантії оплати та захисту від повторного продажу одного й того самого квитка кільком покупцям.

Інженерна складність проблеми визначається такими чинниками:

- **Конкурентний доступ.** Один квиток може одночасно намагатися придбати кілька покупців; необхідно гарантувати, що угоду завершить рівно один із них (захист від подвійного продажу).
- **Автентичність квитка.** Платформа не має прямого доступу до систем організаторів, тому перевірка має спиратися на унікальні ідентифікатори, контроль дублікатів та криптографічний перевипуск квитка.
- **Автоматизований трафік.** Боти та масові покупці здатні викуповувати значну частину пропозиції за лічені секунди, що потребує обмеження частоти запитів і лімітів на рівні облікового запису.
- **Фінансова цілісність.** Кошти покупця мають утримуватися до підтвердження передачі квитка, а кожна зміна стану угоди — фіксуватися в незмінному журналі аудиту.

## 3. Сформована ідея продукту

Захищена веб-платформа перепродажу квитків, побудована як **модульний моноліт** (Modular Monolith) на Spring Boot з чітко ізольованими доменними модулями (`identity`, `catalog`, `listing`, `order`, `payment`, `audit`, `notification`) та **асинхронною подієвою взаємодією** через брокер повідомлень із патерном **Transactional Outbox**. Межі модулів спроєктовано так, що модулі `payment` і `notification` можуть бути виділені в окремі сервіси без зміни контрактів.

Ключовий фокус роботи — безпека застосунку: централізоване керування ідентичністю (OAuth 2.0 / OpenID Connect), захист від автоматизованого трафіку, криптографічно підписані квитки, незмінний журнал аудиту та DevSecOps-конвеєр зі статичним, композиційним і динамічним аналізом безпеки.

## 4. Ключові функції (Core Features)

1. **Автентифікація та авторизація через Identity Provider.** Інтеграція з Keycloak за протоколом OpenID Connect, JWT-токени, рольова модель доступу (RBAC): покупець, продавець, адміністратор. Обов'язкова двофакторна автентифікація (TOTP) для продавців.
2. **Розміщення квитка з верифікацією.** Реєстрація квитка за унікальним ідентифікатором; зберігання хешу (SHA-256) штрихкоду замість відкритого значення; автоматичне відхилення дублікатів; обмеження ціни перепродажу відносно номінальної вартості (price cap).
3. **Резервування з запобіганням подвійному продажу.** Тимчасове блокування квитка на час оформлення замовлення (розподілений lock у Redis з TTL) та оптимістичне блокування на рівні БД (`@Version`); автоматичне звільнення резерву після завершення TTL.
4. **Захист від ботів і масової скупки.** Обмеження частоти запитів за алгоритмом Token Bucket (Bucket4j + Redis) на рівні IP та облікового запису; ліміт кількості квитків на один обліковий запис для однієї події; тимчасове блокування облікових записів з аномальною активністю.
5. **Безпечна угода з утриманням коштів (escrow).** Скінченний автомат станів замовлення (`CREATED → PAID → TICKET_REISSUED → COMPLETED / CANCELLED`); кошти утримуються до перевипуску квитка; платіжний шлюз реалізовано як заглушку зі стабільним контрактом.
6. **Криптографічний перевипуск квитка.** Після оплати попередній QR-код анулюється, а новому покупцю видається квиток із QR-кодом, підписаним цифровим підписом ECDSA; надається ендпоінт перевірки підпису для контролю на вході.
7. **Незмінний журнал аудиту та сповіщення.** Усі зміни станів угод і події безпеки публікуються через Transactional Outbox у брокер повідомлень, записуються в append-only журнал аудиту та ініціюють email-сповіщення. Звернення щодо проблемного квитка приймаються через форму з відповіддю на email після розгляду.

## 5. Технологічний стек

| Категорія | Технології |
|---|---|
| Мови програмування | Java 21, TypeScript, SQL |
| Архітектурні патерни | Modular Monolith, Layered Architecture, Transactional Outbox, State Machine, Event-Driven Communication, Repository, DTO / Mapper |
| Backend | Spring Boot 3, Spring Security (OAuth2 Resource Server), Spring Data JPA, Spring AMQP, Bucket4j, Flyway |
| Frontend | React, TypeScript, Vite |
| Безпека та IAM | Keycloak (OIDC, RBAC, TOTP), JWT, ECDSA (Java Cryptography Architecture), SHA-256 |
| Бази даних та сховища | PostgreSQL (основна БД), Redis (rate limiting, розподілені блокування) |
| Брокер повідомлень | RabbitMQ |
| Тестування | JUnit 5, Mockito, Testcontainers, k6 (навантажувальне тестування) |
| DevSecOps | GitHub Actions, SonarQube (SAST, якість коду), OWASP Dependency-Check (SCA), Trivy (сканування образів), OWASP ZAP (DAST) |
| Контейнеризація та розгортання | Docker (multi-stage build), Docker Compose, хмарний VPS |
| Observability | Spring Boot Actuator, Prometheus, Grafana |
| Документування | Markdown (Docs-as-Code), OpenAPI / Swagger, C4 Model, ADR |

---

## English Version

### 1. Working Title

Development of a Secure Web Platform for Ticket Resale

### 2. Relevance and Problem Statement

The secondary ticket market for concerts, sports and cultural events operates mainly through social networks, messengers and classified boards. These channels provide no ticket authenticity verification, no payment guarantee and no protection against selling the same ticket to several buyers.

The engineering complexity of the problem is determined by:

- **Concurrent access.** Several buyers may attempt to purchase the same ticket simultaneously; exactly one transaction must succeed (double-sale prevention).
- **Ticket authenticity.** The platform has no direct access to organizers' systems, so verification relies on unique identifiers, duplicate detection and cryptographic ticket reissuance.
- **Automated traffic.** Bots and bulk buyers can acquire a large share of the supply within seconds, which requires request rate limiting and per-account quotas.
- **Financial integrity.** Buyer funds must be held until ticket transfer is confirmed, and every transaction state change must be recorded in an immutable audit log.

### 3. Product Concept

A secure ticket resale web platform implemented as a **Modular Monolith** on Spring Boot with isolated domain modules (`identity`, `catalog`, `listing`, `order`, `payment`, `audit`, `notification`) and **asynchronous event-driven communication** via a message broker using the **Transactional Outbox** pattern. Module boundaries allow `payment` and `notification` to be extracted into separate services without contract changes.

The core focus is application security: centralized identity management (OAuth 2.0 / OpenID Connect), automated traffic protection, cryptographically signed tickets, an immutable audit log and a DevSecOps pipeline with static, composition and dynamic security analysis.

### 4. Core Features

1. **Authentication and authorization via an Identity Provider.** Keycloak integration over OpenID Connect, JWT tokens, role-based access control (RBAC): buyer, seller, administrator. Mandatory two-factor authentication (TOTP) for sellers.
2. **Ticket listing with verification.** Ticket registration by unique identifier; storage of a SHA-256 barcode hash instead of the raw value; automatic duplicate rejection; resale price cap relative to face value.
3. **Reservation with double-sale prevention.** Temporary ticket lock during checkout (Redis distributed lock with TTL) and database-level optimistic locking (`@Version`); automatic release after TTL expiry.
4. **Bot and bulk-buying protection.** Token Bucket rate limiting (Bucket4j + Redis) per IP and per account; per-account ticket quota for each event; temporary suspension of accounts with anomalous activity.
5. **Secure escrow transaction.** Order state machine (`CREATED → PAID → TICKET_REISSUED → COMPLETED / CANCELLED`); funds are held until the ticket is reissued; the payment gateway is implemented as a stub with a stable contract.
6. **Cryptographic ticket reissuance.** After payment the previous QR code is revoked and the buyer receives a ticket with an ECDSA-signed QR code; a signature verification endpoint is provided for entry control.
7. **Immutable audit log and notifications.** All transaction state changes and security events are published via Transactional Outbox to the message broker, written to an append-only audit log and trigger email notifications. Reports on problematic tickets are accepted via a form, with an email reply after review.

### 5. Technology Stack

| Category | Technologies |
|---|---|
| Programming languages | Java 21, TypeScript, SQL |
| Architectural patterns | Modular Monolith, Layered Architecture, Transactional Outbox, State Machine, Event-Driven Communication, Repository, DTO / Mapper |
| Backend | Spring Boot 3, Spring Security (OAuth2 Resource Server), Spring Data JPA, Spring AMQP, Bucket4j, Flyway |
| Frontend | React, TypeScript, Vite |
| Security and IAM | Keycloak (OIDC, RBAC, TOTP), JWT, ECDSA (Java Cryptography Architecture), SHA-256 |
| Databases and storage | PostgreSQL (primary DB), Redis (rate limiting, distributed locks) |
| Message broker | RabbitMQ |
| Testing | JUnit 5, Mockito, Testcontainers, k6 (load testing) |
| DevSecOps | GitHub Actions, SonarQube (SAST, code quality), OWASP Dependency-Check (SCA), Trivy (image scanning), OWASP ZAP (DAST) |
| Containerization and deployment | Docker (multi-stage build), Docker Compose, cloud VPS |
| Observability | Spring Boot Actuator, Prometheus, Grafana |
| Documentation | Markdown (Docs-as-Code), OpenAPI / Swagger, C4 Model, ADR |
