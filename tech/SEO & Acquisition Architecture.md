# SEO / Acquisition Architecture
## 1. Цель
SEO Ekho не должно строиться вокруг:
* массового блога;
* AI-generated статей;
* тысяч почти одинаковых landing pages;
* keyword stuffing;
* страниц под каждую комбинацию фильтров;
* SEO-контента, который существует отдельно от продукта.
Основная acquisition-модель:
`Google / Search`
↓
`University / Program / Tool page`
↓
`полезная информация или результат`
↓
`Save University / Check Requirements / Add Deadline`
↓
`Signup`
↓
`Application Created`
↓
`Ekho core workflow`
Главный принцип:
> Каждая индексируемая страница должна быть полезной даже человеку, который никогда не зарегистрируется.
Это совпадает с текущей позицией Google: ranking systems ориентированы на helpful, reliable, people-first content, а массовое создание страниц ради поискового трафика может подпадать под scaled content abuse или doorway abuse.
---
# 2. Почему SEO органично подходит Ekho
Admissions имеет естественную search-driven структуру.
Пользователи уже ищут:
* конкретные университеты;
* admission requirements;
* application deadlines;
* tuition;
* financial aid;
* English requirements;
* GPA conversion;
* конкретные university programs;
* способы организовать applications.
Существующие продукты подтверждают такой паттерн.
BigFuture имеет публичные university profiles с admissions, academics и costs. CollegeVine индексирует university profiles с admissions и requirements. UCAS имеет публичный индекс университетов и десятков тысяч courses.
Reddit также многократно показывает использование spreadsheets для deadlines/requirements, вопросы международных applicants о GPA conversion и проблемы с определением реальной стоимости обучения.
Следовательно, SEO Ekho может быть продолжением основной utility продукта.
---
# 3. Основные SEO surfaces
Приоритет:
## P0
1. University pages
2. Program pages
3. University directory
4. Free tools
## P1
5. Subject hubs
6. Country hubs
7. Program directory
## P2
8. Source-grounded guides
Не начинать с большого generic blog.
---
# 4. Каноническая URL architecture
Использовать стабильные читаемые URLs.
```text
/
```
```text
/universities
```
```text
/universities/{university-slug}
```
```text
/universities/{university-slug}/programs/{program-slug}
```
```text
/programs
```
```text
/subjects/{subject-slug}
```
```text
/countries/{country-slug}/universities
```
```text
/tools
```
```text
/tools/{tool-slug}
```
Опционально позже:
```text
/guides/{guide-slug}
```
---
# 5. URL rules
URLs должны быть:
* lowercase;
* HTTPS only;
* human-readable;
* стабильными;
* без session IDs;
* без tracking information;
* без database UUID в публичном URL без необходимости;
* без случайных query parameters.
Google рекомендует логичные crawlable URLs и минимизацию альтернативных URLs, возвращающих одинаковый контент.
---
# 6. Slug strategy
University:
```text
harvard-university
```
Program:
```text
computer-science-bs
```
или официальное название:
```text
computer-science-bsc
```
Главное:
**slug stability > постоянная SEO-оптимизация slug.**
Не менять URL каждый раз при:
* небольшом rename;
* изменении title;
* изменении degree wording;
* изменении intake.
В database хранить:
```text
entity_id
canonical_slug
previous_slugs[]
```
---
# 7. Rename handling
Если university/program был переименован:
старый URL:
```text
/universities/old-name
```
должен делать permanent redirect:
```text
301
```
на:
```text
/universities/new-name
```
Google рекомендует permanent redirects для постоянно перемещённых страниц.
---
# 8. Indexable page matrix
## INDEX
### Homepage
```text
/
```
### University directory
```text
/universities
```
### Valid university profiles
```text
/universities/*
```
### Valid program profiles
```text
/universities/*/programs/*
```
### Tools
```text
/tools/*
```
### High-quality subject hubs
```text
/subjects/*
```
### High-quality country hubs
```text
/countries/*/universities
```
---
# 9. NOINDEX
По умолчанию:
```text
/search
```
```text
/search?q=...
```
filter combinations:
```text
?country=
?ranking=
?sort=
?tuition=
?test=
```
user-specific tool results;
personalized requirements;
compare combinations;
dashboard;
My Applications;
tasks;
documents;
account pages;
settings;
authentication pages;
internal admin pages;
empty university/program templates.
---
# 10. Search results
Internal search results **не являются SEO landing pages**.
Например:
```text
/search?q=computer+science
```
должен быть:
```text
noindex
```
и не попадать в sitemap.
Search нужен пользователю, а не Google.
---
# 11. Faceted navigation
Ekho Search потенциально может создавать огромное количество URL combinations:
```text
?country=us
```
```text
?country=us&program=cs
```
```text
?country=us&program=cs&tuition=low
```
```text
?country=us&program=cs&tuition=low&sort=ranking
```
Если дать поисковикам индексировать всё это, появляется практически бесконечное пространство URL и duplicate/faceted content.
Google прямо указывает, что filtering/sorting URLs часто создают duplicate content и рекомендует избегать индексирования filter/sort variations.
### Ekho rule
Arbitrary filters:
```text
NOINDEX
```
Никогда автоматически не превращать каждую комбинацию filters в SEO landing page.
---
# 12. Static SEO hubs вместо filter spam
Если существует реальный search intent:
вместо:
```text
/universities?country=germany
```
создаётся отдельная качественная страница:
```text
/countries/germany/universities
```
Но только если страница содержит самостоятельную пользу.
Например:
* список университетов;
* admissions system overview;
* language context;
* application platforms;
* tuition context;
* international applicant context;
* source information.
---
# 13. Doorway protection
Запрещено автоматически создавать:
```text
best-universities-in-x
```
для каждой страны/города/района,
если страницы практически одинаковы.
Запрещено создавать:
```text
harvard-requirements-2027
harvard-requirements-international-2027
harvard-sat-requirements-2027
harvard-gpa-requirements-2027
```
если всё это просто ведёт на одну и ту же информацию.
Google определяет создание substantially similar pages под похожие queries как потенциальный doorway abuse.
Одна сильная university page лучше десяти thin landing pages.
---
# 14. University page — основная SEO единица
Canonical URL:
```text
/universities/{university-slug}
```
Пример:
```text
/universities/stanford-university
```
---
# 15. University page — search intent
Страница должна уметь закрывать запросы типа:
```text
Stanford admissions
Stanford requirements
Stanford application deadline
Stanford international student requirements
Stanford tuition
Stanford financial aid international students
Stanford SAT requirement
Stanford DET requirement
Stanford application
```
Не создавать отдельную страницу под каждый вариант.
Одна качественная entity page должна охватывать связанные intents.
Google также рекомендует не создавать множество страниц исключительно под вариации запросов; их системы способны понимать релевантность без exact keyword page для каждой формулировки.
---
# 16. University page — mandatory sections
## Hero
Показывать:
* official university name;
* location;
* university type;
* undergraduate/graduate availability если известно;
* Save University;
* Add Application.
---
## Admissions snapshot
Например:
* application platform;
* active intake;
* major deadlines;
* testing policy;
* English requirement status;
* recommendation requirement;
* application fee;
* international applicant status.
Только подтверждённые данные.
---
## Application deadlines
Разделять:
* Early Action;
* Early Decision;
* Regular Decision;
* Priority;
* Rolling;
* specific intake;
* international deadline;
* financial aid deadline.
Не объединять разные типы deadline в одну дату.
---
# 17. Requirements
Показывать general requirements:
* school qualification;
* transcripts;
* standardized tests;
* English tests;
* essays;
* recommendations;
* interviews;
* portfolio;
* application platform;
* additional documents.
Если что-то неизвестно:
```text
Unknown
```
а не предположение.
---
# 18. Requirements for Me
SEO page показывает public/general requirements.
Поверх неё Ekho может предложить:
```text
Check requirements for me
```
После минимального profiling пользователь получает:
* Satisfied;
* Missing;
* Optional;
* Unknown;
* Action required.
Персональный результат не индексируется.
---
# 19. Tuition & financial aid
University page может показывать:
* tuition;
* mandatory fees;
* estimated living costs;
* housing;
* international pricing;
* scholarship availability;
* need-based aid;
* merit aid;
* need-blind/need-aware policy;
* financial aid forms;
* aid deadline.
Только если источник позволяет сделать утверждение.
College Board также отделяет sticker price, net price и дополнительные расходы и предупреждает, что actual net price зависит от обстоятельств конкретного студента.
---
# 20. International applicant section
Это важный differentiator Ekho.
Показывать:
* international eligibility;
* required qualification;
* English tests;
* test exemptions;
* translations;
* credential evaluation;
* international fees;
* visa-related admissions information только где релевантно;
* financial aid international eligibility;
* applicant-specific deadlines.
---
# 21. Programs section
University page должна crawlably ссылаться на program pages.
Например:
```text
Computer Science
Economics
Mechanical Engineering
Business
Psychology
```
Использовать реальные:
```html
<a href="...">
```
а не JavaScript-only navigation.
Google использует crawlable links для discovery; crawlers обычно не «нажимают» UI buttons для нахождения следующего контента.
---
# 22. Sources section
Каждая важная university page должна иметь:
## Official sources
Например:
* admissions;
* requirements;
* international admissions;
* tuition;
* financial aid;
* program catalog.
Источник должен быть:
* кликабельным;
* понятным;
* связанным с соответствующей информацией.
---
# 23. Freshness
Показывать:
```text
Last verified: August 2026
```
или более точную дату.
Лучше:
```text
Admissions requirements
Verified Aug 8, 2026
Financial aid
Verified Aug 5, 2026
```
чем один общий fake timestamp.
---
# 24. Freshness rule
`last_verified_at`
обновляется только когда информация действительно проверена.
Нельзя автоматически каждую ночь менять:
```text
Last updated: today
```
без проверки данных.
Google рекомендует, чтобы sitemap `lastmod` отражал последний **значимый** update и использует его, если значение постоянно оказывается достоверным.
---
# 25. University page title
Default template:
```text
{University Name} Admissions: Requirements, Deadlines & Costs | Ekho
```
Например:
```text
Stanford University Admissions: Requirements, Deadlines & Costs | Ekho
```
Title должен быть:
* unique;
* descriptive;
* concise;
* соответствовать содержанию.
Google использует `<title>`, headings и другие signals для формирования title link и может переписывать title.
---
# 26. H1
```text
Stanford University
```
Не:
```text
Stanford University Admissions Requirements Deadlines Tuition Financial Aid SAT GPA 2027
```
Не keyword stuffing.
---
# 27. Meta description
Пример:
```text
Explore Stanford University's current application deadlines, admissions requirements, international student policies, tuition, financial aid and official sources.
```
Meta description должна быть:
* уникальной;
* фактической;
* соответствовать странице;
* автоматически generated только из verified structured data.
Google может использовать meta description, но может выбрать snippet непосредственно из page content.
Не строить архитектуру вокруг магического «155 character rule».
---
# 28. Не использовать year keyword без причины
Не:
```text
Stanford Requirements 2027
```
просто ради SEO.
Допустимо только если страница действительно описывает:
```text
2027 entry
```
и данные относятся именно к этому admission cycle.
---
# 29. Program pages
Canonical:
```text
/universities/{university}/programs/{program}
```
Пример:
```text
/universities/eth-zurich/programs/computer-science-bsc
```
---
# 30. Почему program pages важны
University и program requirements часто отличаются.
UCAS, например, организует discovery вокруг university/provider + individual courses и показывает десятки тысяч отдельных course entries.
Поэтому Ekho нельзя хранить всё только на university level.
---
# 31. Program page — mandatory fields
## Identity
* program name;
* university;
* degree;
* degree level;
* faculty/school если известно.
## Study
* duration;
* mode;
* campus;
* language;
* start term/intake.
## Admissions
* program-specific qualifications;
* GPA/grades если официально определены;
* standardized tests;
* English tests;
* prerequisites;
* portfolio;
* interviews;
* documents.
## Deadlines
* intake;
* deadline;
* applicant category;
* round;
* international-specific deadline.
## Financial
* tuition;
* program-specific fees;
* scholarships если применимы.
## Application
* application platform;
* application URL/source.
## Trust
* official sources;
* verified date.
---
# 32. Program title
```text
{Program Name} at {University}: Requirements, Deadlines & Tuition | Ekho
```
Например:
```text
Computer Science BSc at ETH Zurich: Requirements & Deadlines | Ekho
```
---
# 33. Program H1
```text
Computer Science BSc
```
Subheading:
```text
ETH Zurich
```
---
# 34. Program indexability gate
Не каждый DB program автоматически становится indexable.
Program page может быть indexed только если:
* существует реальный program;
* university определён;
* canonical program name определён;
* active/current source существует;
* страница содержит meaningful program-specific information;
* нет unresolved duplicate;
* страница не является пустой template;
* данные не состоят исключительно из boilerplate.
---
# 35. Thin program pages
Если известны только:
```text
Program name
University
```
страница:
```text
NOINDEX
```
до получения полезных данных.
Google people-first guidance рекомендует substantial, useful content, а массовое создание low-value pages может подпадать под scaled content abuse.
---
# 36. University indexability gate
University page может быть indexable только если:
* university entity verified;
* official source существует;
* basic identity заполнена;
* есть полезная admissions/program/cost информация;
* существует clear source provenance;
* нет duplicate university entity;
* page отвечает реальной задаче applicant.
---
# 37. SEO publishing status
Добавить к SEO entities:
```text
seo_status
```
Allowed:
```text
draft
indexable
noindex
retired
```
Также:
```text
canonical_url
last_significant_update_at
last_verified_at
```
---
# 38. Sitemap generator
Sitemap получает URL только если:
```text
seo_status = indexable
```
и URL:
* отвечает `200`;
* canonical;
* public;
* indexable.
---
# 39. Free Tools strategy
Free tools нужны не ради количества.
Каждый tool должен делать хотя бы одно:
1. решать частую applicant problem;
2. приводить пользователя в University Intelligence;
3. приводить к Save University;
4. приводить в My Applications;
5. помогать Ekho получать relevant search demand.
---
# 40. Tool priority — #1
## University Deadline Finder / Calendar
**Strength: STRONG**
Page:
```text
/tools/university-deadline-calendar
```
Пользователь:
1. ищет university;
2. выбирает program/intake если необходимо;
3. видит verified deadlines;
4. видит official source;
5. добавляет deadline;
6. сохраняет university/application.
Почему fit высокий:
Reddit регулярно показывает, что applicants используют spreadsheets именно для deadlines и requirements.
Этот tool непосредственно использует core Ekho data.
---
# 41. Deadline tool SEO
Index:
```text
/tools/university-deadline-calendar
```
Не создавать тысячи:
```text
/tools/deadline/stanford
/tools/deadline/harvard
/tools/deadline/mit
```
если информация уже существует на university/program pages.
Это создало бы cannibalization и duplicate landing pages.
University deadline search должен вести либо:
* на university page;
* program page;
* общий tool.
---
# 42. Tool #2
## International GPA Converter
**Acquisition: STRONG**
**Trust risk: MEDIUM/HIGH**
URL:
```text
/tools/gpa-converter
```
User pain подтверждается повторяющимися вопросами international applicants о переводе разных national grading systems в US-style GPA.
Но Reddit также показывает важную проблему: универсального корректного GPA conversion может не существовать, а некоторые institutions сами интерпретируют international grades или требуют credential evaluation.
---
# 43. GPA tool trust rule
Нельзя писать:
```text
Your official US GPA is 3.84
```
если Ekho не располагает официальной methodology.
Вместо:
```text
Estimated 4.0-scale equivalent
```
или:
```text
This conversion is for comparison only.
Your university may evaluate grades differently.
```
Если конкретный university публикует свою formula:
можно использовать:
```text
University-specific method
```
с official source.
---
# 44. GPA tool inputs
Например:
* country;
* grading scale;
* grade;
* course grades;
* credits если нужны.
Не требовать account для первого расчёта.
Immediate value сначала.
Signup только после результата:
```text
Save this to my Ekho profile
```
---
# 45. Tool #3
## University Cost Calculator
**Strength: STRONG**
URL:
```text
/tools/university-cost-calculator
```
Calculate:
```text
tuition
+
mandatory fees
+
housing
+
food
+
insurance
+
books
+
estimated personal/living costs
-
known scholarships
```
---
# 46. Cost calculator restriction
Ekho не должен притворяться, что умеет точно предсказать financial aid.
Для international students net price calculators нередко вызывают вопросы о применимости и точности. Reddit содержит повторяющиеся вопросы именно на эту тему.
Поэтому:
```text
Estimated annual cost
```
можно делать.
```text
Guaranteed net price
```
нельзя.
---
# 47. Tool #4
## English Test Requirements Checker
**Strength: STRONG**
URL:
```text
/tools/english-test-requirements
```
Inputs:
* university;
* program;
* applicant education context;
* intended intake;
* optionally nationality/education language only если реально влияет.
Output:
```text
DET accepted
IELTS accepted
TOEFL accepted
minimum score
subscore requirement
exemption possible
unknown
```
Каждая critical value:
```text
official source
last verified
```
CTA:
```text
Add this university to Ekho
```
---
# 48. Tool #5
## Application Checklist
**Strength: STRONG PRODUCT FIT**
```text
/tools/application-checklist
```
User chooses:
```text
University
Program
Applicant type
```
Output:
* documents;
* tests;
* essays;
* recommendations;
* deadlines;
* application platform.
CTA:
```text
Create this application in Ekho
```
Это практически acquisition version основного Ekho workflow.
---
# 49. Tool #6
## University Compare
**Strength: STRONG**
```text
/tools/compare-universities
```
Compare:
* requirements;
* deadlines;
* tuition;
* financial aid;
* testing;
* program availability;
* international policies.
---
# 50. Compare SEO rule
Сам tool:
```text
INDEX
```
Arbitrary result:
```text
/compare?one=stanford&two=mit
```
по умолчанию:
```text
NOINDEX
```
Не создавать автоматически миллионы:
```text
stanford-vs-mit
stanford-vs-harvard
stanford-vs-yale
...
```
Google doorway guidance делает такую programmatic strategy рискованной, если страницы substantially similar и существуют в основном ради search ranking.
---
# 51. Curated comparison exception
В будущем отдельная comparison page может индексироваться только если:
* есть заметный user demand;
* она содержит real comparison;
* unique analysis;
* meaningful structured differences;
* не является автоматически сгенерированной таблицей.
---
# 52. Medium priority tools
## SAT/ACT checker
**MEDIUM**
Причина:
* сильный US intent;
* weaker global fit.
## Application Fee Calculator
**MEDIUM**
Хорошо связан с applications, но narrower demand.
## Reach / Target / Safety organizer
**MEDIUM / RISKY**
Не должен превращаться в fake admission probability engine.
---
# 53. Low priority
## Essay Word Counter
**WEAK**
Даже если SEO demand существует:
* почти не использует Ekho data;
* слабая differentiation;
* слабая связь с core workflow;
* высокая конкуренция с generic tools.
Можно сделать позже почти бесплатно, но не как acquisition wedge.
---
# 54. Tool results privacy
Нельзя помещать в URL:
* GPA;
* family income;
* test score;
* nationality profile;
* financial information;
* other sensitive applicant data.
Не:
```text
/tools/cost?income=42000&gpa=3.8
```
Лучше:
* state;
* local storage;
* POST;
* authenticated storage.
Result pages:
```text
NOINDEX
```
---
# 55. Subject hubs
Например:
```text
/subjects/computer-science
```
Может содержать:
* что относится к subject;
* related degree types;
* available programs;
* countries;
* universities;
* common application requirements;
* related tool/search.
Не писать generic 2,000-word SEO article ради word count.
---
# 56. Subject page indexability
INDEX только если:
* есть meaningful program inventory;
* есть unique structured information;
* page действительно помогает discovery.
Otherwise:
```text
NOINDEX
```
---
# 57. Country hubs
Например:
```text
/countries/germany/universities
```
Может содержать:
* universities available in Ekho;
* admissions system;
* application platforms;
* tuition context;
* international applicant context;
* common academic calendar;
* official national sources.
Но country information должна быть source-grounded.
---
# 58. Не создавать city spam
Не автоматически:
```text
universities-in-small-city-x
universities-near-small-city-x
best-universities-small-city-x
```
только потому что database позволяет это.
---
# 59. Homepage structured data
Использовать:
```text
WebSite
```
для сайта Ekho.
Properties:
```text
@context
@type: WebSite
name: Ekho
url: https://ekho.club/
```
Опционально:
```text
alternateName
```
Google прямо рекомендует `WebSite` structured data на homepage как основной способ указать preferred site name.
---
# 60. Organization structured data
Homepage/About:
```text
Organization
```
Минимально:
```text
name
url
logo
sameAs
```
Только реальные properties.
Google говорит, что Organization markup помогает понять и disambiguate organization и может влиять на отображение logo/organization information.
---
# 61. University structured data
На university page:
```text
WebPage
```
с:
```text
mainEntity
```
типа:
```text
CollegeOrUniversity
```
Также:
```text
BreadcrumbList
```
Schema.org содержит отдельный тип `CollegeOrUniversity`.
Но `CollegeOrUniversity` **не является обещанием отдельного Google rich result**.
Google rich-result eligibility определяется отдельным Search Gallery, а не самим существованием schema.org type.
---
# 62. University JSON-LD concept
```json
{
  "@context": "https://schema.org",
  "@type": "WebPage",
  "mainEntity": {
    "@type": "CollegeOrUniversity",
    "@id": "https://ekho.club/universities/example-university#university",
    "name": "Example University",
    "sameAs": "OFFICIAL_UNIVERSITY_URL"
  }
}
```
Дополнять только проверенными fields.
Например:
* address;
* alternateName;
* foundingDate;
только если они нужны и достоверны.
---
# 63. Program structured data
Основной semantic type:
```text
EducationalOccupationalProgram
```
Schema.org предоставляет этот тип специально для educational/occupational programs.
Concept:
```json
{
  "@context": "https://schema.org",
  "@type": "WebPage",
  "mainEntity": {
    "@type": "EducationalOccupationalProgram",
    "name": "Computer Science BSc",
    "provider": {
      "@type": "CollegeOrUniversity",
      "name": "Example University"
    }
  }
}
```
---
# 64. Course schema — осторожно
Google поддерживает `Course` rich results.
Но Google требует:
* реальный course;
* valid course name;
* provider;
* минимум три courses в course list;
* соответствующий `ItemList` на summary/all-in-one page;
* unique canonical URL для каждого item.
Поэтому **не маркировать все university degree programs как Course автоматически только ради rich results.**
Default:
```text
EducationalOccupationalProgram
```
Использовать:
```text
Course
```
только когда конкретная сущность действительно соответствует Course semantics и implementation полностью соответствует Google guidelines.
---
# 65. Breadcrumb structured data
Каждая:
* university page;
* program page;
* subject page;
* country page;
* tool page
должна иметь:
```text
BreadcrumbList
```
если breadcrumb реально существует в UI.
Google использует breadcrumb markup для понимания позиции страницы в hierarchy.
Пример:
```text
Universities
>
ETH Zurich
>
Computer Science BSc
```
---
# 66. Tools structured data
Google поддерживает:
```text
SoftwareApplication
```
и также расширенный subtype:
```text
WebApplication
```
для web apps.
Но Google SoftwareApplication rich result требует:
* app name;
* offer price;
* rating или review.
Поэтому:
### launch
Не придумывать fake rating.
Если rating/reviews отсутствуют:
использовать обычный semantic markup только если он корректен, но **не считать tool eligible for rich result**.
Никогда:
```text
ratingValue: 5
ratingCount: 100
```
из воздуха.
---
# 67. Structured data truth rule
Structured data должна описывать content, который реально существует на page.
Не:
```text
schema says $40,000 tuition
page says $45,000
```
Не schema-only SEO text.
Google требует, чтобы structured data соответствовала visible page content и guidelines.
---
# 68. JSON-LD
Использовать:
```text
application/ld+json
```
Google рекомендует JSON-LD среди поддерживаемых structured-data formats.
Structured data должна генерироваться из тех же DB values, что и page.
Не делать отдельный manually maintained SEO database.
---
# 69. Structured data testing
CI должен проверять JSON-LD:
* valid JSON;
* valid schema shape;
* required internal fields;
* no missing canonical URLs.
Перед rollout:
* Schema Markup Validator;
* Google Rich Results Test;
* Search Console URL Inspection.
Google рекомендует Rich Results Test и URL Inspection для validation.
---
# 70. Canonical architecture
Каждая indexable page:
```html
<link rel="canonical" href="CANONICAL_URL">
```
Self-canonical.
Google использует canonicalization для выбора representative URL среди duplicate/similar pages.
---
# 71. Tracking parameters
Например:
```text
/universities/stanford?utm_source=reddit
```
canonical:
```text
/universities/stanford
```
---
# 72. Duplicate query parameters
Если:
```text
?ref=
?utm_*
?source=
```
не меняют content:
canonical → clean page.
---
# 73. Pagination
Directory:
```text
/universities?page=2
```
Если pagination используется:
каждая page должна иметь собственный URL.
Google рекомендует каждому paginated page собственный canonical URL и crawlable `<a href>` links между pages.
Не canonical page 2 → page 1.
---
# 74. Infinite scroll
Можно использовать в UX.
Но crawlable fallback должен существовать.
Не полагаться только на:
```text
Load more
```
Googlebot обычно ищет links в `href` и не обязан нажимать JS buttons.
---
# 75. Rendering strategy
Public SEO pages должны предоставлять основной content непосредственно в initial HTML.
Использовать:
* SSR;
* SSG;
* server components;
в зависимости от stack.
Не делать critical university information только после client JS fetch.
Google способен выполнять JavaScript, но server-rendered HTML проще и надёжнее для crawling/indexing; Google сам описывает classic/SSR pages как content already present in initial response.
---
# 76. Dynamic rendering
Не использовать bot-specific dynamic rendering.
Google считает dynamic rendering workaround, а не рекомендуемым долгосрочным решением.
User и crawler должны получать один и тот же meaningful content.
---
# 77. Internal linking
Основная hierarchy:
```text
Homepage
↓
Universities
↓
University
↓
Programs
```
Дополнительно:
```text
Country
→ Universities
```
```text
Subject
→ Programs
```
```text
Tool
→ Relevant Universities
```
```text
University
→ Related Programs
```
```text
Program
→ University
```
Google использует links и anchor text для discovery и понимания relevance.
---
# 78. Anchor text
Хорошо:
```text
Computer Science BSc
```
```text
Stanford University
```
```text
International admissions requirements
```
Не:
```text
Click here
```
на каждом internal link.
---
# 79. Related universities
University page может показывать небольшой блок:
```text
Similar universities
```
Но similarity должна быть продуктово обоснована:
* country;
* program overlap;
* cost;
* university type;
* user-relevant attributes.
Не добавлять десятки links только ради PageRank distribution.
---
# 80. Sitemap architecture
Создать:
```text
/sitemap.xml
```
как sitemap index.
Например:
```text
/sitemaps/universities.xml
/sitemaps/programs.xml
/sitemaps/tools.xml
/sitemaps/subjects.xml
/sitemaps/countries.xml
```
---
# 81. Sitemap limit
Google ограничивает один sitemap:
```text
50,000 URLs
```
или:
```text
50 MB uncompressed
```
после этого нужен sitemap index/multiple files.
Для Ekho лучше разделять sitemap по entity type даже до достижения лимита.
Это упрощает monitoring.
---
# 82. Sitemap content rule
В sitemap попадут только:
* canonical URLs;
* 200 URLs;
* indexable URLs;
* public URLs.
Не:
* redirects;
* 404;
* noindex;
* filters;
* search;
* user pages;
* duplicate URLs.
Google рекомендует включать URLs, которые вы хотите видеть в Search, прежде всего canonical URLs.
---
# 83. Sitemap `lastmod`
Использовать:
```text
<lastmod>
```
только из:
```text
last_significant_update_at
```
Не:
```text
request time
```
Не:
```text
deployment time
```
если content не менялся.
Google игнорирует:
```text
<priority>
<changefreq>
```
и может использовать accurate `lastmod`.
---
# 84. robots.txt
Минимальная модель:
```text
User-agent: *
Disallow: /app/
Disallow: /account/
Disallow: /settings/
Disallow: /admin/
Disallow: /api/
Sitemap: https://ekho.club/sitemap.xml
```
Не block:
* public university pages;
* program pages;
* necessary JS;
* necessary CSS.
---
# 85. robots.txt не security
Private user documents никогда нельзя защищать только:
```text
robots.txt
```
Google прямо предупреждает, что robots.txt предназначен для crawler management, а URL может теоретически остаться известным/indexed без crawling; для private data нужны authentication/access controls.
---
# 86. noindex vs robots
Если URL нужно удалить из Search:
использовать:
```text
noindex
```
или настоящий access restriction/status code.
Не считать:
```text
Disallow
```
эквивалентом removal.
Google должен иметь возможность crawl page, чтобы увидеть `noindex`.
---
# 87. Deleted universities/programs
Если entity больше не существует и нет логичного replacement:
```text
410
```
или:
```text
404
```
Если program заменён новым equivalent URL:
```text
301
```
Google рекомендует true `404/410` для удалённого content вместо страницы с 200 response.
---
# 88. Soft 404 protection
Никогда:
```text
HTTP 200
```
с page:
```text
University not found
```
Это soft 404.
Использовать настоящий:
```text
404
```
Google прямо исключает soft-404 pages из Search и рекомендует meaningful HTTP status codes.
---
# 89. Missing data ≠ missing page
University существует, но tuition неизвестна:
не 404.
Показывать:
```text
Tuition data not currently verified
```
University entity остаётся valid.
---
# 90. Data freshness and SEO
Admissions information имеет temporal value.
Page должна различать:
```text
verified
unknown
outdated
not applicable
```
Не silently показывать прошлогодний deadline как current.
---
# 91. Significant data update
Если изменились:
* application deadline;
* requirement;
* standardized testing policy;
* English requirement;
* tuition;
* financial aid policy;
* application platform;
обновить:
```text
last_significant_update_at
```
и sitemap `lastmod`.
Google считает изменение main content/structured data/links значимым update для `lastmod`.
---
# 92. Official sources as trust layer
Ekho page должна делать больше, чем просто переписывать university website.
Value add:
```text
official data
+
normalization
+
structured requirements
+
international interpretation
+
last verified
+
personalization
+
next action
```
Google people-first guidance спрашивает, добавляет ли page original value beyond simple rewriting of other sources.
Это критически важно для SEO defensibility Ekho.
---
# 93. Source linking
Official source links являются обычными editorial citations.
Не ставить:
```text
nofollow
```
на каждый official university link без причины.
Google говорит, что regular links обычно не требуют специального `rel`; `ugc`/`sponsored` используются для соответствующих relationship types.
---
# 94. Localization — launch
На launch:
```text
English
```
как primary public SEO language.
Не создавать автоматически machine-translated:
```text
/de
/fr
/es
/it
```
пока нет качественного translated main content.
---
# 95. Localization — future
Когда появятся полноценные translations:
например:
```text
/de/universities/...
```
```text
/es/universities/...
```
Использовать separate URLs.
Google рекомендует отдельные URLs для разных language versions и `hreflang` для связи локализованных вариантов.
---
# 96. hreflang
Каждая localized page должна:
* ссылаться на себя;
* ссылаться на alternate versions;
* иметь reciprocal links.
Google требует взаимные hreflang references; иначе annotations могут быть ignored.
---
# 97. x-default
Когда localization станет реальной:
использовать:
```text
x-default
```
для default/fallback version.
Google рекомендует `x-default` для unmatched languages/locale selection.
---
# 98. Не менять language по IP на одном URL
Не отдавать:
```text
German content
```
и:
```text
English content
```
на одном URL в зависимости от IP.
Locale-adaptive pages сложнее для Google crawling, поскольку crawler не обязательно увидит все варианты.
---
# 99. Images
University page может показывать:
* university logo;
* campus image.
Но:
* использовать только legally permitted assets;
* real alt text;
* responsive formats;
* dimensions reserved;
* оптимизированные size/format.
Google также использует title, description, surrounding context и image metadata при понимании image content.
---
# 100. Image sitemap
Не нужен автоматически на launch.
Добавить только если:
* image discovery становится meaningful acquisition surface;
* есть большое количество original/useful university imagery.
Google поддерживает image sitemap как дополнительный discovery mechanism.
---
# 101. Performance
SEO public pages должны соблюдать уже зафиксированный Ekho Performance Budget.
Минимальный external reference:
```text
LCP ≤ 2.5 s
INP < 200 ms
CLS < 0.1
```
для категории `good` Core Web Vitals.
Не создавать отдельную более слабую performance policy для SEO pages.
---
# 102. Mobile
Public pages проектировать mobile-first.
Основной content на mobile не должен быть урезан относительно desktop.
Navigation может адаптироваться, content truth — нет.
---
# 103. Interstitials
Не показывать после organic landing:
```text
SIGN UP BEFORE YOU CAN READ
```
на весь экран.
User сначала должен получить value.
Google page-experience guidance отдельно рекомендует избегать intrusive interstitials и делать основной content легко доступным.
---
# 104. Signup gates
Public:
* university overview;
* general requirements;
* deadlines;
* public costs;
* sources.
Free without signup.
Signup нужен для:
* Save;
* personalized requirements;
* My Applications;
* tasks;
* monitoring;
* personalized cost state.
Это одновременно:
* лучше UX;
* лучше SEO;
* соответствует immediate-value principle Ekho.
---
# 105. IndexNow
Ekho должен поддерживать:
```text
IndexNow
```
для search engines, поддерживающих protocol.
При:
```text
university published
program published
critical content updated
page deleted
```
отправлять canonical URL через IndexNow.
IndexNow официально предназначен для уведомления participating search engines о созданных, обновлённых и удалённых URLs.
---
# 106. IndexNow ≠ sitemap replacement
Оставляем оба:
```text
Sitemap
+
IndexNow
```
Sitemap описывает общий indexable inventory.
IndexNow сообщает о конкретных recent changes.
Bing также рекомендует IndexNow для additions/updates/removals.
---
# 107. Google Search Console
Обязательно:
* domain property verification;
* sitemap submission;
* indexing monitoring;
* performance monitoring;
* Core Web Vitals;
* structured-data reports;
* manual actions;
* security issues.
Google Search Console предоставляет index status, live testing и request crawling через URL Inspection.
---
# 108. Bing Webmaster Tools
Также подключить:
* domain;
* sitemap;
* IndexNow monitoring;
* indexing diagnostics.
Потому что Ekho global-first, а acquisition architecture не должна зависеть от одного search engine.
---
# 109. Не scrape Google rankings
Не строить собственный cron:
```text
search Google for 10,000 keywords every day
```
Google относит automated queries/rank-check scraping без разрешения к machine-generated traffic, нарушающему spam policies/Terms.
Использовать:
* Search Console Performance data;
* Search Console API;
* Bing Webmaster data;
* compliant SEO provider при необходимости.
---
# 110. SEO monitoring dimensions
Отдельно отслеживать:
```text
University pages
Program pages
Tools
Country hubs
Subject hubs
```
---
# 111. Core Search Console metrics
Для каждого page type:
* impressions;
* clicks;
* CTR;
* average position;
* indexed URLs;
* excluded URLs;
* search queries;
* countries;
* devices.
Search Console Performance report предоставляет impressions/clicks/search performance data.
---
# 112. Product acquisition metrics
SEO успех не заканчивается кликом.
Из Analytics:
```text
organic landing
→ university_saved
```
```text
organic landing
→ signup_completed
```
```text
organic landing
→ application_created
```
```text
organic tool
→ university_saved
```
```text
organic tool
→ signup_completed
```
---
# 113. Primary SEO business metric
Не:
```text
organic traffic
```
сам по себе.
Primary:
```text
Organic users reaching core product value
```
Например:
```text
Organic → University Saved
```
и затем:
```text
Organic → Application Created
```
---
# 114. Supporting SEO metrics
* non-branded organic clicks;
* organic signup;
* organic save rate;
* organic application creation;
* indexed quality pages;
* pages receiving organic clicks;
* tool usage;
* search → save;
* program → save;
* university → save.
---
# 115. No fake SEO targets
Не писать сейчас:
```text
100k organic users/month after 6 months
```
```text
20% CTR
```
```text
top 3 for Stanford requirements
```
без данных.
Launch:
1. establish baseline;
2. segment by page type;
3. measure conversion;
4. identify pages/query clusters;
5. only then set targets.
---
# 116. University SEO cohort
Отслеживать:
```text
number published
number indexed
number with impressions
number with clicks
number creating saves
```
---
# 117. Program SEO cohort
Отслеживать:
```text
number published
number indexed
number with impressions
number with clicks
program → save
program → application
```
---
# 118. Tool cohort
На каждый tool:
```text
organic entrances
tool started
tool completed
university viewed
university saved
signup completed
application created
```
---
# 119. Query → product feedback loop
Search Console queries используются для понимания:
* что applicants реально ищут;
* где не хватает data;
* где user wording отличается от Ekho terminology;
* какие programs имеют demand.
Но query data **не должно автоматически создавать страницы**.
---
# 120. Example
Search Console начинает показывать impressions по:
```text
does university x accept duolingo
```
Не создавать автоматически:
```text
/university-x-duolingo-requirements
```
Сначала проверить:
есть ли эта информация на University page.
Если нет:
улучшить соответствующий:
```text
English requirements
```
section.
---
# 121. SEO content expansion rule
Новая page type создаётся только если:
1. существует отдельный user intent;
2. существующая page не может нормально его закрыть;
3. новая page имеет independent value;
4. есть достаточные данные;
5. она не duplicate;
6. maintenance оправдан.
---
# 122. Guides
Guides не являются launch priority.
Позже допустимы:
```text
/guides/how-to-apply-to-universities-in-germany
```
```text
/guides/ielts-vs-det-for-university-admissions
```
только если Ekho способен дать:
* official sources;
* real analysis;
* structured workflow;
* useful tools/data.
---
# 123. Не делать generic AI blog
Запрещено:
```text
generate 10,000 articles
```
по prompts:
```text
Best colleges for X
How to get into X
X acceptance rate
X GPA
```
Google прямо предупреждает, что массовое generative-AI content без дополнительной user value может нарушать scaled content abuse policy.
---
# 124. AI use for SEO
AI можно использовать для:
* structuring source data;
* detecting missing fields;
* generating draft summaries from verified fields;
* metadata drafting;
* internal QA.
Но final factual output должен быть grounded в structured source data.
AI не должен придумывать:
* requirements;
* GPA;
* deadlines;
* tuition;
* acceptance rates;
* financial aid.
---
# 125. Duplicate prevention
Перед publish university/program:
проверять:
```text
canonical entity ID
official domain
normalized name
alternate names
location
program provider
degree
```
---
# 126. Entity aliases
Например:
```text
MIT
Massachusetts Institute of Technology
```
одна entity.
Не две SEO pages.
---
# 127. Program duplicates
Например:
```text
BSc Computer Science
Computer Science B.Sc.
Bachelor of Science in Computer Science
```
не обязательно три programs.
Normalization pipeline должен решить entity identity перед publication.
---
# 128. Internal search and SEO must use one entity model
SEO pages нельзя генерировать из отдельного content CMS, не связанного с product DB.
Source of truth:
```text
Ekho University Intelligence database
```
От неё получают данные:
```text
Search
University pages
Program pages
Requirements
SEO
Tools
Structured data
```
---
# 129. Public vs private architecture
Public SEO domain:
```text
ekho.club
```
Не выносить SEO university database на:
```text
seo.ekho.club
```
без необходимости.
Product application может использовать:
```text
ekho.club/app
```
или уже зафиксированную IA.
Основные public entities должны оставаться на strongest primary domain.
---
# 130. University page + logged-in user
Один и тот же canonical public URL.
Anonymous:
```text
Save university
Check requirements for me
```
Logged in:
```text
Saved
My requirements
Application status
```
Public server-rendered content остаётся тем же.
Не создавать:
```text
/universities/stanford?user=123
```
indexable.
---
# 131. Personalized content
Personalization может добавляться client-side после public content.
Она не должна заменять весь public SSR document.
Это сохраняет:
* crawlability;
* privacy;
* stable canonical page.
---
# 132. Indexation rollout
Не выпускать все database rows в Google в один момент просто потому, что они существуют.
Порядок:
## Phase 1
Наиболее complete university pages.
## Phase 2
Complete program pages.
## Phase 3
Country/subject hubs.
## Phase 4
Long-tail program expansion.
---
# 133. Для начального каталога
Даже если Ekho имеет 1,000 universities:
```text
DB university ≠ automatically SEO indexable university
```
Сначала publish high-confidence entities.
Incomplete entity остаётся:
```text
noindex
```
или unpublished.
---
# 134. New page publishing workflow
Когда university/program становится ready:
```text
data verification
↓
duplicate check
↓
SEO quality gate
↓
structured data validation
↓
canonical validation
↓
seo_status = indexable
↓
page published
↓
sitemap updated
↓
IndexNow notification
```
---
# 135. Update workflow
Critical data changed:
```text
source updated
↓
Ekho data updated
↓
page regenerated/revalidated
↓
last_significant_update_at updated
↓
sitemap lastmod updated
↓
IndexNow
```
---
# 136. Removal workflow
Entity permanently removed:
```text
remove from sitemap
↓
404/410
↓
IndexNow deletion notification
```
Если replacement существует:
```text
301 replacement
```
---
# 137. CI SEO checks
Каждый deployment должен проверять representative pages.
Tests:
* status `200`;
* canonical exists;
* canonical matches expected URL;
* title exists;
* H1 exists;
* meta description exists;
* robots state correct;
* JSON-LD valid;
* sitemap consistency;
* no private URLs in sitemap;
* no staging domain in metadata;
* no broken internal links.
---
# 138. Sitemap consistency test
Для каждого sitemap URL:
```text
HTTP 200
```
```text
indexable
```
```text
self-canonical
```
обязательны.
---
# 139. Noindex consistency
URL не может одновременно быть:
```text
seo_status = noindex
```
и находиться в sitemap.
CI failure.
---
# 140. Canonical consistency
Indexable URL:
```text
canonical = itself
```
кроме специально документированных duplicate cases.
---
# 141. Structured data consistency
Например:
DB:
```text
tuition = 50,000
```
visible page:
```text
50,000
```
JSON-LD:
```text
50,000
```
Один source of truth.
---
# 142. Link validation
Official sources:
* не должны возвращать очевидный 404;
* должны соответствовать university/program;
* регулярно проверяются Data Pipeline.
Broken source не должен автоматически удалять public page, но должен вызвать:
```text
source verification required
```
---
# 143. Index health dashboard
Показывать:
```text
Indexable URLs
Submitted URLs
Indexed URLs
Excluded URLs
Crawled not indexed
Discovered not indexed
404
Soft 404
Redirect errors
Canonical mismatch
Structured-data errors
```
---
# 144. Priority alerts
P0:
* homepage accidentally noindex;
* `/universities/*` accidentally noindex;
* robots blocks public content;
* sitemap empty;
* canonical points to staging;
* mass 5xx;
* private pages indexed.
P1:
* index coverage collapse;
* structured data mass failure;
* large unexplained organic traffic drop.
---
# 145. Structured-data rollout
Не выкатывать новый schema template сразу на весь catalog.
Порядок:
```text
5–20 representative pages
↓
validate
↓
Search Console check
↓
expand
```
Google также рекомендует deploy на нескольких pages, проверить Rich Results Test и URL Inspection, а затем расширять rollout.
---
# 146. SEO QA representative set
Обязательно включить:
* US university;
* UK university;
* EU university;
* university with multiple campuses;
* university with many programs;
* university with missing data;
* renamed university;
* undergraduate program;
* postgraduate program;
* free tool;
* country hub.
---
# 147. Core Web Vitals QA
Проверять отдельно:
* university page;
* program page;
* directory;
* tool page.
Не только homepage.
Google оценивает page experience в значительной степени на page level, хотя существуют и более широкие assessments.
---
# 148. Staging
Staging:
```text
NOINDEX
```
и желательно authentication.
Не полагаться только на robots.
Canonical staging pages не должен указывать на staging production-equivalent URL случайно.
---
# 149. Preview URLs
CMS/admin preview:
```text
NOINDEX
```
и inaccessible without authorization.
---
# 150. Legal/trust footer
Public pages должны ясно показывать:
```text
Ekho is not the university.
```
и official-source attribution там, где это может быть неоднозначно.
Нельзя выглядеть как официальный admissions portal конкретного university.
Google spam policies отдельно запрещают impersonation и deceptive pages.
---
# 151. University logo/domain
Всегда различать:
```text
Official university website
```
и:
```text
Ekho profile
```
CTA на official source обозначать явно.
---
# 152. No fake authority
Не писать:
```text
Official Stanford admissions requirements
```
если Ekho не является Stanford.
Писать:
```text
Stanford admissions requirements
Source: Stanford University
```
---
# 153. Acquisition loop
Лучший SEO loop Ekho:
```text
searches university requirement
↓
lands directly on answer
↓
sees official source + freshness
↓
checks Requirements for Me
↓
saves university
↓
Ekho creates application workspace
↓
deadlines/tasks keep user returning
```
Это значительно сильнее generic:
```text
SEO blog
↓
article
↓
signup banner
```
---
# 154. Tool acquisition loop
```text
searches GPA / deadline / cost question
↓
uses tool without signup
↓
receives result
↓
Save to Ekho
↓
signup
↓
result becomes part of profile/application
```
Free tool не должен быть dead-end.
---
# 155. SEO differentiation
BigFuture уже предоставляет detailed college profiles и cost information. CollegeVine предоставляет university profiles и admissions tools. UCAS предоставляет extensive course search.
Поэтому Ekho не выигрывает просто:
```text
мы тоже сделали university database
```
SEO differentiator должен быть:
```text
current verified data
+
official sources
+
global coverage
+
international requirements
+
program-level data
+
personalized requirements
+
immediate next action
```
---
# 156. Что НЕ является moat
Не moat:
* SEO articles;
* university descriptions;
* generic acceptance rates;
* AI summaries;
* listicles;
* GPA calculator сам по себе.
Всё это легко копируется.
---
# 157. Что может стать moat
Потенциально:
```text
structured admissions knowledge graph
+
program-level sources
+
change monitoring
+
personalization
+
historical changes
+
application state
```
SEO становится distribution layer над этим data advantage.
---
# 158. SEO launch priorities
## MUST
* University pages
* Program page architecture
* canonical URLs
* SSR/public HTML
* sitemap
* robots
* noindex rules
* redirects/404s
* BreadcrumbList
* WebSite
* Organization
* CollegeOrUniversity semantic data
* Search Console
* Bing Webmaster
* IndexNow
* Core Web Vitals
* source/freshness UI
* analytics attribution
---
# 159. SHOULD
* Deadline tool
* GPA converter
* Cost calculator
* English requirement checker
* Compare tool
* country hubs
* subject hubs
---
# 160. LATER
* guides;
* multilingual SEO;
* curated comparison pages;
* image SEO expansion;
* editorial content.
---
# 161. НЕ ДЕЛАТЬ
Никогда не:
* index internal search;
* index arbitrary filters;
* index user-specific results;
* index dashboard/application pages;
* generate thousands of thin pages;
* create pages for every keyword variation;
* mass-generate AI articles;
* fabricate structured-data ratings;
* fabricate freshness timestamps;
* use fake “2027” titles;
* publish pages without source provenance;
* use robots.txt as security;
* canonical all pagination to page 1;
* hide critical content behind JS-only interactions;
* rely on sitemap as guarantee of indexing;
* confuse Ekho with official university websites.
Google прямо указывает, что sitemap submission является hint, а не гарантией crawling/indexing.
---
# 162. Definition of Done
* [ ] Canonical public URL model defined.
* [ ] University URL implemented.
* [ ] Program URL implemented.
* [ ] Tool URL implemented.
* [ ] Internal search is noindex.
* [ ] Arbitrary filters are noindex.
* [ ] Private app pages cannot be indexed.
* [ ] `seo_status` implemented.
* [ ] University indexability gate implemented.
* [ ] Program indexability gate implemented.
* [ ] Duplicate entity protection implemented.
* [ ] Slug history + 301 implemented.
* [ ] True 404/410 implemented.
* [ ] Soft 404 prevented.
* [ ] Self-canonical implemented.
* [ ] Pagination self-canonical.
* [ ] Crawlable `<a href>` navigation exists.
* [ ] Public entity content renders in initial HTML.
* [ ] University title template implemented.
* [ ] Program title template implemented.
* [ ] Unique meta descriptions implemented.
* [ ] H1 logic implemented.
* [ ] Visible sources implemented.
* [ ] Visible verified date implemented.
* [ ] Sitemap index implemented.
* [ ] Sitemap segmented by entity type.
* [ ] Sitemap contains only canonical indexable URLs.
* [ ] Accurate `lastmod` implemented.
* [ ] robots.txt implemented.
* [ ] WebSite structured data implemented.
* [ ] Organization structured data implemented.
* [ ] BreadcrumbList implemented.
* [ ] CollegeOrUniversity semantic markup implemented.
* [ ] EducationalOccupationalProgram markup implemented.
* [ ] Course schema not abused.
* [ ] Structured-data CI validation implemented.
* [ ] Google Search Console connected.
* [ ] Bing Webmaster Tools connected.
* [ ] IndexNow implemented.
* [ ] Core Web Vitals monitored.
* [ ] Organic acquisition attribution implemented.
* [ ] University/program/tool cohorts separated.
* [ ] SEO health dashboard exists.
* [ ] staging/preview protected from indexing.
* [ ] no fake reviews/rankings/data.
* [ ] no mass AI-generated SEO pages.
---
# 163. Финальная SEO Architecture
```text
                         Google / Search
                              │
            ┌─────────────────┼──────────────────┐
            │                 │                  │
      University Page    Program Page         Free Tool
            │                 │                  │
            ├──── Official Sources ──────────────┤
            │
      Verified Ekho Data
            │
        Immediate Value
            │
     Save / Check / Compare
            │
           Signup
            │
    Application Created
            │
 Requirements + Tasks
            │
          Return
```
---
# 164. Финальный принцип
**Ekho SEO — не content machine.**
Это публичная версия Ekho University Intelligence.
Каждая indexed page должна давать:
```text
answer
+
structured data
+
source
+
freshness
+
next action
```
University/program database создаёт search acquisition.
Free tools создают entry points.
Personalization превращает visitor в user.
My Applications превращает acquisition в retention.
Именно такая архитектура сохраняет одновременно:
* SEO scalability;
* product simplicity;
* data trust;
* international differentiation;
* long-term defensibility.
