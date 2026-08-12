# Ekho Search & Filtering System v1.0
**Status:** READY TO LOCK
**Date:** 2026-08-12
**Scope:** University/program discovery, search, autocomplete, filtering, relevance ranking and result navigation.
---
# 1. Purpose
Search exists to help a user answer one of two questions:
```text
1. I know what university/program I want.
   → Find it immediately.
2. I know roughly what I want.
   → Narrow the available options quickly.
```
Search must therefore support two distinct jobs:
```text
Known-item search
+
Discovery search
```
Do not merge them into one complicated experience.
---
# 2. Product principle
Ekho Search is **not** a university ranking website.
Ekho Search is:
```text
fast entity finding
+
structured programme discovery
+
reliable filtering
```
Primary rule:
> Search relevance must answer the user's query, not promote whichever university is famous.
Never let:
* university prestige;
* paid placement;
* external university ranking;
* acceptance rate;
* arbitrary Ekho score;
override a substantially better textual/query match.
---
# 3. Evidence from existing products
Common App currently supports college search by name plus filters including location, region, application availability, campus setting, financial aid, institution type and enrollment size.
BigFuture currently organizes discovery filters around categories including Majors, Location, Type, Affordability, Campus Life and Admissions.
UCAS course search is much more course-oriented and currently supports filters including course type/year, applicant location for vacancies, start date, study mode, qualification, institution, subject, duration, country and region.
Bachelorsportal similarly uses global-study filters such as subject, country, tuition, duration, study format and language requirements.
### Ekho conclusion
For a global applicant product, the strongest common search dimensions are:
```text
Institution
Program / Subject
Country
Degree level
Language
Study mode
Intake
Cost
Financial aid
```
But Ekho should expose only filters backed by sufficiently structured data.
---
# 4. Search philosophy
The system must optimize for:
1. **Exactness** — exact institution/program should be easy to find.
2. **Tolerance** — spelling mistakes should usually still work.
3. **Global naming** — local and English names should work.
4. **Simplicity** — no 40-filter sidebar on first view.
5. **Trust** — unknown data must not pass a filter as if known.
6. **Determinism** — same query/state should produce stable results.
7. **Speed** — autocomplete must feel immediate.
8. **Shareability** — filters/search state should survive refresh and URL sharing.
---
# 5. Core search surfaces
Ekho has two search experiences.
## A. Quick Search
Used when the user primarily wants to find an entity.
Appears in:
```text
First Home empty state
Global search entry
Add university flow
```
Searches:
```text
Universities
Programs
Countries
Subjects
```
No giant filter UI.
---
## B. Explore Search
Route:
```text
/universities
```
Used for discovery.
Includes:
```text
Search
Filters
Results
Sorting where useful
```
---
# 6. Quick Search ≠ Explore
Do not put the entire Explore filtering UI into:
```text
Add your first university
```
First-user flow should remain:
```text
Search
→ result
→ Add
```
If the user wants discovery:
```text
View all results
→ Explore
```
---
# 7. Searchable entities
Canonical search entities:
```text
institution
program
country
subject
```
Optional later:
```text
city
region
```
Do not search arbitrary page content in the main admissions search.
---
# 8. Institution Search Document
Conceptually:
```text
id
entity_type = institution
canonical_name
official_local_name nullable
aliases[]
abbreviations[]
country_code
city
subdivision nullable
institution_type nullable
supported_status
data_coverage_status
search_active
```
Example:
```text
canonical_name:
Technical University of Munich
official_local_name:
Technische Universität München
aliases:
TU Munich
Munich Technical University
abbreviations:
TUM
```
Aliases must come from curated/verified data.
An LLM must not dynamically invent university aliases at query time.
---
# 9. Program Search Document
Conceptually:
```text
id
entity_type = program
institution_id
canonical_name
official_local_name nullable
aliases[]
subject_ids[]
degree_level
country_code
city nullable
languages[]
study_modes[]
intakes[]
active_status
data_coverage_status
```
---
# 10. Country Search Document
Use canonical geographical IDs.
For countries:
```text
ISO 3166-1 alpha-2
```
Example:
```text
DE → Germany
NL → Netherlands
US → United States
GB → United Kingdom
```
Store:
```text
country_code
canonical_name
aliases[]
```
Examples:
```text
United States
USA
US
U.S.
```
```text
United Kingdom
UK
U.K.
Britain
```
Aliases must be curated.
---
# 11. UK handling
Do not incorrectly model:
```text
England
Scotland
Wales
Northern Ireland
```
as four independent sovereign-country values when using ISO country filtering.
Canonical country:
```text
GB
```
Subdivision may separately represent:
```text
England
Scotland
Wales
Northern Ireland
```
This is important because UCAS itself distinguishes country/region concepts in course discovery.
---
# 12. Subject taxonomy
Programs need a normalized subject taxonomy.
Example:
```text
Computer Science & IT
    ├── Computer Science
    ├── Artificial Intelligence
    ├── Data Science
    ├── Cybersecurity
    └── Information Systems
```
Program title and subject are different fields.
Example:
```text
Program:
Computing and Management
Subjects:
Computer Science
Business & Management
```
---
# 13. Subject taxonomy rule
Do not make every program title its own subject.
Bad:
```text
Economics and Management
Economics with Finance
Economics and Data Science
Economics BSc
```
as four unrelated top-level subjects.
Instead:
```text
Economics
Finance
Business & Management
Data Science
```
can map to programs.
This is essential for reliable cross-country discovery because programme naming conventions differ across universities.
---
# 14. Subject aliases
Maintain curated synonyms.
Example:
```text
Computer Science
Aliases:
CS
CompSci
Computing
```
But synonyms must be applied carefully.
`Computing` may be broader than `Computer Science`.
Subject mappings should therefore support:
```text
exact synonym
related term
```
Do not treat every related term as identical.
---
# 15. Abbreviation safety
Short abbreviations are dangerous.
Examples:
```text
MIT
UCL
LSE
ETH
EPFL
UVA
USC
NYU
```
Do not use aggressive fuzzy matching on very short abbreviations.
Rule:
```text
query length <= 3
→ prefer exact curated abbreviation matching
→ heavily restrict typo tolerance
```
This prevents:
```text
MIT
```
from matching unrelated three-letter strings.
---
# 16. Ambiguous abbreviations
Example:
```text
UVA
```
may refer to more than one institution in different contexts.
Do not silently decide one.
Return multiple exact alias matches where appropriate.
Possible presentation:
```text
University of Amsterdam
Amsterdam, Netherlands
University of Virginia
Charlottesville, United States
```
Country context/filter can resolve the ambiguity.
---
# 17. Text normalization
Search matching should normalize:
```text
case
extra whitespace
common punctuation
hyphen variation
apostrophe variation
diacritics where safe
```
Examples:
```text
Université
universite
```
should be searchable together.
```text
Paris-Saclay
Paris Saclay
```
should match.
Display must always preserve the canonical proper name.
---
# 18. Unicode
Search must support Unicode safely.
Do not force all indexed text into ASCII.
Global-first means names may contain:
```text
é
ü
ö
å
ø
ł
č
ş
etc.
```
Normalization is for matching.
It must not destroy the displayed official name.
---
# 19. Multilingual architecture
The search architecture must not assume all future queries will be English.
Modern search systems can operate over multiple writing systems and multiple languages, and this should be preserved architecturally even if English is the first Ekho UI language.
### v1 decision
Search primarily using:
```text
English canonical name
+
official local name
+
curated aliases
```
Do not implement automatic AI translation of every query in v1.
---
# 20. Search query normalization
Conceptually:
```text
raw_query
↓
trim
↓
Unicode normalization
↓
case normalization
↓
whitespace normalization
↓
safe punctuation handling
↓
search
```
Never mutate the visible query while the user is typing.
---
# 21. Search must be typo tolerant
Modern search systems deliberately support:
* missing characters;
* extra characters;
* swapped characters;
* substituted characters.
Algolia also ranks perfect matches above one-typo matches and one-typo matches above two-typo matches.
Ekho should preserve the same general principle.
---
# 22. Typo ranking rule
Conceptual order:
```text
0 typo
>
1 typo
>
2 typo
```
Never allow fuzzy matching to outrank an otherwise identical exact match.
---
# 23. Fuzzy matching restrictions
Typo tolerance should be stronger for longer strings.
Example:
```text
Massachusets Institute of Technology
```
should still find:
```text
Massachusetts Institute of Technology
```
But:
```text
MIT
```
should not receive broad fuzzy expansion.
---
# 24. Prefix matching
Search must support as-you-type prefix matching.
Example:
```text
stan
→ Stanford University
```
```text
imper
→ Imperial College London
```
Prefix matching is a standard mechanism for responsive as-you-type search.
---
# 25. Exact matches always matter more
For query:
```text
Oxford
```
a direct institution-name/alias match should rank above a program containing Oxford somewhere in secondary text.
---
# 26. Searchable attribute priority
Institution priority:
```text
1. canonical_name
2. exact curated abbreviation
3. exact curated alias
4. official_local_name
5. city/location
```
Program priority:
```text
1. canonical program name
2. exact program alias
3. mapped subject
4. institution name
5. location
```
Search systems such as Algolia explicitly allow higher-priority searchable attributes to rank matches above lower-priority attributes.
---
# 27. Search result ranking principle
Ranking must answer:
> Which result most likely corresponds to this query?
Not:
> Which university does Ekho think is best?
---
# 28. Canonical ranking hierarchy
Conceptually:
```text
1. Entity/intent match
2. Exactness
3. Number of matched query terms
4. Attribute importance
5. Word/token proximity
6. Typo distance
7. Filter/context match
8. Search quality tie-breakers
9. Stable final tie-break
```
This follows the broader search principle that textual relevance should be established before custom/business ranking. Modern relevance systems similarly consider typo count, matching words, attributes, proximity and exact matches before custom ranking.
---
# 29. No prestige ranking in relevance
Forbidden ranking features:
```text
QS rank
THE rank
US News rank
acceptance rate
brand prestige
```
as primary search relevance signals.
Example:
Query:
```text
University of Amsterdam
```
must not rank Harvard above Amsterdam because Harvard has a better external ranking.
---
# 30. No paid ranking
If Ekho ever sells university advertising or partnerships:
```text
paid ≠ organic ranking signal
```
Sponsored results, if ever introduced, must:
* be clearly labeled;
* remain separate from organic relevance.
Do not silently alter search ordering.
---
# 31. Search quality tie-breakers
Only after textual relevance is substantially tied, Ekho may use:
```text
verified data coverage
active program status
user-specific exact context
historical interaction popularity
```
as weak tie-breakers.
---
# 32. Data coverage tie-breaker
Suppose two broad program results match equally.
It is reasonable to prefer the record with:
```text
verified active program
+
usable admissions data
```
over a sparse/unsupported record.
But:
**data completeness may never defeat a clear exact query match.**
---
# 33. Popularity tie-breaker
Internal popularity may later use aggregated:
```text
search result clicks
application adds
```
But it must be:
```text
weak
capped
after textual relevance
```
Otherwise famous institutions will systematically bury smaller but better-matching institutions.
---
# 34. Cold-start ranking
Before Ekho has enough behavioral data:
Use:
```text
text relevance
→ verified/active status
→ deterministic final ordering
```
Do not invent popularity numbers.
---
# 35. Final deterministic tie-break
Results with otherwise equal scores need stable order.
Possible final tie-break:
```text
canonical_name ascending
```
or stable internal rank key.
Never randomize organic results on each request.
---
# 36. Query intent
Autocomplete may detect a small number of **deterministic entity intents**.
Example:
```text
Germany
```
exactly matches a country entity.
Autocomplete can therefore show:
```text
Country
Germany
```
before generic text results.
Likewise:
```text
Computer Science
```
can show:
```text
Subject
Computer Science
```
This is entity matching.
It is not AI intent prediction.
---
# 37. Do NOT implement generic natural-language parsing in v1
Do not attempt to interpret arbitrary text such as:
```text
cheap good universities in europe for russian student with 130 DET
```
into hidden filters automatically.
Reasons:
* hard to explain;
* introduces uncertainty;
* difficult to debug;
* risks silently applying wrong constraints.
User should search and filter explicitly.
Natural-language discovery can be evaluated separately later.
---
# 38. Autocomplete trigger
Autocomplete begins when:
```text
normalized query length >= 2
```
Exception:
known high-value one-character behavior does not justify changing this in v1.
For empty query, do not send pointless search requests.
---
# 39. Autocomplete debounce
Target implementation:
```text
~100–200ms debounce
```
Recommended default:
```text
150ms
```
Do not wait for the user to press Enter before offering known matches.
---
# 40. Race-condition handling
If query sequence is:
```text
sta
stan
stanf
stanfo
```
and the `sta` request returns after `stanfo`:
**the stale response must not overwrite current results.**
Use:
```text
request cancellation
or
request sequence/version checks
```
---
# 41. Autocomplete result limit
Default:
```text
max ~8 visible suggestions
```
Do not show 40 autocomplete results.
Possible composition:
```text
top matched entity suggestions
+
See all results
```
The exact entity distribution should depend on relevance rather than fixed quotas when one type clearly dominates.
---
# 42. Autocomplete sections
When useful:
```text
Universities
Programs
Countries
Subjects
```
But hide empty groups.
Do not render:
```text
Countries
No results
```
inside the dropdown.
---
# 43. Autocomplete result content
University:
```text
University name
City · Country
```
Program:
```text
Program name
University · Country
```
Country:
```text
Country
```
Subject:
```text
Subject
```
Keep it compact.
---
# 44. Match highlighting
Autocomplete may visually highlight the matched text.
Example:
Query:
```text
stan
```
Result:
```text
Stanford University
```
Do not highlight fuzzy characters in a confusing way.
---
# 45. Autocomplete interaction
Required keyboard behavior:
```text
Arrow Down → next result
Arrow Up → previous result
Enter → open selected result
Escape → close autocomplete
```
Search field must remain usable without mouse input.
---
# 46. Enter without selected suggestion
If a query exists and no suggestion is selected:
```text
Enter
→ full Explore results
```
Example:
```text
/universities?q=computer%20science
```
---
# 47. "See all results"
Autocomplete bottom action:
```text
See all results for "computer science"
```
Navigates to Explore.
---
# 48. Country autocomplete behavior
Query:
```text
Germany
```
may produce:
```text
Country
Germany
Universities
Technical University of Munich
...
Programs
...
```
Selecting `Country — Germany` should apply:
```text
country = DE
```
and open Explore.
---
# 49. Country route state
Canonical concept:
```text
/universities?country=DE
```
Do not use human-readable country text as the database identifier.
---
# 50. Country filter
Type:
```text
multi-select
```
Examples:
```text
Germany
Netherlands
Italy
United States
```
Users can select multiple countries.
Within Country:
```text
Germany OR Netherlands
```
---
# 51. Subject/program autocomplete
Query:
```text
Computer Science
```
can produce:
```text
Subject
Computer Science
```
plus matching program results.
Selecting the subject enters program discovery:
```text
subject = computer-science
```
---
# 52. Program search semantics
Program search must match:
```text
program title
program alias
normalized subjects
institution name
```
But result ranking should distinguish the reason for matching.
Example query:
```text
Computer Science Oxford
```
A Computer Science-related Oxford program should outrank:
* unrelated Oxford programs;
* Computer Science programs at unrelated institutions.
---
# 53. Subject search vs exact program title
Search:
```text
Economics
```
may refer to:
1. programs literally called Economics;
2. broader subject Economics.
Autocomplete should therefore allow:
```text
Subject — Economics
```
and individual exact programs.
The user chooses intent.
---
# 54. Search by institution + program
Support multi-token queries such as:
```text
Oxford PPE
Stanford CS
UCL Economics
Bocconi Economics
```
The search engine should reward records matching both:
```text
institution
+
program/subject
```
rather than requiring one concatenated exact phrase.
---
# 55. Institution alias inheritance
Program search should be searchable through institution aliases.
Example:
```text
LSE Economics
```
can find Economics programs at:
```text
London School of Economics and Political Science
```
if:
```text
LSE
```
is a curated institution abbreviation.
---
# 56. Primary Explore filters
Initial visible filters should be limited to the highest-value dimensions.
### v1 primary
```text
Country
Subject / Program
Degree level
Language
```
Depending on current launch scope, Degree Level should be hidden if Ekho only contains one supported level.
Never show a filter that cannot change the result set.
---
# 57. Secondary filters
Inside:
```text
More filters
```
Potential verified filters:
```text
Study mode
Intake / start period
Tuition
Financial aid
Institution type
```
Only activate them when data quality is sufficient.
---
# 58. Filters intentionally NOT core v1
Do not prioritize:
* campus size;
* campus vibe;
* sports division;
* Greek life;
* religious affiliation;
* gender composition;
* weather;
* nightlife;
* arbitrary prestige tiers;
* acceptance probability;
* "reach/target/safety";
* social popularity.
Common App and BigFuture expose substantially broader lifestyle/institutional filtering.
Ekho's differentiation is not reproducing every filter they have.
---
# 59. Why Country is primary
For an international-first applicant, location determines large parts of:
```text
admissions system
cost structure
visa environment
application route
degree structure
```
Country is therefore a first-class discovery dimension.
---
# 60. Why Subject is primary
UCAS, BigFuture and global study portals all treat subject/major/program as a central search dimension.
Ekho should do the same.
---
# 61. Degree Level filter
Canonical values should come from normalized Ekho taxonomy.
Example:
```text
Bachelor's / Undergraduate
Master's
Doctoral
Foundation
Other supported levels
```
Do not index every university's marketing label as a new degree level.
---
# 62. Degree-level visibility
If launch v1 only supports undergraduate applications:
Do not show:
```text
Degree level: Undergraduate
```
as a useless permanent filter.
The architecture should support future levels, but the interface should only expose useful choices.
---
# 63. Language filter
Filter:
```text
Language of instruction
```
Examples:
```text
English
German
French
Italian
Spanish
Dutch
```
Program may have multiple languages.
If:
```text
languages = [English, German]
```
it should match either selected language.
---
# 64. Language filter semantics
Selecting:
```text
English
```
means:
> Return programs with verified data saying English is a language of instruction.
It does **not** mean:
> Return programs for which Ekho simply doesn't know the language.
---
# 65. Study Mode
Possible normalized values:
```text
Full-time
Part-time
Online
Hybrid
```
Only use modes that can be represented reliably across the dataset.
UCAS currently exposes study mode as a course-search filter, including full-time, part-time and distance learning where available.
---
# 66. Intake filter
Potential values should represent actual program intake/start periods.
Examples:
```text
Fall / Autumn
Spring
January
September
2027 intake
```
Do not force all countries into US-style:
```text
Fall
Spring
```
internally.
Prefer canonical date/start-term representation and localize display.
---
# 67. Intake filter caution
Different programs may have:
* multiple intakes;
* rolling starts;
* exact months;
* academic-year cycles.
Therefore intake filtering must be based on structured program intake data, not inferred from country.
---
# 68. Tuition filter
Tuition is valuable but dangerous.
Studyportals uses tuition as a major discovery filter, confirming real discovery value.
However Ekho must model it more carefully because tuition often changes by applicant category.
---
# 69. Generic tuition must NOT lie
Do not show:
```text
Tuition ≤ $10,000
```
against a generic university-level fee when actual tuition depends on:
```text
nationality
EU/EEA status
residency
program
year
credit load
```
---
# 70. Tuition filtering availability
Enable tuition filtering only when Ekho can determine a meaningful comparable fee scope.
Possible states:
```text
Applicant fee category known
→ personalized tuition filter
Applicant fee category unknown
→ verified international/non-local tuition where explicitly defined
Neither available reliably
→ do not expose the filter yet
```
---
# 71. Tuition normalization
Preserve:
```text
original amount
original currency
original period
source
scope
```
and separately compute normalized comparison fields.
Example:
```text
Original:
€15,000 / academic year
Normalized comparison:
15000 EUR/year
```
Currency conversion for discovery is a derived value and must not overwrite source values.
---
# 72. Financial Aid filter
This is strategically important for Ekho but must be source-grounded.
Potential UI:
```text
Financial aid
```
Options may eventually include:
```text
Aid available to international applicants
Need-based aid available
Merit scholarships available
```
Do not create one vague:
```text
Offers financial aid
```
boolean if policy is more complex.
---
# 73. Aid filter activation
Only ship financial-aid filtering once coverage is sufficiently reliable for the target dataset.
An incorrect negative:
```text
No aid
```
could wrongly eliminate an institution.
Therefore unknown financial-aid data must remain distinct from:
```text
aid unavailable
```
---
# 74. Institution Type
Potential normalized values:
```text
Public
Private nonprofit
Private for-profit
Other
```
But these classifications are not equally meaningful in every country's higher-education system.
Therefore Institution Type belongs under:
```text
More filters
```
rather than being a primary global filter.
---
# 75. Acceptance Rate
**Do not make Acceptance Rate a global v1 filter.**
Reasons:
* unavailable/non-comparable globally;
* may differ substantially by program;
* international admission may differ;
* encouraging simplistic reach/safety classification conflicts with Ekho's trust principle.
It may exist as contextual intelligence where reliable.
Not as a fundamental global search primitive.
---
# 76. Test-optional filter
Do not ship:
```text
Test Optional
```
as a global university boolean.
It can vary by:
```text
cycle
program
applicant type
qualification
international status
```
Any future filter must use the same scoped rule system as Requirements for Me.
---
# 77. Application deadline filter
Do not ship a generic:
```text
Deadline before/after X
```
in initial discovery.
Application deadlines often depend on:
```text
program
round
intake
applicant category
application route
```
A generic institution-level deadline filter would create false certainty.
---
# 78. "Applications open" filter
Potential future feature.
Only implement when Ekho can reliably evaluate:
```text
program
+
intake
+
applicant scope
+
current date
```
Do not infer it from a generic university webpage.
---
# 79. Filter boolean semantics
Within the same facet:
```text
OR
```
Example:
```text
Country:
Germany
Netherlands
```
means:
```text
Germany OR Netherlands
```
Across facets:
```text
AND
```
Example:
```text
Country = Germany OR Netherlands
AND
Subject = Computer Science
AND
Language = English
```
---
# 80. Multiple subjects
If user selects:
```text
Computer Science
Economics
```
default interpretation:
```text
Computer Science OR Economics
```
not programs required to contain both subjects.
Joint-program discovery can be handled separately later.
---
# 81. Filter application
Desktop:
Filters may update results immediately.
Mobile:
User may use a filter sheet with:
```text
Show X results
```
if live updates would make the sheet unstable.
Do not require an extra Apply button on desktop without reason.
---
# 82. Active filter chips
Selected filters must be visible.
Example:
```text
Germany ×
Netherlands ×
Computer Science ×
English ×
```
Actions:
```text
individual remove
Clear all
```
---
# 83. Filters must survive navigation
User:
```text
search
→ filter
→ open university
→ Back
```
must return to:
```text
same query
same filters
same result state
```
Do not reset discovery.
---
# 84. URL is source of shareable search state
Core query/filter state should be serializable.
Conceptual URL:
```text
/universities
?q=computer-science
&country=DE,NL
&subject=computer-science
&language=en
```
Use stable IDs/codes rather than localized labels where practical.
---
# 85. Refresh behavior
Refreshing a filtered Explore page must preserve:
```text
query
filters
sort
result mode
```
Do not rely exclusively on React component state.

