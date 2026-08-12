Notion начинал с одного PostgreSQL и перешёл к шардингу только после серьёзного роста; GitHub до сих пор показывает, что огромный продукт вполне может жить как крупный монолит, а масштабирование БД решается отдельно. Для Ekho отсюда вывод: **один Postgres + хорошие связи + индексы + cache**, без микросервисов и распределённых БД. ([Notion][1])

# EKHO — DATA ARCHITECTURE V1

## 1. Главная иерархия

```text
Institution
    ↓
Program
    ↓
Program Offering
    ↓
Intake
    ↓
Application Round
    ↓
Requirement Set
    ↓
Requirement
```

Пример:

```text
Stanford University
 ↓
Computer Science
 ↓
BS Computer Science / Stanford campus / English / Full-time
 ↓
Fall 2027
 ↓
Regular Decision
 ↓
International applicant requirements
 ↓
SAT
English test
Transcript
Essays
Recommendations
...
```

**Это будет позвоночник всей базы.**

---

# 2. Institutions

### `institutions`

Одна запись = один университет/учебное учреждение.

```text
id
canonical_name
slug

institution_type
country_code
city
region

website_url

founded_year
public_private

latitude
longitude

status
created_at
updated_at
```

Не пихаем сюда SAT, tuition, deadline и прочее.

Это свойства **admission/program**, а не самого университета.

---

# 3. Institution identity

Нам обязательно понадобится нормальная идентификация вузов.

### `institution_aliases`

```text
id
institution_id

name
language
alias_type
```

Например:

```text
MIT
Massachusetts Institute of Technology
Массачусетский технологический институт
```

---

### `institution_external_ids`

```text
institution_id

provider
external_id
```

Например:

```text
ROR
UCAS
UKPRN
Wikidata
government_id
```

ROR именно так решает проблему идентичности организаций: один persistent ID + внешние identifier mappings. Ekho должен использовать собственный ID как основной, а внешние ID — как связи с другими datasets. ([ROR][2])

**Strong.**

---

# 4. Campuses

### `campuses`

```text
id
institution_id

name
country
city
address

latitude
longitude
```

Потому что один университет может иметь:

* несколько кампусов;
* разные программы;
* разные tuition;
* разные locations.

---

# 5. Academic taxonomy

### `academic_fields`

```text
id
parent_id

code
name
level
```

Не придумываем свою классификацию с нуля.

Берём за основу **UNESCO ISCED-F**, который специально создан для международного сравнения образовательных программ и областей обучения. ([UIS][3])

Например:

```text
ICT
  └ Computer Science
     └ Artificial Intelligence
```

А сверху можем добавить свои user-friendly категории Ekho.

---

# 6. Programs

### `programs`

Это **академическая сущность**, не admission.

```text
id
institution_id

name
slug

degree_level
academic_field_id

description

active
created_at
updated_at
```

Пример:

```text
Computer Science
Bachelor
Stanford
```

---

# 7. Program Offering

Это очень важная таблица.

### `program_offerings`

Одна программа может существовать:

* в разных кампусах;
* онлайн/offline;
* на разных языках;
* full-time/part-time.

```text
id
program_id
campus_id

study_mode
attendance_mode
language

duration_months
credits
credits_system
```

То есть:

```text
Program
Computer Science

Offering #1
Stanford / English / Full-time

Offering #2
Online / English / Part-time
```

---

# 8. Intake

### `intakes`

```text
id
program_offering_id

academic_year
term

start_date
end_date

status
```

Например:

```text
Fall 2027
Spring 2028
```

**Deadline не хранить прямо в Program.**

Потому что он меняется каждый admission cycle.

---

# 9. Application rounds

### `application_rounds`

```text
id
intake_id

round_type

opens_at
deadline_at
decision_at

application_platform_id

application_fee_amount
application_fee_currency
```

Например:

```text
Early Action
Early Decision
Regular Decision
Rolling
UCAS
Direct
```

---

# 10. Application platforms

### `application_platforms`

```text
id
name
website

platform_type
```

Например:

```text
Common App
UCAS
University portal
Coalition
Studielink
uni-assist
```

---

# 11. Requirements — критически важная часть

Не делать:

```text
sat_required
ielts_required
gpa_required
essay_required
...
```

Это тупик.

В разных странах будут сотни типов требований.

Делаем:

### `requirement_sets`

```text
id

institution_id
program_id
intake_id
application_round_id

name
```

Nullable поля позволяют задавать уровень.

Например:

```text
Institution-wide requirements

Program requirements

2027 requirements

Regular Decision requirements
```

---

# 12. Requirement

### `requirements`

Одна строка = **одно атомарное требование**.

Это принцип, который мне нравится и из подхода Notion: хранить информацию достаточно гранулярно, чтобы отдельные элементы можно было изменять независимо, а не запирать всё в огромном объекте. ([Notion][4])

```text
id
requirement_set_id

type

title
description

requirement_status

sort_order

created_at
updated_at
```

`type`:

```text
academic
qualification
gpa

SAT
ACT

IELTS
TOEFL
DET

transcript
essay
recommendation
portfolio
interview

application_fee
document

other
```

