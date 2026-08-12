# Scaling Rules
## 1. Цель
Scaling Rules определяют заранее:
* когда текущая архитектура остаётся достаточной;
* когда сначала оптимизируем;
* когда увеличиваем compute;
* когда нужен connection pooling;
* когда нужен Redis;
* когда Postgres Search заменяется/дополняется отдельным search engine;
* когда нужна очередь;
* когда нужен отдельный worker;
* когда нужен read replica;
* когда нужен отдельный backend/service;
* когда analytics нужно выносить из primary Postgres;
* когда можно обсуждать sharding;
* какие метрики являются trigger;
* что **не является** trigger.
Главное правило:
> Масштабировать по измеренному bottleneck, а не по числу зарегистрированных пользователей.
Например:
```text
100,000 users
```
само по себе **не означает**:
```text
Redis required
```
или:
```text
dedicated search required
```
или:
```text
microservices required
```
Всё зависит от:
* concurrent traffic;
* requests per second;
* query complexity;
* cacheability;
* read/write ratio;
* search workload;
* database CPU;
* I/O;
* connections;
* latency;
* queue load.
---
# 2. Основной принцип Ekho
Порядок действий всегда:
```text
Measure
↓
Find bottleneck
↓
Fix query / code / index / caching mistake
↓
Load test
↓
Vertical scaling
↓
Specialized infrastructure only when justified
```
Не:
```text
users increased
↓
add Redis
↓
add Kafka
↓
add Elasticsearch
↓
split into microservices
```
Supabase также рекомендует сначала определить реальную причину slowdown через query statistics и database metrics; плохой query/index может быть причиной даже при наличии свободного compute.
---
# 3. Архитектурная стратегия
До доказанного bottleneck:
```text
Frontend / application
        │
        ▼
Application backend/BFF
        │
        ▼
Postgres
```
Дополнительные системы добавляются **по одному**:
```text
CDN
Queue
Redis
Search Engine
Read Replica
Worker Service
Analytics Store
```
Только если существует конкретная проблема, которую система решает.
---
# 4. Основной scaling target
Ekho проектируется минимум под:
```text
100,000 MAU
```
Это:
**Monthly Active Users.**
Это НЕ:
```text
100,000 simultaneous users
```
и НЕ:
```text
100,000 concurrent DB connections
```
Public university/program pages должны максимально обслуживаться через CDN/cache и не превращать каждый SEO page view в запрос к Postgres.
Поэтому 100k MAU сами по себе не требуют распределённой backend architecture.
---
# 5. Existing performance budgets
Сохраняются уже зафиксированные Ekho budgets:
### Public cached response
```text
p95 < 300 ms
```
### Normal dynamic API
```text
p95 < 500 ms
```
### Heavy personalized operation
```text
p95 < 1,000 ms
```
### Error rate
```text
5xx < 1%
```
### Frontend
```text
LCP ≤ 2.5 s
INP ≤ 200 ms
CLS ≤ 0.1
```
Scaling decision начинается, когда текущая architecture больше не удерживает эти budgets при ожидаемой production load.
---
# 6. Не реагировать на один spike
Infrastructure change нельзя делать из-за одного случайного spike.
## Ekho operational rule
Обычный scaling trigger считается подтверждённым, если threshold нарушается:
```text
≥3 peak periods
```
за:
```text
7 days
```
или:
```text
continuously ≥15 minutes
```
при normal production load.
Исключения:
* outage;
* data loss risk;
* connection exhaustion;
* security incident;
* sustained 5xx;
* hard provider limit.
В этих случаях действуем сразу.
---
# 7. Перед любым scaling change
Обязательно проверить:
1. `pg_stat_statements`;
2. slow queries;
3. query plans;
4. missing indexes;
5. unnecessary sequential scans;
6. oversized result sets;
7. N+1 queries;
8. duplicate API calls;
9. unnecessary DB requests;
10. connection leaks;
11. cacheability;
12. DB CPU;
13. memory pressure;
14. I/O;
15. read/write ratio.
`pg_stat_statements` существует именно для tracking planning/execution statistics SQL statements.
---
# 8. EXPLAIN before infrastructure
Для slow/hot query:
```sql
EXPLAIN (ANALYZE, BUFFERS)
...
```
Проверять:
* execution time;
* rows;
* estimated vs actual rows;
* index scans;
* sequential scans;
* sort;
* loops;
* buffers;
* expensive joins.
Supabase прямо рекомендует `EXPLAIN ANALYZE` перед scaling compute; missing index иногда устраняет проблему значительно дешевле инфраструктурного upgrade.
---
# 9. Query optimization gate
Нельзя масштабировать DB только потому, что query медленный.
Сначала:
```text
Is query indexed correctly?
```
↓
```text
Is it returning too much data?
```
↓
```text
Is JOIN/filter/order correct?
```
↓
```text
Can computation be precomputed?
```
↓
```text
Can public response be cached?
```
↓
только потом:
```text
More compute?
```
---
# 10. Database Performance Advisor
Если используется Supabase:
регулярно проверять:
```text
Performance Advisor
```
Он способен выявлять, среди прочего, missing indexes.
---
# 11. Database monitoring baseline
С launch мониторить:
```text
DB CPU
DB memory
DB disk I/O
DB connections
query latency
transaction latency
locks
slow queries
read/write ratio
database size
index size
```
Также:
```text
API p50
API p95
API p99
5xx
```
---
# 12. Database CPU thresholds
Supabase в своей current scaling guidance определяет sustained:
```text
CPU > 70%
```
как database running hot.
Поэтому Ekho использует:
### Healthy
```text
< 60%
```
### Warning
```text
60–70%
```
### Scaling investigation
```text
>70% sustained
```
### Critical
```text
>85% sustained
```
`60%` и `85%` — Ekho operational thresholds.
`70%` опирается на официальную Supabase guidance.
---
# 13. Что делать при DB CPU >70%
Порядок:
```text
1. pg_stat_statements
2. Find top queries by total execution time
3. EXPLAIN ANALYZE
4. Fix queries/indexes
5. Load test
6. Check read/write ratio
7. Only then scale compute / replicas
```
---
# 14. Vertical DB scaling
Увеличивать primary DB compute, если:
* CPU sustained >70%;
* queries уже оптимизированы;
* bottleneck CPU/RAM primary;
* workload write-heavy или balanced;
* current tier имеет доступный upgrade.
Supabase рекомендует larger compute для write-heavy workloads и как наиболее простой путь scaling, потому что read replicas не разгружают writes.
---
# 15. Vertical scaling first
До архитектурного усложнения:
```text
small DB
↓
optimized DB
↓
larger DB
```
обычно предпочтительнее:
```text
small DB
↓
distributed architecture
```
если проблема решается compute upgrade.
Это operational decision Ekho, а не универсальный numerical law.
---
# 16. DB connections
Connection exhaustion — отдельный bottleneck.
Нельзя судить о capacity только по CPU.
Мониторить:
```text
active_connections
max_connections
pool utilization
waiting clients
connection errors
```
Supabase предупреждает, что приближение connection count к лимиту приводит к connection errors.
---
# 17. Ekho connection thresholds
### Healthy
```text
<60% DB connection capacity
```
### Warning
```text
60–75%
```
### Action
```text
>75% sustained
```
### Critical
```text
>90%
```
Это Ekho internal headroom policy.
---
# 18. Serverless connections
Если backend использует temporary/serverless functions:
использовать transaction-mode pooler.
Supabase официально рекомендует transaction pooling для:
* serverless functions;
* edge functions;
* temporary clients.
---
# 19. Supavisor
При Supabase serverless architecture:
```text
Cloudflare Workers / OpenNext
↓
Supavisor transaction mode
↓
Postgres
```
а не:
```text
every function
↓
direct persistent DB connection
```
Transaction pooling позволяет нескольким clients делить ограниченное количество backend Postgres connections.
---
# 20. Pool size
Не устанавливать pool size случайно.
Supabase рекомендует сохранять headroom для других platform services и отмечает, что фактический pool size должен зависеть от real peak concurrent connection usage.
---
# 21. Redis — launch rule
**Redis не нужен на launch только потому, что приложение должно масштабироваться.**
Не добавлять Redis для:
* public university pages;
* public program pages;
* static dictionaries;
* simple user session state;
* search только ради скорости;
* вещей, которые уже эффективно решает CDN;
* данных, которые Postgres быстро отдаёт по index.
---
# 22. CDN before Redis
Public anonymous GET:
```text
/universities/*
/programs/*
/subjects/*
/countries/*
```
при возможности:
```text
CDN
↓
cached response
```
а не:
```text
CDN miss
↓
Redis
↓
Postgres
```
если CDN уже способен решить проблему.
---
# 23. Когда Redis действительно нужен
Redis рассматривается только когда существует хотя бы один конкретный use case:
1. high-frequency repeated dynamic reads;
2. distributed rate limiting;
3. short-lived distributed state;
4. idempotency keys;
5. distributed locks;
6. expensive computation cache;
7. very hot derived data;
8. short-lived counters.
---
# 24. Redis не source of truth
Никогда не хранить только в Redis:
* applications;
* tasks;
* requirements;
* university truth;
* documents;
* financial data;
* user profile;
* application status.
Source of truth:
```text
Postgres
```
Redis:
```text
derived / ephemeral state
```
---
# 25. Redis cache eligibility
Перед добавлением cache query должна удовлетворять:
```text
expensive enough to matter
+
frequently repeated
+
same result reused
+
safe invalidation strategy
```
Если каждый request уникален:
Redis мало помогает.
---
# 26. Redis trigger
## Ekho internal rule
Рассматривать Redis, если после query/index optimization:
```text
same expensive reads
```
создают:
```text
≥20% of DB execution time
```
или являются заметной причиной:
```text
DB CPU >70%
```
и при этом данные имеют хорошую cacheability.
`20%` — внутренний Ekho threshold.
---
# 27. Redis validation experiment
Не внедрять Redis сразу глобально.
Сначала одна cache candidate.
После pilot оставить Redis только если:
```text
cache hit rate ≥70%
```
и:
```text
DB load reduction ≥30%
```
или user latency materially improves.
Это **Ekho success gate**, а не Redis universal benchmark.
---
# 28. Redis monitoring
Если Redis появляется:
мониторить:
* cache hit rate;
* misses;
* evictions;
* expiration rate;
* memory usage;
* latency;
* connection count;
* oversized keys.
Redis официально рекомендует отслеживать cache hit rate и eviction/expiry metrics; frequent evictions могут указывать на memory pressure или неправильный sizing.
---
# 29. Redis memory
Обязательно:
```text
maxmemory
```
*
explicit:
```text
eviction policy
```
Redis способен автоматически удалять keys согласно eviction policy при достижении memory limit.
---
# 30. Cache TTL
Каждый Redis key должен иметь:
```text
TTL
```
если нет отдельной documented reason для persistent cache.
TTL должен зависеть от freshness requirement.
---
# 31. Cache invalidation
Для university data предпочтительно:
```text
data changed
↓
invalidate cache
```
а не:
```text
wait 24 hours
```
для critical admissions changes.
---
# 32. Не кэшировать stale critical admissions truth
Особенно осторожно:
* deadlines;
* testing policies;
* tuition;
* application status;
* personalized requirements.
Нельзя позволять cache layer давать старую critical value после подтверждённого update.
---
# 33. Postgres Search — launch
Начальный search implementation может оставаться на PostgreSQL, если он выполняет зафиксированный Search & Filtering contract.
PostgreSQL имеет:
* Full Text Search;
* GIN indexes;
* ranking;
* dictionaries;
* `pg_trgm` similarity;
* trigram indexes.
PostgreSQL рекомендует GIN как preferred index type для full-text search.
`pg_trgm` предоставляет indexed similarity matching.
---
# 34. Search index
Search не должен сканировать основные normalized tables каждый keystroke.
Создать derived searchable representation.
Conceptually:
```text
search_entities
```
с:
* entity_id;
* entity_type;
* canonical_name;
* aliases;
* normalized_name;
* country;
* subject;
* degree;
* searchable_text;
* ranking features.
---
# 35. Search indexes
Использовать соответствующие:
```text
B-tree
GIN
pg_trgm
```
индексы по реальным query patterns.
Не индексировать всё автоматически.
Каждый index:
* занимает disk;
* требует maintenance;
* добавляет write cost.
Supabase также предупреждает, что excessive indexes замедляют writes и расходуют storage.
---
# 36. Search architecture abstraction
Frontend не должен знать:
```text
Postgres
```
или:
```text
Algolia
Typesense
Meilisearch
other engine
```
Frontend вызывает:
```text
Search API
```
Search implementation находится за abstraction.
Так migration search engine не требует переписывания UI.
Это уже обязательная architecture rule.
---
# 37. Не переключать search по количеству universities
Неверное правило:
```text
>10,000 universities → search engine
```
или:
```text
>100,000 rows → search engine
```
Размер dataset сам по себе не определяет необходимость migration.
PostgreSQL GIN специально предназначен в том числе для scalable full-text search workloads.
---
# 38. Search SLO Ekho
Search получает более строгий target, чем generic API.
### Search query
```text
p95 ≤250 ms
```
### Autocomplete
```text
p95 ≤150 ms
```
### Search p99
```text
≤500 ms
```
Это **Ekho product SLO**, не внешний universal benchmark.
---
# 39. Search migration trigger #1 — latency
Начать dedicated search evaluation если:
после:
* correct indexes;
* normalized search document;
* query optimization;
* limiting result set;
* production-like load test;
Postgres Search всё ещё нарушает:
```text
p95 >250 ms
```
в:
```text
3+ peak periods / 7 days
```
или projected load test.
---
# 40. Search migration trigger #2 — DB contention
Evaluate dedicated search если search queries:
```text
≥25% of total DB execution time
```
и одновременно мешают transactional workload.
`25%` — Ekho internal threshold.
---
# 41. Search migration trigger #3 — CPU
Если search traffic является существенным фактором:
```text
DB CPU >70%
```
и его перенос разгрузит primary DB:
начать comparison отдельного search engine.
Supabase рассматривает sustained >70% CPU как горячую database load.
---
# 42. Search migration trigger #4 — relevance complexity
Dedicated search можно внедрить **до capacity limit**, если Postgres implementation становится слишком сложной для требуемого product behavior.
Например, если трудно устойчиво реализовать одновременно:
* typo tolerance;
* prefix autocomplete;
* aliases;
* abbreviations;
* deterministic ranking;
* faceting;
* synonyms;
* multilingual normalization;
* ranking tuning.
Это functional trigger.
---
# 43. Search migration trigger #5 — language support
Ekho global-first.
Если реальный multilingual dataset показывает, что текущая PostgreSQL text configuration / trigram strategy не обеспечивает необходимые languages и relevance:
это достаточный trigger для отдельного search evaluation.
Не ждать CPU saturation.
---
# 44. Не мигрировать Search при одном условии
Не мигрировать только потому что:
```text
Search engine feels more professional
```
или:
```text
big startups use one
```
Нужен measurable improvement.
---
# 45. Dedicated search rollout
Порядок:
```text
1. Build new index in parallel
2. Keep Postgres source of truth
3. Shadow production queries
4. Compare latency
5. Compare relevance
6. Validate sync correctness
7. Gradually route traffic
8. Keep rollback
```
---
# 46. Search shadow mode
До cutover:
production query отправляется:
```text
Primary → current Postgres search
```
а asynchronously:
```text
Shadow → candidate search engine
```
Пользователь получает только primary response.
Сохраняем comparison metrics.
---
# 47. Search migration success criteria
Новый engine должен доказать:
### Performance
```text
p95 ≤150 ms preferred
```
под expected peak load.
### Reliability
```text
error rate <0.5%
```
### Correctness
```text
no missing indexed active entities
```
### Relevance
manually curated benchmark queries проходят acceptance suite.
---
# 48. Search relevance benchmark
Создать фиксированный dataset, например:
```text
MIT
Massachusetts Institute Technology
UCL
University College London
ETH
computer science ETH
economics Milan
Cambrdge
Stanfrd
```
и edge cases из Search spec.
Каждый search change проходит regression suite.
---
# 49. Search source of truth
Даже после migration:
```text
Postgres
```
остаётся source of truth.
Search engine:
```text
derived index
```
Он должен быть полностью rebuildable.
---
# 50. Search sync
Нельзя полагаться только на:
```text
application code remembered to update search
```
Предпочтительно:
```text
DB change
↓
outbox / queue
↓
index worker
↓
search engine
```
---
# 51. Search sync metrics
Мониторить:
```text
search_index_lag
indexing_failures
missing_entities
stale_entities
queue_depth
```
---
# 52. Search freshness SLO
Для обычного university metadata:
```text
index update ≤5 min
```
Для critical admissions change:
```text
≤60 sec preferred
```
Это Ekho internal SLO.
---
# 53. Queue — использовать раньше Redis
Queue и cache решают разные проблемы.
Redis:
```text
reuse data quickly
```
Queue:
```text
process work asynchronously/reliably
```
Не использовать Redis как случайную background-job систему только потому, что Redis уже подключён.
---
# 54. Когда нужна queue
Queue нужна, если operation:
* не обязана закончиться внутри request;
* должна retry после failure;
* должна survive deployment;
* может быть processed later;
* может резко spike;
* должна decouple producer и consumer.
Для Ekho queues нужны именно для decoupling request handling от background processing и absorbing traffic spikes. Используется Cloudflare Workers queue/workflow mechanism, когда он соответствует documented trigger; Supabase Queues остаются допустимой provider capability, но не являются отдельным launch default.
---
# 55. Ekho queue candidates
С самого начала подходят для queue:
* university data ingestion;
* website monitoring;
* re-verification;
* source checking;
* search indexing;
* notification delivery;
* email;
* sitemap updates;
* IndexNow notifications;
* batch processing;
* non-interactive document processing.
---
# 56. Не делать queue для simple user mutation
Например:
```text
Save University
```
должен обычно завершать durable primary DB state непосредственно.
Не:
```text
click save
↓
queue
↓
maybe save later
```
если UX ожидает immediate confirmed state.
---
# 57. Queue lag SLO
## Interactive-background jobs
Например notification/index update after user action:
```text
p95 queue start lag ≤60 sec
```
## Data pipeline
```text
p95 lag ≤5 min
```
если source type не требует быстрее.
Это Ekho internal SLO.
---
# 58. Queue autoscaling trigger
Scale worker capacity если:
```text
queue lag >2× SLO
```
дольше:
```text
5 minutes
```
при стабильном incoming rate.
---
# 59. Queue health
Мониторить:
* queue depth;
* oldest message age;
* processing latency;
* retry count;
* failure rate;
* dead-letter count;
* throughput.
---
# 60. Retry policy
Background jobs должны быть:
```text
idempotent
```
где возможно.
Retry:
```text
exponential backoff + jitter
```
и finite attempt limit.
---
# 61. Dead-letter handling
После max attempts:
```text
dead-letter / failed state
```
а не infinite retry.
Critical failures должны создавать alert.
---
# 62. Worker service
Queue сама по себе не требует отдельного backend server.
На небольшом объёме:
```text
Queue
↓
serverless worker/function
```
достаточно.
---
# 63. Когда нужен dedicated worker
Рассматривать persistent worker service если одновременно наблюдается одно или несколько:
* continuously non-empty queue;
* high sustained throughput;
* long-running CPU-heavy jobs;
* large memory requirement;
* expensive function cold/start/runtime model;
* need for custom concurrency;
* need for persistent connections/processes;
* serverless limits становятся реальным constraint.
---
# 64. Serverless function capacity
Cloudflare Workers / OpenNext capacity и execution limits должны оцениваться по текущему plan/configuration и реальному измеренному workload.
Поэтому:
```text
we have 10k users
```
не является причиной писать отдельный backend.
---
# 65. Function duration
Cloudflare Workers / OpenNext имеют execution limits, зависящие от configuration/plan.
Если workload регулярно приближается к function duration limits:
не увеличивать timeout бесконечно.
Рассмотреть:
```text
Queue + worker/workflow
```
---
# 66. Отдельный backend — launch rule
**Не создавать отдельный backend service на launch только ради будущего scaling.**
Один application backend/BFF проще:
* deploy;
* debug;
* observe;
* secure;
* change.
---
# 67. User count не trigger backend extraction
Неверно:
```text
10k MAU → dedicated backend
```
```text
50k MAU → microservices
```
```text
100k MAU → Kubernetes
```
Таких универсальных thresholds не существует.
---
# 68. Separate backend trigger #1 — runtime requirement
Нужен отдельный runtime/service если workload требует:
* continuously running process;
* persistent workers;
* unsupported runtime;
* specialized CPU/memory;
* persistent connections;
* serverless model объективно не подходит.
---
# 69. Separate backend trigger #2 — independent scaling
Extract service если один domain:
```text
consumes ≥30% backend compute
```
и имеет:
```text
different scaling profile
```
от остального app.
`30%` — Ekho internal investigation threshold.
---
# 70. Separate backend trigger #3 — independent reliability
Extract если failure domain должен быть изолирован.
Пример:
```text
crawler/data pipeline outage
```
не должен:
```text
break applicant dashboard
```
---
# 71. Separate backend trigger #4 — deployment independence
Extract если один subsystem:
* часто deploy;
* имеет отдельную team ownership;
* требует другой release cadence;
* регулярно создаёт risk для main app deployments.
До этого оставляем вместе.
---
# 72. Separate backend trigger #5 — cost
Раз в месяц сравнивать:
```text
serverless cost
vs
persistent worker/service cost
```
Если dedicated service:
* materially дешевле;
* стабильно загружен;
* снижает latency/complexity;
тогда migration обоснована.
---
# 73. Не делать full backend rewrite
Если bottleneck только ingestion:
extract:
```text
ingestion worker
```
Не:
```text
rewrite entire Ekho backend
```
Если bottleneck search:
extract:
```text
search service/index
```
Не весь backend.
---
# 74. Service extraction principle
Разделять по:
```text
clear workload boundary
```
а не по database table.
Хорошие будущие boundaries:
```text
Search
Data ingestion
Change monitoring
Notifications
Heavy processing
```
Core user application workflow может оставаться одним backend значительно дольше.
---
# 75. Read replica — не launch default
Не добавлять read replica «на будущее».
Сначала primary.
---
# 76. Read replica trigger — read ratio
Supabase приводит:
```text
≥80% reads
```
как workload, где read replicas могут эффективно распределять load.
Это один из немногих внешне подтверждённых numerical scaling thresholds.
---
# 77. Read replica trigger — analytics
Если heavy analytics queries мешают production:
создать replica даже если обычный app traffic пока небольшой.
Supabase называет isolation analytical queries одной из основных причин использовать read replicas.
---
# 78. Read replica trigger — geography
Если реальные users в другом регионе имеют значительную database-read latency:
regional read replica может помочь read-only traffic.
Supabase официально указывает lower regional latency как use case для read replicas.
---
# 79. Не route writes на replica
Replica:
```text
SELECT
```
Primary:
```text
INSERT
UPDATE
DELETE
```
Supabase replicas являются read-only.
---
# 80. Read-after-write
Не отправлять на potentially lagging replica flow:
```text
user saves university
↓
immediately fetch saved state
```
если consistency requirement требует увидеть write мгновенно.
Такой read остаётся primary.
---
# 81. Good replica candidates
* public university reads;
* public program reads;
* catalog browsing;
* analytics;
* heavy reporting;
* non-critical discovery.
---
# 82. Poor replica candidates
* application creation confirmation;
* freshly completed task;
* immediately updated user profile;
* transaction workflows requiring latest data.
---
# 83. Analytics database scaling
Product analytics не должна превращать primary transactional database в data warehouse.
---
# 84. Stage 1 analytics
На старте:
* PostHog/product analytics provider;
* operational Postgres queries;
* small internal reports.
Не нужен warehouse.
---
# 85. Stage 2 analytics
Если internal analytical SQL начинает влиять на production:
```text
Read Replica
```
первая isolation step.
---
# 86. Stage 3 analytics warehouse
Supabase рекомендует специализированную analytical infrastructure, если analytical queries регулярно сканируют миллионы/десятки миллионов rows и long historical datasets.
Тогда:
```text
Postgres
↓ CDC / pipeline
Warehouse / analytical store
```
---
# 87. Warehouse trigger
Рассматривать warehouse если ≥2:
* reports regularly scan millions of rows;
* analytics hurts operational latency;
* multi-year event history;
* large aggregates;
* BI workloads;
* replica itself становится overloaded;
* analytics retention значительно превышает transactional retention.
---
# 88. CDN scaling
Public data должна масштабироваться сначала через edge caching.
Target:
```text
anonymous repeated university page request
```
не должен каждый раз выполнять expensive DB reconstruction.
---
# 89. CDN target
## Ekho internal target
Для stable public GET pages:
```text
edge/cache hit ratio >90%
```
при mature production traffic.
Если меньше:
сначала исследовать:
* cache headers;
* accidental cookies;
* URL fragmentation;
* query parameters;
* revalidation;
* personalization.
Не добавлять Redis первым.
---
# 90. Cache key explosion
Особенно следить за:
```text
?utm_source=
?ref=
?sort=
?filter=
```
Чтобы tracking parameters не превращали один public page в тысячи cache variants.
---
# 91. Public vs personalized cache
Public:
```text
shareable cache
```
Personalized:
```text
private/no shared cache
```
Не кэшировать personalized applicant data общим CDN cache.
---
# 92. Cache stampede
Если expensive cache item истёк:
не позволять:
```text
1000 simultaneous requests
↓
1000 DB recomputations
```
Использовать:
* stale-while-revalidate;
* request coalescing;
* lock/single-flight;
где необходимо.
---
# 93. Data pipeline scaling
University data pipeline — отдельный workload от application UX.
Нельзя позволять массовой re-verification:
```text
slow down Save University
```
---
# 94. Pipeline isolation
При росте ingestion:
```text
fetch
↓
queue
↓
parse
↓
normalize
↓
validate
↓
write
```
с controlled concurrency.
---
# 95. Pipeline concurrency
Не увеличивать scraper concurrency только чтобы queue стала пустой.
Учитывать:
* official website rate limits;
* robots/site rules;
* source stability;
* database write load;
* legal/data pipeline policies.
---
# 96. Pipeline scaling trigger
Scale workers когда:
```text
oldest job age > freshness SLO
```
а не просто:
```text
queue has many items
```
10,000 queued low-priority annual verifications могут быть нормальны.
100 critical deadline changes с 12-hour lag — нет.
---
# 97. Search + pipeline isolation
Search indexing не должно находиться внутри critical user transaction.
Не:
```text
Save DB data
↓
wait for search engine
↓
finish transaction
```
Лучше:
```text
commit DB
↓
enqueue index update
↓
return
```
---
# 98. Notifications
Notification delivery:
```text
queue
```
с retries.
Не отправлять SMTP/push synchronously внутри core DB transaction.
---
# 99. Rate limiting
Не добавлять Redis только ради rate limiting, если:
* CDN/WAF;
* API gateway;
* platform-level controls
уже позволяют реализовать нужный policy.
Redis-based distributed rate limiter нужен только если rate state действительно должен быть shared application-side.
---
# 100. Database write scaling
Read replicas не помогают write bottleneck.
Supabase прямо указывает:
```text
write-heavy → bigger primary compute
```
---
# 101. Write optimization
Перед horizontal write scaling:
* batch safe writes;
* remove unnecessary writes;
* reduce duplicate analytics writes;
* review indexes;
* async non-critical writes;
* optimize transactions;
* control queue concurrency.
---
# 102. Sharding
**Не проектировать sharding сейчас.**
Sharding — last-resort architecture.
---
# 103. Sharding не нужен при 100k MAU автоматически
Количество MAU не говорит:
* write rate;
* DB size;
* working set;
* query complexity.
Поэтому:
```text
100k MAU ≠ shard
```
---
# 104. Before sharding
Должны быть исчерпаны или объективно неподходящи:
```text
query optimization
indexes
connection pooling
caching
vertical compute
read replicas
workload separation
archiving
analytics offload
```
---
# 105. Sharding trigger
Только если primary write workload больше не может быть sustainably обслужен vertical scaling и architecture достигла hard limits.
Это не launch concern.
Supabase также указывает, что для большинства workloads текущий практический ответ остаётся larger compute или read replicas; их горизонтальная write-scaling система Multigres в 2026 всё ещё описывается как early.
Поэтому Ekho не должен строить production вокруг experimental sharding layer заранее.
---
# 106. Partitioning
Partitioning ≠ sharding.
PostgreSQL table partitioning можно рассматривать раньше для:
* huge append-only event tables;
* logs;
* historical changes;
* audit records;
если query/maintenance patterns объективно выигрывают.
Не partition every table.
---
# 107. Event/history data
Наиболее вероятные будущие large tables:
```text
analytics events
source checks
change observations
notification logs
crawl history
audit events
```
Не:
```text
universities
```
Сами 1,000 universities — крошечный dataset относительно DB scaling concern.
---
# 108. Current university catalog
Даже если:
```text
1,000 universities
```
и десятки тысяч programs:
это само по себе не justification для dedicated database architecture.
Search complexity и traffic важнее entity count.
---
# 109. 100k MAU architecture expectation
При хорошем caching и нормальном usage pattern вполне допустимо, что Ekho на 100k MAU всё ещё использует:
```text
1 primary Postgres
+
connection pooling
+
CDN
+
queue
```
а Redis/search replicas появляются только если metrics требуют.
Это architecture expectation, а не capacity guarantee.
---
# 110. Capacity must be load-tested
Нельзя заявлять:
```text
this architecture supports 100k MAU
```
только теоретически.
Нужно моделировать concurrency.
---
# 111. Existing load-test ladder
Использовать:
```text
100 concurrent dynamic users
↓
500
↓
1,000
```
и:
```text
5,000+ cached public spike
```
Также:
```text
10× spike test
```
и:
```text
soak test
```
---
# 112. Load-test routes
Минимум:
* university page;
* program page;
* search;
* filters;
* signup/auth;
* Save University;
* application creation;
* requirements;
* task completion;
* documents metadata;
* personalized requirements.
---
# 113. Capacity headroom
Не эксплуатировать систему на tested maximum.
## Ekho rule
Normal expected peak:
```text
≤50% of demonstrated stable load-test capacity
```
Это даёт примерно:
```text
2× headroom
```
на unexpected spikes.
---
# 114. Scale before launch event
Если ожидается large PR/launch:
проводить load test заранее.
Не ждать production failure.
Supabase production checklist также рекомендует load testing before production and отдельно советует предупреждать support при ожидаемом крупном traffic surge на соответствующих plans.
---
# 115. Scaling observability dashboard
Один dashboard:
## Application
```text
request_rate
p50
p95
p99
5xx
```
## Postgres
```text
CPU
connections
IO
query time
locks
```
## Search
```text
search QPS
p95
p99
zero results
errors
index lag
```
## CDN
```text
requests
hit ratio
origin requests
```
## Redis
```text
hit rate
memory
evictions
latency
```
## Queue
```text
depth
oldest age
lag
failure
retry
```
---
# 116. Alert severity
## P0
Immediate:
* DB unavailable;
* connection exhaustion;
* 5xx >10%;
* auth unavailable;
* data corruption;
* queue processing totally stopped for critical jobs.
## P1
Urgent:
* API p95 >2× budget;
* DB CPU >85% sustained;
* DB connections >90%;
* search unavailable;
* queue lag >5× SLO.
## P2
Investigation:
* DB CPU >70%;
* Redis eviction growth;
* search p95 >250ms;
* CDN hit ratio degradation;
* index lag >SLO.
---
# 117. Cost monitoring
Scaling architecture должна учитывать:
```text
performance per dollar
```
Не только speed.
Ежемесячно смотреть:
* DB compute;
* DB storage;
* bandwidth;
* function compute;
* search;
* Redis;
* queue;
* storage;
* observability.
---
# 118. Cost trigger
Если отдельный service появился для:
```text
“future scale”
```
но почти не используется:
remove it.
Infrastructure должна оправдывать:
* reliability;
* performance;
* developer productivity;
* cost.
---
# 119. Scaling decision document
Каждый major architecture addition:
```text
Redis
Search engine
Replica
Worker
Backend service
Warehouse
```
получает короткий ADR.
---
# 120. ADR fields
```text
Problem
Observed metrics
Current bottleneck
Options
Chosen solution
Expected improvement
Cost
Complexity
Rollback
Success metric
Review date
```
---
# 121. No scaling by intuition
Запрещено ADR:
```text
Reason:
We may need it later.
```
---
# 122. Redis ADR example trigger
```text
Problem:
requirements summary endpoint p95 620ms.
Evidence:
same data requested repeatedly;
query accounts for 24% DB execution time;
DB CPU reaches 74%.
Attempted:
indexes + query rewrite.
Hypothesis:
cache will reduce repeated DB reads.
Pilot success:
hit rate ≥70%;
DB workload -30%;
p95 ≤300ms.
```
Только тогда Redis оправдан.
---
# 123. Search ADR example
```text
Problem:
autocomplete p95 = 310ms
under expected peak.
Attempted:
GIN/trigram indexes;
derived search table;
query optimization.
Additional problem:
ranking SQL complexity increasing.
Candidate:
dedicated search engine.
Success:
p95 <150ms;
relevance regression suite passes;
index lag <60 sec;
zero missing active entities.
```
---
# 124. Replica ADR example
```text
Problem:
DB CPU 76%.
Traffic:
87% reads.
Queries:
optimized.
Observation:
catalog reads dominate.
Decision:
evaluate read replica.
```
Это совпадает с Supabase guidance, где 80%+ read traffic является хорошим candidate для read replicas.
---
# 125. Backend extraction ADR
```text
Problem:
crawler workers consume 40% backend compute
and have different runtime/retry/concurrency requirements.
Decision:
extract crawler worker.
Do NOT:
rewrite application API.
```
---
# 126. What NOT to add at launch
Без evidence:
```text
Redis
Kafka
Elasticsearch/OpenSearch cluster
Kubernetes
multiple DB primaries
database sharding
microservices
event-sourcing architecture
service mesh
```
---
# 127. Infrastructure complexity budget
Каждый additional stateful system означает:
* new failure mode;
* backups;
* security;
* secrets;
* monitoring;
* deployment;
* incident handling;
* vendor cost;
* data consistency problem.
Поэтому:
> Добавление infrastructure должно решать проблему больше, чем создаёт новых проблем.
---
# 128. Scaling stage 0 — Development
Architecture:
```text
Application
Postgres
Storage
Auth
```
Search:
```text
Postgres
```
Redis:
```text
NO
```
Replica:
```text
NO
```
Dedicated backend:
```text
NO
```
Dedicated search:
```text
NO
```
---
# 129. Scaling stage 1 — Launch
Add:
```text
CDN/cache
monitoring
connection pooling
queue where async correctness requires it
```
Still preferably:
```text
single primary Postgres
```
---
# 130. Scaling stage 2 — Early traction
First responses to load:
```text
query optimization
index optimization
cache tuning
compute upgrade
worker concurrency tuning
```
Не distributed architecture.
---
# 131. Scaling stage 3 — Measured bottlenecks
Теперь допустимы specialized components:
```text
Redis
Dedicated Search
Read Replica
Dedicated Worker
```
каждый только по отдельному ADR.
---
# 132. Scaling stage 4 — Large-scale workload
Только здесь обсуждать:
```text
analytical warehouse
multiple replicas
regional read architecture
service extraction
large event partitioning
```
---
# 133. Scaling stage 5 — Extreme scale
Только при реально доказанной необходимости:
```text
horizontal write scaling
sharding
multi-primary strategies
complex distributed infrastructure
```
До реальных metrics это не часть Ekho implementation roadmap.
---
# 134. Decision matrix — Postgres Search
## Keep Postgres Search
если:
```text
p95 ≤250ms
```
и:
```text
relevance acceptable
```
и:
```text
search not harming DB
```
---
## Investigate Search Engine
если:
```text
p95 >250ms after optimization
```
OR
```text
search ≥25% DB execution workload
```
OR
```text
search causes CPU >70%
```
OR
```text
required relevance/features become unmanageable
```
---
# 135. Decision matrix — Redis
## No Redis
если:
```text
DB healthy
+
CDN handles public reads
+
dynamic reads cheap
```
---
## Pilot Redis
если:
```text
repeatable expensive read workload
+
cacheable
+
material DB impact
```
---
## Keep Redis
только если pilot demonstrates:
```text
hit rate ≥70%
```
AND one of:
```text
DB read reduction ≥30%
```
or:
```text
material latency improvement
```
---
# 136. Decision matrix — DB compute
## Keep tier
```text
CPU <70%
+
latency budgets met
```
---
## Investigate
```text
CPU >70% sustained
```
---
## Upgrade compute
когда:
```text
query/index optimization completed
+
hardware remains bottleneck
```
---
# 137. Decision matrix — Read Replica
Consider when:
```text
reads ≥80%
```
AND primary load needs relief.
Или:
```text
analytics hurts production
```
Или:
```text
regional read latency is significant
```
---
# 138. Decision matrix — Queue
Use when:
```text
work can be asynchronous
+
requires retries/reliability
```
Не ждать traffic threshold.
Queue — architectural correctness mechanism, не только scaling tool.
---
# 139. Decision matrix — Dedicated worker
Consider when:
```text
queue continuously loaded
```
AND:
```text
serverless execution model is inefficient/limiting
```
или specialized runtime нужен.
---
# 140. Decision matrix — Separate Backend Service
Do not extract по MAU.
Extract when:
```text
independent scaling
OR
independent runtime
OR
independent failure domain
OR
independent deployment
```
имеет доказанную ценность.
---
# 141. Decision matrix — Warehouse
Keep analytics outside transactional critical path.
Move heavy analytics when:
```text
large historical scans
+
production contention
```
появляются регулярно.
Supabase отдельно указывает millions-of-rows analytical scans как case для dedicated analytical destinations.
---
# 142. Decision matrix — Sharding
```text
NO
```
до исчерпания:
```text
optimization
vertical scaling
replicas
cache
workload isolation
```
---
# 143. Scaling Rules must be automated
Codex должен реализовать metrics так, чтобы scaling decision можно было принимать из dashboard.
Нельзя через год обнаружить:
```text
we don't know why database is slow
```
---
# 144. Required DB metrics
Минимум:
```text
cpu_utilization
connection_utilization
db_size
query_calls
query_mean_time
query_total_time
read_write_ratio
locks
```
---
# 145. Required search metrics
```text
search_requests
search_p50
search_p95
search_p99
search_errors
zero_result_rate
index_lag
index_failures
```
---
# 146. Required cache metrics
```text
cdn_hit_ratio
redis_hit_ratio
redis_evictions
redis_memory
```
---
# 147. Required queue metrics
```text
queue_depth
oldest_message_age
processing_latency
retries
failures
dead_letters
```
---
# 148. Required API metrics
Per route:
```text
requests
p50
p95
p99
5xx
```
Especially:
```text
/search
/university
/applications
/requirements
/tasks
```
---
# 149. Weekly scaling review
До meaningful traction:
```text
monthly
```
После traction:
```text
weekly
```
проверять:
* latency;
* errors;
* DB;
* search;
* queue;
* cache;
* cost.
---
# 150. Automatic alerts
Scaling metrics должны alerts делать автоматически.
Не рассчитывать на:
```text
someone will notice it feels slow
```
---
# 151. Final prohibited rules
Нельзя создавать правила:
```text
10k users → Redis
```
```text
50k users → Elasticsearch
```
```text
100k users → microservices
```
```text
1M rows → sharding
```
```text
100k MAU → Kubernetes
```
Они технически необоснованны.
---
# 152. Final scaling ladder
```text
                     TRAFFIC GROWS
                           │
                           ▼
                      MEASURE
                           │
                           ▼
                 Query / Code problem?
                    │            │
                   YES          NO
                    │            │
                    ▼            ▼
                  FIX        Resource limit?
                                 │
                                 ▼
                         Vertical scaling
                                 │
                   ┌─────────────┼──────────────┐
                   │             │              │
               Read-heavy    Cacheable       Search-heavy
                   │             │              │
                   ▼             ▼              ▼
              Read Replica     Redis      Search Engine
                   │
                   │
        Background workload?
                   │
                   ▼
                 Queue
                   │
                   ▼
          Serverless insufficient?
                   │
                   ▼
          Dedicated Worker
                   │
                   ▼
        Domain independently scales?
                   │
                   ▼
           Separate Service
                   │
                   ▼
    Vertical/write scaling exhausted?
                   │
                   ▼
            Consider Sharding
```
---
# 153. Definition of Done
* [ ] Scaling decisions are metric-based.
* [ ] 100k MAU is not treated as concurrency.
* [ ] Existing performance SLOs remain authoritative.
* [ ] `pg_stat_statements` enabled/available.
* [ ] Slow query monitoring exists.
* [ ] `EXPLAIN ANALYZE` workflow documented.
* [ ] DB CPU monitored.
* [ ] DB connection utilization monitored.
* [ ] DB I/O monitored.
* [ ] read/write ratio measurable.
* [ ] DB CPU >70% triggers investigation.
* [ ] connection >75% triggers investigation.
* [ ] serverless DB connections use appropriate pooling.
* [ ] CDN used before Redis for public pages.
* [ ] Redis is not installed by default.
* [ ] Redis has explicit use case before adoption.
* [ ] Redis pilot success metrics defined.
* [ ] Redis data is always disposable/rebuildable where used as cache.
* [ ] Postgres remains initial Search implementation candidate.
* [ ] Search implementation is behind Search API abstraction.
* [ ] Search p95 target ≤250ms.
* [ ] autocomplete p95 target ≤150ms.
* [ ] Search migration triggers documented.
* [ ] Dedicated search index remains derived from Postgres.
* [ ] Search index can be rebuilt.
* [ ] Search index lag monitored.
* [ ] Queue used for retryable asynchronous jobs.
* [ ] Core user mutations do not unnecessarily depend on queues.
* [ ] Queue lag SLO exists.
* [ ] queue retries are bounded.
* [ ] failed jobs are observable.
* [ ] dedicated worker has measurable trigger.
* [ ] separate backend is not tied to user count.
* [ ] read replica is not installed prematurely.
* [ ] ≥80% read workload is a read-replica evaluation signal.
* [ ] read-after-write requests stay on primary where required.
* [ ] heavy analytics cannot degrade primary indefinitely.
* [ ] warehouse trigger documented.
* [ ] sharding explicitly excluded from launch architecture.
* [ ] public cache hit ratio monitored.
* [ ] production load tests exist.
* [ ] expected production peak stays below tested capacity.
* [ ] 2× headroom target exists.
* [ ] scaling cost monitored.
* [ ] every major infrastructure addition requires ADR.
* [ ] every infrastructure addition has rollback plan.
* [ ] every infrastructure addition has measurable success criteria.
---
# 154. Final architecture rule
Ekho должен начинаться **настолько просто, насколько возможно**, но не быть тупиком.
То есть:
```text
simple now
+
clear abstractions
+
monitoring
+
known migration triggers
```
а не:
```text
complex now
because maybe we get big
```
Правильная progression:
```text
Postgres
→ optimize
→ scale compute
→ cache/read replicas/search engine where metrics demand
→ isolate workers
→ extract services only when workloads diverge
→ sharding only as last resort
```
Именно поэтому мы заранее создаём:
* Search API abstraction;
* queue boundaries;
* cache boundaries;
* observability;
* stable IDs;
* derived search indexes;
* stateless application backend where possible;
но **не запускаем всю инфраструктуру заранее**.
---
# Trusted research basis
1. PostgreSQL 18 official documentation: Full Text Search, GIN indexes, `pg_trgm`, `pg_stat_statements`. PostgreSQL прямо называет GIN preferred full-text index type и предоставляет trigram similarity/index support.
2. Supabase official scaling guidance, January 2026: sustained CPU >70% означает hot database; ~80%+ read workload — candidate для replicas; write-heavy workloads масштабируются primary compute; сначала требуется query/index diagnostics.
3. Supabase official connection documentation: serverless/edge traffic должен использовать transaction-mode pooling; connection pool должен оставлять capacity для остальных database/platform workloads.
4. Supabase official Read Replica documentation: replicas предназначены для read capacity, geographic latency reduction и isolation read workloads.
5. Supabase official Queues documentation: queues предназначены для durable background tasks; для Ekho Cloudflare queue/workflow mechanisms используются только при documented trigger.
6. Redis official documentation: cache hit ratio, evictions, expiration и memory pressure являются основными cache-health signals; eviction policy управляет поведением при memory limits.
7. Cloudflare Workers/OpenNext runtime: background processing может быть вынесено в queues/workflows/workers без необходимости превращать весь application backend в отдельный сервер.
8. Supabase official analytics scaling guidance: heavy historical analytical workloads, регулярно сканирующие миллионы строк, являются поводом отделять analytical workload от transactional Postgres.
