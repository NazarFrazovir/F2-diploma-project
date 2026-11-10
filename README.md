# F2-diploma-project

Кваліфікаційний проєкт бакалавра за спеціальністю **F2 Інженерія програмного забезпечення**.

**Тема (робоча назва):** Інтелектуальна система складського обліку для малого бізнесу з автоматизованим розпізнаванням накладних постачальників

**Working title:** Intelligent Inventory Management System for Small Businesses with Automated Supplier Invoice Recognition

**Автор:** Фрацовір Н.В.

## Структура репозиторію

| Каталог | Призначення |
|---|---|
| `/src` | Вихідний код системи |
| `/docs` | Документація за принципом Documentation as Code |
| `/docs/adr` | Architecture Decision Records (шаблон Майкла Нігардза) |

## Документація

- [Thesis Proposal](docs/proposal.md) — технічна картка проєкту: актуальність, тип архітектури, ключові функції, технологічний стек
- [Аналіз предметної області та вимоги (Розділ 1)](docs/1_analysis.md) — As-Is/To-Be, функціональні та нефункціональні вимоги
- [Специфікація вимог SRS (ISO/IEC/IEEE 29148)](docs/srs.md) — System Context, Functional/Non-Functional Requirements з ID, Verification Criteria, Glossary
- [Проєктування архітектури (Розділ 2)](docs/2_design.md) — C4 Model (Context, Container, Component) з наскрізним трасуванням вимог, посилання на ADR
- [ADR-001: Вибір сховища для векторних ембедінгів](docs/adr/0001-choice-of-technology.md) — **Approved**
- `F2_Diploma_Chapter1_Фрацовір.docx` — повний текст Розділу 1 (7 стор.)

## Матриця нефункціональних вимог (NFR)

| Категорія | Метрика |
|---|---|
| Продуктивність | API Latency p95 ≤ 200 мс при 100 RPS |
| Продуктивність | Обробка накладної ≤ 15 с (до 30 позицій) |
| Масштабованість | До 50 одночасних задач у черзі, at-least-once |
| Безпека | JWT/OAuth2-OIDC (access ≤ 15 хв), RBAC, Rate Limiting, мінімізація PII при передачі до LLM API |
| Надійність | Circuit Breaker до LLM API, Dead Letter Queue, бекап БД щодоби (retention ≥ 14 днів) |

Повна специфікація з унікальними ID та критеріями верифікації — у [docs/srs.md](docs/srs.md).

## Статус календарного плану

| Етап | Статус |
|---|---|
| Аналіз предметної області та аналогів | Done |
| Обґрунтування технологічного стеку та архітектурного стилю | Done |
| Проєктування архітектури системи (UML, C4 Model) | **Done** |

## Угода щодо комітів

Повідомлення комітів оформлюються за специфікацією [Conventional Commits](https://www.conventionalcommits.org/): `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `ci`.
