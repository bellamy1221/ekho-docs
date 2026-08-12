По исследованиям и существующим продуктам здесь прослеживается хороший паттерн: Common App разделяет **Explore Colleges → My Colleges → requirements**, Scoir — **Discover → My Colleges → College Profile**, а внутри профиля отделяет Overview / Academics / Admissions / Cost & Aid. Это подтверждает, что исследование вузов и управление своими заявками лучше не смешивать в один огромный dashboard. ([Common App](https://www.commonapp.org/apply/first-year-students/?utm_source=chatgpt.com "Application guide for first-year students"))

# EKHO — INFORMATION ARCHITECTURE v1

## 0. Важная поправка к Data Architecture

Исправить сохранённую модель:

```text
applications

institution_id       REQUIRED

program_id           NULLABLE
program_offering_id  NULLABLE

intake_id             NULLABLE/CONDITIONAL
application_round_id NULLABLE
```

Не каждая заявка в мире является заявкой строго на конкретную программу; Common App, например, организует работу вокруг добавленных colleges/programs, причём требования зависят от конкретной организации/программы. ([commonapp.org](https://www.commonapp.org/apply/first-year-students/?utm_source=chatgpt.com "Application guide for first-year students"))

---

# 1. Главная модель Ekho

Пользователь должен мысленно понимать всего четыре пространства:

```text
HOME
Что мне делать сейчас?

EXPLORE
Куда я могу поступить?

APPLICATIONS
Что происходит с моими заявками?

UPDATES
Что важного изменилось?
```

Это и будут **4 primary navigation items**.

Не 10–15 разделов.

NNGroup рекомендует progressive disclosure: основные функции показывать сразу, редкие/сложные прятать глубже — это улучшает learnability и снижает количество ошибок. ([Nielsen Norman Group](https://www.nngroup.com/articles/progressive-disclosure/?utm_source=chatgpt.com "Progressive Disclosure"))

---

# 2. Desktop primary navigation

```text
EKHO

⌂ Home
⌕ Explore
▣ Applications
◉ Updates


─────────────

Profile
Documents
Settings
```

### Не помещать отдельными primary items:

```text
Requirements
Financial Aid
Essays
Tasks
Scores
Recommendations
Calendar
Notes
Documents
Profile
AI
```

Они живут **в контексте приложения или университета**.

Notion также рекомендует держать мало top-level destinations, потому что sidebar должен оставаться простой картой workspace. ([Notion](https://www.notion.com/help/guides/the-best-way-to-set-up-your-teams-sidebar-for-clear-organization?utm_source=chatgpt.com "The best way to set up your team's sidebar for clear ..."))

---

# 3. Global Search

Всегда доступен:

```text
Search universities, programs...
```

Shortcut позже:

```text
⌘K / Ctrl+K
```

Ищет:

```text
universities
programs
saved universities
applications
```

Не превращать command menu в отдельный продукт.

---

# 4. HOME

### URL

```text
/app
```

### Главный вопрос:

> **What should I do next?**

Не analytics dashboard.

Страница:

```text
Home

Next actions
────────────
Upload transcript
Finish MIT essay
Submit Bocconi application

Upcoming
────────
Nov 1   Stanford
Nov 15  Bocconi
Dec 1   ...

Recent important changes
────────────────────────
MIT changed English requirement

Applications
────────────
4 in progress
2 ready
1 submitted
```

---

# 5. Home priority

Порядок:

```text
1. urgent action
2. next action
3. upcoming deadline
4. important change
5. application overview
```

Не:

```text
Welcome George 👋

12 applications
87% completed
14 documents
5 universities
random chart
random motivation
```

Ekho не нужен giant SaaS dashboard.

---

# 6. EXPLORE

### Canonical URL

```text
/universities
```

**Не `/app/explore`.**

Почему: университетская база должна быть доступна без аккаунта для SEO, acquisition и immediate value.

Авторизованный пользователь видит на той же странице дополнительные состояния:

```text
Saved
Applied
Requirements for me
```

---

# 7. Explore layout

```text
Explore universities

[ Search universities or programs... ]

Filters
────────────────────────────────

Country
Degree
Field
Tuition
Language
Tests
Financial aid
Deadline
...

────────────────────────────────

Results
```

---

# 8. Filters

Не показывать 30 фильтров сразу.

Первичные:

```text
Country
Program / Field
Degree
Cost
```

`More filters`:

```text
language
tests
application deadline
financial aid
study mode
etc.
```

Это опять progressive disclosure. ([Nielsen Norman Group](https://www.nngroup.com/articles/progressive-disclosure/?utm_source=chatgpt.com "Progressive Disclosure"))

Scoir также использует search + progressive filters для academic focus, degree, location и других параметров. ([help.scoir.com](https://help.scoir.com/article/y4io5y6rdb-searching-for-colleges?utm_source=chatgpt.com "Search and Suggest Colleges to Your Student"))

---

# 9. Explore URL state

Фильтры должны жить в query parameters:

```text
/universities?country=italy&degree=bachelor&field=business
```

Плюсы для продукта:

```text
back button works
share link works
refresh preserves state
```

---

# 10. Saved

Не создаём primary:

```text
Saved Universities
```

В Explore:

```text
All
Saved
```

или фильтр:

```text
Saved only
```

---

# 11. UNIVERSITY PROFILE

### URL

```text
/universities/[institutionSlug]
```

Пример:

```text
/universities/stanford-university
```

---

# 12. University navigation

Я бы оставил **4 основных раздела**:

```text
Overview
Programs
Admissions
Cost & Aid
```

Optional позже:

```text
Updates
```

Scoir использует очень похожую проверенную информационную декомпозицию: Overview, Academics, Admissions, Cost & Aid и Student Life. Нам Student Life пока не нужен как core Ekho. ([help.scoir.com](https://help.scoir.com/article/77ayu7ezp7-college-details-pages?utm_source=chatgpt.com "For Students: College profiles"))

---

# 13. University — Overview

```text
Stanford University

Stanford, California, USA

[Save] [Add application]


Overview

Key facts
Location
Institution type
Study levels
Languages

Upcoming deadlines

Programs

Cost snapshot

Admissions snapshot

Official sources
Last verified
```

---

# 14. University — Programs

### URL

```text
/universities/[institutionSlug]/programs
```

Содержит:

```text
Search programs

Bachelor
Master
Doctoral

Computer Science
Economics
Design
...
```

---

# 15. Program Profile

### URL

```text
/universities/[institutionSlug]/programs/[programSlug]
```

Пример:

```text
/universities/bocconi-university/programs/world-bachelor-in-business
```

---

# 16. Program page

Основные sections:

```text
Overview
Requirements
Deadlines
Cost & Aid
```

Не повторять всю university page.

---

# 17. Program selectors

Сверху могут находиться:

```text
Intake
Fall 2027 ▼

Application round
Regular ▼

Applicant profile
International · Russian qualification
```

Поскольку requirements могут зависеть от этих dimensions.

---

# 18. Selected intake в URL

Например:

```text
?intake=fall-2027
&round=regular
```

Identity программы остаётся той же.

Поэтому intake не надо превращать в новый program URL.

---

# 19. REQUIREMENTS FOR ME

**Не отдельный primary раздел Ekho.**

Он появляется там, где нужен.

### На program page:

```text
Requirements

For you

✓ Qualification accepted
✓ DET satisfied
! SAT action required
? Transcript requirement unclear
```

---

# 20. Requirements without profile

Если информации недостаточно:

```text
Requirements

Tell us your education system
to see requirements for you.

[Add education]
```

Не заставлять пользователя при регистрации проходить 20 вопросов.

---

# 21. COST & AID

Тоже **не primary navigation**.

На university/program:

```text
Tuition
Estimated living costs
Scholarships
Need-based aid
International eligibility
Required forms
Aid deadline
```

В application этот же information domain превращается уже в **личные действия**.

---

# 22. ADD APPLICATION

Основные точки входа:

```text
University page
Program page
Explore result
```

Кнопка:

```text
Add to applications
```

Не заставлять сначала искать раздел Applications.

---

# 23. APPLICATIONS

### URL

```text
/app/applications
```

Common App и Scoir оба делают отдельное пространство для выбранных colleges/applications и показывают требования/дедлайны именно относительно этого списка. ([Common App](https://www.commonapp.org/apply/first-year-students/?utm_source=chatgpt.com "Application guide for first-year students"))

---

# 24. Applications layout

Default:

```text
My Applications

[All] [In progress] [Submitted] [Decisions]

University       Deadline      Progress     Status

Stanford         Jan 5         8/12         In progress
MIT              Jan 6         10/13        In progress
Bocconi          Jan 22        9/9          Ready
```

Не board по умолчанию.

Table/list лучше подходит для сравнения deadlines/status.

Scoir также предоставляет table representation college list с status и важными comparative fields. ([help.scoir.com](https://help.scoir.com/article/43ghnlfaf5-for-students-view-your-college-list-in-a-data-table-format?utm_source=chatgpt.com "For Students: View your college list in a table layout"))

---

# 25. Optional view

Можно добавить позже:

```text
List | Board
```

Board:

```text
Researching
Planning
In progress
Submitted
Decisions
```

Но **P1**, не P0.

---

# 26. APPLICATION DETAIL

### URL

```text
/app/applications/[applicationId]
```

Например:

```text
/app/applications/8a72...
```

Не использовать название университета как identity application.

---

# 27. Application navigation

Вот здесь я бы сделал ровно:

```text
Overview
Checklist
Materials
Aid
Timeline
```

---

# 28. Application — Overview

Главная страница конкретной заявки:

```text
Stanford University

Regular Decision
Jan 5


Next action
────────────
Finish supplemental essay

Progress
8 / 12 requirements


Upcoming
────────
Essay          Dec 20
Transcript     Jan 5
Application    Jan 5


Important
─────────
SAT policy changed 2 days ago
```

---

# 29. Application — Checklist

Объединяет:

```text
Requirements
+
Tasks
```

Но **не смешивает их в database**.

UI:

```text
Academic

✓ Qualification
✓ GPA
! SAT

Language

✓ DET

Documents

✓ Transcript
○ Recommendation

Writing

○ Personal statement
○ Supplement #1
```

Common App dashboard/checklists также используются для tracking deadlines и college-specific requirements. ([Common App](https://www.commonapp.org/mobile/?utm_source=chatgpt.com "Common App for mobile"))

---

# 30. Requirement detail

Click:

```text
DET score

Status
✓ Satisfied

Your score
135

Required
120+

Applies because
International applicant

Source
Official Stanford Admissions

Verified
2 days ago
```

Это главный trust interaction Ekho.

---

# 31. MATERIALS

### URL

```text
/app/applications/[id]/materials
```

Здесь:

```text
Documents
Essays
Recommendations
Test scores
Portfolio
```

Не делать пять разных primary tabs.

---

# 32. Documents

Application-specific view:

```text
Transcript
Uploaded ✓

Passport
Uploaded ✓

Financial statement
Missing
```

Файл может использоваться в нескольких applications.

---

# 33. Essay

```text
Supplemental Essay #1

Prompt
...

Limit
250 words

Status
Draft

[Open]
```

Мы пока не превращаем Ekho в полноценный Google Docs.

---

# 34. Recommendations

```text
Teacher recommendation
Required: 2

1 received
1 pending
```

---

# 35. Aid

### `/app/applications/[id]/aid`

Персональный финансовый слой:

```text
Estimated cost

Tuition
Living costs
Application fees


Aid

Need-based aid
International eligibility

Scholarships


Actions

○ CSS Profile
○ Scholarship form
○ Financial documents
```

---

# 36. Timeline

```text
Aug 12
University saved

Aug 15
Application created

Sep 1
DET requirement satisfied

Oct 3
Deadline changed

Jan 2
Application submitted
```

Notes могут быть здесь же либо справа.

Отдельный primary Notes раздел не нужен.

---

# 37. UPDATES

### URL

```text
/app/updates
```

Это **не университетские новости**.

Только изменения, которые могут изменить действие пользователя:

```text
Deadline changed

Requirement changed

Test policy changed

Tuition changed

Financial aid policy changed

Scholarship changed

Program/application status changed
```

---

# 38. Updates feed

```text
Updates


MIT
English requirement changed

Old
DET 120

New
DET 125

Affected
Your MIT application

Detected
Aug 11

[View source]
```

---

# 39. Update detail

### URL

```text
/app/updates/[changeEventId]
```

Показывает:

```text
what changed
old value
new value

who is affected

when detected

official source

snapshot/history
```

---

# 40. DOCUMENT LIBRARY

### URL

```text
/app/documents
```

Secondary navigation.

Не primary.

```text
Documents

Transcript
Passport
DET certificate
CV
Financial documents
...
```

Отсюда пользователь может управлять reusable files.

---

# 41. APPLICANT PROFILE

### URL

```text
/app/profile
```

Sections:

```text
Basics
Education
Qualifications
Tests
Languages
Financial profile
```

---

# 42. Profile philosophy

Profile — **не onboarding questionnaire**.

Ekho постепенно говорит:

```text
To check your MIT English requirement,
tell us your language of instruction.
```

И добавляет конкретный field.

Это соответствует progressive disclosure и снижению cognitive load в формах. ([Nielsen Norman Group](https://www.nngroup.com/articles/progressive-disclosure/?utm_source=chatgpt.com "Progressive Disclosure"))

---

# 43. SETTINGS

### URL

```text
/app/settings
```

Только:

```text
Account
Notifications
Appearance
Privacy
Data
```

Не забивать settings product functionality.

---

# 44. Compare

### Public URL

```text
/compare
```

Состояние:

```text
/compare?universities=stanford,mit,bocconi
```

Не primary navigation.

Доступ через:

```text
Explore → Compare
```

---

# 45. Free Tools

Для acquisition:

```text
/tools
```

И:

```text
/tools/gpa-converter
/tools/deadline-calendar
/tools/det-checker
/tools/cost-calculator
/tools/essay-word-counter
...
```

Каждый tool — отдельная SEO landing page.

---

# 46. Marketing/public pages

P0:

```text
/

 /universities
 /universities/[slug]

 /compare

 /tools
 /tools/[slug]

 /methodology

 /login
 /signup

 /privacy
 /terms
```

---

# 47. Methodology

### `/methodology`

Я считаю обязательной.

Объясняет:

```text
Where Ekho gets data

What Verified means

How often information is checked

Official vs derived data

How changes are detected

What Unknown means
```

Это усиливает доверие к data product.

---

# 48. Auth

```text
/login
/signup
```

После signup:

```text
/onboarding
```

Но onboarding минимальный.

Например:

```text
What are you looking for?

Bachelor
Master
Other

Expected start
2027
```

→ `/app`

Остальное progressive profiling.

---

# 49. No giant onboarding

Common App требует много данных, потому что фактически является application submission system. Ekho пока решает другую задачу — research + management — поэтому копировать объём Common App onboarding нам нет смысла. Common App действительно собирает profile, education, testing, activities и college-specific information для самой заявки. ([Common App](https://www.commonapp.org/apply/first-year-students/?utm_source=chatgpt.com "Application guide for first-year students"))

---

# 50. Mobile navigation

Bottom nav:

```text
Home
Explore
Applications
Updates
```

Profile/avatar:

```text
top-right
```

Так mental model одинаков на desktop/mobile.

---

# 51. Desktop layout

```text
┌─────────────┬──────────────────────────────────┐
│             │                                  │
│ EKHO        │                                  │
│             │                                  │
│ Home        │             CONTENT              │
│ Explore     │                                  │
│ Applications│                                  │
│ Updates     │                                  │
│             │                                  │
│             │                                  │
│ Profile     │                                  │
└─────────────┴──────────────────────────────────┘
```

Sidebar collapsible позже.

Notion использует sidebar как центральный navigation hub и позволяет его скрывать для focused workspace. ([Notion](https://www.notion.com/help/navigate-with-the-sidebar?utm_source=chatgpt.com "Navigate with the sidebar"))

---

# 52. Breadcrumbs

Используем только там, где hierarchy реально помогает.

Например:

```text
Universities
>
Bocconi
>
World Bachelor in Business
```

Не:

```text
Home > Applications > Stanford > Checklist
```

везде подряд.

---

# 53. URL rules — Codex должен соблюдать

### Public entities

```text
human-readable slug
```

```text
/universities/stanford-university
```

### Private objects

```text
opaque database ID
```

```text
/app/applications/01K...
```

Никогда:

```text
/app/applications/george-stanford-2027
```

---

# 54. Slug changes

Public entity:

```text
old slug
→ 301
→ new slug
```

Internal database identity не меняется.

---

# 55. Canonical public URL

У одного institution только один canonical URL.

Не создавать:

```text
/app/universities/stanford

/university/stanford

/college/stanford

/universities/stanford
```

одновременно.

---

# 56. Route state rule

### PATH

для настоящей сущности/раздела:

```text
/universities/stanford/programs
```

### QUERY PARAM

для выбранного состояния:

```text
?intake=fall-2027
```

### LOCAL UI STATE

для временных вещей:

```text
dropdown open
tooltip
hover
```

Очень важно для Codex.

---

# 57. Meaningful tabs get URLs

Например:

```text
/app/applications/[id]/checklist

/app/applications/[id]/materials

/app/applications/[id]/aid
```

Чтобы:

```text
refresh works
back works
deep-link works
```

Next.js App Router напрямую поддерживает file-system routing и dynamic route segments для таких сущностей. ([Next.js](https://nextjs.org/docs/app/api-reference/file-conventions/dynamic-routes?utm_source=chatgpt.com "File-system conventions: Dynamic Segments"))

---

# 58. Search preview — P1

Позже можно:

```text
Explore

Stanford
MIT
Harvard
```

клик:

```text
              ┌───────────────┐
              │ Stanford      │
              │               │
              │ key data      │
              │ requirements  │
              │ aid           │
              │               │
              │ Open profile →│
              └───────────────┘
```

Scoir использует похожий preview → full College Details workflow. ([help.scoir.com](https://help.scoir.com/article/43ghnlfaf5-for-students-view-your-college-list-in-a-data-table-format?utm_source=chatgpt.com "For Students: View your college list in a table layout"))

Next.js Intercepting Routes позволяют сделать modal/preview с отдельным deep-link URL, если позже захотим такой polish. ([Next.js](https://nextjs.org/docs/app/api-reference/file-conventions/intercepting-routes?utm_source=chatgpt.com "File-system conventions: Intercepting Routes"))

**P1, не сейчас.**

---

# 59. Next.js route organization

Codex позже может делать:

```text
src/app/

(marketing)/
    page.tsx
    methodology/

(public)/
    universities/
    compare/
    tools/

(auth)/
    login/
    signup/
    onboarding/

(workspace)/
    app/
        page.tsx
        applications/
        updates/
        documents/
        profile/
        settings/
```

Route Groups в Next.js специально позволяют организовать разные layouts/группы страниц, не добавляя имя группы в URL. ([Next.js](https://nextjs.org/docs/app/api-reference/file-conventions/route-groups?utm_source=chatgpt.com "File-system conventions: Route Groups"))

---

# 60. Ownership rule

Это нужно **прямо сохранить Codex**, иначе он начнёт дублировать данные по экранам.

```text
UNIVERSITY / PROGRAM
→ objective admissions information

PROFILE
→ facts about applicant

APPLICATION
→ applicant × university relationship

CHECKLIST
→ personalized requirements/actions

HOME
→ cross-application priorities

UPDATES
→ changes affecting user

DOCUMENTS
→ reusable files

SETTINGS
→ product/account preferences
```

---

# 61. Financial aid ownership

```text
University / Program
→ objective aid policy

Application
→ aid for this application

Home
→ urgent aid action
```

Не создавать три разных financial-aid databases/features.

---

# 62. Requirement ownership

```text
Program
→ official requirement

Profile
→ applicant fact

Application
→ personalized result

Home
→ next action
```

Это очень важно.

---

# 63. Information duplication rule

Одни данные могут **отображаться** в разных местах.

Но source of truth всегда один.

Например:

```text
Stanford deadline
```

видна:

```text
University
Application
Home
Calendar
```

Но хранится **один canonical fact**.

---

# 64. Empty states

Каждый major section должен иметь нормальный first-use state.

### Applications empty:

```text
You don't have any applications yet.

Find a university and add your first application.

[Explore universities]
```

### Updates empty:

```text
No important changes yet.
We'll show changes affecting your universities here.
```

### Home empty:

```text
Start by adding a university you're considering.
```

---

# 65. Error / unknown states

UI обязательно различает:

```text
No data
Unknown
Not applicable
Not published
Couldn't load
Outdated
Conflicting sources
```

Не показывать одинаковый `—`.

Это связано с уже сохранённым Data Standard.

---

# 66. Global hierarchy

Итого:

```text
EKHO

PUBLIC
│
├── Landing
│
├── Universities
│   │
│   └── University
│       │
│       ├── Overview
│       ├── Programs
│       │   └── Program
│       │       ├── Overview
│       │       ├── Requirements
│       │       ├── Deadlines
│       │       └── Cost & Aid
│       │
│       ├── Admissions
│       └── Cost & Aid
│
├── Compare
│
├── Tools
│
└── Methodology


WORKSPACE
│
├── Home
│
├── Explore → same Universities
│
├── Applications
│   │
│   └── Application
│       ├── Overview
│       ├── Checklist
│       ├── Materials
│       ├── Aid
│       └── Timeline
│
├── Updates
│
├── Documents
├── Profile
└── Settings
```

# 67. Самое главное решение

**Primary navigation = только:**

```text
Home
Explore
Applications
Updates
```

А всё сложное Ekho раскрывает **контекстно**.

Это и отличает наш IA от типичного admissions software, где пользователя встречает огромное количество sections/forms. Подход согласуется и с progressive disclosure NNGroup, и с тем, как успешные admissions systems разделяют discovery, college list/application management и college-specific requirements. ([Nielsen Norman Group](https://www.nngroup.com/articles/progressive-disclosure/?utm_source=chatgpt.com "Progressive Disclosure"))

