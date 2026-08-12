# Ekho Performance Budget v1.0

**Status:** READY TO LOCK  
**Date:** 2026-08-12  
**Scope:** Web application performance, Core Web Vitals, JS/CSS, images, fonts, network requests, APIs, caching, prefetching, CI and runtime monitoring.

---

# 1. Purpose

Performance is a product requirement for Ekho.

The product should feel:

```text
instant
stable
light
predictable
```

especially during the core loop:

```text
Open Ekho
→ Search
→ Add university
→ View requirements
→ Complete Next Action
```

Performance must not gradually degrade as features are added.

Therefore Ekho uses explicit budgets rather than:

> "Optimize later."

---

# 2. External performance baseline

Current Core Web Vitals define "good" performance as:

```text
LCP ≤ 2.5 s
INP ≤ 200 ms
CLS ≤ 0.1
```

measured at the **75th percentile**, separately across mobile and desktop.

Supporting metrics:

```text
FCP ≤ 1.8 s
TTFB ≤ ~0.8 s
TBT < 200 ms in lab testing
```

are current web.dev guidance.

### Ekho decision

Do not design Ekho to barely pass those boundaries.

Use stricter internal targets.

---

# 3. Ekho field performance targets

## LCP

Target:

```text
≤ 2.0 s p75
```

Absolute acceptable boundary:

```text
≤ 2.5 s p75
```

Anything above:

```text
2.5 s
```

is a performance incident for a core route.

---

# 4. INP

Target:

```text
≤ 150 ms p75
```

Absolute acceptable boundary:

```text
≤ 200 ms p75
```

Above:

```text
200 ms
```

requires investigation.

---

# 5. CLS

Target:

```text
≤ 0.05 p75
```

Absolute acceptable boundary:

```text
≤ 0.1 p75
```

Ekho should visually feel extremely stable.

---

# 6. FCP

Target:

```text
≤ 1.5 s p75
```

Maximum normal boundary:

```text
≤ 1.8 s p75
```

web.dev currently classifies FCP ≤1.8 seconds as good.

---

# 7. TTFB

Target:

```text
≤ 500 ms p75
```

Maximum normal boundary:

```text
≤ 800 ms p75
```

web.dev currently gives approximately 0.8 seconds as a useful TTFB target for most sites.

---

# 8. TBT — laboratory metric

Target:

```text
≤ 150 ms
```

Hard CI boundary:

```text
≤ 200 ms
```

TBT is primarily a lab metric and web.dev recommends less than 200 ms on average mobile hardware.

Do not treat TBT as a replacement for real-user INP.

---

# 9. Performance hierarchy

When trade-offs exist, prioritize:

```text
1. Correct content
2. Core interaction responsiveness
3. LCP
4. Layout stability
5. Secondary visual polish
6. Non-essential animation
```

Never sacrifice admissions correctness to produce an artificial performance score.

---

# 10. Core routes

Performance budgets apply particularly to:

```text
/app

/universities

/universities/[institutionSlug]

/universities/[institutionSlug]/programs/[programSlug]

/app/applications

/app/applications/[applicationId]

/app/updates
```

---

# 11. Critical routes

The strictest budgets apply to:

```text
/app
/universities
/app/applications/[applicationId]
```

because these directly support the Ekho core loop.

---

# 12. JavaScript philosophy

Default:

```text
do not ship JS
unless the browser actually needs JS
for the current route
```

Do not turn static/read-only content into client JavaScript unnecessarily.

---

# 13. Initial JavaScript budget

For a core route:

```text
Initial first-party JS transferred
≤ 170 KB compressed
```

Target:

```text
≤ 150 KB compressed
```

Absolute exception ceiling:

```text
200 KB compressed
```

Anything above 170 KB requires explicit performance review.

web.dev has historically used approximately **170 KB compressed JavaScript** as a recommended bundle budget example, and separately recommends roughly **100 KB per individual script** as a useful target to reduce costly script evaluation.

---

# 14. What counts as Initial JS

Include JavaScript required before the route becomes meaningfully usable:

```text
framework runtime
shared application runtime
current route code
required component code
required state/data client code
```

Do not exclude framework JS merely because Ekho did not write it.

---

# 15. Lazy route JS

Code loaded only after:

```text
opening a modal
opening advanced filters
opening source details
opening secondary settings
```

does not count toward initial route JS.

But it still requires its own bundle budget.

---

# 16. Single JS chunk

Target:

```text
≤ 100 KB
```

for any individual initial script resource.

web.dev explicitly recommends about 100 KB as a useful individual-script target because large scripts create larger evaluation tasks.

---

# 17. Feature chunk budget

A newly lazy-loaded feature should normally remain:

```text
≤ 50 KB compressed
```

Examples:

```text
advanced filters
document preview
complex editor
chart
```

Anything above:

```text
75 KB
```

requires review.

---

# 18. Heavy dependencies

Before introducing a dependency that adds:

```text
> 20 KB compressed
```

to a core route, Codex must check:

1. can native browser APIs solve it?
    
2. do we already have equivalent functionality?
    
3. can the dependency be tree-shaken?
    
4. can it be dynamically imported?
    
5. is the feature core enough to justify the cost?
    

---

# 19. Forbidden dependency pattern

Do not import an entire library when only one small function is used.

Bad:

```text
import entire utility library
```

for:

```text
one date formatter
```

or:

```text
one debounce function
```

---

# 20. Tree shaking

Production builds must remove:

```text
unused components
unused icons
unused utilities
development code
debug libraries
```

Bundle analysis must be available.

---

# 21. Route splitting

Each major route should load its own non-shared code.

Do not create:

```text
one giant app.js
```

containing the whole Ekho product.

---

# 22. Dynamic import

Use dynamic loading for heavy functionality that is not necessary immediately.

Examples:

```text
document preview
rich editor
large chart library
PDF tooling
special analytics visualization
```

---

# 23. JavaScript execution

Bundle bytes are not enough.

Avoid long main-thread tasks.

A task exceeding approximately:

```text
50 ms
```

is considered a Long Task in Chrome performance terminology.

Large synchronous transforms must therefore be avoided on the main thread.

