# EKHO — LAUNCH ARCHITECTURE v1

## 1. Launch targets

```text
Universities:       1,000
Target capacity:    100,000 MAU
Primary user:       international applicant
Architecture:       global-first
```

Важно:

```text
100k MAU ≠ 100k concurrent users
```

Под **100k users-ready** считаем способность спокойно обслуживать ~100k активных пользователей в месяц + переживать резкие всплески трафика.

Supabase Pro сейчас включает 100k обычных Auth MAU в квоту, поэтому этот target хорошо ложится на выбранную инфраструктуру. ([Supabase](https://supabase.com/docs/guides/platform/manage-your-usage/monthly-active-users?utm_source=chatgpt.com "Manage Monthly Active Users usage - Docs"))

---

# 2. Финальные 1,000 университетов

```text
USA                       350
Europe                    350
Asia                      150
Canada                     40
Australia / New Zealand    35
Middle East                30
Latin America              25
Africa                     20

TOTAL                   1,000
```

---

# 3. Europe — 350

```text
Germany        55
UK             45
France         45
Italy          40
Netherlands    30
Spain          25

Switzerland    15
Ireland        12
Sweden         12
Austria        10
Belgium        10

Finland         8
Denmark         8
Poland          7
Norway          7
Portugal        6

Czechia         5
Hungary         5
Cyprus          3
Greece          2
```

---

# 4. Asia — 150

```text
Japan           20
China           18
South Korea     18
India           15

Hong Kong       12
Malaysia        12
Singapore       10
Taiwan          10

Thailand         8
Indonesia        8
Vietnam          7

Philippines      5
Central Asia     5
Macau            2
```

---

# 5. Rest

### Canada

```text
40
```

### Australia / New Zealand

```text
Australia       28
New Zealand      7
```

### Middle East

```text
UAE              8
Turkey           7
Saudi Arabia     5
Qatar            4
Israel           3
Other            3
```

### Latin America

```text
Mexico           7
Brazil           6
Argentina        3
Chile            3
Colombia         3
Other            3
```

### Africa

```text
South Africa     6
Egypt            4
Morocco          2
Kenya            2
Nigeria          2
Ghana            1
Other            3
```

---

# 6. Не собирать 1,000 одновременно

Seed rollout:

```text
WAVE 0
25 universities

WAVE 1
100

WAVE 2
250

WAVE 3
500

WAVE 4
1,000
```

Каждая wave должна пройти pipeline и проверки до начала следующей.

---

# 7. Wave 0 — система должна сломаться здесь

Первые 25 специально выбирать из разных admissions-систем:

```text
USA
UK
Germany
France
Italy
Netherlands

Canada
Australia

Singapore
Hong Kong
Japan
```

Не только Harvard/MIT/Stanford.

Нам нужны разные:

```text
application platforms
qualification systems
deadlines
tests
financial aid models
program structures
```

Именно на них проверяем Data Architecture + Data Standard + Pipeline.

---

# 8. Wave 1 — 100 Full Verified

Берём самые востребованные университеты, где:

```text
official information accessible
source rights approved
program structure understood
requirements structured
deadlines verified
fees verified
international rules available
```

Цель — **максимальная глубина**, не количество.

---

# 9. Coverage levels

У каждого institution:

```text
coverage_level
```

Три значения:

```text
CORE
FULL
PARTIAL
```

Но `PARTIAL` публично лучше вообще не показывать.

---

# 10. Core Verified

Минимум:

```text
identity
location
official website

degree levels
program availability

current intake

application method
deadline status

tuition status

English requirement status
test policy status

official source
last_verified_at
```

---

# 11. Full Verified

Дополнительно:

```text
program-level requirements

qualification requirements
academic requirements

SAT / ACT
IELTS / TOEFL / DET

documents
essays
recommendations

interviews
portfolio

application fees

detailed tuition

financial aid
scholarships

international rules

complete evidence links
```

---

# 12. Public-launch target

Я бы поставил:

```text
1,000 Core Verified

+

top 200–250 Full Verified
```

Потом постепенно:

```text
250 Full
→ 500
→ 1,000
```

---

# 13. University seed eligibility

Codex должен применять:

```text
candidate
↓
active institution?
↓
degree granting?
↓
relevant applicants accepted?
↓
official website available?
↓
official admissions source?
↓
legal source policy approved?
↓
minimum fields available?
↓
validation
↓
review
↓
publish
```

---

# 14. Seed priority score

Не использовать один ranking.

Внутренний:

```text
launch_priority_score
```

Факторы:

```text
international applicant demand

global reputation / awareness

international student relevance

quality of official information

source accessibility

program coverage

country priority

English-taught availability

data completeness
```

---

# 15. Rankings не должны определять dataset

Не:

```text
QS Top 1000
→ automatically Ekho 1000
```

Rankings можно использовать как **один из discovery signals**, но canonical inclusion решает наша система.

---

# 16. Seed manifest

Не seed SQL вручную.

```text
/data/seeds/

institutions/
countries/
academic-fields/
qualifications/
tests/
application-platforms/
```

---

# 17. Institution seed record

Минимум:

```text
ekho_id
canonical_name
country

official_domain
official_website

external_ids

launch_priority
coverage_target
```

Admissions facts приходят через Data Pipeline, а не прописываются руками внутрь seed.

---

# 18. Seed должен быть idempotent

Обязательное требование Codex:

```text
seed once
seed twice
seed 100 times
```

результат одинаков.

Использовать:

```text
stable IDs
unique constraints
upsert
```

Никаких дублей Stanford.

---

# 19. Seed versioning

```text
seed_version
dataset_version
schema_version
```

Например:

```text
seed: 1.3
schema: 1.0
dataset: 2027.1
```

---

# 20. Dataset release manifest

Каждый production dataset release:

```text
dataset_version

generated_at

institution_count
program_count
country_count

core_verified_count
full_verified_count

fact_count
source_count

validation_errors
blocking_conflicts
```

---

# 21. Environments

Обязательно:

```text
LOCAL
↓
PREVIEW
↓
STAGING
↓
PRODUCTION
```

Supabase поддерживает локальную разработку и version-controlled schema migrations; branching также позволяет тестировать отдельные database environments без изменения production. ([Supabase](https://supabase.com/docs/guides/local-development/overview?utm_source=chatgpt.com "Local development with schema migrations | Supabase Docs"))

---

# 22. Database environments

Минимум:

```text
local database

staging database

production database
```

Codex **никогда** не использует production DB для разработки.

---

# 23. Production access rule

Codex не должен иметь workflow:

```text
AI
→ arbitrary SQL
→ production
```

Все изменения:

```text
migration
↓
local
↓
tests
↓
staging
↓
production
```

---

# 24. Migrations

Все изменения schema:

```text
supabase/migrations/
```

Version control mandatory.

Supabase прямо рекомендует локально тестировать schema changes и сохранять их migration files в version control. ([Supabase](https://supabase.com/docs/guides/local-development/overview?utm_source=chatgpt.com "Local development with schema migrations | Supabase Docs"))

---

# 25. Git workflow

```text
feature branch

↓
PR

↓
automated checks

↓
preview deployment

↓
staging

↓
main

↓
production
```

---

# 26. Automated pre-deploy checks

Codex должен запускать **только релевантные тесты**, но перед production обязательно:

```text
typecheck

lint

unit tests

database migration check

schema validation

critical integration tests

build

smoke tests
```

---

# 27. Production deployment

Для обычного early product:

```text
deploy
↓
smoke test
↓
monitor
```

Когда трафик заметно растёт:

```text
5%
↓
25%
↓
50%
↓
100%
```

Cloudflare Workers поддерживает percentage-based gradual deployments именно для постепенного перевода production traffic на новую версию. ([Cloudflare Docs](https://developers.cloudflare.com/workers/versions-and-deployments/gradual-deployments/?utm_source=chatgpt.com "Gradual deployments - Workers"))

---

# 28. Version affinity

При gradual rollout один пользователь должен по возможности оставаться на одной версии, иначе разные запросы могут попасть на разные releases.

Cloudflare отдельно предоставляет version affinity именно для этой проблемы. ([Cloudflare Docs](https://developers.cloudflare.com/workers/versions-and-deployments/gradual-deployments/version-affinity/?utm_source=chatgpt.com "Version affinity · Cloudflare Workers docs"))

---

# 29. Rollback

Обязательное требование:

```text
bad deploy
↓
previous stable Worker version
```

без нового rebuild.

Cloudflare позволяет сразу откатиться на ранее deployed Worker version. ([Cloudflare Docs](https://developers.cloudflare.com/workers/versions-and-deployments/rollbacks/?utm_source=chatgpt.com "Rollbacks - Workers"))

---

# 30. Database migrations должны быть backward-compatible

Очень важно.

Сначала:

```text
add new field/table
↓
deploy compatible code
↓
migrate data
↓
switch usage
↓
later remove old field
```

Не:

```text
delete column
↓
old production code explodes
```

---

# 31. Beta stages

Это наша launch recommendation:

```text
DOGFOOD
5–10

CLOSED ALPHA
30–50

PRIVATE BETA
150–300

PUBLIC BETA
1,000–3,000

PUBLIC LAUNCH
open
```

---

# 32. Dogfood

Используем весь продукт сами:

```text
search
save university
create application
requirements
documents
tasks
updates
```

Не просто проверяем UI.

---

# 33. Alpha users

Нужны настоящие applicants из разных систем:

```text
IB
A-Level
US High School

German
French
Italian
Indian
Chinese
post-Soviet systems
etc.
```

И разные destinations.

---

# 34. Beta нельзя собирать только из США

Минимум нужны пользователи:

```text
applying only USA

USA + Europe

only Europe

UK

Asia

multiple countries simultaneously
```

Иначе global-first personalization мы нормально не проверим.

---

# 35. Alpha success questions

Не:

> «Нравится дизайн?»

А:

```text
Can they find their university?

Can they find their program?

Are requirements correct?

Do personalized requirements apply correctly?

Do they understand Unknown?

Do they understand next action?

Do they trust the source?

Can they manage an application without explanation?
```

---

# 36. Built-in Report Issue

Рядом с critical information:

```text
Report an issue
```

Reasons:

```text
Incorrect

Outdated

Missing information

Wrong requirement for me

Broken source

Wrong deadline

Wrong cost

University/program missing

Other
```

---

# 37. User report pipeline

```text
Report
↓
review_case
↓
source check
↓
data correction
↓
validation
↓
publish
↓
resolved
```

User никогда напрямую не редактирует canonical data.

---

# 38. Missing university requests

Если search возвращает zero results:

```text
Can't find your university?
[Request it]
```

Сохраняем:

```text
requested_name
country
search_query
request_count
```

Это становится сигналом расширения после первых 1,000.

---

# 39. После 1,000 расширяем только по demand

Ranking expansion:

```text
zero-result searches

university requests

country searches

saved universities

applications created

program searches
```

Не придумываем expansion roadmap заранее на 10,000 вузов.

---

# 40. Monitoring stack

Я бы оставил всего **3 уровня**:

```text
Cloudflare
→ infrastructure/runtime

Sentry
→ application errors/performance

Supabase
→ database/auth
```

Не подключать пять observability SaaS одновременно.

---

# 41. Cloudflare monitoring

Отслеживаем:

```text
requests
errors

CPU time
execution duration

memory/resource errors

response status

deployments
```

Cloudflare Workers observability уже предоставляет request count, error rate, CPU time, wall time и execution-duration metrics. ([Cloudflare Docs](https://developers.cloudflare.com/workers/observability/?utm_source=chatgpt.com "Observability · Cloudflare Workers docs"))

---

# 42. Workers Logs

Включить:

```text
observability.enabled = true
```

Cloudflare Workers Logs собирают invocation logs, custom logs, exceptions и позволяют фильтровать их; sampling можно уменьшить после роста трафика. ([Cloudflare Docs](https://developers.cloudflare.com/workers/observability/logs/workers-logs/?utm_source=chatgpt.com "Workers Logs"))

---

# 43. Structured logging only

Не:

```text
console.log("something went bad lol")
```

А:

```text
event
request_id
route

user_id_hash

duration
status

error_code
release

timestamp
```

---

# 44. Никаких секретов в logs

Запрещено:

```text
password

auth token
session token

full document contents

passport data

essay contents

financial documents

Supabase service key
```

---

# 45. Request ID

Каждый request:

```text
request_id
```

Передаётся:

```text
Cloudflare
↓
application
↓
database/logs
↓
Sentry
```

Так один баг можно проследить через всю систему.

---

# 46. Sentry

Используем для:

```text
frontend errors

server errors

unhandled exceptions

performance traces

failed requests

release regression

source maps
```

Sentry официально поддерживает Next.js error monitoring и performance tracing. ([docs.sentry.io](https://docs.sentry.io/platforms/javascript/guides/nextjs/?utm_source=chatgpt.com "Sentry for Next.js"))

---

# 47. Sentry Replay

**Не включать full replay для всех пользователей.**

Особенно потому что Ekho будет содержать:

```text
grades
documents
application information
financial information
```

Если Replay используется для debugging:

```text
sample heavily
mask inputs
block sensitive components
scrub PII
```

Sentry отдельно предупреждает учитывать PII при Session Replay и поддерживает sensitive-data scrubbing. ([docs.sentry.io](https://docs.sentry.io/platforms/javascript/guides/nextjs/session-replay/?utm_source=chatgpt.com "Set Up Session Replay - Next.js"))

---

# 48. Error severity

```text
SEV-0
security / data loss

SEV-1
Ekho unavailable
login unavailable
applications inaccessible
wrong data published at scale

SEV-2
major feature broken

SEV-3
individual workflow bug

SEV-4
cosmetic
```

---

# 49. Alert only actionable errors

Не будить себя из-за:

```text
one random 404
```

Alert:

```text
error spike

new production regression

login failures

high 5xx rate

database unavailable

critical cron missed

data pipeline stopped

large latency regression
```

Sentry alerting поддерживает rules для new errors, regressions, spikes и threshold-based problems. ([docs.sentry.io](https://docs.sentry.io/product/monitors-and-alerts/alerts/?utm_source=chatgpt.com "Alerts"))

---

# 50. Uptime monitoring

Минимум endpoints:

```text
/
 /universities

/app

/api/health/live
/api/health/ready
```

Sentry имеет uptime monitoring для endpoint availability и может создавать downtime alerts. ([docs.sentry.io](https://docs.sentry.io/product/monitors-and-alerts/monitors/uptime-monitoring/?utm_source=chatgpt.com "Uptime Monitoring"))

---

# 51. Liveness

```text
/api/health/live
```

Проверяет только:

> application process жив.

Не делает 15 DB queries.

---

# 52. Readiness

```text
/api/health/ready
```

Проверяет критические dependencies:

```text
app
database
```

Если personalization не может работать:

```text
ready = false
```

---

# 53. Background job monitoring

Отдельно мониторим:

```text
source crawler
source policy checks
data extraction
change detection
notification generation
```

Sentry Cron Monitoring умеет обнаруживать missed/failed recurring jobs. ([docs.sentry.io](https://docs.sentry.io/product/monitors-and-alerts/monitors/crons/?utm_source=chatgpt.com "Cron Monitoring"))

---

# 54. Pipeline monitoring dashboard

Минимум:

```text
sources scheduled

sources fetched

fetch failures

facts extracted

facts rejected

conflicts

review queue size

stale universities

last successful pipeline run
```

Это важнее какого-нибудь fancy infrastructure dashboard.

---

# 55. Database monitoring

Следим:

```text
CPU

connections

slow queries

database size

connection pool utilisation

query latency

locks
```

Serverless должен подключаться через Supavisor; Supabase прямо объясняет, что connection pooling уменьшает overhead и помогает масштабированию serverless workloads. ([Supabase](https://supabase.com/docs/guides/database/connecting-to-postgres?utm_source=chatgpt.com "Connect to your database | Supabase Docs"))

---

# 56. Backup

Production:

```text
Supabase automated backup
+
periodic logical backup
```

Supabase предоставляет database backups и позволяет отдельно делать logical dump через CLI. ([Supabase](https://supabase.com/docs/guides/platform/backups?utm_source=chatgpt.com "Database Backups | Supabase Docs"))

---

# 57. Backup rule

Backup считается существующим **только после проверки restore**.

Периодически:

```text
backup
↓
temporary environment
↓
restore
↓
integrity check
```

Иначе это просто файл, который мы надеемся восстановить.

---

# 58. Never backup secrets with dataset

Разделяем:

```text
user data backup

public admissions dataset

application configuration

secrets
```

Secrets хранятся отдельно.

---

# 59. Performance targets

Public-facing Ekho должен стремиться как минимум к Google's `good` Core Web Vitals:

```text
LCP ≤ 2.5s
INP ≤ 200ms
CLS ≤ 0.1

at p75
```

Это текущие официальные Core Web Vitals thresholds. ([web.dev](https://web.dev/articles/vitals?utm_source=chatgpt.com "Web Vitals | Articles"))

---

# 60. Ekho internal API targets

Наши engineering targets, не внешний SLA:

```text
cached public request:
p95 < 300ms

normal dynamic API:
p95 < 500ms

heavy personalized operation:
p95 < 1s

5xx:
< 1%
```

Это стартовые budgets; после реальных production metrics корректируем.

---

# 61. Load testing

Использовать:

```text
k6
```

Grafana k6 позволяет задавать pass/fail performance thresholds и использовать их в CI. ([Grafana Labs](https://grafana.com/docs/k6/latest/?utm_source=chatgpt.com "Grafana k6 documentation"))

---

# 62. Обязательные load scenarios

```text
university page

university search

filters

login

save university

create application

applications list

application detail

requirements for me
```

Не тестировать только `/`.

---

# 63. Load stages

Начинаем:

```text
100 concurrent dynamic users
↓
500
↓
1,000
```

И отдельно public spike:

```text
5,000+ requests/users hitting cached pages
```

Это **тестовый safety margin**, а не прогноз трафика.

---

# 64. Spike test

Сценарий:

```text
normal traffic
↓
TikTok / Reddit / launch spike
↓
10× traffic
↓
normal
```

Проверяем:

```text
CDN
rate limits
database connections
search
auth
```

---

# 65. Soak test

Несколько часов стабильной нагрузки.

Ищем:

```text
memory leaks
connection leaks
queue backlog
progressive latency
```

---

# 66. Graceful degradation

Очень важное правило.

Если personalization временно упал:

**университетская база должна продолжать работать.**

Если monitoring упал:

**Ekho должен продолжать работать.**

Если data crawler упал:

**существующие данные должны продолжать работать.**

---

# 67. Feature isolation

```text
Public university intelligence
        ≠
Data pipeline
        ≠
Personal workspace
```

Ошибка crawler не должна убивать frontend.

---

# 68. Rate limiting

Обязательно для:

```text
login

signup

search

report issue

upload

password reset

expensive personalized endpoint
```

Особенно auth endpoints.

---

# 69. Search abuse

Не разрешать:

```text
1 user
→ 10,000 searches/sec
→ Postgres
```

Добавляем:

```text
debounce
cache
rate limit
minimum query length where useful
```

---

# 70. File uploads

Перед production:

```text
file size limits

allowed file types

random storage IDs

private access

signed URLs

malware strategy before broad launch
```

Documents никогда не public bucket.

---

# 71. Feature flags

Нужен простой feature flag layer.

Пример:

```text
live_updates
financial_aid_v2
new_search
personalization_v2
```

Чтобы отключить сломанный модуль без полного rollback.

Но не подключать LaunchDarkly на старте — достаточно простой собственной системы.

---

# 72. Kill switches

Codex должен предусмотреть:

```text
disable_signups

disable_uploads

disable_pipeline

disable_notifications

disable_personalization

maintenance_mode
```

Одной настройкой.

---

# 73. Launch gate — product

Перед public launch:

```text
Core flows complete

no blocker bugs

onboarding works

search works

applications work

requirements work

sources visible

mobile usable
```

---

# 74. Launch gate — data

```text
1,000 Core Verified

200–250 Full Verified

0 blocking conflicts

0 unapproved sources

critical facts sourced

freshness checks running
```

---

# 75. Launch gate — infrastructure

```text
production/staging separated

migrations tested

backup restored successfully

Sentry active

uptime monitoring active

pipeline monitoring active

alerts active

rollback tested

load tests passed
```

---

# 76. Launch gate — security

```text
RLS enabled

private files verified

no service-role keys client-side

secrets configured

rate limits active

sensitive logs scrubbed

dependency vulnerabilities checked
```

---

# 77. Launch gate — performance

```text
Core Web Vitals good

critical routes within budgets

load test passes

no DB connection exhaustion

cache working

images optimized
```

---

# 78. First 24 hours after public launch

Watch only:

```text
availability

error rate

auth failures

database connections

latency

search failures

report issue volume

pipeline failures

data conflicts
```

Не сидеть смотреть vanity analytics.

---

# 79. First-week expansion rule

**Не добавлять features.**

Исправлять:

```text
data correctness
critical UX problems
performance
bugs
missing universities
```

---

# 80. What NOT to add for launch

```text
Kubernetes

microservices

Kafka

multi-region Postgres

read replicas

Redis cluster

Elastic cluster

complex autoscaling

enterprise observability stack

custom APM

data warehouse
```

Необходимость любого из них должна быть доказана production bottleneck.

---

# 81. Масштабирование после 100k MAU

Только когда metrics покажут необходимость:

```text
Postgres bottleneck
→ optimize queries/indexes first

read bottleneck
→ cache/materialized views

search bottleneck
→ dedicated search

DB read pressure
→ read replica

backend complexity
→ selective service separation
```

Не наоборот.

---

# 82. Финальная launch system

```text
                  GITHUB
                     │
            CI / PREVIEW / TESTS
                     │
                     ▼
                CLOUDFLARE
          ┌──────────┴──────────┐
          │                     │
      CDN / CACHE            WORKERS
          │                     │
          │              ┌──────┴──────┐
          │              │             │
          │          SUPABASE        R2
          │          POSTGRES       FILES
          │
          └──────────── USERS


DATA SOURCES
     │
     ▼
LEGAL GATE
     │
     ▼
PIPELINE / QUEUES
     │
     ▼
STAGING
     │
     ▼
VALIDATION
     │
     ▼
POSTGRES


OBSERVABILITY

Cloudflare → infrastructure
Sentry     → application
Supabase   → database/auth
```

## 83. Финальное решение

**Launch Architecture v1 = Strong.**

Codex в будущем должен получить четыре жёстких принципа:

```text
1. Build for 100k MAU, not imaginary hyperscale.

2. 1,000 universities:
   quality and sources before publication.

3. Every production change:
   test → staging → deploy → monitor → rollback.

4. Every critical system must fail independently
   without taking the entire Ekho down.
```

Вот это я бы уже **фиксировал как финальную Launch Architecture v1**. Следом логичнее всего разобрать **Core User Flows**, потому что после данных, структуры продукта и запуска нам надо зафиксировать точное поведение пользователя экран за экраном — это потом напрямую превратится в задачи для Codex.