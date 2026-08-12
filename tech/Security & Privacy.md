# Ekho Security & Privacy v1.0

**Status:** READY TO LOCK  
**Date:** 2026-08-12  
**Scope:** User data, authentication, authorization, PostgreSQL RLS, private documents, object storage, permissions, secrets, backups, deletion, audit logging, monitoring and incident response.

---

# 1. Purpose

Ekho will store data that users reasonably expect to remain private:

```text
profile data
education history
grades
test scores
application lists
essays
notes
recommendations metadata
financial-aid information
uploaded documents
```

A privacy or authorization failure would directly damage the main product promise:

```text
Trust
```

Security must therefore be an architectural constraint.

Not:

> We'll harden it after launch.

---

# 2. Security baseline

Ekho security should use:

```text
OWASP ASVS
OWASP Cheat Sheet Series
NIST security guidance
PostgreSQL native security controls
GDPR privacy principles where applicable
```

as primary engineering references.

OWASP ASVS exists specifically as a technical security-control and verification standard for web applications.

Ekho target:

```text
OWASP ASVS Level 2
```

as the default application-security baseline.

Additional controls should be applied to:

```text
private files
financial information
administrative access
authentication
```

where risk is higher.

---

# 3. Core security principles

All Ekho engineering must follow:

```text
Deny by default

Least privilege

Server-side authorization

Defense in depth

Data minimization

Private by default

Explicit trust boundaries

Short-lived access

Auditable privileged actions

Secure failure

Recoverable infrastructure
```

OWASP specifically recommends least privilege, deny-by-default authorization and validating permission on every request.

---

# 4. Primary threat model

Ekho must explicitly defend against at least:

```text
User A reading User B's data

User A modifying User B's data

User A deleting User B's data

IDOR / broken object-level authorization

RLS misconfiguration

public object-storage exposure

stolen signed file URLs

malicious file uploads

XSS

CSRF

SQL injection

credential stuffing

session theft

privilege escalation

admin-account compromise

service-key leakage

secrets committed to Git

database backup theft

accidental production deletion

ransomware / infrastructure failure

log leakage

third-party/vendor compromise

over-permissioned employees

deleted-user data surviving indefinitely
```

Horizontal privilege escalation—one authenticated user reaching another user's resources—is specifically identified by OWASP as a major authorization risk.

---

# 5. Security boundary

Never assume:

```text
frontend = trusted
```

Frontend is always untrusted.

The browser may send:

```text
fake user_id
fake role
fake application_id
fake document_id
fake completed state
modified API request
```

Every protected action must be verified server-side and/or at the database layer.

---

# 6. User identity

Canonical ownership identity:

```text
user_id
```

must come from the authenticated server-side identity/session.

Never accept canonical ownership from:

```text
request.body.user_id
query.user_id
hidden form input
localStorage user_id
```

---

# 7. Data classification

Ekho data should be classified before permissions are designed.

Use four classes.

---

# 8. CLASS 0 — Public

Examples:

```text
universities
programs
public admissions requirements
public deadlines
tuition information
official sources
public scholarships
countries
subjects
```

Security priority:

```text
integrity
availability
```

Confidentiality is normally not required.

---

# 9. CLASS 1 — Private account/workspace

Examples:

```text
email
profile preferences
saved universities
application list
application progress
tasks
notes
test scores
education history
```

Required:

```text
authenticated
user-isolated
encrypted in transit
encrypted at rest
RLS / equivalent authorization
```

---

# 10. CLASS 2 — Sensitive private

Examples:

```text
essays
transcripts
diplomas
recommendation-related documents
financial-aid information
financial documents
uploaded school records
```

Required:

```text
strict private storage
no public URLs
restricted staff access
audit logging
short-lived download authorization
strong retention/deletion rules
```

---

# 11. CLASS 3 — Restricted secrets / credentials

Examples:

```text
database passwords
API keys
OAuth client secrets
signing secrets
storage credentials
encryption keys
service-role credentials
backup encryption keys
```

Never exposed to:

```text
browser
user
analytics
logs
repository
```

OWASP recommends dedicated secrets-management systems, key rotation, separate key storage and explicitly warns against hardcoding or committing keys to source control.

---

# 12. Data that Ekho should avoid collecting

Do not collect data merely because university applications sometimes contain it.

Core Ekho should avoid unnecessary storage of:

```text
passport scans
government ID numbers
social security numbers
medical records
biometric data
bank account numbers
card data
race/ethnicity
religion
sexual orientation
```

unless a separately approved product requirement genuinely requires it.

GDPR principles include purpose limitation, data minimization, storage limitation, integrity and confidentiality.

---

# 13. Data minimization rule

Before adding a personal-data field:

```text
Why do we need it?
Which feature uses it?
Is it required now?
How long do we retain it?
Does a less-sensitive alternative exist?
```

If no strong answer exists:

```text
do not collect it
```

---

# 14. Authentication architecture

Do not build custom password authentication from scratch.

Use a proven authentication provider/library appropriate to the chosen stack.

Authentication system should support:

```text
secure password handling if passwords exist
email verification
session revocation
account recovery
MFA/passkeys
rate limiting
secure session management
```

---

# 15. Staff authentication

All production administrative access must require MFA.

Preferred:

```text
phishing-resistant MFA / passkeys / hardware-backed authentication
```

NIST's current AAL2 guidance requires two distinct factors, and NIST recommends MFA and passkeys as strong account-protection mechanisms.

---

# 16. User MFA

For normal applicants:

```text
MFA should be available
```

but does not need to create heavy onboarding friction in early v1.

For accounts containing particularly sensitive documents, Ekho may later encourage MFA contextually.

---

# 17. Session handling

Sessions must be:

```text
cryptographically strong
revocable
server-verifiable
time-limited
```

For cookie-based sessions:

```text
Secure
HttpOnly
SameSite appropriate to architecture
```

must be configured.

---

# 18. Never expose auth tokens unnecessarily

Do not put long-lived auth/session secrets into:

```text
URL
query string
analytics
logs
client-visible HTML
```

---

# 19. Session revocation

User must be able to invalidate sessions after:

```text
password change
suspected compromise
account recovery
security-sensitive identity change
```

---

# 20. Authorization ≠ authentication

This must remain explicit.

```text
Authenticated
```

means:

> We know who the user is.

It does **not** mean:

> They can access this resource.

OWASP explicitly distinguishes authentication from authorization.

---

# 21. Canonical authorization model

Every protected request conceptually evaluates:

```text
Who is requesting?
↓
What resource?
↓
What operation?
↓
Does this identity own/have permission?
↓
Allow or deny
```

---

# 22. Deny by default

When permission logic cannot confidently prove access:

```text
DENY
```

Never:

```text
permission unknown → allow
```

OWASP explicitly recommends deny-by-default authorization.

---

# 23. Authorization layers

Ekho should use defense in depth:

```text
API authorization
+
database permissions
+
RLS
+
object-storage authorization
```

Do not depend on only one frontend/API `if` statement.

---

# 24. PostgreSQL Row-Level Security

For tables containing user-owned rows:

```text
ENABLE ROW LEVEL SECURITY
```

must be standard.

PostgreSQL RLS can independently restrict which rows can be selected, inserted, updated or deleted. When RLS is enabled and no applicable policy exists, PostgreSQL uses default-deny behavior.

