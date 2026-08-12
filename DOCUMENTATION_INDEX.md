# Ekho Documentation Index

## Precedence

When documents conflict, apply this order:

1. explicit newer locked decision;
2. domain-specific standard;
3. `main info/STACK.md` for stack and infrastructure;
4. `main info/PROJECT CONTEXT.md` for product direction;
5. older or general notes.

Do not infer a new architectural decision from a lower-precedence document.

## Categories and authorities

| Area | Documents | Authoritative document |
| --- | --- | --- |
| Product direction, scope and strategy | `main info/PROJECT CONTEXT.md`, `main info/ROADMAP.md`, `main info/FUNCTIONAL.md` | `main info/PROJECT CONTEXT.md` |
| Technical stack and infrastructure | `main info/STACK.md`, `tech/Development Workflow.md`, `tech/Launch Architecture.md`, `tech/Scaling Rules.md`, `tech/Performance Budget.md` | `main info/STACK.md` unless a newer explicit locked decision applies |
| Data model and ingestion | `tech/Data Standard v1.md`, `tech/Data Architecture v1.md`, `tech/Data Pipeline.md`, `tech/Import & Ingestion Specification v1.md` | `tech/Data Standard v1.md` for canonical data semantics; `tech/Data Architecture v1.md` for schema architecture |
| API and security | `tech/API & Error Contract.md`, `tech/Security & Privacy.md`, `tech/Auth & Account Lifecycle v1.md` | Domain-specific standard for the relevant boundary |
| Search and user flows | `tech/Search & Filtering System.md`, `tech/Core User Flows standard.md`, `tech/Information Architecture.md` | Domain-specific standard for the relevant behavior |
| Operations and reliability | `tech/Observability, SLO & Incident Response.md`, `tech/Failure, Recovery & Degraded Mode.md`, `tech/Admin & Data Operations.md` | Domain-specific standard for the relevant operation |
| Quality and platform behavior | `tech/Quality & Testing Standard.md`, `tech/Internationalization & Localization Standard.md`, `tech/Feature Flags & Runtime Configuration.md`, `tech/SEO & Acquisition Architecture.md` | Domain-specific standard for the relevant concern |
| Hypotheses and working notes | `12-15 aug/` | Non-authoritative unless explicitly promoted by a newer locked decision |

## Technical source of truth

- `main info/STACK.md` — runtime, hosting, database, storage, auth, cache, ORM and initial search baseline.
- `tech/Data Standard v1.md` — canonical admissions-data semantics.
- `tech/Data Architecture v1.md` — data entities and persistence architecture.
- `tech/API & Error Contract.md` — API/server boundaries and API security.
- `tech/Security & Privacy.md` — security, privacy, RLS, private files and secrets.
- `tech/Launch Architecture.md` — environments, releases, launch gates and operational rollout.
- `tech/Scaling Rules.md` — measured triggers for adding infrastructure.