---

# 24. Large client computations

Do not run large synchronous operations during first render such as:

```text
ranking thousands of universities
filtering the whole dataset
parsing huge admissions JSON
processing documents
PDF extraction
large date transformations
```

These belong:

```text
server-side
search service
worker
preprocessing pipeline
```

depending on the task.

---

# 25. Search dataset

Never download the complete Ekho university/program database to the browser merely to provide search.

Search must use:

```text
client query
→ search API/service
→ paginated results
```

---

# 26. Search filtering

Likewise, do not send tens of thousands of programs to React and run:

```text
array.filter()
```

for global discovery.

Filtering belongs to Search System/backend.

---

# 27. CSS budget

Core route initial compressed CSS:

Target:

```text
≤ 30 KB
```

Hard budget:

```text
≤ 50 KB
```

---

# 28. CSS rule

Do not ship CSS for:

```text
unused routes
unused themes
unused component libraries
unused demo components
```

---

# 29. Component-library caution

Do not install a huge UI framework simply to obtain:

```text
Button
Input
Modal
Dropdown
```

Ekho Design System should stay lightweight.

---

# 30. Critical CSS

Above-the-fold layout must not depend on a long chain of secondary CSS requests.

Critical route styling should be discoverable early.

---

# 31. Render-blocking resources

Minimize render-blocking resources.

The browser should be able to discover:

```text
HTML
critical CSS
critical font where needed
LCP media
```

as early as possible.

---

# 32. Total initial transfer budget

Core authenticated route:

Target:

```text
≤ 600 KB transferred
```

Hard budget:

```text
≤ 800 KB transferred
```

excluding data loaded only after explicit user interaction.

---

# 33. Public Intelligence page transfer

University/program pages may legitimately contain more visual content.

Target:

```text
≤ 800 KB
```

Hard budget:

```text
≤ 1.2 MB
```

for initial page load.

---

# 34. Absolute network discipline

Lighthouse guidance has historically recommended keeping total network payload well below roughly 1.6 MB for performant initial loads on constrained connections. Ekho intentionally sets stricter internal limits.

---

# 35. Images — general rule

Every raster image must be:

```text
correctly sized
compressed
responsive
modern format where appropriate
```

web.dev recommends responsive images, correct dimensions and optimized formats because oversized images waste bandwidth and can worsen LCP.

---

# 36. Image formats

Preferred:

```text
AVIF
WebP
```

Fallback only when necessary:

```text
JPEG
PNG
```

Use SVG for appropriate:

```text
icons
simple logos
vector artwork
```

after safe sanitization.

---

# 37. No giant source image

Never serve:

```text
4000 × 3000
```

when rendered at:

```text
400 × 300
```

Use responsive variants.

---

# 38. LCP image budget

If a route has an image as its LCP element:

Mobile target:

```text
≤ 150 KB
```

Desktop target:

```text
≤ 250 KB
```

Hard exception:

```text
≤ 350 KB
```

Anything larger needs explicit justification.

---

# 39. Above-the-fold image budget

Combined images required for first viewport:

Target:

```text
≤ 300 KB
```

---

# 40. Individual non-hero image

Normal content image:

Target:

```text
≤ 150 KB
```

Do not routinely deliver:

```text
500 KB+
```

images inside ordinary university pages.

---

# 41. University logo

Target:

```text
≤ 20 KB
```

Normal maximum:

```text
≤ 40 KB
```

Use optimized SVG/WebP/PNG according to the source asset.

---

# 42. Search result media

Search result cards should generally not download large university photography.

Search must remain scan-focused.

Prefer:

```text
logo
small identity asset
or no image
```

---

# 43. Image dimensions

Every rendered image should provide intrinsic dimensions through:

```text
width
height
```

or equivalent aspect-ratio reservation.

This prevents layout movement.

---

# 44. LCP image loading

Never:

```text
loading="lazy"
```

on the LCP image.

web.dev explicitly warns that lazy-loading an LCP image delays the resource and worsens LCP.

---

# 45. LCP image priority

If an image is the LCP resource:

use appropriate:

```text
fetchpriority="high"
```

when beneficial.

web.dev recommends this for important LCP images.

---

# 46. LCP image discoverability

LCP media should preferably be discoverable directly from initial HTML.

Avoid requiring:

```text
JS executes
→ component mounts
→ browser finally discovers image
```

web.dev recommends minimizing LCP resource discovery delay.

---

# 47. Below-the-fold images

Use browser-native:

```text
loading="lazy"
```

for suitable off-screen images.

Modern browsers support native image lazy loading without requiring a JavaScript library.

---

# 48. Lazy-loading library

Do not add a JavaScript lazy-loading package when native browser behavior is sufficient.

---

# 49. Responsive images

Use:

```text
srcset
sizes
picture
```

or framework equivalent.

Browser should receive approximately the resolution needed for the actual viewport.

---

# 50. Image CDN

If infrastructure supports it, use image transformation/CDN capabilities for:

```text
resize
format conversion
compression
device variants
```

Do not store dozens of manually maintained duplicate images when infrastructure can safely derive them.

---

# 51. Fonts philosophy

Fonts must not make Ekho visually blank while downloading.

web.dev recommends controlling web-font rendering and preloading only genuinely important fonts to avoid delayed/invisible text and layout instability.

---

# 52. Font format

Prefer:

```text
WOFF2
```

for web fonts.

Do not ship:

```text
TTF
OTF
```

to browsers unless explicitly required.

---

# 53. Font families

Target:

```text
1 primary font family
```

Avoid loading multiple decorative families in the core application.

---

# 54. Font weights

Do not download separate files for:

```text
300
400
500
600
700
800
```

unless truly required.

Prefer:

```text
variable font
```

or a minimal subset of weights.

---

# 55. Initial font budget

Total font transfer required for initial viewport:

Target:

```text
≤ 80 KB
```

Hard budget:

```text
≤ 120 KB
```

---

# 56. Font preload

Preload at most the font resources genuinely needed for first render.

Typical:

```text
1 primary font resource
```

possibly:

```text
2
```

if there is a justified separate style.

Do not preload every font weight.

