## 1. Цель

Analytics в Ekho должна отвечать на пять основных вопросов:

1. Пользователь зарегистрировался?
    
2. Получил ли он первую реальную ценность — нашёл и сохранил университет?
    
3. Начал ли он реально использовать Ekho как application workspace?
    
4. Выполнил ли он первое полезное действие внутри application workflow?
    
5. Возвращается ли он после первого использования и продолжает ли получать ценность?
    

Главная последовательность:

`signup_completed`  
→ `university_saved`  
→ `application_created`  
→ `task_completed`  
→ `meaningful_return`

`meaningful_return` — **не отправляемое событие**, а вычисляемая метрика.

Retention-системы Mixpanel и Amplitude строят возврат именно как выполнение стартового события и последующее возвращение к другому событию, а не как искусственный event `user_returned`.

---

# 2. Базовые правила

## 2.1. Tracking Plan — обязательный источник истины

До добавления analytics event в код он должен существовать в Tracking Plan.

Для каждого события должны быть определены:

- `event_name`
    
- точное значение события;
    
- момент firing;
    
- client или server source;
    
- обязательные properties;
    
- optional properties;
    
- privacy classification;
    
- owner;
    
- dashboards/metrics, где event используется;
    
- `schema_version`;
    
- tests.
    

Amplitude прямо рекомендует определить tracking plan **до instrumentation**, чтобы events и properties имели единое значение для product/engineering/analytics. Segment использует тот же подход.

---

# 3. Naming convention

Использовать:

`object_action`

в `lower_snake_case`.

Примеры:

- `signup_completed`
    
- `university_saved`
    
- `application_created`
    
- `requirements_viewed`
    
- `task_completed`
    

Не использовать:

- `saveUniversity`
    
- `Saved`
    
- `click_save_button`
    
- `event_32`
    
- `applicationEvent`
    

Segment рекомендует Object–Action naming convention для behavioural events; Ekho использует ту же семантику, но стандартизирует casing как `snake_case`.

### Правило

Event описывает **то, что реально произошло**, а не UI element.

Правильно:

`university_saved`

Неправильно:

`save_button_clicked`

если университет фактически мог не сохраниться.

---

# 4. Event tiers

## P0 — Product Truth

Критические события:

- `signup_completed`
    
- `university_saved`
    
- `application_created`
    
- `task_completed`
    

Они используются для core funnel и должны иметь максимально надёжную instrumentation.

Для P0 изменение должно подтверждаться successful backend/domain state change.

---

## P1 — Core Product Behaviour

- `signup_started`
    
- `university_search_performed`
    
- `university_result_opened`
    
- `university_unsaved`
    
- `requirements_viewed`
    
- `application_status_changed`
    
- `task_reopened`
    
- `document_uploaded`
    
- `admissions_update_viewed`
    
- `notification_opened`
    

---

## P2 — Diagnostic

Например:

- `page_viewed`
    
- UI navigation
    
- feature exposure
    
- secondary interactions.
    

P2 events нельзя использовать как главный показатель получения пользователем ценности.

---

# 5. Core events

## 5.1. `signup_started`

Fire:

когда пользователь реально начинает authentication/signup flow.

Не fire:

- просто при открытии landing page;
    
- при показе signup CTA.
    

Properties:

- `auth_method`
    
    - `email`
        
    - `google`
        
    - `apple`
        
    - `other`
        
- `source_surface`
    
- `has_anonymous_history`
    

Purpose:

- signup conversion;
    
- signup drop-off.
    

---

# 6. `signup_completed`

Fire **один раз на пользователя** в момент, когда:

- создан persistent Ekho account;
    
- получен стабильный internal `user_id`;
    
- пользователь может пользоваться authenticated product.
    

Если verification обязательна для доступа, считать signup completed только после неё.

Source:

**server/domain preferred.**

Properties:

- `auth_method`
    
- `source_surface`
    
- `has_anonymous_history`
    

Не передавать:

- email;
    
- имя;
    
- пароль;
    
- auth token.
    

---

# 7. `university_search_performed`

Fire:

после завершения search request и получения результата.

Один search request = один `search_id`.

Properties:

- `search_id`
    
- `search_mode`
    
    - `query`
        
    - `filter`
        
    - `autocomplete`
        
    - `mixed`
        
- `query_present`
    
- `query_length`
    
- `filters_count`
    
- `result_count`
    
- `zero_results`
    
