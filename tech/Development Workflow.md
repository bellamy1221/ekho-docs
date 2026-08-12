# Development Workflow — Ekho v1

**Цель:** Codex должен уметь безопасно менять Ekho практически без твоего ручного кодинга: одна задача → одна ветка → автоматические проверки → staging → твоя проверка → production.

## 1. GitHub repository

**Решение: один основной repository.**

```text
ekho/
├─ src/
├─ public/
├─ tests/
├─ db/
│  └─ migrations/
├─ .github/
│  └─ workflows/
├─ docs/
├─ AGENTS.md
├─ package.json
└─ lockfile
```

Не дробим сейчас frontend/backend/data на отдельные repos без реальной необходимости.

---

# 2. Branch model

Используем простой **GitHub Flow / trunk-based workflow**.

```text
main
  ├─ feat/university-search
  ├─ feat/application-timeline
  ├─ fix/deadline-display
  ├─ data/uk-deadlines
  ├─ infra/ci-migrations
  └─ hotfix/auth-login
```

**Единственная постоянная ветка — `main`.**

Не создаём:

```text
develop
development
production
release
staging
```

GitHub сам рекомендует отдельную ветку для каждого независимого изменения, PR и удаление ветки после merge. ([GitHub Docs](https://docs.github.com/en/get-started/using-github/github-flow "GitHub flow - GitHub Docs"))

### Naming

```text
feat/*
fix/*
data/*
infra/*
chore/*
hotfix/*
```

**Правило:** одна задача = одна branch = один PR.

---

# 3. `main`

`main` = **source of truth**.

Но:

> `main` ≠ мгновенный production deploy.

Flow:

```text
feature branch
      ↓
Pull Request
      ↓
CI
      ↓
merge main
      ↓
STAGING
      ↓
smoke tests
      ↓
manual approval
      ↓
PRODUCTION
```

Это позволяет откатить релиз до того, как пользователь его увидит.

---

# 4. GitHub protection

На `main` включить Ruleset.

### Запретить

- direct push;
    
- force push;
    
- deletion;
    
- merge при failed CI;
    
- merge unresolved PR conversations.
    

### Require

- Pull Request;
    
- required status checks;
    
- up-to-date branch;
    
- linear history.
    

GitHub Rulesets/branch protection поддерживают PR requirement, required checks, conversation resolution и linear history. ([GitHub Docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule?utm_source=chatgpt.com "Managing a branch protection rule"))

### Пока Ekho делаешь один

Не требуем второго human approval.

Ты сам:

```text
Codex → PR → checks → review diff → merge
```

Когда появится второй разработчик:

```text
minimum approvals = 1
```

---

# 5. Merge strategy

Только:

**Squash and merge.**

Пример:

```text
feat: add personalized university requirements
```

а не 17 коммитов:

```text
fix
fix again
test
oops
final
final2
```

После merge branch автоматически удаляется.

---

# 6. Environments

Нужны **3 логических environment**.

### Local

```text
localhost
local/dev DB
development keys
```

Используется Codex и локальной разработкой.

### Staging

```text
staging.ekho.club
staging DB
staging auth
staging APIs
staging analytics
```

Максимально похож на production, но:

**никаких production user secrets / production PII.**

### Production

```text
ekho.club
production DB
production API keys
production auth
production monitoring
```

Все credentials физически разделены.

GitHub environments позволяют иметь отдельные secrets и protection rules для разных deployment environments. ([GitHub Docs](https://docs.github.com/actions/deployment/targeting-different-environments/using-environments-for-deployment?utm_source=chatgpt.com "Managing environments for deployment"))

---

# 7. Preview deployments

Каждый PR получает временный deployment:

```text
feat/university-search

→ unique preview URL
```

Ты можешь открыть его и визуально проверить изменение до merge.

Для Ekho каждый PR должен получать изолированный preview deployment через Cloudflare Workers / OpenNext, отдельно от production и без production secrets.

### Поэтому для Ekho v1

**Cloudflare Workers / OpenNext — основной deployment layer.**

---

# 8. Deployment model

Я бы зафиксировал такой.

### PR

```text
branch
 ↓
CI
 ↓
Preview
```

Production не затрагивается.

### Merge в `main`

```text
main
 ↓
build
 ↓
migration check
 ↓
deploy STAGING
 ↓
smoke tests
```

### Затем

ты нажимаешь:

```text
Promote to Production
```

И production получает **тот же проверенный commit/build**, а не новую сборку с неизвестными отличиями.

Production должен собираться и деплоиться из того же проверенного commit, без новых изменений между staging и production.

---

# 9. Production approval

Поначалу **только manual production promotion**.

Не:

```text
merge → instant production
```

А:

```text
merge
↓
staging
↓
ты смотришь
↓
production
```

Когда Ekho станет стабильным, безопасные изменения можно автоматизировать.

Но migrations/auth/payments/data pipeline я бы оставлял protected.

---

# 10. CI

GitHub Actions.

Каждый PR запускает:

```text
1. install
2. lint
3. typecheck
4. tests
5. production build
6. migration validation if DB changed
```

Required checks должны пройти до merge — GitHub поддерживает это как branch protection. ([GitHub Docs](https://docs.github.com/en/pull-requests/reference/status-checks?utm_source=chatgpt.com "Status checks"))

### Минимальный набор

```text
lint
typecheck
test
build
migration-check
```

Не добавляем 25 бесполезных CI jobs.

---

# 11. Tests

Codex запускает **только релевантные tests для своей задачи** во время разработки.

Например:

```text
University Search change

→ search tests
→ relevant API tests
→ typecheck
```

а не весь repository после изменения одного компонента.

Но перед merge CI проверяет минимальный общий safety suite.

Для критических flows позже:

```text
signup/login
add university
create application
requirements loading
deadline display
```

добавляются E2E smoke tests.

---

# 12. CI concurrency

Старый CI run должен отменяться, если в тот же PR пришёл новый commit.

```text
commit A → CI
commit B → cancel A → CI B
```

Но production deployments:

```text
one deployment at a time
```

GitHub Actions поддерживает concurrency именно для предотвращения параллельных конфликтующих workflows/deployments. ([GitHub Docs](https://docs.github.com/en/actions/concepts/workflows-and-actions/concurrency?utm_source=chatgpt.com "Concurrency"))

Это ещё и экономит CI usage.

---

# 13. Database migrations

Это один из самых строгих разделов.

**Schema никогда не меняется вручную в production.**

Нельзя:

```text
open production DB
ALTER TABLE manually
```

Все изменения:

```text
code
+
versioned migration
```

например:

```text
db/migrations/

001_initial.sql
002_university_programs.sql
003_application_round.sql
004_financial_aid.sql
```

Конкретный migration engine закрепим вместе с database stack; сам workflow от ORM не зависит.

---

# 14. Migration lifecycle

```text
Codex creates migration
↓
local DB
↓
CI temporary/test DB
↓
staging
↓
verify
↓
production
```

Codex **никогда не создаёт migration непосредственно на production**.

---

# 15. Migration rules

После применения migration:

**никогда не редактировать её.**

Если ошибка:

```text
004_bad_change.sql
005_fix_bad_change.sql
```

а не менять `004`.

История БД должна быть воспроизводимой.

---

# 16. Destructive migrations

Особенно важно после появления пользователей.

Не делать одним release:

```text
DROP COLUMN
rename critical field
change type destructively
```

Используем **expand → migrate → contract**.

Например:

### Release A

```text
add new column
```

### Release B

```text
application reads/writes new column
```

### Migration/backfill

```text
old data → new column
```

### Later

```text
remove old column
```

Это позволяет откатить приложение без уничтожения совместимости.

PostgreSQL отдельно предупреждает, что некоторые `ALTER TABLE` операции могут брать locks или переписывать таблицы, поэтому потенциально тяжёлые schema changes нельзя рассматривать как безобидный deploy. ([PostgreSQL](https://www.postgresql.org/docs/current/sql-altertable.html?utm_source=chatgpt.com "Documentation: 18: ALTER TABLE"))

---

# 17. Migration transactions

Где операция поддерживает transaction:

```text
BEGIN
migration
COMMIT
```

При ошибке:

```text
ROLLBACK
```

PostgreSQL transactions дают all-or-nothing поведение для поддерживаемых операций. ([PostgreSQL](https://www.postgresql.org/docs/current/tutorial-transactions.html?utm_source=chatgpt.com "Documentation: 18: 3.4. Transactions"))

Но Codex не должен слепо считать, что **любая** DDL операция безопасна внутри одной transaction.

---

# 18. Production rollback

Для application:

```text
bad deployment
↓
redeploy previous known-good deployment
```

Не:

```text
Codex срочно правит production вручную
```

### Для DB

Предпочитаем:

**roll-forward migration**, а не автоматический destructive rollback.

Именно поэтому новые версии приложения и schema должны некоторое время оставаться backward-compatible.

---

# 19. Environment variables

Никаких secret keys в Git.

```text
.env
.env.local
.env.production
```

не коммитятся.

Next.js рекомендует держать `.env.*` вне repository, а `NEXT_PUBLIC_*` использовать только для действительно публичных значений. ([nextjs.org](https://nextjs.org/docs/app/guides/production-checklist?utm_source=chatgpt.com "Guides: Production"))

Пример:

```text
DATABASE_URL
AUTH_SECRET
OPENAI_API_KEY
CRAWLER_KEY
SENTRY_DSN
```

хранятся в environment secret stores.

---

# 20. Codex — главный development workflow

Для Ekho:

```text
Ты
 ↓
task/spec
 ↓
Codex
 ↓
branch/worktree
 ↓
implementation
 ↓
tests
 ↓
self-review
 ↓
PR
 ↓
CI
 ↓
ты проверяешь
```

OpenAI Codex поддерживает isolated worktrees для параллельной работы агентов, чтобы они не ломали локальное состояние друг друга. ([OpenAI](https://openai.com/index/introducing-the-codex-app/?utm_source=chatgpt.com "Introducing the Codex app"))

---

# 21. `AGENTS.md`

В root обязательно:

```text
/AGENTS.md
```

Codex автоматически читает `AGENTS.md` перед работой и позволяет добавлять directory-specific инструкции ниже по дереву repository. ([OpenAI Developers](https://developers.openai.com/codex/agent-configuration/agents-md "Custom instructions with AGENTS.md | ChatGPT Learn"))

Там фиксируем постоянные правила Ekho:

```text
architecture
code conventions
data safety
migration rules
testing rules
UI constraints
dependencies policy
security
Codex workflow
code review rules
```

Это будет **конституция Codex для Ekho**.

---

# 22. Nested AGENTS.md

Не плодим их сразу.

Позже только там, где правила действительно отличаются:

```text
AGENTS.md

src/data/AGENTS.md
src/auth/AGENTS.md
db/AGENTS.md
```

Например `db/AGENTS.md`:

```text
Never modify production data manually.
Never edit an applied migration.
Never create destructive migration without explicit approval.
```

OpenAI рекомендует scoped repository rules располагать рядом с кодом, к которому они относятся, вместо огромного глобального списка. ([OpenAI Developers](https://developers.openai.com/blog/custom-code-review-rules-for-codex "Custom Code Review rules for Codex | OpenAI Developers"))

---

# 23. Codex task format

Каждая нормальная задача задаётся четырьмя вещами:

```text
Goal
Context
Constraints
Done when
```

Это прямо соответствует текущим рекомендациям OpenAI для Codex. ([OpenAI Developers](https://developers.openai.com/codex/learn/best-practices "Best practices | ChatGPT Learn"))

Например:

```text
Goal:
Add deadline filtering to university search.

Context:
src/...
existing university filters...

Constraints:
Do not redesign the page.
Do not change unrelated components.
Do not add dependencies unless necessary.

Done when:
Filtering works,
types pass,
relevant tests pass,
production build succeeds.
```

---

# 24. Codex scope rule

Codex запрещено автоматически делать:

```text
"while I'm here I refactored..."
"I redesigned..."
"I replaced the library..."
"I cleaned the whole architecture..."
```

Если задача:

```text
fix deadline sorting
```

он исправляет **deadline sorting**.

Не весь Search.

Это особенно важно для сохранения контроля над кодовой базой.

---

# 25. Codex planning

Мелкое изменение:

```text
short plan
→ execute
```

Сложная новая система:

```text
ExecPlan
→ implementation
```

Не заставляем Codex писать огромный planning document для каждой кнопки.

OpenAI отдельно рекомендует `PLANS.md` / ExecPlans для сложных длительных изменений, а не как обязательный overhead для каждой задачи. ([OpenAI Developers](https://developers.openai.com/cookbook/articles/codex_exec_plans "Using PLANS.md for multi-hour problem solving"))

---

# 26. Codex dependencies

По умолчанию:

```text
do not add production dependency
```

если проблема решается существующим stack.

Если dependency реально нужна, Codex сообщает:

```text
package
reason
alternatives
impact
```

до её добавления.

---

# 27. Codex MCP

MCP **не включаем «на всякий случай»**.

Используем только когда задача реально требует внешнего контекста.

Это уменьшает context/noise и расход лимитов.

---

# 28. Codex completion report

После задачи Codex должен вернуть максимум:

```text
Done

Changed:
- file A
- file B

Verified:
- typecheck
- relevant tests
- build

Migration:
- none / xxx

Dependencies:
- none / xxx

Risks:
- none / short note
```

Не писать огромные эссе о том, что он сделал.

---

# 29. Codex review

Перед PR:

```text
git diff
↓
Codex self-review
↓
tests
↓
PR
```

Особое внимание:

```text
breaking changes
security
PII
auth
database
source/data integrity
API compatibility
```

Codex Code Review сейчас поддерживает custom review rules прямо из `AGENTS.md`, но OpenAI подчёркивает: это дополнительный reviewer, а не замена CI и branch protection. ([OpenAI Developers](https://developers.openai.com/blog/custom-code-review-rules-for-codex "Custom Code Review rules for Codex | OpenAI Developers"))

---

# 30. Hotfix workflow

Production bug:

```text
main
 ↓
hotfix/login-loop
 ↓
minimal fix
 ↓
relevant tests
 ↓
PR
 ↓
CI
 ↓
staging
 ↓
production
```

Даже hotfix не должен превращаться в:

```text
edit production directly
```

Если возможно просто rollback:

**rollback лучше нового срочного кода.**

---

# 31. Что специально НЕ делаем

Для Ekho v1 не нужны:

- GitFlow;
    
- `develop`;
    
- release branches;
    
- Kubernetes;
    
- Jenkins;
    
- отдельный DevOps repository;
    
- десятки GitHub Actions;
    
- production deploy каждого commit;
    
- manual DB editing;
    
- огромный integration test suite;
    
- обязательные multi-agent Codex tasks;
    
- repo-wide Codex analysis для мелких задач.
    

Это сейчас только увеличит вероятность ошибок.

---

# 32. Когда усложнять

### Сейчас

```text
1 developer
Codex
main
feature branches
PR
CI
staging
manual production
```

### Когда появятся 2–5 developers

добавить:

```text
1 required reviewer
CODEOWNERS
stronger branch rules
```

### Когда PR много одновременно

добавить:

```text
merge queue
parallel CI
isolated preview databases
```

### Когда schema changes частые

добавить:

```text
ephemeral DB branch per PR
automated migration compatibility testing
```

Не раньше.

---

# Финальная архитектура

```text
                    ┌─────────────┐
                    │    TASK     │
                    └──────┬──────┘
                           ↓
                    ┌─────────────┐
                    │    CODEX    │
                    └──────┬──────┘
                           ↓
                  feature / fix branch
                           ↓
                    Pull Request
                           ↓
        ┌─────────────────────────────┐
        │ lint                        │
        │ typecheck                   │
        │ relevant tests              │
        │ build                       │
        │ migration validation        │
        └──────────────┬──────────────┘
                       ↓
                    Preview
                       ↓
                     MAIN
                       ↓
                    STAGING
                       ↓
                  Smoke tests
                       ↓
                 Manual approval
                       ↓
                  PRODUCTION
```

## Итоговое решение

**Strong. Зафиксировал бы именно это как Development Workflow v1 для Ekho.**

Ключевые решения:

1. **Один repo.**
    
2. **`main` — единственная long-lived branch.**
    
3. **Одна задача = одна branch/PR.**
    
4. **Никакого `develop`.**
    
5. **PR Preview → main → staging → production.**
    
6. **Production только после твоего approval.**
    
7. **Schema только через versioned migrations.**
    
8. **Никаких ручных production DB changes.**
    
9. **CI блокирует плохой merge.**
    
10. **`AGENTS.md` становится постоянной инструкцией Codex.**
    
11. **Codex работает узко по задаче и запускает только релевантные проверки.**
    
12. **Никаких ненужных refactor/dependencies/MCP.**
    
13. **Rollback application должен быть простым; DB changes проектируем backward-compatible.**
    
14. **Усложняем workflow только когда появляется реальная нагрузка/команда.**
    

Это уже достаточно конкретно, чтобы потом превратить почти без переосмысления в **`AGENTS.md` + GitHub Ruleset + Actions + Codex instructions**.
