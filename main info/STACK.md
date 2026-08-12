## EKHO — TECH FOUNDATION
### 1. Язык
**TypeScript — Strong**
Практически весь продукт:
* frontend
* backend
* API
* data scripts, где возможно
* shared types
Один язык = меньше сложности для нас и Codex.
---
### 2. Frontend
**Next.js + React — Strong**
Почему:
* SEO для страниц университетов
* Server Components
* SSR
* SSG
* ISR
* streaming
* code splitting
* image optimization
* можно смешивать статические и динамические части
Next.js специально позволяет обновлять тысячи контентных страниц через ISR **без полного rebuild сайта**. ([nextjs.org][1])
**Стек:**
`Next.js → React → TypeScript`
---
### 3. Styling / Design Engineering
**Tailwind CSS + собственная design system — Strong**
Поверх:
* Radix Primitives — сложные accessibility-компоненты
* Motion — micro-interactions
* собственные Ekho components
* CSS variables/tokens
Не брать огромный Material UI / Ant Design.
Иначе Ekho быстро станет выглядеть как очередной SaaS.
Пример:
```text
Button
Input
Select
Popover
Tooltip
Command
UniversityCard
RequirementRow
Deadline
Status
ApplicationCard
```
и всё это из одной Ekho Design System.
---
# 4. Где будет работать сайт
## Cloudflare Workers — мой основной выбор
```text
User
 ↓
Cloudflare Global Network
 ↓
Next.js / Worker
 ↓
Supabase / R2
```
Cloudflare сейчас официально поддерживает Next.js через OpenNext: App Router, SSR, SSG, ISR, Server Components, Server Actions, streaming и image optimization. ([Cloudflare Docs][2])
Это очень сильный вариант именно для Ekho, потому что серверная логика выполняется глобально, а статика раздаётся через CDN.
---
# 5. Главное правило архитектуры
**Не обращаться к серверу и БД без необходимости.**
Например человек открывает:
`ekho.club/university/stanford`
Не надо делать:
```text
User
↓
Server
↓
Database
↓
20 queries
↓
Generate HTML
↓
User
```
каждый раз.
Делаем:
```text
Postgres
↓
Next.js generates page
↓
Cloudflare cache
↓
User
```
Следующие тысячи пользователей получают уже готовую страницу.
---
# 6. University pages
### ISR / cached rendering
Это идеально для:
* университетов
* programs
* tuition
* admissions information
* scholarships
* requirements
Контент обновился → инвалидируем конкретную страницу.
Не rebuild:
`30,000 universities`
а только:
`Stanford CS`
Next.js прямо называет снижение server load одной из целей ISR. ([Next.js][1])
---
# 7. Dynamic части
Динамически грузятся только вещи типа:
```text
Your application
Your requirements
Your tasks
Saved universities
Your documents
Notifications
Personalization
```
То есть университетская информация может быть cached, а:
> Requirements for George
подгружается отдельно.
---
# 8. Database
## PostgreSQL — Strong
Конкретно:
### **Supabase Postgres**
Почему:
* настоящий PostgreSQL
* auth
* Row Level Security
* backups
* API
* realtime при необходимости
* extensions
* нормальная миграция с него в будущем
Supabase не использует какую-то закрытую БД — каждому проекту дают полноценный Postgres. ([Supabase][3])
---
# 9. Как хранить admissions data
Не делаем один огромный JSON.
Примерно:
```text
universities
programs
admission_rounds
deadlines
requirements
tests
qualifications
tuition
financial_aid
scholarships
sources
source_snapshots
changes
verification_history
users
user_profiles
applications
application_tasks
documents
notes
scores
```
Это даст нормальные фильтры и связи.
---
# 10. Но admissions системы разных стран разные
Поэтому используем гибрид:
### SQL columns
для важных стандартных данных:
```text
deadline
country
degree
tuition
test_required
application_platform
```
### JSONB
для нестандартных вещей:
```text
country_specific_rules
qualification_details
special_conditions
```
Postgres позволяет индексировать JSONB через GIN indexes. ([PostgreSQL][4])
Получаем гибкость **без перехода на MongoDB**.
---
# 11. Search
На старте:
## PostgreSQL Full Text Search + pg_trgm
Например:
> stanford computer science
> business italy english
> universities accepting DET
Postgres имеет полноценный full-text search и GIN indexes для ускорения таких запросов. ([Supabase][5])
### Не добавляем пока:
* Elasticsearch
* Algolia
* Meilisearch
* Typesense
Это дополнительная инфраструктура.
Когда Postgres реально станет недостаточно — тогда добавим отдельный search engine.
---
# 12. Фото университетов
Вот здесь важно.
**НЕ:**
```text
Postgres → image blob
```
И даже не Supabase Storage как основное долгосрочное хранилище.
### Cloudflare R2
```text
Postgres
university:
cover_image_id = abc123
↓
R2
abc123/original.jpg
```
R2 сейчас стоит примерно **$0.015/GB-month** для standard storage и не берёт плату за internet egress. ([Cloudflare Docs][6])
Для media-heavy Ekho это очень выгодно.
---
# 13. Image optimization
Храним **один качественный original**.
Cloudflare автоматически делает:
```text
400px
800px
1200px
WebP
AVIF
JPEG fallback
```
под экран пользователя.
Cloudflare Images умеет responsive resize и автоматическую конвертацию в современные форматы. ([Cloudflare Docs][7])
На бесплатном уровне сейчас есть до **5,000 unique transformations/month**. ([Cloudflare Docs][8])
---
# 14. Правило изображений Ekho
Например university card:
не грузим:
`5000×3000 / 4 MB`
а браузеру прилетает условно:
`600×400 / ~60–150 KB`
Hero загружаем сразу.
Остальные изображения:
`lazy loading`.
Google/Web.dev рекомендует lazy-loading изображений ниже первого экрана, но **не LCP/hero image**. ([web.dev][9])
---
# 15. Documents
PDF, certificates, transcripts и т.п.:
### R2 private bucket
Postgres хранит только:
```text
document_id
user_id
type
filename
storage_key
created_at
```
Сам файл → R2.
Доступ → временный signed URL.
---
# 16. Raw admissions data
Это важная архитектурная штука.
Когда Ekho проверяет Stanford:
### R2
храним исходник:
```text
HTML
PDF
snapshot
raw extracted text
```
### Postgres
храним структурированную истину:
```text
SAT: optional
deadline: Jan 5
IELTS: accepted
```
Так мы всегда можем доказать:
> откуда Ekho взял эту информацию.
---
# 17. Live Admissions Updates
Для мониторинга:
### Cloudflare Cron
периодически запускает проверки официальных страниц. Cloudflare прямо рекомендует Cron Triggers для периодического получения актуальных данных/API. ([Cloudflare Docs][10])
Дальше:
```text
Cron
 ↓
Queue
 ↓
Fetch source
 ↓
Extract
 ↓
Compare
 ↓
Postgres
 ↓
Change event
```
---
# 18. Очереди
Не проверять 20,000 университетов одним request.
```text
Queue:
Stanford
MIT
Harvard
Oxford
Bocconi
...
```
Workers постепенно обрабатывают задачи.
Это намного устойчивее.
---
# 19. Authentication
## Supabase Auth
На старте:
* Google
* Apple позже
* email magic link
* email/password при необходимости
Не пишем свою auth систему.
Supabase Free сейчас включает до **50,000 MAU** для Auth. ([Supabase][11])
---
# 20. ORM
### Drizzle ORM
Я бы выбрал его вместо Prisma.
```text
Next.js
 ↓
Drizzle
 ↓
PostgreSQL
```
Причина практическая:
* тонкий слой
* SQL остаётся понятным
* TypeScript types
* меньше магии
---
# 21. Database connections
Используем connection pooling.
Supabase использует **Supavisor**, который позволяет большому количеству клиентов использовать ограниченное количество настоящих Postgres connections. ([Supabase][12])
Это важно для serverless архитектуры.
---
# 22. API
Не создавать отдельный:
`api.ekho.club`
на старте.
Next.js:
```text
Server Actions
Route Handlers
Server Components
```
Для продукта нашего масштаба отдельный backend сейчас только добавит сложности.
Позже backend можно выделить.
---
# 23. Что компилируется
Pipeline:
```text
GitHub
 ↓
push
 ↓
Cloudflare Build
 ↓
Next.js build
 ↓
Static assets → CDN
Server code → Workers
 ↓
ekho.club
```
Cloudflare сейчас даёт бесплатные и неограниченные запросы к static assets на Workers. ([Cloudflare Docs][13])
---
# 24. Не генерировать все страницы во время build
Это критично.
Не:
```text
build
→ generate 300,000 university/program pages
→ 40 minute deployment
```
А:
```text
popular pages → prebuilt
rare page opened
↓
generate once
↓
cache
↓
reuse
```
Именно для этого нужен ISR. ([Next.js][1])
---
# 25. Client JavaScript
Чем меньше — тем лучше.
По умолчанию:
### Server Components
Client Components только когда реально нужна интерактивность:
```text
dropdown
drag
search
animation
form
application board
```
Не делать весь Ekho огромным SPA.
---
# 26. Performance lessons из реальных продуктов
Rakuten после работы над Core Web Vitals использовали:
* code splitting
* dynamic imports
* lazy loading
* CDN
* удаление unused JS
* responsive images
* caching
и получили существенное улучшение business metrics; их web.dev case study сообщает **+33.13% conversion rate и +53.37% revenue per visitor**. Это не гарантирует такие же цифры Ekho, но хорошо показывает, что performance — продуктовая метрика, а не косметика. ([web.dev][14])
---
# 27. Главный урок Notion
Очень важный пример.
Notion **не начинал с какого-то безумного distributed database architecture**.
Сначала у них был один PostgreSQL instance.
Только когда продукт вырос очень сильно, они начали shard'ить PostgreSQL. ([Notion][15])
### Значит для Ekho:
**НЕ НАДО сейчас:**
* Kubernetes
* microservices
* Kafka
* database sharding
* Redis clusters
* Elasticsearch clusters
* 15 backend services
Это будет инженерный cosplay.
---
# 28. Cache hierarchy Ekho
Я бы сделал:
```text
Browser cache
      ↓
Cloudflare CDN
      ↓
Next.js cache / ISR
      ↓
Application
      ↓
Postgres
```
Чем выше запрос остановился — тем дешевле и быстрее.
---
# 29. Пример Stanford
Пользователь:
> открывает Stanford
### Первый запрос после изменения
```text
Cloudflare
 ↓
Next
 ↓
Postgres
 ↓
render
 ↓
cache
```
### Следующие 100,000
```text
Cloudflare/cache
 ↓
User
```
БД практически не страдает.
---
# 30. Стоимость
На самой ранней версии потенциально:
```text
Cloudflare       $0–5
Supabase         $0
R2               ~$0
GitHub           $0
Domain           already owned
```
Production позже:
```text
Cloudflare Workers ~$5+
Supabase Pro       ~$25
R2                 usage based
```
То есть вполне реально держать основную инфраструктуру около **$30–50/month на раннем production**, пока трафик умеренный. Это оценка, а не фиксированная цена.
Для масштаба: в собственном примере Cloudflare приложение с **15M requests/month**, где 80% — static assets, выходит у них в $5 Workers cost; это не включает БД, storage и остальные сервисы. ([Cloudflare Docs][13])
---
# 31. Финальный стек Ekho
```text
FRONTEND
Next.js
React
TypeScript
DESIGN
Tailwind CSS
Radix
Motion
Ekho Design System
HOSTING
Cloudflare Workers
CACHE/CDN
Cloudflare
Next.js ISR
DATABASE
Supabase PostgreSQL
DATABASE ACCESS
Drizzle ORM
Supavisor pooling
AUTH
Supabase Auth
SEARCH v1
Postgres FTS
pg_trgm
GIN indexes
IMAGES
Cloudflare R2
Cloudflare Images
DOCUMENTS
Private R2
BACKGROUND JOBS
Cloudflare Workers
Cron
Queues / Workflows
RAW SOURCE ARCHIVE
R2
STRUCTURED ADMISSIONS DATA
PostgreSQL
CODE
GitHub
DEPLOY
GitHub → Cloudflare
```
## 32. Оценка
| Решение                   | Оценка                                                             |
| ------------------------- | ------------------------------------------------------------------ |
| Next.js                   | **Strong**                                                         |
| Cloudflare Workers        | **Strong**                                                         |
| Supabase/Postgres         | **Strong**                                                         |
| R2                        | **Strong**                                                         |
| Drizzle                   | **Strong**                                                         |
| Postgres Search initially | **Strong**                                                         |
| Separate backend          | **Weak сейчас**                                                    |
| Microservices             | **Weak сейчас**                                                    |
| MongoDB                   | **Weak**                                                           |
| Kubernetes                | **Очень Weak**                                                     |
| Elasticsearch сразу       | **Weak**                                                           |
| Vercel                    | **Не входит в v1 production architecture** — может быть пересмотрен только новым явным решением |
### Итоговая архитектура
**Ekho = mostly cached information platform + небольшой динамический personalized layer.**
Это очень выгодная архитектура именно для нашего продукта: информации может быть **огромное количество**, фотографий тоже, но большая её часть меняется редко. Поэтому вместо дорогих серверов мы заставляем **CDN и cache выполнять почти всю тяжёлую работу**.
Следующим логичным шагом я бы уже зафиксировал **`Ekho Data Architecture v1` — какие конкретно таблицы будут в БД, что в каждой хранится и как university → program → requirements → deadline → source → change связываются между собой.**
[1]: https://nextjs.org/docs/app/guides/incremental-static-regeneration?utm_source=chatgpt.com "How to implement Incremental Static Regeneration (ISR)"
[2]: https://developers.cloudflare.com/workers/framework-guides/web-apps/nextjs/?utm_source=chatgpt.com "Next.js · Cloudflare Workers docs"
[3]: https://supabase.com/docs?utm_source=chatgpt.com "Supabase Docs"
[4]: https://www.postgresql.org/docs/current/datatype-json.html?utm_source=chatgpt.com "Documentation: 18: 8.14. JSON Types"
[5]: https://supabase.com/docs/guides/database/full-text-search?utm_source=chatgpt.com "Full Text Search | Supabase Docs"
[6]: https://developers.cloudflare.com/r2/pricing/?utm_source=chatgpt.com "R2 pricing"
[7]: https://developers.cloudflare.com/images/optimization/transformations/flows/?utm_source=chatgpt.com "Create transformation flows"
[8]: https://developers.cloudflare.com/images/pricing/ "Pricing · Cloudflare Images docs"
[9]: https://web.dev/articles/lcp-lazy-loading?utm_source=chatgpt.com "The performance effects of too much lazy loading | Articles"
[10]: https://developers.cloudflare.com/workers/configuration/cron-triggers/?utm_source=chatgpt.com "Cron Triggers · Cloudflare Workers docs"
[11]: https://supabase.com/pricing "Pricing & Fees | Supabase"
[12]: https://supabase.com/docs/guides/troubleshooting/supavisor-and-connection-terminology-explained-9pr_ZO?utm_source=chatgpt.com "Supavisor and Connection Terminology Explained"
[13]: https://developers.cloudflare.com/workers/platform/pricing/ "Pricing · Cloudflare Workers docs"
[14]: https://web.dev/case-studies/rakuten "How Rakuten 24's investment in Core Web Vitals increased revenue per visitor by 53.37% and conversion rate by 33.13%  |  web.dev"
[15]: https://www.notion.com/blog/sharding-postgres-at-notion "Herding elephants: Lessons learned from sharding Postgres at Notion"
