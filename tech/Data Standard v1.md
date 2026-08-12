Основа правильная: UCAS отдельно хранит course-level fees, deadlines, entry requirements, tests, interviews, portfolio, duration/start date; Common App — testing, writing, deadlines и individual college requirements; College Scorecard/IPEDS — institutional characteristics, costs, admissions и outcomes. То есть копировать одну систему нельзя — Ekho нужен глобальный superset. ([UCAS](https://www.ucas.com/business/brands/courses-data-service "Courses Data Service | UCAS"))

# EKHO DATA STANDARD v1

## 1. Главный принцип

Каждый важный факт в Ekho должен отвечать на **6 вопросов**:

```text
WHAT       что известно
ABOUT      к какой сущности относится
WHO        для какого applicant
WHEN       для какого admission cycle/intake
SOURCE     откуда взято
FRESHNESS  когда последний раз проверено
```

Не просто:

```text
SAT required
```

а:

```text
Stanford
Undergraduate
Fall 2027
International applicant
SAT: required
Source: official admissions page
Verified: 2026-08-12
```

Это критично: UCAS прямо предупреждает, что исторические admission данные нельзя выдавать за текущие требования. ([UCAS](https://www.ucas.com/applying/before-you-apply/what-and-where-to-study/entry-requirements/understanding-historical-entry-grades-data "Understanding historical entry grades data | UCAS"))

---

# 2. Universal metadata

Практически каждый **изменяемый admissions fact** получает:

```text
value
value_status

valid_from
valid_until

academic_year
intake_id
application_round_id

source_id
snapshot_id

observed_at
verified_at

verification_status

raw_value
normalized_value
```

Не обязательно физически повторять эти колонки в каждой таблице — часть может жить через evidence layer.

---

# 3. Никакого обычного `null = неизвестно`

Это одна из самых важных вещей.

Ekho должен различать:

```text
known
unknown
not_published
not_found
not_applicable
not_offered
conflicting_sources
stale
```

Например:

```text
IELTS minimum = null
```

ничего не объясняет.

А:

```text
IELTS
status = not_published
```

объясняет.

### Запрещаем

```text
unknown = false
unknown = 0
unknown = optional
```

---

# 4. Verification status

Не используем выдуманный AI confidence типа:

> 94% true.

Делаем понятные состояния:

```text
verified_primary
verified_official_secondary
verified_external
unverified
conflict
stale
```

W3C PROV именно разделяет сам факт и provenance — сущности, процессы и источники, которые привели к появлению информации. Мы берём этот принцип, но реализуем проще в Postgres. ([W3C](https://www.w3.org/TR/prov-o/ "PROV-O: The PROV Ontology"))

---

# 5. Source authority

Не делаем тупой рейтинг «government всегда лучше university».

Источник зависит от факта.

### Admission deadline

```text
university admissions office
application platform
```

### Accreditation

```text
government / accreditation body
```

### Visa

```text
government immigration authority
```

### Tuition

```text
university official fee schedule
```

Типы:

```text
primary_owner
government
official_application_platform
official_registry
trusted_dataset
secondary
```

---

# 6. Institution identity

### MUST

```text
ekho_id
canonical_name
display_name
original_name

aliases[]
acronyms[]

institution_type

status

official_domain
official_website

country_code
subdivision_code
city

latitude
longitude

established_year

external_ids[]
```

ROR уже использует stable ID, names/aliases, domains, external IDs, locations, relationships и status; это хороший проверенный шаблон для identity layer. ([ROR](https://ror.readme.io/docs/ror-data-structure "ROR Data Structure"))

---

# 7. University relationships

Нельзя предполагать:

> university = одна организация.

Поддерживаем:

```text
parent
child
related
predecessor
successor
```

Например mergers/rebrands.

ROR использует такую модель именно для организационных и исторических связей. ([ROR](https://ror.readme.io/docs/relationships "ROR relationships and hierarchies"))

---

# 8. География

Используем стандарты:

```text
country       → ISO 3166
subdivision   → ISO 3166-2
currency      → ISO 4217
date/time     → ISO 8601
timezone      → IANA tz database
```

Это международные стандарты, а IANA timezone database обновляется при изменении реальных timezone/DST правил. ([ISO](https://www.iso.org/popular-standards.html "ISO - Popular standards"))

---

# 9. Program identity

### MUST

```text
program_id

official_name
display_name
slug

institution_id

degree_name
degree_level

academic_field

description

official_program_url

active_status
```

---

# 10. Degree level

Не делаем только:

```text
Bachelor
Master
PhD
```

Храним одновременно:

```text
native_degree_name
ekho_degree_category
isced_level
```

ISCED:

```text
5 short-cycle tertiary
6 bachelor/equivalent
7 master/equivalent
8 doctoral/equivalent
```

UNESCO использует ISCED именно для международной сопоставимости образовательных программ. ([UIS Data Browser](https://databrowser.uis.unesco.org/glossary?utm_source=chatgpt.com "Glossary"))

---

# 11. Academic field

Храним:

```text
official_field_name

isced_f_code
isced_f_name

ekho_category
```

Например:

```text
official:
Computing Science

ISCED-F:
0613 Software and applications development and analysis

Ekho:
Computer Science
```

Так UX остаётся простым, а данные — международно сопоставимыми. ISCED-F предназначен именно для классификации программ и qualifications по fields of study. ([UIS](https://www.uis.unesco.org/en/methods-and-tools/isced?utm_source=chatgpt.com "International Standard Classification of Education - ISCED"))

---

# 12. Program ≠ Program Offering

Это мы оставляем.

```text
PROGRAM
Computer Science BSc

OFFERING

campus
study_language
study_mode
attendance
duration
credits
```

Schema.org также разделяет `Course` и `CourseInstance`: один course может иметь разные время, место или mode delivery. ([Schema.org](https://schema.org/CourseInstance "CourseInstance - Schema.org Type"))

---

# 13. Offering fields

### MUST

```text
campus_id

delivery_mode:
onsite
online
hybrid

attendance_mode:
full_time
part_time
flexible

languages[]

duration_value
duration_unit

credits
credit_system

study_location

professional_accreditation

active_status
```

Не смешиваем:

`online` и `part-time`.

Это разные dimensions.

---

# 14. Intake

```text
academic_year

term:
fall
spring
summer
winter
other

official_term_name

start_date
end_date

applications_open

status:
planned
open
closed
cancelled
unknown
```

Обязательно `official_term_name`, потому что далеко не весь мир использует Fall/Spring.

---

# 15. Application round

```text
round_type

official_round_name

opens_at

deadline_at
deadline_timezone

decision_date
decision_date_type

reply_deadline

rolling_admission

application_platform

application_url
```

---

# 16. Deadline standard

Deadline — это не просто date.

```text
date
time
timezone
deadline_type

hard_or_priority
applicant_scope
```

Например:

```text
2027-01-05
23:59
America/Los_Angeles
regular_application
hard
international + domestic
```

Если университет написал только:

> January 5

Ekho **не выдумывает 23:59**.

```text
time_status = not_published
```

---

# 17. Requirement base standard

Каждое requirement:

```text
type

title
description

obligation

applicability

structured_rule

raw_text

source
```

### `obligation`

```text
required
optional
recommended
conditional
not_required
```

Common App показывает реальный смысл такой модели: personal essay может быть required или optional, а отдельные writing supplements зависят от конкретного колледжа и иногда от ответов applicant. ([Common App](https://www.commonapp.org/apply/first-year-students/ "Application guide for first-year students"))

---

# 18. Requirement categories

Нам нужно покрыть как минимум:

```text
qualification
academic_grade
subject

standardized_test
english_language_test
admission_test

transcript
predicted_grades

essay
personal_statement
short_answer

recommendation
school_report

resume
activities

portfolio
audition

interview

work_experience

application_form
application_fee

passport
visa_document

financial_document

health_requirement
background_check

other_document
other
```

UCAS отдельно подтверждает qualification/grades, admissions tests, interviews, portfolio и дополнительные health/background requirements. ([UCAS](https://www.ucas.com/applying/you-apply/what-and-where-study/entry-requirements "University Entry Requirements | UCAS"))

---

# 19. Qualification requirements

Это один из самых сложных блоков.

Нам нужно хранить:

```text
qualification_id

country
education_system

minimum_grade
recommended_grade

grade_scale

required_subjects[]

subject_minimums[]

equivalency

conditions
```

---

# 20. Requirement logic

Нельзя делать только список требований.

Нужны:

```text
ALL
ANY
NOT
```

Пример:

```text
ALL
 ├ GPA >= 3.0
 └ ANY
    ├ SAT >= 1250
    ├ ACT >= 26
    └ 3 AP exams >= 3
```

Потому что реальные entry requirements часто состоят из combinations qualifications/subjects/grades. UCAS прямо описывает именно такие комбинации. ([UCAS](https://www.ucas.com/applying/you-apply/what-and-where-study/entry-requirements "University Entry Requirements | UCAS"))

---

# 21. Applicant scope

Requirement обязательно может иметь ограничения:

```text
citizenship
residence
international_status

education_country
education_system
qualification

program
degree_level

applicant_type

age

first_language
language_of_instruction

previous_study
```

---

# 22. Applicant types

Минимум:

```text
first_year
transfer
graduate
returning
mature
exchange
visiting
other
```

Нельзя применять first-year requirement к transfer applicant.

---

# 23. Standardized tests

### Test definition

```text
test_id
name
provider

score_min
score_max

sections[]
```

### Requirement

```text
policy

minimum_total
recommended_total

section_minimums

superscore_policy

self_report_allowed
official_report_required

validity_period

deadline
```

---

# 24. Test policy

Наш canonical enum:

```text
required
optional
flexible
conditional
not_considered
not_required
unknown
```

Это хорошо соответствует реальной сложности policies: Common App отдельно показывает test policies, а colleges устанавливают их самостоятельно. ([Common App](https://www.commonapp.org/apply/first-year-students/?utm_source=chatgpt.com "Application guide for first-year students"))

---

# 25. English proficiency

Отдельно от SAT/ACT.

```text
test

accepted

minimum_overall
recommended_overall
minimum_sections

waiver_available

waiver_rules

validity_period

self_report_allowed

official_report_required
```

Особенно важен:

### `waiver_rules`

Например:

```text
4 years education in English
citizen of specified countries
qualification taught in English
```

---

# 26. Essays

```text
essay_type

prompt

required_status

min_words
max_words
max_characters

response_format

shared_or_program_specific

deadline
```

Prompt обязательно versioned по application cycle.

Не перезаписываем прошлогодний.

---

# 27. Recommendations

```text
recommender_type

minimum_count
maximum_count

required_count

teacher_subject_restriction

submission_method

deadline
```

Recommender types:

```text
teacher
counselor
academic
employer
coach
peer
other
```

Common App действительно различает teacher и other recommenders и позволяет каждому college задавать собственные requirements. ([Common App](https://www.commonapp.org/apply/first-year-students/ "Application guide for first-year students"))

---

# 28. Portfolio / audition / interview

```text
required_status

format

platform

deadline

invite_only

location

online_available

instructions
```

---

# 29. Application fee

```text
amount
currency

fee_status

fee_waiver_available

waiver_rules

payment_platform
```

### `fee_status`

```text
required
no_fee
conditional
unknown
```

Не:

```text
fee = 0
```

если мы просто не нашли fee.

---

# 30. Tuition

Очень важный стандарт.

```text
amount

currency

billing_basis:
year
semester
term
credit
program

academic_year

student_scope:
domestic
EU
international
residency_based
other

degree_level
program_id

mandatory_fees

source
```

Не существует одного `university.tuition`.

---

# 31. Cost of attendance

Отдельно:

```text
tuition
mandatory_fees

housing
food

books_supplies

insurance

transportation

personal_expenses

visa_costs

other

total_estimated_cost
```

IPEDS тоже разделяет tuition/fees и estimated student budgets с разными living situations; значит Ekho не должен показывать tuition как «стоимость обучения вообще». ([Центр статистики образования](https://nces.ed.gov/ipeds/about-ipeds?utm_source=chatgpt.com "About IPEDS - National Center for Education Statistics (NCES)"))

---

# 32. Currency

Храним **оригинальную валюту**.

```text
amount = 45000
currency = GBP
```

Если UI хочет показать:

```text
≈ $58,400
```

это derived value.

Никогда не заменяем оригинал конвертацией.

ISO 4217 используем для currency code. ([ISO](https://www.iso.org/iso-4217-currency-codes.html?utm_source=chatgpt.com "ISO 4217 — Currency codes"))

---

# 33. Scholarships

Отдельная сущность.

```text
name

provider

type

eligibility

international_eligible

program_scope

amount_type
amount_min
amount_max
currency

coverage

renewable

renewal_conditions

automatic_consideration

separate_application

deadline

application_url

source
```

---

# 34. Scholarship amount type

```text
fixed
range
percentage
full_tuition
full_cost
variable
unknown
```

Не превращаем:

> up to £10,000

в:

```text
amount = £10,000
```

---

# 35. Financial aid policy

```text
aid_available

international_eligible

need_based_available
merit_based_available

need_blind_status
need_aware_status

meets_full_need_status

required_forms[]

aid_deadline

separate_application

source
```

И обязательно:

```text
unknown
```

если политика не доказана.

---

# 36. Application platform

```text
platform_name

application_url

platform_program_code

external_program_id

application_method
```

Например:

```text
Common App
UCAS
uni-assist
Studielink
Direct
```

---

# 37. Program codes

Храним **несколько external IDs**, не один:

```text
provider_program_code
UCAS_code
government_code
other
```

По той же логике, что ROR хранит multiple external identifiers. ([ROR](https://ror.readme.io/docs/ror-data-structure "ROR Data Structure"))

---

# 38. Accreditation

```text
accreditation_status

accrediting_body

program_or_institution

valid_from
valid_until

professional_qualification

source
```

Для regulated professions это может быть важнее рейтинга.

---

# 39. University statistics

Не смешиваем с текущими requirements.

Отдельные year-based facts:

```text
applications
offers/admitted
enrolled

acceptance_rate

student_count

international_students

graduation_rate

retention_rate
```

College Scorecard/IPEDS используют отдельные annual/statistical measures для institutional/admission/outcome data. ([Центр статистики образования](https://nces.ed.gov/ipeds/?utm_source=chatgpt.com "IPEDS - National Center for Education Statistics (NCES)"))

---

# 40. Historical admissions

Очень аккуратно.

Храним:

```text
cycle

metric

population

sample_size

aggregation_level

value

methodology

source
```

Например:

```text
2024–2026
accepted A-level grades
UK applicants ≤18
n=420
Computer Science
```

Никогда не превращаем это в:

> Your chance = 74%.

UCAS специально предупреждает, что historical grades **не предсказывают шанс поступления**. ([UCAS](https://www.ucas.com/applying/before-you-apply/what-and-where-to-study/entry-requirements/understanding-historical-entry-grades-data "Understanding historical entry grades data | UCAS"))

---

# 41. Rankings

Я бы поддержал schema, но **не P0**.

```text
ranking_provider

ranking_name

year

rank

subject

region
```

Никогда:

```text
ranking = 7
```

без provider/year.

---

# 42. Student outcomes

Тоже P1/P2:

```text
metric
cohort
graduation_year

value
unit

field_of_study

methodology
source
```

College Scorecard показывает, почему cohort/methodology обязательны: разные metrics используют разные cohorts, locations и definitions. ([College Scorecard](https://collegescorecard.ed.gov/data/glossary/ "Glossary | College Scorecard"))

---

# 43. User profile standard

Ekho хранит только то, что может повлиять на результат.

```text
citizenships[]
residence_country

education_country
education_system

qualification
graduation_year

grades

intended_degree
intended_start_year

tests[]

language_background

financial_aid_need
```

И progressively спрашивает остальное.

---

# 44. GPA

Никогда не уничтожаем исходный GPA.

```text
original_value
original_scale

original_system

converted_value
conversion_method
```

Например:

```text
4.73 / 5
```

остаётся source of truth.

---

# 45. User test score

```text
test_id

test_date

total_score

section_scores

official_status
```

---

# 46. Application

```text
program
offering
intake
round

application_platform

status

created
submitted

decision

decision_date
```

---

# 47. Application status

Canonical:

```text
researching
saved

planning
in_progress

ready
submitted

incomplete
withdrawn

waitlisted

accepted
rejected

enrolled
```

---

# 48. Personalized requirement result

Ekho output:

```text
satisfied
missing
optional
action_required
not_applicable
unknown
```

Дополнительно:

```text
reason
evaluated_at
based_on_profile_version
```

Пример:

```text
SAT
SAT 1450 >= required 1400

→ satisfied
```

или:

```text
English proficiency

University waiver rule unclear

→ unknown
```

---

# 49. Freshness

Не надо считать всю информацию одинаковой.

Например категории:

```text
identity
low-change

program
medium-change

tuition
annual

requirements
admission-cycle

deadline
admission-cycle

application status
high-change
```

UCAS сам подчёркивает необходимость актуального course-data и обновляет коммерческий course dataset каждые 24 часа. ([UCAS](https://www.ucas.com/business/brands/courses-data-service "Courses Data Service | UCAS"))

---

# 50. Temporal model

Практически всё admissions-sensitive хранится относительно:

```text
cycle / academic_year
```

Не:

```text
Stanford SAT = required
```

а:

```text
2026 cycle = X
2027 cycle = Y
```

ETER тоже использует institution + year как основу для годовых данных, а не один вечный mutable record. ([eter-project.com](https://eter-project.com/data/overview-data/data-collection-process/?utm_source=chatgpt.com "Data Collection Process - EHESO"))

---

# 51. Raw vs normalized

Всегда:

```text
RAW
"€16.500 per academic year for non-EU students"

NORMALIZED

amount = 16500
currency = EUR
period = academic_year
scope = non_EU
```

Raw сохраняется.

---

# 52. Derived data

Ekho должен маркировать:

```text
source_fact
normalized_fact
derived_fact
user_computed
```

Например:

```text
Official tuition:
€20,000

Official living estimate:
€12,000

Ekho estimated total:
€32,000
```

Последнее **derived**, а не official.

---

# 53. Source snapshot

Для каждого fetch:

```text
source_url

fetched_at

http_status

content_hash

etag
last_modified

content_type

storage_key
```

---

# 54. Evidence

```text
fact

source
snapshot

source_fragment

raw_value

normalized_value

observed_at

verified_at
```

То есть мы сможем потом сделать UI:

> **Source**

> Stanford Undergraduate Admissions  
> Verified 2 days ago

---

# 55. Conflict handling

Например:

University page:

```text
deadline Jan 3
```

Common App:

```text
deadline Jan 5
```

Ekho **не выбирает молча**.

```text
status = conflicting_sources
```

и отправляет на verification.

---

# 56. Versioning

Каждый schema должен иметь:

```text
schema_version
```

Например:

```text
ekho-data-schema: 1.0
```

ROR также versioning-ит свою production schema и публикует JSON Schema для validation. ([ROR](https://ror.readme.io/docs/schema-v2-1 "Schema 2.1"))

Нам нужно делать так же.

---

# 57. Validation

На ingestion:

```text
country must exist
currency must exist

deadline valid date

score within test scale

program must belong to institution

requirement must have source

tuition must have currency

historical metric must have year
```

Ошибка → quarantine.

Не сразу production.

---

# 58. Что хранить как enum

Только вещи, которые реально ограничены:

```text
application_status
requirement_status

degree_category

delivery_mode
attendance_mode

verification_status
source_type

currency
country

test_policy
```

---

# 59. Что НЕ превращать в enum

Не надо enum для:

```text
degree_name
qualification_name
program_name

essay_prompt

scholarship_name
institution_name
```

Мир слишком разнообразен.

---

# 60. Что использовать JSONB

Только для переменной структуры:

```text
requirement rule trees

test subscores

country-specific metadata

unusual qualification rules

raw structured extraction
```

Не запихивать в JSONB весь university.

---

# 61. Data quality dimensions

Для Ekho достаточно:

```text
validity
completeness
freshness
provenance
consistency
```

И не показывать пользователю бессмысленный:

> Quality score: 87/100.

Это внутренний operational слой.

---

# 62. Минимальный completeness standard

Нельзя считать university/program **Ekho Verified**, пока нет:

### Program

```text
identity
degree
location
duration
intake
```

### Admissions

```text
application method
deadline
academic requirements
language requirements
documents
```

### Financial

```text
tuition status
financial aid status
```

### Trust

```text
official source
verified_at
```

---

# 63. P0 — что реально собираем сначала

Чтобы не обанкротиться на сборе данных:

### Institution

```text
identity
location
website
```

### Program

```text
name
degree
field
duration
language
mode
```

### Admissions

```text
intake
round
deadline

qualification

tests
English

documents
essays
recommendations

interview
portfolio

fee
application platform
```

### Money

```text
tuition
basic aid policy
scholarships
```

### Trust

```text
source
last verified
snapshot
```

**Вот это P0.**

---

# 64. P1

После работающего P0:

```text
living costs

detailed financial aid

accreditation

historical admissions

outcomes

student statistics

housing

advanced scholarship data
```

---

# 65. P2

Не сейчас:

```text
rankings aggregation

salary predictions

acceptance probability

student reviews

social data

career outcomes normalization

full global credit equivalency
```

Особенно **acceptance probability** сейчас Weak: UCAS прямо подчёркивает, что даже historical accepted grades нельзя использовать как prediction of admission. ([UCAS](https://www.ucas.com/applying/before-you-apply/what-and-where-to-study/entry-requirements/understanding-historical-entry-grades-data "Understanding historical entry grades data | UCAS"))

---

# 66. Финальный формат одного факта Ekho

Условно:

```text
FACT
DET minimum = 120

ENTITY
University X
Computer Science BSc

SCOPE
International applicants

CYCLE
Fall 2027

STATUS
required

SOURCE
Official admissions page

RAW
"Duolingo English Test: minimum overall score 120"

VERIFIED
2026-08-12

DATA STATUS
verified_primary
```

Вот **это я считаю правильной атомарной единицей Ekho**.

---

# 67. Финальный фундамент

Получается:

```text
Institution
   ↓
Program
   ↓
Offering
   ↓
Intake
   ↓
Round
   ↓
Requirement
   ↓
Applicability Rule

        +

Applicant Profile

        ↓

Personalized Result
```

И параллельно каждому критическому факту:

```text
Fact
 ↓
Evidence
 ↓
Source
 ↓
Snapshot
 ↓
Verification
 ↓
History
```

Главное улучшение относительно предыдущей версии: мы теперь чётко разделили **current vs historical, raw vs normalized vs derived, unknown vs false, university vs program vs cycle, official fact vs Ekho calculation**. Именно это потом спасёт нас от хаоса и ложной уверенности при десятках стран.