- `country_filter_count`
    
- `program_filter_present`
    
- `source_surface`
    

### Критическое правило

**Не отправлять raw search query в analytics.**

Причина: free-text поле потенциально может содержать персональную информацию, даже если поле предназначено только для названия университета.

Для analytics достаточно:

- длины query;
    
- наличия query;
    
- normalized matched entity;
    
- result count;
    
- search outcome.
    

GDPR требует purpose limitation и data minimisation — собирать только необходимые данные.

---

# 8. `university_result_opened`

Fire:

когда пользователь реально открыл university page из результата.

Properties:

- `university_id`
    
- `search_id` если существует;
    
- `result_position`
    
- `source_surface`
    
    - `search`
        
    - `autocomplete`
        
    - `saved`
        
    - `compare`
        
    - `recommendation`
        
    - `direct`
        

Purpose:

измерять:

`search → university opened`

---

# 9. `university_saved`

Это **первый главный value event Ekho**.

Fire:

после successful creation соответствующей связи в database.

Не fire:

- при click;
    
- при optimistic UI update;
    
- при duplicate save;
    
- если DB operation failed.
    

Source:

**server/domain.**

Properties:

- `university_id`
    
- `university_country_code`
    
- `source_surface`
    
- `search_id` если university была найдена через search.
    

Purpose:

- Time to First Value;
    
- signup → save conversion;
    
- search quality;
    
- university discovery adoption.
    

---

# 10. `university_unsaved`

Fire:

только когда сохранённый university действительно удалён из saved list.

Properties:

- `university_id`
    
- `source_surface`
    

Не считать unsave отрицательной product metric автоматически.

Пользователь может просто чистить shortlist.

---

# 11. `application_created`

Это **предварительный activation candidate** для Ekho.

Он означает, что человек перестал просто исследовать университеты и начал использовать Ekho как application workspace.

Fire:

после successful DB creation application.

Source:

**server/domain.**

Properties:

- `university_id`
    
- `university_country_code`
    
- `program_id` если существует;
    
- `source_surface`
    

Не включать:

- essay;
    
- notes;
    
- GPA;
    
- test scores;
    
- application documents;
    
- personal identifiers.
    

---

# 12. `requirements_viewed`

Fire:

когда requirements UI успешно отображён пользователю.

Properties:

- `university_id`
    
- `program_id` если существует;
    
- `requirements_mode`
    
    - `general`
        
    - `personalized`
        
- `satisfied_count`
    
- `missing_count`
    
- `optional_count`
    
- `unknown_count`
    
- `action_required_count`
    
- `source_surface`
    

Передаются только counts/statuses.

Не передавать:

- GPA;
    
- SAT score;
    
- IELTS/DET/TOEFL score;
    
- nationality;
    
- qualification value;
    
- personal explanation;
    
- raw requirement notes.
    

---

# 13. `application_status_changed`

Fire:

только при реальном state transition.

Properties:

- `from_status`
    
- `to_status`
    
- `university_id`
    

Разрешённые analytics statuses должны описывать workflow, например:

- `planning`
    
- `in_progress`
    
- `ready`
    
- `submitted`
    
- `archived`
    

### Не отправлять в third-party analytics

- accepted;
    
- rejected;
    
- waitlisted;
    
- admission decision text.
    

Эти данные остаются product data Ekho.

---

# 14. `task_completed`

Это главный workflow value event.

Fire:

только при transition:

`not_completed → completed`

Source:

**server/domain.**

Properties:

- `task_type`
    
- `task_origin`
    
    - `system`
        
    - `user`
        
- `source_surface`
    
- `deadline_proximity_bucket`
    
    - `overdue`
        
    - `0_3_days`
        
    - `4_7_days`
        
    - `8_30_days`
        
    - `30_plus_days`
        
    - `no_deadline`
        

Не отправлять:

- task title;
    
- task description;
    
- exact user note.
    

---

# 15. `task_reopened`

Fire:

при:

`completed → not_completed`

Properties:

- `task_type`
    
- `task_origin`
    

Если пользователь затем снова завершит task:

`task_completed`

может fire снова как новый state transition.

Но метрика:

**users who completed first task**

использует first occurrence `task_completed`, а не raw event count.

---

# 16. `document_uploaded`

Fire:

после successful upload + сохранения metadata.

Properties:

- `source_surface`
    

Не отправлять в analytics:

- filename;
    
- URL;
    
- file contents;
    