---

# 57. font-display

Use an appropriate:

```text
font-display
```

strategy such as `swap`, `fallback`, or `optional` based on design needs.

Do not intentionally produce long invisible-text periods.

---

# 58. Font fallback

Fallback font metrics should be reasonably compatible with the primary font to reduce:

```text
layout shift
text reflow
```

---

# 59. Icons

Prefer:

```text
SVG
CSS
small icon component set
```

Do not load a massive icon font.

---

# 60. Icon imports

Import icons individually.

Bad:

```text
load entire icon catalogue
```

for six icons.

---

# 61. Initial request budget

Core route initial render:

Target:

```text
≤ 25 network requests
```

Hard budget:

```text
≤ 35 requests
```

before the route is meaningfully usable.

---

# 62. Public visual page request budget

For richer university/program pages:

Target:

```text
≤ 35
```

Hard budget:

```text
≤ 50
```

initial requests.

---

# 63. Requests before LCP

Critical resource chain target:

```text
≤ 10 requests
```

before LCP.

Fewer is better.

---

# 64. API requests for initial route

A normal authenticated route should ideally require:

```text
≤ 3 application API requests
```

to render its primary content.

Target architecture:

```text
1 route data request
+
optional independent secondary request
```

not:

```text
15 component requests
```

---

# 65. N+1 frontend requests

Forbidden:

```text
GET applications
↓
for every application:
GET university
GET requirements
GET deadline
```

Use suitable composed/query endpoints.

---

# 66. Example bad My Applications load

Bad:

```text
1 applications request
+
10 university requests
+
10 requirement requests
+
10 deadline requests
=
31 API requests
```

---

# 67. Correct My Applications load

Prefer:

```text
GET /applications/summary
```

returning only the data needed for that screen.

---

# 68. API overfetching

Do not solve N+1 by returning:

```text
every field in the university database
```

Route payloads should remain purpose-specific.

---

# 69. JSON payload budget

Normal route API response:

Target:

```text
≤ 100 KB compressed
```

Prefer:

```text
≤ 50 KB
```

for common screens.

---

# 70. Large payload

Any single ordinary JSON payload:

```text
> 250 KB compressed
```

requires review.

---

# 71. Search autocomplete payload

Autocomplete response target:

```text
≤ 20 KB compressed
```

Typical response should be significantly smaller.

---

# 72. Search results payload

One Explore page:

Target:

```text
≤ 75 KB compressed
```

Do not include complete University Intelligence for each search card.

---

# 73. Pagination

Never return:

```text
thousands of programs
```

in one API response.

Use:

```text
cursor/page
```

based pagination.

---

# 74. Autocomplete requests

Search autocomplete already uses:

```text
~150 ms debounce
```

from Search & Filtering Standard.

Additionally:

```text
1 current query
=
at most 1 active autocomplete request
```

Cancel or ignore stale requests.

---

# 75. Request deduplication

If multiple components request identical data simultaneously:

deduplicate where practical.

Do not produce:

```text
3 identical GET /profile
```

during one route render.

---

# 76. Third-party requests

Default:

```text
0 third-party blocking requests
```

before primary content renders.

---

# 77. Third-party scripts

Any third-party script requires review for:

```text
bytes
CPU
network calls
privacy
necessity
failure behavior
```

---

# 78. Analytics

Analytics must never block:

```text
HTML
critical CSS
core JS
LCP
```

Load asynchronously/defer appropriately.

---

# 79. Third-party initial budget

Before the page becomes meaningfully usable:

Target:

```text
≤ 5 third-party requests
```

and preferably:

```text
≤ 2 third-party origins
```

---

# 80. No tag-manager dumping ground

Do not allow arbitrary marketing scripts to accumulate through a tag manager without performance review.

---

# 81. API latency budgets

Internal server targets, excluding user network latency:

## Search/autocomplete

```text
p95 ≤ 200 ms
```

Target:

```text
≤ 150 ms
```

---

# 82. Normal read API

Target:

```text
p95 ≤ 300 ms
```

---

# 83. Mutation API

For normal operations such as:

```text
add university
update profile answer
complete action
```

Target:

```text
p95 ≤ 500 ms
```

---

# 84. Slow complex operation

Anything expected to exceed:

```text
1 second
```

should be treated as a distinct workflow.

Do not freeze the interface while waiting.

---

# 85. Optimistic UI

Use optimistic updates only when:

```text
failure is reversible
server validation is predictable
incorrect temporary state is not dangerous
```

---

# 86. Admissions-state caution

Do not optimistically show:

```text
Requirement = Satisfied
```

before the canonical evaluator successfully confirms it.

---

# 87. Safe optimistic example

Possible:

```text
favorite/save visual state
```

or another reversible non-critical interaction.

---

# 88. Application creation

`Add to Applications` should feel immediate.

But persisted server state remains authoritative.

On failure:

```text
Couldn't add this application.
Try again.
```

---

# 89. Loading UI

Do not block the entire application with a giant spinner when only one section is loading.

Use:

```text
skeleton
existing cached content
localized loading state
```

as appropriate.

---

# 90. Skeleton requirements

Skeleton geometry should resemble final content dimensions.

Do not cause:

```text
skeleton
→ final content
→ major layout shift
```

---

# 91. Cache philosophy

Caching must improve performance without creating:

```text
wrong admissions information
cross-user data leaks
stale personalized conclusions
```

---

# 92. Three independent freshness concepts

Never confuse:

```text
HTTP cache freshness

Ekho database freshness

Admissions source verification freshness
```

They are different systems.

---

# 93. Example

A requirement may be:

```text
HTTP response generated 10 seconds ago
```

but underlying official source was:

```text
last verified 60 days ago
```

HTTP freshness does not make the admissions information recently verified.

---

# 94. Static hashed assets

For content-hashed:

```text
.js
.css
woff2
static images
```

use:

```text
Cache-Control:
public, max-age=31536000, immutable
```

MDN explicitly documents this as an appropriate pattern for versioned/hash-addressed immutable assets.

---

# 95. Hash requirement

Long immutable caching is only safe when filenames/URLs change when content changes.

Example:

