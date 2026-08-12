# Feature Flags & Runtime Configuration v1
**Status:** LOCKED
**Scope:** runtime behavior control, progressive rollout, kill switches, environment configuration, secrets boundaries and configuration governance
**Stack:** Next.js + TypeScript + Supabase/PostgreSQL + provider-agnostic feature flag layer
**Depends on:** Security & Privacy, Auth & Account Lifecycle, Admin & Data Operations, Observability, API & Error Contract
**Research verified:** 12 August 2026
---
# 1. Goal
Ekho must be able to change selected product behavior safely without requiring risky emergency code deployments.
Required capabilities:
```text
deploy code without exposing feature
gradually release feature
limit feature to internal users
disable broken feature quickly
control environment-specific behavior
change selected operational parameters
experiment safely later
recover when flag provider fails
```
But runtime control must remain understandable.
The system must never become:
```text
500 flags
+
random environment variables
+
hidden database settings
+
conditional code everywhere
```
---
# 2. Fundamental separation
These are different systems:
```text
Feature Flag
Runtime Configuration
Secret
Authorization
Entitlement
Experiment
```
Never use one as a lazy substitute for another.
---
# 3. Feature flag
Feature flag answers:
> Should behavior X be active for this evaluation context right now?
Example:
```text
new_application_workspace = true
```
Feature flags allow deployment to be separated from feature exposure and can support targeting and gradual rollout. This is the standard feature-management model used by OpenFeature and current commercial/open-source flag platforms.
---
# 4. Runtime configuration
Runtime configuration answers:
> What operational value should the application use?
Examples:
```text
MAX_COMPARE_UNIVERSITIES = 5
IMPORT_MAX_BYTES = 5_000_000
SOURCE_RECHECK_BATCH_SIZE = 50
```
Configuration is generally longer-lived than a release flag.
Do not create boolean feature flags for ordinary numeric/system configuration.
---
# 5. Secret
Secret answers:
> What confidential credential/key does the system need?
Examples:
```text
SUPABASE_SECRET_KEY
SMTP_PASSWORD
APPLE_CLIENT_SECRET
SENTRY_AUTH_TOKEN
```
Secrets are never ordinary configuration values and never feature flags.
OWASP recommends centralized, access-controlled secret management with lifecycle, rotation and auditing rather than hard-coding credentials into application artifacts.
---
# 6. Authorization
Authorization answers:
> Is this actor allowed to perform this operation?
Examples:
```text
student may read own application
data_publisher may publish import
admin may manage internal configuration
```
Feature flags must **never provide security authorization**.
Bad:
```text
if featureFlag("admin_panel")
  allow admin operation
```
Correct:
```text
authorizeAdmin()
AND
featureFlag("admin_panel")
```
Flag availability can hide/enable functionality.
Authorization still independently protects the action.
---
# 7. Entitlement
Entitlement answers:
> Has this account purchased/is it eligible for capability X?
Future example:
```text
free
pro
```
Paid entitlements belong to Billing & Entitlements.
Do not implement:
```text
pro_user = feature flag
```
as the long-term billing system.
---
# 8. Experiment
Experiment answers:
> Which controlled treatment should this eligible user receive for measurement?
Example:
```text
onboarding_variant = control | minimal_v2
```
Experiments may use feature-flag infrastructure for assignment.
Experiment methodology belongs to **Experimentation Standard**.
---
# 9. Ekho flag philosophy
Use feature flags **sparingly**.
Before creating one ask:
> Do we actually need to change this independently after deployment?
If no:
```text
do not create a flag
```
Feature flag systems themselves accumulate technical debt when temporary flags remain after their purpose ends. Current Unleash guidance explicitly treats stale flags as technical debt requiring lifecycle management and cleanup.
---
# 10. Allowed flag types
Ekho supports four conceptual types:
```text
release
operational
experiment
migration
```
Do not invent a new type without a real lifecycle difference.
---
# 11. `release`
Used to safely release new functionality.
Example:
```text
applications_workspace_v2
```
Lifecycle:
```text
off
→ internal
→ 5%
→ 25%
→ 50%
→ 100%
→ remove flag
```
Release flags are temporary.
---
# 12. `operational`
Used to control production behavior during failures/risk.
Examples:
```text
document_uploads_enabled
live_updates_enabled
AI_requirement_explanations_enabled
```
These may act as kill switches.
Operational flags may be long-lived where genuine degraded-mode control is necessary.
---
# 13. `experiment`
Used only for explicitly defined experiments.
Example:
```text
university_card_layout_test
```
Lifecycle:
```text
experiment created
→ assignment active
→ analysis complete
→ winner/decision
→ flag removed
```
Do not leave experiments permanently active.
---
# 14. `migration`
Used during technical transitions.
Example:
```text
requirements_engine_v2
```
Possible rollout:
```text
old implementation
→ shadow comparison
→ small traffic
→ full traffic
→ delete old implementation
→ remove flag
```
Migration flags are temporary.
---
# 15. What is NOT a flag
Do not feature-flag:
```text
user role
GDPR consent
account ownership
subscription entitlement
database ownership
RLS authorization
canonical university fact
deadline
tuition
financial aid eligibility
API secret
password policy
```
Those have independent authoritative systems.
---
# 16. Source of truth
All application feature evaluations must pass through **one feature-flag abstraction**.
Conceptually:
```text
featureFlags.evaluate(...)
```
Never:
```text
component A → Vercel directly
component B → database flag
component C → env variable
component D → homemade cookie
```
---
# 17. Provider-agnostic architecture
Ekho flag evaluation architecture should follow OpenFeature-style semantics.
OpenFeature provides a vendor-neutral feature-flag API specifically so application code does not need to depend directly on a particular feature-flag provider.
Conceptually:
```text
Ekho code
↓
flag evaluation interface
↓
provider
```
---
# 18. Provider choice
Do **not lock Ekho v1 architecture** to:
```text
LaunchDarkly
Statsig
Vercel Flags
Unleash
```
Application code depends on Ekho's thin flag interface.
Provider adapter handles external implementation.
---
# 19. Vercel Flags
If Ekho is hosted on Vercel, Vercel Flags is a reasonable provider candidate because it currently supports:
* targeting;
* segments;
* percentage splits;
* environments;
* gradual rollout;
* experiments.
Vercel's current documentation describes these capabilities, but Vercel Flags remains labeled **Beta** as of its 2026 documentation.
Therefore:
> Ekho architecture must not require Vercel Flags specifically.
---
# 20. Flags SDK
Vercel's Flags SDK is currently open-source and provider-agnostic and can work with different providers or custom implementations.
It may be used if it fits implementation cleanly.
Do not introduce both:
```text
Flags SDK
+
separate OpenFeature abstraction
+
custom flag abstraction
```
without need.
One abstraction layer is enough.
---
# 21. Preferred abstraction
Implementation should expose something equivalent to:
```text
getBooleanFlag()
getStringFlag()
getNumberFlag()
getObjectFlag()
```
or a typed flag definition API.
Do not spread raw provider SDK calls throughout the codebase.
---
# 22. Typed flags
Every flag has an explicit type.
Allowed:
```text
boolean
string
number
structured object where justified
```
Example:
```text
applications_workspace_v2: boolean
```
Do not cast arbitrary provider payloads at runtime and hope they match.
---
# 23. Flag registry
Maintain one canonical registry.
Conceptual:
```text
flags/
  definitions.ts
```
Each flag defines:
```text
key
type
default
category
owner
description
created_at
expected_removal
```
Optional:
```text
ticket/spec
```
---
# 24. Flag naming
Use stable:
```text
lower_snake_case
```
Example:
```text
applications_workspace_v2
requirements_engine_v2
live_updates_enabled
```
Avoid:
```text
newThing
test123
george_feature
final_v2_really_new
```
---
# 25. Name by behavior
Good:
```text
requirements_engine_v2
```
Bad:
```text
august_release
```
because the feature's meaning should still be understandable months later.
---
# 26. Do not reuse flag keys
Once:
```text
applications_workspace_v2
```
has been retired, never reuse that key for an unrelated future feature.
History/telemetry would become ambiguous.
---
# 27. Required metadata
Every temporary flag requires:
```text
owner
purpose
created_at
expected_removal
safe_default
```
Tracking owners and planned expiry is a standard way to prevent stale feature flags from accumulating indefinitely.
---
# 28. Expected lifetime
Ekho defaults:
### release
Target removal:
```text
≤ 60 days after full rollout
```
### experiment
Remove after experiment decision.
### migration
Remove after new path is proven and old path deleted.
### operational
May remain indefinitely when it is a real kill switch.
These are Ekho policy values, not external standards.
---
# 29. Flag lifecycle
Use:
```text
created
→ active
→ cleanup_required
→ archived
```
Current production feature-management systems similarly model feature lifecycle through definition, development, production, cleanup and archive stages to surface stale flag debt.
---
# 30. Completion definition
A release controlled by temporary flag is **not fully done** when rollout reaches 100%.
It is done when:
```text
feature stable
+
decision permanent
+
old path removed
+
flag conditional removed
+
flag archived
```
Current feature-management guidance explicitly recommends removing conditional code and archiving the flag once rollout is complete.
---
# 31. Do not delete history immediately
After flag removal from code:
archive its control-plane record where provider supports it.
This preserves useful operational history without keeping stale branching logic alive.
---
# 32. Stale flag review
At least monthly during active development:
identify:
```text
expired release flags
completed experiments
completed migrations
unused flags
100%-on flags
0%-off abandoned flags
```
Then remove unnecessary branching.
---
# 33. Avoid flag dependency trees
Bad:
```text
flag A
  ↓
flag B
  ↓
flag C
  ↓
different rollout percentages
```
Complex parent-child targeting makes actual exposure difficult to reason about; current large-scale flag guidance recommends keeping flag dependencies simple and avoiding overlapping rollout rules.
---
# 34. Flag interaction limit
Avoid implementing product behavior that depends on many independent flags simultaneously.
Example bad state:
```text
A && !B && C && variant(D) === X
```
If multiple flags must interact, explicitly test meaningful combinations.
---
# 35. Evaluate high in the stack
Prefer evaluating a flag near the domain/screen boundary and passing the resulting behavior downward.
Do not call the provider independently from every nested component.
Large-scale feature-management guidance similarly recommends evaluating flags at a high level and keeping branching simple.
---
# 36. Server-side evaluation default
For anything affecting:
```text
security-sensitive behavior
data access
business logic
server action
API response
expensive provider call
```
evaluate the flag on the server.
Never trust a browser-provided flag result.
---
# 37. Client flags
Client-side flag evaluation is acceptable for purely presentational behavior.
Example:
```text
show redesigned card
```
But client flag state does not authorize backend behavior.
---
# 38. Hide ≠ protect
This is prohibited:
```text
flag off in UI
→ assume API protected
```
Backend authorization and validation remain unchanged.
A technically knowledgeable user can call API endpoints without using the UI.
---
# 39. Evaluation context
OpenFeature defines an **evaluation context** containing information usable for targeting and fractional evaluation.
Ekho may provide bounded context such as:
```text
targeting_key
environment
internal_user
country_group if justified
plan later
```
---
# 40. Minimal evaluation context
Do not send all known user profile information to the feature flag provider.
Bad:
```text
full_name
email
nationality
GPA
SAT score
university list
financial aid profile
```
Use only attributes needed for the particular targeting system.
---
# 41. Targeting key
Use a stable opaque identifier.
Recommended:
```text
user UUID
```
or a provider-safe derived opaque identifier if external privacy architecture requires it.
Do not use email as the primary targeting identifier.
---
# 42. No admissions-sensitive targeting by default
Do not target ordinary product rollouts based on:
```text
GPA
test score
financial need
essay data
documents
```
There is almost never a legitimate release-engineering reason.
---
# 43. Country targeting
Country targeting is allowed only for real product reasons such as:
```text
country-specific availability
legal rollout
data coverage limitations
```
Do not casually create different product quality based on nationality.
---
# 44. Locale ≠ rollout country
Internationalization Standard remains authoritative.
Never:
```text
locale = de-DE
→ assume user in Germany
```
for targeting.
---
# 45. Stable percentage assignment
Gradual percentage rollout must be **deterministic** for the same:
```text
flag
+
targeting key
```
unless intentionally changed.
A user should not see:
```text
new UI
old UI
new UI
old UI
```
on random page refreshes.
---
# 46. Anonymous users
For public anonymous experience where stable rollout matters:
use an opaque first-party assignment identifier.
Do not fingerprint users.
If stable identity is unnecessary, rollout may operate at request/session level according to feature requirements.
---
# 47. Account creation
When an anonymous user becomes authenticated:
do not automatically carry experiment/release assignment across identity boundaries unless the assignment model explicitly defines it.
Experimentation Standard will govern this more precisely.
---
# 48. Rollout strategy
Default release progression for meaningful changes:
```text
development
↓
staging
↓
internal production
↓
5%
↓
25%
↓
50%
↓
100%
```
Exact stages may be skipped for tiny low-risk changes.
This is an Ekho release policy.
---
# 49. Do not rollout slowly for ceremony
Small, low-risk changes do not need:
```text
1%
5%
10%
20%
...
```
Feature flags exist to reduce meaningful risk, not create process theater.
---
# 50. Risk-based rollout
More gradual rollout for:
```text
application writes
requirements computation
document workflows
auth
admissions-data changes
notifications
```
Less gradual rollout for:
```text
small visual adjustment
minor animation
noncritical copy
```
---
# 51. Rollout gates
Before increasing exposure check relevant:
```text
error rate
latency
SLO
data integrity
critical workflow success
support signals
```
according to Observability.
Do not evaluate rollout only by:
```text
page views
```
---
# 52. Automatic rollout
Do not implement autonomous rollout in Ekho v1.
Initially:
```text
measure
→ human reviews
→ increase rollout
```
Automatic production rollout may be added when traffic/observability justify it.
---
# 53. Rollback
Every rollout flag must make rollback obvious.
Example:
```text
applications_workspace_v2
100%
→ severe regression
→ 0%
```
without needing a new deployment where architecture permits.
---
# 54. Kill switch
A kill switch exists to disable a harmful/nonessential capability rapidly.
Examples:
```text
document_uploads_enabled
live_admissions_updates_enabled
outbound_notifications_enabled
```
Not every feature deserves one.
---
# 55. Kill switch criteria
Create kill switch when a feature:
```text
has external side effects
can generate user harm
can create expensive load
depends on unstable external service
cannot otherwise be disabled quickly
```
---
# 56. Kill switch default
Each kill switch requires an explicitly chosen **safe default**.
Examples:
```text
outbound_notifications_enabled
default = false if provider completely unavailable during incident-critical initialization
```
or:
```text
public_university_pages_enabled
default = true
```
depending on which failure mode is safer.
Never blindly define:
```text
all defaults = false
```
or:
```text
all defaults = true
```
---
# 57. Provider failure semantics
OpenFeature requires flag evaluation to return the supplied default value rather than throw when normal evaluation fails.
Ekho follows the same invariant:
```text
flag provider error
→ safe default
→ product continues where possible
→ telemetry
```
---
# 58. Feature flag system must not take Ekho down
If remote flag provider is unavailable:
```text
Ekho application must continue
```
with appropriate cached/default behavior.
Never:
```text
flag provider unavailable
→ every request returns 500
```
---
# 59. Provider readiness
Where provider initialization is required:
handle states conceptually equivalent to:
```text
NOT_READY
READY
STALE
ERROR
FATAL
```
OpenFeature explicitly models provider lifecycle states including `READY`, `STALE`, `ERROR` and `FATAL`.
---
# 60. Stale provider data
If provider has a previously known valid flag snapshot but cannot refresh:
prefer:
```text
last known safe value
```
where SDK/provider supports it and the flag's safety semantics allow it.
Otherwise use registered safe default.
---
# 61. Evaluation latency
Flag evaluation must not add significant remote network latency to every UI operation.
Use provider SDK caching/local evaluation/server architecture appropriately.
Do not manually `fetch()` a flag-management HTTP endpoint during every React render.
---
# 62. Evaluation timeout
Any remote evaluation path must be bounded.
Provider timeout must not exceed the normal user-operation latency budget.
---
# 63. Evaluation errors
Do not spam logs for every failed flag resolution.
OpenFeature explicitly warns against logging directly from hot flag-evaluation paths because a missing/broken flag can create enormous repeated log volume; hooks/controlled telemetry are preferred.
---
# 64. Flag observability
Track useful bounded telemetry:
```text
flag_key
variant
evaluation_reason
provider
```
when needed for rollout debugging.
OpenFeature includes standardized observability conventions for feature-flag evaluation metadata.
---
# 65. No user ID as metric label
Do not put:
```text
user_id
email
```
into metrics labels.
Observability Standard's cardinality/privacy rules remain authoritative.
---
# 66. Release correlation
During rollout telemetry should make it possible to answer:
```text
Are errors concentrated in flag variant B?
```
This may use traces/events/analytics rather than high-cardinality metrics.
---
# 67. Change audit
Production flag changes must be auditable.
Track:
```text
flag
old configuration
new configuration
actor
timestamp
environment
```
Current mature feature-management guidance recommends auditing production flag changes and controlling environment permissions.
---
# 68. Production flag permissions
Do not allow every authenticated Ekho admin to change runtime production behavior automatically.
Conceptual permission:
```text
runtime_config_manager
```
may later be separate from:
```text
data_publisher
```
Solo-founder v1 may assign both to the same admin account.
---
# 69. High-risk flag changes
Examples:
```text
disable all notifications
switch requirements engine
disable auth-related component
activate new write path globally
```
require:
* authenticated privileged operator;
* recent MFA where appropriate;
* clear impact preview;
* audit.
---
# 70. No anonymous control plane
Feature flag management must never be exposed through:
```text
public Supabase table
unauthenticated API
client-writable configuration
```
---
# 71. Internal overrides
Developers/admins may need to override a flag for testing.
Override must be clearly identifiable:
```text
OVERRIDE ACTIVE
```
and never silently become the normal experience.
---
# 72. Production override safety
Do not allow arbitrary users to forge:
```text
?flag_new_admin=true
```
and enable protected functionality.
Developer overrides only affect permitted presentation/evaluation and never bypass authorization.
---
# 73. URL overrides
Do not implement production flag overrides through arbitrary public query parameters.
If provider offers authenticated developer tooling, use that instead.
---
# 74. Environment separation
Flags/configuration must distinguish:
```text
development
preview
staging
production
```
Production state must not accidentally copy from development.
---
# 75. Flag definition vs environment state
A flag may exist once conceptually:
```text
requirements_engine_v2
```
with separate environment configuration.
Example:
```text
development = 100%
staging = 100%
production = 5%
```
Do not create:
```text
requirements_engine_v2_dev
requirements_engine_v2_prod
```
as separate flag names.
---
# 76. Environment parity
Flag existence/typing should remain consistent across environments even if values differ.
This catches:
```text
works staging
→ production flag missing
```
before release.
---
# 77. Preview environments
Preview deployment defaults:
```text
no production secrets
no production user data
safe external side effects
test-friendly feature state
```
Do not let a pull-request preview accidentally send real student emails.
---
# 78. Runtime configuration classes
Ekho runtime config has three classes:
```text
static build configuration
server runtime configuration
dynamic operational configuration
```
---
# 79. Static build configuration
Values that legitimately affect the generated browser bundle.
Examples:
```text
public site origin where build-specific
public analytics identifier
public Supabase publishable key
```
These must be explicitly non-secret.
---
# 80. `NEXT_PUBLIC_*`
Next.js exposes browser environment variables prefixed with:
```text
NEXT_PUBLIC_
```
and inlines their values into the JavaScript bundle during build.
Therefore:
> **Anything under `NEXT_PUBLIC_*` must be assumed public.**
Never store secret credentials there.
---
# 81. Build-time immutability
Because `NEXT_PUBLIC_*` values are inlined at build time, changing the deployment environment after build does not magically change those already bundled values.
Do not use them for values needing true dynamic runtime changes.
---
# 82. Server runtime configuration
Server-only values may be read from the server runtime environment.
Next.js supports server-side runtime environment variables, while browser-visible `NEXT_PUBLIC_*` values follow build-time inlining semantics.
Examples:
```text
DATABASE/internal provider URLs
SMTP configuration references
server feature provider credentials
```
---
# 83. Dynamic operational configuration
Use dynamic configuration only for values that genuinely need to change without deployment.
Examples:
```text
import max size
pipeline batch size
temporary operational thresholds
```
Do not move every constant into a database because "it might change someday."
---
# 84. What stays code
Values defining fundamental product/domain semantics should usually remain version-controlled code/specification.
Examples:
```text
supported application state machine
canonical requirement statuses
database relationships
security model
```
Do not make fundamental invariants runtime-editable.
---
# 85. What stays configuration
Good runtime configuration has characteristics:
```text
environment-dependent
operational
bounded
non-secret or separately protected
reasonable to change independently of deployment
```
---
# 86. Config registry
Maintain one typed configuration schema.
Conceptually:
```text
config/
  schema.ts
  server.ts
  public.ts
```
Do not call `process.env` randomly throughout the codebase.
---
# 87. Central validation
At application/server initialization validate configuration:
```text
presence
type
range
format
cross-field invariants
```
Example:
```text
IMPORT_MAX_BYTES > 0
```
Do not wait until a student hits a rare endpoint to discover a required configuration value is missing.
---
# 88. Fail fast
For required configuration necessary to operate safely:
```text
invalid/missing
→ startup/deployment should fail
```
rather than starting a partially undefined product.
---
# 89. Optional config
Optional config must always define explicit fallback behavior.
Bad:
```text
undefined means something different depending on caller
```
Correct:
```text
optional
+
documented default
```
---
# 90. Configuration types
Never leave all env values as arbitrary strings.
Parse into:
```text
boolean
integer
URL
enum
duration
```
at config boundary.
---
# 91. Boolean parsing
Never do:
```text
Boolean(process.env.FEATURE_X)
```
because:
```text
"false"
```
is truthy in JavaScript.
Use strict parsing:
```text
true | false
```
according to config schema.
---
# 92. Numeric parsing
Reject:
```text
NaN
negative where prohibited
out-of-range
```
rather than silently coercing.
---
# 93. URL config
Validate URLs at startup.
Do not allow malformed environment URL to fail unpredictably during production requests.
---
# 94. Config naming
Use:
```text
UPPER_SNAKE_CASE
```
for environment variables.
Example:
```text
SUPABASE_URL
PUBLIC_APP_URL
IMPORT_MAX_BYTES
```
Names should state meaning rather than provider implementation when possible.
---
# 95. Config documentation
Every config value defines:
```text
name
purpose
type
required?
secret?
public?
default
allowed range
environments
```
Do not make engineers search GitHub for where a mysterious env variable is used.
---
# 96. `.env.example`
Repository may contain:
```text
.env.example
```
with variable names and safe example values.
Never real secrets.
---
# 97. `.env` files
Local `.env*` files containing credentials must remain excluded from version control according to project security workflow.
Do not rely solely on developers remembering not to commit them.
Secret scanning should also exist in CI/repository security tooling.
OWASP recommends preventing credentials from being checked into repositories and scanning for secret leaks.
---
# 98. Secrets do not belong in source
Never:
```typescript
const secret = "sk_live_..."
```
Never commit production credentials into:
```text
source
migration
Dockerfile
CI config
fixture
documentation
```
OWASP explicitly recommends keeping secrets out of source code and build artifacts.
---
# 99. Secret manager
Production secrets should live in trusted deployment/provider secret management.
Possible locations depending on architecture:
```text
deployment platform secret store
Supabase Edge Function secrets
Supabase Vault when database-side access is required
```
Supabase provides encrypted Vault storage for secrets needed inside Postgres functions/triggers/webhooks and separate secret management for Edge Functions.
---
# 100. Supabase Vault scope
Use Supabase Vault only where the database actually needs access to the secret.
Do not put all application secrets into PostgreSQL merely because Vault exists.
---
# 101. Environment variables are not a universal vault
OWASP notes that environment variables can be exposed to processes/logs/dumps and recommends dedicated secret-management approaches where available.
For Ekho:
* platform-managed environment secrets are acceptable for ordinary server deployment where appropriate;
* higher-sensitivity/complex future environments may migrate to dedicated secret-management systems.
Do not over-engineer enterprise Vault infrastructure during solo v1 without a concrete need.
---
# 102. Secret access scope
Each server/component receives only secrets it actually needs.
Example:
```text
browser
→ never receives SMTP secret
import worker
→ does not need Apple auth secret
```
Least privilege applies to configuration too.
---
# 103. Secret rotation
Every long-lived credential must be rotatable without product redesign.
For credentials with external expiration/rotation requirements, operational process must track them.
Apple's six-month web Sign in with Apple secret rotation remains governed by Auth specification.
---
# 104. Secret compromise
If secret may have leaked:
```text
revoke/rotate
↓
deploy/update dependent environment
↓
verify
↓
audit incident
```
Never merely delete leaked secret from Git history and continue using the same credential.
---
# 105. Configuration is not secret by default
Values such as:
```text
MAX_COMPARE_UNIVERSITIES
DEFAULT_LOCALE
IMPORT_MAX_BYTES
```
do not need encryption merely because they are configuration.
Classify values intentionally.
---
# 106. Configuration source priority
Define deterministic precedence.
Conceptually:
```text
hard safety constraints in code
↓
environment/server config
↓
dynamic operational config
```
No random caller-specific override hierarchy.
---
# 107. Hard safety boundaries
Runtime config must never override fundamental security bounds.
Example:
```text
database config says upload max = 500 GB
```
Code-level absolute ceiling can still reject impossible/dangerous value.
Runtime configuration controls safe values **inside bounded limits**.
---
# 108. Config constraints
Example:
```text
IMPORT_MAX_BYTES
minimum = 100 KB
maximum hard ceiling = 25 MB
```
Runtime admin may choose:
```text
5 MB
```
but not:
```text
5 TB
```
This prevents configuration mistakes from bypassing engineering safety limits.
---
# 109. Configuration changes are production changes
Changing:
```text
MAX_BATCH_SIZE 50 → 50,000
```
can be as dangerous as code deployment.
Therefore meaningful dynamic config changes require:
```text
authorization
validation
audit
observability
```
---
# 110. Runtime config permissions
Only explicitly privileged operators may change production dynamic configuration.
Normal `data_editor` must not automatically receive runtime infrastructure access.
---
# 111. Runtime config audit
Record:
```text
key
old value
new value
actor
timestamp
environment
reason where high impact
```
Never make production runtime changes invisible.
---
# 112. Configuration version
Dynamic config should have:
```text
version
```
or equivalent revision metadata.
This makes incident timelines reproducible:
```text
15:20 config revision 42
15:21 errors increased
```
---
# 113. Configuration rollback
Every meaningful dynamic configuration change should support returning to previous known-good value.
Prefer:
```text
explicit new revision restoring old value
```
rather than deleting audit history.
---
# 114. Configuration consistency
When changing multiple values that must stay consistent:
apply as one validated configuration revision.
Do not temporarily create invalid combinations.
---
# 115. Client configuration endpoint
If browser requires dynamic public configuration:
expose only an explicit allowlisted public subset.
Never:
```text
return process.env
```
or:
```text
return entire runtime config object
```
---
# 116. Public configuration
Public runtime config may include:
```text
supported locales
public feature presentation
safe UI limits
```
It must never include:
```text
service secrets
internal provider credentials
abuse thresholds that should remain private
```
---
# 117. Flag vs config examples
### Correct flag
```text
new_compare_page = true
```
### Correct config
```text
compare_max_universities = 5
```
### Correct entitlement
```text
advanced_compare available on plan X
```
### Correct authorization
```text
user owns this compare list
```
### Correct secret
```text
provider_api_key
```
Do not collapse them together.
---
# 118. Flag vs experiment
Feature flag controls exposure.
Experiment adds:
```text
hypothesis
population
assignment
metrics
sample
analysis
decision
```
A 50/50 rollout does not automatically constitute a valid A/B experiment.
Experimentation Standard will define this separately.
---
# 119. Flag vs degraded mode
Feature flag provides the switch.
Failure & Recovery specification determines:
```text
when to activate
what degraded behavior means
how to recover
```
Do not define conflicting failure behavior in both places.
---
# 120. Kill switch dependencies
Critical kill switches should not depend on the exact failing subsystem where avoidable.
Example:
If a feature overloads an external AI provider:
```text
AI kill switch
```
should still be evaluable without calling that AI provider.
---
# 121. Database-dependent flag caveat
If all flag state lives only inside primary Postgres and Postgres is down:
the system cannot query new flag configuration.
Therefore evaluator must have:
```text
safe defaults
and/or
last-known state
```
for applicable operational flags.
---
# 122. Flag caching
Provider SDK/provider may cache evaluated/config state.
Caching must define:
```text
TTL/update strategy
failure behavior
environment isolation
```
Do not manually introduce multiple contradictory caches.
---
# 123. Cache staleness
Kill-switch design must account for propagation delay.
If disabling a dangerous feature requires immediate effect:
know the provider/cache propagation characteristics before relying on the flag operationally.
Do not promise:
```text
instant shutdown
```
without measuring it.
---
# 124. Flag change propagation test
Before production reliance on a kill switch test:
```text
change flag
↓
measure time
↓
all relevant runtimes observe change
```
Record expected maximum propagation.
---
# 125. Serverless/runtime instances
Assume multiple runtime instances may briefly hold different cached flag/config snapshots during propagation.
Critical state must tolerate temporary convergence.
Do not design correctness requiring all instances to flip within the same millisecond.
---
# 126. Data migrations
Never use a flag as the only protection for incompatible database migration.
Safe sequence:
```text
make schema backward-compatible
↓
deploy compatible code
↓
enable new path
↓
verify
↓
remove old code/data
```
Feature flag cannot rescue a database schema that old code can no longer use.
---
# 127. Destructive migration rule
Do not remove old database column/table while a rollback path still requires it.
Feature flags and database migration sequencing must agree.
---
# 128. Flagged writes
Be especially careful when two flag branches write incompatible data.
Before gradual rollout define whether:
```text
old path can read new data
new path can read old data
rollback remains possible
```
---
# 129. Shadow mode
For sensitive computation migrations, optional pattern:
```text
old engine produces user result
+
new engine computes silently
+
compare results
```
without exposing new result.
Useful example:
```text
Requirements Engine v2
```
Only when data/privacy/cost permit.
---
# 130. Shadow output
Shadow computation must never:
* trigger duplicate notifications;
* modify user state;
* create external side effects;
* count as real user action.
---
# 131. Notifications kill switch
Outbound notifications should have an operational control allowing sends to pause during:
```text
incorrect deadline incident
notification duplication
provider incident
```
Queue state should remain recoverable according to Notifications/Failure specification.
---
# 132. AI kill switch
Any future AI-dependent nonessential functionality should be independently disableable.
Example:
```text
ai_requirement_explanation_enabled
```
Core source-grounded factual requirements must continue functioning without AI where possible.
---
# 133. Import kill switch
Importer should permit:
```text
validation continues
publication disabled
```
during severe data-integrity incident where appropriate.
Do not unnecessarily disable read-only review if only publication is dangerous.
---
# 134. Live updates kill switch
If automated source monitoring produces corrupted candidate changes:
ability to pause:
```text
candidate processing
publication propagation
notifications
```
may be more useful than taking whole Ekho offline.
---
# 135. Fine-grained degraded control
Prefer switches at meaningful system boundaries.
Good:
```text
outbound_notifications_enabled
document_uploads_enabled
```
Bad:
```text
button_37_enabled
function_12_enabled
```
---
# 136. Flag security
Never place sensitive secret payloads inside flag variants.
Bad:
```json
{
  "provider_api_key": "secret"
}
```
Feature flag systems are behavioral control planes, not secret stores.
---
# 137. Flag values from client are untrusted
If browser receives:
```text
new_checkout = true
```
server must still evaluate/authorize its own critical behavior.
Never accept:
```json
{
  "feature_enabled": true
}
```
from client as proof.
---
# 138. Configuration injection
Runtime configuration values are untrusted until schema validation even when configured by staff.
Validate them exactly like other external operational input.
---
# 139. Remote provider response
If external flag/config provider returns unexpected type:
```text
expected boolean
received object
```
evaluation must fail safely to registered default.
Do not dynamically coerce.
---
# 140. Missing flag
Missing flag:
```text
→ default value
→ controlled observability
```
not:
```text
→ application crash
```
OpenFeature's evaluation model explicitly specifies defaults for abnormal evaluation including missing/broken provider conditions.
---
# 141. Default values belong in code
Every application flag evaluation supplies an explicit safe default in version-controlled code.
Provider dashboard must not be the only place anyone knows default behavior.
---
# 142. Default should represent safe deploy state
When code is deployed before flag definition exists:
application behavior must remain known.
Example:
```text
applications_workspace_v2 default false
```
---
# 143. Default consistency
Do not evaluate the same flag with:
```text
default true
```
in one component and:
```text
default false
```
in another.
Canonical flag definition owns its default.
---
# 144. Testing flags
Every temporary binary release flag requires tests for:
```text
flag OFF
flag ON
```
while both branches exist.
Large-scale feature-management guidance similarly recommends testing both code paths during a flag's active lifetime.
---
# 145. Remove old tests
After flag cleanup:
* remove tests for removed old branch;
* keep tests for permanent new behavior;
* remove mock flag configuration no longer relevant.
---
# 146. Combination testing
When multiple flags can realistically affect the same critical workflow:
test meaningful combinations.
Do not attempt combinatorial testing of every theoretical flag in the whole product.
---
# 147. Default tests
Test provider unavailable:
```text
expected safe default returned
application remains usable
```
---
# 148. Type tests
* [ ] boolean flag never returns string;
* [ ] number validated;
* [ ] unknown variant handled;
* [ ] malformed structured flag falls back safely.
---
# 149. Targeting tests
* [ ] same user receives stable percentage assignment;
* [ ] different environments independent;
* [ ] anonymous assignment behavior documented;
* [ ] unsupported targeting attribute ignored/rejected safely;
* [ ] nationality-sensitive data not accidentally transmitted.
---
# 150. Permission tests
* [ ] flag ON does not bypass authorization;
* [ ] flag OFF does not replace API authorization;
* [ ] client override cannot gain admin capability;
* [ ] user cannot mutate flag state.
---
# 151. Production kill-switch tests
For every critical operational switch:
* [ ] enable/disable works;
* [ ] propagation measured;
* [ ] existing request behavior understood;
* [ ] default known;
* [ ] provider outage behavior known;
* [ ] degraded UX understandable.
---
# 152. Environment tests
* [ ] development config cannot alter production;
* [ ] staging flag values independent;
* [ ] preview uses safe defaults;
* [ ] preview has no production secrets;
* [ ] production flag exists before dependent rollout.
---
# 153. Config tests
* [ ] required missing value fails startup;
* [ ] malformed boolean rejected;
* [ ] malformed number rejected;
* [ ] invalid URL rejected;
* [ ] out-of-range value rejected;
* [ ] optional default deterministic;
* [ ] public/server configuration separated.
---
# 154. Secret tests
* [ ] no secret exists in client bundle;
* [ ] no secret prefixed `NEXT_PUBLIC_`;
* [ ] no secret committed to repository;
* [ ] no secret logged;
* [ ] preview deployment does not receive unnecessary production secrets;
* [ ] credential rotation procedure tested for critical providers.
---
# 155. Audit tests
* [ ] production flag change creates audit event;
* [ ] production dynamic config change creates audit event;
* [ ] actor identifiable;
* [ ] environment identifiable;
* [ ] old/new values retained where non-secret;
* [ ] secret values never copied into audit plaintext.
---
# 156. Cleanup tests/process
Monthly check:
* [ ] stale flags found;
* [ ] 100%-released temporary flags found;
* [ ] dead OFF flags found;
* [ ] expired experiment flags found;
* [ ] old conditionals removed;
* [ ] archived flags no longer evaluated.
---
# 157. Feature flag inventory
Admin/engineering should be able to answer:
```text
What flags exist?
Why?
Who owns them?
Which environment?
What percentage?
How old?
When should they disappear?
```
If this requires repository archaeology, governance has failed.
---
# 158. Runtime config inventory
Likewise:
```text
What runtime values exist?
Which are public?
Which are secret?
What controls them?
Where are they used?
What are valid ranges?
```
---
# 159. Observability metrics
Track bounded operational metrics such as:
```text
flag_evaluation_error_total
flag_provider_unavailable
config_validation_failure
```
Do not produce one metric series per user.
---
# 160. Flag exposure analytics
Experiment/release analysis may need exposure events.
Do not automatically send an analytics event for every flag evaluation.
Only record exposures when product/release analysis requires them.
This prevents event noise and cost.
---
# 161. Evaluation vs exposure
These are different:
```text
flag was evaluated
```
and:
```text
user actually saw/experienced variant
```
Experimentation Standard must use meaningful exposure semantics rather than counting irrelevant backend evaluations.
---
# 162. Feature flag admin UI
Do not build a custom complex flag dashboard in Ekho v1 if provider control plane already solves it.
Ekho may expose only essential internal operational status later.
Avoid rebuilding LaunchDarkly/Vercel Flags/Unleash.
---
# 163. Dynamic config Admin
A small Ekho Admin config screen is justified only for operational values Ekho itself owns.
It should show:
```text
key
current value
description
environment
last changed
```
and enforce typed validation.
---
# 164. Configuration UX
High-impact setting must state effect clearly.
Bad:
```text
Batch = 1000
```
Good:
```text
Verification batch size
100 jobs/run
Allowed: 10–500
```
---
# 165. No arbitrary config editor
Do not create:
```text
key
value
Save Anything
```
for production.
Configuration keys must be predeclared in registry/schema.
---
# 166. No arbitrary JSON config
Structured config is allowed only when actual domain value is structured and validated.
Do not make:
```text
config_blob JSONB
```
the solution for every future requirement.
---
# 167. Runtime config availability
If dynamic configuration backend becomes unavailable:
use:
```text
last-known validated configuration
```
or:
```text
safe code default
```
according to setting semantics.
Do not crash core product solely because optional operational config cannot refresh.
---
# 168. Invalid new config
When an operator proposes invalid value:
```text
reject change
```
Do not publish then discover problem through production errors.
---
# 169. Config rollout
For exceptionally high-risk numeric/operational changes:
change incrementally.
Example:
```text
worker concurrency
10 → 20
```
before:
```text
10 → 1000
```
while watching system health.
---
# 170. Configuration incident
If configuration causes production incident:
Incident Response timeline must include:
```text
configuration revision
actor
timestamp
rollback
```
Configuration changes are deploy-equivalent operational events.
---
# 171. Failure & Recovery integration
Future Failure/Recovery specification defines which switches/configurations correspond to:
```text
normal
degraded
read_only
maintenance
```
This standard supplies mechanisms, not the complete recovery policy.
---
# 172. Maintenance mode
Do not create a generic:
```text
maintenance_mode = true
```
unless actual degraded-mode workflow requires it.
Prefer preserving usable read functionality over blanking the whole product.
---
# 173. Read-only control
A future emergency read-only mode may be appropriate when:
```text
reads safe
writes unsafe
```
It must be implemented at trusted server/domain boundary, not merely hide buttons.
---
# 174. Provider migration
Changing feature flag provider should require:
```text
provider adapter change
configuration migration
testing
```
not rewriting every product component.
That is the main reason for provider-neutral application code. OpenFeature explicitly exists to reduce provider coupling.
---
# 175. Provider migration safety
During migration verify:
```text
flag keys identical
types identical
defaults identical
targeting behavior understood
percentage assignment stability requirement
environment states replicated
```
Do not assume two providers hash percentage rollouts identically.
---
# 176. Do not preserve cohort accidentally
If provider migration changes deterministic bucketing:
experiments may become invalid.
Release flags may tolerate reassignment depending on feature.
Explicitly decide before migration.
---
# 177. Flag code removal
Cleanup procedure:
```text
1. confirm final behavior
2. make final behavior unconditional
3. remove old branch
4. remove flag evaluation
5. remove registry definition
6. test
7. deploy
8. archive provider flag
```
This aligns with current stale-flag cleanup recommendations.
---
# 178. P0 failures
Any of these blocks production:
* feature flag grants authorization;
* client-controlled flag bypasses server security;
* secret appears in browser flag/config payload;
* service secret uses `NEXT_PUBLIC_*`;
* missing flag crashes critical request;
* flag provider outage takes down Ekho;
* production configuration can be changed anonymously;
* arbitrary config keys can be created from Admin;
* invalid runtime config can bypass hard safety limits;
* production flag/config change has no actor/audit history;
* preview environment sends real production side effects unexpectedly;
* release flag changes canonical admissions truth;
* flag rollout writes data incompatible with rollback without migration plan;
* kill switch exists but has never been tested;
* same flag uses inconsistent defaults across application.
---
# 179. Implementation order for Codex
## Stage 1 — Configuration foundation
1. Inventory existing environment variables.
2. Classify public/server/secret/dynamic.
3. Create typed config schema.
4. Add startup validation.
5. Create public config allowlist.
6. Add `.env.example`.
7. Add secret-scanning checks.
## Stage 2 — Flag foundation
8. Create typed flag registry.
9. Choose one provider abstraction.
10. Define safe defaults.
11. Add evaluation-context helper.
12. Add server evaluation.
13. Add client integration only where needed.
14. Add provider-failure behavior.
## Stage 3 — Environments
15. development configuration.
16. preview configuration.
17. staging configuration.
18. production configuration.
19. environment isolation tests.
## Stage 4 — Operational flags
20. identify genuinely necessary kill switches.
21. implement safe degraded behavior.
22. measure propagation.
23. connect observability.
## Stage 5 — Rollout
24. internal targeting.
25. deterministic percentage rollout.
26. rollout health checks.
27. rollback procedure.
## Stage 6 — Dynamic config
28. registry of approved runtime settings.
29. typed validation.
30. authorization.
31. revision/audit history.
32. rollback.
## Stage 7 — Governance
33. flag metadata.
34. expected-removal tracking.
35. stale flag review.
36. cleanup workflow.
37. inventory visibility.
## Stage 8 — Tests
38. ON/OFF tests.
39. provider outage tests.
40. targeting tests.
41. authorization tests.
42. secret-exposure tests.
43. kill-switch tests.
44. config-validation tests.
45. P0 tests.
Do **not** add a feature flag to every existing feature during this implementation.
---
# 180. Initial Ekho flag set
Do not pre-create dozens of hypothetical flags.
Initial likely candidates only when relevant implementation exists:
```text
live_admissions_updates_enabled
outbound_notifications_enabled
document_uploads_enabled
```
Potential temporary rollout flags later:
```text
applications_workspace_v2
requirements_engine_v2
```
If feature does not exist yet:
do not create its flag merely for preparation.
---
# 181. Initial runtime config set
Only values with actual operational purpose.
Examples:
```text
IMPORT_MAX_BYTES
IMPORT_BATCH_LIMIT
DEFAULT_LOCALE
```
Other existing fixed constants should remain code until there is evidence runtime control is useful.
---
# 182. What Codex must NOT build
Do not build:
```text
custom LaunchDarkly clone
custom targeting DSL
custom experimentation platform
generic config database
generic arbitrary JSON control panel
automatic AI-controlled rollout
100 hypothetical flags
client-side authorization flags
```
Use established provider infrastructure where appropriate.
---
# 183. Codex implementation constraint
Before implementation read:
```text
Security & Privacy
Auth & Account Lifecycle
Admin & Data Operations
Observability, SLO & Incident Response
Internationalization & Localization
API & Error Contract
```
Do not change their security/business invariants through runtime configuration.
A feature flag can select between **valid implementations**.
It cannot redefine the rules of the system.
---
# 184. Definition of Done
Feature Flags & Runtime Configuration foundation is complete when:
* flags/config/secrets/auth/entitlements are distinct concepts;
* one typed feature-flag abstraction exists;
* application code is provider-agnostic;
* every flag has a canonical definition and safe default;
* provider failure falls back safely;
* flag evaluation cannot bypass authorization;
* rollout can target internal users;
* deterministic percentage rollout is supported when needed;
* production rollout can be reversed quickly;
* necessary operational kill switches exist and are tested;
* production flag changes are auditable;
* temporary flags have owner/removal lifecycle;
* stale flags are actively removed;
* environment configuration is separated;
* public/server/secret config is separated;
* `NEXT_PUBLIC_*` contains only public values;
* configuration is validated before use;
* dynamic configuration is typed and bounded;
* secrets are never ordinary runtime settings;
* config changes are auditable/reversible;
* preview/staging cannot accidentally act as production;
* all P0 tests pass.
---
# 185. Final invariant
Correct model:
```text
CODE
defines what Ekho can do
↓
FEATURE FLAG
controls whether selected behavior is exposed
↓
RUNTIME CONFIG
controls bounded operational parameters
↓
AUTHORIZATION
controls who is allowed to do it
↓
ENTITLEMENT
controls what product access they purchased/receive
↓
SECRET
authenticates Ekho to protected external systems
```
Never:
```text
one giant settings table
↓
everything controls everything
```
And for a release:
```text
Deploy
↓
Internal
↓
Small rollout
↓
Observe
↓
Increase
↓
100%
↓
Remove old branch
↓
Remove flag
```
Feature flags are temporary **risk-control mechanisms**, not permanent architecture.
---
# 186. Primary authority sources
This standard was checked primarily against:
1. **OpenFeature Specification** — provider-neutral flag evaluation architecture, providers, context, hooks and lifecycle behavior.
2. **OpenFeature Evaluation Context** — contextual targeting and fractional evaluation model.
3. **OpenFeature Observability specification** — standard feature-flag telemetry conventions.
4. **Next.js official Environment Variables documentation** — server/private variables, `NEXT_PUBLIC_*`, build-time inlining and runtime behavior.
5. **Vercel Flags / Flags SDK official documentation (2026)** — current provider capabilities, gradual rollout, targeting and provider-neutral SDK architecture.
6. **OWASP Secrets Management Cheat Sheet** — secret storage, access, lifecycle and rotation guidance.
7. **OWASP CI/CD Security guidance** — preventing secrets from entering repositories/build artifacts and using controlled secret-management infrastructure.
8. **Supabase Vault / Edge Functions Secrets documentation** — encrypted database-side secrets and server-function secret management.
9. **Unleash 2026 official feature-management guidance** — flag lifecycle, stale-flag technical debt, cleanup, owners and expiry.
