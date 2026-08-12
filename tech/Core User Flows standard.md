# Ekho Core User Flows v1.0
**Status:** READY TO LOCK
**Date:** 2026-08-12
**Scope:** Applicant Experience — core path from first login to completing the next application action.
---
# 1. Purpose
Core User Flows define exactly how an applicant moves through Ekho.
Canonical flow:
```text
First login
→ Find university/program
→ Add application
→ Resolve minimum application context
→ See personalized requirements
→ Provide missing applicant context only when needed
→ Receive one Next Action
→ Complete action
→ Requirements recompute
→ Receive next Next Action
→ Return later and continue
```
The entire product must optimize this loop.
Ekho's core job:
> Tell me exactly what I need to do next for each university, and make sure I don't miss anything important.
Core flows must therefore optimize for:
1. minimum time before first useful result;
2. minimum number of required inputs;
3. correct personalized requirements;
4. clear deadlines;
5. one obvious next action;
6. no fake certainty;
7. immediate persistence;
8. no dead ends.
---
# 2. Evidence behind the flow
## 2.1 Application tracking is a real problem
Common App itself exposes college-specific deadlines, testing, writing and other requirements through College Search and My Colleges.
College Board explicitly recommends application checklists, calendars or spreadsheets to keep deadlines, documents and applications organized, and says college websites are the best place to confirm accurate deadline information.
Applicants repeatedly describe using large spreadsheets or custom Notion/Sheets systems to track applications, deadlines, requirements and progress. This appears across multiple years of r/ApplyingToCollege discussion rather than one isolated comment.
### Ekho decision
The product must replace the manual:
```text
University websites
+
spreadsheet
+
calendar
+
notes
+
application portals
```
management layer.
Ekho does **not** need to replace the actual submission platform.
---
# 3. Why requirements cannot be university-level only
UCAS states that entry requirements vary by course and provider and can include qualifications, subjects and grades.
Oxford separately publishes international qualification requirements and course-specific subject/test requirements.
Germany is even more context-dependent: DAAD states that the correct application route can depend on nationality/background, qualification and subject, with applications potentially going directly to a university, through uni-assist or another system.
UvA demonstrates another pattern: a student may first register through Studielink and then complete the actual application and personalized checklist inside the university's own SIS.
### Ekho decision
Requirements must never be modeled simply as:
```text
University → Requirements
```
The real model is approximately:
```text
Institution
+
Program/course where applicable
+
Degree level
+
Intake/cycle
+
Application route/round where applicable
+
Applicant context
→
Personalized requirements
```
---
# 4. Progressive profiling principle
Long forms increase cognitive load; Nielsen Norman Group recommends progressive disclosure and conditionally showing fields only when they become relevant.
### Ekho decision
**There is no mandatory long onboarding questionnaire.**
Do not ask for:
* nationality;
* qualification;
* GPA;
* SAT;
* ACT;
* IELTS;
* TOEFL;
* DET;
* graduation year;
* financial information;
* intended major;
* preferred countries;
before the data is actually required to answer something useful.
Rule:
```text
Need user field now?
    ↓
YES → ask
NO  → do not ask
```
Profile completion is never a product goal by itself.
---
# 5. Canonical navigation
Use the already-fixed Information Architecture.
## Primary navigation
1. `Home`
2. `Explore`
3. `Applications`
4. `Updates`
## Routes
```text
/app
```
Home.
```text
/universities
```
Explore.
```text
/universities/[institutionSlug]
```
University Intelligence.
```text
/universities/[institutionSlug]/programs/[programSlug]
```
Program Intelligence.
```text
/app/applications
```
My Applications.
```text
/app/applications/[applicationId]
```
Application Detail.
```text
/app/updates
```
Live Admissions Updates.
## Secondary routes
```text
/app/profile
/app/documents
/app/settings
```
These are not primary navigation destinations.
## Explicitly forbidden
Do not create primary navigation items for:
* Requirements;
* Tasks;
* Documents;
* Profile;
* Settings;
* AI Chat.
Requirements and Next Actions live within Home/Application workflows.
---
# 6. Core entity distinction
Codex must not confuse these entities.
## Institution
Example:
```text
University of Oxford
```
## Program
Example:
```text
Economics and Management
```
## Application
A user's intention/process of applying within a defined admissions context.
Conceptually:
```text
Applicant
+
Institution
+
Program where applicable
+
Degree level
+
Intake
+
Application route/round where applicable
```
## Requirement
Something the applicant must, may or is advised to satisfy according to an admissions rule.
## Action
A concrete thing the user can do.
Example:
Requirement:
```text
English proficiency required
```
Action:
```text
Add your IELTS score
```
These are not the same entity.
---
# 7. Application scope
Different admissions systems behave differently.
Application must support:
```text
application_scope = institution | program
```
The system must never assume all applications are program-level or all are institution-level.
Examples supporting this distinction exist across Common App, UCAS and European university systems.
## Institution-scoped application
Program may be:
```text
null
```
until required.
## Program-scoped application
Program must eventually be resolved before Ekho can confidently calculate program-specific requirements.
If it is not resolved:
```text
program = null
```
is permitted.
Dependent results become:
```text
Unknown
```
They must not be guessed.
---
# 8. First-login flow
After authentication:
```text
→ /app
```
Do not redirect a new user into:
* Profile;
* questionnaire;
* settings;
* tutorial;
* walkthrough carousel;
* AI chat;
* blank dashboard.
---
# 9. First-login Home state
When:
```text
applications_count = 0
```
Home becomes an activation screen.
Primary content:
```text
Add your first university
```
Supporting copy:
```text
Search a university or program to see what you need to apply.
```
Primary control:
```text
Search universities and programs…
```
Primary action:
```text
Search
```
Secondary content must remain minimal.
Do not display empty widgets such as:
* `0 applications`;
* `0 tasks`;
* `0 deadlines`;
* `0 documents`;
* `0% complete`.
Empty metrics provide no value.
---
# 10. Time-to-value target
Core principle:
**A competent new user should be able to reach the first meaningful application result in approximately 30 seconds.**
Meaningful result means at least one of:
* personalized requirement;
* confirmed application deadline;
* concrete Next Action;
* clear statement that more applicant context is required.
Account creation alone does not count.
Adding a university alone does not count.
---
# 11. Explore search flow
Entry points:
```text
/app → first-login search
```
or:
```text
/universities
```
or global search entry where supported.
---
# 12. Search input
Placeholder:
```text
Search universities and programs…
```
Search may match:
* canonical institution name;
* recognized institution alias;
* common abbreviation;
* program name;
* verified alternative name.
Examples of aliases must come from stored data.
Do not have an LLM invent aliases or institutions at query time.
---
# 13. Search states
Exactly support:
```text
idle
typing
loading
results
no_results
error
```
## Idle
No fake recommended results are required.
## Loading
Use lightweight skeleton/loading state.
Do not clear an existing query.
## Results
Return only entities existing in the Ekho dataset.
## No results
Copy:
```text
We couldn't find this university or program.
```
Actions:
```text
Try another search
Request university
```
## Error
Copy:
```text
Search couldn't load. Try again.
```
Action:
```text
Retry
```
Never render an empty page.
---
# 14. Search result hierarchy
Results may be grouped:
```text
Universities
Programs
```
Only show `Programs` if program-level data exists.
University result minimum information:
* institution name;
* city;
* country;
* logo/identifier when available.
Program result minimum information:
* program name;
* institution;
* degree level where known;
* location where relevant.
Avoid showing:
* acceptance chance;
* arbitrary ranking score;
* AI compatibility score;
* dozens of metadata fields.
Search is for finding, not comparing.
---
# 15. Search result actions
Each result can support:
Primary interaction:
```text
Open
```
Optional fast action:
```text
+ Add
```
`+ Add` is valid because it reduces steps for a user who already knows where they are applying.
Clicking the result itself opens Intelligence.
---
# 16. University Intelligence → Add flow
Route:
```text
/universities/[institutionSlug]
```
Primary CTA for an institution not yet added:
```text
Add to Applications
```
The page can contain University Intelligence, but adding the institution must remain easy to find.
Do not bury the CTA after a long page.
---
# 17. Program Intelligence → Add flow
Route:
```text
/universities/[institutionSlug]/programs/[programSlug]
```
CTA:
```text
Add to Applications
```
Application creation inherits the selected program automatically.
Never ask the user to select the same program again.
---
# 18. Add Application transaction
When `Add to Applications` is selected:
1. validate institution/program;
2. check for an existing matching application;
3. create or reuse application;
4. persist immediately;
5. resolve missing application context;
6. navigate into Application Detail.
Never require the applicant profile to be completed before saving the application.
---
# 19. Application persistence
The application must be persisted before optional profiling begins.
Reason:
The user must not lose the university because they:
* close the sheet;
* refresh;
* navigate away;
* lose connection;
* decide to answer later.
Minimum persisted identity:
```text
application_id
user_id
institution_id
program_id nullable
created_at
```
Additional context is stored as it becomes known.
---
# 20. Duplicate handling
Clicking `Add` twice must not silently create duplicate applications.
## Exact existing application
CTA becomes:
```text
View application
```
## Existing incomplete institution application
Example:
```text
Oxford
program = null
```
Later user adds:
```text
Oxford
Economics and Management
```
If the existing application can safely be specialized without conflicting user data:
```text
update existing incomplete application
```
rather than creating a duplicate.
## Legitimately different applications
Different applications can exist when the actual admissions structure allows them.
Duplicate logic must respect `application_scope`.
Never decide duplicates using institution name alone.
---
# 21. Application Setup
After the application exists, resolve only context required to determine the correct admissions rules.
Possible fields:
```text
degree level
program/course
intake
application route
application round
```
Not every application requires all fields.
---
# 22. Conditional Application Setup
Decision logic:
```text
Is degree level already known?
YES → skip
NO → ask if needed
Is application program-scoped?
NO → do not require program
YES → program unresolved → ask
Does this institution/intake have multiple relevant application routes?
NO → resolve automatically
YES → ask
Does this application use admission rounds?
NO → do not show round field
YES → ask if unresolved
```
Never display irrelevant controls disabled "just in case."
---
# 23. Unknown option
Every field whose answer a normal applicant may legitimately not know yet must support:
```text
Not sure yet
```
Selecting it must not block saving.
Instead:
```text
dependent output = Unknown
```
and Ekho can later make resolving it a Next Action.
---
# 24. Application Context vs Applicant Profile
Critical distinction.
## Application Context
Specific to one application:
* program;
* intake;
* application round;
* application route;
* potentially application-specific major choice.
## Applicant Profile
Reusable across applications:
* nationality;
* country of education;
* education system;
* qualification;
* graduation year;
* grades/GPA;
* standardized tests;
* English tests.
Never save application-specific information as global profile information.
Never repeatedly ask global profile information already known.
---
# 25. Application Detail
Route:
```text
/app/applications/[applicationId]
```
Default hierarchy:
```text
1. Application identity
2. Next Action
3. Critical deadline
4. Requirements for You
5. Progress summary
6. Secondary application information
7. Sources / verification details
```
The core question above the fold:
> What should I do next?
---
# 26. Application header
Minimum:
* institution;
* program if known;
* degree level if useful;
* intake;
* route/round where relevant;
* nearest critical deadline.
Do not duplicate large University Intelligence content inside Application Detail.
---
# 27. Requirements computation
Requirements must be generated from structured admissions data and applicant/application context.
Conceptual flow:
```text
Verified raw admissions facts
↓
Normalized requirement rules
↓
Application scope
↓
Applicant profile
↓
Rules evaluation
↓
Personalized requirement states
```
LLMs may help interpret/structure content upstream, but **must not invent runtime requirements for the user.**
---
# 28. Three separate concepts must never be mixed
## A. Source/data state
Examples from the Data Standard:
```text
known
unknown
not_published
not_found
not_applicable
not_offered
conflicting_sources
stale
```
## B. Verification state
Examples:
```text
verified_primary
verified_official_secondary
verified_external
unverified
conflict
stale
```
## C. Personalized requirement state
User-facing admissions result.
These are three different layers.
Do not overload one database field to represent all three.
---
# 29. Requirement user-facing states
Canonical UI states:
### Satisfied
Reliable data says the requirement applies and available applicant evidence indicates it is met.
### Missing
Reliable data says the requirement applies and Ekho has enough applicant information to determine it is currently unsatisfied.
### Action required
Reliable data says the requirement applies and a concrete action is currently required.
Examples:
```text
Request transcript
Register for admissions test
Add test score
Upload document
Complete external form
```
### Optional
The requirement/item is officially optional for the relevant scope.
Optional items do not block application readiness.
### Unknown
Ekho does not have enough reliable information to produce another state.
Possible causes:
* missing applicant information;
* unresolved program;
* unresolved round;
* insufficient source coverage;
* source not published;
* stale critical data;
* conflicting data that cannot safely be resolved.
### Not applicable
Ekho has reliable evidence that the requirement does not apply to this applicant/application.
---
# 30. `Conflict` is not a normal completion state
Do not create this UI:
```text
Transcript — Conflict
```
`Conflict` belongs to source/data verification.
The user-facing requirement should normally become:
```text
Unknown
```
with an explanation:
```text
We found conflicting official information for this requirement.
```
and access to the sources.
---
# 31. `Eligible` is not a generic requirement completion state
Do not use:
```text
SAT — Eligible
Transcript — Eligible
Essay — Eligible
```
Eligibility is a higher-level conclusion.
If Ekho later produces an eligibility verdict, it must be separate from individual requirement completion.
Never use:
```text
Eligible
```
as a synonym for:
```text
Satisfied
```
---
# 32. Requirement state precedence
To avoid contradictory states, evaluation should conceptually follow:
```text
1. Can Ekho reliably determine applicability?
   NO → Unknown
2. Does the requirement apply?
   NO → Not applicable
3. Is it officially optional?
   YES → Optional
4. Is there evidence it is satisfied?
   YES → Satisfied
5. Is there a concrete current action required?
   YES → Action required
6. Otherwise required + known unsatisfied
   → Missing
```
Never resolve uncertainty downward into `Missing`.
---
# 33. Absolute trust rule
Forbidden:
```text
absence of data = Missing
```
Correct:
```text
absence of sufficient evidence = Unknown
```
`Missing` requires evidence.
---
# 34. Requirements categories
Only render categories with relevant data.
Possible categories:
## Academic
* qualification;
* required subjects;
* grades;
* GPA where applicable.
## Language
* IELTS;
* TOEFL;
* DET;
* other accepted tests;
* exemptions.
## Admissions tests
* SAT;
* ACT;
* UCAT;
* LNAT;
* TMUA;
* institution/program-specific tests.
UCAS confirms admissions tests can be course-specific and may have registration deadlines before the university application deadline.
## Documents
* transcript;
* predicted grades;
* recommendation;
* school report;
* portfolio;
* CV where relevant.
## Writing
* essays;
* personal statement;
* supplements.
## Interview / audition
Only when relevant.
## Application
* application form;
* fee;
* platform;
* additional university portal.
## Financial aid
Only when aid workflow data has been verified:
* CSS Profile/forms;
* institutional forms;
* supporting financial documents;
* aid deadline.
Do not show empty categories.
---
# 35. Inline Progressive Profiling
When a personalized result is blocked by missing applicant context, do not send the user to Profile.
Insert a contextual prompt directly where it matters.
Example:
```text
We need one detail to check your academic requirements.
What qualification are you completing?
[IB Diploma]
[A Levels]
[Other]
[Not sure]
```
After answer:
```text
save globally if appropriate
→ recompute affected applications
→ update current requirements
```
No Save button is necessary for discrete selections if persistence succeeds immediately.
---
# 36. Question ordering
Ask the question that unlocks the largest amount of immediately relevant information.
Priority:
```text
1. field required to determine deadline/application route
2. field blocking multiple required requirements
3. field blocking a critical eligibility requirement
4. field blocking one normal requirement
5. nonessential profile enrichment
```
Priority 5 should usually not interrupt the core flow at all.
---
# 37. Never ask the same profile question twice
If:
```text
profile.qualification = IB Diploma
```
another application must reuse it.
If a requirement depends on a different qualification interpretation, explain why another answer is required instead of silently overwriting global profile data.
---
# 38. Profile change propagation
When a globally relevant profile field changes:
```text
nationality
qualification
education system
graduation year
test score
etc.
```
Ekho must determine affected applications and recompute them.
Process:
```text
Profile field changed
↓
Identify dependent rules
↓
Recompute affected requirements
↓
Recompute affected progress
↓
Recompute affected Next Actions
↓
Surface meaningful change if necessary
```
Old personalized conclusions must not remain silently cached.
---
# 39. Application context change propagation
Changing:
* program;
* intake;
* application route;
* round;
must trigger the same dependency recomputation for that application.
Do not delete historical user input merely because the context changed.
Items no longer relevant should leave the active requirement set but their user history should not be silently destroyed.
---
# 40. Next Action — central interaction
Every active application should attempt to produce exactly **one primary Next Action**.
Example:
```text
Next Action
Add your English test score
We need it to determine whether your English requirement is satisfied.
[Add score]
```
Other outstanding items remain accessible below.
Do not create six equally prominent actions.
---
# 41. Application-level Next Action hierarchy
Use priority tiers.
## P0 — Critical context blocker
Required application/applicant context is missing and prevents Ekho from determining important requirements.
Example:
```text
Choose your application round
```
## P1 — Imminent hard requirement
A verified required action with a known, approaching hard deadline.
## P2 — Long-lead required action
A required action that has an earlier action/registration/request deadline than the final application itself.
Examples include some admissions tests and recommendations. College Board and UCAS both advise applicants to handle tests/recommendations before final submission deadlines where necessary.
Do not hardcode arbitrary lead times without data.
## P3 — Required blocker
Known required action currently preventing readiness.
## P4 — Other required work
Normal remaining required work.
## P5 — Optional
Optional items must never displace incomplete required work as the primary action.
---
# 42. Next Action tie-breaking
Within the same priority:
```text
1. earliest verified action deadline
2. earliest associated application deadline
3. dependency blocking more requirements
4. stable deterministic ordering
```
Do not randomize Next Action.
The same unchanged application should not show a different action every refresh.
---
# 43. Global Next Action on Home
Returning users land on:
```text
/app
```
Home answers:
> What should I do next across all of my applications?
Ekho selects one **global Next Action** from active applications.
Card includes:
* action;
* university;
* program if needed;
* deadline if known;
* CTA.
Example:
```text
Next Action
Request your transcript
University of Amsterdam
Due before your application deadline
[Mark as requested]
```
Below this may appear:
```text
View all applications
```
Home should not become a giant dashboard.
---
# 44. Global prioritization
Global action selection uses:
```text
application-level Next Actions
↓
compare priority class
↓
compare verified deadline
↓
return one global action
```
An optional task at University A must not outrank an urgent required task at University B.
---
# 45. Completing an action
Action types can include:
```text
enter_value
upload_document
mark_requested
mark_completed
open_external
choose_context
answer_profile_question
```
Do not force every action into one generic checkbox.
---
# 46. Action completion transaction
After successful completion:
```text
1. persist user input/action
2. recompute affected requirement
3. recompute dependent requirements
4. recompute application progress
5. recompute application Next Action
6. recompute global Next Action
7. update UI
```
All must reflect the same final state.
---
# 47. Completion feedback
After action completion:
Do:
```text
Done
```
briefly.
Then reveal the new Next Action.
Do not:
* launch celebration modal;
* add confetti every time;
* give XP;
* create streak pressure;
* redirect to an unrelated screen.
The reward is visible progress.
---
# 48. External actions
Ekho is not automatically the submission system.
Possible external platforms include Common App, UCAS, Studielink, uni-assist and university-specific portals. Official admissions processes demonstrate that students may need to use several different platforms even within one application journey.
Example:
```text
Complete your application in UCAS
[Open UCAS]
```
Critical rule:
```text
open_external ≠ completed
```
Clicking an external link must **not** automatically mark the requirement complete.
Completion requires:
* explicit user confirmation; or
* future verified integration.
---
# 49. External completion language
If Ekho cannot independently verify external completion, use language such as:
```text
Mark as completed
```
Not:
```text
Submitted successfully
```
Ekho must never claim an external submission occurred simply because a link was opened.
---
# 50. Deadlines
Deadline handling must follow the canonical Data Standard.
Store enough structure to represent:
```text
date
time nullable
timezone nullable
deadline_type
hard_or_priority
scope
source
last_verified_at
```
Use:
* ISO 8601 for date/time;
* IANA time zones;
* original published value;
* normalized value separately.
---
# 51. Never invent deadline time
If official information states only:
```text
15 January
```
display:
```text
15 Jan 2027
```
Never convert this into:
```text
15 Jan 2027 · 11:59 PM
```
unless the source actually provides a time or a verified rule supplies it.
Common App itself has explicit deadline-time rules, demonstrating why time semantics must come from the relevant application system rather than assumptions.
---
# 52. Deadline timezone presentation
If source timezone is known:
Primary:
```text
15 Oct · 18:00 UK time
```
If device/profile timezone differs, secondary conversion may be shown:
```text
19:00 your time
```
Never replace the source timezone with the local conversion.
The canonical deadline remains tied to its source timezone.
Oxford, for example, explicitly publishes its undergraduate deadline at 18:00 UK time.
---
# 53. Unknown deadline time
Display:
```text
Time not published
```
when necessary.
If a deadline date is today but its time is unknown:
```text
Due today
Time not published
```
Do not calculate fake hours remaining.
---
# 54. Hard vs priority deadlines
Do not visually treat:
```text
priority deadline
```
as equivalent to:
```text
hard deadline
```
UI must communicate the distinction.
If a university recommends earlier submission for visa/housing reasons but accepts later applications, the earlier date must not be presented as a hard admissions cutoff. UvA currently publishes this exact type of distinction for some programmes.
---
# 55. Requirements source disclosure
Every critical admissions requirement must be traceable.
User should be able to access:
```text
Source
Last verified
```
without cluttering the default screen.
Suggested collapsed representation:
```text
Verified official source · 3 days ago
```
Click:
```text
View source
```
---
# 56. Source hierarchy
Do not silently elevate low-authority sources over official admissions information.
For core admissions requirements, primary official university/application-system information should dominate wherever available.
Common App and College Board both direct applicants toward college-specific information, while uni-assist explicitly states that authoritative application information is available through universities and official uni-assist sources.
Source authority still remains **fact-dependent**, as defined in the Data Standard.
---
# 57. Stale data
If a critical fact has become stale according to its Data Standard policy:
Never continue presenting it identically to current verified information.
Possible UI:
```text
Needs verification
```
Dependent personalized result may become:
```text
Unknown
```
when continued reliance would create unsafe certainty.
Never silently reuse last year's deadline for the current cycle.
---
# 58. Conflicting sources
When official/current sources conflict and the conflict cannot be resolved:
Display:
```text
We found conflicting information about this requirement.
```
Requirement result:
```text
Unknown
```
Provide relevant sources.
Do not:
* average values;
* select whichever appeared first;
* let an LLM decide which source "sounds right";
* hide the disagreement.
---
# 59. Unsupported institution
If institution exists but admissions intelligence has not been verified:
User may save it.
Display:
```text
We don't have verified application data for this university yet.
```
Action:
```text
Request data
```
Do not generate requirements from generic knowledge.
---
# 60. Unsupported program
If institution exists but program does not:
Allow:
```text
Save university
```
Do not create a fake program.
Relevant requirement output remains incomplete/Unknown until verified program information exists.
---
# 61. No verified requirements
Application page must still be useful.
Show:
```text
We haven't verified the requirements for this application yet.
```
Provide:
* known official application link if verified;
* known deadline if independently verified;
* source status;
* request/update option.
Do not render an empty requirements section.
---
# 62. Progress representation
Avoid deceptive generic percentages when requirement coverage is incomplete.
Bad:
```text
70% complete
```
when Ekho has unresolved requirements.
Preferred:
```text
7 of 10 known required items ready
2 requirements unresolved
```
This separates:
```text
user progress
```
from:
```text
Ekho data completeness
```
---
# 63. Progress denominator
Do not count:
* optional items;
* not-applicable items;
as required completion blockers.
Unknown items must not be silently removed from visibility.
If unknown required applicability could affect readiness, show unresolved count separately.
Example:
```text
7 / 10 known required items ready
2 unresolved
```
Do not claim:
```text
Application ready
```
while critical unresolved requirements remain.
---
# 64. Ready state
An application may only appear `ready` in the UI when:
1. all known applicable required requirements are satisfied;
2. no critical requirement remains `Unknown`;
3. no unresolved conflict/stale condition makes completeness uncertain;
4. the submission action itself has not already passed beyond a known hard deadline.
Do not equate:
```text
100% of known items
```
with:
```text
definitely complete
```
when source coverage is incomplete.
---
# 65. My Applications
Route:
```text
/app/applications
```
Purpose:
> See every active application and immediately understand which one needs attention.
Each list item should expose only decision-relevant information.
Minimum:
```text
University
Program if known
Round/route when relevant
Nearest important deadline
Progress summary
Next Action
```
---
# 66. My Applications ordering
Default priority should emphasize actionability.
Recommended:
```text
1. urgent actionable applications
2. nearest important deadline
3. remaining active applications
```
Users may later receive manual sort/filter options, but v1 must have a useful default without configuration.
---
# 67. Application list example
```text
University of Amsterdam
BSc Business Administration
15 Jan
6 / 9 known required items ready
1 unresolved
Next: Add English test score
```
Avoid cards overloaded with:
* acceptance rate;
* tuition;
* ranking;
* campus images;
* weather;
* unrelated university intelligence.
Applications is a work surface.
Explore is the research surface.
---
# 68. Returning user flow
Canonical:
```text
User opens Ekho
↓
/app
↓
Global Next Action visible immediately
↓
User completes action
↓
Requirements recompute
↓
New global Next Action
```
Returning users should not need to open each application manually to discover what matters.
---
# 69. Refresh behavior
After refresh:
* application remains;
* profile answers remain;
* completed actions remain;
* requirements load from persisted state/data;
* current Next Action remains deterministic unless underlying state changed.
Never store core application progress only in local component state.
---
# 70. Back-navigation
Browser Back must behave normally.
Closing an application setup sheet must not delete the saved application.
Returning to search should preserve the search query where technically reasonable.
Do not trap users in forced wizard navigation.
---
# 71. Failed write
If a user changes data and persistence fails:
Do not show successful completion.
Show:
```text
Couldn't save your change.
```
Action:
```text
Try again
```
The UI must return to the confirmed persisted state or clearly mark the unsaved state.
Never let frontend state imply a requirement is complete when backend persistence failed.
---
# 72. Requirement recomputation failure
If profile/action save succeeds but personalized recomputation fails:
Keep confirmed user input.
Show:
```text
Your change was saved, but requirements couldn't refresh.
```
Action:
```text
Retry
```
Do not invent a new Next Action using potentially outdated requirement state.
---
# 73. Application removal
Secondary action:
```text
Remove from Applications
```
Require intentional confirmation because it affects user progress.
After removal:
* remove from active My Applications;
* update global Next Action;
* do not leave the user on a broken Application Detail route.
Redirect:
```text
/app/applications
```
Implementation of long-term deletion/history belongs to the data/lifecycle standard rather than being invented inside the UI.
---
# 74. Changing program/round
Before applying the change, explain if it materially changes requirements.
Example:
```text
Changing your application round may change deadlines and requirements.
```
After confirmation:
```text
save
→ recompute
→ show updated requirements
```
Never retain an old deadline as if it still belongs to the new round.
---
# 75. Live Admissions Update interaction
When underlying verified admissions data changes:
```text
Source change detected
↓
Validate/normalize
↓
Determine affected applications
↓
Recompute requirements
↓
Recompute Next Actions
↓
Surface meaningful update
```
Updates live in:
```text
/app/updates
```
and can also appear contextually inside affected applications.
---
# 76. Update example
```text
English requirement changed
Previous
IELTS 6.5
Current
IELTS 7.0
Affects
Your application to University X
Detected
12 Aug 2026
Source
Official admissions page
```
Only show old/new values after the change has passed the Data Pipeline/verification rules.
---
# 77. Update consequences
If a verified rule change changes user state:
Example:
```text
Satisfied
→ Missing
```
then:
1. requirement changes;
2. progress changes;
3. application Next Action recalculates;
4. global Next Action recalculates;
5. user can see why the change occurred.
Never change the user's state silently without provenance.
---
# 78. Mobile flow
Mobile uses the exact same conceptual flow:
```text
Home
→ Search
→ University/Program
→ Add
→ Application
→ Requirements
→ Next Action
```
Do not build a separate simplified data model for mobile.
Bottom sheets may be used for contextual setup because they are appropriate for progressive disclosure when the underlying context remains relevant, but permanently important information should not live only inside sheets.
---
# 79. Keyboard / desktop interaction
Core flow must be usable without relying on pointer-only actions.
At minimum:
* search field focusable;
* search results keyboard reachable;
* Add reachable;
* setup controls reachable;
* requirement actions reachable;
* visible focus state;
* Escape closes non-destructive overlays;
* Enter does not accidentally submit destructive actions.
Full accessibility rules belong to Quality / Testing Standard.
---
# 80. Core analytics events
Use stable product events.
```text
first_home_viewed
explore_search_started
explore_search_submitted
explore_result_viewed
explore_result_opened
application_add_clicked
application_created
application_existing_opened
application_setup_viewed
application_context_answered
application_setup_resolved
requirements_viewed
requirements_computed
requirements_compute_failed
profiling_question_shown
profiling_question_answered
next_action_shown
next_action_started
next_action_completed
application_removed
activation_reached
```
Do not create a new event name for every button style or UI redesign.
---
# 81. Event properties
Use IDs, not arbitrary display strings where possible.
Example:
```text
user_id
application_id
institution_id
program_id
action_type
entry_surface
requirement_id
```
Possible `entry_surface`:
```text
home
explore
university
program
applications
application_detail
update
```
Do not put sensitive free-text profile contents into analytics unless explicitly required and privacy-reviewed.
---
# 82. Activation definition
## Primary activation
User has:
```text
created ≥1 application
AND
received ≥1 meaningful personalized/verified application result
```
Meaningful result:
* personalized requirement state; or
* verified relevant deadline; or
* actionable Next Action.
## Strong activation
User additionally:
```text
completed ≥1 Next Action
```
---
# 83. Core funnel
Track:
```text
First Home
↓
Search
↓
Result
↓
Application Created
↓
Requirements Viewed
↓
Personalized Result Produced
↓
Next Action Shown
↓
Next Action Completed
```
Measure drop-off at every boundary.
---
# 84. Key metrics
## Time to First Value
```text
first meaningful result timestamp
-
first authenticated Home timestamp
```
Target direction:
```text
≈30 seconds for straightforward cases
```
## Search → Add conversion
Shows whether Explore leads into the workspace.
## Add → Personalized Result
Critical activation metric.
## Next Action completion rate
Tests whether Ekho successfully converts information into action.
## Unknown rate
Track:
```text
requirements_unknown / requirements_evaluated
```
Break down by cause:
* missing applicant context;
* missing program context;
* source not found;
* stale;
* conflict;
* unsupported.
A low `Unknown` rate achieved by guessing is worse than a high honest `Unknown` rate.
---
# 85. Performance expectations for the flow
Core interaction must feel immediate.
Prioritize:
* search responsiveness;
* application creation;
* requirement retrieval;
* profile answer persistence;
* Next Action recomputation.
Do not block application creation on slow non-core data such as:
* rankings;
* campus media;
* descriptions;
* unrelated financial details.
Exact performance thresholds belong to Quality Standard.
---
# 86. Canonical happy path
This is the primary E2E scenario.
```text
01.
New applicant authenticates.
02.
Redirect → /app
03.
Home shows:
"Add your first university"
04.
Applicant searches:
"University of Amsterdam"
05.
Result appears.
06.
Applicant selects:
"+ Add"
07.
Application is persisted immediately.
08.
Ekho determines that program context is required.
09.
Applicant selects:
"Business Administration"
10.
Ekho determines intake automatically or asks if ambiguous.
11.
Application Detail opens.
12.
Verified application rules are loaded.
13.
Qualification is required to personalize academic requirements.
14.
Inline question:
"What qualification are you completing?"
15.
Applicant answers.
16.
Answer is saved to reusable Profile data.
17.
Requirements recompute.
18.
Applicant sees:
Satisfied / Missing / Action required / Optional / Unknown / Not applicable.
19.
Ekho selects one Next Action.
20.
Applicant completes it.
21.
Action persists.
22.
Affected requirements recompute.
23.
Progress recomputes.
24.
Next Action recomputes.
25.
Applicant immediately sees what to do next.
26.
On next visit:
Home shows the highest-priority global Next Action.
```
---
# 87. Happy path from Program page
```text
Explore
↓
Search program
↓
Program Intelligence
↓
Add to Applications
↓
Program prefilled
↓
Resolve only remaining context
↓
Application Detail
```
Never make the user select the same program twice.
---
# 88. Incomplete-context path
```text
Add university
↓
User does not know program
↓
Save application
↓
program = null
↓
Program-dependent requirements = Unknown
↓
Next Action:
"Choose your program"
```
The user is not blocked from Ekho.
---
# 89. Missing-profile path
```text
Requirements evaluation
↓
Nationality required
↓
nationality unknown
↓
Affected requirement = Unknown
↓
Inline question shown
↓
User answers
↓
Recompute
```
Do not mark the requirement Missing before the answer exists.
---
# 90. Unsupported-data path
```text
University added
↓
No verified requirement data
↓
Application still exists
↓
Explain limitation
↓
No hallucinated requirements
```
This is a valid product state.
---
# 91. Conflict path
```text
Rule evaluation
↓
Two authoritative sources disagree
↓
Data verification status = conflict
↓
Personalized requirement = Unknown
↓
Explain conflict
↓
Expose sources
```
No silent resolution.
---
# 92. Stale-data path
```text
Requirement source exceeds freshness policy
↓
verification = stale
↓
critical dependent result no longer treated as fully verified
↓
refresh/reverification pipeline triggered
↓
user sees appropriate uncertainty
```
No invisible reuse.
---
# 93. External-action path
```text
Next Action:
"Complete your application in Studielink"
↓
Open Studielink
↓
User returns to Ekho
↓
Ekho does NOT assume submission
↓
User explicitly confirms completion
or
future trusted integration confirms it
```
---
# 94. Deadline-conflict path
If application deadline itself is conflicting/stale:
Do not use the questionable date as the sole basis for:
```text
3 days remaining
```
or Next Action urgency.
Show uncertainty prominently and resolve through the data verification system.
---
# 95. No-deadline path
Some admissions workflows may not have a single conventional deadline or Ekho may not yet know one.
Then:
```text
Deadline unknown
```
or appropriate Data Standard representation.
Do not create a placeholder date.
Next Actions can still exist if independently verified.
---
# 96. Critical invariants
These must always remain true.
```text
INV-01
No application is lost because optional onboarding was abandoned.
INV-02
No absent data is converted into a confident admissions fact.
INV-03
No Unknown requirement is silently treated as Missing.
INV-04
No external link click is treated as external completion.
INV-05
No deadline time is invented.
INV-06
No stale/conflicting critical source is presented as normal verified data.
INV-07
No duplicate application is created accidentally.
INV-08
No profile question is required before it becomes useful.
INV-09
No optional requirement blocks readiness.
INV-10
No Not-applicable requirement blocks readiness.
INV-11
No profile/context change leaves dependent requirements silently stale.
INV-12
No requirement change leaves Next Action silently stale.
INV-13
Every important requirement can be traced back to evidence/source.
INV-14
There is at most one primary Next Action per application.
INV-15
There is at most one primary global Next Action on Home.
```
---
# 97. Things Codex must NOT invent
When implementing this specification, Codex must not independently add:
* onboarding wizard;
* AI counselor;
* generic chatbot;
* acceptance prediction;
* reach/target/safety probability;
* gamification;
* streaks;
* badges;
* social features;
* public profile;
* recommendations feed;
* mandatory complete profile;
* extra primary navigation;
* separate Tasks primary page;
* separate Requirements primary page;
* fake application submission;
* AI-generated university requirements;
* fabricated deadlines;
* fabricated deadline times;
* guessed eligibility;
* guessed applicant attributes;
* automatic completion after external-link click;
* new application status taxonomy without a separate decision;
* post-admission workflows not covered by this standard.
---
# 98. Out of scope
This Core User Flows standard intentionally does not define the full lifecycle after submission, such as:
* submitted application lifecycle taxonomy;
* admission decision states;
* waitlist workflow;
* offer comparison;
* enrollment;
* visa workflow;
* housing;
* counselor collaboration;
* tutor/curator marketplace;
* social/community features.
Do not implement those merely because adjacent competitor products contain them.
---
# 99. Codex implementation order
When this standard is eventually passed to Codex, implement the vertical slice first.
## Phase 1
```text
/app empty state
→ university search
→ university result
```
## Phase 2
```text
Add application
→ persistence
→ duplicate prevention
```
## Phase 3
```text
application context
→ application detail
```
## Phase 4
```text
requirements evaluation
→ requirement states
→ source states
```
## Phase 5
```text
inline progressive profiling
→ dependency recomputation
```
## Phase 6
```text
application Next Action
→ action completion
→ recomputation
```
## Phase 7
```text
returning Home
→ global Next Action
→ My Applications
```
## Phase 8
Edge/error states and analytics.
Do not build unrelated surfaces before the complete vertical slice works.
---
# 100. Required E2E tests
## E2E-01 — First activation
```text
new account
→ /app
→ search
→ add
→ application
→ requirements
→ next action
```
Must complete without dead end.
## E2E-02 — Progressive profiling
```text
requirement blocked by qualification
→ answer qualification
→ requirement recomputes
```
## E2E-03 — Duplicate
```text
add same scoped application twice
→ one application remains
```
## E2E-04 — Unknown
```text
required context missing
→ requirement = Unknown
```
Never Missing.
## E2E-05 — Application context change
```text
change program/round
→ affected requirements + deadlines + next action recompute
```
## E2E-06 — Profile change
```text
change reusable applicant field
→ all dependent applications update
```
## E2E-07 — External action
```text
open external platform
→ requirement remains incomplete
```
until explicitly confirmed.
## E2E-08 — Deadline without time
```text
official source contains date only
→ UI contains date only
```
No generated `23:59`.
## E2E-09 — Conflict
```text
conflicting source state
→ no confident requirement conclusion
```
## E2E-10 — Persistence
```text
complete action
→ refresh
→ state remains completed
```
## E2E-11 — Recompute failure
```text
save profile answer
→ requirement recompute fails
→ answer remains saved
→ error visible
→ outdated next action not silently regenerated
```
## E2E-12 — Returning user
```text
existing applications
→ /app
→ one global Next Action visible
```
---
# 101. Core acceptance criteria
Core User Flows are **not ready** until all of these are true:
* [ ] New user enters `/app` without mandatory questionnaire.
* [ ] User can begin searching immediately.
* [ ] University search supports real verified institutions.
* [ ] Program search appears only for available program data.
* [ ] No-result state has a useful next step.
* [ ] Application can be added from University Intelligence.
* [ ] Application can be added from Program Intelligence.
* [ ] Program is prefilled when added from Program Intelligence.
* [ ] Application persists before optional profiling.
* [ ] Refresh does not remove the application.
* [ ] Duplicate application is prevented.
* [ ] Application scope supports institution/program differences.
* [ ] Program can remain unknown when legitimately unresolved.
* [ ] Only necessary application context is requested.
* [ ] `Not sure yet` does not block saving.
* [ ] Applicant Profile and Application Context are separate.
* [ ] Requirements are evaluated from structured data.
* [ ] Raw/source state is separate from personalized state.
* [ ] Verification state is separate from personalized state.
* [ ] Missing evidence produces `Unknown`.
* [ ] `Missing` requires sufficient evidence.
* [ ] Conflict never creates confident output.
* [ ] Stale critical data never appears normally verified.
* [ ] Optional does not block readiness.
* [ ] Not applicable does not block readiness.
* [ ] Critical requirements expose source/freshness.
* [ ] Deadline date/time/timezone are modeled separately.
* [ ] Missing deadline time is never invented.
* [ ] Priority deadline and hard deadline are distinguishable.
* [ ] Profile questions appear contextually.
* [ ] Existing profile answers are reused.
* [ ] Profile changes recompute affected applications.
* [ ] Program/round/intake changes recompute the application.
* [ ] Every active application can produce at most one primary Next Action.
* [ ] Home exposes at most one global primary Next Action.
* [ ] Next Action ordering is deterministic.
* [ ] Optional tasks cannot outrank required tasks.
* [ ] External link opening never marks completion automatically.
* [ ] Action completion persists.
* [ ] Action completion recomputes requirements.
* [ ] Action completion recomputes progress.
* [ ] Action completion recomputes Next Action.
* [ ] Global Next Action updates after relevant changes.
* [ ] Unsupported universities never receive hallucinated requirements.
* [ ] Unsupported programs are not fabricated.
* [ ] Search/add/requirements/action states have loading/error handling.
* [ ] Backend write failure cannot appear as success.
* [ ] Requirement recomputation failure is visible.
* [ ] Returning user can immediately continue from Next Action.
* [ ] Core analytics funnel is instrumented.
* [ ] Core flow works on mobile and desktop.
* [ ] All required E2E tests pass.
---
# 102. Final canonical flow
This is the shortest authoritative description of the entire system:
```text
FIRST LOGIN
/app
↓
Search university or program
↓
Add to Applications
↓
Persist application immediately
↓
Resolve only necessary application context
↓
Open Application Detail
↓
Evaluate verified requirements
↓
Ask only missing applicant information required now
↓
Recompute
↓
Show Requirements for You
↓
Choose ONE Next Action
↓
User completes action
↓
Persist
↓
Recompute requirements
↓
Recompute progress
↓
Recompute Next Action
↓
Return to Ekho later
↓
/app shows ONE global Next Action
↓
Repeat
```
---
# 103. Product rule to remember
The user should never have to ask:
> Which page do I need to check now?
or:
> Which of these 20 requirements should I work on first?
Ekho's responsibility is to turn complex admissions information into:
```text
Here is what applies to you.
Here is what is uncertain.
Here is the source.
Here is the deadline.
Here is the one thing you should do next.
```
That is the Core User Flow.