---

# 25. RLS is mandatory on user-owned tables

Examples:

```text
profiles

applications
application_context

education_records
qualification_records
academic_results

test_attempts

user_requirements / evaluations where persisted

tasks
notes

documents
essays

user_preferences

financial_aid_profile

notifications
```

Exact schema may change.

Rule does not:

> User-owned tables require explicit row authorization.

---

# 26. Public reference tables

Tables such as:

```text
institutions
programs
countries
subjects
public_requirements
public_deadlines
official_sources
```

do not require per-user RLS for confidentiality.

They should still have strict write permissions.

Normal users:

```text
READ
```

not:

```text
WRITE
```

---

# 27. Basic ownership model

Preferred direct pattern:

```text
row.user_id = authenticated_user_id
```

Example:

```text
applications.user_id
```

---

# 28. Direct RLS rule

Conceptually:

```text
SELECT
WHERE row.user_id = current_user_id
```

```text
INSERT
WITH CHECK row.user_id = current_user_id
```

```text
UPDATE
USING row.user_id = current_user_id
WITH CHECK row.user_id = current_user_id
```

```text
DELETE
WHERE row.user_id = current_user_id
```

---

# 29. Why `WITH CHECK` matters

An update policy must prevent this pattern:

```text
User A owns row
↓
User A updates row.user_id = User B
↓
row changes ownership
```

Therefore both:

```text
which rows can be accessed
```

and:

```text
what the resulting row may contain
```

must be constrained.

PostgreSQL supports separate policy expressions for visibility and permitted inserted/updated rows.

---

# 30. Nested ownership

Some records may not need a duplicated `user_id`.

Example:

```text
application_requirements
    ↓ application_id
applications
    ↓ user_id
```

Authorization can resolve ownership through the parent.

Conceptually:

```text
EXISTS (
  application
  WHERE application.id = row.application_id
  AND application.user_id = current_user
)
```

---

# 31. Direct ownership duplication

For especially security-critical/high-volume tables, storing:

```text
user_id
```

directly may simplify:

```text
RLS
indexes
authorization review
```

But duplication must have integrity constraints.

Do not allow parent ownership and row ownership to diverge.

---

# 32. Foreign-key ownership attack

Never allow a user to create:

```text
document.application_id = another_user_application
```

even if:

```text
document.user_id = own_user_id
```

Cross-entity references must also validate ownership.

---

# 33. RLS for relation inserts

Example document insert should require:

```text
document.user_id = current_user
AND
application_id belongs to current_user
```

when `application_id` is present.

---

# 34. RLS role danger

Critical PostgreSQL behavior:

```text
superusers
BYPASSRLS roles
table owner
```

may bypass ordinary RLS.

Table owners normally bypass RLS unless:

```text
FORCE ROW LEVEL SECURITY
```

is enabled.

---

# 35. Production runtime database role

The normal application runtime role must **not** be:

```text
superuser
BYPASSRLS
table owner where avoidable
```

It must run under a role subject to RLS.

---

# 36. FORCE RLS

For user-sensitive tables, strongly prefer:

```text
ALTER TABLE ... FORCE ROW LEVEL SECURITY
```

where compatible with the final infrastructure.

Purpose:

```text
protect against accidental owner-context queries
```

---

# 37. Service-role credentials

Any database/service identity capable of bypassing RLS is:

```text
CLASS 3
```

It must never be exposed to:

```text
browser bundle
mobile client bundle
public environment variable
frontend API response
```

---

# 38. Service role use

A bypass-capable service identity should only be used for narrowly scoped operations such as:

```text
controlled migrations
trusted background jobs
backup/maintenance
verified data pipelines
```

Never as the default request identity for all normal user API operations.

---

# 39. Admin bypass

Do not implement:

```text
if role = admin:
  RLS disabled everywhere
```

as normal administrative UX.

Prefer explicitly scoped privileged operations.

---

# 40. RLS migration rule

Any migration adding a user-owned table must include in the same change:

```text
ownership model
RLS enabled
policies
indexes supporting policy
security tests
```

Do not merge:

```text
table now
RLS later
```

---

# 41. RLS default test

New user table with RLS but no policies should be inaccessible.

That is expected.

Do not fix it by granting broad access.

---

# 42. RLS test identities

Automated tests require at least:

```text
User A
User B
anonymous
authorized service role
```

---

# 43. Cross-user SELECT test

For every user-owned table:

```text
User A creates row
User B SELECTs row id
```

Expected:

```text
0 rows / not found
```

---

# 44. Cross-user UPDATE test

```text
User B attempts update User A row
```

Expected:

```text
denied
```

---

# 45. Cross-user DELETE test

Expected:

```text
denied
```

---

# 46. Cross-user INSERT-reference test

User B attempts:

```text
create child object referencing User A parent
```

Expected:

```text
denied
```

---

# 47. Guessable IDs are not authorization

Even if primary IDs are random UUIDs:

```text
UUID secrecy ≠ access control
```

Knowing another resource ID must not grant access.

---

# 48. API object authorization

Every endpoint accepting:

```text
application_id
document_id
essay_id
note_id
```

must verify authorization.

Never assume RLS replaces all application-level permission logic.

Defense in depth remains mandatory.

---

# 49. 404 vs 403

For private resource lookup, prefer responses that do not unnecessarily confirm another user's resource exists.

Example:

```text
404 Not Found
```

may be preferable to:

```text
403: User 738 owns this document
```

depending on endpoint semantics.

---

# 50. Mass assignment protection

Never automatically persist arbitrary client JSON directly into database models.

Bad:

```text
update application with request.body
```

Use explicit accepted fields.

Otherwise users may attempt to modify:

```text
user_id
role
verification_status
admin flags
source authority
security metadata
```

---

# 51. Server-owned fields

Client must not directly control:

```text
user_id

role
permissions

verified
source_verification_status

created_by_system

audit metadata

security flags

billing entitlement

RLS-sensitive ownership

document_scan_status
```

---

# 52. Private document architecture

User documents are never public web assets.

Storage model:

```text
private object storage bucket/container
```

not:

```text
public bucket with hard-to-guess filenames
```

OWASP recommends isolating uploaded files, storing them outside normal public webroot access, authenticating users and enforcing authorization.

---

# 53. Document bucket policy

Default:

```text
PUBLIC ACCESS = OFF
```

No anonymous:

```text
list
read
write
delete
```

---

# 54. Do not use public URLs

Forbidden:

```text
https://storage/.../transcript.pdf
```

that works indefinitely without authentication.

---

# 55. Document object identity

Storage object should use an opaque generated identifier.

Example concept:

```text
documents/{random_uuid}/{random_uuid}.pdf
```

Do not rely on:

```text
George_Transcript_Harvard.pdf
```

as the actual storage key.

---

# 56. Original filename

Original filename may be retained as metadata:

```text
original_filename
```

for user experience.

It is not trusted as:

```text
filesystem path
object key
MIME authority
```

OWASP recommends application-generated filenames and strict filename handling.

---

# 57. Document metadata

Canonical DB record:

```text
document_id

user_id

application_id nullable

document_type

storage_object_key

original_filename

declared_mime
detected_mime

size_bytes

sha256/hash where useful

upload_status

scan_status

created_at
updated_at
```

---

