# Thesis Proposal

**Спеціальність:** F2 Інженерія програмного забезпечення
**Напрям:** AI & Data Engineering (Напрям 2) з елементами подієво-керованої архітектури (Напрям 1) та DevOps & Quality Engineering (Напрям 4)
**Статус:** на погодженні з науковим керівником

---

## Частина 1. Українська версія

### 1. Попередня робоча назва

- **UA:** Інтелектуальна система складського обліку для малого бізнесу з автоматизованим розпізнаванням накладних постачальників
- **EN:** Intelligent Inventory Management System for Small Businesses with Automated Supplier Invoice Recognition

### 2. Актуальність та проблематика

Малі підприємства роздрібної торгівлі, заклади громадського харчування та аптеки отримують товар від багатьох постачальників, які надають первинні документи у паперовій формі або у вигляді PDF без машиночитаної структури. Позиції таких документів вносяться до облікової системи вручну, що призводить до:

- значних трудовитрат на приймання кожної поставки;
- помилок введення та розбіжностей між обліковими й фактичними залишками;
- несвоєчасного дозамовлення і, як наслідок, дефіциту товару або втраченого виторгу.

Поширені облікові системи (BAS, Poster, KeyCRM) автоматизують надходження товарів лише через EDI-обмін або інтеграції з підключеними постачальниками. Документи довільних постачальників вносяться вручну.

Інженерна складність задачі визначається такими чинниками:

1. **Неструктурованість вхідних даних.** Накладні мають різні макети; якість фотографій варіюється (перспективні спотворення, тіні, розмиття); можливі рукописні виправлення.
2. **Семантична неоднозначність найменувань.** Один товар у документах різних постачальників має різні написання, скорочення та одиниці виміру (наприклад, «Молоко Галичина 2,5% 0,9л» та «мол. галич. 2.5»).
3. **Вимоги до достовірності обліку.** Результат імовірнісної моделі не може змінювати залишки без контролю: необхідні валідація, оцінка достовірності та підтвердження користувачем.
4. **Ресурсоємність обробки.** Розпізнавання тексту та виклики LLM тривають від кількох до десятків секунд, тому потребують асинхронного виконання з гарантіями доставки, повторними спробами та ідемпотентністю.
5. **Вимірюваність якості.** Ефективність системи необхідно підтвердити кількісними метриками на контрольному наборі документів.

### 3. Сформована ідея продукту

Веб-платформа складського обліку з подієво-керованим конвеєром інтелектуальної обробки документів (Intelligent Document Processing, IDP).

Архітектура — модульний моноліт (Modular Monolith) з ізоляцією доменних модулів за принципами Clean Architecture та винесеними асинхронними обробниками (workers) для ресурсоємних етапів конвеєра: попередньої обробки зображення, розпізнавання тексту, структурованого витягання даних і зіставлення позицій з каталогом.

Взаємодія між модулем обліку та конвеєром обробки документів реалізується через брокер повідомлень із патерном Transactional Outbox, що забезпечує узгодженість стану документа та гарантовану доставку подій без розподілених транзакцій.

Ключовий принцип — Human-in-the-loop: жодна розпізнана позиція не змінює залишки до перевірки та підтвердження користувачем. Підтверджені зіставлення використовуються як зворотний зв'язок для підвищення точності подальшого зіставлення.

### 4. Ключові функції (Core Features)

1. **Каталог товарів і постачальників.** Ведення номенклатури, одиниць виміру та коефіцієнтів перерахунку фасувань; зберігання варіантів найменувань (аліасів) у розрізі постачальників.
2. **Приймання документів.** Завантаження фото або PDF, збереження оригіналу в S3-сумісному сховищі, попередня обробка зображення (корекція перспективи, вирівнювання, підвищення контрасту), асинхронна постановка на обробку з відстеженням статусу в реальному часі.
3. **Структуроване розпізнавання накладних.** Розпізнавання тексту (OCR) та vision-модель у поєднанні з LLM для перетворення документа у JSON за фіксованою схемою; валідація схеми, перевірка арифметичної узгодженості (кількість × ціна = сума, підсумок документа) та оцінка достовірності кожного поля.
4. **Гібридне зіставлення з каталогом.** Нормалізація найменувань, нечіткий пошук (триграмна схожість), семантичний пошук за векторними ембедінгами (pgvector) та ранжування кандидатів з урахуванням історії підтверджених зіставлень для конкретного постачальника.
5. **Перевірка та проведення документа.** Інтерфейс порівняння розпізнаних позицій з оригінальним зображенням, виділення полів з низькою достовірністю, ручне коригування та атомарне проведення документа в обліку.
6. **Облік руху товарів і звітність.** Незмінний журнал операцій (прихід, списання, продаж, інвентаризаційне коригування) за принципом append-only ledger, розрахунок поточних залишків, аудит змін, звіти за довільний період; рольова модель доступу (власник, працівник).
7. **Сповіщення та прогнозування дозамовлення.** Контроль мінімальних залишків, прогноз дати вичерпання товару на основі середньодобового споживання, доставка сповіщень через Telegram-бот.

