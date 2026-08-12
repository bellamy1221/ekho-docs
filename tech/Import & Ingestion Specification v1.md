# Import & Ingestion Specification v1
**Status:** LOCKED
**Scope:** university/program/admissions-data ingestion into Ekho
**Stack:** Next.js + Supabase/PostgreSQL
**Depends on:** Data Standard v1, Data Architecture v1, Data Pipeline, Security & Privacy
**Research verified:** 12 August 2026
---
# 1. Goal
Ekho must make adding and updating university data extremely fast without sacrificing trust.
Target workflow:
```text
Official sources
→ AI-assisted research
→ Ekho Import JSON
→ Paste into Admin
→ Validate
→ Review diff
→ Publish
→ Canonical database
→ Existing Ekho UI renders automatically
```
The founder/admin must not manually fill dozens of forms for every university.
Target interaction:
```text
Research University X
→ Copy JSON
→ Paste
→ Validate
→ Publish
```
For a clean import, this should take minutes rather than manually entering every field.
---
# 2. Core principle
**Import format is not a second data model.**
`Data Standard v1` remains the canonical definition of:
* universities;
* programs;
* admission cycles;
* requirements;
* deadlines;
* tests;
* qualifications;
* tuition;
* costs;
* financial aid;
* scholarships;
* application platforms;
* source/provenance data;
* other admissions entities already defined there.
This specification defines only:
> how canonical Ekho data enters and updates the system.
Codex must never create duplicate fields just because an import JSON uses different names.
---
# 3. Single canonical pipeline
All ingestion methods must converge into one pipeline.
```text
Manual JSON paste
AI-generated JSON
Future internal importer
Future automated research pipeline
Future bulk imports
        ↓
Ekho Import Contract
        ↓
Validation
        ↓
Staging
        ↓
Diff / Review
        ↓
Publish transaction
        ↓
Canonical database
```
Never build separate database-writing logic for each importer.
---
# 4. v1 ingestion method
Primary v1 method:
**JSON paste into Ekho Admin.**
Admin screen:
`Admin → Data → Imports → New Import`
Main UI:
```text
[ Paste Ekho Import JSON here ]
              Validate
```
After validation:
```text
University of X
✓ 1 university
✓ 84 programs
✓ 27 requirements
✓ 12 deadlines
✓ 4 tuition records
✓ 6 financial aid records
✓ 18 sources
2 warnings
0 errors
[ Review changes ]
```
Then:
`Publish`
---
# 5. Explicit v1 exclusions
Do not build yet:
* giant university CMS forms;
* spreadsheet-based canonical editing;
* arbitrary CSV import;
* Excel import;
* automatic publishing from ChatGPT;
* browser extension ingestion;
* automatic scraping initiated from the import textbox;
* direct database uploads;
* Word/PDF → database ingestion;
* unrestricted external API ingestion.
These can come later only when justified.
---
# 6. AI is not the source
Critical invariant:
> **ChatGPT / LLM output is never a source of truth.**
AI may:
* research;
* extract;
* normalize;
* classify;
* transform;
* structure;
* summarize factual information.
AI may not serve as evidence for:
* deadline;
* requirement;
* tuition;
* test minimum;
* scholarship eligibility;
* financial aid policy;
* application platform;
* international eligibility;
* qualification requirement.
Every publishable critical claim must resolve to an acceptable external source.
---
# 7. Source priority
Use the same authority hierarchy throughout ingestion.
### Tier A — preferred
* official university admissions pages;
* official university program pages;
* official university financial aid pages;
* official university tuition/fees pages;
* official university PDFs;
* official university registrar/catalog pages;
* official government education/admissions sources;
* official national admissions systems.
### Tier B — acceptable when directly authoritative
Examples:
* UCAS;
* Common App;
* College Board where the claim is actually within its authority;
* official examination organizations;
* official scholarship organization.
### Tier C
Third-party sources may be retained internally for discovery but must not independently establish critical university requirements.
Examples:
* rankings sites;
* admissions blogs;
* aggregators;
* Reddit;
* forums;
* Wikipedia;
* AI answers.
For a critical university-specific claim, prefer the university or relevant official admissions authority.
---
# 8. Never infer missing information
If official sources do not establish something:
```text
UNKNOWN
```
not:
```text
AI guess
```
Examples:
Do not infer:
* a TOEFL requirement from an IELTS requirement;
* an application deadline from last year's deadline;
* international eligibility from domestic eligibility;
* undergraduate rules from graduate rules;
* Computer Science rules from general university rules;
* financial aid availability because scholarships exist;
* deadline time as `23:59` because only a date was given;
* tuition from a third-party estimate.
Unknown is acceptable.
Incorrect certainty is not.
---
# 9. Import contract
Every payload must follow:
**Ekho University Import JSON v1**
Canonical root:
```json
{
  "schema_version": "1.0.0",
  "import_type": "university_bundle",
  "operation": "upsert",
  "target": {},
  "research": {},
  "sources": [],
  "data": {},
  "evidence": []
}
```
No additional root properties.
JSON Schema is designed specifically for defining and validating JSON structure; Draft 2020-12 remains the current published JSON Schema version.
---
# 10. Canonical JSON Schema
Repository must contain:
```text
/schemas/import/
  ekho-university-import-v1.schema.json
```
Use:
**JSON Schema Draft 2020-12**
Schema must use strict validation.
For objects:
```json
"additionalProperties": false
```
unless expansion is explicitly intended.
Unknown accidental AI fields must produce an error rather than silently entering Ekho.
JSON Schema Draft 2020-12 provides standardized structural validation rules for JSON documents.
---
# 11. One schema source of truth
Do not maintain:
* one JSON shape in the prompt;
* another in TypeScript;
* another in validation;
* another in database-import code.
Maintain one canonical import contract.
From it derive or test:
* TypeScript types;
* runtime validation;
* AI Structured Output schema later;
* example fixtures;
* importer documentation.
Schema drift must fail CI.
---
# 12. Schema version
Required:
```json
"schema_version": "1.0.0"
```
Never silently reinterpret unsupported versions.
Behavior:
```text
supported current version
→ accept
supported previous version
→ migrate explicitly
unknown future version
→ reject
obsolete version
→ show migration required
```
---
# 13. `import_type`
v1:
```json
"import_type": "university_bundle"
```
One bundle may contain:
* one university;
* its programs;
* admission cycles;
* rounds;
* requirements;
* deadlines;
* costs;
* financial aid;
* scholarships;
* source references.
Do not require separate copy/paste actions for every child entity.
---
# 14. `operation`
v1 supported value:
```json
"operation": "upsert"
```
Meaning:
* create missing entities;
* propose updates to matching existing entities.
The importer must never treat `upsert` as:
> blindly overwrite everything.
PostgreSQL supports conflict-aware inserts through `INSERT ... ON CONFLICT`, and Supabase exposes upsert semantics as well.
Ekho adds validation, matching, conflict detection and review before those writes occur.
---
# 15. New entity identifiers
AI must **not generate Ekho UUIDs** for new records.
For new entities use temporary import references.
Example:
```json
{
  "ref": "university:mit"
}
```
Program:
```json
{
  "ref": "program:mit:computer-science-bs"
}
```
Cycle:
```json
{
  "ref": "cycle:mit:2027"
}
```
Ekho generates canonical database IDs during publication.
---
# 16. Existing entity identifiers
When updating known Ekho records, the export/template may contain:
```json
{
  "ekho_id": "existing-uuid"
}
```
AI must preserve it exactly.
Never ask AI to:
* generate UUIDs;
* predict IDs;
* recreate IDs.
---
# 17. Temporary references
`ref` exists only inside the import payload.
Example:
```text
program
ref = program:mit:cs
requirement
program_ref = program:mit:cs
```
During publish:
```text
temporary ref
→ canonical UUID
→ foreign key
```
Temporary refs must never appear on public pages.
---
# 18. University identity matching
For a **new university**, importer checks potential duplicates using:
* official domain;
* canonical name;
* known aliases;
* country;
* city;
* existing external identifiers if available.
Do not automatically merge solely because names are similar.
Example:
```text
University College London
University of London
```
must never be merged based on text similarity.
---
# 19. Program identity matching
Never identify a program from title alone.
Program matching should consider available canonical dimensions such as:
* university;
* program name;
* degree/qualification;
* level;
* faculty/school;
* campus;
* delivery mode where relevant;
* intake;
* canonical program URL.
Ambiguous match:
```text
WARNING / REVIEW REQUIRED
```
not auto-merge.
---
# 20. System-generated fields
AI must not control:
* database UUID;
* slug;
* `created_at`;
* `updated_at`;
* publication state;
* internal quality score;
* search ranking;
* moderation status;
* audit actor;
* canonical verification timestamp.
These belong to Ekho.
---
# 21. Source object
Each source receives a temporary reference.
Example:
```json
{
  "ref": "src_001",
  "url": "https://example.edu/admissions/requirements",
  "source_type": "official_university",
  "publisher": "Example University",
  "title": "Undergraduate Admission Requirements",
  "locator": "International applicants → English proficiency",
  "retrieved_at": "2026-08-12T19:30:00Z",
  "source_updated_at": null
}
```
---
# 22. Source fields
Required:
```text
ref
url
source_type
retrieved_at
```
Recommended:
```text
publisher
title
locator
source_updated_at
```
`source_updated_at` must only exist when the source itself provides a defensible update/publication date.
Never guess it.
---
# 23. Source locator
`locator` helps an admin quickly verify a claim.
Examples:
```text
"International Applicants → English Language Requirements"
```
```text
"PDF page 17"
```
```text
"Tuition → Undergraduate international"
```
Do not store entire copied webpage paragraphs as evidence.
---
# 24. Source URLs
Allowed schemes:
```text
https
http
```
Reject:
```text
javascript:
data:
file:
ftp:
blob:
```
URLs must be syntactically valid URIs; RFC 3986 defines the generic URI syntax.
Prefer HTTPS whenever the official source supports it.
---
# 25. Source URL normalization
Normalize only safe components.
May normalize:
* hostname casing;
* default ports;
* obvious trailing duplicates;
* known tracking parameters such as `utm_*`.
Do **not** blindly remove query parameters.
Some official pages and PDFs require query strings to identify the resource.
---
# 26. No synchronous source fetching during paste
`Validate` must not blindly fetch every supplied URL from the backend.
Reason:
arbitrary server-side URL fetching creates an SSRF attack surface.
OWASP specifically treats attacker-controlled server-side URL retrieval as an SSRF risk and recommends strict URL/network controls.
Therefore:
```text
paste
→ URL syntax/domain validation
→ staging
separate hardened verification job
→ optional HTTP retrieval
```
Reuse the security rules already established in Data Pipeline.
---
# 27. Field-level provenance
Critical facts require provenance.
A record-level source is insufficient when the same record contains many unrelated claims.
Use:
```json
"evidence": [
  {
    "path": "/data/deadlines/0/date",
    "source_refs": ["src_003"]
  }
]
```
`path` uses JSON Pointer syntax.
RFC 6901 defines JSON Pointer specifically for identifying a precise value within a JSON document.
---
# 28. Critical fields requiring evidence
At minimum, explicit provenance is required for:
* application deadline;
* application opening date;
* application round;
* program availability;
* degree type;
* qualification requirements;
* GPA/grade requirements;
* SAT/ACT policy;
* DET/IELTS/TOEFL requirements;
* minimum scores;
* document requirements;
* essays;
* recommendation requirements;
* interview requirements;
* application platform;
* application fee;
* tuition;
* mandatory fees;
* financial aid policy;
* international financial aid eligibility;
* scholarship eligibility;
* scholarship amount;
* scholarship deadline;
* need-blind/need-aware policy;
* CSS Profile/form requirements.
---
# 29. Non-critical provenance
General identity information may use record-level provenance where appropriate:
* canonical university name;
* official website;
* city;
* country;
* campus name.
Still prefer an official source.
---
# 30. Evidence requirement
If:
```text
critical value exists
AND
no acceptable source supports it
```
then:
```text
ERROR: UNSOURCED_CRITICAL_VALUE
```
The import cannot publish that value.
---
# 31. Conflicting official sources
Example:
```text
Admissions page → January 2
Program page → January 5
```
Do not choose automatically because one "looks newer."
Importer produces:
```text
CONFLICT
```
Admin must resolve it.
Possible resolution requires checking:
* applicant population;
* program;
* cycle;
* application round;
* page freshness;
* archive/current status;
* source specificity.
---
# 32. Scope matters
A fact is incomplete without its scope.
Examples:
```text
International undergraduate
Domestic undergraduate
Graduate
Transfer
First-year
Computer Science
Medicine
2027 entry
Early Action
Regular Decision
```
Never flatten scoped requirements into a single global university requirement.
---
# 33. Academic-cycle scope
Admissions information must be attached to the correct cycle/intake wherever the Data Standard requires it.
Example:
```text
2026 entry
```
must not silently become:
```text
2027 entry
```
When importing new-cycle data, historical values may remain in history rather than being overwritten.
---
# 34. Date representation
Exact calendar dates:
```text
YYYY-MM-DD
```
Example:
```text
2027-01-05
```
Machine timestamps use RFC 3339-compatible date-time representations. RFC 3339 defines an Internet timestamp profile based on ISO 8601.
---
# 35. Deadline time
If official source states:
```text
January 5
```
store the date.
Do not invent:
```text
23:59
```
If source explicitly gives:
```text
January 5 at 11:59 PM Eastern Time
```
then store:
* date;
* local time;
* timezone according to Data Standard.
---
# 36. Countries
Use canonical country values from Data Standard.
When country codes are required use:
**ISO 3166-1 alpha-2.**
ISO identifies alpha-2 as its general-purpose two-letter country-code representation.
Examples:
```text
US
GB
IT
DE
CH
```
---
# 37. Currency
Currency codes must use:
**ISO 4217**
Examples:
```text
USD
GBP
EUR
CHF
```
ISO 4217 defines internationally recognized three-letter currency codes.
Never use ambiguous values such as:
```text
$
£
kr
```
as the machine currency identifier.
---
# 38. Money values
Use the money representation already locked in Data Standard v1.
Do not introduce floating-point currency conventions independently inside ingestion.
Importer must reject values that do not conform to the canonical money model.
---
# 39. Missing vs null vs unchanged
This distinction is critical for updates.
### Property omitted
Means:
> importer makes no assertion about this field.
For an update:
```text
do not change existing value
```
### Explicit `null`
Allowed only where Data Standard explicitly permits it.
Means:
> known empty/not applicable/clear operation according to that field's semantics.
It must never automatically mean:
> AI could not find the value.
### Unknown
Use the canonical `unknown` state defined by Data Standard where applicable.
Never convert uncertainty into `null`.
---
# 40. Update safety
Suppose current Ekho data says:
```text
TOEFL minimum = 100
```
New AI research simply fails to find TOEFL information.
The imported payload omits TOEFL.
Result:
```text
existing 100 remains
```
not:
```text
100 → null
```
Absence in an import is not evidence of removal.
---
# 41. Removing existing information
Removing a previously published fact requires explicit evidence.
Example:
Current:
```text
SAT required
```
Official source now says:
```text
SAT optional
```
Import proposes explicit replacement and supplies source.
But if the new source simply does not mention SAT:
```text
do not delete existing SAT policy automatically
```
Flag:
```text
possibly stale
```
for review.
---
# 42. No arbitrary HTML
Imported content must not contain raw HTML.
Reject fields containing arbitrary:
```html
<script>
<iframe>
<object>
<style>
```
or arbitrary markup intended to be rendered.
Frontend must never use imported strings through unsafe HTML rendering.
OWASP specifically warns about unsafe DOM/HTML rendering and APIs such as unsanitized `dangerouslySetInnerHTML`.
---
# 43. Structured content over prose
Prefer:
```json
{
  "test": "IELTS Academic",
  "minimum_score": 7.0
}
```
instead of:
```text
"Students usually need around IELTS 7 although this can vary..."
```
The importer exists to create structured admissions intelligence.
Not articles.
---
# 44. Copyright-safe ingestion rule
Do not use imports to copy entire university pages into Ekho.
Store:
* structured facts;
* short necessary labels;
* concise original summaries where required;
* source title;
* source URL;
* locator.
Do not store:
* full admission guides;
* long copied FAQ answers;
* long copied policy paragraphs;
* full copyrighted page text merely for convenience.
This also reduces hallucination and maintenance problems.
---
# 45. Import lifecycle
Every import has a lifecycle.
```text
received
→ validating
→ needs_review / ready
→ publishing
→ published
```
Alternative endings:
```text
invalid
failed
cancelled
superseded
```
---
# 46. Validation pipeline
Validation must run in fixed order.
```text
1. Request validation
2. JSON parsing
3. Schema validation
4. Type/format validation
5. Semantic validation
6. Referential validation
7. Source/provenance validation
8. Duplicate detection
9. Conflict detection
10. Current-database diff
11. Publishability gate
```
Never write canonical university data before these stages finish.
---
# 47. Layer 1 — JSON parsing
Reject:
* malformed JSON;
* comments;
* trailing invalid syntax;
* multiple root documents.
UI should identify line/position where possible.
Example:
```text
Invalid JSON at line 143
```
---
# 48. Layer 2 — JSON Schema
Validate:
* required properties;
* allowed types;
* allowed enums;
* patterns;
* array structure;
* additional properties;
* length boundaries;
* URI/date formats where applicable.
Invalid schema blocks progression.
---
# 49. Runtime validator
Use a standards-compliant JSON Schema Draft 2020-12 validator.
For the TypeScript implementation, **Ajv 8 with its Draft 2020-12 implementation is acceptable and recommended for this importer**.
Ajv documents support for Draft 2020-12 through its dedicated 2020 implementation.
Do not build a homemade JSON Schema interpreter.
---
# 50. Layer 3 — semantic validation
Schema-valid does not mean logically valid.
Examples of semantic errors:
```text
deadline_before_application_open
```
```text
minimum_score > maximum_possible_score
```
```text
scholarship_amount < 0
```
```text
program references nonexistent university
```
```text
application round belongs to wrong cycle
```
```text
international requirement marked global
```
Create explicit validators for domain invariants from Data Standard.
---
# 51. Layer 4 — referential validation
Every internal reference must resolve.
Example:
```text
requirement.program_ref
```
must resolve to:
* a program inside the bundle; or
* an existing canonical program ID.
PostgreSQL foreign-key constraints provide final relational integrity, but errors should be detected before publication for understandable admin feedback.
---
# 52. Layer 5 — provenance validation
Validate:
* source ref exists;
* critical field has source;
* source type acceptable;
* URL syntactically valid;
* no malformed locator;
* claimed source fits the entity scope where determinable.
---
# 53. Layer 6 — duplicate detection
Detect:
* duplicate university inside payload;
* duplicate source;
* duplicate program;
* duplicate requirement;
* duplicate deadline;
* likely duplicate canonical entity.
Do not simply deduplicate silently.
Show what was detected.
---
# 54. Layer 7 — database comparison
After payload validation:
```text
canonical current state
vs
proposed state
```
Produce a structured diff.
Examples:
```text
ADDED
CHANGED
UNCHANGED
REMOVED
CONFLICT
UNKNOWN
```
---
# 55. Review diff
Admin must see meaningful changes.
Example:
```text
Regular Decision deadline
Current:
January 3, 2027
Proposed:
January 5, 2027
Source:
Official Undergraduate Admissions
Verified source attached
```
Not:
```text
record 62 changed
```
---
# 56. Important changes highlighted
High-impact changes receive elevated visibility:
* deadline;
* required test;
* minimum test score;
* academic qualification;
* tuition;
* financial aid eligibility;
* scholarship deadline;
* application platform;
* required document.
These should not disappear inside a 200-field diff.
---
# 57. Import issue severity
Three levels:
### ERROR
Blocks publication.
Examples:
* invalid JSON;
* unsupported schema;
* missing required relationship;
* unsourced critical value;
* impossible value;
* unresolved entity reference.
### WARNING
Requires attention but may permit publication after review.
Examples:
* potential duplicate;
* conflicting official pages;
* existing data disappeared from new source;
* old source;
* unusually large change.
### INFO
Non-blocking information.
Example:
```text
12 values unchanged
```
---
# 58. Publish button state
```text
ERROR exists
→ Publish disabled
No ERROR + unresolved required conflict
→ Publish disabled
Warnings only
→ admin can review/approve
Clean
→ Publish enabled
```
---
# 59. No automatic public publishing from AI
Even when JSON is perfectly schema-valid:
```text
AI → production
```
is prohibited in v1.
Required:
```text
AI
→ validation
→ review
→ explicit admin publish
```
Structured correctness does not establish factual correctness.
---
# 60. Atomic publication
Publication of a university bundle must be atomic.
Conceptually:
```text
BEGIN
university
programs
cycles
requirements
deadlines
costs
aid
sources
provenance
audit record
COMMIT
```
Failure:
```text
ROLLBACK
```
PostgreSQL transactions provide all-or-nothing grouping of database operations.
Never publish half a university bundle.
---
# 61. Database publish function
Create one controlled publication path.
Example conceptual function:
```text
publish_university_import(import_id)
```
It must:
1. lock/read staged import;
2. verify status;
3. re-run critical invariants;
4. calculate/check current version;
5. write canonical records;
6. write provenance;
7. write audit/version record;
8. mark import published;
9. commit.
---
# 62. Function security
Administrative publish functions must not be callable by ordinary users.
Supabase notes that database functions can have broad execution privileges by default and recommends explicitly revoking/granting function execution. It also recommends `security invoker` by default; if `security definer` is necessary, `search_path` must be explicitly controlled.
For Ekho:
```text
anon → DENY
authenticated student → DENY
admin server → ALLOW
```
---
# 63. Server-only ingestion
Admin browser must never receive a Supabase secret/service credential.
Supabase secret/service-role access bypasses normal RLS protections and is intended for trusted server environments.
Architecture:
```text
Admin browser
→ authenticated Next.js server endpoint
→ admin authorization
→ validation/import service
→ PostgreSQL
```
Never:
```text
Admin browser
→ service_role
→ database
```
---
# 64. Admin authorization
Import capability is **not** equivalent to being any authenticated Ekho user.
Require explicit staff/admin authorization.
Suggested internal roles:
```text
data_viewer
data_editor
data_publisher
admin
```
v1 may initially use only:
```text
admin
```
but authorization architecture should not equate:
```text
authenticated = admin
```
---
# 65. Staging tables
Import jobs must live separately from canonical application tables.
Suggested private namespace:
```text
private.import_jobs
private.import_payloads
private.import_issues
private.import_changes
private.import_audit
```
Supabase supports separate Postgres schemas, including schemas kept outside normal Data API exposure.
Do not put unvalidated AI payloads directly into canonical university tables.
---
# 66. Suggested `import_jobs`
Conceptual fields:
```text
id
schema_version
import_type
status
created_by
created_at
validated_at
published_at
payload_hash
target_university_id nullable
error_count
warning_count
published_version_id nullable
```
Exact SQL types follow Data Architecture conventions.
---
# 67. Raw payload retention
Store the submitted JSON for audit/debugging.
But:
* it remains private;
* never rendered directly to users;
* never indexed;
* never exposed through public API;
* subject to reasonable retention policy.
Store a deterministic hash as well.
---
# 68. Idempotency
Submitting the exact same payload twice must not accidentally duplicate records.
Generate:
```text
payload_hash
```
If an identical completed import already exists:
```text
Duplicate import detected
```
Admin may inspect it rather than republish blindly.
---
# 69. Upsert keys
Canonical `UPSERT` must rely on actual:
* IDs;
* unique keys;
* constrained deterministic relationships.
Never build SQL such as:
```text
if name approximately matches → overwrite
```
PostgreSQL `ON CONFLICT` relies on unique/exclusion constraints to define conflicts predictably.
---
# 70. Database constraints remain mandatory
Importer validation is not enough.
Keep database:
* primary keys;
* unique constraints;
* foreign keys;
* `NOT NULL`;
* appropriate `CHECK` constraints.
PostgreSQL maintains these constraints independently of application validation.
Defense in depth:
```text
JSON Schema
+
semantic validator
+
database constraints
```
---
# 71. Concurrency protection
Scenario:
```text
Admin A validates import
↓
another import updates university
↓
Admin A publishes old preview
```
Must not silently overwrite newer data.
Before publish compare:
```text
base/current entity version
```
If changed since preview:
```text
DATABASE_CHANGED_SINCE_REVIEW
```
Require fresh diff.
---
# 72. Import version history
Every successful publication creates a version record.
Track:
```text
university_id
version_id
import_id
published_at
published_by
change_summary
```
Ekho must be able to answer:
> Which import changed this value?
---
# 73. Field provenance history
For important fields retain:
```text
value
source
verified_at
valid_from / cycle where applicable
superseded_at
```
Do not destroy prior history whenever a value changes.
This directly supports future **Live Admissions Updates**.
---
# 74. Rollback
Admin should be able to inspect a previous import/version.
Initial v1 rollback may be:
```text
restore through another reviewed import
```
rather than building an unsafe instant database rollback button.
Never implement:
```text
Undo everything blindly
```
without considering later dependent changes.
---
# 75. Frontend rendering invariant
Imported JSON **never controls the site's layout.**
Do not allow payload properties such as:
```text
fontSize
color
component
layout
html
css
position
```
The flow is:
```text
import JSON
→ canonical structured DB
→ Ekho frontend components
```
Therefore once the university is published:
* university page;
* requirements;
* deadlines;
* cost;
* financial aid;
* application workspace;
automatically use the same design system/components as every other university.
This is what makes the process:
> paste data once → site appears correctly.
---
# 76. No per-university frontend coding
Adding Harvard, MIT or Bocconi must never require:
```text
create Harvard.tsx
create MIT.tsx
create Bocconi.tsx
```
University pages are data-driven.
One page template:
```text
/universities/[slug]
```
reads canonical database records.
University-specific data must never be hard-coded inside React components.
---
# 77. Search/index effects
After successful publish:
1. commit canonical data;
2. enqueue/update search representation;
3. invalidate relevant cache;
4. refresh affected university/program page;
5. refresh SEO data where required.
Do not trigger those actions for invalid/staged imports.
---
# 78. Failed side effects
Canonical DB commit is the authoritative success event.
If:
```text
database publish succeeds
but
search refresh fails
```
do not roll back correct canonical data solely because an asynchronous derived system failed.
Instead:
```text
publish = successful
derived_sync = retry_required
```
Retry the derived process.
---
# 79. AI-assisted manual research workflow
Target v1 workflow for you:
### Step 1
Open:
```text
Admin → Data → Imports → New
```
### Step 2
Enter:
```text
University name
Country
Target admission cycle
Applicant scope
```
Example:
```text
MIT
US
2027 entry
International undergraduate
```
### Step 3
Click:
**Copy Research Prompt**
### Step 4
Paste prompt into ChatGPT.
### Step 5
ChatGPT researches official sources and returns:
```text
Ekho University Import JSON v1
```
### Step 6
Copy JSON.
### Step 7
Paste into Ekho.
### Step 8
Click:
**Validate**
### Step 9
Review:
```text
errors
warnings
sources
diff
```
### Step 10
Click:
**Publish**
---
# 80. Generate the AI prompt inside Ekho
Do not permanently hard-code a prompt into personal notes and expect it to match future schemas.
Admin should generate the prompt from the currently supported importer version.
Conceptually:
```text
Copy Research Prompt
```
generates:
```text
task
+
current data rules
+
current schema version
+
required source policy
+
requested university/scope
```
When the schema changes, the generated prompt changes with it.
---
# 81. AI research prompt — canonical v1
Use this base instruction:
```text
You are preparing structured university admissions data for Ekho.
Research the requested university using current official sources only for factual admissions data.
TARGET
University: {{UNIVERSITY_NAME}}
Country: {{COUNTRY}}
Applicant scope: {{APPLICANT_SCOPE}}
Admission cycle / entry year: {{ENTRY_CYCLE}}
TASK
Return one Ekho University Import JSON payload conforming exactly to Ekho University Import Schema v1.0.0.
SOURCE RULES
- Prioritize official university admissions, program, tuition, financial aid, scholarship, registrar/catalog and official PDF sources.
- Use official government or national application systems where they are authoritative.
- Do not use AI knowledge, Wikipedia, Reddit, rankings, blogs or admissions aggregators as evidence for critical facts.
- Every critical admissions fact must reference at least one supporting source.
- Do not infer missing values.
- If a fact cannot be verified from an acceptable source, omit it or represent it as unknown exactly as allowed by the schema.
- If two current official sources conflict, preserve the conflict for review instead of choosing a value without evidence.
- Never reuse requirements across undergraduate, graduate, transfer, domestic, international or program-specific scopes unless the official source explicitly applies to them.
- Never reuse a previous admission cycle as if it were current.
- Do not invent deadline times when only a calendar date is provided.
- Do not invent minimum scores, GPA thresholds, tuition, scholarships or financial aid eligibility.
NORMALIZATION RULES
- Dates: YYYY-MM-DD when an exact date is known.
- Timestamps: RFC 3339.
- Countries: canonical Ekho value / ISO 3166-1 alpha-2 where a country code is required.
- Currencies: ISO 4217.
- Preserve the scope of every requirement and deadline.
- Use temporary ref values for new entities. Never invent Ekho UUIDs.
- Do not include presentation, HTML, CSS or layout data.
- Do not copy long copyrighted passages. Store structured facts, concise original summaries, source URLs and source locators.
QUALITY CHECK BEFORE OUTPUT
Verify:
1. Every critical value has evidence.
2. All source_refs exist.
3. All internal refs resolve.
4. Dates belong to the requested cycle.
5. Applicant scopes are not mixed.
6. No guessed values are present.
7. No duplicate entities are present.
8. JSON follows schema_version 1.0.0.
OUTPUT RULE
Return ONLY valid JSON.
No Markdown fences.
No introduction.
No explanation before or after the JSON.
```
---
# 82. AI schema reliability
The manual ChatGPT workflow still requires the Ekho validator.
Never assume:
```text
model said valid JSON
= valid import
```
The validator is authoritative.
---
# 83. Future OpenAI API automation
If Ekho later automates the research-to-JSON step through the OpenAI API, use **Structured Outputs with the canonical JSON Schema** rather than asking the model to merely "output JSON."
OpenAI introduced Structured Outputs specifically so model output can conform to a developer-provided JSON Schema; plain JSON mode alone does not guarantee schema conformance.
However:
```text
schema-valid
≠
factually verified
```
The normal Ekho evidence/validation pipeline still applies.
---
# 84. No AI confidence score
Do not store:
```text
confidence: 0.93
```
as evidence of correctness.
AI self-confidence must not determine publication.
Use deterministic states instead:
```text
verified_source_attached
unknown
conflict
needs_review
```
---
# 85. Batch imports
Support batch ingestion after single-bundle flow is stable.
Suggested v1.1:
```text
multiple university_bundle payloads
```
but process each university independently for:
* validation;
* review;
* publication;
* failure handling.
Never make:
```text
100 universities
→ one giant irreversible transaction
```
---
# 86. Initial batch limit
Recommended starting operational limits:
```text
single university bundle:
≤ 5 MB JSON
batch:
≤ 25 universities
```
Keep limits configurable.
This is an Ekho operational decision, not an external standard.
---
# 87. Request-size protection
Importer must reject oversized input before expensive processing.
OWASP recommends limiting input/upload sizes as part of preventing resource-exhaustion and denial-of-service issues.
Do not parse unlimited JSON bodies.
---
# 88. Input security
Treat imported JSON as untrusted input even when pasted by an admin.
OWASP explicitly recommends allowlist-oriented input validation rather than relying on malformed-input cleanup.
Validate server-side.
Client validation is UX only.
---
# 89. SQL safety
Never construct SQL using raw imported strings.
Use:
* Supabase/PostgREST parameterization;
* parameterized queries;
* prepared database logic;
* typed RPC parameters.
OWASP recommends parameterized queries as the primary defense against SQL injection.
---
# 90. Admin import endpoints
Logical API:
```text
POST /api/admin/imports/validate
GET /api/admin/imports/:id
POST /api/admin/imports/:id/publish
POST /api/admin/imports/:id/cancel
```
Optional later:
```text
POST /api/admin/imports/:id/revalidate
```
No public write endpoint.
---
# 91. CSRF / mutation safety
All publishing mutations must:
* require authenticated admin;
* use non-GET requests;
* follow the project's CSRF/origin protections;
* never accept publish actions through a URL query parameter.
OWASP recommends protecting authenticated state-changing browser requests against CSRF and warns against placing security tokens in URLs.
---
# 92. Import audit
Record:
```text
who submitted
who reviewed
who published
when
payload hash
schema version
target university
number of changes
result
```
Do not create untraceable university edits.
---
# 93. Import logging
Log operational metadata:
```text
import_id
status
duration
error_code
records_processed
records_changed
records_added
```
Do not log the entire JSON payload in normal application logs.
The payload already exists in controlled private storage.
---
# 94. Source verification audit
For important data Ekho should later be able to answer:
```text
What is the current value?
Where did it come from?
When was it verified?
Which import introduced it?
What did it replace?
```
This is a core trust requirement.
---
# 95. Admin import screen
Keep UI minimal.
Screen 1:
```text
New Import
[ large JSON input ]
Validate
```
Screen 2:
```text
MIT
2027 International Undergraduate
0 Errors
3 Warnings
124 Changes
Changes
Sources
Warnings
Raw JSON
Publish
```
Do not create a giant dashboard.
---
# 96. Review filters
Useful filters:
```text
All
Critical
Changed
Added
Conflicts
Warnings
```
Default:
**Critical + Changed**
Founder should not inspect hundreds of unchanged rows.
---
# 97. Source inspection
Clicking a proposed value should show:
```text
Value
Scope
Source
Locator
Previous value
```
with:
**Open official source**
This dramatically reduces verification time.
---
# 98. Editing before publish
Admin should be able to correct a small value without returning to ChatGPT.
Two acceptable methods:
### v1
Edit raw JSON → revalidate.
### later
Structured inline edit.
Do not build a full CMS before it becomes necessary.
---
# 99. Revalidation
Any edit after validation invalidates previous validation state.
Required:
```text
edit
→ validation status cleared
→ revalidate
→ publish
```
Never publish a payload different from the one that passed validation.
---
# 100. Payload hash
Hash the exact validated canonicalized payload.
On publish:
```text
current payload hash
==
validated payload hash
```
Otherwise block publication.
---
# 101. Update workflow for existing university
For an existing university:
```text
Open university
→ New research/update import
→ Ekho exports current target identifiers
→ generated AI prompt
→ research
→ import
→ compare
→ publish
```
The AI does not need to reconstruct existing IDs.
---
# 102. Full refresh is not replacement
When re-researching a university:
```text
new import
```
must not imply:
```text
delete all existing children and recreate them
```
Use record-level matching and changes.
This preserves:
* stable IDs;
* application references;
* history;
* analytics;
* user saved data.
---
# 103. Never cascade research mistakes into user data
University-data imports must never directly modify:
* user applications;
* user tasks completed state;
* essays;
* notes;
* documents;
* personal profile.
If a university requirement changes:
```text
canonical institutional data changes
→ personalization/application engine recalculates affected state
```
according to its own rules.
---
# 104. Requirement-change propagation
Example:
```text
TOEFL minimum
100 → 105
```
Publishing changes the canonical requirement.
Then downstream systems may:
* recompute personalized requirement;
* mark affected application;
* create update event;
* notify according to Notifications specification.
Importer itself does not send random notifications.
---
# 105. Delete/archive university
Import payload must not permanently delete a university merely because:
* page disappeared;
* AI failed to find it;
* official URL changed.
Destructive entity deletion requires a separate admin action.
Prefer:
```text
inactive
archived
superseded
```
where appropriate.
---
# 106. Program discontinued
If an official source establishes that a program is discontinued:
do not physically delete its historical record.
Mark according to canonical lifecycle:
```text
inactive / discontinued
```
and preserve relevant historic application information.
---
# 107. Source disappeared
If an existing source URL becomes unavailable:
do not immediately delete every fact derived from it.
Mark source:
```text
unreachable / needs_reverification
```
Then use Data Pipeline/live monitoring rules.
---
# 108. Data freshness
Importer records:
```text
retrieved_at
```
Ekho determines:
```text
verified_at
```
after review/publication according to source policy.
AI must never fake freshness.
---
# 109. Old-cycle information
Historical data may be useful.
Do not convert:
```text
2026 deadline
```
into:
```text
2027 deadline
```
simply because 2027 is the requested cycle.
If current cycle information is unavailable:
```text
UNKNOWN CURRENT VALUE
```
while historical data remains historical.
---
# 110. Import performance target
For a normal single-university payload:
Target:
```text
local structural validation < 1 second where practical
full server validation/diff ≈ few seconds
```
Do not block validation on:
* crawling dozens of pages;
* AI calls;
* search re-indexing;
* cache regeneration.
Those belong to separate processes.
---
# 111. Validation must be deterministic
The same:
```text
payload
+
same database state
+
same schema
```
should produce the same validation result.
Do not ask an LLM:
> Is this import valid?
for the publication gate.
Use code.
---
# 112. Import metrics
Track:
```text
import_started
import_json_parse_failed
import_validation_failed
import_ready
import_published
import_failed
```
Operational metrics:
```text
validation_duration
publish_duration
error_count
warning_count
entities_created
entities_updated
critical_changes
duplicate_candidates
```
No source content or imported long text in analytics.
---
# 113. Quality metrics
Internal data-quality metrics worth tracking:
```text
% critical values with valid source
% records current-cycle verified
% imports passing first validation
average warnings/import
average manual corrections/import
time from research → publish
duplicate-import rate
```
This tells us whether the ingestion workflow actually saves work.
---
# 114. Success metric
Initial target:
> A complete well-researched university bundle can be added without touching SQL and without manually recreating the information field-by-field in Ekho.
Later target:
> Most correctly generated university bundles require only source/diff review before publication.
Do not set an artificial target of zero human review before the evidence supports it.
---
# 115. Required fixtures
Repository must contain test payloads:
```text
valid-new-university.json
valid-update-university.json
invalid-json.json
invalid-schema.json
missing-critical-source.json
duplicate-program.json
broken-reference.json
conflicting-deadline.json
unknown-field.json
malicious-html.json
malicious-url.json
stale-version.json
same-payload-twice.json
```
---
# 116. Required tests — parser/schema
* [ ] Valid payload accepted.
* [ ] Invalid JSON rejected.
* [ ] Unknown root field rejected.
* [ ] Unknown nested field rejected where schema is strict.
* [ ] Missing required field rejected.
* [ ] Invalid enum rejected.
* [ ] Invalid URI rejected.
* [ ] Invalid date rejected.
* [ ] Unsupported schema version rejected.
* [ ] Oversized payload rejected.
---
# 117. Required tests — provenance
* [ ] Every `source_ref` resolves.
* [ ] Critical deadline without evidence blocked.
* [ ] Critical tuition value without evidence blocked.
* [ ] Test-score requirement without evidence blocked.
* [ ] Noncritical record-level evidence works where allowed.
* [ ] Conflicting source state surfaces.
* [ ] Source locator survives import.
* [ ] Historical source does not become current-cycle evidence automatically.
---
# 118. Required tests — relationships
* [ ] Program resolves to university.
* [ ] Requirement resolves to correct program/scope.
* [ ] Deadline resolves to correct cycle.
* [ ] Financial aid scope resolves.
* [ ] Missing temp reference fails.
* [ ] Existing UUID references resolve.
* [ ] AI-generated fake UUID does not create arbitrary record relationships.
---
# 119. Required tests — update
* [ ] Omitted property leaves current value unchanged.
* [ ] Explicit allowed null follows Data Standard semantics.
* [ ] Missing new research does not delete old data.
* [ ] Explicit verified replacement updates value.
* [ ] Stable canonical IDs remain stable.
* [ ] Potential duplicate is not silently merged.
* [ ] Database change after preview forces re-review.
---
# 120. Required tests — atomicity
Force failure midway through publish.
Expected:
```text
0 partial canonical changes
```
* [ ] university not half-created;
* [ ] no orphan programs;
* [ ] no orphan requirements;
* [ ] import status becomes failed;
* [ ] rollback succeeds.
---
# 121. Required tests — security
* [ ] Student cannot access importer.
* [ ] Student cannot invoke publish RPC.
* [ ] Anonymous user cannot access importer.
* [ ] Supabase secret is absent from client bundle.
* [ ] HTML payload never renders as executable markup.
* [ ] `javascript:` source URL rejected.
* [ ] SQL-like imported strings remain data.
* [ ] malformed source URLs never trigger arbitrary server fetch.
* [ ] oversized payload blocked.
* [ ] publish requires authenticated admin.
---
# 122. Required tests — idempotency
* [ ] Same payload twice does not duplicate university.
* [ ] Same program does not duplicate.
* [ ] Same source does not multiply unnecessarily.
* [ ] duplicate payload hash recognized.
* [ ] repeated publish request is safe.
* [ ] retry after network failure cannot double-create records.
---
# 123. Required tests — frontend
After successful import:
* [ ] university page loads;
* [ ] program pages load where applicable;
* [ ] requirements render;
* [ ] deadlines render;
* [ ] tuition renders;
* [ ] financial aid renders;
* [ ] source information renders where product requires;
* [ ] search finds university;
* [ ] no imported HTML affects design;
* [ ] no per-university frontend code exists.
---
# 124. P0 failures
Any of these blocks release:
* AI payload writes directly to production without validation;
* critical admissions value published without acceptable provenance;
* university import can modify another unrelated university accidentally;
* partial bundle commits after transaction failure;
* service/admin key exposed client-side;
* normal student can publish university data;
* arbitrary HTML executes;
* URL validation enables SSRF;
* identical imports create duplicates;
* missing field deletes existing verified value;
* historical cycle becomes current without evidence;
* conflicting critical sources silently resolved;
* university import changes user-owned private documents/data;
* canonical Data Standard is bypassed by importer-specific duplicate fields.
---
# 125. Implementation order for Codex
Implement exactly in stages.
### Stage 1 — Contract
1. Read Data Standard v1.
2. Read Data Architecture v1.
3. Map canonical entities.
4. Create `EkhoUniversityImportV1` schema.
5. Create fixtures.
6. Create schema tests.
### Stage 2 — Validation
7. JSON parsing.
8. JSON Schema validator.
9. semantic validators.
10. temporary-reference resolver.
11. provenance validator.
12. duplicate detector.
13. database diff.
### Stage 3 — Staging
14. private import tables.
15. import job lifecycle.
16. raw payload storage.
17. payload hashing.
18. audit.
### Stage 4 — Admin UI
19. New Import screen.
20. Paste JSON.
21. Validate.
22. issue list.
23. diff view.
24. source view.
### Stage 5 — Publication
25. admin authorization.
26. publication transaction.
27. constrained upserts.
28. provenance/version history.
29. cache/search refresh trigger.
30. failure handling.
### Stage 6 — Workflow
31. generated ChatGPT research prompt.
32. existing-university import template.
33. revalidation.
34. comprehensive E2E tests.
Do not build batch/API automation before the single-university workflow is reliable.
---
# 126. Codex implementation constraint
Before implementing this specification:
> Read the existing Data Standard v1, Data Architecture v1, Data Pipeline and Security & Privacy implementation/specifications first.
Do not redesign those systems.
If the Import Specification conflicts with an existing canonical field name:
```text
canonical Data Standard wins
```
unless we explicitly revise Data Standard.
---
# 127. Definition of Done
Import & Ingestion v1 is finished only when:
* one full university bundle can be pasted as JSON;
* JSON is validated against a versioned schema;
* canonical Data Standard is reused;
* unknown fields fail safely;
* internal relationships are validated;
* critical facts require provenance;
* source URLs and scope are preserved;
* AI cannot invent Ekho UUIDs;
* duplicate detection exists;
* existing values are not deleted merely because an import omits them;
* current vs proposed diff exists;
* important changes are highlighted;
* invalid imports cannot publish;
* publishing requires admin authorization;
* publication is atomic;
* import history is auditable;
* stable canonical IDs survive updates;
* unvalidated payload remains outside canonical tables;
* search/cache updates happen after publication;
* university pages automatically render canonical imported data;
* no university-specific React page/code is necessary;
* exact duplicate imports are idempotent;
* all P0 tests pass.
---
# 128. Final invariant
The final Ekho experience must be:
```text
Research
↓
Structured JSON
↓
Paste
↓
Validate
↓
See exactly what will change
↓
Publish
```
Not:
```text
open database
↓
manually create 70 rows
↓
copy values field by field
↓
edit frontend
↓
hope relationships are correct
```
University onboarding into Ekho must be fundamentally **data-driven, source-grounded and repeatable**.
---
# 129. Authority / research sources
This specification is grounded primarily in:
1. **JSON Schema Draft 2020-12** — import contract and structural validation.
2. **RFC 6901** — precise JSON field addressing for evidence/provenance.
3. **PostgreSQL current documentation** — transactions, constraints, foreign keys and conflict-aware inserts.
4. **Supabase documentation** — database functions, privileges, server secrets, RLS and secure database architecture.
5. **OWASP Input Validation / SQL Injection / SSRF / XSS guidance** — hostile input handling and import security.
6. **ISO 3166 / ISO 4217** — country and currency identifiers.
7. **RFC 3339 / RFC 3986** — timestamps and URI representation.
8. **OpenAI Structured Outputs documentation** — optional future schema-constrained AI generation; it does not replace factual source verification.