# 58. Storage access flow

Canonical read:

```text
User requests document
↓
server authenticates
↓
server authorizes ownership/permission
↓
server generates short-lived access
↓
browser downloads/views file
```

---

# 59. Signed download URLs

If object-storage signed URLs are used:

Ekho default:

```text
5 minutes
```

Maximum normal duration:

```text
15 minutes
```

for document reads.

These are Ekho security policy values.

Longer durations require a specific use case.

---

# 60. Signed URLs are bearer credentials

Anyone possessing a valid signed URL may be able to use it during its lifetime.

Therefore:

```text
never log complete signed URL
never put in analytics
never email long-lived signed URLs
never persist them as canonical document URLs
```

Generate when needed.

---

# 61. Signed upload URLs

Uploads may use short-lived presigned URLs.

Target expiry:

```text
10 minutes
```

Normal maximum:

```text
30 minutes
```

for unusually large supported files.

---

# 62. Signed upload restrictions

Where provider supports it, bind upload authorization to:

```text
specific object key
maximum size
expected method
content constraints where reliable
expiration
```

---

# 63. Storage listing

Normal users should not receive direct bucket-list permission.

Application knows the document objects from authorized DB metadata.

---

# 64. Document download

Prefer:

```text
attachment
```

for unsafe/unknown formats.

Only render inline formats that have an explicitly safe preview strategy.

---

# 65. File upload allowlist

v1 applicant documents should allow only file types actually needed.

Recommended initial allowlist:

```text
PDF

JPEG
PNG
WebP
```

Consider DOCX only if a concrete workflow needs it.

Do not accept arbitrary:

```text
.exe
.js
.html
.svg uploaded by users
.zip
rar
scripts
executables
```

as application documents.

OWASP recommends allowlisting only business-required extensions.

---

# 66. ZIP uploads

Do not support generic ZIP uploads in v1.

OWASP notes that archive files introduce numerous additional attack vectors.

---

# 67. MIME validation

Never trust only:

```text
Content-Type supplied by browser
```

It can be spoofed. OWASP explicitly recommends validating beyond the request's Content-Type header.

---

# 68. File validation chain

Canonical:

```text
extension allowlist
↓
MIME sanity check
↓
magic bytes / file signature
↓
size validation
↓
parser-safe inspection
↓
malware scan where available
↓
store/mark usable
```

Defense in depth is required because no individual file validation method is sufficient.

---

# 69. File signature

Verify expected file signature where practical.

Example:

```text
.pdf extension
+
PDF signature
```

But signature validation alone is not sufficient; OWASP explicitly warns against relying on it alone.

---

# 70. Upload size limit

Initial Ekho policy:

```text
PDF/document:
20 MB maximum per file

image:
10 MB maximum per file
```

Anything larger should fail before expensive parsing where possible.

---

# 71. User storage quota

v1 should impose a per-user storage quota.

Initial policy:

```text
500 MB per account
```

Raise only if evidence shows genuine need.

This protects:

```text
cost
DoS resistance
abuse
```

---

# 72. File-count limit

Also bound file count.

Initial:

```text
500 private files/account
```

Avoid unlimited tiny-file abuse.

---

# 73. Upload rate limiting

Uploads must have:

```text
rate limits
concurrency limits
```

separate from ordinary API calls.

---

# 74. Malware scanning

Uploaded documents should enter:

```text
quarantine / pending
```

until automated validation/scanning finishes where infrastructure supports it.

OWASP recommends antivirus or sandbox scanning when available and CDR for applicable formats such as PDF/DOCX.

---

# 75. Scan states

```text
pending

clean

rejected

scan_failed
```

Do not use:

```text
null
```

for every state.

---

# 76. Scan failure

If scanner is unavailable:

```text
do not automatically mark clean
```

Depending on feature:

```text
retain pending
retry
```

and prevent high-risk server processing.

---

# 77. Uploaded documents are untrusted forever

Even after scanning:

```text
file = user-controlled input
```

A malware scan reduces risk.

It does not turn file bytes into trusted application code/content.

---

# 78. Document parsing isolation

PDF/image/document parsing should execute in an isolated processing environment where practical.

The main API process should not blindly run complex untrusted parsers with broad infrastructure credentials.

---

# 79. Parser permissions

Document-processing workers should have:

```text
read only required input
write only derived output
no broad database admin access
no secrets unrelated to task
```

---

# 80. Never execute uploaded content

Applicant-uploaded content must never execute as:

```text
server code
browser script
shell input
template
```

---

# 81. HTML upload

Do not support arbitrary user HTML files for inline rendering.

---

# 82. SVG upload

Treat uploaded SVG as potentially active content.

Unless a dedicated sanitization pipeline exists:

```text
do not allow arbitrary SVG applicant documents
```

University logos from trusted/admin sources are a different pipeline.

---

# 83. PDF preview

PDF preview must not make the object public.

Flow:

```text
authorized short-lived file access
→ isolated/safe viewer
```

---

# 84. AI document processing

If documents are sent to an AI/OCR provider:

this becomes:

```text
personal-data processing by a third party
```

and requires:

```text
documented purpose
appropriate processor agreement
data-retention review
transfer review
minimum necessary content
```

Do not automatically send every uploaded document to an AI API.

GDPR distinguishes controller and processor responsibilities and requires appropriate safeguards for processors.

---

# 85. Encryption in transit

All production application traffic:

```text
HTTPS/TLS only
```

No authentication/session/private-document traffic over plaintext HTTP.

---

# 86. Encryption at rest

Use managed infrastructure encryption at rest for:

```text
database
object storage
backups
logs containing protected metadata
```

---

# 87. Highly sensitive encryption

For future exceptionally sensitive data, evaluate application-level/envelope encryption.

Do not invent custom cryptography.

OWASP recommends dedicated key-management systems and separation between encryption keys and data.

---

# 88. Encryption keys

Keys must live in:

```text
KMS
secret manager
managed key vault
```

where available.

Not:

```text
source repository
hardcoded config
frontend env
```

---

# 89. Secret management

All production secrets must be stored in an approved secrets-management system.

Examples:

```text
database credentials
OAuth secrets
email service secrets
storage signing keys
API keys
webhook secrets
```

OWASP recommends centralization, access control, rotation and auditing of secrets.

---

# 90. `.env` files

Local `.env` may be used for local development where appropriate.

Production secrets must not depend on developers manually copying `.env` files onto servers.

---

# 91. Git secrets

Repository must never contain live:

```text
API key
private key
database password
service role token
cloud credential
```

---

# 92. Secret scanning

CI/repository security should detect common committed secret patterns.

If a secret is committed:

```text
removing commit is not sufficient
```

Secret must be considered compromised and rotated.

---

# 93. Secret rotation

Every secret must have a defined ability to be:

```text
rotate
revoke
replace
```

Do not create architecture dependent on unchangeable permanent credentials.

---

# 94. Development vs production secrets

Separate:

```text
development
staging
production
```

credentials.

A staging compromise must not expose production.

---

# 95. Environment separation

At minimum:

```text
development
staging
production
```

must use separate:

```text
databases
storage buckets
credentials
```

where reasonably possible.

---

# 96. Production data in staging

Do not copy full production applicant data into staging.

Default:

```text
synthetic test data
```

If production-derived data is absolutely necessary:

```text
minimize
anonymize/pseudonymize
approve explicitly
```

---

# 97. Developer production access

Developers do not automatically receive direct unrestricted access to:

```text
production DB
private storage
backups
```

because they work on the project.

Least privilege applies internally too.

---

# 98. Internal role model

Recommended:

```text
applicant

support
data_editor
developer
security_admin
infrastructure_admin
```

Do not create one:

```text
admin = can see everything
```

role.

---

# 99. Applicant

Can access:

```text
own profile
own applications
own evidence
own documents
own notes
own settings
public admissions data
```

Cannot access:

```text
other users
internal admin systems
source pipeline controls
security logs
```

---

# 100. Support

Default support permission:

```text
account metadata required to resolve support issue
```

Not automatically:

```text
essays
transcripts
financial documents
private notes
```

---

# 101. Support document access

If a support case genuinely requires viewing a private document:

use controlled elevated access.

Requirements:

```text
explicit reason
user/case context
time-limited access
audit event
```

---

# 102. Data editor

Admissions-data staff can modify:

```text
public institution/program/admissions data
```

They do not need normal access to:

```text
user documents
essays
private profiles
```

---

# 103. Developer

Normal developer role:

```text
no routine production user-data browsing
```

Debug through:

```text
structured logs
request IDs
synthetic fixtures
safe metadata
```

first.

---

# 104. Infrastructure admin

Infrastructure permissions may technically allow broader access.

These accounts must use:

```text
MFA
least privilege
audit logging
separate identities
```

---

# 105. Break-glass access

For exceptional security/production incidents, Ekho may maintain break-glass access.

Requirements:

```text
strong MFA
separate credentials
not used daily
reason required
alert generated
audit log
post-event review
```

---

# 106. No shared admin accounts

Forbidden:

```text
admin@ekho.club
password shared by team
```

Every staff operator needs an individual identity.

Audit logs must identify the actual person.

---

# 107. Permission review

Review privileged accounts:

```text
monthly during early operations
```

and immediately when:

```text
employee/contractor leaves
role changes
account compromise suspected
```

---

# 108. Access revocation

Offboarding must revoke:

```text
SSO/access provider
cloud access
GitHub
database
storage
secrets
monitoring
email
admin console
```

not just the Ekho admin UI.

---

# 109. API security

Protected API routes must:

```text
authenticate
authorize
validate input
limit request size
rate limit where appropriate
return safe errors
```

---

# 110. Input validation

Validate server-side:

```text
type
length
format
enum
range
ownership references
```

Frontend validation exists for UX.

Not security.

---

# 111. SQL

Use:

```text
parameterized queries
ORM/query builder safely
```

Never construct database queries through direct string interpolation of user input.

---

# 112. Search queries

Search filters/expressions must be validated against allowed structured filters.

Do not expose raw search/database DSL to clients.

---

# 113. Error messages

Production errors shown to users must not expose:

```text
SQL
stack traces
storage credentials
internal paths
environment variables
server implementation details
```

OWASP recommends generic client-facing error messages and avoiding technical details.

---

# 114. Rate limits

At minimum rate-limit:

```text
authentication attempts
password reset
email verification
search abuse
document upload
document download link generation
account creation
expensive AI/document processing
```

---

# 115. Rate-limit key

Depending on endpoint use combination of:

```text
user_id
IP
session
device/risk signals
```

Do not depend only on IP for authenticated abuse.

---

# 116. Enumeration protection

Authentication/account-recovery responses should avoid unnecessarily confirming:

```text
this email definitely has an Ekho account
```

where the auth provider can support safe behavior.

---

# 117. CSRF

If authentication uses cookies, state-changing endpoints need appropriate CSRF protections.

Do not assume:

```text
React app = no CSRF
```

---

# 118. CORS

Production CORS must use explicit trusted origins.

Never:

```text
Access-Control-Allow-Origin: *
```

with private credentialed application APIs.

---

# 119. Security headers

Production web responses should configure appropriate:

```text
Content-Security-Policy

Strict-Transport-Security

X-Content-Type-Options: nosniff

Referrer-Policy

Permissions-Policy

frame-ancestors via CSP
```

according to final architecture.

---

# 120. CSP

Prefer a restrictive CSP.

Avoid dependence on:

```text
unsafe-inline
unsafe-eval
```

where technically possible.

Third-party origins must be intentionally enumerated.

---

# 121. XSS

Never render untrusted user strings as raw HTML.

Examples:

```text
notes
essay
university-request text
filenames
search input
```

must be safely encoded/sanitized.

---

# 122. Markdown/rich text

If Ekho later supports Markdown or rich text:

```text
render through a proven sanitizer
```

Do not trust generated HTML.

---

# 123. Security logs

Ekho requires application-level security logging.

OWASP explicitly recommends application logging for security events and audit trails such as data additions, changes, deletions and exports.

---

# 124. Events to audit

At minimum:

```text
login success/failure where provider exposes safely

MFA changes

password/account recovery events

session revocation

role/permission changes

admin access

break-glass access

private document upload

private document download authorization

private document deletion

account export

account deletion request

security-setting changes

RLS/authorization denial anomalies

backup restore operation

production migration

secret rotation
```

---

# 125. Audit event model

Conceptually:

```text
event_id

timestamp

actor_type
actor_id

action

resource_type
resource_id

result

reason_code

request_id

source_ip_hash / limited source metadata where justified

security_severity
```

---

# 126. Do not log private content

Logs must not contain:

```text
passwords
auth tokens
signed URLs
encryption keys
database credentials
essay bodies
document bodies
financial documents
full test/document contents
```

OWASP explicitly recommends excluding tokens, passwords, encryption keys, database connection strings and sensitive personal data from normal logs.

---

# 127. Log identifiers

Use:

```text
user_id
document_id
request_id
```

instead of copying full private content into logs.

---

# 128. Signed URL logs

If infrastructure access logs necessarily capture signed URL query parameters:

configure redaction where provider permits.

Signed credentials must not become long-lived logging artifacts.

---

# 129. Audit-log integrity

Normal application users cannot:

```text
edit
delete
forge
```

security audit records.

---

# 130. Admin audit

Admin actions should produce an audit entry even when successful.

Do not log only failures.

---

# 131. Monitoring

Security monitoring should alert on abnormal patterns such as:

```text
large volume of authorization failures

rapid cross-resource ID probing

mass document downloads

admin login from unusual context

role escalation

large account-export volume

repeated malware uploads

secret scanning detection

backup failure
```

---

# 132. Backups — purpose

Backups protect against:

```text
operator error
migration failure
hardware failure
cloud failure
application bugs
malicious deletion
ransomware
```

NIST recommends protecting backups and testing restoration rather than merely creating backup files.

---

# 133. Database backup policy

Recommended launch baseline:

```text
Point-in-time recovery:
≥ 7 days

Daily backup:
35 days retention

Monthly recovery point:
12 months
```

These are Ekho operational policies, not GDPR-prescribed retention periods.

---

# 134. Why PITR

For active application data, a once-per-day backup could lose nearly:

```text
24 hours
```

of user changes.

PITR substantially reduces potential data loss.

---

# 135. Database RPO

Target:

```text
RPO ≤ 15 minutes
```

for primary user database.

Meaning:

> In a major recoverable database failure, target no more than roughly 15 minutes of committed data loss.