---
# 86. Back/forward browser behavior
Browser:
```text
Back
Forward
```
must correctly restore search state.
Changing URL-backed filter state should behave consistently with browser history.

---
# 87. Unknown filter data — fundamental rule
For a positive filter:
```text
Language = English
```
record:
```text
language = unknown
```
must **not** pass the filter.
Likewise:
```text
Aid available = true
```
must not match:
```text
aid = unknown
```
---
# 88. Unknown ≠ false
Critical invariant:
```text
unknown
≠
false
```
For every filterable admissions field.

---
# 89. Filtering unknown data
When a data-sensitive filter removes many records because their value is unknown, Ekho should be transparent where materially useful.
Example:
```text
128 verified matches
```
Optional supporting notice:
```text
Some programs are excluded because tuition data is unavailable.
```
Do not pretend the filtered universe is exhaustive if it is not.
---
# 90. Filtering only on indexed structured fields
Do not filter using:
* AI summaries;
* free-text descriptions;
* scraped unnormalized paragraphs.
Filter values must come from structured canonical fields.
---
# 91. Search index is NOT source of truth
Search index:
```text
derived read model
```
Canonical database:
```text
source of truth
```
Never edit university/program data directly inside the search index.
---
# 92. Reindex flow
Conceptually:
```text
Canonical DB update
↓
validation
↓
search projection generated
↓
search index updated
```
Search documents should be reproducible from canonical data.
---
# 93. Deleted/retired records
If program is:
```text
retired
no longer offered
invalid
```
remove it from normal search.
Do not permanently delete historical canonical data solely because it leaves active search.
---
# 94. Program active status
Programs should conceptually support:
```text
active
temporarily unavailable
not_offered
retired
unknown
```
Only appropriate active/current records should appear in default discovery.
---
# 95. Institution without program coverage
Institution may still appear in institution search.
Example:
```text
University X
```
can be searchable even if:
```text
program coverage unavailable
```
But it must not produce invented programs.
---
# 96. Program without sufficient verified data
A known real program may appear in search even if admissions coverage is incomplete.
Search result can still open Program Intelligence.
Do not fabricate missing metadata to improve the card.
---
# 97. Search result modes
Explore may support:
```text
All
Universities
Programs
```
Default:
```text
All
```
But if program-specific filters are applied:
```text
Subject
Language
Study mode
Intake
```
the interface should naturally prioritize:
```text
Programs
```
rather than showing irrelevant institution-only cards.
---
# 98. Avoid unnecessary tabs
Do not introduce tabs merely because multiple entity types exist.
If one combined ranked list with clear entity labels tests better, that is acceptable.
Invariant:
> Users must always understand whether a result is a university or a program.
---
# 99. University result card
Minimum:
```text
University name
City · Country
relevant matching context if necessary
```
Optional:
```text
number of matching programs
```
when a subject/program filter is active and that count is reliable.
---
# 100. Program result card
Minimum:
```text
Program name
University
Location
Degree level
```
Useful only when known:
```text
Language
Intake
Tuition
```
Do not show unknown values as:
```text
N/A
```
everywhere.
Hide optional fields when absent unless absence itself matters.
---
# 101. Result cards should not become mini university pages
Do not add:
* massive photos;
* long descriptions;
* ten statistics;
* campus trivia;
* full requirement lists.
Search results optimize scanning.
University/Program Intelligence handles detail.
---
# 102. Add action in results
Where context permits:
```text
+ Add
```
can appear directly on university/program search result.
This supports Core User Flow:
```text
Search
→ Add
```
without unnecessary page opening.
---
# 103. Existing application state
If exact application already exists:
replace:
```text
+ Add
```
with:
```text
View application
```
or appropriate existing-state indicator.
Do not create duplicates from Explore.
---
# 104. Sorting
Default:
```text
Relevance
```
for non-empty text queries.
---
# 105. Browse sorting
For empty text query + filters, relevance has less meaning.
Possible supported sort:
```text
Name A–Z
```
Later, only with valid comparable data:
```text
Tuition low → high
```
Do not ship every conceivable sorting option.
---
# 106. No "Best Universities" default sort
Forbidden:
```text
Best
Top
Recommended
Prestige
```
unless Ekho eventually defines and transparently validates exactly what the metric means.
Do not create an invisible Ekho university ranking.
---
# 107. Tuition sorting
Only enable when:
```text
tuition field is comparable
+
scope is appropriate
+
currency normalization available
```
Unknown tuition values:
```text
after known values
```
or excluded according to explicit filter semantics.
Never treat unknown as zero.
---
# 108. Result count
Show:
```text
X results
```
when useful.
Avoid giant celebratory numbers.
Result count must represent current:
```text
query + filters
```
---
# 109. Pagination
Do not load the entire program corpus into the browser.
Use server/search-provider pagination.
Recommended UI:
```text
progressive pagination / load more
```
or conventional pages depending final design.
Infinite scroll is not mandatory.
---
# 110. Result page size
Reasonable starting point:
```text
20–30 results per request
```
Exact number may be tuned based on actual card size/performance.
Do not hardcode thousands of records into a client-side array.
---
# 111. No-results state
Modern search guidance recommends helpful alternatives rather than a dead empty state.
Ekho should distinguish:
```text
query_no_results
filter_no_results
```
---
# 112. Query no results
Example:
```text
No results for "Stanfrod Bussines"
```
Possible actions:
```text
Check spelling
Clear search
Request university/program
```
If a reliable close match exists:
```text
Did you mean Stanford…?
```
But only from deterministic search suggestions.
---
# 113. Filter no results
Example:
```text
No programs match all of these filters.
```
Show active filters.
Actions:
```text
Remove one filter
Clear all
```
Do not erase user selections automatically.
---
# 114. Automatic filter relaxation is forbidden
If:
```text
Germany
Computer Science
English
≤ €5,000
```
returns zero results:
do not silently remove:
```text
≤ €5,000
```
and display programs anyway.
That destroys filter trust.
---
# 115. Relaxed suggestions
Ekho may separately say:
```text
No exact matches.
12 programs match if you remove the tuition filter.
```
Only if clearly labeled.
Never mix relaxed results into exact results without explanation.
---
# 116. Query-word relaxation
Search engines can improve no-results cases by removing less-important query words or using optional words.
For Ekho this should be conservative.
Exact university/program entity searches require precision.
Do not aggressively turn:
```text
Oxford medicine
```
into unrelated:
```text
Oxford OR medicine
```
results without clear ranking.
---
# 117. Synonyms
Maintain synonym dictionaries for predictable high-value concepts.
Examples:
```text
CS
↔ Computer Science
```
```text
AI
↔ Artificial Intelligence
```
```text
Econ
↔ Economics
```
```text
CompSci
↔ Computer Science
```
Search systems use synonyms specifically to accommodate vocabulary variation.
---
# 118. Synonym governance
Every synonym has:
```text
canonical_entity_id
alias
type
reviewed_at
```
Do not create uncontrolled global synonyms.
---
# 119. One-way synonyms
Some queries should broaden one way only.
Example conceptually:
```text
CS
→ Computer Science
```
does not necessarily mean every occurrence of:
```text
Computer Science
```
should be replaced by `CS`.
Search implementation should support this distinction if the provider allows it.
---
# 120. Acronym dictionary
Maintain separately from ordinary spelling synonyms.
Examples:
```text
MIT
TUM
ETH
EPFL
LSE
UCL
NYU
UCLA
USC
```
Ambiguous acronyms can map to multiple entities.
---
# 121. Search history
Optional lightweight feature.
For signed-in users, Quick Search may show recent entities when the field is empty.
Maximum:
```text
~3–5 recent searches/entities
```
Do not turn this into another recommendation feed.
---
# 122. Recent search rules
Recent history must be:
* user-specific;
* removable;
* not used as canonical relevance evidence;
* privacy-compatible.
This is convenience, not personalization infrastructure.
---
# 123. Personalization in search ranking
Do **not** heavily personalize organic search ranking in v1.
Example:
A user previously looked at Germany.
Searching:
```text
Cambridge
```
must still find Cambridge correctly rather than boosting unrelated German institutions.
---
# 124. Safe personal context usage
Known user context may help:
```text
resolve equal ambiguous results
preselect explicitly requested filters
```
but should never override exact query intent.
---
# 125. Autocomplete and application status
Autocomplete may display:
```text
Added
```
for institutions/programs already in Applications.
This prevents accidental duplicate actions.
---
# 126. Search response model
Conceptually:
```text
query
normalized_query
results[]
facets
total
page/cursor
applied_filters
search_request_id
```
Each result:
```text
entity_type
entity_id
title
subtitle
match metadata where needed
search ranking metadata server-side
```
Do not expose sensitive internal relevance implementation unnecessarily to clients.
---
# 127. Search provider abstraction
Frontend must not depend directly on a vendor-specific search SDK if avoidable.
Conceptual architecture:
```text
UI
↓
Ekho Search API / service
↓
Search provider/index
```
This preserves ability to change:
```text
Algolia
Typesense
Meilisearch
Postgres-based implementation
etc.
```
without rewriting product UI.
---
# 128. Do not lock the product spec to Algolia
Algolia documentation is being used here to verify mature search principles:
```text
typo tolerance
prefix matching
synonyms
attribute ranking
relevance
```
It does **not** mean Ekho must use Algolia.
Vendor choice belongs to implementation architecture.
---
# 129. Search API responsibilities
Backend search layer should own:
```text
query normalization
filter validation
filter authorization
provider translation
result normalization
pagination
analytics request id
```
Frontend should not manually reconstruct search logic.
---
# 130. Filter validation
Never trust URL/query parameters blindly.
Example:
```text
country=FAKE
```
must not reach unsafe query construction.
Validate against canonical enum/taxonomy.
---
# 131. Query security
Search input is user input.
Implementation must prevent:
* SQL injection;
* search DSL injection where relevant;
* malformed filter expressions;
* uncontrolled regex execution;
* HTML injection in highlighted results.
Never render provider-generated highlight HTML unsafely.
---
# 132. Search index privacy
Search index contains public university/program discovery information.
It should not contain:
* private user profile data;
* test scores;
* personal notes;
* documents;
* financial documents.
User context should be applied separately where required.
---
# 133. Analytics events
Minimum:
```text
search_opened
search_query_started
search_query_submitted
search_suggestions_returned
search_suggestion_clicked
search_results_viewed
search_result_clicked
search_filter_opened
search_filter_applied
search_filter_removed
search_filters_cleared
search_sort_changed
search_zero_results
search_add_clicked
```
---
# 134. Analytics properties
Useful:
```text
search_request_id
entry_surface
query_length
result_count
entity_type
entity_id
position
filter_keys
sort
zero_result_reason
```
Potential entry surfaces:
```text
first_home
global_search
explore
application_add
```
---
# 135. Query analytics privacy
Free-text search queries can theoretically contain arbitrary user-entered information.
Therefore:
* do not attach unnecessary personal profile fields to search analytics;
* define retention/redaction separately;
* do not assume every entered query is safe public metadata.
---
# 136. Search quality metrics
Track:
## Search success rate
```text
queries producing ≥1 result
/
submitted queries
```
## Zero-result rate
```text
zero-result queries
/
submitted queries
```
## Suggestion CTR
```text
suggestion clicks
/
autocomplete sessions
```
## Result CTR
```text
result clicks
/
result views
```
## Search → Add
Critical Ekho metric:
```text
applications added from search
/
search sessions
```
---
# 137. Position distribution
Track which result positions produce clicks/adds.
If users constantly scroll past position 1–3 and select position 7:
ranking likely needs improvement.
Do not automatically conclude that the clicked university should simply receive a global popularity boost.
Inspect query-level relevance first.
---
# 138. Zero-result analytics
Store/analyze recurring zero-result patterns.
Examples:
```text
misspelled university
missing alias
missing university
missing program
missing subject synonym
unsupported country
```
Modern search guidance specifically recommends inspecting no-result search analytics to improve the index and vocabulary.
---
# 139. Search quality feedback loop
Conceptually:
```text
Query logs
↓
Zero-result analysis
↓
Missing aliases/synonyms detected
↓
Human/data review
↓
Canonical search dictionary updated
↓
Reindex
```
Do not allow an LLM to permanently update production synonyms automatically.
---
# 140. Initial ranking evaluation set
Before launch create a fixed search relevance test set.
At minimum:
```text
100–200 representative queries
```
including:
```text
exact names
abbreviations
typos
countries
subjects
programs
institution + program
diacritics
ambiguous aliases
```
Search changes must be tested against it.
---
# 141. Required institution query tests
Examples:
```text
MIT
→ Massachusetts Institute of Technology
```
```text
stanf
→ Stanford University
```
```text
Massachusets Institute of Technology
→ Massachusetts Institute of Technology
```
```text
LSE
→ London School of Economics and Political Science
```
```text
TUM
→ Technical University of Munich
```
```text
TU Munich
→ Technical University of Munich
```
```text
EPFL
→ EPFL
```
```text
Paris Saclay
→ Université Paris-Saclay
```
provided those aliases/names exist in the canonical search dictionary.
---
# 142. Required program query tests
```text
computer science
```
returns relevant CS programs/subject.
```text
computer scince
```
still finds Computer Science.
```text
CS
```
uses curated CS abbreviation.
```text
Oxford PPE
```
rewards a PPE/Oxford match.
```text
LSE Economics
```
matches Economics at LSE.
```text
Bocconi economics
```
prioritizes Bocconi program matches.
---
# 143. Required country query tests
```text
Germany
→ Country: Germany
```
```text
DE
```
must not automatically be assumed to mean Germany unless explicitly supported as a user-facing alias.
ISO codes may be used internally without necessarily becoming search aliases.
```text
USA
→ United States
```
if curated.
```text
UK
→ United Kingdom
```
if curated.
---
# 144. Required ambiguous query tests
```text
UVA
```
If multiple entities use this abbreviation:
do not silently discard the other legitimate match.
```text
USC
```
must receive the same ambiguity review.
---
# 145. Filter test matrix
Test:
```text
Country only
Subject only
Country + Subject
Country + Subject + Language
Multiple countries
Multiple subjects
No matches
Unknown filter values
```
---
# 146. Unknown filter test
Dataset:
```text
Program A:
language = English
Program B:
language = Unknown
```
Filter:
```text
Language = English
```
Expected:
```text
Program A included
Program B not included
```
Program B must not be interpreted as non-English globally; it is simply excluded from this verified positive match.
---
# 147. Multi-filter semantics test
Dataset:
```text
A: Germany, CS, English
B: Netherlands, CS, English
C: France, CS, English
D: Germany, Economics, English
```
Filters:
```text
Country = Germany OR Netherlands
Subject = CS
Language = English
```
Expected:
```text
A
B
```
---
# 148. Search state test
```text
Search
→ apply filters
→ open result
→ Back
```
Expected:
```text
same query
same filters
same scroll/result context where reasonable
```
---
# 149. URL test
Open directly:
```text
/universities?country=DE&subject=computer-science
```
Expected:
* filter UI reflects state;
* search backend receives same state;
* results match;
* refresh preserves it.
---
# 150. Stale autocomplete test
Simulate:
```text
query A request slow
query B request fast
```
Expected:
Query B results remain visible.
A must not overwrite B after returning.
---
# 151. Search failure state
If provider/backend fails:
Display:
```text
Search couldn't load.
```
Action:
```text
Try again
```
Preserve:
```text
query
filters
```
Do not reset the user's work.
---
# 152. Partial facet failure
If search results can load but one secondary aggregation/facet fails:
do not necessarily destroy the entire result page.
Fail gracefully where architecture permits.
Core results are more important than a secondary count.
---
# 153. Loading state
Autocomplete:
small lightweight loading indicator if required.
Full Explore:
preserve existing results while new filters are fetching where this prevents jarring blank screens.
Do not flash:
```text
0 results
```
between every keystroke.
---
# 154. Mobile search
Mobile first entry:
```text
Search field
↓
autocomplete
↓
result
```
Explore:
```text
Search
[Filters]
results
```
Filters open in a dedicated sheet/surface.
Do not permanently consume half the mobile screen with filter controls.
---
# 155. Mobile active filters
Active filters appear as horizontally scrollable/chip controls or equivalent compact UI.
User must always have access to:
```text
Filters
Clear
```
without returning to top through a giant result list.
---
# 156. Desktop filters
Prefer compact discoverability.
Possible layout:
```text
Search
Country
Subject
Language
More filters
```
Avoid giant permanent sidebars unless actual testing shows the dataset requires it.
---
# 157. Empty Explore state
If:
```text
query = empty
filters = none
```
do not show a meaningless:
```text
1000 universities
```
wall as the main experience without structure.
Possible simple discovery entry points:
```text
Search
Browse by country
Browse by subject
```
Keep them subordinate to search.
---
# 158. Browse by country
Can expose countries represented in Ekho's active dataset.
Do not list countries with zero active institutions/programs just because they exist in ISO.
---
# 159. Browse by subject
Can expose canonical top-level subject taxonomy.
Do not expose every micro-subject simultaneously.
Use hierarchy/progressive disclosure.
---
# 160. Filter result counts
Facets may display counts:
```text
Germany 82
Netherlands 31
```
Counts must represent the current compatible query/filter universe.
Do not compute misleading static counts disconnected from selected filters.
---
# 161. Self-excluding facet behavior
If implementing dynamic facet counts:
Country counts may optionally be calculated while excluding the current Country selection but respecting all other filters.
Whatever semantic is chosen must be consistent.
Do not let counts randomly shift due to implementation inconsistencies.
---
# 162. Search index freshness
Institution/program changes must propagate into search after canonical-data updates.
Examples:
```text
program added
program renamed
program retired
country corrected
language corrected
```
Search must not remain indefinitely stale.
---
# 163. Alias migration
If a university officially changes name:
Preserve old name as a searchable alias when appropriate.
Example conceptual model:
```text
canonical current name
+
former names[]
```
Search should still help applicants who know the old name.
---
# 164. Duplicate institution names
Never assume institution name is globally unique.
Identity is:
```text
institution_id
```
not:
```text
name
```
Display location to disambiguate.
---
# 165. Duplicate program names
Thousands of institutions can have:
```text
Computer Science
Economics
Business Administration
```
Program identity therefore requires:
```text
program_id
+
institution context
```
not program name.
---
# 166. Search data invariant
Every search result must map back to exactly one canonical Ekho entity.
Forbidden:
```text
search-only fake object
```
that cannot open a canonical route.
---
# 167. Entity routes
Institution:
```text
/universities/[institutionSlug]
```
Program:
```text
/universities/[institutionSlug]/programs/[programSlug]
```
Country/subject autocomplete selections open Explore with filters rather than fake detail pages unless those pages are explicitly introduced later.
---
# 168. Slugs are not identity
Routing slugs may change.
Search links should derive routes from canonical entity data.
Internal relationships use IDs.
Never use:
```text
institutionSlug
```
as the database foreign key.
---
# 169. Search data coverage status
Search result internally may know:
```text
full
partial
basic
unsupported
```
or whatever canonical Data Standard ultimately defines.
This can influence weak tie-breaking and UX.
It must not be invented inside Search if another standard owns these states.
---
# 170. Search must respect Data Standard
If Data Standard says a value is:
```text
unknown
not_found
not_published
stale
conflicting_sources
```
Search & Filtering must preserve that meaning.
Do not flatten all of these to:
```text
null
```
and then guess filtering behavior.
---
# 171. Search must respect Core User Flows
Search exists upstream of:
```text
University Intelligence
→ Add
→ Requirements for Me
→ Next Action
```
Therefore search should optimize:
```text
find
understand enough
add
```
not maximize time spent browsing.
---
# 172. Search must respect Launch Architecture
The search system must work correctly even if launch coverage is exactly the seeded Ekho universe rather than every university on Earth.
If a user searches outside the dataset:
```text
No verified result
→ Request university
```
not AI fabrication.
---
# 173. Request university flow
No-result action:
```text
Request university
```
Input may reuse the current search query.
Example:
```text
University or program:
[query prefilled]
```
This should feed data-expansion priorities later.
It does not immediately create an unverified canonical entity unless the Data Pipeline permits it.
---
# 174. Search vs request analytics
Repeated missing queries are valuable product signals.
Track aggregate:
```text
requested institution/program
frequency
countries
```
for data expansion.
Do not equate request popularity with university quality.
---
# 175. Recommended v1 filters — final lock
## Always visible
```text
Country
Subject / Program
Language
```
## Visible when multiple levels are supported
```text
Degree level
```
## More Filters — once data is ready
```text
Study mode
Intake
Tuition
Financial aid
Institution type
```
---
# 176. Explicitly defer
```text
Acceptance rate
Reach / Target / Safety
Prestige ranking
Campus lifestyle mega-filters
Weather
Sports
Social life
Deadline range
Generic test-optional
Generic application-open
AI match score
```
These require stronger evidence/data or conflict with Ekho's focus.
---
# 177. Recommended search matching — final lock
Support:
```text
Exact canonical names
Curated aliases
Curated abbreviations
Official local names
Prefix matching
Typo tolerance
Subject synonyms
Institution + program token matching
Country entity matching
Diacritic-safe matching
```
Do not initially require:
```text
LLM search
semantic embeddings
vector search
natural-language filter extraction
automatic query translation
```
---
# 178. Why no semantic/vector search initially
For launch scope, core entities are highly structured:
```text
~seeded universities
+
programs
+
subjects
+
countries
```
Lexical search + aliases + typo tolerance + subject taxonomy is:
```text
more deterministic
easier to test
cheaper
easier to explain
```
Add semantic retrieval only when actual failed-query analysis demonstrates a gap lexical search cannot solve efficiently.
---
# 179. Search relevance golden rule
```text
Exact identity
>
linguistic similarity
>
behavioral popularity
>
business preference
```
Never reverse that ordering.
---
# 180. Critical invariants
```text
INV-SEARCH-01
Exact matches must not be buried by prestige/popularity.
INV-SEARCH-02
Unknown filter values never masquerade as positive matches.
INV-SEARCH-03
Unknown ≠ false.
INV-SEARCH-04
No university/program is generated dynamically by AI.
INV-SEARCH-05
Every result maps to a canonical entity.
INV-SEARCH-06
Search index is derived, not source of truth.
INV-SEARCH-07
Short acronyms do not receive unsafe broad fuzzy matching.
INV-SEARCH-08
Same search state produces stable ranking unless underlying data/config changed.
INV-SEARCH-09
Filters are OR within a facet and AND across facets.
INV-SEARCH-10
Filters survive refresh and navigation.
INV-SEARCH-11
Search never silently relaxes user-selected filters.
INV-SEARCH-12
Paid placement never silently alters organic results.
INV-SEARCH-13
Program search remains institution-aware.
INV-SEARCH-14
University identity never depends on display name alone.
INV-SEARCH-15
Data-sensitive filters are not exposed before coverage is trustworthy.
INV-SEARCH-16
Search relevance does not equal university quality.
INV-SEARCH-17
Search provider failure never destroys query/filter state.
INV-SEARCH-18
Stale asynchronous responses never overwrite newer queries.
INV-SEARCH-19
Retired programs do not appear as ordinary current options.
INV-SEARCH-20
Core search does not require AI.
```
---
# 181. Things Codex must NOT invent
When implementing this system, Codex must not independently add:
* external rankings to relevance;
* acceptance probability;
* Reach/Target/Safety;
* AI university recommendations;
* embeddings/vector DB without explicit decision;
* LLM query parsing;
* hidden personalization ranking;
* sponsored ranking;
* giant filter sidebar;
* campus lifestyle mega-filter set;
* arbitrary new subjects;
* automatically generated aliases;
* automatically generated university entities;
* fake programs;
* filters based on unstructured AI text;
* fake tuition values;
* unknown=zero;
* unknown=false;
* hidden automatic filter relaxation;
* randomized search rankings;
* browser-state-breaking filter implementation.
---
# 182. Codex implementation order
## Phase 1 — Canonical search projection
Implement/index:
```text
institution
country
subject
```
with:
```text
canonical names
aliases
abbreviations
locations
```
---
## Phase 2 — Institution Quick Search
```text
input
→ debounce
→ autocomplete
→ typo/prefix
→ keyboard
→ open/add
```
Validate against golden query set.
---
## Phase 3 — Program search
Add:
```text
program entities
subject mapping
institution inheritance
degree level
language
```
---
## Phase 4 — Explore
Implement:
```text
query
results
country
subject
language
```
and Degree Level if relevant.
---
## Phase 5 — URL state
```text
query
filters
sorting
pagination
```
become refresh/share/back-forward safe.
---
## Phase 6 — Secondary verified filters
Only after data coverage exists:
```text
Study mode
Intake
Tuition
Financial aid
Institution type
```
---
## Phase 7 — Search analytics
```text
zero results
clicks
positions
add conversion
filter usage
```
---
## Phase 8 — Relevance tuning
Use real query logs + fixed golden query test suite.
Do not tune based on developer intuition alone.
---
# 183. Required E2E tests
## E2E-SEARCH-01 — Exact institution
```text
Search exact university
→ correct institution is first relevant result
```
---
## E2E-SEARCH-02 — Acronym
```text
Search curated acronym
→ correct entity found
```
---
## E2E-SEARCH-03 — Typo
```text
Search one-character typo
→ correct institution/program still found
```
---
## E2E-SEARCH-04 — Short acronym safety
```text
short acronym
→ no uncontrolled fuzzy garbage
```
---
## E2E-SEARCH-05 — Country
```text
Search Germany
→ Country entity available
→ selecting applies DE filter
```
---
## E2E-SEARCH-06 — Subject
```text
Search Computer Science
→ Subject entity available
→ selecting applies subject filter
```
---
## E2E-SEARCH-07 — Institution + program
```text
Search institution alias + subject
→ matching institution program prioritized
```
---
## E2E-SEARCH-08 — Diacritics
```text
unaccented query
→ accented canonical entity can still be found
```
where configured.
---
## E2E-SEARCH-09 — Multi-country filter
```text
DE + NL
→ records from either country
```
---
## E2E-SEARCH-10 — Cross-facet filter
```text
DE + NL
AND CS
AND English
→ exact intersection
```
---
## E2E-SEARCH-11 — Unknown filtering
```text
language = unknown
+
English filter
→ record not counted as verified English match
```
---
## E2E-SEARCH-12 — Refresh
```text
query + filters
→ refresh
→ same search state
```
---
## E2E-SEARCH-13 — Back navigation
```text
filtered search
→ open program
→ Back
→ filtered search preserved
```
---
## E2E-SEARCH-14 — Stale request
```text
slow old request
returns after new request
→ old response ignored
```
---
## E2E-SEARCH-15 — Zero query result
```text
unknown university
→ useful no-result state
→ Request university available
```
---
## E2E-SEARCH-16 — Zero filter result
```text
impossible filter combination
→ filters preserved
→ no automatic relaxation
```
---
## E2E-SEARCH-17 — Duplicate add
```text
search result already in Applications
→ View application
→ no duplicate created
```
---
## E2E-SEARCH-18 — Search provider error
```text
provider unavailable
→ error
→ retry
→ query and filters remain
```
---
## E2E-SEARCH-19 — Retired program
```text
retired program
→ excluded from default active search
```
---
## E2E-SEARCH-20 — Result identity
```text
every result
→ valid canonical Ekho route/entity
```
---
# 184. Acceptance criteria
Search & Filtering System is **not ready** until:
* [ ] Institution search works by canonical name.
* [ ] Curated aliases work.
* [ ] Curated abbreviations work.
* [ ] Official local names are searchable when available.
* [ ] Case differences do not break search.
* [ ] Safe punctuation differences do not break search.
* [ ] Diacritics are handled appropriately.
* [ ] Prefix search works.
* [ ] Typo tolerance works.
* [ ] Exact results outrank typo results.
* [ ] Short acronym fuzzy matching is restricted.
* [ ] Ambiguous acronyms can return multiple legitimate entities.
* [ ] Search never depends on external university prestige.
* [ ] Search never invents university entities.
* [ ] Program search works.
* [ ] Program title search works.
* [ ] Subject mappings work.
* [ ] Institution + program queries work.
* [ ] Country entities are searchable.
* [ ] Subject entities are searchable.
* [ ] Selecting Country creates a real country filter.
* [ ] Selecting Subject creates a real subject filter.
* [ ] Autocomplete begins without pressing Enter.
* [ ] Autocomplete has debounce.
* [ ] Stale autocomplete requests cannot overwrite newer results.
* [ ] Keyboard navigation works.
* [ ] Escape closes autocomplete.
* [ ] Enter opens selected entity.
* [ ] Enter with no selection opens Explore results.
* [ ] Full result state is URL-backed.
* [ ] Refresh preserves state.
* [ ] Browser Back preserves state.
* [ ] Country is multi-select.
* [ ] Subject is multi-select where appropriate.
* [ ] OR semantics work within facets.
* [ ] AND semantics work across facets.
* [ ] Language filter uses verified language data.
* [ ] Unknown language does not count as English.
* [ ] Degree-level filter is hidden if useless.
* [ ] Study Mode is only exposed with reliable data.
* [ ] Intake is only exposed with reliable data.
* [ ] Tuition filter respects applicant/fee scope.
* [ ] Unknown tuition is never treated as zero.
* [ ] Financial-aid filter distinguishes unknown from unavailable.
* [ ] Data-sensitive filters are hidden until sufficiently reliable.
* [ ] Acceptance Rate is not a core v1 filter.
* [ ] Generic Test Optional filter is not implemented.
* [ ] Generic Deadline filter is not implemented.
* [ ] Result cards remain compact.
* [ ] University and Program cards are visually distinguishable.
* [ ] Existing Applications do not create duplicate Adds.
* [ ] Zero-query results have useful recovery.
* [ ] Zero-filter results preserve filters.
* [ ] Filters are never silently relaxed.
* [ ] Search index is derived from canonical data.
* [ ] Search results map to canonical IDs.
* [ ] Search queries are validated/sanitized.
* [ ] Provider failures are handled.
* [ ] Retired programs are excluded from normal search.
* [ ] Search analytics are implemented.
* [ ] Zero-result analytics are implemented.
* [ ] Search → Application Add is measurable.
* [ ] Golden relevance query suite exists.
* [ ] All required E2E tests pass.
---
# 185. Canonical Quick Search flow
```text
User focuses Search
↓
Types query
↓
150ms debounce
↓
Normalize query
↓
Search canonical index
↓
Rank by relevance
↓
Autocomplete:
Universities
Programs
Countries
Subjects
↓
User chooses result
University/Program
→ open entity / Add
Country/Subject
→ Explore with filter
No direct selection
→ Enter
→ Explore full results
```
---
# 186. Canonical Explore flow
```text
/universities
↓
Search query
↓
Optional filters:
Country
Subject
Language
Degree Level if relevant
↓
More filters where verified:
Study Mode
Intake
Tuition
Financial Aid
Institution Type
↓
Search service
↓
Exact filter intersection
↓
Relevance ranking
↓
Results
↓
University / Program
↓
Add to Applications
```
---
# 187. Final locked v1
### Search entities
```text
Universities
Programs
Countries
Subjects
```
### Matching
```text
Canonical name
Official local name
Curated aliases
Curated abbreviations
Prefix
Typo tolerance
Subject synonyms
Institution + program matching
```
### Primary filters
```text
Country
Subject / Program
Language
Degree Level when relevant
```
### Secondary verified filters
```text
Study Mode
Intake
Tuition
Financial Aid
Institution Type
```
### Default ranking
```text
Relevance
```
### Ranking principle
```text
Exact query match
>
textual relevance
>
context
>
weak quality/popularity tie-breaker
```
### Explicitly forbidden
```text
Prestige ranking as search relevance
AI match scores
Fake admissions probabilities
Unknown = false
Unknown = zero
Hidden filter relaxation
Generated university/program data
```
---
# 188. Product rule to remember
Ekho Search should feel like:
```text
I type what I know
→ Ekho understands the entity
→ I narrow only what matters
→ I find the correct university/program
→ I add it
```
Not:
```text
I answer 20 questions
→ configure 35 filters
→ receive a mysterious AI ranking
→ guess why university #1 is university #1
```
The strongest version of Ekho Search is **fast, deterministic, global, forgiving of human input and extremely strict about data certainty.**
