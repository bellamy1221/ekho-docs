# Admin & Data Operations v1
**Status:** LOCKED
**Scope:** internal administration, review, publication and maintenance of Ekho institutional data
**Stack:** Next.js + Supabase Auth + PostgreSQL + Supabase Storage where required
**Depends on:** Data Standard, Data Architecture, Data Pipeline, Import & Ingestion, Security & Privacy, Auth & Account Lifecycle
**Research verified:** 12 August 2026
---
# 1. Goal
Ekho Admin is the internal control plane for institutional data.
It must let authorized operators safely:
* inspect universities;
* inspect programs;
* inspect requirements;
* inspect deadlines;
* inspect costs;
* inspect financial aid;
* inspect sources;
* process imports;
* review proposed changes;
* resolve conflicts;
* correct incorrect data;
* verify stale data;
* archive records;
* merge true duplicates;
* inspect history;
* see exactly who changed what;
* recover from operational mistakes.
The primary operational question is:
> **What data needs attention, why, and what is the safest next action?**
---
# 2. Admin is not the consumer product
Ekho Admin must be completely separated conceptually from the student experience.
Student application:
```text
research
applications
requirements
tasks
documents
financial aid
```
Admin:
```text
institutional data
sources
imports
review
verification
publication
operations
audit
```
Do not mix admin tools into normal student settings/navigation.
---
# 3. Admin URL
Recommended internal route:
```text
/admin
```
Subsections:
```text
/admin
/admin/universities
/admin/programs
/admin/imports
/admin/review
/admin/sources
/admin/issues
/admin/audit
```
Optional later:
```text
/admin/operations
/admin/users
/admin/feature-flags
```
Do not create public indexable admin pages.
---
# 4. Core admin principle
The admin UI must manipulate **canonical Ekho entities**, not arbitrary blobs.
Flow:
```text
Admin action
→ authorization
→ validation
→ domain operation
→ database transaction
→ version/audit record
→ derived systems update
```
Never:
```text
Admin form
→ arbitrary SQL
```
---
# 5. Canonical Data Standard wins
Admin must use exactly the entities and relationships defined by:
**Data Standard v1**
It must not create:
* admin-only versions of university fields;
* duplicate deadline structures;
* separate financial-aid schemas;
* arbitrary JSON fields because they are easier to edit.
Admin is an interface over the canonical data model.
---
# 6. Authority boundaries
Admin operates on:
### Institutional/public data
Examples:
* universities;
* programs;
* requirements;
* deadlines;
* tuition;
* costs;
* scholarships;
* financial aid rules;
* application platforms;
* official sources.
Admin must **not casually operate on private student data**.
Student data remains governed separately by:
* RLS;
* Security & Privacy;
* Support & User Operations;
* legal/privacy procedures.
---
# 7. Administrative access principle
Follow:
**deny by default + least privilege.**
OWASP explicitly recommends denying authorization by default and granting only the minimum privileges required for a role.
No account receives admin capability merely because it is:
```text
authenticated
```
---
# 8. Admin roles
Design for these roles:
```text
data_viewer
data_editor
data_publisher
admin
```
### `data_viewer`
Can:
* view canonical data;
* view sources;
* view imports;
* view issues;
* view history.
Cannot modify anything.
### `data_editor`
Can additionally:
* create draft corrections;
* create/import candidate data;
* resolve non-destructive review items;
* edit staged data.
Cannot publish.
### `data_publisher`
Can additionally:
* approve/publish verified changes;
* archive records;
* execute controlled high-impact operations.
### `admin`
Can additionally:
* manage internal roles;
* perform exceptional administrative operations;
* access security/configuration operations explicitly assigned to Admin.
This is an Ekho role model decision.
---
# 9. Initial solo-founder mode
At launch, there may be only one internal operator.
That does **not** justify designing:
```text
authenticated user = superadmin
```
Implement the role system correctly from the start even if one account initially has:
```text
admin
```
This prevents a future permission redesign when the team grows.
---
# 10. Admin role storage
Do not store authorization in user-editable:
```text
user_metadata
```
Supabase explicitly warns that `user_metadata` is editable by users and must not be relied upon for security-sensitive authorization or RLS.
Use trusted server-controlled role data.
Acceptable architecture:
```text
private.admin_memberships
```
or controlled role/custom claims backed by trusted database state.
---
# 11. Supabase RBAC
Supabase supports role-based access control through custom JWT claims combined with RLS.
However:
JWT claims can be stale until the session/token refreshes.
Therefore high-impact administrative actions must not rely only on a cached browser role claim.
For critical actions:
```text
verified identity
+
current server-side permission check
```
---
# 12. Admin MFA
All privileged admin accounts must use MFA before production data access.
Baseline:
**TOTP MFA**
Supabase supports TOTP MFA and distinguishes authentication assurance levels such as AAL1/AAL2.
For operations such as:
* changing admin permissions;
* destructive merge;
* bulk archive;
* emergency corrections;
require an appropriately recent strong admin session.
---
# 13. No shared admin accounts
Prohibited:
```text
admin@ekho.club
password shared by team
```
Every operator must have an individual identity.
Auditability requires:
```text
action
→ specific actor
```
not:
```text
action
→ generic admin
```
---
# 14. Admin account lifecycle
Administrative access must support:
```text
invited
active
suspended
revoked
```
Removing a person from the team must remove their administrative authorization immediately at the trusted authorization layer.
Do not merely hide `/admin`.
---
# 15. Admin authorization is server-side
Every administrative mutation must verify authorization on the server.
Frontend UI visibility is not authorization.
Bad:
```text
if role === "admin"
  show button
```
then endpoint accepts anyone.
Correct:
```text
UI role check
+
server role check
+
database authorization
```
---
# 16. RLS remains active
Any admin-accessible table exposed through the Supabase Data API must retain appropriate RLS.
Supabase requires RLS for exposed tables and describes RLS as defense in depth when Supabase Auth is used.
Do not disable RLS globally because an admin panel exists.
---
# 17. Private operational schema
Internal-only operational data should preferably live outside the publicly exposed API schema.
Recommended conceptual schema:
```text
private.admin_memberships
private.data_issues
private.data_reviews
private.data_versions
private.admin_audit
private.import_jobs
private.import_issues
```
Public consumer data remains separately exposed according to Data Architecture.
---
# 18. Secret/service key
Supabase secret/service credentials bypass normal RLS and must never reach the browser.
Therefore:
```text
Admin browser
→ Next.js trusted server
→ privileged operation
```
Never:
```text
Admin browser
→ service_role / secret
```
---
# 19. Core admin objects
Admin must provide operational access to:
```text
Universities
Programs
Admission cycles
Application rounds
Requirements
Deadlines
Tests
Qualifications
Documents
Tuition/cost records
Financial aid policies
Scholarships
Application platforms
Sources
Imports
Data issues
Change history
```
Exact naming follows Data Standard.
---
# 20. University admin page
Each university should have one operational overview.
Example:
```text
MIT
Status: Published
Data health: Good
Current cycle: 2027
Programs: 58
Critical issues: 1
Warnings: 3
Last verified: 4 days ago
[Review issues]
[New import]
```
Sections:
```text
Overview
Programs
Admissions
Costs & Aid
Sources
Issues
History
```
Do not duplicate the entire public university UX.
Admin shows operational state.
---
# 21. Operational entity states
Institutional entities should have lifecycle state where appropriate:
```text
draft
published
inactive
archived
```
Specialized entities may additionally support canonical states such as:
```text
discontinued
superseded
```
where Data Standard defines them.
Never use physical deletion as the normal lifecycle mechanism.
---
# 22. Draft
`draft` means:
* exists internally;
* incomplete or unapproved;
* not usable as current public canonical data.
Draft data must not accidentally appear:
* in university search;
* in student requirements;
* in SEO pages;
* in personalization.
---
# 23. Published
`published` means:
* passed required validation;
* has acceptable evidence where required;
* is canonical;
* may affect student-facing behavior.
Publishing is a meaningful operation and must be audited.
---
# 24. Inactive
Use `inactive` where something is temporarily or currently not offered/available but historical identity should remain.
Do not delete it merely to remove it from active discovery.
---
# 25. Archived
`archived` means:
* no longer active in current operations;
* retained for history/audit/reference;
* excluded from normal public/current workflows unless explicitly requested.
Archive ≠ delete.
---
# 26. Physical deletion
Physical deletion of canonical university/program/admissions records must be rare.
Allowed primarily for:
* obvious test data;
* completely invalid accidental objects;
* records created but never legitimately published;
* legally/security-required removal where applicable.
Ordinary historical change should use lifecycle/versioning instead.
---
# 27. No direct canonical free editing by default
The preferred mutation model:
```text
current canonical value
→ proposed change
→ validate
→ diff
→ publish
```
rather than immediately rewriting production data on every keystroke.
This is especially important for critical fields.
---
# 28. Critical fields
Critical changes include at least:
* application deadline;
* opening date;
* academic requirements;
* qualification requirement;
* test policy;
* minimum test score;
* required document;
* application platform;
* application fee;
* tuition;
* major mandatory fees;
* financial aid policy;
* international aid eligibility;
* scholarship eligibility;
* scholarship amount;
* aid/scholarship deadline.
Changes to these require:
```text
source
+
scope
+
review
```
---
# 29. Low-risk fields
Examples:
* display-name typo;
* public description typo;
* source page title;
* formatting normalization.
These may use a lighter workflow when no factual meaning changes.
Still record the operation in history.
---
# 30. Data change object
Every proposed meaningful change should conceptually capture:
```text
entity
field/path
old_value
new_value
source
reason
actor
created_at
```
Publication adds:
```text
published_by
published_at
```
---
# 31. Change reason
Require structured reasons for important manual changes.
Suggested:
```text
official_source_update
correction
cycle_rollover
source_conflict_resolution
duplicate_resolution
program_discontinued
data_normalization
manual_research
other
```
If `other`, allow short note.
---
# 32. Source is mandatory for factual correction
An operator must not be able to change:
```text
IELTS 7.0 → 6.5
```
solely because:
> "I think this is correct."
Critical factual updates must reference acceptable provenance according to Data Standard / Import specification.
---
# 33. Manual edit does not bypass provenance
Admin manual editing must use the same evidence standards as imported data.
Do not create two quality levels:
```text
AI import → strict sources
manual admin edit → anything allowed
```
Both must converge on the same canonical validation rules.
---
# 34. Source management
Admin source objects need:
```text
URL
publisher
source type
title
scope
retrieved_at
source_updated_at if known
verification state
last checked
```
following the source model already established.
---
# 35. Source operational states
Recommended:
```text
active
unreachable
redirected
superseded
archived
needs_reverification
```
A temporary HTTP failure does not automatically mean the information is false.
---
# 36. Broken source workflow
When a source becomes unavailable:
```text
source unreachable
→ create/reopen issue
→ find authoritative replacement
→ review affected claims
→ preserve prior history
```
Never:
```text
HTTP 404
→ delete all derived data
```
---
# 37. Source replacement
When an official university restructures URLs:
```text
old source
→ superseded
new source
→ active
```
Preserve the relationship showing where historical information originally came from.
---
# 38. Source verification
Operator should be able to open:
```text
claim
→ supporting source
→ locator
```
in as few interactions as possible.
The objective is fast verification, not reading entire institutional websites repeatedly.
---
# 39. Data issue system
Create one canonical internal issue system for data problems.
Examples:
```text
source_unreachable
source_conflict
missing_source
missing_current_cycle
potential_duplicate
stale_data
invalid_scope
impossible_value
manual_report
pipeline_failure
```
Do not create separate unrelated issue tables for each feature.
---
# 40. Issue states
```text
open
in_review
blocked
resolved
dismissed
```
Optional later:
```text
assigned
```
---
# 41. Issue severity
Use:
### `critical`
Likely to materially mislead an applicant.
Examples:
* wrong deadline;
* wrong eligibility;
* wrong required exam;
* incorrect financial aid eligibility.
### `high`
Important but less immediately destructive.
### `medium`
Data completeness/freshness problem.
### `low`
Minor metadata/quality issue.
---
# 42. Priority is not only severity
Issue priority should eventually consider:
```text
severity
×
number of potentially affected users
×
time sensitivity
```
Example:
A deadline error affecting 500 active applications tomorrow is operationally more urgent than a missing low-impact field at an obscure university.
This is an Ekho prioritization rule.
---
# 43. Review queue
Primary admin workspace:
```text
/admin/review
```
It should aggregate:
* import warnings;
* source conflicts;
* stale critical data;
* changed critical values;
* potential duplicates;
* correction requests;
* monitoring-detected changes.
One queue is better than operators checking five systems manually.
---
# 44. Review queue ordering
Default ordering:
```text
1. Critical + deadline-sensitive
2. Critical changes
3. High-severity conflicts
4. Current-cycle stale data
5. Other warnings
```
Do not order simply by creation date.
---
# 45. Review queue filters
Useful filters:
```text
Critical
Deadlines
Requirements
Financial Aid
Sources
Imports
Duplicates
Stale
Country
University
```
Avoid dozens of filters until actual operations require them.
---
# 46. Review item
A review item should answer immediately:
```text
What changed?
Why is it flagged?
Who/what generated it?
Who may be affected?
What is current?
What is proposed?
What is the official evidence?
What actions are allowed?
```
---
# 47. Review actions
Depending on issue:
```text
Approve
Reject
Edit
Resolve
Dismiss
Archive
Request re-research
```
Never show destructive actions when they do not apply.
---
# 48. Conflict resolution
When two authoritative sources disagree, show both.
Example:
```text
Current value:
2027-01-02
Source A:
2027-01-02
Source B:
2027-01-05
```
Resolution must capture:
```text
selected outcome
reason
supporting source
actor
timestamp
```
Never silently overwrite one with the other.
---
# 49. Conflict specificity rule
Prefer the authoritative source that actually matches the required scope.
Example:
```text
University general page
vs
official Computer Science international-applicant page
```
The more specific authoritative source may control the scoped record.
But the system must not infer this automatically where scope is ambiguous.
---
# 50. Duplicate detection
Potential duplicate objects must enter review rather than auto-merge.
University matching factors may include:
```text
official domain
canonical name
aliases
country
city
external identifier
```
Program matching additionally considers canonical program dimensions.
---
# 51. Merge is high risk
Merge means:
```text
Entity A
+
Entity B
→ one canonical identity
```
This may affect:
* URLs;
* foreign keys;
* applications;
* saved universities;
* search;
* analytics;
* provenance.
Therefore merging must be an explicit controlled operation.
---
# 52. Merge preview
Before merging show:
```text
surviving entity
retired entity
references to migrate
conflicting fields
sources
affected public URLs
affected user-linked records count
```
No one-click blind merge.
---
# 53. Merge invariant
A merge must preserve:
* canonical surviving ID;
* all valid references;
* source history;
* audit history;
* aliases where useful;
* redirects where SEO/public URLs are affected.
It must not duplicate or orphan user relationships.
---
# 54. Merge transaction
High-impact relational operations must execute transactionally.
PostgreSQL transactions provide the all-or-nothing boundary needed for multi-table operations, while database constraints continue enforcing relational integrity.
If merge fails halfway:
```text
ROLLBACK
```
---
# 55. Concurrency
Two admins may eventually edit the same record.
Ekho must detect stale edits.
Conceptual:
```text
Admin A loads version 12
Admin B publishes version 13
Admin A attempts publish
→ VERSION_CONFLICT
```
Require Admin A to review against version 13.
---
# 56. Do not use last-write-wins blindly
For factual institutional data:
```text
last writer wins
```
is unsafe.
Use optimistic version checks for normal editing.
For operations requiring strict serialization, appropriate database transactions/row locking may be used.
PostgreSQL supports row-level locking such as `SELECT ... FOR UPDATE` to prevent concurrent modifications to selected rows during a transaction.
---
# 57. Database constraints remain final defense
Admin validation does not replace:
* primary keys;
* unique constraints;
* foreign keys;
* `NOT NULL`;
* appropriate `CHECK` constraints.
PostgreSQL provides these integrity constraints at the database layer.
---
# 58. Canonical versioning
Every meaningful publication must produce a version/history record.
Conceptually:
```text
entity_id
entity_type
version
change_id
published_at
published_by
```
Do not overwrite history and leave only the current value.
---
# 59. Field history
For critical information, admin should eventually be able to see:
```text
Jan 2 → Jan 5
12 Aug 2026
Official admissions page
Changed by import #123
Published by George
```
This history directly supports Ekho trust and Live Admissions Updates.
---
# 60. Audit trail vs version history
These are different.
### Version history
Answers:
> What factual data changed?
### Audit trail
Answers:
> What administrative action occurred?
Keep both concepts.
---
# 61. Application audit log
Audit important admin actions:
```text
admin_login_security_event
import_submitted
import_approved
import_rejected
record_created
record_updated
record_published
record_archived
issue_resolved
source_replaced
duplicate_merged
role_changed
bulk_operation_started
bulk_operation_completed
```
---
# 62. Audit event fields
Conceptual minimum:
```text
id
event_type
actor_user_id
target_type
target_id
timestamp
request_id
metadata
```
Metadata must be constrained and must not become a dump of sensitive data.
---
# 63. Database audit
Application-level audit should be the primary understandable business audit.
For additional database/security auditing, Supabase supports `pgAudit`, which extends PostgreSQL logging for selective audit tracking.
Do not use pgAudit as a replacement for domain-level change history.
---
# 64. Do not log everything
Supabase specifically cautions that indiscriminate pgAudit logging can produce database load and excessive noise; audit only what is operationally/security relevant.
Therefore do not enable indiscriminate logging of every `SELECT`.
---
# 65. Audit immutability principle
Normal admin UI must not allow:
```text
edit old audit event
delete audit event
```
Corrections should create new events.
---
# 66. Audit privacy
Do not store in ordinary audit metadata:
* passwords;
* access tokens;
* refresh tokens;
* OAuth secrets;
* full private student documents;
* unnecessary personal data.
Audit does not justify collecting more sensitive data.
---
# 67. Data correction workflow
Operator discovers incorrect canonical fact:
```text
open record
→ propose correction
→ attach evidence
→ validation
→ preview change
→ publish
→ history
→ downstream update event
```
Do not edit production rows through Supabase Dashboard as a normal workflow.
---
# 68. Emergency correction
For severe errors such as:
> Ekho displays tomorrow's deadline incorrectly
allow an accelerated correction workflow.
Still require:
```text
authorization
source/evidence
validation
audit
```
It may skip nonessential waiting/review steps when necessary.
---
# 69. Emergency corrections are visible
Mark:
```text
priority = emergency
```
so the incident can later be reviewed.
Do not make emergency mode an invisible bypass.
---
# 70. University archive workflow
Before archive:
show:
```text
active programs
active applications referencing university
public URLs
current sources
current unresolved issues
```
Archiving must not destroy historical/user records.
---
# 71. Program discontinuation
When program closure is verified:
```text
program
→ discontinued/inactive
```
according to Data Standard.
Existing user applications remain meaningful historical objects.
Never delete their application because the program is no longer offered.
---
# 72. Current cycle rollover
Admissions years must not be handled by overwriting last year's data.
Process:
```text
new cycle discovered
→ create new cycle
→ import/research current facts
→ review
→ publish
→ previous cycle remains historical
```
---
# 73. Cycle rollover queue
Before the next admissions season, operations should identify:
```text
universities missing next-cycle data
universities with only prior-cycle deadlines
programs with unresolved cycle scope
aid pages not yet updated
```
Do not silently carry forward old dates.
---
# 74. Data freshness
Admin needs freshness visibility.
At minimum distinguish:
```text
verified/current
due_for_review
stale
unknown
```
Exact time thresholds must come from the Data Standard/Data Pipeline by data type.
Do not impose one arbitrary freshness interval on every field.
---
# 75. Critical data freshness
Fields such as:
* deadlines;
* test policies;
* tuition;
* aid deadlines;
may require more aggressive freshness monitoring than slow-changing fields such as:
* university city;
* official name.
Freshness policy should be field/category-aware.
---
# 76. Reverification
Reverification must mean:
> an operator or trusted pipeline reconfirmed the claim against an acceptable source.
It must not mean:
> a cron job updated `verified_at`.
Never refresh timestamps without actual verification.
---
# 77. Data health
University operational summary may compute a data-health state from deterministic factors such as:
```text
critical unresolved issues
missing current cycle
unsourced critical values
stale critical values
conflicts
broken sources
```
Do not use an opaque AI-generated "93% accurate" score.
---
# 78. Health states
Prefer understandable states:
```text
healthy
needs_attention
critical
incomplete
```
with explicit reasons.
Example:
```text
Needs attention
• 2027 tuition not verified
• 1 official source unreachable
```
---
# 79. Admin search
Admin search must support at least:
```text
university name
alias
program
domain
country
source URL
Ekho ID
```
Exact search implementation follows Search specification.
Admin search ranking need not equal consumer search ranking.
---
# 80. Canonical ID lookup
Every core object should be directly searchable/openable by canonical ID.
This is important for:
* debugging;
* support;
* audit;
* imports;
* issue references.
---
# 81. Bulk operations
Do not build a generic:
```text
Select all → Edit anything
```
system.
Bulk operations should be explicit domain actions.
Examples later:
```text
mark selected sources for reverification
archive selected obsolete draft imports
assign country review batch
```
---
# 82. Dangerous bulk operations
Actions such as:
```text
archive 500 programs
change current cycle on 300 universities
delete records
```
require:
* explicit privilege;
* preview;
* affected-count display;
* confirmation;
* audit event;
* transaction/job tracking;
* recovery strategy.
---
# 83. Confirmation UX
Do not use confirmation dialogs for every harmless action.
Use strong confirmation only where consequences justify it.
For destructive/high-impact operation show exact impact:
```text
Archive 42 programs?
17 are referenced by active applications.
```
not:
```text
Are you sure?
```
---
# 84. Typed destructive confirmation
For exceptionally high-risk actions, require an explicit value such as:
```text
MERGE MIT-OLD INTO MIT
```
or the target entity name.
Do not use this friction for normal editing.
---
# 85. Asynchronous operations
Long-running tasks should become jobs.
Examples:
```text
large import
bulk reverification
search rebuild
derived-data recomputation
large merge
```
States:
```text
queued
running
succeeded
failed
cancelled
```
Do not hold a browser request open for long-running background work.
---
# 86. Job retry
Retry only operations known to be safe/idempotent.
Never blindly retry a partially destructive operation unless the operation is designed to be idempotent or transactionally safe.
---
# 87. Derived systems
Canonical PostgreSQL data is the primary truth.
Derived systems may include:
```text
search index
cache
SEO representation
personalization derivation
live-update feed
```
An admin edit should mutate canonical data first.
---
# 88. Derived sync
After canonical publication:
```text
canonical commit
→ enqueue derived updates
```
If search/cache refresh fails:
```text
canonical change remains valid
derived system → retry_required
```
Do not reverse correct institutional data solely because an index update failed.
---
# 89. Admin must expose derived sync failure
Example:
```text
University data published
Search sync failed — retrying
```
Do not silently leave systems inconsistent.
---
# 90. Manual retry
Authorized admin may trigger retry for a failed derived operation where operation semantics are safe.
Avoid exposing arbitrary internal job execution.
---
# 91. Data pipeline integration
Pipeline events such as:
```text
official page changed
source disappeared
new source content detected
```
should create:
```text
candidate change / issue
```
not directly mutate canonical production data unless a later separately approved auto-publication policy exists.
---
# 92. Import integration
Import & Ingestion remains the primary bulk-entry mechanism.
Admin must provide:
```text
New Import
Validate
Review
Publish
History
```
Do not rebuild university-entry forms that duplicate Import logic.
---
# 93. Manual editor scope
Manual UI editing exists for:
* small corrections;
* operational state changes;
* conflict resolution;
* source replacement.
For entering a whole university:
```text
use Import & Ingestion
```
---
# 94. User-submitted correction future path
Later Ekho may accept:
```text
"Report incorrect information"
```
from students.
Those reports must create an internal data issue.
They must never directly modify canonical data.
---
# 95. Report is not evidence
A student saying:
> TOEFL requirement is wrong
is a signal to investigate.
It is not authoritative provenance.
Operator must verify against an acceptable source before publication.
---
# 96. Admin comments
Allow short internal notes only where useful.
Do not turn Ekho Admin into:
* Slack;
* Jira;
* social feed;
* giant collaboration suite.
For v1, structured issue state + concise resolution note is enough.
---
# 97. Assignments
Solo-founder v1 does not need sophisticated assignment/workload management.
Architecture may support:
```text
assigned_to nullable
```
but don't build team-management complexity prematurely.
---
# 98. Admin notifications
Admin operational alerts should be reserved for:
* critical data failure;
* urgent deadline conflict;
* pipeline outage affecting freshness;
* dangerous operation failure.
Routine warnings stay inside review queue.
Avoid internal notification spam.
---
# 99. Data export
Authorized operators should eventually be able to export canonical institutional data for:
* analysis;
* backup/debugging;
* migration;
* quality review.
Export must respect:
* authorization;
* private/public boundaries;
* schema version;
* sensitive-data restrictions.
Do not combine private student data into routine university-data exports.
---
# 100. Database backups are not version history
Supabase provides database backups, and paid configurations may add Point-in-Time Recovery.
But backups are for disaster recovery.
They are not a replacement for:
* admin audit;
* entity history;
* correction workflow;
* rollback semantics.
---
# 101. Storage backup caveat
Supabase documentation notes that database backups cover the Postgres database itself; Storage objects are not simply equivalent to database backup contents.
Storage recovery policy belongs to Failure/Recovery and Security specifications.
Do not assume database restore alone restores every stored file.
---
# 102. Undo
Normal low-risk editing may later expose:
```text
Revert this change
```
but reversal must create a **new version**.
Never erase the original historical change.
---
# 103. Revert safety
Before revert:
```text
old version
vs
current version
```
must be checked.
If later changes depend on current state, blind rollback must be blocked.
---
# 104. Operational metrics
Track at minimum:
```text
open_data_issues
critical_open_issues
imports_waiting_review
imports_failed
critical_changes_waiting_review
stale_universities
broken_sources
conflicting_sources
average_review_time
average_issue_resolution_time
```
These are internal operational metrics.
---
# 105. Data-quality metrics
Useful product-quality metrics:
```text
% active universities with current-cycle coverage
% critical fields with accepted provenance
% critical data currently verified
% current programs with valid source
% unresolved source conflicts
% universities containing critical issues
```
Do not collapse all of this into a single unexplained score.
---
# 106. Operator performance metrics
Do **not** optimize operators for:
```text
number of changes published
```
That incentivizes speed over correctness.
Optimize the process for:
```text
accuracy
verification quality
resolution time for important issues
```
---
# 107. Admin homepage
Do not build a giant analytics dashboard.
Homepage should answer:
> What needs attention now?
Example:
```text
Admin
3 Critical issues
12 Changes waiting review
8 Sources need reverification
2 Imports failed
Recent activity
```
One primary action:
**Review critical issues**
---
# 108. Empty state
Healthy system:
```text
No critical issues
Everything important is up to date.
```
Do not invent tasks merely to make the dashboard look active.
---
# 109. Admin errors
Admin errors must expose enough detail to debug without leaking secrets.
Example:
```text
IMPORT_VERSION_CONFLICT
The university changed after this review was created.
Refresh the diff and review again.
```
not:
```text
Something went wrong
```
Detailed API/error conventions will be defined separately in API & Error Contract.
---
# 110. Database function security
Use `SECURITY INVOKER` by default.
When privileged `SECURITY DEFINER` functions are genuinely required:
* restrict execution;
* explicitly control `search_path`;
* expose only narrowly scoped operations.
Supabase recommends `security invoker` by default and explicitly describes revoking function execution permissions for protected database functions.
---
# 111. Function grants
Do not assume creating a database function automatically makes it safe.
Explicitly review:
```text
EXECUTE permissions
anon
authenticated
admin/server role
```
Admin RPCs must not be callable by ordinary student accounts.
---
# 112. Admin database changes
Schema/data migrations remain part of Development Workflow.
Do not add:
```text
Admin → Run SQL
```
to Ekho.
Production schema mutation happens through reviewed migrations, not web-admin convenience controls.
---
# 113. No direct secret management
Ekho Admin must not expose:
* Supabase service keys;
* OAuth secrets;
* SMTP credentials;
* database passwords;
* signing secrets.
Secrets belong to infrastructure configuration.
---
# 114. Admin security advisors
Operational process should periodically review Supabase's Database Security Advisor for issues such as incorrectly configured RLS. Supabase provides database security/performance advisors specifically for checks including RLS setup.
This supplements, not replaces, tests.
---
# 115. Staging vs production
Admin must clearly indicate environment.
Example:
```text
STAGING
```
or:
```text
PRODUCTION
```
Do not make production and staging visually indistinguishable in internal operations.
---
# 116. Production data principle
Do not casually copy private production student data into local development/staging.
Institutional public data may be replicated when required.
Private data handling follows Security & Privacy.
---
# 117. Seed data
Seed operations must distinguish:
```text
test fixture
seed institutional data
production canonical data
```
Never allow automated development seed scripts to overwrite production canonical data.
---
# 118. Admin feature flags
Future experimental admin operations may be hidden behind server-controlled feature flags.
But authorization remains separate:
```text
feature enabled
≠
user authorized
```
Feature flag rules will be defined in Feature Flags & Runtime Configuration.
---
# 119. Required tests — authorization
* [ ] Student cannot open protected admin API.
* [ ] Anonymous user cannot access admin API.
* [ ] `data_viewer` cannot mutate.
* [ ] `data_editor` cannot publish.
* [ ] `data_publisher` can perform approved publish operations.
* [ ] Revoked admin loses privileged access.
* [ ] `user_metadata` modification cannot grant admin.
* [ ] Browser never receives Supabase secret/service key.
* [ ] Admin mutation checks permission server-side.
* [ ] Admin RLS/function grants behave correctly.
---
# 120. Required tests — publishing
* [ ] Invalid factual change cannot publish.
* [ ] Critical change without evidence cannot publish.
* [ ] Valid change creates new version.
* [ ] Old value remains in history.
* [ ] Actor recorded.
* [ ] Source recorded.
* [ ] Derived sync starts only after canonical commit.
* [ ] Derived failure does not corrupt canonical data.
---
# 121. Required tests — concurrency
Use Admin A and Admin B.
* [ ] Both load same version.
* [ ] A publishes.
* [ ] B's stale publish is rejected.
* [ ] B sees updated diff.
* [ ] No silent last-write-wins occurs.
* [ ] concurrent merge/update cannot create broken relationships.
---
# 122. Required tests — issues
* [ ] Pipeline warning creates issue.
* [ ] Conflict appears in review queue.
* [ ] Issue can be resolved.
* [ ] Resolution requires reason where applicable.
* [ ] Dismissed issue retains history.
* [ ] Critical issue sorts above routine issue.
* [ ] Broken source does not automatically delete facts.
---
# 123. Required tests — archive
* [ ] Archiving removes entity from appropriate current discovery.
* [ ] Historical record remains.
* [ ] Existing application relationship remains valid.
* [ ] Unarchive works where allowed.
* [ ] Audit event exists.
* [ ] Archive cannot silently cascade-delete children/user data.
---
# 124. Required tests — merge
* [ ] Duplicate candidates do not auto-merge.
* [ ] Merge preview shows affected references.
* [ ] Surviving ID remains stable.
* [ ] Foreign references migrate correctly.
* [ ] User-linked records remain intact.
* [ ] History remains intact.
* [ ] public alias/redirect strategy works.
* [ ] forced failure produces rollback.
* [ ] merge operation audited.
---
# 125. Required tests — audit
* [ ] Critical admin mutation generates audit event.
* [ ] Actor is correct.
* [ ] Target is correct.
* [ ] timestamp exists.
* [ ] old audit records cannot be edited through normal admin UI.
* [ ] sensitive secrets do not appear.
* [ ] pgAudit configuration does not indiscriminately log unnecessary traffic.
---
# 126. Required tests — data quality
* [ ] Current-cycle missing data appears.
* [ ] Unsourced critical value appears.
* [ ] stale critical value appears.
* [ ] source conflict appears.
* [ ] broken source appears.
* [ ] verified data is not falsely marked stale.
* [ ] `verified_at` cannot be refreshed without verification operation.
---
# 127. P0 failures
Any of these blocks production:
* student can access admin mutation endpoint;
* user-editable metadata grants admin;
* service secret reaches browser;
* admin action bypasses canonical validation;
* critical factual change can publish without provenance;
* stale concurrent edit silently overwrites current data;
* merge corrupts foreign references;
* archive/delete removes private user application data unexpectedly;
* audit trail can be silently rewritten;
* canonical historical values disappear after update;
* current-cycle data is replaced by prior-cycle values without evidence;
* production admin allows arbitrary SQL execution;
* broken source automatically deletes valid canonical data.
---
# 128. Implementation order for Codex
Implement in stages.
## Stage 1 — Authorization
1. admin membership model;
2. role enum;
3. server authorization helper;
4. admin route protection;
5. MFA requirement;
6. RLS/function privileges;
7. authorization tests.
## Stage 2 — Operational model
8. issue model;
9. issue lifecycle;
10. change/version model;
11. audit model;
12. source operational state;
13. environment separation.
## Stage 3 — Core admin
14. admin homepage;
15. university lookup;
16. university operational overview;
17. program overview;
18. source inspection;
19. issue inspection.
## Stage 4 — Review
20. unified review queue;
21. review priority;
22. critical-change diff;
23. source evidence view;
24. approve/reject/edit flow;
25. conflict resolution.
## Stage 5 — Operations
26. correction workflow;
27. archive;
28. program discontinuation;
29. reverification;
30. duplicate detection;
31. safe merge.
## Stage 6 — Reliability
32. concurrency/version protection;
33. derived sync status;
34. retry-safe jobs;
35. admin audit;
36. pgAudit configuration if required;
37. comprehensive E2E/security tests.
Do not build team productivity features before these fundamentals work.
---
# 129. Codex implementation constraint
Before implementation, Codex must read:
```text
Data Standard
Data Architecture
Data Pipeline
Security & Privacy
Auth & Account Lifecycle
Import & Ingestion
```
Do not redesign those specifications during Admin implementation.
If conflict exists:
```text
canonical Data Standard
+
Security rules
```
take priority until an explicit specification revision is made.
---
# 130. Definition of Done
Admin & Data Operations v1 is complete only when:
* admin is completely separate from student UI;
* admin access is explicit and server-authorized;
* user-editable metadata cannot grant privilege;
* privileged accounts use MFA;
* role model exists;
* critical institutional data has safe review/publish flow;
* imports feed the same review system;
* manual edits obey the same provenance rules;
* university/program/source operational pages exist;
* unified issue/review queue exists;
* stale/conflicting/broken data is visible;
* archive preserves history;
* duplicate merge is controlled;
* concurrency cannot silently overwrite newer edits;
* every important publication creates history;
* every important admin operation creates audit;
* canonical data remains primary truth;
* derived failures are visible and retryable;
* private student data is not casually exposed to data operators;
* all P0 tests pass.
---
# 131. Final operating invariant
Admin workflow:
```text
Detect
↓
Understand
↓
Verify
↓
Preview
↓
Publish
↓
Audit
↓
Monitor
```
Never:
```text
see something wrong
↓
open database
↓
change row
↓
forget what happened
```
Ekho institutional data operations must remain **source-grounded, reversible where practical, attributable, permissioned and understandable** even when the dataset grows from hundreds to thousands of universities.
---
# 132. Primary authority sources
This specification was checked primarily against:
1. **OWASP Authorization Cheat Sheet** — least privilege, deny-by-default and authorization controls.
2. **Supabase Row Level Security documentation** — RLS and database-level authorization.
3. **Supabase RBAC / Custom Claims documentation** — trusted custom roles and JWT claims.
4. **Supabase Users documentation** — `user_metadata` must not be trusted for security-sensitive authorization.
5. **Supabase MFA documentation** — TOTP and AAL-based authentication assurance.
6. **Supabase Database Functions documentation** — function execution privileges and `security invoker`/`security definer`.
7. **PostgreSQL documentation** — transactions, constraints and row locking for safe concurrent operations.
8. **Supabase pgAudit documentation** — selective database auditing.
9. **Supabase Database Backups/PITR documentation** — operational disaster recovery boundary.
10. **Supabase Database Security Advisor documentation** — automated checks including RLS configuration problems.