```text
app.a8173cd.js
```

not permanently:

```text
app.js
```

with a one-year immutable cache.

---

# 96. HTML — public routes

For public HTML whose content may change:

Do not use one-year immutable caching.

Recommended baseline:

```text
Cache-Control:
public, max-age=0, s-maxage=300, stale-while-revalidate=3600
```

when supported by the chosen CDN/runtime and when compatible with data freshness.

Exact CDN syntax can vary by provider.

---

# 97. Public University Intelligence API

Recommended starting policy:

Browser freshness:

```text
60 seconds
```

Shared CDN freshness:

```text
5 minutes
```

Background stale window:

```text
up to 1 hour
```

Conceptually:

```text
public
max-age=60
s-maxage=300
stale-while-revalidate=3600
```

This is an **Ekho policy**, not an admissions-data freshness policy.

---

# 98. Public search results

Search index data may change.

Do not use long browser caching.

Possible starting point:

```text
max-age=0
```

with short CDN/search-provider caching where effective.

---

# 99. Personalized HTML

Authenticated personalized responses must never be shared through public caches.

Use:

```text
Cache-Control:
private, no-cache
```

with validators where appropriate.

MDN specifically recommends `private` for personalized responses and explains that `no-cache` allows storage but requires revalidation before ordinary reuse.

---

# 100. Personalized API

Examples:

```text
/applications
/profile
/requirements-for-me
/next-action
```

Default:

```text
private, no-cache
```

unless a more specific safe strategy exists.

---

# 101. Why not `no-store` everywhere

Do not blindly set:

```text
no-store
```

on the entire authenticated application.

MDN warns that liberal `no-store` use discards browser/platform caching advantages, including potential back/forward-cache benefits, and recommends `private` + `no-cache` for many personalized cases.

---

# 102. Truly sensitive responses

Use:

```text
Cache-Control:
private, no-store
```

where responses genuinely should not be stored.

Examples may include:

```text
authentication tokens
temporary secrets
highly sensitive document contents
signed credential responses
```

---

# 103. User-uploaded documents

Private applicant documents must never be exposed through:

```text
public CDN caching
public URLs
shared cache
```

Use:

```text
authenticated access
or
short-lived signed URLs
```

according to Security/Data standards.

---

# 104. ETag

Use validators such as:

```text
ETag
```

for resources that can benefit from conditional requests.

When unchanged, revalidation can return:

```text
304 Not Modified
```

instead of retransmitting the resource.

---

# 105. Last-Modified

Where meaningful, support:

```text
Last-Modified
```

alongside ETag.

HTTP guidance recommends both where possible.

---

# 106. Browser data cache

Client data-fetching cache may use:

```text
stale-while-revalidate
```

patterns for public/reference information.

But the client cache is never canonical.

---

# 107. Personalized client cache

Personalized data may be retained:

```text
in memory
```

for responsive navigation.

But mutation must invalidate affected data.

---

# 108. Requirement cache invalidation

If user changes:

```text
qualification
test score
application round
program
```

invalidate affected:

```text
requirements
progress
next actions
```

immediately.

---

# 109. No stale localStorage truth

Never store canonical:

```text
Requirement = Satisfied
```

in `localStorage` and trust it indefinitely.

Server/domain evaluator remains authoritative.

---

# 110. Offline data caution

Do not implement broad offline admissions-data persistence without a separate specification.

Incorrect stale admissions information is worse than a clear:

```text
You're offline
```

state.

---

# 111. Prefetch philosophy

Prefetch only resources that have a strong probability of being used soon.

Do not prefetch the entire product.

---

# 112. Route prefetch

Allowed examples:

```text
hover/focus result
→ prefetch University Intelligence route

visible application's primary action
→ prefetch likely route
```

when bandwidth cost is small.

---

# 113. Search-result prefetch

Do **not** prefetch:

```text
50 university pages
```

simply because 50 results appear.

---

# 114. Mobile/data saver

Aggressive prefetching should respect:

```text
network conditions
Save-Data where supported
device constraints
```

where practical.

---

# 115. Preload philosophy

`preload` is not:

```text
make everything faster
```

It consumes early network priority.

Use only for truly critical resources.

web.dev explicitly warns that preload should be used intentionally because initial bandwidth is limited.

---

# 116. Valid preload candidates

Typically:

```text
critical font
late-discovered LCP image
critical resource otherwise discovered too late
```

---

# 117. Forbidden preload pattern

Do not preload:

```text
all fonts
all route chunks
all hero images
analytics
future modal code
```

---

# 118. Preconnect

Use preconnect only for origins that will definitely be needed early.

Target:

```text
≤ 2 preconnected external origins
```

---

# 119. DNS/preconnect spam

Do not add:

```text
10 preconnect links
```

for speculative third parties.

---

# 120. Navigation

Client-side navigation should not require full page reloads for normal internal app transitions where the selected architecture supports efficient route navigation.

But do not preserve enormous inactive page trees merely to avoid reloads.

---

# 121. Back/forward cache

Do not intentionally break browser bfcache without reason.

This is another reason not to apply `no-store` indiscriminately.

---

# 122. University Explore performance

`/universities` must not render thousands of DOM nodes.

Initial:

```text
20–30 results
```

as defined in Search System.

Additional results load progressively.

---

# 123. DOM size

Do not render hidden copies of:

```text
every filter
every result
every modal
every university
```

inside the DOM.

Render what is currently needed.

---

# 124. Virtualization

Do not add list virtualization automatically.

For:

```text
20–30 search cards
```

normal rendering should be adequate.

Use virtualization only when measurements prove it necessary.

---

# 125. Search interaction performance

From keystroke to visible autocomplete update:

Target:

```text
≤ 300 ms p95 user-perceived
```

under normal network conditions.

Components:

```text
150 ms debounce
+
fast backend
+
small response
+
fast render
```

---

# 126. Filter interaction

Local UI response:

```text
< 100 ms
```

Results can update asynchronously.

User must immediately see that the filter selection registered.

---

# 127. Requirements screen

Do not make one request per requirement.

Use a composed evaluation payload.

---

# 128. Requirement evaluation

