# Quality / Testing Standard — Ekho v1

**Главный принцип:** Ekho должен быть не просто визуально качественным. Ошибка в deadline / eligibility / financial aid считается **серьёзнее обычного UI-бага**.

---

# 1. Quality layers

Разделяем качество на 5 независимых слоёв:

```text
1. Functional correctness
2. Accessibility
3. Responsive + browser compatibility
4. Visual/UI regression
5. Admissions data correctness
```

PR не считается готовым только потому, что «страница открывается».

---

# 2. Accessibility baseline

Для Ekho фиксируем:

> **WCAG 2.2 Level AA**

как обязательный baseline.

WCAG 2.2 — действующий стандарт W3C; Level AA включает требования к keyboard accessibility, contrast, reflow, forms, target size и другим критическим вещам. ([W3C](https://www.w3.org/TR/WCAG22/?utm_source=chatgpt.com "Web Content Accessibility Guidelines (WCAG) 2.2"))

Не пытаемся сейчас делать полный AAA.

**Решение: WCAG 2.2 AA = release requirement.**

---

# 3. Keyboard

Весь основной Ekho должен работать без мышки.

Обязательно:

```text
Tab
Shift+Tab
Enter
Space
Escape
Arrow keys where expected
```

Проверяем:

- navigation;
    
- university search;
    
- filters;
    
- dropdowns;
    
- dialogs;
    
- forms;
    
- adding university;
    
- application workflow;
    
- requirements;
    
- documents;
    
- settings.
    

Keyboard focus всегда должен быть видимым — это непосредственно входит в WCAG. ([W3C](https://www.w3.org/TR/WCAG22/?utm_source=chatgpt.com "Web Content Accessibility Guidelines (WCAG) 2.2"))

---

# 4. Focus

Запрещено:

```css
outline: none;
```

без нормальной замены.

Focus:

- хорошо заметен;
    
- не скрывается sticky headers/modals;
    
- следует логичному порядку;
    
- после закрытия modal возвращается туда, откуда modal был открыт.
    

---

# 5. Contrast

Минимум WCAG AA:

```text
normal text       ≥ 4.5:1
large text        ≥ 3:1
important UI      ≥ 3:1
```

([W3C](https://www.w3.org/TR/WCAG22/?utm_source=chatgpt.com "Web Content Accessibility Guidelines (WCAG) 2.2"))

Особенно проверять серый текст.

Для минималистичного Ekho нельзя превращать:

> minimalism → barely visible UI.

---

# 6. Color

Цвет **никогда не является единственным носителем смысла**.

Плохо:

```text
green = completed
red = missing
```

Хорошо:

```text
✓ Satisfied
! Action required
— Unknown
```

Цвет используется дополнительно.

---

# 7. Touch targets

WCAG 2.2 AA содержит minimum target-size criterion. ([W3C](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum?utm_source=chatgpt.com "Understanding SC 2.5.8 Target Size (Minimum) (Level AA)"))

Для Ekho делаем собственный более удобный стандарт:

```text
minimum interactive area: 44 × 44 CSS px
preferred touch area:      48 × 48 CSS px
```

48px также является рекомендуемым размером touch target в guidance web.dev. ([web.dev](https://web.dev/articles/accessible-tap-targets?utm_source=chatgpt.com "Accessible tap targets"))

Иконка сама может быть 16–24px.

Нажимаемая область — нет.

---

# 8. Forms

Каждое поле имеет:

```text
visible label
correct input type
keyboard access
clear validation
clear error
```

Placeholder ≠ label.

Ошибка:

```text
Invalid
```

плохо.

Нужно:

```text
Enter a valid email address.
```

WCAG требует, чтобы ошибка была идентифицируема текстом, а поля имели понятные labels/instructions. ([W3C](https://www.w3.org/WAI/WCAG22/Understanding/error-identification.html?utm_source=chatgpt.com "Understanding Success Criterion 3.3.1: Error Identification | WAI - W3C"))

---

# 9. Zoom / reflow

Ekho обязан оставаться usable:

```text
browser zoom → 200%
viewport → 320 CSS px
```

без потери основной информации или функций.

WCAG AA требует text resize до 200% и reflow при ширине, эквивалентной 320 CSS px, с исключениями для действительно двумерного контента. ([W3C](https://www.w3.org/TR/WCAG22/?utm_source=chatgpt.com "Web Content Accessibility Guidelines (WCAG) 2.2"))

---

# 10. Screen-reader testing

Automation недостаточно.

W3C прямо предупреждает, что автоматические accessibility tools **не могут определить полную accessibility compliance**, требуется человеческая проверка. ([W3C](https://www.w3.org/WAI/test-evaluate/tools/selecting/?utm_source=chatgpt.com "Selecting Web Accessibility Evaluation Tools"))

Поэтому перед крупным release:

```text
macOS/iOS → VoiceOver + Safari

Windows → NVDA + Chrome
```

Проверяем минимум основные flows.

---

# 11. Automated accessibility

В CI:

```text
axe-core
+
Playwright
```

на основных страницах и изменённых routes.

Но:

```text
axe passing ≠ accessible
```

Даже axe не анализирует некоторые скрытые элементы, пока они не открыты, поэтому тест должен реально открыть modal/dropdown/menu перед scan. ([Deque](https://www.deque.com/axe/core-documentation/api-documentation/?utm_source=chatgpt.com "Axe API documentation"))

---

# 12. Responsive philosophy

Никаких:

```text
desktop version
tablet version
mobile version
```

как трёх отдельных продуктов.

Один fluid responsive UI.

Responsive design должен адаптироваться к произвольным размерам экранов, а не только конкретным устройствам. ([web.dev](https://web.dev/articles/responsive-web-design-basics?utm_source=chatgpt.com "Responsive web design basics | Articles"))

---

# 13. Mandatory responsive widths

Codex должен проверять минимум:

```text
320
375
390
768
1024
1280
1440
1920 px
```

Не потому что это конкретные iPhone/MacBook, а чтобы поймать transitions между layout states.

---

# 14. Responsive rules

На любом размере:

- нет случайного horizontal scroll;
    
- текст не обрезается;
    
- controls не перекрываются;
    
- modals помещаются;
    
- sticky elements не закрывают content;
    
- таблицы имеют специальное mobile behavior;
    
- tooltips не являются единственным способом получить информацию;
    
- forms работают с mobile keyboard;
    
- portrait/landscape не ломают layout.
    

---

# 15. Breakpoints

Не проектируем:

```text
if iPhone
if iPad
if MacBook
```

Breakpoints появляются там, где **ломается layout**.

Предпочтение:

```text
content-driven breakpoints
container queries where useful
fluid sizing
```

а не десяткам device-specific hacks.

---

# 16. Browser support

### Tier 1 — обязательно

```text
Chrome desktop
Safari macOS
Firefox desktop
Edge desktop
Safari iOS
Chrome Android
```

Ekho ориентируется на browser set, близкий к Web Platform Baseline: Safari, Chrome, Edge и Firefox на desktop/mobile. ([MDN Web Docs](https://developer.mozilla.org/en-US/docs/Glossary/Baseline/Compatibility?utm_source=chatgpt.com "Baseline (compatibility) - Glossary - MDN Web Docs"))

### Versions

Поддерживаем:

```text
current stable
+
previous major stable
```

для Tier-1 браузеров, где это практически возможно.

---

# 17. Web feature policy

Для core functionality предпочтительно:

```text
Baseline Widely Available
```

web platform features.

Новую limited-availability feature разрешаем только если:

```text
progressive enhancement
OR
safe fallback
```

есть.

Baseline существует именно для оценки доступности web features между основными браузерами. ([MDN Web Docs](https://developer.mozilla.org/en-US/docs/Glossary/Baseline/Compatibility?utm_source=chatgpt.com "Baseline (compatibility) - Glossary - MDN Web Docs"))

---

# 18. Browser automation

Используем Playwright:

```text
Chromium
Firefox
WebKit
```

Playwright официально поддерживает эти три browser engines и mobile/device emulation. ([Playwright](https://playwright.dev/docs/browsers?utm_source=chatgpt.com "Browsers"))

Но важное ограничение:

> Playwright WebKit ≠ настоящий Safari.

Playwright прямо пишет, что branded Safari им не запускается; используется отдельная patched WebKit build. ([Playwright](https://playwright.dev/docs/browsers?utm_source=chatgpt.com "Browsers"))

Поэтому реальные Safari smoke tests всё равно нужны.

---

# 19. Actual-device smoke tests

Перед крупным production release:

```text
Safari / macOS
Safari / iPhone
Chrome / Android
Chrome / Windows or macOS
```

Минимум основные user journeys.

Не нужно вручную проверять 30 устройств.

---

# 20. Functional test stack

Для текущего Next.js stack:

### Unit

```text
Vitest
```

### E2E / browser

```text
Playwright
```

Next.js официально документирует оба варианта для testing. ([Next.js](https://nextjs.org/docs/app/guides/testing?utm_source=chatgpt.com "Guides: Testing"))

Не добавляем Cypress/Jest параллельно без причины.

---

# 21. Что покрывать unit tests

Unit tests нужны там, где есть **логика**, а не для каждой кнопки.

Например:

```text
deadline calculations
application progress
requirement status
GPA conversion
financial calculations
qualification matching
normalization
parsers
data transformation
```

Не надо:

```text
Button renders text "Save"
```

тестировать сотнями бессмысленных tests.

---

# 22. Integration tests

Обязательно для границ:

```text
API ↔ database
auth ↔ user
university ↔ program
application ↔ requirements
crawler ↔ parser
parser ↔ normalized data
```

Особенно там, где ошибка может записать неправильные данные.

---

# 23. E2E critical journeys

Минимальный suite:

```text
signup/login

search university
→ open university

add university
→ My Applications

create application
→ choose program/round

view Requirements for Me

mark task complete

deadline appears correctly

source link opens correctly
```

Playwright предназначен именно для end-to-end testing web applications. ([Next.js](https://nextjs.org/docs/pages/guides/testing/playwright?utm_source=chatgpt.com "Testing: Playwright"))

---

# 24. Visual regression

Используем только для важных UI.

Например:

```text
University page
Search
My Applications
Requirements
Application page
Compare
navigation
important dialogs
```

Playwright умеет сохранять reference screenshots и автоматически сравнивать новые screenshots с ними. ([Playwright](https://playwright.dev/docs/test-snapshots?utm_source=chatgpt.com "Visual comparisons"))

Не snapshot'им весь Ekho — будет слишком много noise.

---

# 25. Data correctness — отдельный стандарт

Это **самая критичная часть Ekho**.

Данные проверяем на 5 уровнях:

```text
source
↓
extraction
↓
schema
↓
semantic meaning
↓
user-specific interpretation
```

---

# 26. Critical admissions data

Считаем критическими:

```text
deadlines
application round
eligibility
qualification requirements
GPA/grade requirements
SAT/ACT
DET/IELTS/TOEFL
required documents
essay requirements
application platform
tuition
aid eligibility
scholarship eligibility
financial-aid deadline
international applicant rules
```

Ошибка здесь = **release/data blocker**.

---

# 27. Source hierarchy

Для critical facts:

### Level 1 — canonical

```text
exact official university/program page
```

### Level 2

```text
official university admissions
official financial aid
official tuition/bursar
official international admissions
```

### Level 3

```text
official centralized system
Common App
UCAS
government portals
```

### Level 4

```text
secondary sources
```

Level 4 можно использовать **для discovery**, но не как единственную canonical source critical facts.

UCAS прямо указывает, что университеты и конкретные courses устанавливают собственные entry requirements и они могут существенно различаться. ([ucas.com](https://www.ucas.com/applying/you-apply/what-and-where-study/entry-requirements?utm_source=chatgpt.com "University Entry Requirements"))

Common App также предоставляет requirements/deadlines, но сам в ряде случаев отправляет пользователя на website конкретного университета. ([Common App](https://www.commonapp.org/apply/first-year-students/?utm_source=chatgpt.com "Application guide for first-year students"))

---

# 28. Specificity rule

Если есть:

```text
University general requirement
```

и:

```text
Computer Science BSc requirement
```

для CS applicant приоритет имеет **конкретный program requirement**.

Контекст всегда сохраняем:

```text
university
campus
program
degree
applicant type
nationality
qualification
round
academic cycle
```

Иначе данные нельзя безопасно персонализировать.

---

# 29. No fake certainty

Если Ekho не знает:

```text
UNKNOWN
```

Если официальные источники противоречат:

```text
CONFLICT
```

Если информация устарела:

```text
STALE
```

Никогда:

```text
AI thinks probably required
```

→ показывать как факт.

---

# 30. Critical field provenance

Каждый critical fact должен иметь минимум:

```text
value
source_url
source_title
source_type
retrieved_at
last_verified_at
academic_cycle
applicant_context
```

Дополнительно internally:

```text
parser_version
source_hash
extraction_method
```

Это соответствует общей модели data provenance: должна существовать связь между производным fact и исходной entity/source. W3C PROV формализует именно такие provenance relationships через `wasDerivedFrom` и связанные сущности. ([W3C](https://www.w3.org/TR/prov-o/?utm_source=chatgpt.com "PROV-O: The PROV Ontology"))

---

# 31. Source URL rule

Не сохраняем:

```text
mit.edu
```

как доказательство SAT policy.

Нужен **deep link на конкретную страницу**, например:

```text
official admissions testing requirements page
```

Пользователь должен иметь возможность проверить факт сам.

---

# 32. Database correctness

Часть ошибок должна быть физически невозможна на уровне DB.

Используем:

```text
NOT NULL
UNIQUE
CHECK
FOREIGN KEY
ENUM/domain validation
```

где это соответствует модели.

PostgreSQL поддерживает именно эти integrity constraints для предотвращения некорректных состояний данных. ([PostgreSQL](https://www.postgresql.org/docs/current/ddl-constraints.html?utm_source=chatgpt.com "Documentation: 18: 5.5. Constraints"))

---

# 33. Semantic validation

Schema validation недостаточно.

Например:

```text
deadline = 2026-11-01
```

может быть валидной датой, но относиться:

- к другому program;
    
- domestic applicants;
    
- ED вместо EA;
    
- предыдущему cycle.
    

Поэтому проверяем **значение + контекст**.

---

# 34. Dates

Никогда не превращаем:

```text
November 1
```

автоматически в:

```text
2026-11-01 23:59:59 EST
```

если университет время не указал.

Храним различие:

```text
date-only
exact datetime
timezone known
timezone unknown
```

Не изобретаем precision.

---

# 35. Money

Каждая monetary value требует:

```text
amount
currency
period
academic cycle
applicant category
```

Например:

```text
$65,000
```

без:

```text
USD / year / 2026-27 / international
```

— недостаточная информация.

---

# 36. Tests policy

Не сводим всё к:

```text
SAT required: true/false
```

Нужны состояния вроде:

```text
required
test-optional
test-flexible
not-considered
recommended
conditional
unknown
```

Потому что реальные admissions systems используют гораздо более сложные категории; даже Common App Requirements Grid различает несколько test-policy вариантов. ([Common App](https://www.commonapp.org/apply/first-year-students/?utm_source=chatgpt.com "Application guide for first-year students"))

---

# 37. Source conflict

Если два **актуальных официальных** источника говорят разное:

НЕ делаем автоматически:

```text
source rank 1 wins
```

Делаем:

```text
CONFLICT
↓
re-fetch
↓
context comparison
↓
review
```

До разрешения конфликта Ekho не показывает одну сторону как гарантированную истину.

---

# 38. Source change validation

Когда monitoring обнаруживает:

```text
deadline:
Jan 1 → Jan 5
```

не публикуем изменение после единственного parser result.

Flow:

```text
change detected
↓
fresh fetch
↓
parse again
↓
semantic validation
↓
compare context/cycle
↓
publish OR pending review
```

Для high-impact changes ambiguity → manual review.

---

# 39. Freshness tiers

### Critical / time-sensitive

```text
deadlines
requirements
testing
aid deadlines
```

проверяются чаще всего.

### Medium

```text
tuition
fees
scholarships
```

регулярно в течение cycle.

### Stable

```text
location
school type
degree structure
```

реже.

Не используем один бессмысленный:

```text
verify everything every 30 days
```

для всех данных.

---

# 40. User-facing provenance

Для важных данных показываем:

```text
Official source ↗
Last verified: ...
```

Особенно:

```text
deadline
requirement
tuition
aid
```

Это становится частью **trust UX**, а не внутренней технической metadata.

---

# 41. Derived personalization

Например Ekho говорит:

```text
English test: Satisfied
```

Это не source fact.

Это **derived conclusion**.

Должны быть воспроизводимы:

```text
user profile
+
university requirement
+
matching rule version
=
Satisfied
```

То есть должна существовать возможность понять:

> почему Ekho дал именно такой статус?

---

# 42. Personalized statuses

Разрешаем только:

```text
Eligible
Satisfied
Missing
Optional
Action required
Unknown
Conflict
```

Где нет достаточных данных → `Unknown`.

Не:

```text
Probably eligible
94% eligible
AI confidence: 92%
```

без реально валидированной вероятностной модели.

---

# 43. PR test gate

Чтобы не жечь Codex/CI ресурсы, каждый PR **не гоняет весь мир**.

### Каждый PR

```text
lint
typecheck
relevant unit tests
production build
changed data/schema tests
Chromium smoke
accessibility scan changed flows
```

---

# 44. Main / staging gate

После merge:

```text
full relevant unit/integration suite

Playwright:
Chromium
Firefox
WebKit

critical E2E

visual regression

accessibility

data validation

staging smoke
```

Playwright Projects позволяет один и тот же test suite запускать в Chromium, Firefox и WebKit. ([Playwright](https://playwright.dev/docs/test-projects?utm_source=chatgpt.com "Projects"))

---

# 45. Nightly quality jobs

Не нужно тормозить каждый PR тяжёлыми проверками.

Nightly:

```text
larger cross-browser suite
source availability
stale data detection
source conflicts
parser failures
critical data anomalies
broken source links
```

---

# 46. Production smoke

После deployment автоматически:

```text
homepage loads
login works
search works
university loads
application workspace loads
API healthy
DB reachable
critical source rendering works
```

Если smoke fails:

```text
release unhealthy
→ rollback / investigate
```

---

# 47. Severity

### P0 — immediate

```text
data corruption
security/privacy failure
production unavailable
wrong data at scale
```

### P1 — release blocker

```text
wrong critical admissions fact
missing source for critical fact
broken application workflow
auth failure
critical browser broken
core accessibility failure
```

### P2

```text
non-critical functional bug
responsive issue with workaround
significant visual regression
```

### P3

```text
minor spacing
minor animation
tiny cosmetic inconsistency
```

---

# 48. Definition of Done for Codex

Задача **не готова**, пока Codex не проверил:

```text
Functionality
Types
Relevant tests
Accessibility
Responsive impact
Browser impact
Data impact
Source impact
Build
```

Completion report:

```text
Changed:
...

Tested:
...

Accessibility:
pass / N/A

Responsive:
pass / N/A

Data/source impact:
none / details

Risks:
none / details
```

---

# 49. Что Codex запрещено

Codex не должен:

- отключать failing test, чтобы CI стал зелёным;
    
- менять expected screenshot без проверки причины;
    
- добавлять `eslint-disable` вместо исправления проблемы;
    
- скрывать accessibility warning;
    
- обновлять critical admissions data без source;
    
- придумывать missing data;
    
- автоматически разрешать конфликт официальных sources;
    
- заменять `unknown` предположением;
    
- менять test expectations только потому, что implementation изменилась.
    

---

# 50. Quality dashboard позже

Для внутреннего monitoring:

```text
critical E2E pass rate
production error rate

universities verified
critical fields verified
stale critical fields
conflicting facts
broken official sources
parser failures

accessibility violations
browser failures
```

Не нужен огромный QA dashboard на launch.

---

# Финальная Quality Architecture

```text
                    CODE CHANGE
                         ↓
                ┌─────────────────┐
                │ STATIC QUALITY  │
                │ lint / types    │
                └────────┬────────┘
                         ↓
                ┌─────────────────┐
                │ FUNCTION TESTS  │
                │ unit/integration│
                └────────┬────────┘
                         ↓
             ┌────────────────────────┐
             │ UI QUALITY             │
             │ responsive             │
             │ accessibility          │
             │ browsers               │
             │ visual regression      │
             └───────────┬────────────┘
                         ↓
              ┌───────────────────────┐
              │ ADMISSIONS DATA QA    │
              │ source                │
              │ provenance            │
              │ schema                │
              │ semantics             │
              │ personalization       │
              └───────────┬───────────┘
                          ↓
                       STAGING
                          ↓
                     SMOKE TESTS
                          ↓
                      PRODUCTION
```

## Итог

**Strong — я бы зафиксировал именно этот стандарт.**

Самые важные решения:

1. **WCAG 2.2 AA обязательно.**
    
2. **320px → 1920px responsive без разрушения UX.**
    
3. **Chrome + Safari + Firefox + Edge + iOS + Android.**
    
4. **Playwright Chromium/Firefox/WebKit, но реальный Safari тоже проверяем.**
    
5. **Unit tests только для значимой логики, не ради coverage %.**
    
6. **Critical flows имеют E2E.**
    
7. **Critical admissions fact без official source не существует для Ekho.**
    
8. **Каждый critical fact хранит provenance + cycle + context + verification date.**
    
9. **Official-source conflict → `Conflict`, а не угадывание.**
    
10. **Missing information → `Unknown`.**
    
11. **Program-specific context нельзя смешивать с university-wide requirements.**
    
12. **Персонализированный вывод должен быть воспроизводим из source fact + profile + rule version.**
    
13. **Ошибочный deadline/eligibility/aid policy = release blocker.**
    
14. **Codex не имеет права «починить тест», просто изменив expected result.**
    

Именно **data correctness + provenance** здесь может стать одной из реальных защит Ekho от AdmissionsOS/Google Sheets/ChatGPT: пользователь видит не просто ответ, а **актуальный ответ + откуда он взят + когда проверен**.