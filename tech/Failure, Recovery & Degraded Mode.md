# Failure, Recovery & Degraded Mode v1
**Status:** LOCKED
**Scope:** dependency failure, graceful degradation, backups, disaster recovery, data restoration, retries, read-only operation and recovery procedures
**Stack:** Next.js + Supabase/PostgreSQL/Auth/Storage + managed deployment platform
**Depends on:** Security & Privacy, Data Architecture, Data Pipeline, Auth & Account Lifecycle, Import & Ingestion, Admin & Data Operations, Observability/SLO/Incident Response, API & Error Contract, Feature Flags & Runtime Configuration
**Research verified:** 12 August 2026
---
# 1. Goal
Ekho must not treat every dependency failure as:
> whole product is down.
The system must preserve the safest useful functionality possible.
Desired behavior:
```text
Dependency fails
↓
Detect
↓
Contain blast radius
↓
Degrade only affected capability
↓
Protect data integrity
↓
Recover safely
↓
Verify
↓
Gradually restore normal operation
```
AWS and Google reliability guidance both recommend graceful degradation so a dependency failure does not unnecessarily destroy unrelated functionality.
---
# 2. Core principle
Priority order during failure:
1. **confidentiality and data integrity**
2. **prevent new corruption**
3. **preserve critical user access**
4. **preserve critical writes where safe**
5. **preserve noncritical functionality**
6. **recover normal functionality**
Availability must never be preserved by sacrificing authorization or factual integrity.
---
# 3. Availability vs integrity
Example:
```text
Database integrity uncertain
```
Correct:
```text
disable writes
→ continue verified safe reads where possible
```
Wrong:
```text
keep accepting writes
because downtime looks bad
```
For Ekho:
> **A temporary read-only product is better than writable corrupted data.**
---
# 4. Failure states
Ekho supports four operational states:
```text
NORMAL
DEGRADED
READ_ONLY
RECOVERY
```
These are system operational states, not user/application statuses.
---
# 5. `NORMAL`
All critical systems operate normally.
```text
reads = enabled
writes = enabled
background jobs = enabled
external side effects = enabled
```
---
# 6. `DEGRADED`
One or more capabilities unavailable or restricted.
Example:
```text
AI unavailable
but
universities + applications + requirements still work
```
Degraded mode must preserve unrelated functionality.
AWS describes this as transforming hard dependencies into softer dependencies where the application's core function can continue despite downstream failures.
---
# 7. `READ_ONLY`
Use when:
```text
reads considered safe
but
writes could create risk
```
Possible triggers:
* database integrity uncertainty;
* migration incident;
* restore/cutover;
* severe write-path bug;
* uncontrolled duplicate mutations.
User-facing writes must fail explicitly rather than pretending to save.
---
# 8. `RECOVERY`
System is actively being restored/reconciled.
During recovery:
* dangerous background jobs paused;
* public safe reads may continue;
* writes may remain disabled;
* queues remain controlled;
* operators verify restored state.
`RECOVERY` ends only after verification.
---
# 9. Maintenance page is last resort
Do not make:
```text
any dependency failure
→ maintenance page
```
The whole product should become unavailable only when:
* no safe useful functionality remains;
* security requires shutdown;
* recovery cannot safely happen while serving traffic.
Graceful degradation is preferable to total failure where meaningful functionality can remain safe.
---
# 10. Failure-domain inventory
Ekho must explicitly consider failure of:
```text
Deployment/frontend
Supabase database
Supabase Auth
Cloudflare R2
Google OAuth
Apple OAuth
SMTP/email provider
Search
AI provider
Feature flag provider
Data Pipeline
Import system
Notification system
Background jobs
Observability provider
DNS/domain
External university websites
```
Each dependency requires known fallback behavior.
Google Cloud's current reliability framework recommends identifying dependencies and defining how systems behave when those dependencies fail rather than treating failure handling as an afterthought.
---
# 11. Hard vs soft dependency
Every dependency must be classified.
### Hard dependency
Operation fundamentally cannot complete safely without it.
Example:
```text
saving application
→ canonical database
```
### Soft dependency
Useful but operation can still provide meaningful value without it.
Example:
```text
university page
→ AI explanation
```
AI must be soft.
---
# 12. Soft dependency invariant
Failure of:
```text
AI
analytics
recommendation ranking
email notification
source crawler
```
must not make core university/application data inaccessible.
---
# 13. Recovery objectives
Use:
### RTO — Recovery Time Objective
Maximum target time to restore a capability after disruption.
### RPO — Recovery Point Objective
Maximum target amount of data history that may need to be lost/recovered from an earlier point.
These are standard disaster-recovery concepts used by NIST and major cloud architecture guidance.
---
# 14. RTO/RPO are internal targets
They are:
```text
engineering/recovery objectives
```
not:
```text
public SLA
```
Do not publish recovery guarantees until they are repeatedly demonstrated by recovery tests.
---
# 15. Public-production recovery targets
Once Ekho stores real applicant data at meaningful scale:
| Asset                          |                             RPO target | RTO target |
| ------------------------------ | -------------------------------------: | ---------: |
| PostgreSQL canonical/user data |                                ≤15 min |       ≤4 h |
| Auth/user identity             |                    tied to DB recovery |       ≤4 h |
| Private documents              |                ≤15 min backup-copy lag |       ≤8 h |
| Public institutional data      | ≤15 min canonical DB / cached fallback |       ≤4 h |
| Search/cache                   |                            rebuildable |       ≤8 h |
| Notifications                  |                        queue preserved |      ≤12 h |
| Data Pipeline                  |                ≤24 h operational state |      ≤24 h |
| Admin/import tooling           |                                  ≤24 h |      ≤24 h |
| Analytics                      |                        acceptable loss |      ≤72 h |
These values are **Ekho internal targets**, chosen according to product criticality.
---
# 16. Pre-launch exception
During local development/closed testing, production-level RPO infrastructure does not need to be purchased prematurely.
But before students depend on Ekho for:
* active applications;
* deadlines;
* uploaded documents;
production recovery mechanisms must meet the public-production target.
---
# 17. Database backup baseline
Supabase currently provides managed daily database backups and supports Point-in-Time Recovery on eligible paid projects. PITR allows restoration to a chosen point with very fine time granularity, whereas daily backups alone can imply substantially greater possible data loss.
Therefore meaningful production Ekho should use:
```text
Supabase managed backup
+
PITR or equivalent recovery mechanism
+
independent Ekho-controlled backup
```
---
# 18. PITR requirement
Once Ekho holds important live application state:
**enable PITR or another mechanism capable of satisfying the ≤15-minute database RPO.**
Do not merely write:
```text
RPO = 15 min
```
if the actual backup system can recover only yesterday's database.
---
# 19. PITR does not remove need for independent backup
Provider-managed recovery remains inside the same infrastructure/account boundary.
Therefore maintain an independent Ekho-controlled backup copy.
CISA recommends encrypted recovery copies separated from normal production infrastructure and regular restoration testing.
---
# 20. Independent database backup
Production baseline:
```text
daily logical PostgreSQL backup
→ encrypted
→ separate storage account/provider
```
Supabase documents that logical dumps remain possible through CLI/`pg_dump` even when its managed backup system uses physical backups/PITR.
---
# 21. Independent backup credentials
Backup destination must not share an unrestricted credential with the production application.
Use:
```text
separate credentials
least privilege
restricted backup destination
```
Compromising production application credentials should not automatically provide permission to destroy all recovery copies.
---
# 22. Backup immutability
Where available, important recovery copies should use:
```text
immutable / write-once retention
```
or similarly protected storage.
CISA and NIST guidance recommend isolated/immutable recovery copies so an attacker or destructive event cannot modify primary data and backup copies simultaneously.
---
# 23. Database backup retention
Initial Ekho technical baseline:
```text
daily independent copies
→ ~35 days
```
Exact retention will later be reconciled with:
**Legal / Compliance Operations**
especially:
* account deletion;
* GDPR erasure;
* retention obligations.
Do not keep personal-data backups forever "just in case."
---
# 24. Backup ≠ replication
Supabase replication synchronizes database changes to another destination.
Therefore, as an architectural inference:
> replication may improve availability/data distribution, but it is not Ekho's sole backup strategy because unwanted changes can also be propagated.
Maintain point-in-time/historical recovery copies separately.
---
# 25. Storage backup warning
Supabase database backups **do not include the actual files stored in Cloudflare R2**.
This is critical for Ekho because private applicant documents may live in Storage.
---
# 26. Document durability architecture
Before Ekho treats document storage as durable:
```text
user upload
↓
Cloudflare R2
↓
upload confirmed
↓
independent encrypted backup job
↓
backup confirmed
```
Track conceptual state:
```text
primary_stored
backup_pending
backed_up
backup_failed
```
---
# 27. Document backup target
Completed user uploads should reach the independent backup copy within:
```text
≤15 minutes
```
under normal production operation.
This is an Ekho RPO decision.
---
# 28. Document reconciliation
Run a periodic reconciliation job comparing:
```text
canonical storage metadata
vs
primary Storage objects
vs
backup object inventory
```
This catches backup jobs that were silently lost.
---
# 29. Nightly storage audit
At minimum:
```text
daily
```
verify:
* missing backups;
* unexpected orphan objects;
* backup failures;
* size/hash mismatches where feasible.
---
# 30. Never claim document durability from database backup
Because database backup can contain Storage metadata while the underlying file object itself is absent after restore. Supabase explicitly documents this distinction during backup restoration.
---
# 31. Backup verification
Backup creation is not sufficient.
Verify periodically:
```text
backup exists
can decrypt
can be read
schema is valid
important tables exist
sample rows match
```
NIST CSF 2.0 explicitly includes creating, protecting, maintaining **and testing** backups.
---
# 32. Restore testing
A successful backup job does not prove recoverability.
Google Cloud reliability guidance recommends testing actual restoration and validating the entire application stack with restored data, not merely checking that a backup file exists.
---
# 33. Ekho recovery-test cadence
Production baseline:
### Monthly
```text
backup existence/integrity validation
```
### Quarterly
```text
full database restore into isolated environment
+
application smoke tests
```
### Before major public launch
```text
full disaster-recovery drill
```
### After major architecture/storage changes
repeat the applicable recovery drill.
These cadences are Ekho operating policy.
---
# 34. Recovery tests must measure
Every drill records:
```text
start time
restore point
actual data loss / achieved RPO
restore completion
actual RTO
failures encountered
manual steps
verification result
```
Recovery objectives that have never been measured are assumptions.
---
# 35. Restore into isolated environment
Preferred recovery testing:
```text
production backup
→ isolated recovery project
→ verify
```
not:
```text
restore production just to test backup
```
Supabase currently supports restoring eligible physical/PITR backups into a new project, which allows recovery data to be inspected without immediately replacing the original project.
---
# 36. In-place restore warning
Supabase documents that restoring a database backup to the project makes the project inaccessible during restoration, and restoration duration depends partly on database size.
Therefore:
> never begin an in-place production restore casually.
It is an incident/recovery operation.
---
# 37. Recovery-point selection
When corruption begins at:
```text
14:37
```
do not automatically restore:
```text
latest possible backup
```
Select the last known safe point **before corruption**.
Record:
```text
incident timestamp
selected recovery point
reason
```
---
# 38. Data-integrity incident
If database corruption is suspected:
```text
pause risky writes
pause imports
pause automated publications
pause outbound change notifications
preserve evidence
determine corruption boundary
```
Only then choose repair/restore.
---
# 39. Never restore blindly
A restore candidate must be verified before normal traffic resumes.
Verify:
```text
schema
migrations
constraints
user ownership
RLS
critical row counts
sample applications
critical institutional records
Auth state
Storage references
```
NIST recovery guidance emphasizes restoring operations from trusted/clean recovery assets and validating recovery rather than treating restore as a simple file-copy operation.
---
# 40. Database recovery sequence
Preferred severe-incident workflow:
```text
1. Declare incident
2. Stop dangerous writes/jobs
3. Identify last safe recovery point
4. Create isolated restored environment
5. Verify database integrity
6. Restore/match Storage
7. Apply required current configuration
8. Verify Auth/security
9. Run critical smoke tests
10. Prepare cutover
11. Re-enable reads
12. Re-enable writes
13. Replay safe backlog
14. Monitor
```
---
# 41. Restore configuration separately
Database recovery does not automatically guarantee recovery of:
```text
deployment configuration
OAuth provider setup
SMTP provider setup
feature flags
DNS
secrets
external integrations
```
Maintain a recovery inventory for those separately.
---
# 42. Recovery asset inventory
Maintain one document/manifest identifying:
```text
Postgres
Storage
application repository
database migrations
deployment config
environment variables
secret locations
OAuth configuration
SMTP configuration
DNS/domain
feature flags
runtime configuration
third-party dependencies
```
NIST contingency/recovery guidance recommends documenting recovery resources, relationships, procedures and testing.
---
# 43. Configuration-as-code preference
Where practical keep recoverable configuration in version control:
```text
database migrations
schema
RLS policies
application code
configuration schema
flag definitions
```
Do not depend on remembering what was manually clicked six months earlier.
---
# 44. Secrets recovery
Do not back up secrets into plaintext Git or a normal recovery document.
Critical credentials need a secure recovery path through:
```text
secret manager
provider recovery mechanism
secure recovery codes
```
OWASP stresses that cryptographic key/secret loss can make protected data unrecoverable and therefore secure key recovery/backup must be considered where applicable.
---
# 45. Platform account recovery
Production must not depend on:
```text
one laptop
+
one browser session
```
for access to:
* Supabase;
* GitHub;
* Cloudflare/deployment provider;
* DNS/domain registrar.
Maintain secure MFA recovery methods.
Supabase's production checklist also recommends protecting administrative accounts with MFA and avoiding a single inaccessible owner as organizations grow.
---
# 46. Auth state after database restore
A historical database restore may also affect historical Auth/session state because Supabase Auth is backed by project data.
Therefore after major DB rollback:
```text
Auth/session state
```
must receive explicit security review before reopening normal operation.
---
# 47. Session safety after major restore
If there is a realistic risk that previously revoked/compromised authentication state was restored:
require:
```text
forced reauthentication and/or
supported signing/session revocation procedure
```
before returning to normal operation.
Supabase's current signing-key system supports key rotation/revocation; it also notes that already issued access tokens can remain accepted until expiration unless the relevant key is revoked.
---
# 48. Security incident restore
After a security compromise:
do not restore an old database and assume incident is finished.
Also consider:
```text
rotate compromised credentials
revoke tokens
patch vulnerability
verify access controls
review logs
```
CISA and NIST recovery guidance treat restoration as one part of recovering from a cyber incident rather than the whole response.
---
# 49. Deployment failure
If a new deployment causes severe regression:
preferred mitigation:
```text
rollback deployment
```
before trying to debug it live while users remain affected.
---
# 50. Cloudflare Workers rollback
Cloudflare Workers production deployments must be rolled back to a previously served stable Worker version.
Document:
```text
last known good deployment
rollback procedure
post-rollback verification
```
---
# 51. Code rollback ≠ database rollback
Critical invariant:
```text
frontend/server rollback
≠
database migration rollback
```
A previous application version may not work against a destructive newer schema.
Therefore migrations must follow backward-compatible rollout rules from Development Workflow.
---
# 52. Safe migration sequence
Prefer:
```text
add compatible schema
↓
deploy compatible code
↓
migrate/use new data
↓
verify
↓
remove old behavior later
```
Not:
```text
delete old schema
↓
deploy
↓
hope
```
---
# 53. No destructive schema change before rollback window closes
If old deployment may still need to run:
do not remove database structures that old deployment requires.
Feature flags do not make incompatible schema changes safe.
---
# 54. Retry principle
Retry only failures likely to be transient.
AWS reliability guidance recommends:
```text
timeouts
+
bounded retries
+
exponential backoff
+
jitter
```
rather than immediate repeated calls.
---
# 55. Retry amplification
Retries increase downstream load.
If a dependency is already overloaded:
```text
uncontrolled retries
→ more load
→ more failures
→ more retries
```
Google SRE specifically identifies this as a source of cascading failure.
---
# 56. Ekho retry baseline
For ordinary transient machine-to-machine calls:
```text
max 2–3 attempts
exponential backoff
jitter
```
unless operation-specific requirements say otherwise.
Never infinite retry.
---
# 57. Mutation retries
Mutating operations may retry automatically only when:
```text
idempotent
OR
protected by idempotency key
```
according to API & Error Contract.
---
# 58. Timeout rule
Every external dependency call requires an explicit timeout.
AWS Well-Architected recommends client-side timeouts so a failed/slow dependency does not indefinitely hold upstream resources.
---
# 59. Circuit breaker
Use a circuit breaker where repeated calls to a failing downstream dependency would:
* waste resources;
* increase latency;
* worsen overload;
* repeatedly produce the same failure.
AWS specifically recommends circuit breakers as part of graceful dependency-failure handling.
---
# 60. Circuit-breaker behavior
Conceptually:
```text
CLOSED
normal requests
↓ repeated failure threshold
OPEN
fail fast / use fallback
↓ after cooldown
HALF_OPEN
small test request
↓
healthy → CLOSED
failed → OPEN
```
Do not use one universal threshold for every service.
---
# 61. Backpressure
When a downstream service cannot process current load:
slow or reject new noncritical work before queues become unbounded.
AWS recommends failing fast/limiting queues as part of resilient distributed-system interaction design.
---
# 62. Load shedding order
Under severe resource pressure disable/reduce in this order:
```text
1. analytics/background enrichment
2. AI features
3. bulk imports/reverification
4. nonurgent notifications
5. advanced search/recommendations
6. noncritical admin operations
```
Protect longest:
```text
authentication
critical reads
applications/tasks
canonical requirements
```
This is an Ekho product-priority policy.
---
# 63. Never shed integrity
Do not reduce load by:
```text
skipping authorization
skipping RLS
skipping transaction constraints
publishing unverified admissions data
```
Security/integrity checks are not optional degradation targets.
---
# 64. Database unavailable
If primary database is unavailable:
### Public institutional pages
May continue from a previously published safe cache/snapshot where available.
### Authenticated workspace
Show temporary unavailability unless its private state can be retrieved through an explicitly safe architecture.
### Writes
Disabled.
### Background jobs
Paused/backed off.
---
# 65. Cached public university data
Cached fallback is allowed only if snapshot was:
```text
previously canonical
published
source-grounded
```
and retains:
```text
last verified date
```
according to existing data-freshness rules.
Never generate replacement university facts during outage.
---
# 66. Stale public data
If cached institutional data exceeds acceptable freshness:
do not label it:
```text
up to date
```
Show clear freshness state or restrict the affected information.
Trust beats pretending continuity.
---
# 67. Personalized requirements during DB outage
If Requirements for Me cannot safely load:
do not derive requirements from incomplete browser data and pretend they are authoritative.
Show:
```text
Personalized requirements are temporarily unavailable.
```
Canonical public institutional requirements may still be shown separately where safe.
---
# 68. Auth service unavailable
If Supabase Auth fails:
### Existing sessions
May continue only when their identity/session can still be validated through the normal secure mechanism.
### New login/signup/recovery
May be unavailable.
Never:
```text
Auth down
→ skip authentication
```
---
# 69. One OAuth provider unavailable
Example:
```text
Google OAuth down
```
Ekho should still offer:
```text
Apple
email/password
```
where those providers remain healthy.
One social provider must not become the only account-access path.
---
# 70. All authentication unavailable
Public university research may remain available.
Authenticated/private functionality must remain protected and unavailable until identity can again be validated safely.
---
# 71. Storage unavailable
If Cloudflare R2 is unavailable:
keep working:
```text
university research
applications
deadlines
requirements
tasks
notes where DB-backed
```
Disable:
```text
upload
download/preview where object unavailable
```
Do not take entire Ekho offline.
---
# 72. Failed upload
Never show:
```text
Upload complete
```
until primary upload has actually succeeded.
Independent backup completion may happen asynchronously.
---
# 73. Upload backup failure
If:
```text
primary upload succeeds
backup mirror fails
```
user's primary file remains available.
Internally:
```text
backup_failed
→ retry
→ alert if persistent
```
Do not create duplicate user files.
---
# 74. Do not store private documents in browser recovery caches
Never persist document binary data or sensitive essay/document contents into:
```text
localStorage
```
merely to survive backend outages.
Recovery convenience does not override Security & Privacy.
---
# 75. Preserve unsaved safe form state
If ordinary write fails:
keep the user's current form state in memory where practical and show:
```text
Not saved
```
Do not:
```text
optimistically say saved
→ discard user input
```
Persistence of sensitive drafts outside normal server storage needs separate security review.
---
# 76. Search unavailable
University detail pages must remain accessible directly.
Preferred fallback:
```text
primary search unavailable
→ limited safe database/basic search
```
where load permits.
If that is also unavailable:
show search temporarily unavailable without breaking direct page access.
---
# 77. Search is derived
Search indexes are derived data.
Primary recovery strategy:
```text
canonical database
→ rebuild search
```
Do not treat the search index as the only copy of university data.
---
# 78. AI unavailable
Core Ekho must continue without AI.
Available:
```text
universities
applications
requirements
deadlines
costs
aid
tasks
documents
```
Potentially unavailable:
```text
AI explanation
AI-grounded Q&A
AI-assisted admin research
```
AI is infrastructure, not product identity.
---
# 79. AI fallback
Never:
```text
primary AI provider fails
→ use an unapproved model
→ publish unverified admissions facts
```
Fallback models may be used only where separately validated and still subject to source-grounding rules.
---
# 80. Email provider unavailable
Existing authenticated product use should continue.
Affected:
```text
email verification
password recovery
email-change confirmation
email notifications
```
In-app functionality unrelated to email remains available.
---
# 81. Auth email failure
Do not pretend:
```text
verification email sent
```
when provider returned definitive delivery failure.
Provide controlled retry.
Do not replay expired reset/verification links long after recovery.
---
# 82. Notification-provider outage
Queue eligible notifications rather than discarding them immediately.
Track:
```text
pending
sending
sent
failed
```
with idempotency/deduplication.
---
# 83. Notification backlog recovery
After provider recovery:
do **not** send every old notification blindly.
Re-evaluate:
```text
Is it still relevant?
Has newer event superseded it?
Has deadline already passed?
Has user already acted?
```
Then send only valid notifications.
---
# 84. Notification storm prevention
Backlog replay must be rate-controlled.
Use:
```text
bounded concurrency
backoff
jitter
deduplication
```
to avoid immediately overloading the recovered provider. Retry storms are a known cascading-failure pattern.
---
# 85. Data Pipeline unavailable
Public Ekho continues using existing canonical published data.
Pipeline failure means:
```text
freshness stops advancing
```
not:
```text
delete current data
```
---
# 86. Pipeline lag
Monitor:
```text
oldest pending job age
last successful source check
```
as already required by Observability.
If freshness objectives are exceeded:
create operational issue/incident according to severity.
---
# 87. Never guess during pipeline outage
If university sources cannot be checked:
do not update:
```text
deadline
test policy
tuition
financial aid
```
from memory/AI.
Keep last verified value with existing freshness state.
---
# 88. University website unavailable
One university's site failure must not remove Ekho's existing verified record.
Mark source:
```text
unreachable
needs_reverification
```
according to Data Pipeline/Admin rules.
---
# 89. Suspicious mass source changes
If pipeline suddenly detects:
```text
thousands of requirements changed
```
treat as potential:
```text
parser failure
site redesign
source error
```
not proof that all universities changed policy.
Stop downstream automatic candidate processing if necessary and require review.
---
# 90. Import system unavailable
Impact:
```text
new/admin data import delayed
```
Must not affect:
```text
student research
applications
requirements
existing canonical data
```
---
# 91. Import publication incident
If import publication creates data-integrity risk:
```text
disable new publications
```
while still permitting:
```text
review
validation
public read access
```
where safe.
Feature Flags/Runtime controls provide this lever.
---
# 92. Admin unavailable
Admin failure is not automatically a user outage.
However it becomes incident-relevant if it prevents urgent correction of:
```text
wrong deadline
wrong eligibility
security problem
```
---
# 93. Feature flag provider unavailable
Use:
```text
last known safe state
or
registered safe default
```
according to Feature Flags specification.
OpenFeature explicitly designs evaluation to return defined defaults when normal evaluation cannot succeed.
---
# 94. Observability unavailable
Ekho must continue operating.
Telemetry/export failure must not block user operations.
Independent external monitoring remains the fallback signal.
---
# 95. DNS/domain failure
Keep:
```text
registrar access
DNS configuration
domain ownership records
recovery procedure
```
in operational recovery inventory.
Do not make restoring `ekho.club` depend on one person's remembered DNS settings.
---
# 96. Background jobs
Every critical background job must define:
```text
idempotency
retry policy
maximum attempts
failure state
checkpoint/progress
recovery procedure
```
Never create:
```text
while true:
  retry()
```
---
# 97. Failed-job state
After bounded attempts:
```text
failed
```
or equivalent dead-letter state.
Do not retry forever invisibly.
---
# 98. Background job replay
After recovery:
```text
failed/pending backlog
→ validate still relevant
→ prioritize
→ replay gradually
```
Do not simply restart every historical job simultaneously.
---
# 99. Job priority after outage
Highest:
```text
deadline-sensitive user actions
critical data updates
security operations
```
Lower:
```text
analytics
historical enrichment
nonurgent reindexing
```
---
# 100. Queue recovery and overload
Google and AWS reliability guidance both warn that recovering services can be taken down again when a backlog/retry wave arrives simultaneously.
Therefore recovery traffic must be controlled.
---
# 101. Read-only implementation rule
Read-only mode cannot mean:
```text
hide Save buttons
```
only.
Write prevention must exist at a trusted server/database/API boundary.
---
# 102. Direct Supabase writes
Because Ekho intentionally allows some safe direct Supabase + RLS operations, emergency write shutdown must account for those paths too.
Possible implementation may use:
```text
database-level read-only control
and/or
central PostgREST pre-request/domain gate
```
rather than client UI alone.
Supabase itself supports database read-only operation and documents that writes fail while the database is in read-only state.
Exact mechanism must be tested before production reliance.
---
# 103. Do not casually toggle PostgreSQL global read-only
Database-level read-only is a powerful emergency/migration mechanism.
Changing it can affect:
```text
application
Auth
background jobs
internal services
```
Therefore it requires an explicit runbook.
---
# 104. Subsystem kill switches
Prefer granular control:
```text
application_writes_enabled
document_uploads_enabled
outbound_notifications_enabled
import_publication_enabled
live_update_processing_enabled
AI_features_enabled
```
over:
```text
everything_off
```
when the failure is isolated.
---
# 105. Recovery levers
AWS Well-Architected explicitly recommends emergency operational levers as part of failure-resistant distributed system design.
Every Ekho emergency lever must therefore have:
```text
purpose
owner
safe default
activation procedure
rollback
test
```
---
# 106. Recovery mode entry
Do not automatically enter severe recovery/read-only mode because of one transient request failure.
Entry should require:
```text
high-confidence automated condition
or
authorized operator decision
```
depending on failure.
---
# 107. Recovery mode exit
Never exit because:
```text
5 minutes passed
```
Exit based on:
```text
dependency healthy
data integrity verified
critical smoke tests pass
queues controlled
observability healthy
```
---
# 108. Re-enable gradually
Recovery sequence:
```text
safe reads
↓
critical writes
↓
normal writes
↓
background jobs
↓
notifications
↓
noncritical enrichment
```
Do not turn every subsystem on simultaneously after a major outage.
---
# 109. Dependency recovery is not Ekho recovery
Vendor status:
```text
Resolved
```
does not automatically mean:
```text
Ekho recovered
```
Verify:
* our requests;
* our DB;
* our queues;
* our sessions;
* our data integrity.
---
# 110. Supabase recovery verification
After a significant Supabase incident/restore verify at least:
```text
DB reads
DB writes
RLS
Auth
Storage
API
Realtime if used
connection health
critical functions
```
before declaring normal operation.
---
# 111. Derived-system rebuild order
After canonical DB recovery:
```text
canonical DB
↓
search
↓
cache
↓
derived personalization state if applicable
↓
analytics
```
Canonical truth always comes first.
---
# 112. Cache after restore
Invalidate caches that may contain data newer than the restored canonical database or otherwise inconsistent with restored state.
Never serve:
```text
cache version 51
against
database version 48
```
without deliberate reconciliation.
---
# 113. Search after restore
Rebuild/reconcile search from canonical restored data.
Do not attempt to force database state to match a stale search index.
---
# 114. Realtime after restore
If Realtime/subscriptions are used:
clients must be capable of reconnecting/reloading canonical state.
Do not assume every event emitted during outage can be replayed perfectly.
---
# 115. Client reconciliation
After reconnection:
prefer:
```text
fetch current canonical state
```
rather than relying exclusively on missing realtime events.
---
# 116. Write uncertainty
If client loses connection after sending a mutation:
state may be:
```text
succeeded
or
failed
```
Client must use idempotency/current-state reconciliation rather than assuming failure.
This follows the idempotency requirements from API & Error Contract.
---
# 117. No duplicate application creation
Example:
```text
Create application
→ server commits
→ response lost
→ user retries
```
must not create two applications.
Recovery behavior must remain compatible with idempotency protections.
---
# 118. Data corruption vs service outage
Differentiate:
### Availability failure
```text
system unavailable
```
### Integrity failure
```text
system available
but state may be wrong
```
Integrity incidents usually require stricter write restrictions.
---
# 119. Wrong admissions data
Example:
```text
deadline imported incorrectly
```
Recovery:
```text
stop propagation
↓
identify last verified safe value
↓
restore/correct canonical value
↓
preserve wrong version in history
↓
identify affected users
↓
correct notifications if required
```
Do not erase history.
---
# 120. Source truth recovery
A database backup can restore:
```text
what Ekho previously stored
```
but it cannot prove:
```text
the university information is still current
```
After a sufficiently old recovery point, critical admissions data may require source reverification.
---
# 121. Recovery freshness state
Restored records retain historical:
```text
verified_at
```
Do not change it to recovery date.
Bad:
```text
backup restored today
→ all universities verified today
```
---
# 122. Missing changes after restore
If restoring to an earlier point loses legitimate later mutations:
use available:
```text
audit logs
import history
external side-effect logs
backup copies
```
to identify lost changes.
Do not manually replay uncertain operations without verification.
---
# 123. Recovery notifications
Do not send:
```text
Everything is fixed
```
until critical workflows are actually verified.
User messaging must be factual.
---
# 124. User-facing degraded message
Good:
> Application updates are temporarily unavailable. You can still view your existing applications and university information.
Bad:
> Database error 57P03.
---
# 125. Write failure UX
When write is unavailable:
show:
```text
Not saved
Try again later
```
or preserve an explicit pending state.
Never silently drop user's mutation.
---
# 126. Deadline-sensitive failure UX
If an applicant is near a deadline:
prioritize showing:
```text
official deadline
source
source-local timezone
```
from safe canonical/cached data even if noncritical personalization features are unavailable.
---
# 127. No false reassurance
Do not show:
```text
Your application is safe
```
unless Ekho actually knows that.
Prefer:
> We can't currently save changes. Your previously saved data remains available.
only when that statement is verified.
---
# 128. Disaster recovery runbook
Maintain one top-level DR runbook with:
```text
Detection
Incident declaration
Write freeze
Backup inventory
Restore options
Recovery-point selection
Database restoration
Storage restoration
Auth verification
Configuration restoration
Derived-system rebuild
Critical smoke tests
Cutover
Backlog replay
Exit criteria
```
---
# 129. Runbook must be executable
Avoid documentation like:
```text
Restore database if needed.
```
Runbook must identify:
```text
where backup exists
who can access it
what exact restore method exists
what must be checked afterward
```
---
# 130. Do not store recovery runbook only inside failed system
Recovery documentation must remain accessible if:
```text
production DB unavailable
Ekho Admin unavailable
```
NIST storage/recovery guidance recommends keeping recovery procedures themselves protected and available.
---
# 131. Recovery access
At least one secure copy of recovery instructions must be accessible independently from production application infrastructure.
---
# 132. Recovery ownership
At solo-founder stage:
```text
primary owner = founder
```
But procedures must still be written so another competent engineer could execute them later.
The business must not depend permanently on undocumented personal memory.
---
# 133. Full-provider disaster
If the entire Supabase production project is unrecoverable:
preferred path:
```text
create replacement project
↓
restore database
↓
restore Storage
↓
configure Auth/providers
↓
restore secrets/config
↓
deploy application against replacement
↓
verify
↓
cut over
```
Supabase documents both database restoration to new projects and separate handling of Storage objects.
---
# 134. Project deletion risk
Supabase documents that deleting a project removes database data, Storage objects and access to its automated backups.
Therefore independent backups are mandatory before Ekho depends materially on one Supabase project.
---
# 135. Do not self-host for hypothetical disaster recovery
Ekho v1 should not introduce a self-hosted Supabase replica merely to appear more resilient.
Managed Supabase currently provides managed backups/PITR capabilities that self-hosted deployments do not automatically provide.
Revisit multi-provider/self-hosting only when:
```text
measured risk
SLO
scale
compliance
```
justify the operational burden.
---
# 136. No active-active multi-region v1
Ekho v1 does **not** require active-active databases across multiple clouds/regions.
Current strategy:
```text
managed primary
+
public safe caching
+
PITR
+
independent backups
+
tested restoration
+
graceful degradation
```
This is an Ekho complexity/cost decision.
---
# 137. Read replicas are not DR requirement
Read replicas may later help:
* read scaling;
* regional latency;
* availability architecture.
Supabase currently supports read replicas through WAL/physical backup mechanisms.
But do not add one before measured demand.
---
# 138. Recovery exercises
Google Cloud explicitly recommends testing system behavior under failures rather than assuming recovery paths work.
Ekho must test at least:
```text
bad deployment
database unavailable
database restore
Auth outage
Storage outage
email outage
AI outage
pipeline outage
wrong critical admissions data
```
---
# 139. Graceful-degradation tests
Google SRE notes that rarely exercised degradation code is more likely to fail when finally needed and recommends exercising degraded behavior.
Therefore degraded modes are part of normal staging/recovery testing.
---
# 140. Database drill
Quarterly staging/recovery drill:
```text
restore production-like backup
→ start application
→ sign in with test user
→ open application
→ modify test task
→ verify RLS
→ open university
→ verify source data
```
Record achieved RTO.
---
# 141. Storage drill
Test:
```text
restore metadata
+
restore independent document objects
+
verify object ownership
+
download as correct test user
+
reject wrong user
```
A file that exists but violates authorization is not a successful recovery.
---
# 142. Auth drill
Test:
* login;
* refresh;
* logout;
* recovery;
* OAuth callback;
* session validation.
After major restore, verify no unexpected stale security state exists.
---
# 143. Pipeline drill
Simulate:
```text
crawler unavailable for 24h
```
Expected:
```text
public canonical data remains
freshness ages
issue created
no fake updates
recovery resumes safely
```
---
# 144. Notification drill
Simulate email/push provider outage.
Expected:
```text
queue controlled
no duplicate sends
irrelevant messages discarded after recovery
important valid messages replayed gradually
```
---
# 145. AI drill
Disable AI provider.
Expected:
```text
core Ekho remains usable
```
If product becomes unusable:
AI has accidentally become a hard dependency and architecture must be corrected.
---
# 146. Dependency timeout drill
Simulate dependency returning after:
```text
60 seconds
```
Ekho should hit its own bounded timeout and fail/degrade safely rather than letting the request consume resources indefinitely.
---
# 147. Retry storm drill
Simulate:
```text
dependency returns 503
```
Verify:
* retries bounded;
* jitter exists;
* no exponential request amplification;
* circuit breaker/fail-fast behavior where applicable.
---
# 148. Restore validation checklist
After restore verify:
* [ ] expected DB schema/version;
* [ ] migrations match deployed code;
* [ ] row counts plausible;
* [ ] RLS enabled;
* [ ] foreign keys valid;
* [ ] critical DB functions present;
* [ ] users/auth work;
* [ ] application ownership correct;
* [ ] sources/provenance intact;
* [ ] Storage objects available;
* [ ] document ownership correct;
* [ ] search rebuild succeeds;
* [ ] public university page works;
* [ ] application write works;
* [ ] requirements load;
* [ ] admin access works;
* [ ] observability works.
---
# 149. Backup validation checklist
* [ ] backup generated on schedule;
* [ ] encrypted;
* [ ] stored outside production project;
* [ ] retention correct;
* [ ] decryptable;
* [ ] integrity/checksum passes where applicable;
* [ ] restore tested;
* [ ] Storage independently covered;
* [ ] access restricted;
* [ ] failed backup alerts.
---
# 150. RPO validation
For recovery drill:
identify:
```text
last valid production write
vs
latest recovered write
```
Actual gap must satisfy target.
Do not calculate RPO from backup schedule alone.
---
# 151. RTO validation
Measure:
```text
incident/recovery start
→ verified usable service
```
not:
```text
database restore command finished
```
Google's recovery testing guidance stresses validating the restored application stack, not just the storage restoration process.
---
# 152. Recovery completion
Recovery is complete only when:
```text
core SLO recovered
critical smoke tests pass
data integrity verified
Auth verified
backlog controlled
monitoring normal
```
---
# 153. Post-recovery monitoring
After major restore:
keep elevated monitoring for a defined observation window.
Watch:
```text
5xx
DB errors
Auth errors
RLS failures
queue backlog
duplicate operations
notification failures
Storage errors
data-quality issues
```
---
# 154. Recovery postmortem
Every real disaster recovery event produces:
```text
Incident postmortem
+
actual RTO
+
actual RPO
+
what failed in recovery
+
recovery improvements
```
according to Observability/Incident Response.
---
# 155. Recovery plan changes
When:
```text
storage provider changes
Auth architecture changes
DB architecture changes
notification provider changes
deployment platform changes
```
update recovery procedures and re-test relevant paths.
---
# 156. Recovery debt
Any manual recovery step repeated during drills should be evaluated for automation.
But:
```text
automation
```
must improve safety, not hide dangerous operations behind one button.
---
# 157. Dangerous one-click restore
Do not create:
```text
Admin → Restore Production Database
```
inside normal Ekho Admin.
Disaster restoration remains a privileged infrastructure operation with deliberate confirmation.
---
# 158. Recovery credentials
Recovery credentials must never be:
```text
stored in Obsidian plaintext
pasted into runbook
committed to Git
```
Runbooks reference **where/how to retrieve** credentials, not the credentials themselves.
---
# 159. Backup observability
Monitor:
```text
last successful database backup
last successful independent backup
last document mirror success
backup failures
reconciliation mismatch
last restore test
```
---
# 160. Recovery dashboard
Admin/operations should eventually answer:
```text
Last DB backup?
PITR enabled?
Last external copy?
Last Storage reconciliation?
Last restore test?
Achieved RTO/RPO?
```
Do not build a giant custom backup platform.
---
# 161. Alert thresholds
PAGE when:
```text
critical backup system repeatedly failing
independent backup absent beyond RPO
Storage backup backlog exceeds RPO
```
TICKET when:
```text
single retryable backup fails
noncritical backup delayed
```
according to Observability alert principles.
---
# 162. Backup failure must not crash product
Backup process is asynchronous.
If backup destination is unavailable:
```text
Ekho stays online
+
backup alert
+
retry bounded
```
Do not make every application save synchronously wait on independent backup storage.
---
# 163. Exception — primary document upload
User upload success depends on:
```text
primary object stored
```
not independent backup completion.
Independent backup follows asynchronously and is operationally monitored.
---
# 164. External university source outage
Source retrieval jobs should use:
```text
timeout
bounded retries
backoff
```
and then mark source unreachable.
Do not let one slow university site block an entire processing batch.
---
# 165. Isolation
One failed:
```text
university
provider
job
notification
```
must not fail the entire batch unless atomic semantics genuinely require it.
---
# 166. Atomicity where correctness requires
Some workflows must remain atomic:
```text
Import publication
canonical merge
account deletion transaction portions
```
If these fail:
```text
rollback
```
rather than partially degrade consistency.
Graceful degradation does not mean accepting half-completed integrity-critical transactions.
---
# 167. Failure matrix
| Failure               | User behavior                                   |
| --------------------- | ----------------------------------------------- |
| AI                    | Core Ekho continues                             |
| Search                | Direct pages + basic fallback                   |
| Email notifications   | Product continues, queue                        |
| OAuth provider        | Offer other auth methods                        |
| All Auth              | Public research only; private remains protected |
| Storage               | Everything except document operations           |
| Pipeline              | Existing canonical data remains                 |
| Import/Admin          | Students unaffected                             |
| Observability         | Product continues                               |
| Flag provider         | Safe defaults / cached state                    |
| DB writes unsafe      | Read-only                                       |
| DB fully unavailable  | Safe public cache where possible                |
| Bad deploy            | Rollback                                        |
| Full backend disaster | DR restore process                              |
---
# 168. What Ekho must NEVER do during failure
Never:
```text
disable RLS
skip authentication
invent admissions facts
silently lose writes
retry forever
duplicate notifications
serve another user's cache
publish partially restored data
mark stale data freshly verified
accept document upload then lose it silently
declare recovery without testing
```
---
# 169. P0 failures
Any of these blocks production:
* database has no recoverable backup;
* backup has never been restored/tested;
* private Storage objects have no recovery strategy;
* Supabase database backup is incorrectly assumed to contain Storage files;
* no independent recovery copy exists once real user data is important;
* corrupted data can continue accepting writes uncontrollably;
* read-only mode exists only as hidden buttons;
* AI/email/search failure takes entire Ekho down;
* retries can become unbounded;
* mutation retry can duplicate critical records;
* restored RLS/ownership is not verified;
* restored Auth state is trusted without security review;
* backup credentials exposed client-side;
* recovery documentation exists only inside failed infrastructure;
* restored admissions data receives fake fresh `verified_at`;
* backlog replay can create notification storm;
* system declares recovery before critical workflows are verified.
---
# 170. Implementation order for Codex
## Stage 1 — Failure map
1. inventory all dependencies;
2. mark hard vs soft;
3. define failure behavior;
4. define criticality;
5. define RTO/RPO.
## Stage 2 — Graceful degradation
6. AI fallback;
7. search fallback;
8. email/notification fallback;
9. Storage failure UX;
10. pipeline failure behavior;
11. database read-only UX;
12. system operational-state model.
## Stage 3 — Resilience primitives
13. timeouts;
14. bounded retries;
15. exponential backoff;
16. jitter;
17. idempotency;
18. circuit breakers where needed;
19. queue backpressure;
20. load shedding.
## Stage 4 — Database recovery
21. verify managed backups;
22. configure PITR when production target requires;
23. create independent logical backup;
24. encrypt/separate backup storage;
25. backup monitoring;
26. restore runbook.
## Stage 5 — Storage recovery
27. independent private-document backup flow;
28. backup-state tracking;
29. retry;
30. daily reconciliation;
31. restore tooling;
32. ownership validation.
## Stage 6 — Recovery controls
33. subsystem kill switches;
34. publication pause;
35. notification pause;
36. write freeze/read-only mechanism;
37. recovery mode;
38. gradual re-enable.
## Stage 7 — DR
39. full recovery manifest;
40. infrastructure/config recovery;
41. Auth recovery checks;
42. cache/search rebuild;
43. backlog replay;
44. recovery smoke tests.
## Stage 8 — Exercises
45. database restore test;
46. Storage restore test;
47. Auth outage;
48. AI outage;
49. email outage;
50. bad deployment rollback;
51. pipeline outage;
52. data corruption exercise;
53. measure actual RTO/RPO.
Do not build active-active multi-region architecture during this stage.
---
# 171. Codex implementation constraint
Before implementing this specification read:
```text
Data Architecture
Data Pipeline
Security & Privacy
Auth & Account Lifecycle
Import & Ingestion
Admin & Data Operations
Observability / SLO / Incident Response
API & Error Contract
Feature Flags & Runtime Configuration
```
Do not create separate conflicting:
```text
retry policy
read-only policy
backup model
error contract
flag system
```
---
# 172. Definition of Done
Failure, Recovery & Degraded Mode v1 is complete when:
* every important dependency has documented failure behavior;
* hard/soft dependencies are explicit;
* graceful degradation exists for noncritical services;
* AI is not a hard dependency;
* search/email/pipeline failures do not take down core Ekho;
* read-only state exists;
* write freeze occurs at trusted boundary;
* external calls have timeouts;
* retries are bounded;
* backoff + jitter are implemented where appropriate;
* mutation retries remain idempotent;
* database backup strategy exists;
* production RPO is supported by real infrastructure;
* independent database backup exists;
* Cloudflare R2 files are separately protected;
* document backup lag is monitored;
* backups are encrypted/separated;
* restore procedure exists;
* recovery instructions survive production outage;
* quarterly restore drills work;
* actual RTO/RPO are measured;
* restored Auth/RLS/ownership are checked;
* derived systems rebuild from canonical DB;
* queued work recovers gradually;
* recovery is verified before declaring normal;
* all P0 tests pass.
---
# 173. Final invariant
Ekho must behave like this:
```text
ONE SYSTEM FAILS
↓
Isolate failure
↓
Protect truth + user data
↓
Keep everything safe that can still work
↓
Recover from known-good state
↓
Verify
↓
Gradually restore full functionality
```
Not:
```text
Supabase/AI/email has a problem
↓
whole Ekho dies
↓
retry everything
↓
randomly restore backup
↓
hope data is correct
```
And for catastrophic failure:
```text
Production lost
↓
Independent recovery assets exist
↓
Restore canonical DB
↓
Restore private Storage
↓
Restore config/security
↓
Verify Auth + RLS + data
↓
Rebuild derived systems
↓
Smoke test
↓
Cut over
↓
Measure RTO/RPO
```
---
# 174. Primary authority sources
This specification was checked against a broader source set than the previous documents:
1. **NIST Cybersecurity Framework 2.0** — backup protection/testing, Respond and Recover outcomes.
2. **NIST SP 800-61 Rev.3 (2025)** — current incident response and recovery guidance.
3. **NIST SP 800-34 Rev.1** — contingency planning, RTO/RPO and recovery planning.
4. **NIST SP 800-184 — Guide for Cybersecurity Event Recovery** — recovery playbooks, testing and improvement.
5. **NIST SP 800-209 — Security Guidelines for Storage Infrastructure** — recovery copies, backup validation, DR documentation and restoration.
6. **CISA StopRansomware guidance** — isolated/encrypted backups and regular recovery testing.
7. **Google SRE — Addressing Cascading Failures** — graceful degradation, overload and retry amplification.
8. **Google SRE — Handling Overload** — load shedding and degraded responses.
9. **Google Cloud Well-Architected Framework, current 2026** — dependency isolation, graceful degradation and failure recovery testing.
10. **Google Cloud Disaster Recovery Planning Guide** — RTO/RPO and DR planning.
11. **Google Cloud Recovery from Data Loss guidance** — validating recovery through the complete restored application stack.
12. **AWS Well-Architected Reliability Pillar** — graceful degradation, backpressure, timeouts, circuit breakers and bounded retries.
13. **Amazon Builders' Library — Timeouts, Retries and Backoff with Jitter**, updated 2026.
14. **Supabase current 2026 Database Backup/PITR documentation** — managed backups, PITR and restoration behavior.
15. **Supabase current Database documentation** — Storage objects are not contained in database backups.
16. **Supabase Restore-to-New-Project documentation** — isolated new-project restoration path.
17. **Supabase backup/restore documentation** — separate Storage-object restoration requirements.
18. **Supabase Auth signing-key/session documentation** — session/key behavior relevant after security/recovery events.
19. **Supabase production checklist** — production security, MFA, backups/PITR and operational readiness.
20. **Cloudflare Workers rollback documentation** — application deployment rollback to a previously deployed Worker version.