If evaluation can be done server-side during route data generation, do not send raw rule databases to the browser merely to recalculate them client-side.

---

# 129. Personalization questions

Submitting one profiling answer should update affected results without forcing a full browser page reload.

---

# 130. Images on Application pages

Application management is task-oriented.

Do not use:

```text
large campus hero image
full-width photography
video background
```

inside `/app/applications/[id]`.

This wastes bytes without helping the job.

---

# 131. Home page imagery

Authenticated `/app` Home should remain extremely light.

Prefer:

```text
UI
text
icons
```

over decorative media.

---

# 132. Animations

Animations must not delay interaction.

Preferred:

```text
transform
opacity
```

for normal motion.

Avoid expensive layout-triggering animation where possible.

---

# 133. Animation duration

Performance and UX are different standards, but animations should never be required before an action can execute.

The app should function correctly when reduced motion is enabled.

---

# 134. Glass / blur

Ekho's visual style may use sophisticated effects.

But large:

```text
backdrop-filter
blur
layer stacks
```

must be measured on mid-range mobile hardware.

Do not sacrifice scrolling responsiveness for visual effects.

---

# 135. Shadows

Avoid massive multiple-layer shadows on hundreds of repeated search cards.

Measure repeated component paint cost.

---

# 136. Mobile is primary performance constraint

Do not approve a feature merely because:

```text
MacBook Pro + fiber
```

feels fast.

Core performance testing must include mobile-like CPU/network constraints.

---

# 137. Laboratory environment

CI should use a **fixed, version-controlled performance test configuration**.

Do not allow each developer's laptop/network to define pass/fail.

---

# 138. Lighthouse CI routes

At minimum test:

```text
/app or representative authenticated fixture

/universities

/universities/[fixtureInstitution]

/app/applications/[fixtureApplication]
```

---

# 139. Lighthouse score

Target:

```text
Performance ≥ 90
```

for tested core routes.

However:

> Lighthouse score is diagnostic, not the primary product metric.

Core Web Vitals + absolute budgets remain more important because Lighthouse scoring methodology can evolve.

---

# 140. CI metric requirements

Core test routes:

```text
LCP ≤ 2.5 s
TBT ≤ 200 ms
CLS ≤ 0.1
```

under the pinned CI environment.

Internal target remains stricter.

---

# 141. Bundle CI

Every pull request must compare:

```text
initial JS
route JS
CSS
```

against:

```text
absolute budget
+
main branch baseline
```

---

# 142. Bundle regression warning

Warn on a core route if compressed JS increases by:

```text
> 5 KB
```

or:

```text
> 5%
```

whichever is more meaningful.

---

# 143. Bundle hard failure

Fail CI when:

```text
absolute JS budget exceeded
```

unless an explicit temporary exception exists.

---

# 144. Temporary exception

Performance exception must include:

```text
route
metric/budget exceeded
reason
owner
planned removal/fix
```

Do not permanently increase the global budget to make CI green.

---

# 145. Dependency PR review

If a PR adds a significant dependency:

bundle diff should reveal:

```text
dependency
compressed size impact
routes affected
```

---

# 146. Image CI

CI/build should reject obviously oversized static images.

Recommended raw asset warning:

```text
> 500 KB
```

for normal website imagery.

Exceptions require optimization or justification.

---

# 147. Source image vs delivered image

A large original may exist in media storage.

What matters for page budget is:

```text
delivered optimized variant
```

Do not serve the raw original automatically.

---

# 148. Missing responsive image sizes

Lint/build should warn when a responsive content image lacks required sizing metadata.

---

# 149. RUM — Real User Monitoring

Production must collect:

```text
LCP
INP
CLS
```

from real users.

Lab testing alone is insufficient.

Core Web Vitals themselves are intended to be evaluated at the 75th percentile of real page loads.

---

# 150. RUM dimensions

At minimum aggregate by:

```text
route group
mobile / desktop
country/region where privacy-safe
app version/release
```

---

# 151. Avoid high-cardinality telemetry

Do not send:

```text
full URL containing user IDs
application IDs
private queries
document names
```

as unrestricted metrics labels.

Normalize dynamic routes.

Example:

```text
/app/applications/:id
```

---

# 152. Performance analytics privacy

Performance telemetry must not include:

```text
test scores
financial data
essay text
document contents
personal notes
```

---

# 153. Field alert — LCP

Alert when a core route's p75 LCP exceeds:

```text
2.5 s
```

for a statistically meaningful sample.

---

# 154. Field alert — INP

Alert when:

```text
p75 INP > 200 ms
```

---

# 155. Field alert — CLS

Alert when:

```text
p75 CLS > 0.1
```

---

# 156. Regression investigation

Performance incident should inspect:

```text
release change
bundle diff
third-party scripts
API latency
cache hit ratio
LCP resource
image sizes
long tasks
layout shifts
```

---

# 157. Cache monitoring

Track where practical:

```text
CDN hit ratio
origin requests
304 rate
cacheable response coverage
```

Do not optimize blindly.

---

# 158. API monitoring

Track:

```text
p50
p95
p99
error rate
```

for important endpoints.

Average alone can hide bad tail latency.

---

# 159. Search monitoring

Track separately:

```text
autocomplete latency
full search latency
zero results
search errors
```

---

# 160. Performance under failure

Slow backend must not produce a frozen blank page.

If public cached data exists and policy permits:

```text
show stale-safe cached data
+
refresh
```

---

# 161. Admissions-critical stale data

Do not use stale HTTP fallback to override Data Standard.

Example:

If Data Standard marks a deadline:

```text
stale / unsafe
```

a CDN cache must not cause Ekho to present it as verified/current.

---

# 162. HTTP SWR vs admissions stale

Critical distinction:

```text
HTTP stale-while-revalidate
```

may serve a response a few minutes old.

That is not the same as:

```text
admissions fact not verified for current cycle
```

The second must still be shown according to Data Standard.

---

# 163. Compression

Text resources must use modern HTTP compression:

```text
Brotli where supported
gzip fallback
```

for:

```text
HTML
JS
CSS
JSON
SVG
```

where appropriate.

---

# 164. Already compressed media