`requirement_status`:

```text
required
optional
recommended
not_required
conditional
```

---

# 13. Structured requirement details

Не надо хранить всё текстом.

Например SAT:

### `test_requirements`

```text
requirement_id
test_id

minimum_score
recommended_score

superscore_allowed
self_report_allowed
```

---

### `tests`

```text
id
code
name

provider
score_min
score_max
```

Например:

```text
SAT
ACT
DET
IELTS
TOEFL
```

---

# 14. Applicability rules

Вот здесь Ekho сможет стать реально сильным.

Requirement может относиться только к:

```text
international students

Indian CBSE students

EU students

non-native English speakers

applicants from Russia

IB students

Computer Science applicants
```

### `requirement_rules`

```text
id
requirement_id

rule_json
```

Например концептуально:

```json
{
  "all": [
    {
      "field": "international_student",
      "equals": true
    },
    {
      "field": "english_education_years",
      "less_than": 4
    }
  ]
}
```

Вот здесь **JSONB оправдан**, потому что мировые admission rules слишком разнообразны для сотен nullable колонок. PostgreSQL поддерживает JSON/JSONB наряду с нормальными relational constraints и indexes. ([PostgreSQL][5])

Но важное правило:

> **Core searchable data → normal SQL columns.
> Weird/variable admission logic → JSONB.**

---

# 15. Qualifications

### `qualifications`

```text
id

country_code
education_system

name
short_name

level
```

Например:

```text
IB Diploma
A Levels
Abitur
French Baccalauréat
Russian Attestat
Indian CBSE
Italian Diploma
```

---

# 16. Sources

Вот здесь будет одно из главных преимуществ Ekho.

### `sources`

```text
id

institution_id

url
source_type

title

official
authority_level

active

check_frequency
last_checked_at
```

`source_type`:

```text
admission_page
program_page
financial_aid_page
PDF
government
application_platform
```

---

# 17. Source snapshots

### `source_snapshots`

Не перезаписываем прошлое.

```text
id
source_id

fetched_at

content_hash
storage_key

http_status
etag
last_modified
```

Сам HTML/PDF:

**R2**

не Postgres.

---

# 18. Evidence

Вот это я считаю **обязательным для Ekho**.

### `fact_evidence`

```text
id

entity_type
entity_id

field_path

source_id
snapshot_id

raw_value
normalized_value

observed_at
verified_at

verification_status
```

Например:

```text
entity:
Stanford RD 2027

field:
deadline

normalized:
2027-01-05

source:
admission.stanford.edu/...
```

То есть Ekho всегда может ответить:

> Почему вы показываете 5 января?

И показать источник.

---

# 19. Почему не превращать `fact_evidence` в основную БД

Не делать EAV:

```text
entity
property
value
```

для **всего** Ekho.

Тогда запрос:

> universities with tuition < $30k and DET ≥120

станет кошмаром.

Поэтому:

### Canonical data

```text
normal PostgreSQL tables
```

### Evidence/history

```text
fact_evidence
```

Два слоя.

---

# 20. Change detection

### `change_events`

```text
id

entity_type
entity_id
field_path

old_value
new_value

source_id
snapshot_id

detected_at

affected_scope

review_status
```

Например:

```text
Stanford

SAT policy

test optional
↓
test required

Detected:
2026-08-14
```

Это напрямую кормит:

**Live Admissions Updates.**

---

# 21. User

Supabase Auth уже хранит login/auth identity.

Ekho создаёт:

### `profiles`

```text
user_id

display_name
locale
timezone

created_at
updated_at
```

`user_id` → `auth.users`.

Supabase официально рекомендует свою public profile table связывать с `auth.users` через foreign key и защищать пользовательские данные RLS. ([Supabase][6])

---

# 22. Applicant profile

Не делаем одну таблицу из 80 полей.

### `applicant_profiles`

```text
user_id

residence_country
graduation_year

intended_start_year
```

---

### `user_nationalities`

```text
user_id
country_code
```

Потому что гражданств может быть несколько.

---

# 23. User education

### `user_qualifications`

```text
id
user_id

qualification_id
institution_name

start_year
graduation_year

gpa_value
gpa_scale
```

Не конвертируем оригинальный GPA безвозвратно.

Храним:

```text
3.87
4.0
```

отдельно.

---

# 24. User tests

### `user_test_scores`

```text
id
user_id
test_id

total_score
test_date

section_scores JSONB
```

Например:

```text
DET 135

Literacy 130
Production 140
...
```

---

# 25. User documents

### `user_documents`

```text
id
user_id

document_type
filename

storage_key

uploaded_at
```

Сам файл → private R2.

Не в Postgres.

---

# 26. Saved programs

### `saved_programs`

```text
user_id
program_id

created_at
```

Простой many-to-many.

---

# 27. Applications

### `applications`

Это центр личного workspace.

```text
id
user_id

program_id
intake_id
application_round_id

status
priority

created_at
submitted_at
```

Statuses:

```text
researching
planning
in_progress
ready
submitted
decision_received
accepted
waitlisted
rejected
withdrawn
```

---

# 28. Application requirements

Ekho вычисляет персональный результат.