- OCR content;
    
- document owner;
    
- passport/transcript details.
    

---

# 17. `admissions_update_viewed`

Fire:

когда пользователь открыл Live Admissions Update.

Properties:

- `university_id`
    
- `update_type`
    
- `source_surface`
    

Не отправлять персонализированный текст объяснения.

---

# 18. `notification_opened`

Позволяет понимать, что реально возвращает пользователей.

Properties:

- `channel`
    
    - `email`
        
    - `push`
        
    - `in_app`
        
- `notification_type`
    
    - `deadline`
        
    - `admissions_update`
        
    - `task`
        
    - `application`
        
- `destination_surface`
    

Использовать для attribution:

notification → return → meaningful action.

---

# 19. Что такое `meaningful_return`

**Не создавать event `user_returned`.**

Return вычисляется из существующих events.

Пользователь считается Meaningfully Returned, если после предыдущего периода использования он снова выполняет хотя бы одно core action:

- `university_search_performed`
    
- `university_result_opened`
    
- `university_saved`
    
- `application_created`
    
- `requirements_viewed`
    
- `task_completed`
    
- `document_uploaded`
    
- `application_status_changed`
    
- `admissions_update_viewed`
    

Простой:

`page_viewed`

или:

`session_started`

сам по себе **не считается meaningful return**.

Это предотвращает ситуацию:

пользователь случайно открыл Ekho → ничего не сделал → analytics считает его retained.

Retention analysis по определению анализирует выполнение начального event и последующее возвращение к другому event.

---

# 20. Core Funnel

Основной lifecycle funnel:

### Step 1

`signup_completed`

↓

### Step 2

`university_saved`

↓

### Step 3

`application_created`

↓

### Step 4

`task_completed`

↓

### Step 5

`meaningful_return`

Amplitude Funnel Analysis также строится на последовательности behavioural events пользователя.

---

# 21. Funnel measurement

Для каждого funnel step показывать:

- unique users;
    
- conversion from previous step;
    
- conversion from signup;
    
- median time-to-convert;
    
- P75 time-to-convert;
    
- drop-off;
    
- cohort.
    

Не использовать event count вместо unique users для funnel conversion.

---

# 22. Conversion windows

Не объявлять один произвольный срок «правильным».

Показывать отдельно:

- within 24 hours;
    
- within 7 days;
    
- within 30 days.
    

Например:

`signup → university_saved within 24h`

и отдельно:

`signup → university_saved within 7d`.

Когорту включать только если прошло достаточно времени для полного measurement window.

Пример:

для 30-day conversion нельзя использовать пользователя, зарегистрированного пять дней назад, как окончательно non-converted.

---

# 23. Time to Value

Измерять:

### `time_to_first_search`

`signup_completed → first university_search_performed`

### `time_to_first_university_view`

`signup_completed → first university_result_opened`

### `time_to_first_save`

`signup_completed → first university_saved`

### `time_to_first_application`

`signup_completed → first application_created`

### `time_to_first_task_completion`

`signup_completed → first task_completed`

Главная immediate-value metric:

`% users reaching university_saved within ~30 seconds of starting the usable product experience`

Потому что один из фундаментальных принципов Ekho — дать полезный результат примерно за 30 секунд.

Не ставить сейчас выдуманный benchmark conversion rate.

Сначала получить реальные Ekho cohorts.

---

# 24. Activation

Сейчас **нет подтверждённых данных Ekho**, позволяющих утверждать, какой event причинно определяет retention.

Поэтому:

### Candidate A

`university_saved`

### Candidate B

`application_created`

### Candidate C

`task_completed`

Первоначальная operating hypothesis:

**application_created после university_saved = provisional activation.**

Но это гипотеза, не установленный факт.

После накопления данных сравнить retention пользователей:

- signup only;
    
- university_saved;
    
- application_created;
    
- task_completed.
    

Если определённое действие стабильно связано с более высоким последующим retention, его можно рассматривать как более сильный activation milestone.

Correlation сама по себе не доказывает causation; behavioural analytics-инструменты также используют такие event correlations как диагностический инструмент, а не доказательство причинности.

---

# 25. Retention

Основной принцип:

**мерить возврат к ценности, а не возврат к странице.**

Primary cohorts:

### W1

пользователь делает ≥1 meaningful action через 7–13 дней после signup.

### W4

пользователь делает ≥1 meaningful action через 28–34 дня после signup.

Дополнительно:

- D1
    
- D7
    
- D30
    

Но для admissions workspace weekly retention потенциально информативнее daily retention, потому что продукт не обязан использоваться ежедневно.

Это продуктовая гипотеза Ekho, которую затем проверяем данными.

---

# 26. Provisional engagement metric

Не фиксировать окончательный North Star до реальных данных.

На launch использовать operating metric:

## Weekly Meaningful Active Users — WMAU

Количество unique users за неделю, совершивших хотя бы один meaningful core action.

Не считать WMAU по:

- login;
    
- session;
    
- pageview.
    

---

# 27. Search Analytics

Search — отдельный critical subsystem.

Измерять:

### Search usage

`users performing search / active users`

### Zero Result Rate

`searches with result_count = 0 / all searches`

### Result Open Rate

`searches followed by university_result_opened / searches`

### Search → Save Rate

`searches that lead to university_saved / searches`

Связь обеспечивается через:

`search_id`.

Также измерять:

- result position;
    
- filters count;
    
- autocomplete usage;
    
- country filter usage;
    
- program filter usage.
    

Не хранить raw search text.

---

# 28. University Intelligence analytics

Измерять:

- university pages opened;
    
- search → university page;
    
- university page → save;
    
- save → application;
    
- requirements viewed;
    
- personalized vs general requirements usage.
    

Не оптимизировать продукт под количество page views.

Главное:

**приводит ли University Intelligence к реальному application action.**

---

# 29. My Applications analytics

Измерять:

- applications created;
    
- users with ≥1 application;
    
- requirements viewed;
    
- first task completed;
    
- tasks completed per active applicant;
    
- document upload adoption;
    
- application workflow progress.
    

Количество текущих applications/tasks должно вычисляться из production database.

Event analytics не является source of truth для текущего состояния пользователя.

---

# 30. Common event envelope

Каждый analytics event должен поддерживать:

```text
event_id
event_name
occurred_at
user_id
anonymous_id
session_id
schema_version
source
platform
app_version
environment
properties
```

### `event_id`

UUID.

Используется для idempotency/deduplication.

Duplicate events — реальная проблема event pipelines; Segment использует `messageId` для deduplication, PostHog использует UUID-based deduplication.

### `occurred_at`

UTC timestamp.

### `user_id`

Internal immutable Ekho UUID.

Никогда email.

### `anonymous_id`

Используется только до identification.

### `session_id`

Один browsing/application session.

### `schema_version`

Начать:

`1`

### `source`

- `client`
    
- `server`
    

### `environment`

- `production`
    
- `staging`
    
- `development`
    

---

# 31. Identity

До signup:

anonymous identity.

После успешного signup:

anonymous identity должна корректно связаться с stable Ekho `user_id`.

PostHog и аналогичные product analytics системы используют именно anonymous → identified identity linking, чтобы pre-signup и post-signup behaviour принадлежали одному пользователю.

### Правила

- stable ID = Ekho DB user UUID;
    
- никогда не использовать email как primary analytics identifier;
    
- `null`, `undefined`, `guest`, `anonymous` нельзя использовать как shared user ID;
    
- logout должен корректно reset anonymous identity;
    
- один пользователь не должен превращаться в несколько analytics profiles из-за разных SDK.
    

---

# 32. User properties

Разрешённый минимальный набор:

- `account_created_at`
    
- `locale`
    
- `plan`
    
    - `free`
        
    - `paid`
        
- `acquisition_channel`
    
- `profile_completeness_bucket`
    

Дополнительные user profile данные не отправлять автоматически.

Если для аналитики понадобится nationality, education system, GPA или другие applicant dimensions:

сначала документировать **конкретный analytics purpose**.

По умолчанию такие данные остаются в first-party Ekho database.

---

# 33. Запрещённые analytics данные

Никогда не отправлять в third-party product analytics:

- full name;
    
- email;
    
- phone;
    
- date of birth;
    
- address;
    
- passport information;
    
- auth tokens;
    
- password;
    
- raw nationality/profile forms без отдельной необходимости;
    
- GPA;
    
- SAT/ACT;
    
- DET/IELTS/TOEFL scores;
    
- essays;
    
- notes;
    
- task free text;
    
- recommendation contents;
    
- document names;
    
- uploaded document contents;
    
- transcript contents;
    
- financial aid documents;
    
- exact financial information;
    
- application decision;
    
- acceptance/rejection information;
    
- raw search queries;
    