Do not waste CPU compressing already compressed formats such as:

```text
AVIF
WebP
JPEG
WOFF2
```

through generic HTTP compression where it provides negligible value.

---

# 165. Static compression

Where infrastructure supports it, precompressed static assets are preferable to recompressing unchanged bundles on every origin request.

---

# 166. CDN

Public static assets should be delivered through edge/CDN infrastructure appropriate to the deployment architecture.

Do not route immutable JS/image traffic through expensive dynamic application computation.

---

# 167. Redirects

Avoid unnecessary redirect chains.

Bad:

```text
/uni
→ /universities
→ /universities/
→ locale route
```

before content loads.

---

# 168. Authentication redirect

Authenticated entry should reach `/app` through the minimum necessary redirect chain.

---

# 169. API geography

As Ekho becomes global, monitor latency by region.

Do not assume good performance from one European test location means good performance in:

```text
US
India
East Asia
Latin America
```

---

# 170. Database queries

Performance budget applies to database access too.

No endpoint should routinely execute uncontrolled:

```text
N+1 database queries
```

---

# 171. Query limits

Every list endpoint requires:

```text
bounded pagination
```

and database indexes appropriate to its actual access pattern.

---

# 172. Search database separation

Do not force transactional database queries to emulate full-text global search if the Search Architecture later uses a dedicated search engine.

Follow the Search System abstraction.

---

# 173. Requirement recomputation

Cross-application recomputation after one profile update must not block the browser with sequential network calls.

Prefer:

```text
one mutation
→ server determines affected applications
→ server recomputes/batches
→ client receives coherent result
```

---

# 174. Bulk operations

Use batching where multiple closely related reads/writes belong to one user action.

Do not create dozens of tiny network round-trips.

---

# 175. But do not over-batch

Do not create one enormous:

```text
GET /everything
```

payload containing the whole workspace.

Batch by route/use case.

---

# 176. Server response streaming

If chosen framework/runtime supports safe streaming:

it may be used for independent secondary sections.

But:

```text
streaming
```

must not become an excuse for a slow primary route.

Primary useful content should still arrive quickly.

---

# 177. Hydration

Avoid unnecessarily hydrating large read-only trees.

Interactive islands/components should be limited to where interaction is required if architecture permits.

---

# 178. Client-only rendering

Do not make public University Intelligence depend entirely on:

```text
blank HTML
→ download JS
→ fetch API
→ render content
```

when server rendering/static rendering can expose useful content earlier.

---

# 179. Search client rendering

Autocomplete is naturally interactive.

But initial Explore shell/content should not require excessive JS before the user sees anything useful.

---

# 180. Error tracking scripts

Error monitoring is important but its browser SDK must respect the same third-party/JS budget.

Do not enable every session replay feature by default without measuring cost and privacy impact.

---

# 181. Session replay

If ever used:

separate decision required.

It can add:

```text
JS
CPU
network
privacy risk
```

Do not turn it on automatically.

---

# 182. Video

No autoplay video in core Ekho screens.

---

# 183. Background video

Forbidden for:

```text
Home
Explore
Applications
Requirements
```

---

# 184. University video

If later included on Intelligence:

```text
poster image first
video loaded only on user intent
```

Do not download video on initial page load.

---

# 185. PDF previews

Do not ship a PDF renderer on every application route.

Load it only after:

```text
user opens PDF/document preview
```

---

# 186. Charts

Do not ship charting libraries on pages without charts.

---

# 187. Rich editors

Essay/rich text editor code must not load on:

```text
Home
Search
Applications list
```

unless the editor is actually opened.

---

# 188. Maps

Do not add heavy map SDKs to basic university pages merely to show a location.

Use a lightweight static representation or lazy-load a map only on user intent if maps are later required.

---

# 189. AI features

AI code/network calls must not be part of normal first render unless AI is required for that exact surface.

Ekho's core workflow must work without waiting for an AI request.

---

# 190. No AI loading gate

Forbidden:

```text
Open Application
→ wait for LLM
→ requirements appear
```

Requirements are driven by structured deterministic data.

---

# 191. Service Worker

Do not introduce a complex Service Worker merely because PWAs can use one.

Use only with a defined need:

```text
offline
advanced caching
push
```

and a separate invalidation strategy.

---

# 192. Service Worker danger

A bad service-worker cache can keep outdated:

```text
JS
HTML
admissions data
```

alive after deployment.

Therefore it is not part of default v1 architecture unless deliberately specified.

---

# 193. Performance budget summary

## Field

```text
LCP target       ≤ 2.0 s
LCP maximum      ≤ 2.5 s

INP target       ≤ 150 ms
INP maximum      ≤ 200 ms

CLS target       ≤ 0.05
CLS maximum      ≤ 0.1

FCP target       ≤ 1.5 s
FCP maximum      ≤ 1.8 s

TTFB target      ≤ 500 ms
TTFB maximum     ≤ 800 ms
```

---

# 194. Laboratory summary

```text
TBT target       ≤ 150 ms
TBT hard         ≤ 200 ms

Lighthouse
core routes      ≥ 90 target
```

---

# 195. JavaScript summary

```text
Initial first-party JS target:
≤ 150 KB compressed

Initial first-party JS budget:
≤ 170 KB compressed

Absolute exception ceiling:
≤ 200 KB compressed

Individual initial script target:
≤ 100 KB

Lazy feature chunk target:
≤ 50 KB
```

---

# 196. CSS summary

```text
Initial CSS target:
≤ 30 KB compressed

Hard budget:
≤ 50 KB compressed
```

---

# 197. Font summary

```text
Initial fonts target:
≤ 80 KB

Hard:
≤ 120 KB

Primary families:
1 preferred
```

---

# 198. Image summary

```text
Mobile LCP image target:
≤ 150 KB

Desktop LCP image target:
≤ 250 KB

Normal image target:
≤ 150 KB

University logo target:
≤ 20 KB

Above-fold combined images:
≤ 300 KB
```

---

# 199. Network summary

Core authenticated route:

```text
Total initial transfer target:
≤ 600 KB

Hard:
≤ 800 KB
```