---

# 136. Database RTO

Target:

```text
RTO ≤ 4 hours
```

for major primary database recovery.

Long-term goal:

```text
≤ 1 hour
```

as infrastructure matures.

---

# 137. Backup encryption

All backups containing personal data must be encrypted at rest and during transfer.

---

# 138. Backup access

Backup access is more privileged than ordinary production database access.

Allowed only to:

```text
infrastructure/security recovery identities
```

Not ordinary developers/support.

---

# 139. Backup location

Maintain backup isolation from the primary failure domain where supported.

Do not make every backup destructible through the same ordinary application credential that writes production.

---

# 140. Immutable/protected backups

At least one backup layer should resist accidental or attacker-driven immediate deletion where infrastructure supports:

```text
immutability
object lock
separate account
protected snapshot
```

NIST storage guidance emphasizes storage protection and restoration assurance.

---

# 141. Backup != replica

A database replica is not sufficient as the only backup.

If bad data is deleted:

```text
primary delete
→ replica delete
```

A backup must preserve historical recovery points.

---

# 142. Object-storage backup

Private documents need a recovery strategy separate from DB backups.

Recommended:

```text
storage versioning / backup
+
protected retention
+
object metadata backup
```

---

# 143. Document DB/storage consistency

Database backup without object backup can restore:

```text
document metadata
```

whose file no longer exists.

Object backup without DB metadata can leave orphaned files.

Restore procedures must address both.

---

# 144. Backup test

A backup is not considered healthy merely because:

```text
backup job = success
```

Regular restore testing is mandatory. NIST guidance specifically calls for maintaining and testing restoration capability.

---

# 145. Restore test cadence

Launch baseline:

```text
monthly automated/controlled restore verification

quarterly full recovery exercise
```

---

# 146. Restore test must verify

```text
database starts

schema valid

row counts plausible

RLS still enabled

private documents resolve correctly

authentication integration works

critical application flows work

backup encryption keys available

deletion tombstones are reapplied where required
```

---

# 147. RLS after restore

Critical test:

```text
restored database
→ User A cannot access User B
```

Never assume backup restore preserved every permission configuration correctly without testing.

---

# 148. Backup monitoring

Alert immediately on:

```text
backup failure

PITR disabled

retention unexpectedly changed

backup encryption failure

restore verification failure
```

---

# 149. Backup deletion protection

Destructive commands affecting:

```text
all backups
PITR archive
object versions
```

must require elevated permissions.

---

# 150. Privacy deletion

Users should eventually be able to:

```text
delete account
```

and have their personal production data removed according to defined retention/legal requirements.

The EU GDPR framework includes rights to access, rectification and erasure in applicable circumstances.

---

# 151. Account deletion flow

Canonical:

```text
User requests deletion
↓
reauthenticate
↓
show consequences
↓
confirm
↓
disable/revoke account access
↓
enqueue deletion workflow
↓
delete/anonymize user-owned production data
↓
delete private objects
↓
record minimal non-personal/audit proof where legally justified
```

---

# 152. Reauthentication for destructive operations

Require recent authentication for high-impact actions such as:

```text
delete account
change primary email
disable MFA
download full data export
```

---

# 153. Production deletion target

Ekho policy target:

```text
user-accessible data removed from active production systems within 7 days
```

unless legal/security obligations require specific retention.

---

# 154. Backups and deletion

Immediate physical deletion from every immutable backup may be incompatible with recovery architecture.

Therefore:

```text
deleted production data
may remain inside encrypted backups
until backup expiry
```

but must:

```text
not be restored into normal active use
```

without deletion reconciliation.

---

# 155. Deletion ledger

Maintain a minimal protected deletion/tombstone mechanism sufficient to ensure:

```text
restore old backup
↓
reapply deletions
↓
deleted accounts do not reappear
```

This ledger itself must minimize personal data.

---

# 156. Maximum ordinary backup retention

Under the above policy:

```text
monthly backups: 12 months
```

means user data could physically remain inside encrypted recovery media until that backup expires.

This must be accurately documented in the real privacy policy once architecture is final.

---

# 157. Storage limitation

Do not retain private documents forever merely because storage is cheap.

GDPR includes storage limitation as a core data-processing principle.

---

# 158. Retention policy registry

Every personal-data category should eventually document:

```text
purpose
retention
deletion trigger
backup behavior
legal exception
```

---

# 159. Orphaned uploads

Uploads that never become attached to a valid document record must be garbage-collected.

Example policy:

```text
uncommitted upload
→ delete after 24 hours
```

---

# 160. Deleted application

Deleting one Application should not necessarily delete reusable:

```text
global test scores
global education records
```

But application-specific:

```text
notes
essays
application document copies
```

must follow explicit lifecycle rules.

---

# 161. User data export

Prepare architecture for:

```text
Download my data
```

Export should be generated server-side after authorization.

---

# 162. Export security

Data export is highly sensitive.

Requirements:

```text
reauthentication
short-lived download
audit event
private storage
automatic expiry
```

---

# 163. Export retention

Temporary export package:

```text
delete ≤ 24 hours after generation
```

unless user regenerates.

---

# 164. Analytics privacy

Analytics must not ingest unnecessary:

```text
document contents
essay contents
financial amounts
exact grades if not required
signed URLs
access tokens
```

Use IDs and categorical events.

---

# 165. Search analytics

Free-form search input can contain personal information accidentally.

Therefore define retention/redaction for raw queries.

Do not treat every search string as harmless analytics metadata.

---

# 166. Error monitoring

Before enabling error-reporting SDK:

configure:

```text
request-body scrubbing
header scrubbing
cookie scrubbing
token scrubbing
URL-query scrubbing
```

---

# 167. Session replay

Session replay is **not approved by default** for Ekho.

Reason:

Ekho screens may display:

```text
grades
essays
documents
application details
financial information
```

If ever enabled:

```text
separate privacy/security review
strict masking
field exclusions
limited sampling
vendor review
```

required.

---

# 168. Third-party processors

Maintain an inventory of vendors that receive personal data.

Examples may include:

```text
authentication provider
hosting provider
database provider
object storage
email provider
error monitoring
analytics
AI provider
```

---

# 169. Vendor review

Before adding a processor:

check:

```text
what data it receives
why
retention
data location
security controls
subprocessors
deletion capability
contract/DPA where required
```

GDPR places obligations on controllers when outsourcing personal-data processing to processors.

---

# 170. Data transfers

If Ekho serves EU users while processors operate outside relevant jurisdictions:

data-transfer requirements must be reviewed before production.

Do not assume:

```text
US SaaS account = automatically GDPR-safe
```

---

# 171. Privacy policy accuracy

Never write:

```text
"We immediately delete all data forever"
```

if encrypted backups retain it temporarily.

Never write:

```text
"We never share data"
```

if cloud processors handle it.

Privacy language must describe actual architecture.

---

# 172. Security incident

Security incident examples:

```text
unauthorized document access

database credential leak

RLS failure

public bucket exposure

malware execution

account takeover

admin account compromise

production data deletion

backup compromise
```

---

# 173. Incident response lifecycle

Canonical:

```text
Detect
↓
Triage
↓
Contain
↓
Preserve evidence
↓
Assess affected data/users
↓
Eradicate cause
↓
Recover
↓
Notify where required
↓
Postmortem
↓
Prevent recurrence
```

