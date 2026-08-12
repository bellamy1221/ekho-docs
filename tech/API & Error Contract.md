# API & Error Contract v1
**Status:** LOCKED
**Scope:** HTTP APIs, server boundaries, request/response contracts, errors, validation, pagination, idempotency, concurrency, caching, retries and API security
**Stack:** Next.js + TypeScript + Drizzle ORM + Supabase PostgreSQL/Auth; direct Supabase/PostgREST access only where this contract explicitly permits it
**Depends on:** Data Standard, Data Architecture, Security & Privacy, Auth & Account Lifecycle, Import & Ingestion, Admin & Data Operations, Observability
**Research verified:** 12 August 2026
---
# 1. Goal
Ekho APIs must be:
* predictable;
* small;
* secure;
* typed;
* observable;
* easy for Codex to extend without inventing conventions;
* resistant to accidental duplicate actions;
* consistent across errors;
* compatible with future native/mobile clients;
* independent from raw Supabase/Postgres implementation details where a domain boundary exists.
Every endpoint should make these obvious:
```text
What operation is being requested?
Who may perform it?
What inputs are allowed?
What happens on success?
What can fail?
Can the request be retried safely?
Can it be cached?
```
---
# 2. Do not build an API for everything
Ekho uses **two legitimate data-access paths**.
### Path A — Direct Supabase
Use for simple user-owned operations where:
* RLS fully expresses authorization;
* operation is straightforward CRUD/read;
* no server secret required;
* no third-party API involved;
* no cross-table privileged transaction required;
* no dangerous side effect;
* exposing the table/view shape is acceptable.
Example:
```text
student
→ Supabase client
→ RLS
→ own applications/tasks
```
### Path B — Ekho server API
Use Next.js server endpoints/domain services when:
* privileged authorization required;
* admin action;
* external API/service involved;
* secret required;
* multi-step business transaction;
* account/security operation;
* import publication;
* data merge/archive;
* idempotency required;
* abuse-sensitive operation;
* request/response must hide database implementation;
* operation coordinates several systems.
This keeps architecture simple without losing important security boundaries.
---
# 3. Do not proxy Supabase unnecessarily
Bad architecture:
```text
Browser
→ Next.js API
→ identical Supabase SELECT
→ database
```
for every simple RLS-protected read.
That creates:
* extra latency;
* duplicate code;
* more failure points;
* more maintenance.
Do not add a backend wrapper unless it provides a real domain/security boundary.
---
# 4. Route Handlers are public endpoints
Next.js Route Handlers are HTTP endpoints accessible to clients; they must be treated as exposed API surfaces rather than trusted internal functions.
Therefore every Route Handler must independently consider:
```text
authentication
authorization
input validation
rate limits
CSRF/origin where relevant
logging
error mapping
```
Never assume:
```text
only our frontend knows this URL
→ therefore it is safe
```
---
# 5. Server Actions are not authorization
If Ekho uses Next.js Server Actions later:
they remain entry points into server-side business logic.
They do not replace:
```text
authorization
ownership validation
domain validation
```
Core domain operations should remain reusable independently of whether they were called through:
```text
Route Handler
Server Action
background job
admin operation
```
---
# 6. Domain service layer
High-impact operations should follow:
```text
transport
→ authentication
→ authorization
→ request validation
→ domain service
→ database/provider
→ domain result
→ HTTP response mapping
```
Do not embed entire business workflows directly inside `route.ts`.
---
# 7. HTTP standard
Ekho follows current HTTP semantics defined by **RFC 9110** rather than inventing meanings for status codes.
Use existing HTTP semantics whenever they adequately describe the outcome.
---
# 8. API format
Ekho custom HTTP APIs use:
```text
JSON
```
for normal structured request/response bodies.
Request:
```text
Content-Type: application/json
```
Success:
```text
Content-Type: application/json
```
Errors:
```text
Content-Type: application/problem+json
```
RFC 9457 defines `application/problem+json` specifically as the standardized JSON representation for HTTP API problem details.
---
# 9. No universal success wrapper
Do **not** return:
```json
{
  "success": true,
  "status": 200,
  "message": "Success",
  "data": {}
}
```
HTTP already communicates success/status.
Prefer:
```json
{
  "id": "...",
  "status": "in_progress"
}
```
or for collections:
```json
{
  "items": [],
  "page": {}
}
```
Less ceremony = fewer inconsistent contracts.
---
# 10. Errors are different
Errors use one standardized structure:
**RFC 9457 Problem Details.**
RFC 9457 exists specifically to avoid every API defining its own incompatible error structure and supersedes RFC 7807.
---
# 11. Canonical error response
Ekho extension:
```json
{
  "type": "https://ekho.club/problems/validation-failed",
  "title": "Request validation failed",
  "status": 422,
  "detail": "Some fields are invalid.",
  "instance": "urn:ekho:request:01ABC...",
  "code": "VALIDATION_FAILED",
  "request_id": "01ABC...",
  "errors": []
}
```
Core RFC fields:
```text
type
title
status
detail
instance
```
RFC 9457 defines these fields and allows problem-specific extension members.
Ekho extensions:
```text
code
request_id
errors
retryable
```
where appropriate.
---
# 12. `type`
`type` is the canonical problem-type identity.
RFC 9457 specifies that consumers should use the `type` URI as the primary problem identifier and encourages stable resolvable absolute URIs where practical.
Ekho convention:
```text
https://ekho.club/problems/<problem-name>
```
Examples:
```text
/problems/validation-failed
/problems/auth-required
/problems/resource-not-found
/problems/version-conflict
/problems/rate-limited
```
---
# 13. `code`
Ekho additionally uses a machine-friendly stable code:
```text
VALIDATION_FAILED
AUTH_REQUIRED
FORBIDDEN
RESOURCE_NOT_FOUND
VERSION_CONFLICT
RATE_LIMITED
```
Frontend code may branch on:
```text
code
```
Never branch on human `detail` text.
Supabase likewise recommends branching programmatically on stable error codes rather than mutable error-message strings.
---
# 14. Code format
Error code convention:
```text
UPPER_SNAKE_CASE
```
Examples:
```text
EMAIL_ALREADY_USED
APPLICATION_NOT_FOUND
DEADLINE_INVALID
IMPORT_VERSION_CONFLICT
SOURCE_CONFLICT
```
Once a code is consumed by production clients, changing its meaning is a breaking contract change.
---
# 15. `title`
`title`:
* short;
* describes the problem type;
* normally stable between occurrences.
RFC 9457 specifies that `title` should generally remain constant for a given problem type except localization.
Example:
```text
Request validation failed
```
---
# 16. `detail`
`detail` describes the specific occurrence.
Example:
```text
The application round does not belong to the selected admission cycle.
```
Do not expose:
```text
SQL
stack trace
internal table name
secret
provider credential
```
through `detail`.
---
# 17. `status`
When included:
```text
status
```
must equal the actual HTTP response status.
RFC 9457 explicitly requires generators to keep the Problem Details `status` member consistent with the real HTTP status code.
Never:
```text
HTTP 200
{
  "status": 500
}
```
---
# 18. `instance`
Use `instance` to identify one specific failure occurrence.
Ekho:
```text
urn:ekho:request:<request_id>
```
RFC 9457 permits `instance` to be an opaque URI identifying the individual problem occurrence.
Do not expose internal database paths through it.
---
# 19. Validation error extension
Validation errors:
```json
{
  "errors": [
    {
      "pointer": "/profile/country",
      "code": "INVALID_COUNTRY",
      "detail": "Unsupported country code."
    }
  ]
}
```
RFC 9457 specifically demonstrates an `errors` extension containing multiple validation issues located using JSON Pointer.
---
# 20. Error pointer
For JSON bodies use JSON Pointer-compatible paths:
```text
/email
/program/id
/requirements/2/test
```
Do not use UI-specific paths such as:
```text
form.field3
```
API validation errors describe the request contract.
UI maps them to fields separately.
---
# 21. Multiple errors
When several validation errors share the same problem type:
return one:
```text
VALIDATION_FAILED
```
problem containing multiple `errors`.
RFC 9457 explicitly supports this approach.
For unrelated failures:
return the most relevant problem rather than an arbitrary bag of unrelated error types.
---
# 22. Human text is not API logic
Never do:
```typescript
if (error.message.includes("duplicate"))
```
Supabase's current guidance explicitly recommends branching on `error.code`, because messages can change while codes are designed for programmatic handling.
---
# 23. Hide raw Supabase errors
Browser UI should not receive raw:
```text
Postgres error
PostgREST hint
database table
constraint name
```
unless intentionally exposed to trusted admin debugging.
Supabase/PostgREST errors can include detailed database messages and hints suitable for engineering diagnosis.
Ekho server boundary must map those into domain problems.
---
# 24. Internal error preservation
Internally log:
```text
Ekho error code
+
original provider error code
+
request_id
+
trace_id
```
where safe.
Example:
```text
VERSION_CONFLICT
postgres = 23505
```
Do not send every internal field to user.
---
# 25. Status code policy
Primary Ekho statuses:
```text
200 OK
201 Created
202 Accepted
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
412 Precondition Failed
413 Content Too Large
415 Unsupported Media Type
422 Unprocessable Content
429 Too Many Requests
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```
Use additional standard statuses only when their semantics genuinely fit.
---
# 26. `200 OK`
Use for successful operation returning a response representation.
Examples:
```text
GET university
PATCH application returning updated application
POST validation operation returning result
```
---
# 27. `201 Created`
Use when the HTTP operation creates a new identifiable resource.
Example:
```text
POST /api/applications
→ 201
```
Return the created representation or identifier.
Where useful include:
```text
Location
```
for the new resource.
---
# 28. `202 Accepted`
Use when request is accepted but work will continue asynchronously.
Example:
```text
POST /api/admin/reverification
→ 202
```
Response:
```json
{
  "job": {
    "id": "...",
    "status": "queued"
  }
}
```
Do not hold interactive connections open for long background operations.
---
# 29. `204 No Content`
Use when operation succeeds and returning a representation provides no value.
Examples:
```text
DELETE notification preference
simple revocation
```
Do not attach JSON body to `204`.
---
# 30. `400 Bad Request`
Use for malformed request-level input such as:
* invalid JSON syntax;
* malformed query parameter syntax;
* missing structurally required request information;
* invalid request format that cannot be interpreted.
Do not use 400 for every possible business error.
---
# 31. `401 Unauthorized`
Meaning in HTTP practice:
> authentication credentials are missing/invalid.
Ekho examples:
```text
not signed in
expired/invalid auth session
```
Do not use `401` for authenticated users who merely lack permission.
---
# 32. `403 Forbidden`
Use when:
```text
identity known
+
operation understood
+
user not authorized
```
Examples:
```text
student tries admin endpoint
viewer tries publish operation
```
---
# 33. `404 Not Found`
Use when target resource does not exist.
For private objects, Ekho may also intentionally use:
```text
404
```
instead of revealing that an inaccessible object exists.
This prevents object enumeration.
OWASP identifies authorization checks on every object identifier as essential because manipulating IDs is a common API attack path.
---
# 34. `409 Conflict`
Use when request conflicts with **current state**.
RFC 9110 defines 409 specifically for requests that cannot complete because of a conflict with current target-resource state.
Examples:
```text
duplicate canonical university
application already exists
merge conflict
invalid state transition
```
---
# 35. `412 Precondition Failed`
Use when a conditional request such as:
```text
If-Match
```
fails because the resource changed.
RFC 9110 defines conditional requests using `If-Match` specifically to prevent unintended updates and returns 412 when the condition fails.
This is preferred for standards-based optimistic concurrency.
---
# 36. `413 Content Too Large`
Use when body/upload exceeds allowed size.
Examples:
```text
oversized Import JSON
oversized request body
```
Do not attempt expensive processing before enforcing known limits.
---
# 37. `415 Unsupported Media Type`
Use when endpoint expects:
```text
application/json
```
but receives an unsupported representation type.
---
# 38. `422 Unprocessable Content`
Use when:
```text
content type valid
+
syntax valid
+
semantic/domain validation failed
```
RFC 9110 defines 422 exactly for content whose syntax is correct but whose instructions cannot be processed semantically.
Example:
```text
deadline date exists
but occurs before application opens
```
---
# 39. `429 Too Many Requests`
Use when request rate/abuse limits are exceeded.
RFC 6585 defines 429 for rate limiting and permits use of `Retry-After`.
Ekho may intentionally avoid exposing exact internal anti-abuse thresholds.
---
# 40. `500 Internal Server Error`
Use for an unexpected Ekho failure.
Client message:
```text
Something went wrong.
```
Server logs receive:
```text
request_id
trace
actual exception
```
Never return stack trace to user.
---
# 41. `502 Bad Gateway`
Use where Ekho acts as intermediary and receives an invalid/unusable response from required upstream dependency.
Examples later:
```text
AI provider malformed response
external integration invalid response
```
Do not automatically map every dependency failure to 502.
---
# 42. `503 Service Unavailable`
Use when Ekho cannot temporarily process the operation because of:
```text
temporary overload
maintenance
dependency unavailable
degraded mode
```
RFC 9110 defines 503 as temporary inability to handle requests and permits `Retry-After`.
---
# 43. `504 Gateway Timeout`
Use when Ekho is acting as gateway and an upstream dependency does not respond within required time.
RFC 9110 defines 504 specifically for an upstream timeout encountered while acting as gateway/proxy.
---
# 44. `Retry-After`
When server knows the appropriate retry delay, provide:
```text
Retry-After
```
for temporary failures such as:
```text
429
503
```
RFC 9110 defines `Retry-After` as either an HTTP date or delay in seconds.
Prefer seconds for Ekho machine APIs where practical.
---
# 45. Error retry hint
Ekho may additionally include:
```json
{
  "retryable": true
}
```
This is advisory for our clients.
It does **not** replace HTTP status semantics.
---
# 46. Retryable errors
Normally retryable:
```text
network failure
408 where applicable
429
502
503
504
```
Sometimes retryable:
```text
500
```
depending on operation.
Normally not retryable:
```text
400
401
403
404
409
412
413
415
422
```
without changing client state/input.
---
# 47. No blind retry system
Never globally retry every failed API request.
Bad:
```text
POST failed
→ retry
→ retry
→ retry
```
This can duplicate destructive operations.
Retry behavior must depend on:
```text
HTTP method
domain operation
idempotency
error class
```
---
# 48. Safe-method retry
GET/HEAD may normally be retried after transient network/server failures.
Ekho baseline:
```text
max 2 automatic retries
exponential backoff
jitter
```
for ordinary first-party reads.
This is an Ekho operational baseline.
---
# 49. Mutation retries
Automatically retry POST/PATCH only when:
```text
operation is proven idempotent
OR
idempotency key exists
```
Otherwise surface uncertainty to the caller.
---
# 50. Timeout is not proof of failure
Critical rule:
```text
client timeout
≠
server did not perform operation
```
Example:
```text
POST create application
server commits
network dies before response
```
Retrying blindly may create a duplicate.
This is why idempotency matters.
---
# 51. Idempotent HTTP methods
Implement:
```text
PUT
DELETE
```
with genuinely idempotent domain behavior when used.
Do not claim an operation is idempotent merely because it uses PUT/DELETE.
---
# 52. Idempotency-Key
For non-idempotent operations that may reasonably be retried, support:
```text
Idempotency-Key
```
The IETF HTTPAPI working group currently maintains an Internet-Draft specifically defining this header to make operations such as POST/PATCH fault-tolerant; as of August 2026 it remains a draft rather than a finalized RFC.
Ekho adopts the convention intentionally.
---
# 53. Idempotency key format
Client generates a high-entropy unique value.
Recommended:
```text
UUIDv4
```
or comparable cryptographically strong unique identifier.
Do not use:
```text
user email
timestamp alone
application name
```
as the key.
---
# 54. Idempotency scope
Key uniqueness should be scoped by at least:
```text
authenticated actor
+
operation
+
idempotency key
```
Do not allow User A's key to collide with User B's operation.
---
# 55. Idempotency payload fingerprint
Store/request-check a fingerprint of relevant request body.
If same idempotency key is reused with a different request:
```text
409 IDEMPOTENCY_KEY_REUSED
```
Never silently interpret it as the first operation.
---
# 56. Idempotency result
If the first request completed:
repeated request with same key returns the stored equivalent operation result rather than performing the side effect again.
---
# 57. Initial idempotency operations
Require for retry-prone high-value actions such as:
```text
create application where duplicate matters
publish import
large admin operation
future payment/subscription mutation
future external-provider side effect
```
Do not require keys for every checkbox toggle.
---
# 58. Domain idempotency first
When possible, design operation itself to be naturally safe.
Example:
```text
mark notification as read
```
can safely mean:
```text
read = true
```
repeated many times.
Do not introduce idempotency storage unnecessarily.
---
# 59. Optimistic concurrency
Use optimistic concurrency when a stale write could destroy newer state.
Primary cases:
```text
admin institutional data
review items
imports
canonical university/program facts
shared future operational resources
```
---
# 60. ETag
For resources supporting HTTP-level optimistic concurrency, server returns:
```text
ETag
```
representing resource version.
Example conceptual:
```text
ETag: "13"
```
---
# 61. If-Match
Mutation sends:
```text
If-Match: "13"
```
If resource is now version 14:
```text
412 PRECONDITION_FAILED
```
RFC 9110 defines `If-Match` specifically as a conditional guard usable to prevent lost updates.
---
# 62. Do not require ETag everywhere
Simple user interactions such as:
```text
task completed = true
```
do not necessarily need full ETag concurrency.
Apply concurrency controls where stale overwrite causes meaningful harm.
---
# 63. Application-level version
Where existing Ekho specifications already use:
```text
version
```
the API may expose that version and derive ETag from it.
Do not maintain:
```text
database version
API version
ETag version
```
as three unrelated values.
---
# 64. Request validation
Every custom mutation validates:
```text
path parameters
query parameters
headers used by domain
request body
```
server-side.
Browser validation is UX only.
---
# 65. Allowlist input
Accept only explicitly supported fields.
Do not:
```text
spread(req.body)
→ database update
```
OWASP API Security identifies broken property-level authorization as a risk when clients can access/change object properties they should not control.
---
# 66. Mass-assignment prevention
Never accept:
```json
{
  "role": "admin",
  "user_id": "someone-else",
  "is_verified": true,
  "created_at": "...",
  "published": true
}
```
simply because those properties exist on the database object.
Request DTO/schema must explicitly define writable properties.
---
# 67. Input DTO ≠ database row
Do not expose every database field merely because TypeScript generated a table type.
Separate:
```text
DatabaseRow
CreateInput
UpdateInput
PublicResponse
AdminResponse
```
where boundaries differ.
---
# 68. Unknown fields
For security/critical mutation requests:
reject unknown properties.
Example:
```text
422 UNKNOWN_FIELD
```
This catches:
* frontend/backend drift;
* AI-generated wrong payloads;
* malicious mass assignment.
---
# 69. Null vs omitted
Follow Data Standard semantics.
For PATCH-like operations:
```text
field omitted
→ unchanged
```
```text
field: null
→ explicit null/clear only if field permits it
```
Never interpret omission as deletion automatically.
---
# 70. IDs
Canonical IDs:
```text
UUID
```
where Data Architecture defines them.
Client-supplied IDs must still undergo authorization.
OWASP explicitly warns that UUID/random identifiers do not remove the need for object-level authorization.
---
# 71. Never trust `user_id` from client
For user-owned creation:
Bad:
```json
{
  "user_id": "..."
}
```
Correct:
```text
authenticated identity
→ server/RLS derives owner
```
Do not allow client to select arbitrary ownership.
---
# 72. Object authorization
Every operation using:
```text
application_id
document_id
task_id
user_id
admin object id
```
must ensure caller may access that exact object.
OWASP classifies missing per-object authorization as its API1:2023 risk.
---
# 73. Function authorization
Endpoint existence does not grant permission.
Examples:
```text
/admin/imports/:id/publish
/admin/universities/:id/archive
/account/delete
```
require specific authorization independently of route visibility.
OWASP identifies broken function-level authorization as a major API risk.
---
# 74. Response property authorization
Do not return:
```text
private admin notes
internal quality metadata
provider identifiers
security flags
```
because the underlying database query happened to include them.
OWASP identifies unauthorized exposure of individual object properties as an API security problem.
---
# 75. Route naming
Custom Ekho HTTP routes use:
```text
/api/<resource>
```
Examples:
```text
/api/applications
/api/applications/:id
/api/admin/imports/:id
```
Use plural nouns for collections.
---
# 76. Domain actions
Not every business operation must be awkwardly forced into CRUD.
Acceptable:
```text
POST /api/admin/imports/:id/publish
POST /api/admin/imports/:id/cancel
POST /api/account/delete
```
These represent explicit domain commands.
Clarity is more important than pretending everything is a database table.
---
# 77. Avoid deep nesting
Prefer:
```text
/api/tasks/:id
```
over:
```text
/api/users/:userId/applications/:applicationId/tasks/:taskId
```
when task identity already uniquely resolves ownership/context.
Authorization still checks ownership.
---
# 78. Query parameters
Use query parameters for:
```text
filter
sort
search
pagination
view options
```
Example:
```text
?country=US&level=undergraduate
```
Do not encode filtering logic into dozens of custom path patterns.
---
# 79. No generic filtering language
Do not expose raw PostgREST filtering syntax through custom Ekho APIs.
Bad:
```text
?or=(status.eq.x,and(...))
```
for an Ekho domain endpoint.
Define intentional query parameters per endpoint.
This avoids coupling public/domain contracts directly to PostgREST internals.
---
# 80. Pagination default
Potentially unbounded/changeable collections use **cursor pagination**.
Examples:
```text
admin review queue
notifications
activity/history
search results where appropriate
```
---
# 81. Cursor response
Canonical:
```json
{
  "items": [],
  "page": {
    "next_cursor": "...",
    "has_more": true
  }
}
```
Do not include an exact total by default.
---
# 82. Cursor opaque
Client must treat:
```text
next_cursor
```
as opaque.
Do not require client to decode or construct it.
Server may change internal cursor representation later.
---
# 83. Stable cursor ordering
Cursor pagination requires deterministic ordering.
Example:
```text
created_at DESC
+
id DESC
```
to break ties.
Never paginate dynamic data on a non-deterministic sort.
---
# 84. Page limit
Every collection endpoint has bounded:
```text
limit
```
Ekho baseline:
```text
default 25
maximum 100
```
unless endpoint semantics justify another bound.
This is an Ekho operational default.
---
# 85. No unlimited list
Prohibited:
```text
GET /api/universities
→ entire global database
```
Resource limits are an explicit API security concern; OWASP recommends controlling record counts, payload sizes and other resource consumption.
---
# 86. Exact totals
Only return:
```text
total_count
```
where:
* UX actually needs it;
* computing it is cheap;
* semantics are clear.
Do not execute expensive counts automatically on every paginated request.
---
# 87. Sorting
Explicit allowlist:
```text
sort=deadline
sort=updated_at
```
Do not accept arbitrary SQL column/order strings.
---
# 88. Search endpoint
Search-specific behavior stays under Search specification.
API Contract only requires:
* bounded result count;
* validated filters;
* deterministic response;
* stable errors;
* no raw query-language injection.
---
# 89. Response dates/times
Machine API uses canonical formats from Data Standard.
Do not localize JSON API values.
Example:
```json
{
  "deadline": "2027-01-05"
}
```
Localized display belongs to UI/i18n layer.
---
# 90. Enum values
API returns canonical language-neutral enums:
```text
missing
satisfied
optional
unknown
```
not localized UI strings.
---
# 91. Unknown values
Follow Data Standard exactly.
Do not convert:
```text
unknown
```
into guessed field values merely because frontend prefers a concrete answer.
---
# 92. API versioning — current web app
Ekho v1 does **not** need:
```text
/api/v1
```
for every private first-party web endpoint.
The Next.js web frontend and backend are deployed together.
Avoid versioning ceremony before multiple independently deployed API clients exist.
---
# 93. When versioning becomes mandatory
Introduce explicit compatibility/version strategy before supporting:
```text
native iOS/Android clients
third-party developers
public API
external partners
clients that cannot update atomically
```
Do this **before** such clients launch, not afterward.
---
# 94. Breaking change
Examples:
```text
removing response field
changing field meaning
changing enum semantics
changing required request field
changing error-code meaning
changing URL behavior
```
are breaking for independent API consumers.
Adding an optional field generally should not require a new API generation where clients follow tolerant parsing.
---
# 95. OpenAPI
Document custom Ekho HTTP endpoints using **OpenAPI**.
OpenAPI defines a language-agnostic description format for HTTP APIs, allowing tools and humans to understand endpoint contracts without reading source code.
The currently published OpenAPI Specification is **3.2.0**, released in September 2025.
---
# 96. OpenAPI scope
Document:
```text
custom Next.js APIs
admin APIs
future external API
```
Do not manually duplicate every direct Supabase/PostgREST CRUD operation into a second huge OpenAPI document unless it is actually part of Ekho's stable domain API.
---
# 97. Contract source of truth
Do not maintain three incompatible definitions:
```text
TypeScript type
validation schema
OpenAPI schema
```
without synchronization.
Preferred model:
```text
runtime schema / canonical contract
→ TypeScript types
→ OpenAPI representation
→ tests
```
or enforce equivalence through CI.
Exact code-generation library is implementation detail.
---
# 98. API contract CI
CI should fail for:
* invalid OpenAPI;
* undocumented custom endpoint where policy requires docs;
* response/schema mismatch in contract tests;
* duplicate conflicting error codes.
---
# 99. Request ID
Every custom server request receives:
```text
request_id
```
Return:
```text
X-Request-ID
```
for support/debugging.
`X-Request-ID` is an Ekho convention, not an IETF standard.
---
# 100. Trace context
Distributed tracing should propagate standardized:
```text
traceparent
tracestate
```
W3C Trace Context defines these headers to propagate trace identity consistently across systems/vendors.
Do not invent a second incompatible tracing protocol.
---
# 101. Request ID vs trace ID
They are related but distinct.
```text
request_id
```
= easy human support identifier.
```text
trace_id
```
= distributed observability identity.
One request may participate in a larger distributed trace.
---
# 102. Errors expose request ID
Every unexpected/operational error should provide:
```text
request_id
```
Example UI:
```text
Something went wrong.
Reference: 01ABC...
```
User can provide it to support without seeing internal stack traces.
---
# 103. Logging
API logs include:
```text
request_id
trace_id
route template
method
status
duration
error_code
```
according to Observability standard.
Never log full private request bodies by default.
---
# 104. Cache principle
Caching is allowed only when semantics and data sensitivity are understood.
RFC 9111 is the current HTTP caching standard.
Never let framework/CDN defaults decide sensitive caching accidentally.
---
# 105. Private responses
Authenticated sensitive endpoints default to:
```text
Cache-Control: no-store
```
Examples:
```text
account
documents
security settings
admin
private applications
personalized requirements
```
RFC 9205 clarifies that `no-store` prevents storage, while `no-cache` still permits storage followed by validation.
---
# 106. Public institutional data
Published non-personalized university/program data may use explicit shared caching.
Example conceptual:
```text
Cache-Control: public, ...
```
Exact TTL follows data freshness/cache strategy.
Never combine:
```text
public cache
+
user-personalized content
```
in one response.
---
# 107. Cache key correctness
If response varies by:
```text
locale
authorization
query/filter
```
cache strategy must account for that variation.
A German response must not become English cache output.
A User A response must never become User B output.
---
# 108. ETag for public reads
Public stable institutional data may use:
```text
ETag
```
and conditional GET requests where useful.
This allows clients/caches to validate whether a representation changed without downloading it again.
RFC 9110 defines entity tags and conditional requests for this purpose.
---
# 109. External dependency timeout
Every server-side HTTP call to a third party must have an explicit timeout.
No:
```text
fetch(...)
```
that can wait indefinitely.
OWASP specifically identifies missing timeouts and unsafe handling of third-party API responses as risks when consuming external APIs.
---
# 110. Ekho timeout baseline
Initial interactive external-call budget:
```text
~8 seconds maximum
```
unless operation has a separately documented requirement.
If an operation naturally takes longer:
```text
202 + background job
```
instead.
This is an Ekho architecture decision.
---
# 111. DB request target
Normal user-facing DB operations should meet SLO targets defined in Observability.
Do not solve slow database work by raising API timeout to 60 seconds.
Fix:
```text
query
index
data model
workflow
```
first.
---
# 112. External response validation
Treat third-party API data as untrusted input.
Validate:
```text
HTTP status
content type
response schema
size
required fields
```
before integrating it.
OWASP API10:2023 specifically warns against blindly trusting third-party API responses and recommends validation, size/resource limits and timeouts.
---
# 113. Redirects from providers
Do not blindly follow arbitrary third-party redirects in sensitive/server-fetch workflows.
Follow SSRF and external-fetch rules from Security/Data Pipeline.
---
# 114. Provider errors
Map dependency-specific failures into Ekho domain errors.
Example:
```text
Supabase PGRST...
→ DATABASE_UNAVAILABLE
```
```text
SMTP provider 5xx
→ EMAIL_DELIVERY_UNAVAILABLE
```
Client should not need knowledge of vendor error taxonomy.
---
# 115. Preserve provider code internally
Internally retain provider code for diagnostics.
Example Supabase/PostgREST:
```text
23505
PGRST...
```
Supabase publishes specific mappings from PostgreSQL/PostgREST error codes to HTTP statuses and recommends code-based handling.
---
# 116. Direct Supabase client errors
When frontend intentionally performs direct Supabase operations:
create a small adapter translating expected errors into canonical domain state.
Do not scatter:
```typescript
if (error.code === ...)
```
through 40 React components.
---
# 117. API resource limits
Every endpoint must consider bounds for applicable inputs:
```text
request body bytes
array length
page size
file size
query length
processing time
concurrent operations
provider calls
```
OWASP API4:2023 specifically identifies unrestricted CPU, memory, storage, bandwidth and third-party resource consumption as an API risk.
---
# 118. Rate limiting
Apply according to abuse/security risk rather than one universal rate.
Examples:
```text
auth → Auth specification
imports → strict admin limits
public search → sensible anti-abuse limits
password reset → strict
normal task mutation → generous
```
Do not make normal users feel rate limits during legitimate workflows.
---
# 119. 429 behavior
When rate-limited:
```text
HTTP 429
code = RATE_LIMITED
```
and include:
```text
Retry-After
```
when a meaningful retry time is available.
RFC 6585 explicitly defines this pattern.
---
# 120. Do not expose security algorithm
Response need not reveal:
```text
remaining account attempts
exact anti-bot threshold
internal abuse score
```
when doing so materially weakens abuse protection.
---
# 121. Authentication boundary
Custom protected API:
```text
request
→ verified Supabase identity
→ authorization
→ domain operation
```
Use rules locked in Auth & Account Lifecycle.
Never trust:
```text
user_id request body
email header
frontend state
```
as authentication.
---
# 122. Authorization boundary
Authentication:
```text
Who are you?
```
Authorization:
```text
May you perform this operation on this object?
```
Every endpoint must answer both independently where required.
---
# 123. RLS remains final user-data boundary
Direct Supabase APIs rely on correctly tested RLS.
Custom server endpoints using privileged DB access must manually enforce equivalent domain authorization before privileged execution.
A service credential bypassing RLS makes server authorization especially critical.
---
# 124. Admin API
All admin mutation routes require:
```text
verified identity
+
current admin authorization
+
MFA where specification requires
```
Never infer admin rights from:
```text
/admin URL
frontend route
email domain
```
---
# 125. API CSRF
Cookie-authenticated state-changing browser routes must follow the CSRF/origin rules locked in Security/Auth specifications.
At minimum:
* mutation methods only;
* validate trusted origin where applicable;
* SameSite strategy;
* no state-changing GET.
---
# 126. GET must be safe
Never implement:
```text
GET /api/delete-account
GET /api/admin/import/publish?id=...
```
GET must not intentionally perform destructive state changes.
---
# 127. DELETE semantics
DELETE should remain safely repeatable.
Deleting already-deleted/nonexistent own resource may return:
```text
204
```
where domain semantics permit, making retries easy.
For operations where existence matters, use documented behavior consistently.
---
# 128. PATCH semantics
PATCH updates only supplied mutable fields.
It must not reset omitted fields.
For critical concurrent resources require appropriate version/ETag check.
---
# 129. PUT semantics
Use PUT only when caller conceptually replaces/defines complete target representation or performs naturally idempotent resource-setting semantics.
Do not use PUT simply because "update endpoint sounds RESTful."
---
# 130. Batch APIs
Avoid generic:
```text
POST /api/batch
```
executing arbitrary unrelated operations.
Batch only homogeneous/domain-specific operations where semantics are clear.
---
# 131. Partial batch failure
Every batch endpoint must explicitly define:
```text
atomic all-or-nothing
```
or:
```text
independent per-item result
```
Never leave this ambiguous.
---
# 132. Critical domain batches
Imports/merges follow atomic transaction rules from their own specifications.
Do not invent weaker API behavior around them.
---
# 133. Async jobs
Long operations return job resource.
Conceptual:
```json
{
  "job": {
    "id": "...",
    "status": "queued"
  }
}
```
Allowed states follow operation-specific job specification:
```text
queued
running
succeeded
failed
cancelled
```
---
# 134. Job polling
Conceptual:
```text
GET /api/jobs/:id
```
or a domain-specific job endpoint.
Polling interval must be bounded.
Do not make frontend poll every 100ms.
---
# 135. Polling backoff
Start around:
```text
1–2 seconds
```
then back off for longer jobs.
Later Realtime may replace polling where it materially improves UX.
Do not introduce realtime infrastructure merely because polling exists.
---
# 136. Webhooks
Do not build general webhook platform in v1.
When external providers require inbound webhooks later:
each webhook requires:
```text
provider authentication/signature verification
replay/idempotency handling
payload validation
timestamp/replay protections where supported
audit/logging
```
---
# 137. File APIs
Large private files should use Storage flows defined in Security/Data Architecture.
Do not encode files as giant base64 JSON request bodies.
---
# 138. File metadata
File API returns structured metadata, not unrestricted storage internals.
Example:
```text
id
name
size
mime_type
created_at
```
Never expose private bucket implementation details unnecessarily.
---
# 139. Content-Disposition
Download behavior should deliberately distinguish:
```text
inline-safe preview
attachment
```
according to file/security rules.
Do not allow arbitrary untrusted filename header injection.
---
# 140. API localization
Machine API errors use stable:
```text
type
code
```
Human `title/detail` may eventually be localized.
Internationalization Standard remains authoritative.
Never localize:
```text
error code
enum identity
field path
```
---
# 141. API errors in UI
Frontend must not display backend `detail` blindly when:
* detail may be technical;
* more helpful UI copy exists;
* security requires generic response.
Map stable `code` to product UX.
Example:
```text
SESSION_EXPIRED
→ "Your session expired. Sign in again."
```
---
# 142. Unknown error
Frontend always handles unknown/unrecognized code.
Fallback:
```text
Something went wrong. Try again.
Reference: <request_id>
```
Never crash because backend added a new error code.
RFC 9457 likewise requires consumers to tolerate extensions they do not recognize.
---
# 143. No empty catch
Prohibited:
```typescript
catch {
  return null
}
```
for critical operations.
A failure must become one of:
```text
handled domain state
canonical error
observability event
```
---
# 144. Error ownership
Define mapping centrally.
Conceptual modules:
```text
api/errors
api/problem
api/validation
api/request-id
api/response
```
Do not construct arbitrary error JSON independently in every route.
---
# 145. Domain error class
Internal domain errors may conceptually carry:
```text
code
http_status
public_detail
internal_context
retryable
```
Public serialization strips internal context.
---
# 146. Error taxonomy
Base categories:
```text
AUTH
AUTHORIZATION
VALIDATION
NOT_FOUND
CONFLICT
RATE_LIMIT
DEPENDENCY
TEMPORARY
INTERNAL
```
Specific error codes live beneath these concepts.
Do not create a new error code for every insignificant implementation exception.
---
# 147. Core initial error codes
At minimum:
```text
BAD_REQUEST
VALIDATION_FAILED
AUTH_REQUIRED
SESSION_EXPIRED
FORBIDDEN
RESOURCE_NOT_FOUND
RESOURCE_ALREADY_EXISTS
VERSION_CONFLICT
PRECONDITION_FAILED
RATE_LIMITED
CONTENT_TOO_LARGE
UNSUPPORTED_MEDIA_TYPE
DEPENDENCY_UNAVAILABLE
DEPENDENCY_TIMEOUT
INTERNAL_ERROR
```
Domain specifications may add their own.
---
# 148. Ekho-specific errors
Examples:
```text
APPLICATION_NOT_FOUND
INVALID_APPLICATION_STATE
REQUIREMENT_SCOPE_CONFLICT
SOURCE_CONFLICT
IMPORT_INVALID
IMPORT_ALREADY_PUBLISHED
IMPORT_VERSION_CONFLICT
ADMIN_REQUIRED
```
Do not duplicate synonymous codes:
```text
NOT_AUTHORIZED
NO_PERMISSION
ACCESS_DENIED
FORBIDDEN_USER
```
for the same semantics.
---
# 149. Error registry
Repository maintains one registry/document of public API/domain error codes.
For each:
```text
code
problem type
HTTP status
meaning
retryable?
public?
```
CI/tests should catch duplicates where practical.
---
# 150. Observability integration
Every custom endpoint emits:
```text
method
route template
status
duration
request_id
trace_id
error_code if failed
```
following Observability specification.
Do not log raw private payloads merely because endpoint failed.
---
# 151. API SLO connection
Critical endpoint metrics contribute to SLIs defined in Observability.
Do not maintain one set of endpoint status semantics for UI and another for SLO measurement.
Expected:
```text
422 user validation
```
must not count as backend availability failure.
---
# 152. Dependency failure classification
Example:
```text
Supabase unavailable
→ 503
```
```text
external provider timed out
while Ekho is gateway
→ 504
```
```text
provider returned unusable response
→ 502
```
Use semantics, not arbitrary status assignment. RFC 9110 defines these 5xx distinctions.
---
# 153. API inventory
Maintain inventory of custom production API routes.
OWASP API Security specifically identifies forgotten/deprecated/beta API inventory as an operational security risk.
Do not leave:
```text
/api/test
/api/old-admin
/api/v0-secret
```
accessible in production.
---
# 154. Deprecated endpoints
When a custom endpoint becomes obsolete:
```text
remove consumers
→ verify zero traffic
→ remove endpoint
```
Do not maintain abandoned APIs indefinitely.
---
# 155. Dev-only endpoints
Debug/test endpoints must not ship to production unless explicitly designed for production operation.
Examples prohibited:
```text
/api/debug-user
/api/dump-db
/api/test-email
```
---
# 156. CORS
First-party Ekho APIs should use the narrowest required origin access.
Do not return:
```text
Access-Control-Allow-Origin: *
```
on authenticated/private APIs without explicit reason.
Future public API gets separate CORS policy.
---
# 157. Content Security boundary
CORS is not authorization.
Even an endpoint blocked by browser CORS remains reachable outside a browser.
Always enforce actual auth/authz server-side.
---
# 158. API performance
Avoid payload over-fetching.
Return what endpoint/product workflow requires.
Do not return enormous full university objects when UI only needs:
```text
id
name
country
deadline
```
for a card.
---
# 159. Response projections
Prefer explicit server-defined response DTOs for custom endpoints.
Do not accept:
```text
?fields=any,database,column,secret
```
unless a future public API deliberately supports field selection.
---
# 160. Compression
Allow platform-standard HTTP compression.
Do not manually compress JSON inside:
```text
base64
zip-in-json
```
for ordinary API responses.
---
# 161. Null empty collection semantics
Collection with no records:
```json
{
  "items": [],
  "page": {
    "next_cursor": null,
    "has_more": false
  }
}
```
not:
```text
404
```
The collection exists; it is simply empty.
---
# 162. Single missing resource
Specific resource:
```text
GET /applications/:id
```
that does not exist/should not be disclosed:
```text
404
```
not:
```text
200 null
```
unless a specifically documented optional lookup requires null semantics.
---
# 163. Boolean operations
Avoid meaningless:
```json
{
  "success": true
}
```
Prefer:
```text
204
```
or return actual updated resource/state.
---
# 164. Create race
Uniqueness must ultimately be protected by database constraints, not only:
```text
SELECT if exists
→ INSERT
```
Two concurrent requests can both pass the preliminary check.
PostgREST/Supabase maps PostgreSQL uniqueness violations to conflict-style HTTP errors, including 409 for `23505`.
Map this into Ekho `RESOURCE_ALREADY_EXISTS` or appropriate domain conflict.
---
# 165. Foreign-key race
Database integrity remains authoritative.
PostgREST maps PostgreSQL foreign-key violations into conflict responses, reinforcing the role of database constraints beneath API validation.
Still validate earlier when it produces better UX.
---
# 166. API transactions
When one domain action must be atomic:
execute inside one database transaction/RPC/domain function according to Data Architecture.
Do not:
```text
client calls endpoint A
then B
then C
```
and pretend they are one atomic operation.
---
# 167. Client orchestration
Frontend may orchestrate independent UX actions.
But integrity-critical workflows belong server-side.
Examples:
```text
publish university bundle
merge canonical entities
delete account
```
---
# 168. No secret in URL
Never put:
```text
password
access token
refresh token
service key
recovery secret
document secret
```
inside query parameters/path.
URLs commonly appear in logs/history/referrers.
Use secure headers/body/cookie mechanisms from the relevant security specification.
---
# 169. Request body logging
Default:
```text
DO NOT LOG BODY
```
for:
```text
auth
documents
notes
essays
account
personal profile
```
Operationally safe metadata may be logged separately.
---
# 170. API tests
Every custom endpoint requires tests for:
```text
success
unauthenticated
unauthorized
invalid input
not found
conflict
rate limit where applicable
dependency failure
unexpected failure
```
---
# 171. Contract tests
At minimum verify:
* [ ] status code is correct;
* [ ] success body matches schema;
* [ ] error body matches Problem Details;
* [ ] `code` stable;
* [ ] request ID returned on failures;
* [ ] unknown request fields handled correctly;
* [ ] internal provider errors not leaked.
---
# 172. Authorization tests
For every user-owned object endpoint:
Create:
```text
User A
User B
```
Verify A cannot:
* [ ] read B;
* [ ] mutate B;
* [ ] delete B;
* [ ] infer sensitive B resource existence where policy hides it.
OWASP states every endpoint receiving object identifiers must validate permission to that exact object.
---
# 173. Property tests
Verify malicious body cannot set:
* [ ] `user_id`;
* [ ] admin role;
* [ ] verification state;
* [ ] timestamps;
* [ ] publication state;
* [ ] internal quality state;
* [ ] server-generated IDs where prohibited.
---
# 174. Pagination tests
* [ ] page bounded;
* [ ] cursor opaque;
* [ ] stable sort;
* [ ] no duplicates between pages;
* [ ] no missing items under normal traversal assumptions;
* [ ] malformed cursor returns controlled error;
* [ ] limit maximum enforced.
---
# 175. Idempotency tests
* [ ] first request executes once;
* [ ] retry returns equivalent result;
* [ ] no duplicate DB record;
* [ ] concurrent identical requests execute side effect once;
* [ ] reused key with changed body rejected;
* [ ] key scoped to actor/operation;
* [ ] expired idempotency record behavior documented.
---
# 176. Concurrency tests
* [ ] resource returns ETag/version;
* [ ] valid `If-Match` succeeds;
* [ ] stale `If-Match` returns 412;
* [ ] no stale overwrite occurs;
* [ ] updated representation/version returned after success.
---
# 177. Retry tests
* [ ] GET retries transient error.
* [ ] Retry uses bounded attempts.
* [ ] 429 respects `Retry-After`.
* [ ] mutation without idempotency is not blindly retried.
* [ ] idempotent mutation retry does not duplicate.
* [ ] permanent 4xx is not endlessly retried.
---
# 178. Dependency tests
Simulate:
```text
timeout
invalid JSON
500
503
unexpected schema
oversized response
```
Verify:
* [ ] explicit timeout;
* [ ] provider response validated;
* [ ] canonical Ekho error returned;
* [ ] original provider error logged safely;
* [ ] no secret leakage;
* [ ] user receives stable behavior.
OWASP explicitly recommends timeouts and validating third-party API data instead of trusting it by default.
---
# 179. Cache tests
* [ ] private endpoint cannot be shared-cached;
* [ ] account/security uses no-store;
* [ ] public response cache behavior explicit;
* [ ] locale part of relevant cache identity;
* [ ] personalized result cannot leak cross-user;
* [ ] ETag invalidation works where used.
---
# 180. Error tests
* [ ] all API errors use correct HTTP status;
* [ ] `application/problem+json`;
* [ ] stable `type`;
* [ ] stable `code`;
* [ ] status body equals HTTP status;
* [ ] detail contains no stack;
* [ ] unknown errors become `INTERNAL_ERROR`;
* [ ] frontend handles unknown code gracefully.
---
# 181. Resource-consumption tests
Test:
```text
huge limit
huge JSON
huge array
many requests
slow provider
```
Verify bounded behavior.
OWASP treats unbounded API resource consumption as a major API security risk because requests consume CPU, memory, storage, bandwidth and paid provider capacity.
---
# 182. P0 failures
Any of these blocks production:
* user can manipulate object ID to access another user's data;
* client can mutate protected database properties;
* student can invoke admin function;
* service secret appears client-side;
* raw SQL/provider errors expose sensitive internals;
* private authenticated response becomes shared-cacheable;
* state-changing GET exists;
* request timeout can cause duplicate high-impact operation with no idempotency protection;
* stale admin write silently overwrites newer canonical data;
* unlimited endpoint can exhaust database/provider resources;
* route accepts arbitrary body fields directly into DB;
* error response exposes token/secret;
* third-party response is trusted without validation;
* critical custom endpoint has no contract tests.
---
# 183. Implementation order for Codex
## Stage 1 — Foundation
1. Define canonical Problem Details type.
2. Define domain error class.
3. Create error registry.
4. Request ID middleware/helper.
5. Trace context integration.
6. Response helpers.
## Stage 2 — Validation
7. Route request schemas.
8. path/query/body validators.
9. unknown-field policy.
10. DTO vs database types.
11. provider error adapters.
## Stage 3 — Security boundaries
12. auth helper.
13. authorization helper.
14. object ownership checks.
15. admin permission checks.
16. request-size limits.
17. rate-limit response standard.
## Stage 4 — Reliability
18. timeout wrapper.
19. retry policy.
20. `Retry-After`.
21. idempotency storage/helper.
22. idempotent-operation tests.
23. concurrency/ETag helper where required.
## Stage 5 — Collections
24. cursor pagination.
25. bounded limits.
26. validated filters.
27. deterministic sorting.
## Stage 6 — Contract
28. OpenAPI for custom endpoints.
29. contract/schema CI.
30. route inventory.
31. deprecated-route cleanup rules.
## Stage 7 — Testing
32. authorization matrix.
33. property/mass-assignment tests.
34. pagination tests.
35. retry/idempotency tests.
36. dependency failure tests.
37. cache tests.
38. Problem Details tests.
39. P0 security tests.
Do not rewrite simple safe Supabase/RLS CRUD into Route Handlers merely to satisfy this specification.
---
# 184. Codex implementation constraint
Before changing API architecture, Codex must read:
```text
Data Standard
Data Architecture
Security & Privacy
Auth & Account Lifecycle
Import & Ingestion
Admin & Data Operations
Observability
Internationalization & Localization
```
Do not:
```text
duplicate Supabase unnecessarily
change canonical database models
invent another error format
return raw Supabase errors
create universal REST abstractions
add API versioning ceremony without consumers
```
---
# 185. Definition of Done
API & Error Contract v1 is complete when:
* clear rule exists for direct Supabase vs Ekho server API;
* Route Handlers are treated as exposed security boundaries;
* custom APIs use predictable HTTP semantics;
* all custom API errors follow RFC 9457;
* stable error-code registry exists;
* raw provider/database errors do not reach normal users;
* request IDs exist;
* W3C trace context propagates;
* request validation is server-side;
* writable fields are allowlisted;
* user ownership never comes from request body;
* object-level authorization is tested;
* property-level authorization is tested;
* collection endpoints are bounded;
* cursor pagination exists where required;
* retry behavior is bounded;
* high-risk repeatable POSTs support idempotency;
* dangerous stale writes have concurrency protection;
* provider calls have explicit timeouts;
* third-party responses are validated;
* sensitive responses cannot be shared-cached;
* custom endpoints are inventoried/documented;
* critical endpoint contracts are tested;
* all P0 tests pass.
---
# 186. Final invariant
Ekho request lifecycle:
```text
REQUEST
↓
Authenticate
↓
Authorize
↓
Validate
↓
Execute one clear domain operation
↓
Return standard HTTP result
↓
Observe
```
Failure:
```text
ERROR
↓
Correct HTTP status
↓
RFC 9457 Problem Details
↓
Stable Ekho code
↓
Request ID
↓
Safe user message
↓
Detailed internal telemetry
```
Never:
```text
random endpoint
↓
random JSON
↓
raw database error
↓
frontend guesses what happened
```
The API layer must make Ekho's system **more predictable**, not add another layer of abstraction simply because professional products have APIs.
---
# 187. Primary authority sources
This specification was checked against:
1. **RFC 9110 — HTTP Semantics**, current core HTTP method/status/conditional-request semantics.
2. **RFC 9457 — Problem Details for HTTP APIs**, current standard replacing RFC 7807.
3. **RFC 9111 — HTTP Caching**.
4. **RFC 9205 — Building Protocols with HTTP**, including correct caching guidance.
5. **RFC 6585 — 429 Too Many Requests**.
6. **W3C Trace Context** — standardized `traceparent` / `tracestate`.
7. **OpenAPI Specification 3.2.0**, current published OpenAPI specification.
8. **Next.js current 2026 Route Handler / Backend-for-Frontend documentation**.
9. **Supabase current PostgREST/error handling documentation**.
10. **OWASP API Security Top 10 2023** — object authorization, property authorization, resource consumption, function authorization and unsafe third-party API consumption.
11. **IETF HTTPAPI Idempotency-Key draft**, current work-in-progress standardization used here deliberately as a convention, not misrepresented as a final RFC.
