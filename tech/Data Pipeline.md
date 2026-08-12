Важно: это **техническая и продуктовая политика снижения риска, а не юридическое заключение**. Перед публичным коммерческим запуском в ЕС/США я бы один раз дал её проверить IP/privacy-юристу.
# EKHO — DATA PIPELINE v1

1. **Главный принцип — Ekho не “scraping company”.** Источники идут по приоритету:
    
```text
1. Open official API / dataset
2. Government / regulator open data
3. Direct university feed / permission
4. Public official university page
5. Manual verification

НЕ:
competitor database
paywalled database
logged-in content
private APIs
student accounts
```

ROR, например, специально предоставляет registry dump/API под CC0 без ограничений использования. HESA публикует ряд UK higher-education datasets как open data, в том числе под CC BY 4.0. ([Research Organization Registry (ROR)](https://ror.org/?utm_source=chatgpt.com "Research Organization Registry (ROR) | Home"))

2. **Нам нужен Legal Gate до любого crawler.** Codex не должен позволять новому домену автоматически попадать в pipeline.

```text
SOURCE
↓
LEGAL POLICY CHECK
↓
FETCH
```

Для каждого source храним:

```text
source_id

access_type
license
license_url
terms_url

commercial_use_allowed
redistribution_allowed
automated_access_allowed

robots_status
tdm_reservation_status

full_snapshot_allowed
image_reuse_allowed

attribution_required

policy_status

last_policy_review_at
```

3. **Три уровня источников.**

```text
GREEN
можно автоматически использовать

YELLOW
можно извлекать только факты по строгим правилам

RED
не трогаем
```

4. **GREEN source.**
Только когда есть достаточно ясные права, например:

```text
CC0
CC BY permitting commercial use
Open Government licence
public-domain dataset
explicit commercial-use API licence
direct written permission
```

ROR → отличный пример GREEN: registry metadata находится под CC0. ([Research Organization Registry (ROR)](https://ror.org/registry/?utm_source=chatgpt.com "Requesting registry updates"))

5. **YELLOW source.**
Типичный пример:

```text
admissions.stanford.edu/...
```

Публичная официальная страница университета, но без явной open-data лицензии.

Ekho может рассматривать её для извлечения **минимальных фактических данных**, только после проверки:

```text
public access
no login
no paywall
no CAPTCHA bypass

robots permits
terms do not prohibit intended use
no explicit rights reservation blocking use

no personal data collection

low crawl rate
```

Это не означает «всё публичное разрешено копировать».

6. **RED source.**

Автоматически запрещаем:

```text
login required
paywall

CAPTCHA
anti-bot circumvention

private API
stolen/leaked dataset

competitor database

terms prohibit scraping

robots Disallow

commercial reuse forbidden

unknown restrictive licence

personal applicant data
```

7. **Конкурентов вообще не используем как source of truth.**

Например:

```text
Scoir
AdmissionsOS
CollegeVine
commercial rankings/databases
```

Scoir сейчас прямо запрещает scraping, harvesting, downloading или extracting данных из своего сервиса без явного разрешения. ([Scoir](https://www.scoir.com/terms-of-service "Privacy and Legal Policies | Scoir"))

И нам эти данные всё равно менее нужны, чем первоисточник.

8. **Очень важное: UCAS Courses Data сейчас НЕ использовать для Ekho по стандартной лицензии.**

Текущая лицензия UCAS разрешает использование только для согласованной цели, а clause 4.10 прямо говорит, что использование Courses Data для создания admissions service, который может конкурировать с UCAS, является material breach. ([UCAS](https://www.ucas.com/terms-and-conditions/sale-of-products-services/sale-of-products-services-courses-data-service?utm_source=chatgpt.com "Sale of products & services: Courses Data Service"))

То есть:

```text
UCAS public data product
≠
free data for Ekho
```

Для UK предпочтительнее:

```text
university official sources
HESA / Discover Uni open datasets
other properly licensed government sources
```

HESA действительно публикует Discover Uni dataset как open data. ([HESA](https://www.hesa.ac.uk/support/tools-and-downloads/unistats?utm_source=chatgpt.com "Discover Uni dataset (formerly Unistats) - HESA"))

9. **Не полагаемся на “ну информация же публичная”.**
    

В ЕС существует отдельное **sui generis database right**: правообладатель может препятствовать extraction/re-utilisation существенной части защищённой базы. Это может применяться даже к базе, которая свободно доступна через интернет. ([Eur-Lex](https://eur-lex.europa.eu/eli/dir/1996/9/oj/eng?utm_source=chatgpt.com "Directive - 96/9 - EN - EUR-Lex - European Union"))

Поэтому:

```text
scrape 100k UCAS records
→ плохая идея

прочитать официальный Stanford deadline
→ совершенно другой риск-профиль
```

10. **Факты и чужой текст — не одно и то же.**
    

Например:

```text
Deadline: January 5
Tuition: $68,000
DET minimum: 125
```

— фактические данные.

Но целый paragraph университета:

> длинное описание admission policy...

— уже чужое выражение.

U.S. Copyright Office прямо указывает, что copyright не защищает сами факты, но может защищать форму их выражения. ([Офис авторских прав США](https://www.copyright.gov/help/faq/faq-protect.html?utm_source=chatgpt.com "What Does Copyright Protect? (FAQ) | U.S. Copyright Office"))

Поэтому Ekho:

```text
extract → normalize → paraphrase
```

а не:

```text
copy → republish
```

11. **Не копируем university descriptions.**
    

Не:

```text
copy 600-word Stanford description
```

А создаём собственное структурированное представление:

```text
Location
Institution type
Programs
Requirements
Deadlines
Costs
Aid
```

12. **Полные HTML snapshots — поправка к прошлой Data Architecture.**
    

Я бы **НЕ сохранял автоматически полный HTML/PDF каждой страницы навечно**.

Вместо этого:

```text
URL
page title
HTTP metadata

content hash

section hash

retrieved_at

parser version

normalized facts

source locator
```

Полный snapshot разрешён только если:

```text
open licence
explicit permission
internal retention clearly permitted
```

Почему: коммерческий text/data mining в ЕС не является универсальным разрешением на копирование — правообладатель может надлежащим образом зарезервировать TDM rights; для online content такая reservation может быть machine-readable и отражаться также через metadata/website terms. ([Eur-Lex](https://eur-lex.europa.eu/legal-content/EN/TXT/PDF/?uri=CELEX%3A32019L0790&utm_source=chatgpt.com "[PDF] DIRECTIVE (EU) 2019/ 790 OF THE EUROPEAN PARLIAMENT ..."))

13. **Поэтому Evidence v1 меняем на:**
    

```text
fact_id

source_url
source_id

page_title

source_locator
section_heading

raw_factual_value
normalized_value

content_hash
section_hash

observed_at
verified_at
```

И только при разрешении:

```text
snapshot_id
```

14. **Пользователю обычно не показываем copied paragraph.**

Показываем:

```text
DET requirement
125 minimum

Source:
Official Admissions

Verified:
Aug 12, 2026

[Open official source]
```

Это и чище UX, и юридически консервативнее.

15. **Crawler обязан соблюдать robots.txt.**
    

RFC 9309 стандартизирует Robots Exclusion Protocol как способ владельца сайта управлять поведением crawlers. ([RFC Editor](https://www.rfc-editor.org/info/rfc9309/?utm_source=chatgpt.com "RFC 9309: Robots Exclusion Protocol"))

Но:

```text
robots Allow
≠ legal licence
```

Поэтому нужны одновременно:

```text
robots
+
terms
+
licence
+
source policy
```

16. **Для Ekho robots Disallow = hard stop.**
    

Даже если существуют юридические аргументы, почему robots не равен закону.

Нам нет смысла судиться с университетом ради одного deadline.

```text
Disallow
→ automatic crawler disabled
→ manual/legal source alternative
```

17. **Никакого обхода технических ограничений.**
    

Codex должен получить жёсткий prohibition:

```text
NO CAPTCHA solving
NO proxy rotation to evade blocking
NO fake browser fingerprints
NO session-cookie theft
NO authentication bypass
NO rate-limit bypass
NO hidden/private API reverse engineering
```

18. **Crawler должен идентифицировать себя.**
    

Например user-agent:

```text
EkhoBot/1.0
Admissions information monitoring
contact: data@ekho.club
```

Не маскируемся под Chrome-пользователя специально для обхода ограничений.

19. **Crawler должен быть очень вежливым.**
    

По домену:

```text
concurrency: 1–2

rate limit

exponential backoff

Retry-After support

ETag

If-Modified-Since

timeouts
```

Если страница не менялась:

```text
304 Not Modified
→ ничего не скачиваем заново
```

Это одновременно дешевле и безопаснее.

20. **Source scheduling не одинаковый.**
    

```text
institution identity
→ monthly

program data
→ weekly/monthly

tuition
→ weekly during update season

admissions requirements
→ weekly

deadline near application period
→ daily

known volatile page
→ daily
```

Не надо:

```text
50,000 pages × every hour
```

21. **Pipeline целиком:**
    

```text
SOURCE REGISTRY
      ↓
LEGAL GATE
      ↓
FETCH QUEUE
      ↓
FETCH
      ↓
TEMPORARY RAW CONTENT
      ↓
EXTRACTION
      ↓
NORMALIZATION
      ↓
VALIDATION
      ↓
STAGING
      ↓
CONFLICT DETECTION
      ↓
AUTO / HUMAN REVIEW
      ↓
CANONICAL DATABASE
      ↓
EVIDENCE
      ↓
CHANGE DETECTION
      ↓
PUBLISH
```

22. **Raw content сначала temporary.**
    

```text
fetch
↓
parse
↓
extract facts
↓
delete raw response
```

если source policy не позволяет постоянное хранение.

Это уменьшает IP/storage risk.

23. **Extraction Layer не пишет прямо в production.**
    

Никогда:

```text
LLM/parser
→ production deadline
```

Только:

```text
parser
↓
staging_fact
↓
validation
↓
canonical_fact
```

24. **Staging fact.**
    

Пример:

```text
source:
Stanford official admissions

raw:
"Regular Decision: January 5"

extracted:
deadline = 2027-01-05

confidence:
parser confidence

status:
pending_validation
```

`parser confidence` — только внутреннее качество extraction.

Не:

> Stanford deadline is 94% likely January 5.

25. **Normalization Layer.**
    

Например:

```text
"5 January 2027"
"January 5"
"Jan. 5"

↓

2027-01-05
```

Но если year невозможно доказать:

```text
year = unknown
```

Не угадываем.

26. **Validation Layer.**
    

Codex должен валидировать:

```text
known country
valid currency

valid date

score inside test range

institution exists

program belongs to institution

intake exists

fact has source

source passed legal gate
```

Если нет:

```text
quarantine
```

а не production.

27. **Conflict Engine.**
    

Например:

```text
University:
deadline Jan 3

Government dataset:
deadline Jan 5
```

Pipeline:

```text
CONFLICT

do not overwrite

→ review_queue
```

28. **Source priority не должен слепо решать конфликт.**
    

Например government database может быть старее самой university admissions page.

Смотрим:

```text
authority
freshness
scope
cycle
specificity
```

29. **Human review нужен только для спорных случаев.**
    

Автоматически можно пропускать:

```text
GREEN source
+
schema valid
+
no conflict
+
normal expected change
```

Review:

```text
new requirement type

deadline suddenly changed

source conflict

large tuition change

policy changed

parser uncertainty
```

Так дешёво масштабируется.

30. **Change detection работает по фактам, не по HTML.**
    

Не:

```text
page changed
→ ALERT USER
```

Потому что университет поменял footer.

А:

```text
old normalized facts
vs
new normalized facts
```

Например:

```text
DET 120
↓
DET 125

→ meaningful change
```

31. **Page hash используется только как дешёвый первый фильтр.**
    

```text
same hash
→ stop

different hash
→ extract

same admissions facts
→ stop

important fact changed
→ ChangeEvent
```

32. **Персональные данные crawler вообще не собирает.**
    

Это очень важно.

Не сохраняем с university websites:

```text
employee names
personal emails
phone numbers
student profiles
student photos
forum posts
social accounts
applicant data
```

Если они случайно попали:

```text
PII filter
→ strip before staging
```

GDPR требует purpose limitation и устанавливает отдельные обязанности, когда personal data получены не непосредственно от человека; поэтому проще вообще не допускать такие данные в admissions pipeline. ([Eur-Lex](https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX%3A02016R0679-20160504&utm_source=chatgpt.com "Consolidated TEXT: 32016R0679 — EN — 04.05.2016"))

33. **Generic institutional contact можно хранить отдельно.**
    

Например:

```text
admissions@university.edu
```

если нужен пользователю.

Но нам даже это не нужно для P0.

34. **Reddit/X/forums не входят в factual pipeline.**
    

Они могут использоваться позже для:

```text
product research
student pain research
qualitative context
```

Но никогда Reddit comment:

> Stanford doesn't require SAT anymore

не становится canonical requirement.

35. **AI никогда не является source.**
    

```text
SOURCE:
ChatGPT
Claude
Gemini
```

→ запрещено.

AI только:

```text
extract
normalize
classify
compare
summarize
```

Source всегда остаётся реальным документом/API.

36. **Images — отдельный pipeline.**
    

Не скачиваем автоматически фотографии университетов с их сайтов.

Используем только:

```text
own images
explicitly licensed assets
CC0
CC BY
university media kit permitting intended reuse
other explicit licence
```

Creative Commons, например, разрешает commercial reuse по CC BY при соблюдении attribution; CC BY-NC уже нельзя использовать как обычный коммерческий asset Ekho. ([Creative Commons](https://creativecommons.org/cc-licenses/?utm_source=chatgpt.com "Sharing Openly, Sharing Globally"))

37. **Для каждого media asset:**
    

```text
asset_id

source_url
creator

license
license_url

attribution_text

commercial_use_allowed
modification_allowed

retrieved_at
```

Нет license metadata → не публикуем.

38. **University logos тоже не считать “free image”.**
    

Logo может иметь copyright/trademark protection; Copyright Office отдельно отмечает, что художественное logo может иметь copyright, а названия/знаки могут охраняться trademark law. ([Офис авторских прав США](https://www.copyright.gov/help/faq/faq-protect.html?utm_source=chatgpt.com "What Does Copyright Protect? (FAQ) | U.S. Copyright Office"))

Поэтому P0:

```text
university name
+
neutral Ekho visual
```

Если потом нужны logos → отдельная brand/permission policy.

39. **Open data тоже сохраняет лицензию.**
    

Даже если:

```text
HESA CC BY 4.0
```

храним:

```text
source_dataset
version
licence
attribution
retrieved_at
```

CC BY требует attribution. ([Creative Commons](https://creativecommons.org/licenses/by/4.0/deed.en?utm_source=chatgpt.com "Deed - Attribution 4.0 International"))

40. **License changes должны мониториться.**
    

Например сегодня:

```text
API = commercial use allowed
```

завтра terms изменились.

Поэтому:

```text
last_policy_review_at
policy_hash
```

Если policy hash изменился:

```text
disable ingestion
→ review
```

41. **Таблицы, которые я добавил бы к нашей Data Architecture:**
    

```text
source_policies

fetch_jobs
fetch_runs

extraction_runs
staging_facts

review_cases
```

А уже существующие:

```text
sources
fact_evidence
change_events
source_snapshots
```

оставляем.

Но `source_snapshots` теперь **conditional**, а не обязательный полный archive.

42. **`source_policies` — критическая таблица.**
    

```text
source_id

risk_level

access_type

license_type
license_url

terms_url
robots_url

commercial_use_allowed
automated_access_allowed
redistribution_allowed

full_snapshot_allowed
media_reuse_allowed

attribution_required

personal_data_prohibited = true

reviewed_at
review_status
```

43. **`fetch_runs`:**
    

```text
id
source_id

started_at
finished_at

http_status

etag
last_modified

content_hash

bytes_downloaded

result
error_code
```

Это ещё позволит контролировать стоимость crawler.

44. **`extraction_runs`:**
    

```text
fetch_run_id

parser_name
parser_version

model_name
model_version

started_at

facts_found
facts_rejected
```

Это потом крайне важно:

> почему Ekho ошибся?

Можно определить конкретную parser/model version.

45. **Нельзя позволять Codex самостоятельно добавлять sources.**
    

Pipeline API:

```text
register source
↓
policy status = unreviewed

crawler:
DENIED
```

Пока:

```text
policy status = approved
```

46. **Удаление источника.**
    

Если university пишет:

> stop automated crawling

или появляется запрет:

```text
disable source
cancel queued jobs
stop future fetches
review retained material
```

Но canonical historical facts можно оценивать отдельно в зависимости от лицензии/закона; автоматически считать их навечно разрешёнными нельзя.

47. **Главный cheap architecture.**
    

На нашем стеке:

```text
Cloudflare Cron
↓
Cloudflare Queue
↓
Fetcher Worker
↓
temporary content
↓
Extractor
↓
Normalizer
↓
Supabase staging
↓
Validator
↓
Postgres canonical DB
```

И R2 только для материалов, **которые действительно разрешено хранить**.

48. **Что точно запрещаем Codex реализовывать без отдельного решения:**
    

```text
generic "scrape any URL" endpoint

headless browser farm

proxy rotation

CAPTCHA bypass

competitor scraping

social-media scraping

full-web archive

automatic image downloader

paywall scraping

authenticated scraping

personal-data scraping
```

49. **Финальная source strategy Ekho:**
    

```text
GLOBAL IDENTITY
→ ROR / other open registries

US
→ official government datasets
→ university official pages

UK
→ HESA / Discover Uni open data
→ university official pages
→ NOT standard UCAS Courses Data licence

OTHER COUNTRIES
→ government/regulator open sources where available
→ university official sources

EVERYWHERE
→ official source remains evidence
```

50. **Главная архитектурная мысль:**
    

```text
Internet
   ↓
Source Registry
   ↓
RIGHTS CHECK
   ↓
Official factual content only
   ↓
Temporary ingestion
   ↓
Structured facts
   ↓
Validation
   ↓
Evidence
   ↓
Ekho
```

Не:

```text
Internet
↓
scrape everything
↓
store everything
↓
pray nobody complains
```

### Итог: **Strong**

Я бы именно эту версию **Data Pipeline v1 сохранял как спецификацию для Codex**.

Самые важные решения здесь: **whitelisted sources only, legal gate before crawling, no competitor/paywall/auth scraping, UCAS standard licence пока исключаем, raw HTML хранится только когда разрешено, изображения только с лицензией, personal data crawler не собирает вообще, AI не является источником, а каждый опубликованный факт ведёт обратно к официальному source.**