### `application_requirements`

```text
application_id
requirement_id

status

reason

evaluated_at
```

Status:

```text
satisfied
missing
optional
action_required
unknown
not_applicable
```

Очень важно:

**это derived data.**

Source of truth:

```text
Requirement
+
Applicant Profile
```

Если человек обновил DET:

```text
105 → 135
```

мы пересчитываем requirements.

---

# 29. Tasks

### `application_tasks`

```text
id
application_id

requirement_id

title
description

due_at
status

sort_order
```

Requirement:

> Upload transcript

превращается в Task:

> Upload your transcript before Jan 5

---

# 30. Application documents

### `application_documents`

```text
application_id
document_id
requirement_id
```

Один transcript можно использовать для нескольких applications.

Мы **не копируем файл 10 раз**.

---

# 31. Notes

### `application_notes`

```text
id
application_id
user_id

content

created_at
updated_at
```

---

# 32. Timeline

### `application_events`

```text
id
application_id

event_type
data

created_at
```

Например:

```text
University added
SAT requirement satisfied
Essay uploaded
Application submitted
Decision received
```

---

# 33. Relations обязательно через foreign keys

Например:

```text
program.institution_id
→ institutions.id

application.user_id
→ users.id

requirement.requirement_set_id
→ requirement_sets.id
```

Postgres foreign keys обеспечивают referential integrity — например нельзя получить программу, которая ссылается на несуществующий университет. ([PostgreSQL][7])

---

# 34. Public vs private data

Разделяем буквально концептуально.

### PUBLIC

```text
institutions
programs
program_offerings
intakes
rounds
requirements
sources
```

### PRIVATE

```text
profiles
qualifications
scores
applications
documents
tasks
notes
```

Private tables:

**RLS mandatory.**

Supabase RLS позволяет ограничивать строки на уровне самой БД, а не надеяться только на frontend/backend checks. ([Supabase][8])

---

# 35. Нормализация

Не хранить:

```text
Stanford
Stanford University
Stanford University USA
Stanford, CA
```

в 15 таблицах.

Храним:

```text
institution_id = 154
```

и одну canonical запись.

---

# 36. Что можно денормализовать

Только там, где это реально ускорит продукт.

Например позже сделать:

```text
university_search_view
```

где уже собраны:

```text
university
country
program_count
min_tuition
deadline
DET
```

PostgreSQL поддерживает materialized views, которые физически сохраняют результат сложного запроса и могут обновляться отдельно. ([PostgreSQL][9])

**Но не нужно в первый день.**

---

# 37. Финальные таблицы V1

Я бы реально создавал примерно такой костяк:

```text
CORE

institutions
institution_aliases
institution_external_ids
campuses

academic_fields

programs
program_offerings
intakes

application_platforms
application_rounds


ADMISSIONS

requirement_sets
requirements
requirement_rules

tests
test_requirements

qualifications


TRUST / DATA

sources
source_snapshots
fact_evidence
change_events


USER

profiles
applicant_profiles
user_nationalities
user_qualifications
user_test_scores

user_documents
saved_programs


APPLICATIONS

applications
application_requirements
application_tasks
application_documents
application_notes
application_events
```

**≈30 таблиц.**

Это нормально. Не означает 30 микросервисов — это просто структурированные сущности внутри одного Postgres.

---

# 38. Чего сейчас НЕ делать

### Weak

```text
MongoDB
Graph database
Neo4j

Elasticsearch database
Vector database

Kafka
Microservices

Data warehouse
Snowflake

database sharding

event sourcing всего продукта

generic EAV database
```

До появления реальной необходимости это только увеличит стоимость и вероятность багов.

---

# 39. Что особенно важно заложить сейчас

### Must-have

1. **Canonical IDs**
2. **External IDs**
3. **Institution → Program → Offering → Intake → Round**
4. **Atomic requirements**
5. **Machine-readable requirement rules**
6. **Applicant profile отдельно от application**
7. **Sources**
8. **Immutable source snapshots**
9. **Fact evidence**
10. **Change history**
11. **Original values не уничтожаем**
12. **Unknown ≠ false**
13. **RLS для user data**
14. **Foreign keys**
15. **Global taxonomy через ISCED**

---

# 40. Главная идея архитектуры

```text
                    OFFICIAL SOURCE
                          ↓
                     Snapshot
                          ↓
                       Evidence
                          ↓
                    Canonical Data
                          ↓
          ┌───────────────┴───────────────┐
          ↓                               ↓
 University Intelligence           Applicant Profile
          ↓                               ↓
          └───────────────┬───────────────┘
                          ↓
                 Personalization Engine
                          ↓
                    My Application
                          ↓
        Missing / Satisfied / Action needed
```

Вот **это уже настоящий data moat Ekho**.

Не база «Stanford — deadline Jan 5», а система:

> **Entity → Fact → Source → Evidence → History → Applicant → Personalized action**

И самое главное: эту архитектуру можно начать буквально на **одном Supabase Postgres**, а усложнять инфраструктуру только после появления реальной нагрузки. Это соответствует тому, как крупные продукты вроде Notion эволюционировали БД уже после роста, а не до него.