Public Intelligence:

```text
Target:
≤ 800 KB

Hard:
≤ 1.2 MB
```

---

# 200. Request summary

```text
Core route target:
≤ 25 initial requests

Hard:
≤ 35

Public richer route:
≤ 50 hard

Before LCP:
≤ 10 critical requests

Application API calls required for initial route:
≤ 3 target
```

---

# 201. API summary

```text
Autocomplete/search backend:
p95 ≤ 200 ms

Normal read:
p95 ≤ 300 ms

Normal mutation:
p95 ≤ 500 ms
```

---

# 202. Caching summary

## Hashed static assets

```text
public
max-age=31536000
immutable
```

## Public HTML

Short shared-cache TTL + revalidation.

## Public university/program data

Short browser cache + moderate CDN cache.

## Personalized data

```text
private, no-cache
```

## Truly sensitive responses

```text
private, no-store
```

## Private documents

Never public-cache.

---

# 203. Critical performance invariants

```text
INV-PERF-01
Core Web Vitals must be measured in the field, not only Lighthouse.

INV-PERF-02
Core route initial JS must stay within the JS budget.

INV-PERF-03
Large features not needed immediately must be lazy-loaded.

INV-PERF-04
The complete university/program dataset is never downloaded to the client.

INV-PERF-05
Search filtering is never performed over the complete global dataset in the browser.

INV-PERF-06
LCP images are never lazy-loaded.

INV-PERF-07
Below-fold media is lazy-loaded where appropriate.

INV-PERF-08
Responsive images are served at appropriate sizes.

INV-PERF-09
Large raw images are not sent directly to ordinary page layouts.

INV-PERF-10
Fonts remain within the font budget.

INV-PERF-11
Third-party JavaScript never blocks core rendering.

INV-PERF-12
N+1 API requests are forbidden.

INV-PERF-13
N+1 database access is forbidden.

INV-PERF-14
Public immutable assets use content hashing + long caching.

INV-PERF-15
Personalized responses are never shared through public cache.

INV-PERF-16
Private documents are never publicly cached.

INV-PERF-17
HTTP cache freshness never replaces admissions-data freshness.

INV-PERF-18
Profile/application mutations invalidate dependent personalized results.

INV-PERF-19
Canonical requirement states are never trusted from stale localStorage.

INV-PERF-20
Prefetching cannot download dozens of unrequested routes.

INV-PERF-21
Preload is reserved for genuinely critical resources.

INV-PERF-22
Performance regressions are checked in CI.

INV-PERF-23
Performance budgets cannot simply be increased to make CI pass.

INV-PERF-24
Core product routes must remain usable without waiting for AI.

INV-PERF-25
Visual effects cannot make interaction noticeably slower.
```

---

# 204. Things Codex must NOT invent

When implementing Ekho, Codex must not independently add:

```text
huge client-side university dataset

client-side global search over all programs

giant app-wide JS bundle

whole icon libraries

whole utility libraries for one function

heavy UI frameworks without need

multiple font families

all font weights

large campus photography in Applications

autoplay video

background video

eager-loading below-fold imagery

lazy-loading LCP imagery

prefetching every search result

preloading every route

preloading every font

large map SDK by default

PDF renderer on every route

charting library on every route

rich editor on every route

AI request as page-render prerequisite

public caching of personalized data

public caching of applicant documents

no-store everywhere

localStorage as canonical admissions state

Service Worker without explicit specification

session replay without review

hidden N+1 APIs

GET /everything mega payload

thousands of results in one response

performance-budget increases merely to pass CI
```

---

# 205. Required performance tests

## PERF-01 — Home

```text
/app
```

Test:

```text
initial JS
transfer
requests
LCP
TBT
CLS
```

---

# 206. PERF-02 — Explore

```text
/universities
```

with first result page.

No global dataset download.

---

# 207. PERF-03 — Autocomplete

Type:

```text
Stanford
```

Expected:

```text
stale requests cancelled/ignored
small payload
results update quickly
```

---

# 208. PERF-04 — University Intelligence

Test normal representative page with:

```text
logo
requirements preview
cost data
sources
```

against public-page budget.

---

# 209. PERF-05 — Application Detail

Test:

```text
requirements
next action
deadline
source metadata
```

No one-request-per-requirement behavior.

---

# 210. PERF-06 — Image LCP

Verify:

```text
LCP image not lazy
correct responsive variant
priority appropriate
dimensions reserved
```

---

# 211. PERF-07 — Below-fold images

Verify off-screen images:

```text
not downloaded unnecessarily during critical load
```

where native lazy loading should apply.

---

# 212. PERF-08 — Font load

Verify:

```text
no invisible content delay
no unnecessary font weights
font budget respected
```

---

# 213. PERF-09 — Cold cache

Run with:

```text
empty browser cache
```

This is the primary asset-budget test.

---

# 214. PERF-10 — Warm cache

Repeat navigation.

Verify immutable assets:

```text
not retransferred unnecessarily
```

---

# 215. PERF-11 — Personalized caching

Authenticated response for User A must never be served to User B through shared cache.

---

# 216. PERF-12 — Static asset caching

Hashed JS/CSS:

```text
Cache-Control:
public, max-age=31536000, immutable
```

---

# 217. PERF-13 — Profile mutation

```text
change test score
↓
requirements recompute
```

No stale personalized cache result remains visible.

---

# 218. PERF-14 — Back navigation

Navigate:

```text
Explore
→ University
→ Back
```

Return quickly with preserved search/filter state where browser/app cache permits.

---

# 219. PERF-15 — Large dependency

Add test dependency.

CI must expose bundle regression.

---

# 220. PERF-16 — Mobile constrained test

Test core routes under pinned mobile CPU/network simulation.

No desktop-only approval.

---

# 221. PERF-17 — Slow API

Artificially delay secondary API.

Primary content should not disappear unnecessarily.

---

# 222. PERF-18 — Failed API

Failure should produce local error state rather than:

```text
blank white application
```

---

# 223. PERF-19 — Third-party unavailable

Analytics/error-monitoring outage must not block core Ekho.

---

# 224. PERF-20 — Search result scaling