---

# 174. Incident severity

At least:

```text
SEV-1 Critical
SEV-2 High
SEV-3 Medium
SEV-4 Low
```

---

# 175. SEV-1 examples

```text
confirmed cross-user private-document leak

database dump exposed

service-role secret publicly leaked

large-scale account compromise

production private bucket public
```

---

# 176. Emergency credential rotation

Maintain runbooks for:

```text
database credential rotation
storage key rotation
auth secret rotation
API key rotation
service-role revocation
```

before a real incident.

---

# 177. Data breach obligations

For GDPR-covered personal-data breaches, notification to the competent supervisory authority may be required without undue delay and, where applicable, within 72 hours of becoming aware of the breach.

Therefore:

```text
legal/privacy escalation must begin immediately
```

not on day three.

---

# 178. Do not automatically report every incident as a GDPR breach

Security incident:

```text
≠ automatically reportable personal-data breach
```

Assessment must determine:

```text
whether personal data was affected
risk
legal applicability
notification requirements
```

---

# 179. Vulnerability management

Dependencies must be monitored for security advisories.

Critical exploitable vulnerabilities on internet-facing production paths require urgent remediation.

---

# 180. Dependency policy

Do not add dependencies with:

```text
unknown maintenance
unnecessary permissions
unresolved critical vulnerabilities
```

when a safe alternative exists.

---

# 181. Lockfiles

Production dependency versions must be reproducible through committed lockfiles.

---

# 182. Automated dependency updates

Use automated alerts/update PRs where appropriate.

Do not automatically deploy every dependency release to production without tests.

---

# 183. CI permissions

CI/CD must use least-privileged credentials.

PR builds from untrusted contexts must not automatically receive production secrets.

---

# 184. Production deployment

Only approved protected branches/workflows may deploy production.

This must align with the separate Development Workflow standard.

---

# 185. Database migrations

Production migrations must be:

```text
reviewed
version controlled
reversible/forward-fixable
tested on staging
```

Security-sensitive migrations require explicit review.

---

# 186. RLS migration regression

A migration must not accidentally:

```text
disable RLS
drop policy
change ownership
grant broad table access
```

without CI detecting it.

---

# 187. Schema security test

CI should query system catalogs to verify every table classified as:

```text
user_private
```

has:

```text
RLS enabled
expected policy
```

---

# 188. Database permissions

Avoid broad:

```text
GRANT ALL
```

to runtime roles.

Grant only required:

```text
SELECT
INSERT
UPDATE
DELETE
```

per schema/table as necessary.

---

# 189. DDL permissions

Normal application runtime must not have:

```text
CREATE TABLE
DROP TABLE
ALTER ROLE
CREATE EXTENSION
```

permissions.

---

# 190. Public schema creation

Do not permit untrusted users/roles to create arbitrary database objects in schemas used by trusted functions.

PostgreSQL warns that functions, triggers and RLS policies can create security hazards if untrusted users can define objects in trusted search paths.

---

# 191. Security-definer functions

Avoid unless needed.

If used:

```text
fixed safe search_path
minimal privilege
explicit input validation
reviewed ownership
```

They must never become an accidental RLS escape hatch.

---

# 192. Background jobs

Background workers should use dedicated service identities.

Examples:

```text
document scanner
email worker
live-update worker
requirement recomputation
```

Each receives only required permissions.

---

# 193. No universal worker credential

Do not give every background worker:

```text
full production DB admin
+
full storage admin
```

because it is convenient.

---

# 194. Email security

Never attach highly sensitive Ekho documents to ordinary notification emails by default.

Email:

```text
"You have an update"
```

should link user back into authenticated Ekho.

---

# 195. Password reset links

Password/account recovery tokens must be:

```text
short-lived
single-purpose
unpredictable
```

and invalidated appropriately.

Delegate to auth provider where possible.

---

# 196. Public profile exposure

Ekho v1 has no reason to make applicant profiles public.

Default:

```text
profile = private
applications = private
documents = private
essays = private
scores = private
```

---

# 197. Sharing

Do not implement:

```text
Anyone with link can view application
```

without a separate permissions/sharing specification.

---

# 198. Counselor/curator access

Future counselor/tutor/curator features must **not** inherit blanket account access.

They require a separate delegation model:

```text
specific user consent
specific resources
specific permissions
revocable
audited
```

---

# 199. Future sharing permission example

Conceptually:

```text
viewer
commenter
editor
```

per specific resource/workspace.

Not:

```text
curator = admin
```

---

# 200. Recommendation privacy

Future recommendation content may require especially careful visibility rules.

Do not assume applicant should automatically view every recommender-uploaded document.

That requires a separate product/legal decision.

---

# 201. Financial aid documents

If implemented:

classify as:

```text
CLASS 2 Sensitive Private
```

with no support-staff access by default.

---

# 202. Payment data

If Ekho later accepts subscriptions:

use a PCI-compliant payment provider.

Do not store raw:

```text
card number
CVV
```

in Ekho database.

---

# 203. Backups of secrets

Do not casually include plaintext production secrets in generic database/application backups.

Secret-manager recovery has its own secure procedure.

---

# 204. Production snapshots

Before risky migrations:

temporary snapshots may be appropriate.

They must follow:

```text
encryption
access controls
automatic retention/deletion
```

---

# 205. Test backups

Never send production backups to developers' laptops for debugging.

---

# 206. Local development

Local dev database should contain:

```text
synthetic fixture users
```

not real applicants.

---

# 207. Privacy-safe fixtures

Create reusable fake profiles covering:

```text
international applicant
multiple citizenship
different qualifications
tests
documents metadata
financial-aid state
```

without using real user data.

---

# 208. Data integrity

Security includes protecting data from unauthorized modification.

Examples:

```text
official deadline
requirement rule
source verification
financial aid policy
```

should have different write permissions from user workspace data.

---

# 209. Admissions-data write role

Normal applicants:

```text
cannot edit canonical admissions facts
```

Data Pipeline/admin tools modify those through separately authorized flows.

---

# 210. User state cannot alter source truth

User may mark:

```text
my transcript requested
```

They cannot set:

```text
University requires no transcript
```

in canonical data.

---

# 211. Security ownership matrix

## Public admissions data

```text
Applicant:
read

Data editor:
controlled write

Runtime:
read

Public anonymous:
read where intended
```

---

# 212. Private applicant data

```text
Owner:
read/write according to field

Other applicant:
none

Support:
minimal metadata

Data editor:
none

Anonymous:
none
```

---

# 213. Private files

```text
Owner:
authorized read/upload/delete

Other applicant:
none

Support:
none by default

Data editor:
none

Anonymous:
none

Break-glass:
temporary audited access
```

---

# 214. Security audit data

```text
Applicant:
limited own security history if product exposes it

Support:
limited

Security admin:
read

Normal runtime user:
no direct table access
```

---

# 215. Backups

```text
Applicant:
none

Support:
none

Developer:
none by default

Infrastructure/security recovery role:
controlled access
```

---

# 216. Security testing baseline

Before launch:

```text
SAST
dependency scanning
secret scanning
RLS integration tests
authorization tests
file-upload tests
security-header tests
backup restore test
```

---

# 217. External security review

Before storing significant numbers of real user documents:

obtain an independent security review / penetration test when budget allows.

Highest-priority areas:

```text
auth
RLS
IDOR
private storage
signed URLs
file uploads
admin interfaces
```

---

# 218. Authorization test philosophy

Do not only test:

```text
authorized user succeeds
```

Every protected feature needs negative tests:

```text
anonymous fails
wrong user fails
wrong role fails
expired access fails
malformed ID fails
```

---

# 219. Required SEC test — RLS SELECT

```text
User A creates application
User B queries exact UUID
```

Expected:

```text
not visible
```

---

# 220. SEC — RLS UPDATE

```text
User B updates User A application
```

Expected:

```text
denied
```

---

# 221. SEC — RLS DELETE

Expected:

```text
denied
```

---

# 222. SEC — ownership transfer

```text
User A attempts:
user_id = User B
```

Expected:

```text
denied
```

---

# 223. SEC — nested ownership

```text
User B creates note/document
application_id = User A application
```

Expected:

```text
denied
```

---

# 224. SEC — service role client leak

Production client bundle/search:

```text
service credentials
```

Expected:

```text
none
```

---

# 225. SEC — public storage

Unauthenticated request directly to private object key.

Expected:

```text
denied
```

---

# 226. SEC — expired signed URL

```text
generate
wait expiry
request
```

Expected:

```text
denied
```

---

# 227. SEC — foreign document signing

User B asks API to generate download URL for User A document.

Expected:

```text
denied
no URL generated
```

---

# 228. SEC — filename traversal

Upload name:

```text
../../document.pdf
```

Expected:

```text
storage-generated object identity
no path traversal
```

---

# 229. SEC — spoofed MIME

```text
malicious/non-PDF
filename = file.pdf
Content-Type = application/pdf
```

Expected:

```text
rejected/quarantined
```

---

# 230. SEC — oversized upload

```text
> configured limit
```

Expected:

```text
rejected
```

before expensive processing where possible.

---

# 231. SEC — malicious document

Known malware test fixture:

Expected:

```text
quarantined/rejected
not available as clean document
```

---

# 232. SEC — executable upload

Expected:

```text
rejected
```

---

# 233. SEC — XSS filename

Filename containing:

```text
<script>...
```

Expected:

```text
displayed safely as text
no execution
```

---

# 234. SEC — API mass assignment

Client submits:

```text
role = admin
user_id = other_user
verified = true
```

Expected:

```text
fields ignored/rejected
```

---

# 235. SEC — CORS

Untrusted origin attempts credentialed protected API request.

Expected:

```text
blocked
```

according to configured origin policy.

---

# 236. SEC — account deletion

Delete account.

Expected:

```text
sessions revoked
active user data deletion workflow starts
documents removed
account cannot authenticate
```

---

# 237. SEC — restored deleted account

Restore old backup containing previously deleted user.

Expected:

```text
deletion reconciliation removes/re-disables restored data
```

---

# 238. SEC — backup restore

Quarterly recovery exercise must successfully restore:

```text
database
RLS
critical workflows
private storage linkage
```

---

# 239. SEC — backup authorization

Normal developer/support identity attempts to access backup.

Expected:

```text
denied
```

---

# 240. SEC — secrets

CI scans repository/build artifacts.

Expected:

```text
no production secrets
```

---

# 241. SEC — logs

Trigger:

```text
login
document upload
signed URL
API error
```

Verify logs contain:

```text
safe IDs
```

but not:

```text
token
password
signed URL secret
document body
```

---

# 242. Security acceptance criteria

Security & Privacy is **not ready** until:

-  Data classification exists.
    
-  Public and private schemas/resources are clearly separated.
    
-  Personal-data collection follows minimization.
    
-  Authentication uses proven infrastructure.
    
-  Production staff access requires MFA.
    
-  Sessions are secure and revocable.
    
-  Authorization is checked server-side.
    
-  Default authorization behavior is deny.
    
-  All user-owned tables are identified.
    
-  RLS is enabled on user-owned tables.
    
-  RLS policies exist for SELECT.
    
-  RLS policies exist for INSERT.
    
-  RLS policies exist for UPDATE.
    
-  RLS policies exist for DELETE.
    
-  UPDATE policies prevent ownership reassignment.
    
-  Nested ownership is validated.
    
-  Runtime DB role does not use superuser.
    
-  Runtime DB role does not use BYPASSRLS.
    
-  Table-owner RLS behavior is explicitly addressed.
    
-  FORCE RLS is used where appropriate.
    
-  Service-role credentials never reach clients.
    
-  RLS schema tests run in CI.
    
-  Cross-user tests run automatically.
    
-  Random UUIDs are not treated as authorization.
    
-  API performs object-level authorization.
    
-  Mass assignment is prevented.
    
-  Server-owned fields cannot be client-modified.
    
-  Private documents are stored in private object storage.
    
-  Public bucket access is disabled.
    
-  Private documents have no permanent public URLs.
    
-  Download authorization is short-lived.
    
-  Upload authorization is short-lived.
    
-  Signed URLs are never stored as canonical file URLs.
    
-  Signed URLs are excluded/redacted from logs.
    
-  Storage object keys are generated.
    
-  Original filenames are treated as untrusted metadata.
    
-  File extensions use an allowlist.
    
-  Content-Type alone is never trusted.
    
-  File signatures are validated.
    
-  File size limits exist.
    
-  User storage quotas exist.
    
-  Upload rate limits exist.
    
-  Malware scanning/quarantine exists where feasible.
    
-  Scanner failure does not mean clean.
    
-  Document parsers run with minimal privileges.
    
-  Uploaded content is never executed.
    
-  Arbitrary HTML uploads are not rendered.
    
-  Arbitrary SVG applicant uploads are not allowed without sanitization.
    
-  TLS is mandatory.
    
-  Database/storage/backups are encrypted at rest.
    
-  Encryption keys are separate from source code/data.
    
-  Production secrets use secret management.
    
-  Secret scanning exists.
    
-  Secret rotation is possible.
    
-  Development/staging/production credentials are separate.
    
-  Real production user data is not routinely copied to staging.
    
-  Developers do not automatically receive unrestricted production access.
    
-  Internal permissions use least privilege.
    
-  Support cannot see documents by default.
    
-  Admissions-data editors cannot see private user data by default.
    
-  Break-glass access is auditable and rare.
    
-  No shared staff admin accounts exist.
    
-  Privileged-access reviews occur.
    
-  Staff offboarding revokes all relevant systems.
    
-  Server-side input validation exists.
    
-  Database queries are parameterized.
    
-  Errors do not reveal internal secrets/details.
    
-  Authentication endpoints are rate-limited.
    
-  Upload/download signing endpoints are rate-limited.
    
-  CSRF protection matches authentication architecture.
    
-  CORS is restricted.
    
-  Security headers are configured.
    
-  CSP exists.
    
-  User-generated text cannot produce XSS.
    
-  Security audit logs exist.
    
-  Privileged actions are logged.
    
-  Private content/secrets are excluded from logs.
    
-  Audit events cannot be altered by normal users.
    
-  Security anomalies generate alerts.
    
-  Database PITR exists.
    
-  Database backups are encrypted.
    
-  Backups have restricted permissions.
    
-  Backup failure alerts exist.
    
-  Document storage has a recovery strategy.
    