- raw URLs containing identifiers;
    
- URL query parameters без allowlist.
    

European Commission отдельно указывает, что IP address и cookie identifiers также могут являться personal data.

---

# 34. Privacy principles

Analytics должна соблюдать:

- purpose limitation;
    
- data minimisation;
    
- storage limitation;
    
- accuracy;
    
- integrity/confidentiality;
    
- accountability.
    

Это непосредственно основные GDPR principles.

### Поэтому

Не использовать подход:

> «соберём всё сейчас, потом вдруг пригодится».

Каждое property должно иметь конкретный use case.

---

# 35. Consent

Analytics architecture должна поддерживать:

- consent state;
    
- opt-out;
    
- analytics disabled where legally required;
    
- deletion request;
    
- user account deletion propagation.
    

Не считать автоматически, что любой analytics tracking всегда требует consent или что consent всегда является единственным lawful basis — это зависит от конкретного processing/vendor/cookie implementation.

Финальная legal basis определяется Security & Privacy layer.

---

# 36. Session Replay

Для authenticated Ekho application:

**OFF by default до отдельной privacy validation.**

Если когда-либо включается:

- mask all inputs;
    
- mask all text by default;
    
- never capture document screens;
    
- never capture essays;
    
- never capture notes;
    
- never capture application personal data;
    
- scrub network bodies;
    
- scrub query strings;
    
- exclude sensitive routes.
    

PostHog прямо предоставляет masking inputs/text и предупреждает, что network requests/responses могут содержать sensitive information.

---

# 37. Autocapture

Autocapture не использовать как источник core product metrics.

Можно использовать ограниченно для UX investigation.

Core metrics должны использовать explicitly defined events.

Причины:

- стабильная semantics;
    
- меньше analytics noise;
    
- проще schema;
    
- проще privacy review;
    
- metrics не ломаются после изменения DOM/UI.
    

Amplitude также рекомендует управляемую taxonomy и documented events вместо хаотичного набора telemetry.

---

# 38. Client vs Server ownership

## Server/domain events

- `signup_completed`
    
- `university_saved`
    
- `university_unsaved`
    
- `application_created`
    
- `application_status_changed`
    
- `task_completed`
    
- `task_reopened`
    
- `document_uploaded`
    

Fire только после successful state change.

---

## Client behavioural events

- `university_result_opened`
    
- `requirements_viewed`
    
- navigation/page views.
    

---

## Search

`university_search_performed`

должен fire только после завершённого search request, чтобы были известны:

- `result_count`
    
- `zero_results`.
    

---

# 39. Critical rule: exactly one logical event

Один domain transition = один logical analytics event.

Network retry не должен создавать вторую logical action.

Каждый critical event получает deterministic/stable `event_id` для retry/deduplication.

Segment и PostHog отдельно реализуют deduplication именно потому, что retries могут создать duplicate telemetry.

---

# 40. Production isolation

Production metrics никогда не должны включать:

- local development;
    
- staging;
    
- automated tests;
    
- internal synthetic monitoring;
    
- bots.
    

Предпочтительно:

отдельные analytics projects/environments.

Если provider этого не позволяет:

обязательный `environment` property + hard production filters.

---

# 41. Dashboard 1 — Core Funnel

Показывать:

`signup_completed`

↓

`university_saved`

↓

`application_created`

↓

`task_completed`

↓

`meaningful_return`

Для каждого:

- unique users;
    
- conversion;
    
- drop-off;
    
- median time;
    
- P75 time;
    
- 24h;
    
- 7d;
    
- 30d.
    

Breakdowns:

- acquisition channel;
    
- platform;
    
- destination university country;
    
- new vs returning cohort.
    

---

# 42. Dashboard 2 — Time to Value

Показывать:

- signup → first search;
    
- signup → first university opened;
    
- signup → first save;
    
- signup → first application;
    
- signup → first task completed.
    

Statistics:

- P50;
    
- P75;
    
- P90.
    

Также:

`% reaching university_saved within ~30 seconds`

Не ставить artificial success percentage до launch data.

---

# 43. Dashboard 3 — Retention

Cohorts by signup week.

Показывать:

- W1 meaningful retention;
    
- W4 meaningful retention;
    
- D1;
    
- D7;
    
- D30.
    

Сравнения:

- saved vs never saved;
    
- application created vs not created;
    
- task completed vs not completed;
    
- acquisition source;
    
- platform.
    