Explore with large production-like corpus.

Client rendering/request size must remain bounded because results are paginated/server-filtered.

---

# 225. Performance acceptance criteria

Performance Budget is **not satisfied** until:

-  Field LCP target is monitored.
    
-  Field INP target is monitored.
    
-  Field CLS target is monitored.
    
-  RUM measures p75.
    
-  Mobile and desktop are separable.
    
-  Core routes have Lighthouse CI.
    
-  Core route JS is measured automatically.
    
-  Initial JS stays ≤170 KB compressed or has explicit exception.
    
-  Any core-route JS >200 KB fails review.
    
-  Single large scripts are identified.
    
-  Route-level code splitting works.
    
-  Heavy secondary features use dynamic import.
    
-  Production tree shaking works.
    
-  CSS stays within budget.
    
-  Fonts stay within budget.
    
-  Images are responsive.
    
-  LCP image is never lazy-loaded.
    
-  Below-fold images use lazy loading where appropriate.
    
-  LCP resource is discoverable early.
    
-  Large university images are optimized.
    
-  Logo assets are lightweight.
    
-  Search cards do not pull full-resolution photography.
    
-  Initial transfer is within route budget.
    
-  Request count is within route budget.
    
-  Initial application API request count is controlled.
    
-  No frontend N+1 data loading exists.
    
-  No backend N+1 database pattern exists on core routes.
    
-  Search does not download the entire dataset.
    
-  Search responses are paginated.
    
-  Search payloads are compact.
    
-  Autocomplete stale requests cannot overwrite new ones.
    
-  Third-party scripts are non-blocking.
    
-  Analytics does not block rendering.
    
-  Static hashed assets use long immutable caching.
    
-  Mutable public resources use appropriate revalidation.
    
-  Personalized responses are `private`.
    
-  Sensitive responses use appropriate `no-store`.
    
-  Applicant documents cannot enter public cache.
    
-  ETag/conditional requests are supported where useful.
    
-  HTTP caching does not hide Data Standard freshness.
    
-  Profile mutations invalidate personalized caches.
    
-  Requirement results are never permanently trusted from browser storage.
    
-  Prefetch is selective.
    
-  Preload is limited to critical resources.
    
-  No aggressive all-route prefetch exists.
    
-  Error monitoring remains inside budget.
    
-  AI does not block core route rendering.
    
-  No autoplay/background video exists in core product.
    
-  PDF/editor/chart code is lazy-loaded.
    
-  Bundle diffs appear in PR/CI.
    
-  Significant bundle regression generates warning.
    
-  Absolute budget violations fail CI.
    
-  Performance exceptions are explicit and temporary.
    
-  Production RUM alerts on CWV regression.
    
-  API p95 latency is monitored.
    
-  Search latency is monitored.
    
-  All required PERF tests pass.
    

---

# 226. Codex implementation order

## Phase 1 — Base delivery

Implement:

```text
production compression
static asset hashing
route splitting
basic cache headers
```

before feature growth.

---

# 227. Phase 2 — Core route budgets

Set CI budgets for:

```text
/app
/universities
/app/applications/[id]
```

including:

```text
JS
CSS
transfer
requests
```

---

# 228. Phase 3 — Images/fonts

Implement:

```text
responsive images
image optimization
font subsets/variable font
font-display
dimensions
lazy loading
LCP priority
```

---

# 229. Phase 4 — API efficiency

Audit:

```text
Home
Explore
Applications
Application Detail
```

for:

```text
N+1
overfetching
payload size
latency
```

---

# 230. Phase 5 — Caching

Implement explicit policies for:

```text
hashed static assets
public HTML
public admissions data
search
personalized application data
sensitive/private documents
```

---

# 231. Phase 6 — Lighthouse CI

Pin testing environment.

Run core representative routes on every relevant PR.

---

# 232. Phase 7 — RUM

Production monitoring:

```text
LCP
INP
CLS
```

with route grouping.

---

# 233. Phase 8 — Regression alerts

Connect releases to performance monitoring.

A regression should be attributable to:

```text
deployment
route
metric
```

---

# 234. Final locked budget

```text
CORE WEB VITALS

LCP
target ≤ 2.0 s
max ≤ 2.5 s

INP
target ≤ 150 ms
max ≤ 200 ms

CLS
target ≤ 0.05
max ≤ 0.1


SUPPORTING

FCP
target ≤ 1.5 s
max ≤ 1.8 s

TTFB
target ≤ 500 ms
max ≤ 800 ms

TBT lab
target ≤ 150 ms
max ≤ 200 ms


JAVASCRIPT

initial first-party JS
target ≤ 150 KB compressed
budget ≤ 170 KB
absolute exception ceiling 200 KB

single initial script
target ≤ 100 KB

lazy feature chunk
target ≤ 50 KB


CSS

target ≤ 30 KB compressed
max ≤ 50 KB


FONTS

target ≤ 80 KB initial
max ≤ 120 KB


IMAGES

mobile LCP ≤ 150 KB target
desktop LCP ≤ 250 KB target

normal image ≤ 150 KB target
logo ≤ 20 KB target

above-fold combined images ≤ 300 KB


TOTAL TRANSFER

core authenticated:
target ≤ 600 KB
max ≤ 800 KB

public intelligence:
target ≤ 800 KB
max ≤ 1.2 MB


REQUESTS

core route:
target ≤ 25
max ≤ 35

richer public page:
max ≤ 50

critical before LCP:
≤ 10

initial application API:
≤ 3 target


SERVER

search/autocomplete p95 ≤ 200 ms
read API p95 ≤ 300 ms
normal mutation p95 ≤ 500 ms
```

---

# 235. Final performance rule

The Performance Budget exists so Ekho cannot slowly become:

```text
beautiful
feature-rich
correct
but heavy and sluggish
```

Every new feature spends from a finite budget:

```text
bytes
requests
CPU
network
rendering time
```

If a feature pushes Ekho outside the budget, the default decision is:

```text
optimize it
lazy-load it
simplify it
or remove it
```

—not silently increase the budget.

Ekho should remain fast because **performance is an architectural constraint, not a cleanup task before launch.**