**Обсяг MVP:** функції 1–5, облік залишків (функція 6) та сповіщення про мінімальний залишок у Telegram (функція 7).

### 5. Попередній технологічний стек

| Категорія | Технології |
|---|---|
| Мови програмування | Python 3.12, TypeScript, SQL |
| Архітектурні патерни | Modular Monolith, Clean Architecture, Event-Driven Architecture, Transactional Outbox, Idempotent Consumer, Retry з експоненційною затримкою та Dead Letter Queue, Circuit Breaker, Strategy (взаємозамінні OCR/LLM-провайдери), Append-only Ledger |
| Backend-фреймворки | FastAPI, Pydantic v2, SQLAlchemy 2.0, Alembic, Celery |
| Frontend | React, TypeScript, Vite, TanStack Query |
| Обробка документів та ШІ | OpenCV, PaddleOCR / Tesseract, мультимодальна LLM зі структурованим виводом (JSON Schema), багатомовна модель ембедінгів (sentence-transformers), RapidFuzz |
| Бази даних і сховища | PostgreSQL 16 (pgvector, pg_trgm), Redis (кеш, результати задач), MinIO (S3-сумісне сховище) |
| Брокер повідомлень | RabbitMQ (підтвердження доставки, Dead Letter Exchange) |
| Інтеграції | Telegram Bot API (aiogram) |
| Тестування та якість коду | pytest, Testcontainers, Ruff, mypy, SonarQube (Quality Gate), k6 |
| DevSecOps | Docker, Docker Compose, GitHub Actions (CI/CD), Trivy, Gitleaks, Dependabot |
| Observability | OpenTelemetry, Prometheus, Grafana, Jaeger |
| Документація | OpenAPI (Swagger), C4-модель, ADR, MkDocs |

### 6. Критерії оцінювання результату

- точність витягання полів накладної (precision, recall, F1 на рівні полів) на контрольному наборі знеособлених накладних;
- точність зіставлення з каталогом (Top-1 та Top-3 accuracy) та її динаміка зі збільшенням кількості підтверджень;
- середній час від завантаження накладної до її проведення порівняно з ручним введенням;
- затримка конвеєра обробки (p95) та пропускна здатність під навантаженням (k6).

---

## Part 2. English Version

### 1. Working Title

- **UA:** Інтелектуальна система складського обліку для малого бізнесу з автоматизованим розпізнаванням накладних постачальників
- **EN:** Intelligent Inventory Management System for Small Businesses with Automated Supplier Invoice Recognition

### 2. Relevance and Problem Statement

Small retail businesses, food service establishments, and pharmacies receive goods from multiple suppliers who issue source documents on paper or as PDFs without a machine-readable structure. Line items from these documents are entered into the accounting system manually, which results in:

- significant labor costs for processing each delivery;
- data entry errors and discrepancies between recorded and actual stock levels;
- untimely reordering, leading to stockouts or lost revenue.

Widely used accounting systems (BAS, Poster, KeyCRM) automate goods receipt only through EDI or integrations with connected suppliers. Documents from arbitrary suppliers are entered manually.

The engineering complexity of the problem is determined by the following factors:

1. **Unstructured input data.** Invoices have varying layouts; photo quality varies (perspective distortion, shadows, blur); handwritten corrections may occur.
2. **Semantic ambiguity of product names.** The same product appears in different suppliers' documents with different spellings, abbreviations, and units of measure (e.g., "Молоко Галичина 2,5% 0,9л" and "мол. галич. 2.5").
3. **Accounting reliability requirements.** The output of a probabilistic model must not change stock levels without control: validation, confidence scoring, and user confirmation are required.
4. **Computational cost.** OCR and LLM calls take from several to tens of seconds and therefore require asynchronous execution with delivery guarantees, retries, and idempotency.
5. **Measurable quality.** System effectiveness must be demonstrated with quantitative metrics on a reference document set.