Цель:

найти реальные behaviours, связанные с долгосрочным использованием Ekho.

---

# 44. Dashboard 4 — Search Quality

Показывать:

- searches;
    
- unique search users;
    
- zero-result rate;
    
- result-open rate;
    
- search → save rate;
    
- autocomplete usage;
    
- filters usage;
    
- result position → open;
    
- result position → save.
    

---

# 45. Dashboard 5 — Applications

Показывать:

- users creating first application;
    
- applications per activated user;
    
- requirements viewed;
    
- personalized requirements adoption;
    
- users completing first task;
    
- task completion;
    
- document upload adoption.
    

---

# 46. Dashboard 6 — Analytics Health

Отдельный технический dashboard.

Показывать:

- P0 events received;
    
- schema violations;
    
- duplicate rate;
    
- missing required properties;
    
- unknown events;
    
- anonymous/identified identity failures;
    
- events from staging accidentally entering prod;
    
- ingestion anomalies.
    

Segment поддерживает schema validation, violations и blocking events/properties, которые не соответствуют Tracking Plan.

---

# 47. Analytics schema validation

Любое новое событие должно быть заранее известно schema.

Неизвестное:

`event_name`

или неизвестный property не должны молча становиться новым analytics contract.

Использовать typed event definitions.

Например conceptual API:

```text
analytics.track("university_saved", {
  university_id,
  university_country_code,
  source_surface,
  search_id
})
```

Не разрешать:

```text
analytics.track(dynamicEventName, arbitraryObject)
```

Segment даже предоставляет strongly typed analytics libraries, создаваемые из tracking plan, именно для предотвращения schema drift.

---

# 48. Analytics wrapper

Application code не должен напрямую зависеть от конкретного analytics vendor.

Использовать один Ekho abstraction:

```text
analytics.identify(...)
analytics.track(...)
analytics.reset(...)
```

Никаких прямых provider SDK calls по всему repository.

Это позволяет:

- менять analytics provider;
    
- централизованно фильтровать PII;
    
- enforce schema;
    
- контролировать consent;
    
- добавлять tests;
    
- предотвращать duplicate events.
    

---

# 49. Event versioning

Нельзя менять значение production event задним числом.

Если semantics существенно изменяется:

- создать новую версию;
    
- увеличить `schema_version`;
    
- deprecated old definition;
    
- обновить dashboards.
    

Название event не должно означать две разные вещи в разные периоды времени.

---

# 50. QA tests — обязательные

## Signup

Одна успешная регистрация:

→ ровно один `signup_completed`.

Refresh:

→ никаких новых signup events.

---

## Save university

Первый save:

→ `university_saved`.

Повторный click при уже saved:

→ нового event нет.

Unsave:

→ `university_unsaved`.

Save снова:

→ новый `university_saved`, потому что произошёл новый реальный state transition.

---

## Application

Successful create:

→ один `application_created`.

Failed request:

→ никакого `application_created`.

Refresh:

→ никакого нового event.

---

## Task

Incomplete → complete:

→ `task_completed`.

Refresh:

→ ничего.

Complete → reopened:

→ `task_reopened`.

Reopened → completed:

→ новый `task_completed`.

---

# 51. Identity QA

Проверить flow:

anonymous visitor

→ search

→ university opened

→ signup

→ save

После identify все допустимые pre-signup events должны принадлежать тому же логическому пользователю, если такое linking разрешено consent/privacy configuration.

Проверить:

- refresh;
    
- logout;
    
- login;
    
- second device;
    
- OAuth;
    
- email auth;
    
- anonymous visitor;
    
- deleted account.
    

Identity fragmentation способна создавать несколько analytics users для одного человека; PostHog отдельно рекомендует проверять, что anonymous и identified IDs корректно объединены.

---

# 52. Privacy QA

Автоматический test должен гарантировать отсутствие запрещённых properties:

- `email`
    
- `name`
    
- `phone`
    
- `password`
    
- `token`
    
- `essay`
    
- `note`
    
- `filename`
    
- `document_url`
    
- `raw_query`
    
- `gpa`
    
- test scores.
    

Добавление потенциально sensitive property должно требовать explicit review.

---

# 53. Analytics reconciliation

P0 events должны периодически сверяться с actual product state/domain transitions.

Например:

- real account creation vs `signup_completed`;
    
- successful saved-university transitions vs `university_saved`;
    
- real application creation vs `application_created`;
    
