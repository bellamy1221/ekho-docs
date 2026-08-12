# Observability, SLO & Incident Response v1
**Status:** LOCKED
**Scope:** production observability, reliability objectives, alerting, incident response and postmortems
**Stack:** Next.js + Supabase/PostgreSQL + OpenTelemetry-compatible instrumentation
**Depends on:** Data Architecture, Data Pipeline, Security & Privacy, Auth & Account Lifecycle, Import & Ingestion, Admin & Data Operations
**Research verified:** 12 August 2026
---
# 1. Goal
Ekho must be able to answer quickly:
```text
Is the product working?
↓
Are users actually able to complete critical actions?
↓
What is failing?
↓
Who is affected?
↓
When did it start?
↓
What changed?
↓
How do we mitigate it?
↓
How do we prevent recurrence?
```
Monitoring must be designed around **user impact**, not merely infrastructure being online.
OpenTelemetry explicitly distinguishes reliability from simple uptime: a service can technically be available while still failing to do what users expect.
---
# 2. Core reliability principle
The primary reliability unit is:
> **A user successfully completing an important Ekho action.**
Examples:
* open a university;
* view requirements;
* save an application;
* update a task;
* authenticate;
* upload/access a document;
* view current admissions information.
Do not define reliability only as:
```text
server returns HTTP 200
```
---
# 3. Observability model
Ekho uses three primary telemetry signals:
```text
Logs
Metrics
Traces
```
OpenTelemetry defines logs, metrics and traces as core observability signals, while context propagation enables those signals to be correlated across a request path.
---
# 4. Tooling principle
Instrumentation should be **vendor-neutral where practical**.
Use:
**OpenTelemetry-compatible instrumentation**
as the conceptual standard.
OpenTelemetry is specifically designed as a vendor-neutral framework for generating, collecting and exporting telemetry.
---
# 5. Ekho v1 observability stack
Recommended v1:
```text
Next.js instrumentation
→ OpenTelemetry conventions
Application errors/traces
→ Sentry
Supabase infrastructure
→ Supabase Logs + Metrics API
External availability
→ independent synthetic uptime monitor
```
Sentry's Next.js SDK supports errors, logs and traces, while Supabase exposes logs plus a Prometheus-compatible Metrics API containing Postgres health/performance metrics.
---
# 6. Next.js instrumentation
Use the official Next.js instrumentation mechanism.
Next.js currently supports OpenTelemetry instrumentation and provides `instrumentation.ts/js` specifically for observability integration.
Do not scatter unrelated monitoring initialization across random components.
---
# 7. Sentry role
Use Sentry primarily for:
```text
application exceptions
frontend failures
server failures
stack traces
distributed traces
release correlation
```
Sentry supports distributed tracing for Next.js and can correlate requests across application layers.
Do not turn Sentry into the canonical storage for product analytics.
---
# 8. Source maps
Production JavaScript source maps should be uploaded privately to the error-monitoring service so stack traces resolve to original source.
Sentry's Next.js integration supports build-time source-map upload and uses source maps to convert minified stack traces back to original source.
Do not deliberately expose private production source maps publicly unless separately justified.
---
# 9. Release identification
Every production telemetry event should contain:
```text
environment
release
git_sha / deployment identifier
service
```
Sentry release tracking is specifically designed to associate errors with deployed code versions.
This allows:
```text
errors started
↓
deployment 8f4c...
```
to be identified quickly.
---
# 10. Environments
Telemetry must distinguish:
```text
development
staging
production
```
Never mix staging errors into production dashboards.
Every signal must contain:
```text
deployment.environment
```
or equivalent standardized environment field.
---
# 11. Development telemetry
Development should favor debugging.
Allow:
* verbose local logs;
* full traces where useful;
* detailed stack traces.
Production must favor:
* signal;
* privacy;
* controlled volume;
* cost.
---
# 12. Structured logging only
Server logs must be structured.
Preferred conceptual event:
```json
{
  "timestamp": "...",
  "level": "error",
  "service": "web",
  "environment": "production",
  "event": "application_save_failed",
  "request_id": "...",
  "trace_id": "...",
  "route": "/api/applications/:id",
  "status_code": 500,
  "duration_ms": 421,
  "error_code": "DATABASE_WRITE_FAILED"
}
```
Do not depend on unstructured sentences such as:
```text
Something failed somewhere
```
---
# 13. Log levels
Use:
```text
DEBUG
INFO
WARN
ERROR
FATAL
```
Production expectations:
### DEBUG
Normally disabled or heavily restricted.
### INFO
Important normal lifecycle events.
### WARN
Unexpected but recoverable condition requiring future investigation.
### ERROR
Operation failed or meaningful functionality was affected.
### FATAL
Process/system state cannot continue safely.
Do not log routine successful requests individually unless there is a clear operational need.
---
# 14. Logs are not analytics
Do not log:
```text
user opened university
user clicked compare
user viewed scholarship
```
merely to measure product behavior.
Those are product analytics.
Observability logs exist for:
```text
operation
reliability
debugging
security
```
Keep the systems conceptually separate.
---
# 15. Request correlation
Every server request should receive or propagate a:
```text
request_id
```
Distributed operations should additionally propagate:
```text
trace_id
```
OpenTelemetry context propagation exists specifically to correlate telemetry across components involved in the same operation.
---
# 16. Error correlation
A support/incident operator should be able to move:
```text
error
→ trace
→ request
→ deployment
→ relevant logs
```
without manually searching timestamps across unrelated tools.
---
# 17. No sensitive telemetry
Never log:
* passwords;
* access tokens;
* refresh tokens;
* OAuth codes;
* verification tokens;
* password-reset tokens;
* Supabase service secrets;
* private document contents;
* essay contents;
* full uploaded files.
This remains subordinate to Security & Privacy.
---
# 18. Personal information
Avoid logging:
* email;
* full name;
* address;
* phone;
* raw IP unless operational/security purpose requires it;
* university application essays;
* private notes.
If an internal `user_id` is operationally necessary, treat it as potentially personal data and restrict retention/access accordingly.
---
# 19. Error-monitoring scrubbing
Configure sensitive-data scrubbing at both:
```text
SDK/application boundary
+
observability backend
```
Sentry supports client-side filtering/scrubbing as well as server-side data-scrubbing rules.
Never rely on one regex to protect all sensitive data.
---
# 20. Query-string safety
Do not include sensitive URL query values in telemetry.
Examples to redact:
```text
token
code
secret
key
email
invite
reset
verification
```
Sensitive authentication secrets must never become:
```text
URL
→ log
→ Sentry
→ long-term telemetry
```
---
# 21. Metrics principle
Metrics should answer numerical questions over time.
Examples:
```text
requests
errors
latency
database connections
queue backlog
import failures
auth failures
```
OpenTelemetry describes metrics as numerical measurements aggregated over time.
---
# 22. Metric cardinality
Never use unbounded metric dimensions such as:
```text
user_id
email
raw URL
application_id
document_id
university_id
```
Prometheus explicitly warns against labels containing high-cardinality values such as user IDs and email addresses because every unique label combination creates additional time series.
OpenTelemetry similarly notes that high-cardinality attributes increase memory/storage costs.
---
# 23. Good metric dimensions
Acceptable examples:
```text
environment
service
operation
route_template
HTTP method
status class
country group where genuinely useful
provider
result
```
Use:
```text
/universities/:slug
```
not:
```text
/universities/mit
/universities/harvard
/universities/stanford
...
```
as separate metric labels.
---
# 24. Core application metrics
At minimum collect:
```text
http_requests_total
http_request_duration
http_request_errors_total
```
Plus domain operations:
```text
application_save_total
application_save_failed_total
requirement_load_total
requirement_load_failed_total
document_upload_total
document_upload_failed_total
auth_attempt_total
auth_failed_total
import_validate_total
import_publish_total
import_publish_failed_total
```
Exact naming should follow OpenTelemetry/Prometheus conventions where applicable.
---
# 25. Database telemetry
Monitor Supabase/Postgres:
```text
CPU
memory
disk
connections
query latency
slow queries
locks
IO
database errors
```
Supabase exposes project health/performance metrics through its Prometheus-compatible Metrics API and provides database debugging tools for CPU, RAM, disk IO, query performance and locks.
---
# 26. Supabase subsystem telemetry
Monitor relevant health for:
```text
Database
Auth
Storage
Realtime
API
```
Supabase's own Reports expose observability views for these platform components.
---
# 27. Auth telemetry
Monitor:
```text
login success/failure
signup failure
OAuth callback failure
verification failure
password reset failure
session refresh failure
rate-limit events
```
Do not create metrics keyed by email address.
---
# 28. Storage telemetry
Monitor:
```text
upload success rate
download success rate
latency
storage provider errors
authorization failures
```
Separate expected:
```text
403 unauthorized
```
from:
```text
500 provider failure
```
---
# 29. Data Pipeline telemetry
Monitor:
```text
sources_checked
sources_failed
changes_detected
verification_jobs_failed
queue_backlog
oldest_job_age
processing_duration
```
A pipeline that continues running but is 24 hours behind can be operationally broken.
---
# 30. Import telemetry
Monitor:
```text
imports_started
imports_validation_failed
imports_ready
imports_published
imports_failed
publish_duration
```
Do not page because somebody pasted malformed JSON.
Page only when the importer/system itself is malfunctioning.
---
# 31. Admin operations telemetry
High-risk admin operations should emit operational telemetry:
```text
publish_failed
merge_failed
archive_failed
derived_sync_failed
```
Detailed actor auditing remains in Admin Audit, not generic metrics.
---
# 32. Trace principle
Use distributed tracing to answer:
> Where did this request spend time and where did it fail?
OpenTelemetry traces model an operation as a sequence/tree of spans representing the request's path through the system.
---
# 33. Important spans
Trace important operations such as:
```text
HTTP request
auth verification
database query
external provider call
storage operation
import publication
search lookup
pipeline processing
```
Do not create spans for every trivial JavaScript function.
---
# 34. Trace naming
Use stable operation names.
Good:
```text
POST /api/applications/:id
db.application.update
storage.document.upload
```
Bad:
```text
POST /api/applications/6c79c...
```
Stable names prevent high-cardinality telemetry.
---
# 35. Trace sampling
Ekho launch baseline:
```text
development/staging:
high or 100% trace sampling when practical
production:
start around 10% for ordinary successful traffic
```
Error events should still be captured independently.
Sampling rate must be centrally configurable without source-code rewrites.
This percentage is an **Ekho cost/operations starting point**, not an external standard.
---
# 36. Increase sampling during incident
During investigation:
```text
10%
→ temporarily higher
```
may be used where cost/privacy constraints allow.
After resolution return to normal rate.
Do not permanently increase collection because someone forgot to reset it.
---
# 37. Real user performance
Track real-user web performance.
Primary browser metrics:
```text
LCP
INP
CLS
```
Current Core Web Vitals guidance defines "good" performance approximately as:
```text
LCP ≤ 2.5s
INP ≤ 200ms
CLS ≤ 0.1
```
evaluated at the 75th percentile of page visits.
---
# 38. Ekho frontend performance target
Ekho target:
At least **75% of real-user visits** should remain within the current "good" Core Web Vitals thresholds for major user-facing pages.
Measure separately where possible by:
```text
mobile / desktop
major page type
```
Do not optimize solely against local Lighthouse runs.
---
# 39. Field data over lab-only assumptions
Production performance decisions should favor real-user field data when enough traffic exists.
Web performance guidance notes that lab and field metrics can differ because real users experience varied devices, networks and conditions.
---
# 40. Synthetic monitoring
Run an external monitor independent from the application stack.
Minimum production probes:
```text
public homepage
public university page
basic authenticated health flow where practical
```
A monitor inside the same failing infrastructure is insufficient as the sole availability signal.
---
# 41. Synthetic account
If production authenticated synthetic checks are introduced:
use a dedicated:
```text
synthetic_test_account
```
Never use a real student account.
Tag synthetic operations and exclude them from product analytics.
---
# 42. Synthetic writes
Production synthetic writes must:
* use isolated test objects;
* be idempotent;
* clean themselves up safely;
* never touch real applications;
* never send real notifications.
Do not repeatedly create random production records.
---
# 43. Service Level Indicator
An **SLI** measures the observed reliability experienced by users.
Google SRE recommends defining SLIs around user-facing service behavior and using them as the basis for SLOs.
Conceptually:
```text
good events
───────────
valid events
```
---
# 44. Service Level Objective
An **SLO** is an internal reliability objective for an SLI.
It is not automatically a contractual promise.
Google SRE distinguishes objectives from error budgets and explicitly argues against targeting 100% reliability for ordinary services.
---
# 45. SLA
Ekho v1 does **not** promise a public uptime SLA.
Do not put:
```text
99.99% guaranteed uptime
```
into marketing/legal terms merely because an internal SLO exists.
Public contractual SLAs require a separate business/legal decision.
---
# 46. SLO window
Ekho launch reliability window:
**rolling 28 days**
Review:
```text
continuously through dashboards
+
formal monthly reliability review
```
28 days is an Ekho operational baseline inspired by common SRE four-week examples, not a universal standard. Google SRE frequently models error budgets over four-week windows.
---
# 47. Critical user journeys
Initial SLO coverage:
```text
1. Public university research
2. Authenticated workspace loading
3. Saving application state
4. Loading personalized requirements
5. Authentication
6. Private document access
```
Do not create SLOs for every internal endpoint.
---
# 48. Public university availability SLI
Eligible event:
```text
request to a published university page
```
Good event:
```text
correct university content successfully loads
```
Initial SLO:
**99.9% over rolling 28 days**
Ekho internal target.
---
# 49. Workspace availability SLI
Eligible event:
authenticated request for:
```text
My Applications workspace
```
Good event:
the workspace successfully returns usable application data.
Initial SLO:
**99.9%**
---
# 50. Application write SLI
Eligible events:
```text
valid application/task mutations
```
Good event:
```text
mutation committed successfully
AND
confirmed to client
```
Initial SLO:
**99.9%**
Do not count validation failures caused by invalid user input as server failures.
---
# 51. Requirements SLI
Eligible event:
```text
request for available personalized requirement data
```
Good event:
```text
service returns correct stored/personalized result without system error
```
Initial technical availability SLO:
**99.9%**
This does **not** mean:
```text
99.9% factual university-data accuracy
```
Never use availability metrics to imply factual accuracy.
---
# 52. Authentication SLI
Eligible events:
valid login/refresh operations excluding:
* incorrect credentials;
* deliberately rejected abuse;
* provider cancellation.
Good event:
successful authentication/session operation.
Initial Ekho SLO:
**99.9%**
Third-party provider outages still affect user experience and should remain visible in the user-facing SLI.
---
# 53. Document access SLI
Eligible events:
authorized upload/download operations.
Good event:
private document is successfully stored/retrieved.
Initial SLO:
**99.9%**
Authorization denial from an unauthorized request is not a reliability failure.
---
# 54. Internal admin services
Internal admin/import operations may use a lower launch objective such as:
**99.5%**
because they are not directly user-facing and can often be retried safely.
This is an Ekho prioritization decision.
---
# 55. Latency SLI
For interactive server operations measure:
```text
request latency
```
excluding intentionally asynchronous/background tasks.
Initial Ekho targets:
```text
critical reads:
p95 ≤ 1 second server-side
critical writes:
p95 ≤ 1.5 seconds server-side
```
These are starting engineering budgets, not external standards.
Recalibrate after real production measurements.
---
# 56. Do not combine browser and server latency
Keep separate:
```text
server request latency
```
and:
```text
real browser experience
```
A 100ms API does not guarantee a fast page.
Core Web Vitals are the preferred user-facing browser performance measure.
---
# 57. SLO denominator rules
Every SLI must explicitly define:
```text
eligible event
good event
bad event
excluded event
measurement source
window
objective
```
Never change denominator rules during a bad month to make reliability look better.
---
# 58. Dependency failures
If Supabase or another dependency fails and Ekho users cannot complete an operation:
```text
user-facing SLI = bad
```
Do not exclude the event simply because:
> "It wasn't our code."
Users experience Ekho as one product.
---
# 59. Client/network errors
Do not automatically count every network interruption as Ekho failure.
Classify where determinable:
```text
Ekho/server failure
dependency failure
client cancellation
offline client
unknown
```
Keep user-experience measurements separately where useful.
---
# 60. Error budget
For an SLO:
```text
error budget = 100% - SLO
```
Google SRE uses error budgets specifically to balance reliability work against continued product change.
For a 99.9% event-based SLO:
```text
0.1% of eligible events may be bad
```
during the evaluation window before the objective is missed.
---
# 61. Error budget dashboard
For every critical SLO display:
```text
current SLI
target
remaining error budget
budget consumed
burn rate
window
```
Do not make engineers manually calculate reliability from raw logs.
---
# 62. Error budget policy
Ekho launch policy:
### >50% budget remaining
Normal development.
### 25–50%
Investigate recurring contributors.
### <25%
Prioritize reliability and restrict unnecessary high-risk deployment.
### Exhausted
Freeze nonessential risky production changes until:
* major cause understood;
* urgent reliability actions completed;
* service is stable.
This is an Ekho adaptation of the SRE error-budget model. Google describes error budgets as a mechanism for shifting effort from features toward reliability when reliability is poor.
---
# 63. Do not chase 100%
Do not use:
```text
100% uptime
0 errors ever
```
as engineering objectives.
Google SRE notes that 100% targets can produce disproportionate cost and overly conservative engineering while reducing deployment velocity.
---
# 64. Alert principle
An alert should mean:
> **A human should act.**
If nobody needs to act:
```text
dashboard / log / issue
```
not:
```text
page
```
Alert fatigue is itself an operational failure.
---
# 65. User-impact alerts first
Primary paging alerts should derive from:
```text
availability
latency
error budget consumption
critical correctness/data integrity
```
not merely:
```text
CPU reached 72%
```
Google SRE specifically recommends using SLI-based alerting as the highest-quality indication that an operator should respond.
---
# 66. Infrastructure alerts
Infrastructure signals such as:
```text
database CPU
connections
disk
queue backlog
```
may page only when they:
* threaten service availability;
* predict imminent failure;
* indicate destructive resource exhaustion.
Otherwise use warning/ticket/dashboard.
---
# 67. Burn-rate alerts
Where traffic volume supports them, alert on **error-budget burn rate** rather than fixed arbitrary error percentages.
Google SRE's recommended high-fidelity SLO alerting approach uses multi-window, multi-burn-rate alerts to balance detection speed and alert precision.
---
# 68. Low traffic
Early Ekho may not have enough traffic for statistically useful burn-rate alerting.
For low-volume services combine:
```text
real SLI
+
synthetic checks
+
absolute failure events
```
Google SRE explicitly discusses low-traffic services as a special SLO-alerting case.
---
# 69. Alert levels
Use two operational alert classes:
### PAGE
Immediate human attention required.
### TICKET
Needs investigation but not immediate interruption.
Do not invent ten alert priorities.
Incident severity is separate.
---
# 70. Every page alert needs a runbook
Before enabling a paging alert define:
```text
What does it mean?
What user impact is likely?
Where should I look first?
What safe mitigation exists?
How do I escalate?
```
If those questions cannot be answered, the alert is probably not ready.
---
# 71. Alert ownership
Every production alert must have:
```text
owner
service
severity expectation
runbook
```
No orphan alert rules.
---
# 72. Alert deduplication
The same failure must not generate:
```text
100 database alerts
+
100 API alerts
+
100 Sentry alerts
```
as separate pages.
Group correlated symptoms into one incident where possible.
---
# 73. Alert recovery
An alert should recover automatically when the measured condition is genuinely healthy.
Avoid alerts that continue indefinitely after recovery.
Google SRE identifies reset time as an important property of an effective alerting strategy.
---
# 74. Dashboard hierarchy
Ekho should have three core operational views:
```text
1. User Reliability
2. System Health
3. Data Operations
```
Avoid one giant dashboard containing 200 graphs.
---
# 75. User Reliability dashboard
Show:
```text
critical SLOs
error budgets
request error rate
latency
Core Web Vitals
current active incident
recent deployment markers
```
This is the first production dashboard.
---
# 76. System Health dashboard
Show only useful infrastructure:
```text
database health
connections
CPU
disk
query latency
API errors
Auth errors
Storage failures
background queue backlog
external dependency failures
```
---
# 77. Data Operations dashboard
Show:
```text
pipeline failures
sources failing
oldest verification job
import failures
derived-sync failures
critical data issues
```
This connects technical reliability with Ekho's admissions-data reliability.
---
# 78. Data correctness is reliability
For Ekho:
```text
website online
+
wrong deadline
=
product failure
```
Operational reliability therefore includes critical admissions-data incidents.
Do not treat data correctness only as a content problem.
---
# 79. No fake accuracy SLO
Do not claim:
```text
99.9% university data accurate
```
unless Ekho possesses a defensible measurable denominator and verification methodology.
Instead monitor observable quality indicators:
```text
critical claims with source
current-cycle coverage
stale critical records
source conflicts
verified critical values
```
---
# 80. Critical data incident
Examples:
```text
wrong published deadline
wrong international eligibility
wrong required exam
wrong financial-aid eligibility
wrong tuition that materially affects decisions
```
These may be higher priority than a temporary visual bug.
---
# 81. Deployment markers
Every production deployment must appear in observability timelines.
Example:
```text
18:41 deployment abc123
18:43 error rate rises
```
This dramatically reduces diagnosis time.
Release/deployment correlation is supported directly by Sentry's release model.
---
# 82. Deploy health
After deployment inspect automatically/operationally:
```text
error rate
latency
critical SLI
new Sentry issues
```
Do not assume deployment success because CI returned green.
---
# 83. Canary rollout later
As infrastructure supports it, risky releases may use canary/gradual rollout.
Google SRE recommends canarying as a way to expose a new release to only a small production population before broad rollout, reducing the blast radius of regressions.
Detailed rollout behavior belongs to Development Workflow / Feature Flags.
---
# 84. Incident definition
An incident is:
> an unplanned event materially affecting confidentiality, integrity, availability, correctness, or expected operation of Ekho.
This includes:
```text
outage
severe degradation
data corruption
cross-user access
critical admissions-data error
security event
```
NIST's current SP 800-61 Rev.3 integrates incident response into broader cybersecurity risk-management, detection, response and recovery activities.
---
# 85. Incident severity
Use:
```text
SEV0
SEV1
SEV2
SEV3
```
Do not combine these with issue priority labels such as P0/P1 used elsewhere.
---
# 86. SEV0 — Critical
Examples:
* cross-user private data exposure;
* credentials/secrets compromised;
* destructive data corruption;
* account takeover vulnerability actively exploited;
* widespread inability to access critical product;
* incorrect admissions data creating immediate severe applicant risk at scale.
Response:
```text
immediate incident
highest priority
stop risky deployments
mitigate first
```
---
# 87. SEV1 — Major
Examples:
* major authenticated workspace outage;
* broad university-data pages unavailable;
* authentication broadly broken;
* document access broadly broken;
* critical deadline data wrong for a significant affected cohort;
* major data-pipeline failure threatening current information.
Requires immediate attention.
---
# 88. SEV2 — Significant
Examples:
* partial feature outage;
* one provider broken but workaround exists;
* elevated errors affecting limited users;
* one university's important incorrect data without immediate mass deadline impact;
* internal admin import outage.
Needs prompt investigation but may not require full incident structure.
---
# 89. SEV3 — Minor
Examples:
* isolated low-impact bug;
* minor performance regression;
* noncritical internal tool failure;
* single retryable background task issue.
Normal engineering workflow.
---
# 90. Severity based on impact
Severity should consider:
```text
number of affected users
criticality of affected action
duration
data/security impact
deadline sensitivity
availability of workaround
```
Do not define severity solely by number of errors.
---
# 91. Admissions deadline multiplier
Ekho-specific rule:
An issue near an application deadline may receive higher severity than the same issue months earlier.
Example:
```text
wrong RD deadline
24 hours before deadline
```
can be SEV0/SEV1 depending on exposure.
---
# 92. Security incidents
Security events follow:
```text
Incident Response
+
Security & Privacy
+
future Legal / Compliance Operations
```
Do not make public disclosure decisions ad hoc from the engineering incident channel.
---
# 93. Incident lifecycle
Use:
```text
Detected
↓
Declared
↓
Triaged
↓
Mitigated
↓
Resolved
↓
Monitored
↓
Reviewed
```
Resolution is not the same as understanding root cause.
---
# 94. Mitigation before root cause
During active user impact:
priority is:
```text
reduce harm
```
before:
```text
perfectly understand every technical detail
```
Examples:
```text
rollback deployment
disable broken feature
serve cached safe data
stop destructive job
switch to read-only mode
```
Root-cause investigation continues after stability.
---
# 95. Incident roles
For major incidents define:
```text
Incident Commander
Operations / Technical Lead
Communications
Scribe
```
Google SRE incident-response practices emphasize defined coordination and communication roles rather than uncontrolled "everyone debugging everything".
---
# 96. Solo-founder mode
While Ekho has one operator:
one person may hold all roles.
Still structure thinking as:
```text
IC
technical work
communications
timeline
```
When more people join, separate roles.
---
# 97. Incident Commander
Responsible for:
```text
severity
coordination
priorities
delegation
decision making
resolution declaration
```
IC should not become lost debugging one low-level detail when multiple responders exist.
---
# 98. Technical lead
Responsible for:
```text
diagnosis
mitigation
rollback
dependency checks
verification
```
Reports state to IC.
---
# 99. Communications
Responsible for:
```text
status page
user-facing updates
support coordination
internal stakeholder updates
```
Communication should report known facts, impact and current actions.
No speculation.
---
# 100. Scribe
Maintain incident timeline:
```text
detection
alerts
deployments
decisions
mitigations
important observations
resolution
```
This becomes the factual basis for the postmortem.
---
# 101. Incident record
Every SEV0/SEV1 creates one incident record.
Minimum:
```text
incident_id
severity
status
started_at
detected_at
declared_at
mitigated_at
resolved_at
affected_services
affected_users/cohort where known
incident commander
summary
```
---
# 102. One source of truth
During an incident choose one canonical internal incident record/channel.
Do not coordinate independently through:
```text
WhatsApp
Telegram
random DMs
GitHub
five chats
```
with conflicting timelines.
---
# 103. External status communication
Before meaningful public launch Ekho should have an external status mechanism.
For SEV0/SEV1 user-facing outages publish:
```text
Investigating
Identified
Monitoring
Resolved
```
where appropriate.
Do not publish confidential security details during active investigation.
---
# 104. Status update content
Good:
```text
Some users are unable to save application updates.
We identified the issue and are deploying a mitigation.
Existing saved application data is unaffected.
```
Bad:
```text
Supabase is broken again lol
probably fixed soon
```
State only verified information.
---
# 105. Status timestamps
Use explicit absolute timestamps with timezone.
Prefer:
```text
UTC
```
for operational incident records.
User-facing UI may localize timestamps.
---
# 106. Communication cadence
Ekho internal launch baseline:
For unresolved SEV0/SEV1:
**update approximately every 30 minutes when there is meaningful user impact**, even if the update is simply that investigation continues.
This is an Ekho operational policy.
Do not send constant meaningless updates every few minutes.
---
# 107. Do not declare resolved too early
Before:
```text
RESOLVED
```
verify:
```text
SLI recovered
error rate normal
critical operation succeeds
backlog understood
data integrity checked
```
Then continue monitoring.
---
# 108. Incident commands/actions
Safe emergency operations may include:
```text
rollback release
disable feature flag
stop job
pause imports
block dangerous mutation
enable read-only mode
invalidate cache
fail over dependency where supported
```
Detailed degraded-mode behavior belongs to Failure, Recovery & Degraded Mode.
---
# 109. Runbooks
Create runbooks for at least:
```text
database unavailable
auth failure
Storage failure
high 5xx rate
critical deployment regression
pipeline backlog
import publication failure
critical incorrect university data
suspected data exposure
```
Each runbook should be short and executable.
---
# 110. Runbook structure
Every runbook:
```text
Symptoms
Impact
Checks
Safe mitigations
Escalation
Recovery verification
Related dashboards
```
Avoid 30-page incident manuals nobody can use during an outage.
---
# 111. Dependency incidents
Track dependency state independently for:
```text
Supabase
email provider
OAuth providers
observability provider
future AI providers
```
But user impact remains Ekho's responsibility.
---
# 112. Dependency status page is evidence, not diagnosis
If Supabase reports an incident:
record it.
But still verify:
```text
Does this actually explain Ekho impact?
```
Do not stop investigating solely because a vendor status page is yellow.
---
# 113. Observability dependency failure
If Sentry itself is unavailable:
Ekho must not fail.
Observability is out-of-band support infrastructure.
Application code must never require:
```text
Sentry successful
```
before returning a user response.
---
# 114. Telemetry failure
Telemetry exporters should fail safely.
If telemetry backend is unreachable:
```text
application continues
```
wherever possible.
Do not create a production outage because logs cannot be exported.
---
# 115. Monitoring blind spot
If all telemetry disappears unexpectedly:
treat this as an operational issue.
Possible meanings:
```text
traffic disappeared
instrumentation broke
collector broke
deployment broke
service died
```
"No alerts" does not automatically mean healthy.
---
# 116. External synthetic monitor role
Independent synthetic monitoring acts as a backstop when internal telemetry itself fails.
This is why at least one availability signal must originate outside Ekho's application runtime.
---
# 117. Database slow queries
Investigate:
```text
query latency
execution plan
locks
indexes
connection pressure
```
rather than immediately adding infrastructure.
Supabase provides inspection tooling specifically for slow queries, query performance, locks and resource pressure.
---
# 118. Queue health
For asynchronous jobs monitor:
```text
queue depth
oldest pending job age
failure rate
processing duration
retry count
```
Queue size alone may be misleading.
A queue of 1 job stuck for 12 hours may be worse than 500 jobs processed in one minute.
---
# 119. Background-job SLO
Do not define background reliability only as:
```text
worker is running
```
Measure outcome:
```text
jobs completed successfully
within expected time
```
---
# 120. Alert on age, not only count
For time-sensitive admissions pipelines:
```text
oldest_unprocessed_job_age
```
is often more useful than:
```text
queue_length
```
because current data freshness is time-sensitive.
---
# 121. Source-monitoring incident
If monitoring of official university sources stops globally:
this may become SEV1 even though normal website requests still work.
Why:
Ekho may continue serving increasingly stale admissions data.
---
# 122. Wrong-data rollback
When a newly published admissions-data change is discovered to be wrong:
```text
stop propagation
restore last verified safe value
retain history
mark incorrect version superseded
investigate source/review failure
```
Never erase the evidence that the incorrect publication occurred.
---
# 123. Notification safety during data incident
If a wrong canonical change already triggered notifications:
do not silently correct backend data and pretend nothing happened.
Identify affected recipients and handle correction through the Notifications / Support procedures.
---
# 124. Postmortem requirement
Mandatory postmortem for:
```text
every SEV0
every SEV1
security/data-exposure event
significant data corruption
incident consuming ≥20% of a critical error budget
repeated related SEV2 incidents
```
Google's example error-budget policy similarly requires postmortem work when a single outage consumes a material fraction of the budget.
The 20% threshold is an Ekho policy.
---
# 125. Postmortem timing
Target:
```text
within 3 business days after stability
```
for SEV0/SEV1.
Do not write a postmortem while service is still actively broken unless recording incident notes.
This timing is an Ekho process decision.
---
# 126. Blameless postmortem
Postmortems must focus on:
```text
what system conditions allowed the incident
```
not:
```text
who made the mistake
```
Google SRE explicitly recommends blameless postmortems focused on system design rather than individual blame.
---
# 127. Postmortem template
```text
Incident
Severity
Date
Duration
User impact
Detection
Timeline
Trigger
Contributing factors
Root/system causes
What went well
What went poorly
Where we got lucky
Mitigation
Recovery
Action items
SLO/error-budget impact
```
---
# 128. Root cause
Avoid simplistic:
```text
Developer deployed bug
```
Ask:
```text
Why could this bug reach production?
Why did monitoring not catch it earlier?
Why was blast radius large?
Why was rollback difficult?
Why did safe fallback not exist?
```
Focus on system conditions.
---
# 129. Action items
Postmortem action items must have:
```text
owner
priority
measurable completion condition
```
Google SRE specifically identifies ownership, prioritization and measurable end states as characteristics of useful postmortem actions.
---
# 130. No fake action items
Bad:
```text
Be more careful.
```
```text
Test better.
```
Good:
```text
Add automated cross-user RLS regression test.
```
```text
Reject publication when deadline source scope does not match cycle.
```
```text
Add alert when oldest pipeline job exceeds 30 minutes.
```
---
# 131. Action item categories
Classify:
```text
Prevent
Detect
Mitigate
Recover
```
A mature postmortem should not rely only on prevention.
Some failures will still happen.
---
# 132. Do not fix every hypothetical issue
Action items should address:
```text
actual contributing risk
+
meaningful expected reduction
```
Do not create 40 weeks of engineering work from a 2-minute harmless glitch.
Prioritize by risk.
---
# 133. Postmortem follow-up
An unfinished postmortem action is not improvement.
Review open SEV0/SEV1 actions regularly until closed or deliberately rejected with justification.
Google SRE emphasizes concrete follow-up rather than postmortems that simply become documents.
---
# 134. Reliability review
Run a lightweight monthly reliability review.
Review:
```text
SLOs
error budget
SEV0/SEV1 incidents
top Sentry issues
slowest critical operations
database health
pipeline lag
open postmortem actions
```
Avoid weekly reliability meetings when there is nothing meaningful to discuss.
---
# 135. SLO revision
SLOs are not permanent.
Change them when:
```text
real user expectations change
architecture changes
measurement improves
target proves unnecessarily strict
target proves too weak
```
Do not change an SLO retroactively merely because Ekho missed it.
Google SRE recommends iterating SLOs based on operational experience rather than treating the first target as permanent.
---
# 136. Toil
Track recurring operational tasks such as:
```text
manually restarting jobs
clearing stuck queue
rechecking same alert
repairing same import failure
```
Google SRE defines toil as repetitive, predictable operational work that is often automatable and recommends intentionally reducing it.
---
# 137. Automation priority
If an operational task happens repeatedly:
```text
measure frequency
↓
understand cause
↓
automate or eliminate
```
Do not hire humans to repeatedly compensate for a predictable system flaw.
---
# 138. Observability cost control
Telemetry has cost.
Control:
```text
trace sampling
log volume
retention
metric cardinality
unnecessary breadcrumbs
duplicate tools
```
Do not disable critical visibility merely to save a small amount of money.
---
# 139. Log retention
Initial Ekho operational baseline:
```text
ordinary application observability:
~30 days where provider/cost permits
```
Longer-lived security/audit records follow Security / Legal Retention policies.
This is an Ekho starting policy, not a universal requirement.
Supabase log retention itself varies by project pricing plan.
---
# 140. Trace retention
Do not require multi-year distributed-trace retention.
Traces are primarily debugging/performance artifacts.
Use provider retention based on:
```text
cost
traffic
debugging usefulness
privacy
```
Long-term factual history belongs to domain audit/versioning instead.
---
# 141. Metrics retention
Retain enough aggregated metrics to identify:
```text
weekly trends
monthly SLO
seasonal admissions load
regressions
capacity growth
```
Raw high-resolution data does not need indefinite retention.
---
# 142. Availability testing
Production synthetic monitoring should run often enough to detect meaningful outages quickly.
Initial Ekho baseline:
```text
critical public endpoint:
every 1–5 minutes
```
Exact interval depends on monitoring provider and cost.
---
# 143. Do not page from one transient failure
For ordinary availability checks:
require either:
```text
multiple consecutive failures
```
or equivalent confidence logic before paging, unless failure itself is catastrophic/security-critical.
Avoid paging on one random timeout.
---
# 144. Region diversity later
When Ekho becomes globally used, synthetic probes should eventually originate from multiple regions.
Do not infer global availability from one European probe.
This is particularly relevant to Ekho's global-first product thesis.
---
# 145. Startup on-call honesty
Do not document:
```text
24/7 response within 5 minutes
```
unless Ekho can actually staff it.
During solo-founder stage:
* critical alerts must reach the founder;
* escalation path must exist;
* reliability expectations remain internal.
Formal response-time guarantees require actual coverage.
---
# 146. On-call later
As team grows, establish an explicit on-call rotation.
Google SRE defines on-call as a period where an engineer is available to respond to production incidents with appropriate urgency.
No permanent "everyone is always on-call".
---
# 147. Alert channels
Critical alerts should reach a channel that can interrupt:
```text
push / pager / phone-class notification
```
Routine warnings:
```text
issue tracker / email / dashboard
```
Do not rely on someone remembering to check Sentry manually.
---
# 148. Alert testing
Every critical alert must be tested before relying on it.
Test:
```text
rule fires
notification arrives
link works
runbook opens
recovery clears alert
```
An untested alert is not operational protection.
---
# 149. Incident drills
Before significant launch, run basic failure drills:
```text
database unavailable
auth provider failure
bad deployment
critical university deadline incorrect
```
The goal is to verify:
```text
detection
alert
diagnosis
mitigation
communication
```
not theatrical chaos exercises.
---
# 150. Observability as production requirement
A feature is not production-ready if:
```text
it can fail
AND
Ekho has no way to detect meaningful failure
```
Critical features must ship with their relevant telemetry.
---
# 151. Required tests — logging
* [ ] Production logs are structured.
* [ ] Request ID exists.
* [ ] Trace ID propagates where applicable.
* [ ] Environment exists.
* [ ] Release identifier exists.
* [ ] Passwords never logged.
* [ ] Tokens never logged.
* [ ] Private document content never logged.
* [ ] Sensitive query parameters are redacted.
* [ ] Error events can be correlated to requests.
---
# 152. Required tests — tracing
* [ ] Important server request produces trace.
* [ ] Database spans appear.
* [ ] External dependency spans appear where supported.
* [ ] Trace names use route templates.
* [ ] Raw IDs do not create span-name cardinality.
* [ ] Sampling is configurable.
* [ ] telemetry-backend failure does not break user request.
---
# 153. Required tests — metrics
* [ ] Critical operation success/failure metrics exist.
* [ ] Latency exists.
* [ ] Metrics do not contain user/email labels.
* [ ] Metric cardinality remains bounded.
* [ ] Database health metrics available.
* [ ] Queue backlog and age measurable.
* [ ] Import/pipeline failure rates measurable.
---
# 154. Required tests — SLO
* [ ] Each SLO defines numerator.
* [ ] Each SLO defines denominator.
* [ ] Exclusions documented.
* [ ] Measurement source documented.
* [ ] 28-day window works.
* [ ] Current performance visible.
* [ ] Error budget visible.
* [ ] Dependency failures count toward user-facing SLI where appropriate.
* [ ] Invalid user input does not falsely count as server outage.
---
# 155. Required tests — synthetic monitoring
* [ ] Public monitor detects outage.
* [ ] Monitor runs outside application runtime.
* [ ] False one-off failure does not page unnecessarily.
* [ ] Synthetic account is isolated.
* [ ] Synthetic events excluded from product analytics.
* [ ] Synthetic mutation never touches real student data.
---
# 156. Required tests — alerts
* [ ] PAGE alert reaches responsible operator.
* [ ] Every PAGE has runbook.
* [ ] Alert links to useful dashboard/context.
* [ ] Alert clears after recovery.
* [ ] Duplicate symptoms do not create alert storm.
* [ ] Ticket-level issue does not wake operator.
* [ ] User-impact failure pages even when infrastructure metrics look healthy.
---
# 157. Required tests — incidents
Simulate SEV1:
* [ ] Incident declared.
* [ ] Severity assigned.
* [ ] Incident record created.
* [ ] Timeline recorded.
* [ ] Safe mitigation executed.
* [ ] User impact verified.
* [ ] Recovery verified using telemetry.
* [ ] Incident resolved.
* [ ] Postmortem generated.
* [ ] action items assigned.
---
# 158. Required tests — data incident
Publish intentionally wrong test deadline in staging.
Verify:
* [ ] data quality issue detected;
* [ ] affected source/value identifiable;
* [ ] historical version preserved;
* [ ] safe value restorable;
* [ ] downstream impact identifiable;
* [ ] audit history remains;
* [ ] incident process works.
---
# 159. Required tests — observability failure
Disable telemetry exporter.
Expected:
```text
Ekho remains operational
```
* [ ] user request succeeds;
* [ ] exporter failure is bounded;
* [ ] no infinite retry loop;
* [ ] external synthetic monitoring remains available.
---
# 160. P0 failures
Any of these blocks production:
* cross-user data incident cannot be detected/investigated;
* critical application failures produce no telemetry;
* passwords/tokens appear in logs;
* telemetry backend failure causes Ekho outage;
* production and staging telemetry indistinguishable;
* critical user SLO cannot be measured;
* no independent availability monitor exists before meaningful launch;
* critical PAGE alert has no recipient;
* data pipeline can stop silently;
* critical wrong admissions data has no incident path;
* deployment cannot be correlated with new errors;
* incident history disappears after resolution.
---
# 161. Implementation order for Codex
## Stage 1 — Instrumentation
1. Next.js instrumentation entry point.
2. OpenTelemetry conventions.
3. request IDs.
4. trace propagation.
5. environment/release metadata.
6. structured logger.
## Stage 2 — Error monitoring
7. Sentry Next.js integration.
8. production source maps.
9. error capture.
10. sensitive-data scrubbing.
11. release correlation.
## Stage 3 — Metrics
12. HTTP metrics.
13. domain-operation metrics.
14. Supabase Metrics API.
15. pipeline/import metrics.
16. queue metrics.
## Stage 4 — User reliability
17. define SLI queries.
18. 28-day SLO calculations.
19. error-budget calculations.
20. Core Web Vitals field collection.
21. reliability dashboard.
## Stage 5 — Monitoring
22. external synthetic monitor.
23. critical PAGE alerts.
24. ticket alerts.
25. burn-rate alerts where traffic supports them.
26. runbooks.
## Stage 6 — Incident system
27. severity definitions.
28. incident record/template.
29. incident timeline.
30. status communication workflow.
31. postmortem template.
32. action-item tracking.
## Stage 7 — Validation
33. failure drills.
34. telemetry privacy tests.
35. synthetic tests.
36. alert tests.
37. SLO tests.
38. incident simulation.
Do not build a custom observability platform.
---
# 162. Codex implementation constraint
Before implementation Codex must read:
```text
Data Architecture
Data Pipeline
Security & Privacy
Auth & Account Lifecycle
Import & Ingestion
Admin & Data Operations
```
Do not:
* invent duplicate logging systems;
* log sensitive data for easier debugging;
* disable RLS for monitoring;
* add analytics events into observability logs;
* instrument every function indiscriminately.
---
# 163. Definition of Done
Observability, SLO & Incident Response v1 is complete when:
* structured production logging works;
* request/trace correlation works;
* errors are visible with readable source-mapped stacks;
* every event identifies environment/release;
* sensitive data is scrubbed;
* critical operations expose metrics;
* metric cardinality is controlled;
* Supabase infrastructure health is visible;
* pipeline health is visible;
* Core Web Vitals are measured from real users;
* critical user journeys have defined SLIs;
* internal SLOs and error budgets exist;
* an external synthetic monitor exists;
* paging is based primarily on user impact;
* every PAGE has a runbook;
* critical alerts reach a real operator;
* SEV0–SEV3 classification exists;
* major incidents have a structured timeline;
* critical admissions-data failures count as incidents;
* postmortems are blameless and actionable;
* observability provider failure cannot take down Ekho;
* all P0 tests pass.
---
# 164. Final invariant
Ekho production operations must work like:
```text
User impact occurs
↓
Signal detects it
↓
Alert reaches right person
↓
Trace/logs show where
↓
Safe mitigation
↓
Service recovers
↓
Cause understood
↓
System improved
```
Never:
```text
User complains
↓
we had no idea
↓
open Supabase randomly
↓
refresh pages
↓
restart something
↓
hope it is fixed
```
For Ekho, reliability means both:
```text
SYSTEM WORKS
+
IMPORTANT ADMISSIONS INFORMATION REMAINS TRUSTWORTHY
```
---
# 165. Primary authority sources
This specification was checked primarily against:
1. **OpenTelemetry official documentation/specification** — traces, metrics, logs, context propagation and vendor-neutral instrumentation.
2. **Google Site Reliability Engineering / SRE Workbook** — SLOs, error budgets, burn-rate alerting, incident response and postmortems.
3. **NIST SP 800-61 Rev.3 (2025)** — current incident-response guidance integrated with CSF 2.0.
4. **Supabase Telemetry documentation** — logs, metrics, reports and Postgres health telemetry.
5. **Next.js official documentation** — current OpenTelemetry and instrumentation support.
6. **Sentry official Next.js documentation** — errors, traces, releases, source maps and sensitive-data scrubbing.
7. **Prometheus official best practices** — metrics and cardinality management.
8. **Google Web Vitals guidance** — LCP, INP, CLS and real-user performance thresholds.