### 3. Product Concept

A web-based inventory management platform with an event-driven Intelligent Document Processing (IDP) pipeline.

The architecture is a Modular Monolith with domain modules isolated according to Clean Architecture principles and dedicated asynchronous workers for the computationally expensive pipeline stages: image preprocessing, text recognition, structured data extraction, and catalog matching.

Communication between the inventory module and the document processing pipeline is implemented through a message broker using the Transactional Outbox pattern, which ensures consistency of document state and guaranteed event delivery without distributed transactions.

The key principle is Human-in-the-loop: no recognized line item changes stock levels until it has been reviewed and confirmed by the user. Confirmed matches serve as feedback that improves the accuracy of subsequent matching.

### 4. Core Features

1. **Product and supplier catalog.** Management of product master data, units of measure, and package conversion factors; storage of supplier-specific name variants (aliases).
2. **Document intake.** Upload of photos or PDFs, storage of originals in S3-compatible object storage, image preprocessing (perspective correction, deskewing, contrast enhancement), asynchronous processing with real-time status tracking.
3. **Structured invoice recognition.** OCR and a vision model combined with an LLM to convert the document into JSON with a fixed schema; schema validation, arithmetic consistency checks (quantity × price = amount, document total), and per-field confidence scoring.
4. **Hybrid catalog matching.** Name normalization, fuzzy search (trigram similarity), semantic search over vector embeddings (pgvector), and candidate ranking that incorporates the history of confirmed matches for each supplier.
5. **Review and posting.** An interface comparing recognized line items with the original image, highlighting of low-confidence fields, manual correction, and atomic posting of the document to the ledger.
6. **Stock movement accounting and reporting.** An immutable operation log (receipt, write-off, sale, inventory adjustment) following the append-only ledger principle, current stock calculation, change auditing, reports for any period; role-based access control (owner, employee).
7. **Notifications and reorder forecasting.** Minimum stock level monitoring, stock-out date forecasting based on average daily consumption, notification delivery via a Telegram bot.

**MVP scope:** features 1–5, stock level accounting (feature 6), and minimum stock notifications via Telegram (feature 7).

### 5. Preliminary Technology Stack

| Category | Technologies |
|---|---|
| Programming languages | Python 3.12, TypeScript, SQL |
| Architectural patterns | Modular Monolith, Clean Architecture, Event-Driven Architecture, Transactional Outbox, Idempotent Consumer, Retry with exponential backoff and Dead Letter Queue, Circuit Breaker, Strategy (interchangeable OCR/LLM providers), Append-only Ledger |
| Backend frameworks | FastAPI, Pydantic v2, SQLAlchemy 2.0, Alembic, Celery |
| Frontend | React, TypeScript, Vite, TanStack Query |
| Document processing and AI | OpenCV, PaddleOCR / Tesseract, multimodal LLM with structured output (JSON Schema), multilingual embedding model (sentence-transformers), RapidFuzz |
| Databases and storage | PostgreSQL 16 (pgvector, pg_trgm), Redis (cache, task results), MinIO (S3-compatible storage) |
| Message broker | RabbitMQ (delivery acknowledgements, Dead Letter Exchange) |
| Integrations | Telegram Bot API (aiogram) |
| Testing and code quality | pytest, Testcontainers, Ruff, mypy, SonarQube (Quality Gate), k6 |
| DevSecOps | Docker, Docker Compose, GitHub Actions (CI/CD), Trivy, Gitleaks, Dependabot |
| Observability | OpenTelemetry, Prometheus, Grafana, Jaeger |
| Documentation | OpenAPI (Swagger), C4 model, ADR, MkDocs |

### 6. Evaluation Criteria

- field-level extraction accuracy (precision, recall, F1) on a reference set of anonymized invoices;
- catalog matching accuracy (Top-1 and Top-3 accuracy) and its change as the number of confirmations grows;
- average time from invoice upload to posting compared with manual entry;
- pipeline latency (p95) and throughput under load (k6).