-  Backup restoration is regularly tested.
    
-  Restore tests verify RLS.
    
-  RPO is defined.
    
-  RTO is defined.
    
-  Account deletion workflow exists.
    
-  Deleted production data does not reappear after restore.
    
-  Backup deletion/expiry is documented accurately.
    
-  Orphaned uploads are cleaned.
    
-  Data export uses strong authorization.
    
-  Temporary exports expire automatically.
    
-  Analytics avoids unnecessary personal data.
    
-  Error-monitoring data is scrubbed.
    
-  Session replay is disabled by default.
    
-  Third-party processors are inventoried.
    
-  Vendor data handling is reviewed.
    
-  Privacy policy reflects actual architecture.
    
-  Incident-response runbook exists.
    
-  Credential-rotation runbook exists.
    
-  Dependency vulnerability monitoring exists.
    
-  CI does not expose production secrets to untrusted builds.
    
-  Security-sensitive migrations receive review.
    
-  All required security tests pass.
    

---

# 243. Things Codex must NOT invent

Codex must never independently introduce:

```text
public user-document buckets

permanent public transcript URLs

frontend service-role keys

browser-accessible DB admin credentials

RLS-disabled user tables

BYPASSRLS runtime role

superuser runtime database connection

one global admin account

one admin role that can see everything

support access to all transcripts

production database copies on developer laptops

production data in staging by default

raw user_id ownership supplied by client

trusting UUID secrecy

client-controlled role fields

client-controlled verification fields

request.body passed directly into DB updates

Content-Type-only file validation

arbitrary file extensions

ZIP uploads by default

executable uploads

raw HTML document rendering

arbitrary user SVG rendering

unlimited uploads

unlimited storage

scanner failure = clean

uploaded files executed or shell-processed unsafely

signed URLs stored permanently

signed URLs sent to analytics

private data in logs

passwords/tokens/secrets in logs

secrets committed to Git

production secrets shared between environments

unencrypted backups

backup access for all developers

replica treated as the only backup

backup system without restore tests

account deletion that leaves active files indefinitely

restoring deleted users from old backups

session replay enabled without privacy review

AI provider receiving every document automatically

custom cryptography

custom password storage

authorization only in frontend

CORS wildcard with credentialed private APIs

technical stack traces shown to users

```

---

# 244. Codex implementation order

## Phase 1 — Data classification

Before writing authorization:

```text
classify each table
classify each storage bucket
define owner
define sensitivity
define allowed actors
```

---

# 245. Phase 2 — Authentication foundation

Implement:

```text
trusted auth provider
session validation
secure cookies/tokens
session revocation
staff MFA
```

---

# 246. Phase 3 — Database roles

Create separate:

```text
migration/admin role

application runtime role

background-worker roles
```

Runtime role:

```text
not superuser
not BYPASSRLS
not table owner where possible
```

---

# 247. Phase 4 — RLS

For every user-owned table:

```text
ENABLE RLS
FORCE RLS where appropriate

SELECT policy
INSERT policy
UPDATE policy
DELETE policy
```

Then negative tests immediately.

---

# 248. Phase 5 — API authorization

Implement:

```text
authenticated identity
ownership checks
field allowlists
input validation
safe errors
```

Do not rely only on RLS.

---

# 249. Phase 6 — Private files

Implement:

```text
private bucket
document metadata
opaque object keys
upload authorization
download authorization
short-lived signed URLs
```

No public file path.

---

# 250. Phase 7 — Upload security

Implement:

```text
allowlist
size limits
MIME validation
signature validation
quota
rate limit
quarantine
malware scan
safe parsing
```

---

# 251. Phase 8 — Secrets / infrastructure permissions

Implement:

```text
secret manager
environment separation
rotation
CI secret scanning
least-privilege cloud roles
```

---

# 252. Phase 9 — Audit / monitoring

Implement:

```text
security audit events
admin actions
document access
permission changes
security alerts
safe log redaction
```

---

# 253. Phase 10 — Backups

Implement and verify:

```text
PITR
daily backups
object recovery
backup encryption
backup isolation
monitoring
```

---

# 254. Phase 11 — Restore

Actually run:

```text
restore DB
restore files
verify RLS
run core flows
```

Do not mark backup work complete before restore succeeds.

---

# 255. Phase 12 — Privacy lifecycle

Implement:

```text
account deletion
document deletion
export
retention
orphan cleanup
restore deletion reconciliation
```

---

# 256. Phase 13 — Security CI

CI must automatically check:

```text
RLS coverage
cross-user isolation
secrets
dependencies
authorization
file-upload rules
headers
```

---

# 257. Phase 14 — Launch review

Before real user documents:

```text
manual security review
RLS review
bucket-policy review
staff-permission review
restore drill
incident-response dry run
```

---

# 258. Canonical RLS security model

```text
CLIENT
↓
authenticated session

API
↓
validate input
authorize action

DATABASE
↓
runtime role subject to RLS

RLS
↓
row belongs to authenticated user?

NO
→ deny

YES
→ perform allowed operation
```

---

# 259. Canonical private-document model

```text
User chooses document
↓
Ekho authenticates user
↓
creates DB document record owned by user
↓
server issues short-lived upload authorization
↓
upload → PRIVATE bucket
↓
validation
↓
malware scan
↓
clean
↓
usable document

READ:

User requests document
↓
API authenticates
↓
API verifies ownership/permission
↓
short-lived signed read URL
↓
file available briefly
↓
URL expires
```

---

# 260. Canonical backup model

```text
PRIMARY DATABASE
↓
continuous/PITR backup

+
daily snapshots

+
protected historical recovery points

PRIVATE OBJECT STORAGE
↓
versioning/backup strategy

↓
encrypted backup storage

↓
restricted recovery identity

↓
regular restore testing

↓
verified RLS + file linkage after restore
```

---

# 261. Final locked security targets

```text
AUTHORIZATION
Deny by default

RLS
All user-owned tables

Runtime DB role
No superuser
No BYPASSRLS

Private documents
Private bucket only

Public document URLs
Never

Download signed URL
5 min target
15 min normal max

Upload signed URL
10 min target

Documents
PDF/JPEG/PNG/WebP initial allowlist

Document size
20 MB max

Image size
10 MB max

User file storage
500 MB initial quota

Staff MFA
Required

Production secrets
Secret manager only

Database PITR
≥ 7 days

RPO
≤ 15 min target

RTO
≤ 4 h target

Daily backups
35 days

Monthly recovery points
12 months

Restore verification
Monthly

Full recovery exercise
Quarterly

Active account-deletion target
≤ 7 days

Orphaned upload retention
≤ 24 h
```

---

# 262. Final security rule

Ekho must assume:

```text
frontend can be manipulated

IDs can be discovered

users may be malicious

files may be malicious

staff accounts may be compromised

cloud credentials may leak

bugs will eventually happen

hardware/services can fail
```

Therefore security cannot depend on:

```text
"Nobody will try this."
```

It must depend on:

```text
They can try it
→
the system still denies access.
```

For Ekho specifically, the most important invariant is:

> **No user, employee, service or accidental configuration should gain access to more applicant data than it explicitly needs.**

And for private documents:

> **A transcript, essay or financial document is private by default, inaccessible by URL alone, and recoverable without becoming publicly exposed.**