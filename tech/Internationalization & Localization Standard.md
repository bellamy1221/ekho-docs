# Internationalization & Localization Standard v1
**Status:** LOCKED
**Scope:** languages, locales, international formatting, multilingual content, RTL, time zones and international UX
**Stack:** Next.js + TypeScript + PostgreSQL + ECMAScript Intl/Unicode standards
**Depends on:** Data Standard, Data Architecture, Auth & Account Lifecycle, Import & Ingestion, Admin & Data Operations
**Research verified:** 12 August 2026
---
# 1. Goal
Ekho must work correctly for applicants, universities and admissions systems across countries.
Internationalization must be architectural, not a future translation patch.
Ekho must correctly handle:
```text
languages
scripts
countries
locales
currencies
dates
times
time zones
number formats
names
RTL
translated UI
multilingual institutional data
```
W3C recommends considering internationalization requirements at specification/architecture level rather than treating them only as translated strings.
---
# 2. Fundamental distinction
Never treat these as the same field:
```text
language
locale
country
nationality
education system
timezone
currency
```
Example:
```text
Nationality: BR
Education system: IB
UI language: en
Locale: en-GB
Timezone: Europe/Paris
University country: US
Tuition currency: USD
```
All are independently valid.
---
# 3. Global-first ≠ translate everything immediately
Ekho may launch its UI initially in English.
But the system must be able to add languages without:
* redesigning database tables;
* rewriting components;
* changing date storage;
* replacing currency representation;
* rebuilding URLs;
* rewriting every string;
* breaking RTL layouts.
The requirement is:
> **Localization-ready from v1, multilingual rollout when justified.**
---
# 4. Terminology
Use:
### Internationalization / i18n
Engineering that enables multiple languages/locales.
### Localization / l10n
Adapting product experience to a specific language/locale.
### Translation
Changing natural-language content from one language into another.
### Locale
A collection of linguistic/cultural preferences used for formatting/processing.
W3C distinguishes natural language identification from locale preferences and recommends standardized language tags for these purposes.
---
# 5. Language identifier standard
Use:
**BCP 47 language tags**
Examples:
```text
en
en-US
en-GB
es
es-MX
pt-BR
zh-Hans
zh-Hant
ar
```
RFC 5646 defines the language-tag structure used by BCP 47.
Do not invent:
```text
english_uk
BrazilianPortuguese
CHINESE_SIMPLIFIED
```
---
# 6. Canonical language representation
Store language/locale values using canonical valid BCP 47 tags.
Recommended application type:
```text
locale: string
```
validated against supported locale configuration.
Do not create application enums containing every possible world language.
---
# 7. Locale is not country
Never infer:
```text
locale = en-US
→ user lives in US
```
or:
```text
country = CA
→ language = English
```
BCP 47 language tags may contain language, script, region and other subtags, but those identifiers describe language/locale selection rather than a person's citizenship or physical location.
---
# 8. Nationality is separate
Ekho admissions personalization may need nationality.
Store it separately according to Data Standard.
Never derive nationality from:
```text
locale
IP
browser language
timezone
```
---
# 9. Education system is separate
Examples:
```text
A-Levels
IB
CBSE
Abitur
French Baccalauréat
Attestat
```
must not be inferred solely from country or UI locale.
This belongs to applicant profiling/admissions personalization.
---
# 10. UI language preference
User preference conceptually:
```text
preferred_locale
```
Example:
```text
en
de
es
```
Optional region-specific variants later:
```text
en-GB
pt-BR
```
Do not require a locale choice during signup.
---
# 11. Locale selection precedence
For authenticated users:
```text
1. explicit user selection
2. saved account preference
3. request/browser language negotiation
4. Ekho default
```
Explicit user choice must override browser defaults.
W3C specifically recommends allowing users to change language and remembering that choice rather than continually overriding it with browser settings.
---
# 12. Default locale
Ekho v1 default:
```text
en
```
Use neutral English where practical.
Avoid unnecessary US-only terminology in global product copy.
---
# 13. `Accept-Language`
Browser `Accept-Language` may be used as a **first-visit hint**.
It must not become the permanent user locale.
W3C warns against relying on `Accept-Language` alone to determine a user's locale because browser language preference does not necessarily represent all of their international preferences.
---
# 14. Do not forcibly redirect language
When multilingual public pages exist:
do not repeatedly force users to another language because of:
* IP address;
* guessed country;
* browser language.
Google advises against automatic language redirects that prevent users/search engines from accessing alternative language versions.
Instead:
```text
suggest preferred language
+
provide obvious switcher
```
---
# 15. Language switcher
Language selection must be:
* easy to find;
* persistent;
* independent from account nationality;
* usable without changing country;
* available on public pages.
Do not hide it inside a large settings hierarchy once Ekho supports multiple UI languages.
---
# 16. No flags for language
Ekho policy:
Do not represent languages using national flags.
Use language labels:
```text
English
Deutsch
Español
Français
Italiano
العربية
```
Reason:
```text
language ≠ country
```
BCP 47 explicitly treats language and region as different components.
---
# 17. Language names
Prefer understandable endonyms in the selector:
```text
English
Deutsch
Español
Français
Italiano
العربية
```
Do not display only machine codes:
```text
EN
DE
ES
FR
```
---
# 18. HTML language metadata
Every rendered page must set correct:
```html
<html lang="...">
```
W3C recommends declaring the page's default language on the HTML element so browsers and language-aware technologies can process the content appropriately.
---
# 19. Mixed-language content
If a substantial embedded string uses another language:
```html
<span lang="it">Politecnico di Milano</span>
```
where useful.
Language metadata should be attached at the appropriate string/content scope rather than guessed from characters. W3C's current internationalization guidance specifically recommends field/string-level language metadata for localizable natural-language values.
---
# 20. Text direction metadata
Language and direction are related but **not identical concepts**.
For strings where direction matters, direction must be determinable separately.
W3C specifically warns that base direction should not simply be inferred from a language tag in all cases.
---
# 21. Character encoding
All Ekho web content:
```text
UTF-8
```
W3C's current authoring guidance recommends UTF-8 for web content.
No legacy encoding pipelines.
---
# 22. Unicode throughout
Database/application strings must support Unicode.
Ekho must correctly preserve:
```text
é
ß
ñ
ø
Ł
İ
ع
م
日
本
한
अ
```
without transliterating or stripping characters merely for internal convenience.
---
# 23. Unicode normalization
For machine comparison/search keys, normalize Unicode consistently.
Recommended default:
```text
NFC
```
for ordinary natural-language canonical text where normalization is needed.
Unicode UAX #15 defines normalization forms so canonically equivalent strings can obtain equivalent normalized representations.
Do not blindly apply compatibility normalization to every user/source string.
---
# 24. Preserve source text
Where exact official wording/name matters, retain canonical source text.
Normalization/search transformations should be derived data.
Never destructively rewrite an official university name purely to improve search matching.
---
# 25. UI messages
No user-visible application string should be scattered randomly through components once localization infrastructure is introduced.
Use message identifiers.
Conceptual:
```text
applications.add
applications.deadline
applications.status.submitted
requirements.missing
```
not:
```text
"Add application"
```
repeated in 15 components.
---
# 26. Message keys
Keys describe meaning, not English wording.
Good:
```text
application.deadline.remaining
```
Bad:
```text
threeDaysLeftText
```
Changing English wording should not require renaming translation keys.
---
# 27. No sentence concatenation
Never build translated sentences from fragments.
Bad:
```text
"You have " + count + " days " + "left"
```
Languages differ in grammar, word order and pluralization.
Unicode MessageFormat exists specifically to support dynamic localized messages with formatting and grammatical selection.
---
# 28. Dynamic messages
Messages containing variables must treat the entire sentence as one localization unit.
Conceptually:
```text
application.deadline_remaining(
  count
)
```
rather than assembling fragments.
---
# 29. Plurals
Never implement:
```text
count === 1
? "day"
: "days"
```
as the global plural system.
CLDR defines locale-specific plural categories because plural behavior differs by language.
Use CLDR/Intl-compatible plural handling.
---
# 30. MessageFormat
Architecture should support the semantics of modern Unicode MessageFormat:
* variables;
* plurals;
* selection;
* formatted numbers;
* formatted dates;
* grammatical variation.
Unicode MessageFormat 2 reached Stable status in CLDR 47.
Do not require a specific JS MessageFormat library until implementation review determines the best supported option.
---
# 31. Runtime formatting
Prefer native:
```text
Intl.DateTimeFormat
Intl.NumberFormat
Intl.RelativeTimeFormat
Intl.PluralRules
Intl.Collator
Intl.DisplayNames
```
where supported by the chosen runtime.
ECMA-402 defines ECMAScript internationalization APIs for locale-sensitive formatting and language/cultural conventions; the 2026 edition is the current published ECMA standard.
---
# 32. CLDR
Do not manually maintain:
```text
month names
currency symbols
country translations
plural rules
number separators
```
when standard locale data already exists.
Unicode CLDR provides standardized locale data used for dates, numbers, currencies, display names and related internationalization behavior.
---
# 33. Date storage vs display
Canonical machine data and localized display are separate.
Store according to Data Standard:
```text
2027-01-05
```
Display depending on locale:
```text
January 5, 2027
5 January 2027
5 janvier 2027
5. Januar 2027
```
W3C's locale guidance explicitly illustrates that one date value may require different display formats across locales.
---
# 34. Critical deadline UX
For important admissions deadlines, avoid ambiguous numeric formats when space allows.
Prefer localized formats containing a month name.
Example:
```text
5 January 2027
```
instead of an ambiguous:
```text
05/01/27
```
This is an Ekho trust/UX rule.
---
# 35. Relative deadline text
Allowed:
```text
3 days left
Tomorrow
In 2 weeks
```
But for critical deadlines always retain an accessible exact date nearby.
Never make:
```text
Tomorrow
```
the only representation of an application deadline.
---
# 36. Date-only values
A university deadline published only as:
```text
January 5
```
remains a **calendar date**.
Do not convert it into a UTC timestamp.
---
# 37. Time-bearing deadlines
If an official source provides:
```text
January 5, 11:59 PM Eastern Time
```
store:
```text
local date
local time
source timezone
```
according to Data Standard.
Then derive user-local display.
---
# 38. Timezone standard
Use **IANA Time Zone Database identifiers**.
Examples:
```text
America/New_York
Europe/London
Europe/Rome
Asia/Singapore
```
IANA's timezone database is maintained to represent civil time and is periodically updated when governments change offsets or daylight-saving rules.
---
# 39. No fixed-offset timezone identity
Do not store a user's long-term timezone as:
```text
UTC+1
```
when:
```text
Europe/Stockholm
```
is known.
Fixed offsets do not encode future daylight-saving/political timezone transitions, while IANA timezone identifiers reference timezone rule histories.
---
# 40. Timezone data must update
Runtime/dependency timezone data must be kept reasonably current.
Government timezone rules can change, and IANA updates the database accordingly.
Do not freeze timezone data indefinitely inside Ekho.
---
# 41. User timezone
User preference:
```text
preferred_timezone
```
Optional.
On first use, browser-reported IANA timezone may seed the preference.
Allow explicit user override.
---
# 42. University timezone
Where time-bearing deadlines require it:
store university/source timezone independently from user timezone.
Example:
```text
University:
America/New_York
Applicant:
Europe/Stockholm
```
Do not overwrite one with the other.
---
# 43. Deadline display
When a deadline has a precise time and timezone:
Ekho may show:
```text
11:59 PM ET
5:59 AM your time
```
where this helps prevent mistakes.
But the authoritative source-local deadline must remain identifiable.
---
# 44. DST
Never implement custom:
```text
if summer → +1
```
timezone logic.
Use IANA timezone data/runtime date-time APIs.
IANA maintains timezone transitions including daylight-saving changes.
---
# 45. UTC timestamps
Internal event timestamps such as:
```text
created_at
updated_at
sent_at
published_at
```
should remain absolute timestamps according to Data Standard.
Localized formatting occurs at presentation boundary.
---
# 46. User-facing timezone ambiguity
Never display:
```text
Deadline: 11:59 PM
```
when timezone materially changes meaning and the official source provided one.
Display the timezone or clear local conversion.
---
# 47. Number formatting
Store numeric values as numbers.
Display using locale-aware formatting.
Examples may differ:
```text
1,234.50
1 234,50
1.234,50
```
ECMA-402 `Intl.NumberFormat` exists specifically for locale-sensitive numerical formatting.
---
# 48. Never parse display strings as canonical numbers
Do not store:
```text
"1,250.00"
```
as the source of truth for a numeric value.
Store:
```text
1250
```
plus appropriate semantic metadata.
Formatting is presentation.
---
# 49. Currency identifiers
Machine currency representation follows the Data Standard:
```text
USD
EUR
GBP
CHF
```
Do not use currency symbols as identifiers.
---
# 50. Currency formatting
Presentation should use locale-aware formatting.
Conceptually:
```text
Intl.NumberFormat(locale, {
  style: "currency",
  currency: "USD"
})
```
ECMA-402 standardizes locale-sensitive currency formatting through `Intl.NumberFormat`.
---
# 51. Locale does not change currency
Changing:
```text
en-US → de-DE
```
must **not** turn:
```text
USD 80,000
```
into EUR.
Locale controls display conventions.
Currency conversion is a separate financial operation.
---
# 52. Currency conversion
If Ekho later shows:
```text
≈ €72,400
```
this must be treated as a derived estimate with:
* FX source;
* rate timestamp;
* original currency/value;
* clear approximation.
Never modify the canonical university tuition value.
---
# 53. Thousands/decimal separator
Never implement:
```text
replace(".", ",")
```
manually.
Use locale-aware APIs.
Formatting conventions are part of CLDR/ECMA-402 locale data.
---
# 54. Percentages
Store canonical numeric meaning.
Format percentages according to locale.
Do not store:
```text
"15%"
```
where the machine value is needed.
---
# 55. Sorting
Do not sort human-readable names using raw byte/codepoint comparison.
Use locale-aware collation where language-sensitive ordering matters.
ECMA-402 provides `Intl.Collator` for language-sensitive string comparison.
---
# 56. Search vs sorting
Search normalization and display sorting are separate concerns.
Search may use:
* normalized aliases;
* accent-insensitive matching where appropriate;
* transliteration-derived search helpers.
Display must preserve canonical text.
---
# 57. Institution names
Canonical university entity should preserve its official institutional name.
Examples:
```text
Università Commerciale Luigi Bocconi
École Polytechnique
Technische Universität München
```
Do not forcibly translate every proper name into English.
---
# 58. Localized institution names
Where an institution itself publishes an official English/localized name, Ekho may store it as an alternate localized name.
Conceptually:
```text
official_name
localized_names[]
```
Each localized natural-language value should carry language metadata.
W3C recommends language metadata at string level for localizable values.
---
# 59. AI-translated university names
Never publish an AI-created translation as if it were the university's official name.
Machine-generated translations may be:
```text
derived search alias
```
or separately marked generated content.
They are not authoritative identity data.
---
# 60. Program names
Preserve official program name.
Optional localized display name may exist separately.
Do not rewrite:
```text
Laurea Magistrale...
```
into an English canonical name solely for convenience.
---
# 61. Qualification names
Preserve nationally meaningful qualification terminology where necessary.
Examples:
```text
Abitur
Maturità
Baccalauréat
A-level
Attestat
```
Localized explanation can be separate.
Do not destroy admissions meaning through aggressive translation.
---
# 62. Source-language metadata
Sources may have:
```text
language
```
using BCP 47.
Example:
```text
language: "it"
```
This makes it possible to distinguish original content from localized interpretation.
---
# 63. Facts are language-neutral where possible
Prefer structured canonical fact:
```text
test = IELTS Academic
minimum_score = 7
```
over storing:
```text
"You need an IELTS score of at least seven."
```
Structured data dramatically reduces localization duplication.
---
# 64. Explanation vs fact
Separate:
```text
canonical fact
```
from:
```text
localized explanation
```
Example:
```text
minimum_score = 7.0
```
then UI generates locale-appropriate explanation.
Do not duplicate the number inside translated prose unless needed.
---
# 65. Critical translations
Translation must never alter the underlying meaning of:
* deadlines;
* minimum scores;
* qualification requirements;
* eligibility;
* tuition;
* financial aid;
* application round.
The structured canonical value remains authoritative.
---
# 66. Machine translation
Machine translation may eventually assist translation workflows.
It must not independently overwrite canonical admissions information.
Workflow:
```text
source fact
→ structured canonical data
→ translated explanation
```
not:
```text
official page
→ machine translation
→ treat translation as new source
```
---
# 67. Translation provenance
For important localized editorial content, retain where useful:
```text
source_language
target_language
translation_status
updated_at
```
Possible status:
```text
draft
machine_generated
reviewed
approved
```
---
# 68. Translation status
`machine_generated` must not imply:
```text
reviewed
```
Do not blur the two states.
---
# 69. Translation workflow
When multiple languages launch:
```text
source message
→ translation
→ automated checks
→ linguistic review
→ publish
```
For small low-risk UI strings, review may be lightweight.
Critical admissions explanations deserve stronger review.
---
# 70. Translation source language
Ekho UI's canonical source language:
```text
English
```
unless explicitly changed later.
Message identifiers remain stable independently of English wording.
---
# 71. Translation glossary
Maintain one canonical glossary for admissions terminology.
Examples:
```text
application
applicant
deadline
requirement
financial aid
scholarship
early action
early decision
regular decision
rolling admission
conditional offer
```
This prevents translators from using inconsistent terms across screens.
---
# 72. Translator context
Messages with ambiguous meaning should include developer/translator context.
Example:
```text
"Round"
```
could mean:
```text
application round
numeric rounding
circular object
```
Do not make translators guess.
---
# 73. No text embedded in images
Important UI information must not be baked into raster graphics merely because translating it is inconvenient.
Text should remain actual localizable content whenever possible.
---
# 74. Text expansion
Layouts must tolerate longer translations.
Do not assume:
```text
English label width × 1.0
```
Design components to survive significant expansion without:
* clipping;
* overlap;
* tiny fonts;
* horizontal overflow.
This is an Ekho localization QA requirement.
---
# 75. Fixed-width buttons
Avoid fixed button widths dependent on English text.
Prefer content-based sizing within sensible layout constraints.
---
# 76. Truncation
Never truncate critical meaning without an accessible/full alternative.
Especially:
* requirements;
* deadlines;
* application statuses;
* university/program names.
---
# 77. RTL readiness
Ekho architecture must be capable of supporting right-to-left interfaces such as Arabic or Hebrew without rebuilding components.
W3C recommends setting the document direction with `dir="rtl"` for RTL pages and designing styles around logical direction.
---
# 78. Root direction
When locale is RTL:
```html
<html lang="ar" dir="rtl">
```
Direction should be declared structurally rather than implemented by manually reversing every element.
---
# 79. CSS logical properties
Prefer:
```text
margin-inline-start
padding-inline-end
inset-inline-start
text-align: start
```
where appropriate.
Avoid building core layout around:
```text
margin-left
right
text-align: left
```
when the meaning is actually logical start/end.
CSS Logical Properties exists specifically to make layout relative to writing direction rather than fixed physical sides.
---
# 80. Do not mirror everything
RTL does not mean:
```text
transform: scaleX(-1)
```
on the application.
Text and direction-sensitive layout change.
Some visual elements remain physically oriented.
Handle component semantics intentionally.
---
# 81. Direction-sensitive icons
Review icons such as:
```text
Back
Forward
Undo/Redo
Next
Breadcrumb arrows
```
for RTL mirroring.
Icons whose meaning is not directional must remain unchanged.
---
# 82. Bidirectional user content
University names, URLs, numbers and Latin abbreviations may appear inside RTL text.
This creates bidirectional text cases.
W3C documents that mixed RTL/LTR strings require explicit direction handling in some contexts because the Unicode Bidirectional Algorithm alone cannot always determine the intended surrounding punctuation/order.
---
# 83. Unknown-direction strings
For unknown external/user-supplied short strings, use appropriate isolation/direction metadata instead of injecting directional control characters into stored content.
W3C recommends keeping language/direction metadata separate from string content for interchange rather than encoding it invisibly into the text itself.
---
# 84. Accessibility + language
Correct page language metadata also supports language-aware assistive processing.
Ekho accessibility and localization standards must not conflict.
W3C accessibility techniques use the HTML `lang` attribute to identify a document's language.
---
# 85. Person names
Never assume every person has:
```text
first name
last name
```
in Western order.
CLDR's person-name specification explicitly accounts for substantial variation in the way names are structured and displayed across languages.
---
# 86. Ekho profile name
For ordinary consumer UI prefer:
```text
display_name
```
where a structured legal name is not required.
Do not require first/last-name decomposition merely to personalize UI.
---
# 87. Legal applicant names
If future application automation genuinely requires legal-name components:
design a separate model based on actual destination/application requirements.
Do not reuse casual:
```text
display_name
```
as a guaranteed legal identity field.
---
# 88. Name ordering
Never globally generate:
```text
given_name + " " + family_name
```
as the only display method.
Use locale-aware/person-name logic where structured names are displayed.
CLDR provides locale-aware person-name formatting models for this purpose.
---
# 89. Addresses
If Ekho later stores applicant addresses:
do not globally require:
```text
State
ZIP code
```
because address structures vary internationally.
Address modelling must receive a separate global-address specification before implementation.
---
# 90. Country display names
Canonical value:
```text
IT
```
Display:
```text
Italy
Italia
Italie
Italien
```
according to locale.
Use standard locale data / `Intl.DisplayNames` where supported rather than manually maintaining translated country lists. ECMA-402 provides locale-sensitive display names, while CLDR supplies underlying locale data.
---
# 91. Country codes remain stable
Changing UI language must never alter canonical country identity.
```text
country_code = IT
```
stays `IT`.
Only display label changes.
---
# 92. Timezone display names
Store:
```text
America/New_York
```
Display a localized readable timezone label where helpful.
Do not store abbreviations such as:
```text
EST
CST
IST
```
as canonical timezone identifiers.
IANA uses stable zone identifiers and notes that timezone abbreviations are not unique legal identifiers.
---
# 93. Calendar conventions
Locale may affect:
* first day of week;
* date order;
* hour-cycle preference.
CLDR includes locale-specific week and time-format preference data.
Calendar components should not hard-code US conventions.
---
# 94. Week start
Application calendar may follow user's locale preference by default.
Potential later explicit user override:
```text
week_start
```
Do not ask during onboarding.
---
# 95. 12/24-hour clock
Use locale conventions by default.
Allow explicit preference later only if users actually need it.
Do not infer admissions meaning from the user's hour-cycle preference.
---
# 96. Locale-aware input
Display formatting and data-entry parsing must be designed carefully.
Where possible, use structured controls:
```text
date picker
numeric input
currency-aware display
```
instead of asking users to type ambiguous formatted values.
---
# 97. Date input
Internally preserve machine date:
```text
YYYY-MM-DD
```
UI may display a localized representation.
Do not depend on a free-text parser guessing whether:
```text
03/04/2027
```
means March 4 or April 3.
---
# 98. Numeric input
Where decimal values are user-entered:
parser must understand the UI locale or provide an unambiguous control.
Never silently interpret:
```text
3,5
```
as `35`.
---
# 99. Database collation
Database identity/uniqueness rules must not be casually changed based on UI locale.
Locale-sensitive presentation sorting belongs at appropriate search/application boundaries.
Canonical identifiers remain locale-neutral.
---
# 100. Slugs
Public canonical entity identity must not depend solely on translated display names.
Example:
```text
/universities/mit
```
should not become an entirely unrelated entity because the UI language changed.
Locale routing and canonical university identity are separate.
---
# 101. Localized public URLs
When Ekho launches multiple indexable public languages, use explicit stable locale variants.
Conceptually:
```text
/en/universities/mit
/de/universities/mit
/es/universities/mit
```
Exact routing implementation may be decided during frontend architecture.
Do not return substantially different indexable languages from one opaque URL solely via `Accept-Language`.
Google warns that locale-adaptive URLs can prevent all language variants from being crawled/indexed reliably.
---
# 102. `hreflang`
When multiple localized public versions exist, declare relationships between them using:
```text
hreflang
```
according to Search specification.
Google recommends `hreflang` to identify localized alternatives of the same page.
---
# 103. Self-reference
Each localized page's `hreflang` set must include itself and its corresponding alternates according to Google's published rules.
Do not generate incomplete one-way locale mappings.
---
# 104. `x-default`
For appropriate global selectors/fallback public pages, consider:
```text
hreflang="x-default"
```
Google supports `x-default` for language/region-neutral alternatives.
---
# 105. Canonical URLs
Localized pages should normally canonicalize to the corresponding page in the same language rather than collapsing all languages into English.
Google recommends that `hreflang` pages use a canonical page in the same language when possible.
---
# 106. SEO translation quality
Do not launch:
```text
English university page
+
translated navbar
+
English body
```
as a supposedly localized SEO page.
A locale URL must provide meaningful localized value.
---
# 107. Public vs authenticated localization
Public localized pages need:
* stable URLs;
* SEO metadata;
* `hreflang`;
* correct page language.
Authenticated workspace localization does not require duplicate SEO URLs.
Keep the concerns separate.
---
# 108. Metadata localization
When a localized public page exists, localize where appropriate:
```text
<title>
meta description
Open Graph textual metadata
structured explanatory content
```
Canonical factual university identifiers remain stable.
---
# 109. Email localization
Authentication and notification emails should eventually use the user's saved locale.
Fallback:
```text
user locale unavailable
→ English
```
Do not infer language from nationality.
---
# 110. Email links
A localized email should send the user into the corresponding localized application experience when practical.
Preserve safe `next`/destination semantics from Auth specification.
---
# 111. Notifications
Notification content must use the locale at **render/send time**, not store only a permanently rendered English sentence.
Prefer event:
```text
deadline_changed
```
plus structured parameters.
Then localize presentation.
---
# 112. Canonical notification events
Store:
```text
event type
data parameters
```
rather than only:
```text
"Your deadline changed."
```
This allows:
* later language switching;
* email localization;
* push localization;
* accessibility.
---
# 113. API localization
Machine API fields remain locale-neutral.
Example:
```json
{
  "status": "missing"
}
```
not:
```json
{
  "status": "Fehlend"
}
```
Localization belongs to UI/message layer.
---
# 114. Error codes
API returns stable machine errors:
```text
AUTH_REQUIRED
INVALID_DEADLINE
SOURCE_CONFLICT
```
Human-readable localized messages may be attached at presentation boundary.
Detailed contract belongs to API & Error Contract.
---
# 115. Database enum values
Canonical enums remain language-neutral:
```text
eligible
missing
optional
unknown
```
Never store localized values as database enum identities.
---
# 116. Admin localization
Ekho Admin may remain English-only initially.
Institutional/source data remains multilingual.
Do not delay global consumer functionality because internal admin UI is not translated.
---
# 117. Imported multilingual text
Import JSON may contain natural-language fields.
Where a field can legitimately have multiple language variants, include explicit language metadata according to Data Standard/schema.
Never guess language based solely on text characters when provenance already knows the source language.
W3C recommends explicit language/direction metadata rather than heuristic inference for strings where this information matters.
---
# 118. Source original text
A source summary/excerpt used internally should retain:
```text
original language
```
if known.
Translation is a separate representation.
---
# 119. Search aliases
Search may include:
```text
official names
official translations
common verified aliases
normalized search forms
```
Derived transliterations may be added later.
Never display a derived transliteration as the official institutional name without evidence.
---
# 120. Transliteration
Transliteration is:
```text
script conversion
```
not translation.
Keep it separate.
Example:
```text
Москва → Moskva
```
is not equivalent to translating institutional meaning.
---
# 121. Fallback policy — UI messages
Initial fallback:
```text
requested locale
→ supported language fallback
→ en
```
Example:
```text
en-GB
→ en
```
Fallback must be deterministic.
W3C defines locale fallback as a deterministic search from more-specific resources toward suitable more-general resources.
---
# 122. Do not reuse CLDR inheritance as application language negotiation blindly
CLDR's own documentation distinguishes locale-data inheritance from language matching; they are related but serve different purposes.
Ekho's available-language negotiation must be explicit.
---
# 123. Content fallback
If a localized explanation is unavailable:
acceptable:
```text
display English explanation
```
with correct language metadata.
Not acceptable:
```text
machine invent translation silently
```
---
# 124. Facts do not disappear because translation is missing
If German translation is absent but verified structured requirement exists:
show the factual requirement.
Do not hide important admissions data solely because localized editorial copy is unavailable.
---
# 125. Partial translation
A screen should never accidentally mix languages because a developer forgot to add localization keys.
Translation tests must detect missing application messages.
Intentional fallback content is different and should retain language metadata.
---
# 126. Missing translation behavior
Production:
```text
missing optional translation
→ deterministic fallback
→ observability event
```
Never:
```text
MISSING_application_save_button
```
shown to users.
---
# 127. Missing English source message
If canonical source-language UI message itself is missing:
treat as implementation error.
Do not silently render an empty button.
---
# 128. Localization configuration
Central registry:
```text
supportedLocales
defaultLocale
rtlLocales / direction resolver
localeFallback
```
Do not duplicate supported-language arrays across components.
---
# 129. Locale config is runtime/domain configuration
A locale should not become "supported" merely because one translation JSON file exists.
Enable locale only after:
* UI coverage;
* formatting QA;
* core workflows tested;
* fallback behavior tested;
* metadata/SEO configured where public;
* RTL verified where applicable.
---
# 130. Language rollout
Adding language:
```text
translation ready
→ automated tests
→ linguistic QA
→ staging
→ limited rollout
→ production
```
Do not publish an entire language because AI generated translations in one pass.
---
# 131. Translation completeness
Track:
```text
total messages
translated
missing
reviewed
```
Do not use translation completion percentage as a proxy for translation quality.
---
# 132. Critical workflow QA
Every supported language must pass:
```text
signup/login
university search
university page
application creation
requirements
deadlines
financial aid
settings
account deletion
```
before full rollout.
---
# 133. Pseudolocalization
Before multiple real languages, maintain test modes capable of exposing:
* hard-coded strings;
* text expansion;
* clipping;
* RTL assumptions;
* concatenated sentences.
This is an Ekho QA requirement.
---
# 134. RTL test locale
Maintain an internal pseudo-RTL test mode before an actual RTL-language launch.
It should reveal:
```text
left/right assumptions
unmirrored navigation
incorrect text alignment
overflow
direction bugs
```
---
# 135. Locale QA matrix
At minimum test representative locales such as:
```text
en-US
en-GB
de-DE
fr-FR
es-ES
tr-TR
ar
he
zh-Hans
ja-JP
hi-IN
```
These are test coverage choices, not necessarily launch languages.
---
# 136. Why representative locales
The objective is to expose differences in:
```text
text length
scripts
RTL
date formatting
number formatting
case behavior
font coverage
```
not to claim full localization support for those locales.
---
# 137. Font coverage
Selected product fonts must be tested against every supported script.
If the primary brand font lacks appropriate glyph coverage:
use a deliberate fallback stack.
Do not accept broken tofu/square glyphs in production.
---
# 138. Font fallback
CSS font stacks must preserve:
* legibility;
* language support;
* hierarchy;
* approximate visual harmony.
Do not force one Latin-only font across Arabic, CJK or Devanagari.
---
# 139. Do not fake letter spacing
Typography rules must be script-aware.
A letter-spacing value that looks good for uppercase Latin text must not be globally applied to all writing systems.
---
# 140. Line breaking
Do not implement line breaks by inserting manual `<br>` into translated sentences unless semantics genuinely require it.
Different scripts/languages have different line-breaking behavior, which CSS text processing standards account for.
---
# 141. Search result highlighting
Highlighting must respect Unicode text boundaries.
Do not slice strings by assumptions such as:
```text
1 byte = 1 character
```
Unicode text may represent user-perceived characters using multiple code points/code units.
Use Unicode-aware APIs.
---
# 142. Character limits
For text fields where a product limit is necessary:
define whether the limit refers to:
```text
bytes
code points
grapheme clusters
```
Do not accidentally count UTF-16 code units as "characters" in UI.
---
# 143. User-visible character limits
For fields such as notes or future essays, user-visible limits should generally follow user-perceived characters/graphemes where practical.
Do not let emoji or combining characters create obviously incorrect counters.
---
# 144. Case conversion
Never implement locale-sensitive casing with manual ASCII-only transformations.
Where presentation needs locale casing, use locale-aware mechanisms.
ECMA-402 standardizes locale-aware case conversion behavior through ECMAScript internationalization integration.
---
# 145. IDs remain case rules independent
Locale-sensitive casing must never influence:
* UUIDs;
* API keys;
* enum identities;
* database relation IDs;
* source hashes.
Machine identity remains locale-neutral.
---
# 146. University source URLs
Never localize or translate URL strings supplied by official sources.
A URL is an identifier/reference.
Display may shorten it visually, but target remains exact.
---
# 147. Official PDFs
Multilingual source PDFs may remain in original language.
Ekho may provide localized structured interpretation while linking the original source.
Never pretend a translated Ekho explanation is the original university document.
---
# 148. Legal pages
When Ekho publishes Terms/Privacy in multiple languages:
each version requires explicit locale/version metadata.
Legal equivalence/controlling-language policy belongs to Legal & Compliance Operations.
---
# 149. Analytics
Analytics event names remain locale-neutral.
Good:
```text
application_added
```
Bad:
```text
bewerbung_hinzugefügt
```
Locale may be included as a bounded analytical property where genuinely useful.
---
# 150. Observability
Logs/error codes remain machine-readable and locale-neutral.
User-facing error presentation is localized separately.
Do not translate internal operational error identifiers.
---
# 151. Cache keys
Any response whose rendered content depends on locale must include locale in the cache variation strategy.
Conceptually:
```text
page + locale
```
must not share one rendered-language cache entry.
---
# 152. Authenticated cache
Authenticated personalized caching remains governed by Security/Observability rules.
Localization must never create cross-user cache leakage.
---
# 153. Public localized cache
Localized public pages can be cached independently by locale.
Never return:
```text
German URL
→ cached English body
```
because locale was omitted from cache identity.
---
# 154. Locale in database entities
Do not add:
```text
locale
```
to every canonical table automatically.
Only natural-language localized fields need locale/language metadata.
Structured universal values do not.
---
# 155. Translation table architecture
When one entity has translated content, use a deliberate localized representation.
Conceptually:
```text
university
university_localizations
```
rather than:
```text
name_en
name_de
name_es
name_fr
name_it
...
```
Exact schema must follow Data Architecture.
---
# 156. No language columns
Prohibited scalable schema:
```text
description_en
description_fr
description_de
description_es
```
Adding a language must not require a database migration across every entity.
---
# 157. Localized record identity
A localized representation should conceptually reference:
```text
entity_id
locale/language
field/value
translation status
```
Exact normalized layout depends on field type and Data Architecture.
---
# 158. Uniqueness
Where localized entities exist:
enforce appropriate uniqueness such as:
```text
entity + locale
```
rather than allowing accidental duplicate translations.
---
# 159. Source-grounded localization
Changing UI language must never change:
```text
which university requirement applies to user
```
Localization affects representation.
Requirements Engine controls eligibility/logic.
---
# 160. No geography-based hidden policy
Never show different eligibility facts merely because the request originates from an IP in a certain country.
Applicant-specific requirements come from explicit applicant profile + canonical policy scope.
---
# 161. Regional product differences
If Ekho someday genuinely has region-specific product/legal behavior:
model it explicitly.
Do not bury business logic inside:
```text
if locale === "de-DE"
```
Locale is not a substitute for jurisdiction.
---
# 162. Jurisdiction is separate
Legal jurisdiction may depend on:
* residence;
* service entity;
* age;
* applicable law;
* product market.
It must not be inferred solely from UI language.
---
# 163. Feature availability
Feature availability by country belongs to Feature Flags/Runtime Configuration.
Do not use translation availability as a feature entitlement system.
---
# 164. Failure behavior
If locale service/resource loading fails:
```text
fallback safely to English
```
where possible.
A missing translation must not take down Ekho.
---
# 165. Critical formatting failure
If a deadline cannot be formatted safely:
prefer canonical understandable representation:
```text
2027-01-05
```
over hiding the deadline entirely.
Trust > visual perfection.
---
# 166. Intl dependency failure
Do not implement a fragile remote translation/formatting API for basic formatting that can be performed locally through standard runtime internationalization APIs.
Core:
```text
numbers
dates
currencies
plural rules
```
should not require external network requests.
ECMA-402 provides these capabilities directly within conforming JavaScript runtimes.
---
# 167. Supported locale test
For every configured supported locale:
verify runtime actually supports required Intl operations.
Do not assume a locale exists merely because it is present in configuration.
---
# 168. Dependency updates
Unicode/CLDR/timezone behavior evolves over time.
Keep runtime/dependency upgrades within Development Workflow so locale/timezone data is not indefinitely stale. IANA explicitly updates tzdb as civil-time rules change, while CLDR is continuously updated for locale data.
---
# 169. Internationalization code ownership
Create a small shared layer rather than allowing every component to decide:
```text
how dates format
how currencies format
how locales fallback
how languages resolve
```
Conceptual modules:
```text
i18n/config
i18n/messages
i18n/format
i18n/locale
i18n/direction
```
Exact repository paths may follow project conventions.
---
# 170. Formatting utilities
Shared helpers may wrap standard APIs for:
```text
formatDate
formatDateTime
formatMoney
formatNumber
formatRelativeTime
formatCountry
```
They must take locale explicitly or from trusted request/user context.
---
# 171. No `toLocaleString()` without intent
Do not scatter:
```text
value.toLocaleString()
```
without explicit locale/options across the codebase.
Output should be reproducible based on Ekho's resolved locale.
---
# 172. Server/client consistency
Server and browser must resolve the same locale for a rendered route.
Avoid:
```text
server renders English
→ hydration
→ browser switches to German
```
causing layout/content mismatch.
---
# 173. Hydration
Locale must be known sufficiently early in SSR/route processing for localized server-rendered pages.
Do not depend on late client-side locale detection for initial public render.
---
# 174. Critical date test matrix
For every deadline component test:
```text
date only
date + timezone
DST transition
leap year
year boundary
user timezone behind source
user timezone ahead of source
```
---
# 175. Currency test matrix
Test:
```text
USD
EUR
GBP
CHF
JPY
```
including zero-decimal conventions where runtime formatting requires them.
Formatting must come from locale/currency data rather than hard-coded decimal assumptions.
---
# 176. RTL tests
* [ ] Root `dir` changes correctly.
* [ ] Navigation direction works.
* [ ] Breadcrumbs work.
* [ ] Forms align correctly.
* [ ] Icons mirror only when appropriate.
* [ ] Modals work.
* [ ] University cards work.
* [ ] Requirement rows work.
* [ ] Deadline timeline works.
* [ ] Mixed Latin/Arabic text works.
* [ ] URLs/numbers remain readable.
---
# 177. Translation tests
* [ ] No hard-coded user-visible strings in localized surfaces.
* [ ] Missing key falls back safely.
* [ ] Variable interpolation works.
* [ ] Pluralization works.
* [ ] Translation keys are stable.
* [ ] No fragmented sentence assembly.
* [ ] Critical structured values remain unchanged across locales.
---
# 178. Locale resolution tests
* [ ] First visit may use browser language.
* [ ] Explicit user selection overrides it.
* [ ] Preference persists.
* [ ] Unsupported locale falls back.
* [ ] User can manually switch.
* [ ] Nationality does not change UI language.
* [ ] Country does not overwrite locale.
* [ ] IP does not permanently determine language.
---
# 179. Date/time tests
* [ ] Date-only deadline remains same calendar date.
* [ ] Exact timestamp converts correctly.
* [ ] Source timezone retained.
* [ ] User timezone conversion works.
* [ ] DST handled by timezone library/runtime.
* [ ] No invented timezone exists.
* [ ] Critical date remains visible if localization fails.
---
# 180. Number tests
* [ ] Decimal separator localized.
* [ ] Thousands separator localized.
* [ ] Percent formatting localized.
* [ ] Canonical numeric value unchanged.
* [ ] Numeric input parser does not misinterpret locale values.
* [ ] Display strings are never reused as stored numeric truth.
---
# 181. Currency tests
* [ ] Currency code remains canonical.
* [ ] Symbol/display localizes.
* [ ] Locale change does not convert currency.
* [ ] Any FX conversion is clearly derived.
* [ ] Original university value remains visible/available.
---
# 182. Institution tests
* [ ] Original university name preserved.
* [ ] Official localized name supported.
* [ ] AI translation never becomes official automatically.
* [ ] Language metadata survives import.
* [ ] Search aliases do not modify canonical display name.
* [ ] Program official name preserved.
---
# 183. SEO tests
When multilingual public pages launch:
* [ ] Locale has stable URL.
* [ ] correct `<html lang>`.
* [ ] correct canonical.
* [ ] correct reciprocal `hreflang`.
* [ ] sitemap/alternate relationships valid.
* [ ] pages are not forcibly redirected away.
* [ ] translated page contains meaningful localized content.
* [ ] local metadata rendered.
Google requires reciprocal `hreflang` relationships for alternate pages to be honored and recommends explicit localized URLs rather than depending only on locale-adaptive responses.
---
# 184. Performance
Localization must not ship every translation for every language to every browser.
Load only the locale/messages necessary for the current experience.
Do not turn:
```text
10 supported languages
```
into:
```text
10× UI message payload
```
for every request.
---
# 185. Static vs dynamic translations
Stable UI strings can be packaged/versioned with application code.
Dynamic institutional information remains database-backed.
Do not place university admissions facts inside translation files.
---
# 186. Translation cache invalidation
Localized database content must invalidate only relevant locale/entity representations when possible.
Do not purge the entire global site because one German university description changed.
---
# 187. Data freshness across translations
Updating a canonical critical fact:
```text
IELTS 6.5 → 7.0
```
must update every localized presentation automatically because the number is structured.
Do not maintain separate translated copies of critical numeric truth.
---
# 188. Translation freshness
Localized editorial explanation may have:
```text
translation_updated_at
```
or equivalent.
If canonical explanation changes materially, affected translations may become:
```text
needs_review
```
rather than silently pretending they still match.
---
# 189. Translation versioning
Translation history does not need the same complexity as admissions-source provenance for every button.
But important localized admissions explanations should remain traceable when materially changed.
---
# 190. Analytics for localization
Useful metrics later:
```text
locale usage
language switch rate
missing_translation_count
fallback_usage
localized_page traffic
```
Do not collect more personal information merely for localization analytics.
---
# 191. Success metrics
Internationalization is successful when:
* adding a language does not require product architecture rewrite;
* formatted values are correct;
* critical admissions meaning is stable across languages;
* RTL can be enabled without rebuilding screens;
* user preference overrides detection;
* multilingual institutional data retains provenance;
* public localized pages are independently discoverable.
---
# 192. Explicit v1 exclusions
Do not build yet unless justified:
* automatic translation of the whole university database;
* automatic language detection for every text field;
* dozens of UI languages;
* separate regional Ekho products;
* automatic IP-based country switching;
* live FX conversion everywhere;
* manual currency-format tables;
* custom timezone database;
* translation social/community system;
* custom Unicode library;
* per-country forks of the frontend.
---
# 193. P0 failures
Any of these blocks multilingual production rollout:
* wrong deadline caused by timezone conversion;
* date-only deadline shifts to another date;
* locale changes canonical financial value;
* UI language changes admissions eligibility;
* RTL makes critical controls unusable;
* user cannot override automatic language choice;
* country/nationality is overwritten from locale;
* official source meaning changes through translation;
* untranslated critical fact disappears;
* localized cache serves wrong language;
* one locale can corrupt canonical university data;
* language variant URLs become inaccessible to users/search engines;
* translated university name replaces verified canonical identity without evidence.
---
# 194. Implementation order for Codex
## Stage 1 — Foundation
1. BCP 47 locale model.
2. central locale configuration.
3. locale resolver.
4. saved user preference.
5. UTF-8/Unicode validation.
6. direction resolver.
## Stage 2 — Formatting
7. shared `Intl` wrappers.
8. date formatting.
9. date/time + timezone formatting.
10. money formatting.
11. numbers.
12. relative time.
13. country/display names.
14. locale-aware collation.
## Stage 3 — Messages
15. message registry.
16. locale dictionaries/resources.
17. interpolation.
18. plural/select support.
19. deterministic fallback.
20. missing-message handling.
## Stage 4 — Layout
21. root `lang`.
22. root `dir`.
23. CSS logical properties.
24. directional icons.
25. text expansion.
26. mixed-direction strings.
27. font fallbacks.
## Stage 5 — Institutional data
28. source language metadata.
29. localized institution/program names.
30. localized explanations.
31. translation statuses.
32. canonical fact separation.
33. multilingual search aliases.
## Stage 6 — Public multilingual web
34. explicit localized routes.
35. canonical metadata.
36. `hreflang`.
37. localized SEO metadata.
38. sitemap integration.
## Stage 7 — QA
39. pseudolocalization.
40. pseudo-RTL.
41. representative-locale matrix.
42. timezone tests.
43. currency tests.
44. SEO localization tests.
45. full critical workflow E2E tests.
Do not implement ten actual translations before Stage 1–7 infrastructure works correctly.
---
# 195. Codex implementation constraint
Before implementation read:
```text
Data Standard
Data Architecture
Requirements Engine
Auth & Account Lifecycle
Import & Ingestion
Admin & Data Operations
```
Do not redesign canonical values merely to make translation easier.
Specifically do not:
```text
translate enums in DB
infer nationality from locale
convert currency from locale
convert date-only values to timestamps
hard-code country-specific formatting
```
---
# 196. Definition of Done
Internationalization foundation is complete when:
* BCP 47 locale handling exists;
* explicit user preference works;
* browser language is only a hint;
* locale/country/nationality/timezone are separate;
* UI strings are localizable;
* sentence concatenation is eliminated;
* plural logic is locale-aware;
* dates are locale formatted;
* date-only deadlines remain date-only;
* IANA timezones are used;
* currency value and locale are separate;
* number formatting uses standard Intl behavior;
* UTF-8/Unicode works end-to-end;
* official institution names are preserved;
* language metadata can accompany natural-language institutional fields;
* `<html lang>` works;
* RTL architecture works;
* CSS logical layout is used where appropriate;
* deterministic English fallback exists;
* localized public URLs/SEO architecture is defined before multilingual SEO launch;
* canonical admissions logic produces identical factual outcomes regardless of UI language;
* all P0 tests pass.
---
# 197. Final invariant
Ekho must operate like:
```text
CANONICAL FACT
deadline = 2027-01-05
currency = USD
country = US
timezone = America/New_York
↓
USER CONTEXT
locale = de-DE
timezone = Europe/Berlin
↓
LOCALIZED PRESENTATION
5. Januar 2027
80.000 $
6:00 Uhr deiner Zeit
```
Never:
```text
change language
↓
change the underlying fact
```
Localization controls:
> **how information is understood.**
It must never control:
> **what the admissions truth is.**
---
# 198. Primary authority sources
This standard was checked primarily against:
1. **W3C Internationalization Best Practices for Spec Developers, current July 2026 version** — architecture, language/direction metadata, Unicode and internationalization requirements.
2. **W3C Language Tags and Locale Identifiers** — language, locale, BCP 47 and fallback principles.
3. **IETF BCP 47 / RFC 5646** — standardized language tags.
4. **Unicode CLDR / LDML** — locale data, numbers, dates, currencies, plurals, names and MessageFormat.
5. **Unicode UAX #15** — Unicode normalization.
6. **ECMA-402 Internationalization API, 2026 edition** — JavaScript locale-sensitive formatting APIs.
7. **IANA Time Zone Database** — canonical timezone identifiers and changing civil-time rules.
8. **W3C Working with Time and Timezones** — global date/time modelling guidance.
9. **W3C CSS Logical Properties / RTL guidance** — direction-aware layouts and RTL.
10. **Google Search Central international/multilingual guidance** — localized URLs, `hreflang`, canonical relationships and locale-adaptive crawling risks.
11. **Unicode MessageFormat 2 / CLDR 47** — structured multilingual messages and grammatical selection.