- task state transition vs `task_completed`.
    

Product analytics vendor **не является primary database**.

Database остаётся source of truth.

Analytics отражает behaviour.

---

# 54. Что не измерять на launch

Не создавать десятки событий вида:

- every button clicked;
    
- every hover;
    
- every scroll;
    
- sidebar opened;
    
- dropdown opened;
    
- modal viewed;
    
- tooltip viewed;
    
- cursor movement.
    

Добавлять interaction event только если существует конкретный product question, на который без него нельзя ответить.

---

# 55. Что нельзя делать

Не:

- считать pageviews North Star;
    
- считать login retention;
    
- считать session start meaningful activity;
    
- использовать event counts вместо unique users для funnel;
    
- использовать frontend click как подтверждение backend action;
    
- менять semantics существующих events;
    
- отправлять arbitrary user-generated text;
    
- использовать email как analytics user ID;
    
- смешивать staging и production;
    
- создавать события напрямую из разных компонентов без analytics wrapper;
    
- включать session replay на sensitive screens без masking;
    
- собирать данные «на всякий случай».
    

---

# 56. Launch metrics

До появления реальных данных Ekho отслеживает:

### Acquisition

`signup_started → signup_completed`

### First Value

`signup_completed → university_saved`

### Activation candidate

`university_saved → application_created`

### Workflow Value

`application_created → task_completed`

### Retention

`task/application/university activity → meaningful_return`

### Search Quality

`search → university opened → university saved`

### Time to Value

`signup → first university_saved`

---

# 57. Не устанавливать сейчас выдуманные benchmarks

Не писать:

- «activation должна быть 60%»;
    
- «D7 retention должен быть 35%»;
    
- «search save rate должна быть 20%».
    

У Ekho пока нет historical cohorts, поэтому такие значения были бы информацией из воздуха.

Правильный порядок:

1. Instrument correctly.
    
2. Launch.
    
3. Получить baseline.
    
4. Разделить cohorts.
    
5. Проверить data quality.
    
6. Найти activation behaviour.
    
7. Только потом установить targets.
    

---

# 58. Definition of Done

Analytics считается готовой только если:

-  Tracking Plan существует.
    
-  Все P0/P1 events документированы.
    
-  Event semantics однозначны.
    
-  Используется единая naming convention.
    
-  Есть typed analytics schema.
    
-  Есть один analytics wrapper.
    
-  P0 events подтверждаются backend state.
    
-  Events имеют unique `event_id`.
    
-  Есть deduplication.
    
-  Anonymous → identified flow протестирован.
    
-  Production отделён от staging/dev.
    
-  Raw free text не отправляется.
    
-  PII denylist действует.
    
-  Search query не отправляется.
    
-  Session replay sensitive screens выключен.
    
-  Core funnel настроен.
    
-  Time-to-value dashboard настроен.
    
-  Search dashboard настроен.
    
-  Retention cohorts настроены.
    
-  Analytics-health dashboard настроен.
    
-  P0 events сверяются с product/domain data.
    
-  Account deletion распространяется на analytics согласно Privacy policy.
    
-  Новые events нельзя добавить без обновления Tracking Plan.
    

---

# 59. Финальная структура Ekho Analytics

```text
Signup
↓
signup_completed

Discovery
↓
university_search_performed
↓
university_result_opened
↓
university_saved

Workspace
↓
application_created
↓
requirements_viewed

Action
↓
task_completed

Retention
↓
meaningful core action in a later period
```

Главная логика Analytics Ekho:

**не измерять, сколько пользователь нажимает.**

Измерять:

**как быстро он доходит до ценности → начинает управлять application → выполняет следующий action → возвращается за следующей ценностью.**

---

# Research basis

- Amplitude: Tracking Plan должен заранее определять events, properties, причины сбора и source instrumentation.
    
- Segment: tracking plan, Object–Action naming, schema validation и violations.
    
- Amplitude: Funnel Analysis строится как последовательность behavioural events.
    
- Mixpanel: retention определяется через исходное событие и последующее возвращение к событию.
    
- Segment/PostHog: event IDs используются для deduplication retries/duplicate data.
    
- PostHog: anonymous → identified identity resolution используется для объединения поведения до и после signup.
    
- European Commission: purpose limitation, data minimisation, storage limitation, integrity/confidentiality и другие GDPR principles.
    
- PostHog Session Replay: inputs/text/network information требуют masking и privacy controls.