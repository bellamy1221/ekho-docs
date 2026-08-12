# Ekho Personalization Logic v1.0

**Status:** READY TO LOCK
**Date:** 2026-08-12
**Scope:** Applicant data, requirement-rule evaluation, personalized requirement states, progressive profiling, recomputation and uncertainty.

---

# 1. Purpose

Personalization Logic answers:

> Which admissions requirements actually apply to this applicant, and which of them are already satisfied?

Canonical transformation:

```text
Verified admissions rules
+
Application context
+
Applicant data
+
Applicant evidence/progress
↓
Personalized requirement result
```

Primary results:

```text
Satisfied
Missing
Optional
Unknown
```

Secondary internal result:

```text
Not applicable
```

These results must never be guessed.

---

# 2. Fundamental product rule

Ekho does **not** ask users to complete a universal admissions profile.

Instead:

```text
Application added
↓
Rules loaded
↓
Rules declare which applicant fields they require
↓
Ekho checks which fields are already known
↓
Only missing useful fields are requested
↓
Requirements recompute
```

This is the canonical progressive-profiling model.

---

# 3. Why this is necessary

Admissions requirements are not determined from one universal applicant profile.

UCAS states that course entry requirements can include qualifications, subjects, grades, interviews, admissions tests and medical requirements, and meeting those requirements does not itself guarantee an offer.

Common App confirms that testing, recommendations, essays and supplements differ between colleges, and some writing supplements are requested only depending on answers to other questions.

UvA determines international undergraduate requirements using the applicant's prior education/diploma, and explicitly says applicants whose prior education or country is not represented may require individual assessment.

uni-assist evaluates German university entrance qualification partly using the country of origin of the applicant's educational certificate; its own tool also warns that the result is orientation rather than a binding admissions decision.

Yale provides another type of conditional rule: its current English-proficiency requirement depends on whether the applicant is a non-native English speaker and whether they have completed at least two years in an English-medium school.

### Ekho conclusion

Never assume:

```text
citizenship
=
education system
=
qualification country
=
language background
=
residency
```

They are separate concepts.

---

# 4. Privacy principle

Ekho operates on:

```text
minimum data necessary
```

not:

```text
collect everything now because it may be useful later
```

The European Commission's GDPR guidance states that personal data should be limited to what is necessary for the defined purpose and should remain accurate and up to date.

Therefore:

> Every applicant field Ekho requests must have a current product purpose.

---

# 5. Core personalization architecture

Never implement:

```text
User Profile
↓
giant if/else
↓
requirements
```

Use:

```text
Canonical admissions data
↓
Versioned Requirement Rules

Applicant Profile
+
Application Context
+
Applicant Evidence
↓
Deterministic Rule Engine
↓
Requirement Evaluations
↓
Next Actions
```

---

# 6. Critical correction to Core User Flows

Earlier Core User Flow terminology included:

```text
Action required
```

alongside requirement statuses.

Lock the architecture more precisely now:

## Requirement result

```text
Satisfied
Missing
Optional
Unknown
Not applicable
```

## Action

Separate object:

```text
Add test score
Upload transcript
Choose qualification
Request recommendation
```

Therefore:

```text
Requirement state
≠
Action state
```

---

# 7. Why `Action required` is separate

Example:

Requirement:

```text
English proficiency
```

Personalized result:

```text
Missing
```

Possible Next Action:

```text
Register for IELTS
```

Later:

```text
Add IELTS score
```

Requirement did not change identity.

Only the best action changed.

---

# 8. `Eligible` is also separate

Never use:

```text
eligible
```

as a requirement completion status.

Bad:

```text
IELTS — Eligible
Transcript — Eligible
SAT — Eligible
```

Eligibility is a possible higher-level admissions conclusion.

It is not equivalent to:

```text
Satisfied
```

---

# 9. Requirement satisfaction does not mean admission

Critical invariant:

```text
all known requirements satisfied
≠
admission guaranteed
```

UCAS explicitly distinguishes entry requirements from an actual offer.

Therefore Ekho must not say:

```text
You will be admitted
```

or:

```text
You qualify for admission
```

merely because known requirements are satisfied.

---

# 10. Three data layers

Personalization combines three separate layers.

## Layer A — Application Context

Information about **this application**.

## Layer B — Applicant Profile

Reusable facts about **the applicant**.

## Layer C — Applicant Evidence / Progress

What the applicant has:

* completed;
* taken;
* uploaded;
* requested;
* received;
* submitted.

Never collapse all three into one Profile table.

---

# 11. Application Context fields

Potential fields:

```text
applicant_type
degree_level
institution_id
program_id
intake
academic_cycle
application_route
application_round
point_of_entry
study_mode
```

Only fields relevant to a supported application need to exist.

---

# 12. Applicant type

Canonical examples:

```text
first_year
transfer
graduate
other_supported_type
```

Do not infer applicant type purely from age.

Example:

A 19-year-old who attended university may be:

```text
transfer
```

not automatically:

```text
first_year
```

---

# 13. Program

```text
program_id
```

is application context.

Never store:

```text
intended_program
```

as one permanent global applicant field and assume it applies everywhere.

The same applicant can apply to:

```text
CS at University A
Economics at University B
Design at University C
```

---

# 14. Application round

Examples:

```text
Early Decision
Early Action
Regular Decision
Rolling
UCAS route
local equivalent
```

Round is application-specific.

Common App demonstrates why this matters: some forms can depend directly on application route/round; for example, its current first-year guide says parents only submit a form when the student applies through a college's Early Decision deadline.

---

# 15. Intake

Store canonical intake identity.

Do not reduce globally to:

```text
Fall
Spring
```

because different countries use different admissions cycles.

---

# 16. Academic cycle

Example:

```text
2026-2027
2027-2028
```

Every requirement rule must belong to the correct admissions cycle where relevant.

Never evaluate:

```text
2027 applicant
```

using a rule verified only for:

```text
2026 entry
```

without an explicit data policy allowing it.

UvA currently explicitly labels its international diploma information as applying to the 2026–2027 academic year, showing why cycle scope matters.

---

# 17. Applicant Profile categories

Potential reusable applicant information:

```text
Citizenship / residency
Education history
Qualifications
Subjects
Grades
Language background
Standardized tests
English tests
Other admissions tests
```

Not all of these should be requested initially.

---

# 18. Citizenship

Model:

```text
citizenships[]
```

not:

```text
nationality: string
```

because applicants can hold multiple citizenships.

Example:

```text
citizenships:
- IT
- BR
```

---

# 19. Never choose one citizenship silently

If a rule applies differently depending on citizenship and the applicant has multiple:

Do not automatically pick:

```text
citizenships[0]
```

The rule itself must define how multiple citizenships affect applicability.

If official guidance is unclear:

```text
Unknown
```

---

# 20. Citizenship is not country of education

Example:

```text
citizenship = India
school_country = Singapore
qualification = IB
```

Do not evaluate the applicant as:

```text
Indian curriculum
```

unless a source actually says citizenship determines the rule.

---

# 21. Country of residence

Separate:

```text
residence_country
```

from:

```text
citizenships[]
```

Residency may affect:

* tuition category;
* financial aid;
* application process;
* visa-related workflow;

depending on the institution/system.

Do not infer residency from IP address.

---

# 22. Immigration / residency status

Potential fields:

```text
residency_status
visa_status
```

must only be requested when a verified requirement actually depends on them.

Never infer:

```text
visa_required
```

solely from nationality without the relevant legal/admissions rules.

---

# 23. Education history

Model education as records.

Conceptually:

```text
education_records[]
```

Each record can contain:

```text
institution_name
institution_country

education_level

curriculum
qualification

qualification_country

start_date
expected_or_actual_end_date

instruction_languages[]

graduation_status
```

---

# 24. School country ≠ qualification country

Example:

Applicant studies:

```text
IB Diploma
```

at a school in:

```text
UAE
```

The qualification itself is:

```text
IB
```

and cannot simply be evaluated as a generic UAE national qualification.

Keep these dimensions separate.

---

# 25. Qualification

Use canonical qualification IDs where possible.

Examples:

```text
IB Diploma
A Level
Abitur
French Baccalauréat
US High School Diploma
national secondary qualification
```

Never store only:

```text
qualification = "high school"
```

when the actual qualification type matters.

---

# 26. Qualification status

Store:

```text
completed
in_progress
expected
not_started
```

where meaningful.

Do not assume every applicant already holds their final diploma.

---

# 27. Graduation date

Store:

```text
actual_graduation_date
```

or:

```text
expected_graduation_date
```

separately.

Do not turn expected dates into confirmed completion.

---

# 28. Predicted vs final academic results

Never collapse:

```text
predicted_grade
final_grade
```

into:

```text
grade
```

They have different admissions meaning.

---

# 29. Academic result model

Conceptually:

```text
qualification_id

result_type:
predicted | final | current

overall_result nullable

subject_results[]

grading_scale
```

---

# 30. Preserve original grading system

Store source result:

```text
18 / 20
```

not only:

```text
3.7 GPA
```

Any normalization must be derived separately.

---

# 31. No generic GPA conversion

Forbidden:

```text
foreign grade
↓
random online GPA formula
↓
US GPA
↓
requirement result
```

Use converted GPA only if:

1. university explicitly requires/accepts that conversion; or
2. Ekho has a defensible official equivalency source.

Otherwise:

```text
Unknown
```

---

# 32. Subject-level academic data

Many requirements are subject-specific.

Store:

```text
subject_id
qualification_context
level
grade
grade_type
```

Example:

```text
Mathematics
IB Analysis and Approaches HL
5
final
```

---

# 33. Subject taxonomy

Academic subject IDs should use normalized taxonomy but preserve original labels.

Example:

```text
canonical_subject = Mathematics
original_subject = Mathematics: Analysis and Approaches HL
```

Never throw away the more precise source value.

---

# 34. Language background

Possible fields:

```text
first_languages[]
education_instruction_languages[]
years_educated_in_language
```

But only ask these when required.

---

# 35. Never infer first language

Forbidden:

```text
citizenship = France
→ first_language = French
```

or:

```text
country = India
→ non-native English
```

These are unsafe assumptions.

---

# 36. English-medium education

Do not store only:

```text
english_medium = true
```

when rules depend on duration.

Prefer:

```text
education_record
instruction_language = English
start_date
end_date
```

Then derived:

```text
years_english_medium
```

can be calculated when necessary.

---

# 37. Why duration matters

Yale's current English-test rule specifically references whether a non-native English speaker has completed at least two years in an English-medium school.

Therefore:

```text
English-medium: yes
```

may be insufficient information.

---

# 38. Standardized test data model

Each test attempt is separate.

```text
test_attempt_id
test_type
test_variant nullable

test_date

overall_score nullable
section_scores{}

score_status

evidence_basis
```

Examples:

```text
SAT
ACT
```

---

# 39. English proficiency test model

Examples:

```text
IELTS Academic
TOEFL iBT
Duolingo English Test
Cambridge English
PTE Academic
```

Do not collapse every variant under a generic:

```text
English test
```

because universities may accept specific versions only.

---

# 40. Test attempt identity

Never store:

```text
IELTS = 7.0
```

as the complete model.

Need at least:

```text
test
variant
date
overall
subscores where relevant
```

---

# 41. Test date

Test date may affect:

* validity;
* submission deadline;
* accepted version;
* scoring format.

Therefore it must remain attached to the attempt.

---

# 42. Test subscores

Store when available.

Example:

```text
IELTS:
overall: 7.5
reading: 8
listening: 8
writing: 7
speaking: 7
```

A university may require:

```text
overall >= X
AND
every_component >= Y
```

A single overall score cannot evaluate this correctly.

---

# 43. Test version changes

Threshold interpretation may depend on test date/version.

Therefore requirement rules may include:

```text
test_version
valid_from
valid_to
```

Never assume historical and current score scales are identical.

---

# 44. Superscoring

Never combine test attempts unless the institution's verified policy allows it.

Yale currently permits certain SAT/ACT superscores but also defines restrictions on how ACT composites may be combined.

Therefore rules need:

```text
score_combination_policy
```

Possible values:

```text
single_attempt
official_superscore
best_sections
best_overall
unsupported
```

---

# 45. No homemade superscore

Forbidden:

```text
attempt 1 best Math
+
attempt 2 best English
=
Ekho superscore
```

unless the rule explicitly allows that combination.

---

# 46. Test evidence status

Possible:

```text
self_reported
document_uploaded
officially_verified
external_verified
```

Do not represent all as:

```text
verified = true/false
```

---

# 47. Self-reported scores

Self-reported data can still be useful for planning.

Yale currently accepts self-reported SAT/ACT scores during application but requires official matching scores later for admitted students.

Therefore Ekho needs to distinguish:

```text
requirement satisfied for planning/application tracking
```

from:

```text
officially verified by university
```

---

# 48. Satisfaction basis

Every `Satisfied` evaluation should internally know:

```text
satisfaction_basis
```

Examples:

```text
user_self_report
uploaded_document
official_external_verification
rule_exemption
workspace_completion
```

---

# 49. User-facing wording

If satisfaction is based on self-report:

Ekho can show:

```text
Satisfied
Based on the information you've added.
```

Do not say:

```text
Verified by University X
```

unless that actually happened.

---

# 50. Documents

Applicant Evidence can include:

```text
document_type
document_status
issuer
language
translation_status
uploaded_at
```

Potential types:

```text
transcript
school_report
diploma
predicted_grades
recommendation
financial_document
portfolio
```

---

# 51. Document existence vs upload

Separate:

```text
has_document
```

from:

```text
uploaded_to_ekho
```

and:

```text
submitted_to_university
```

These are not equivalent.

---

# 52. External submission

Never infer:

```text
submitted_to_university = true
```

because the user uploaded a document to Ekho.

---

# 53. Document workflow states

Possible:

```text
not_started
requested
received
uploaded_to_ekho
submitted_externally
externally_confirmed
```

Not every document uses every state.

---

# 54. Recommendations

Recommendation evidence needs:

```text
recommender_role
status
count
```

Possible roles:

```text
teacher
counselor
academic
professional
other
```

Common App confirms recommendation requirements differ by college.

Therefore never create one global rule:

```text
everyone needs 2 recommendations
```

---

# 55. Recommendation counts

Example rule:

```text
required:
2 teacher recommendations
```

Evaluation requires:

```text
count qualifying recommendations
```

not:

```text
count all recommendation records
```

A counselor recommendation cannot silently satisfy a teacher-specific requirement.

---

# 56. Essays and supplements

These are usually application-specific.

Store under Application Evidence/Tasks, not global Profile.

Example:

```text
application_id
essay_requirement_id
status
```

---

# 57. Conditional supplements

Common App notes that some supplements are requested based on answers to other questions.

Therefore Personalization Logic must support:

```text
answer
↓
conditional requirement appears/disappears
```

---

# 58. Do not call conditional false `Optional`

If:

```text
supplement required only for Program X
```

and applicant selected Program Y:

Result:

```text
Not applicable
```

Not:

```text
Optional
```

---

# 59. Financial aid data is a separate domain

Do not request family financial information as part of basic admissions personalization.

Financial Aid Intelligence may separately need:

```text
citizenship
residency
household size
income
assets
aid intent
```

depending on the aid programme.

---

# 60. Aid intent

Potential field:

```text
plans_to_apply_for_financial_aid
```

Ask only when relevant to:

* aid requirements;
* aid deadline;
* personalized cost intelligence.

Do not require it to add universities.

---

# 61. Financial data minimization

Do not collect:

```text
family income
family assets
bank balances
```

just to improve a generic profile.

These should require a clear aid/cost use case.

---

# 62. Sensitive data not required by default

Do not collect for core requirement personalization:

* race/ethnicity;
* religion;
* political views;
* health conditions;
* disability;
* sexual orientation;
* biometric data.

If a future accommodation/special-process feature genuinely requires sensitive data, it needs a separate privacy/legal design.

---

# 63. Parent data

Common App itself collects some parent/legal-guardian information, but Ekho is not recreating the entire Common App submission form. Common App describes those fields within its own application flow.

Therefore:

> Do not collect parent occupation, education etc. merely because Common App does.

Only collect fields that Ekho needs for a defined feature.

---

# 64. Date of birth

Do not ask date of birth universally.

Only request if a verified rule depends on:

```text
age
date of birth
```

or another product requirement legitimately needs it.

---

# 65. User-data state is NOT just nullable

Critical architecture.

Do not represent:

```text
IELTS score = null
```

because this could mean:

1. user has not answered;
2. user has never taken IELTS;
3. score pending;
4. user does not plan to take IELTS;
5. data import failed.

These states mean different things.

---

# 66. Canonical answer state

For applicant factual fields:

```text
answer_state:
unknown
known
known_none
pending
```

---

# 67. `unknown`

Means:

> Ekho does not know.

Example:

```text
SAT status = unknown
```

This must **not** produce Missing automatically.

---

# 68. `known`

Means a real value exists.

Example:

```text
SAT score = 1450
```

---

# 69. `known_none`

Means the applicant explicitly indicates absence.

Examples:

```text
I have not taken the SAT.
```

```text
I do not have this qualification.
```

This may support:

```text
Missing
```

if the item is required.

---

# 70. `pending`

Means a relevant event exists but the final result is unavailable.

Examples:

```text
SAT taken, score pending
Diploma expected
Recommendation requested
```

Do not treat `pending` as either:

```text
Satisfied
```

or:

```text
known_none
```

---

# 71. Workflow states can be known automatically

For requirements created as Ekho tasks:

```text
not_started
```

can be an explicit known workflow state.

Example:

```text
Required essay
status = not_started
```

This can produce:

```text
Missing
```

because Ekho knows the task is incomplete.

---

# 72. Absence of a Profile value is different

Example:

```text
IELTS attempts = []
```

must not necessarily mean:

```text
applicant has never taken IELTS
```

unless the product explicitly captured that meaning.

---

# 73. Field provenance

Every important applicant value should support:

```text
source
updated_at
```

Possible source:

```text
user_entered
document_extracted
imported
external_verified
derived
```

---

# 74. Derived fields

Avoid storing derived data unnecessarily.

Example:

Instead of permanently storing:

```text
years_in_english_medium = 3.6
```

Ekho can derive it from education records.

Derived outputs should retain:

```text
derivation_version
inputs
```

where auditability matters.

---

# 75. User correction

User must be able to correct:

```text
qualification
grades
test score
citizenship
education country
etc.
```

After correction:

```text
save
↓
identify affected rules
↓
recompute
```

---

# 76. Never silently override user data

If automated extraction says:

```text
IELTS = 7.0
```

but user previously entered:

```text
IELTS = 7.5
```

do not silently choose one.

Resolve the conflict explicitly or maintain separate evidence records.

---

# 77. Requirement Rule object

Canonical conceptual model:

```text
RequirementRule
```

with fields approximately:

```text
rule_id
rule_version

requirement_type

scope

applicability_expression

obligation

satisfaction_expression

accepted_alternatives

evidence_policy

stage

deadline_rule

source_ids[]

verification_status
freshness_status

valid_from
valid_to
```

---

# 78. Rule scope

May contain:

```text
institution_id
program_id nullable
degree_level
academic_cycle
intake nullable
application_route nullable
application_round nullable
applicant_type nullable
```

Specific rules should override broader rules only under an explicit precedence model.

---

# 79. Rule scope specificity

Conceptual priority:

```text
program + cycle + applicant context
>
program + cycle
>
institution + cycle
>
institution generic
```

But do not rely solely on "more fields = more authority."

Source authority and rule version still matter.

---

# 80. Rule applicability

Example:

```text
applies_if:
  qualification = IB
```

or:

```text
applies_if:
  first_language != English
  AND english_medium_years < 2
```

---

# 81. Rule obligation

Canonical internal values:

```text
required
optional
recommended
unknown
```

Do not collapse:

```text
recommended
```

into:

```text
required
```

---

# 82. Recommended items

If university says:

```text
recommended
strongly recommended
encouraged
```

Ekho must preserve that wording/strength where structured.

User-facing primary category may remain:

```text
Optional
```

with qualifier:

```text
Recommended
```

Never upgrade recommendation into a mandatory requirement.

---

# 83. Satisfaction expression

Examples:

```text
score >= 7.0
```

```text
overall >= 7.0
AND
each_component >= 6.5
```

```text
document.status >= received
```

```text
recommendation_count(teacher) >= 2
```

---

# 84. Alternatives

Many requirements can be satisfied through alternatives.

Model explicitly:

```text
ANY_OF
```

Example:

```text
IELTS
OR
TOEFL
OR
DET
OR
official exemption
```

Do not create four independent Missing requirements if only one is needed.

---

# 85. `ANY_OF`

Example:

```text
English proficiency:
ANY_OF(
  IELTS rule,
  TOEFL rule,
  DET rule,
  exemption rule
)
```

If one qualifying alternative is satisfied:

```text
Satisfied
```

---

# 86. `ALL_OF`

Example:

```text
Academic requirement:
ALL_OF(
  accepted diploma,
  mathematics requirement,
  minimum overall grade
)
```

All necessary components must pass.

---

# 87. Nested conditions

Rule engine must support:

```text
ALL_OF
  ├─ accepted qualification
  └─ ANY_OF
      ├─ Math option A
      └─ Math option B
```

Do not flatten complex official logic into generic booleans.

---

# 88. Count rules

Support:

```text
COUNT_AT_LEAST
```

Example:

```text
teacher recommendations >= 2
```

---

# 89. Threshold rules

Support typed comparisons:

```text
>=
>
<=
<
=
IN
NOT_IN
```

Do not evaluate numbers using string comparisons.

---

# 90. Exemption rules

An exemption is an official path to satisfying a requirement.

Example:

```text
English proficiency requirement
+
official English-medium education exemption
→ Satisfied
```

Internal reason:

```text
official_exemption_applies
```

---

# 91. Exemption is not guessed

Do not infer an exemption from:

```text
user seems fluent in English
DET practice score
English-language chat messages
nationality
```

Only use the verified exemption criteria.

---

# 92. Rule DSL

Use a typed declarative rule representation.

Conceptual:

```text
{
  "all": [
    {"field": "qualification", "op": "eq", "value": "IB"},
    {"field": "math.level", "op": "in", "value": ["HL", "AA_HL"]},
    {"field": "math.grade", "op": "gte", "value": 4}
  ]
}
```

---

# 93. No arbitrary runtime code in rule data

Do not store:

```text
eval("user.grade > 4 && ...")
```

or arbitrary JavaScript.

Reasons:

* security;
* debugging;
* versioning;
* auditability.

Use a constrained rule DSL.

---

# 94. Minimum rule operators

v1 should support:

```text
eq
neq

in
not_in

exists

gte
gt
lte
lt

all
any
not

count_at_least

date_before
date_after

duration_at_least
```

Add more only when actual admissions rules require them.

---

# 95. No LLM runtime evaluator

Forbidden:

```text
Send profile + university website to LLM
→ Ask "does this student satisfy the requirement?"
```

as the canonical runtime system.

The result would be difficult to:

* reproduce;
* test;
* audit;
* version;
* trust.

---

# 96. Where AI may help

AI may assist upstream:

```text
official source
↓
extract possible rule
↓
structure fields
↓
validation
↓
canonical rule database
```

But runtime personalization should use deterministic structured rules.

---

# 97. Rule versioning

Every rule needs:

```text
rule_version
```

Never mutate an important production rule without retaining the ability to determine which version produced earlier evaluations.

---

# 98. Rule validity period

Potential:

```text
valid_from
valid_to
```

or academic-cycle scope.

This prevents:

```text
2026 rule
```

from silently becoming:

```text
forever
```

---

# 99. Rule source

Every critical rule must reference source data.

Conceptually:

```text
source_ids[]
```

Requirement evaluation should remain traceable:

```text
Result
→ Rule
→ Admissions Fact
→ Official Source
```

---

# 100. Rule-source conflict

If the underlying authoritative facts conflict and cannot be resolved:

```text
Rule verification = conflict
```

Dependent personalization:

```text
Unknown
```

Never run a conflicted rule as if verified.

---

# 101. Stale rules

If freshness policy marks a critical rule stale:

Dependent result may need:

```text
Unknown
```

until reverification.

Never silently maintain:

```text
Satisfied
```

from an admissions rule no longer trusted for the current cycle.

---

# 102. Requirement evaluation dimensions

Do **not** immediately calculate one status.

First calculate:

```text
1. Scope validity
2. Applicability
3. Obligation
4. Satisfaction
5. Evidence confidence
```

Then derive the UI result.

---

# 103. Applicability

Internal values:

```text
applies
does_not_apply
unknown
```

---

# 104. Obligation

Internal values:

```text
required
optional
recommended
unknown
```

---

# 105. Satisfaction

Internal values:

```text
satisfied
unsatisfied
unknown
```

---

# 106. Why these dimensions are separate

Example:

```text
Portfolio
```

could be:

Applicant A:

```text
does_not_apply
```

Applicant B:

```text
applies + optional + unsatisfied
```

Applicant C:

```text
applies + required + satisfied
```

One boolean cannot represent this correctly.

---

# 107. Canonical final result precedence

Evaluate in this order:

```text
1. Is source/rule reliable enough?
   NO → Unknown

2. Is application scope resolved enough?
   NO → Unknown

3. Can applicability be evaluated?
   NO → Unknown

4. Does requirement apply?
   NO → Not applicable

5. Can obligation be determined?
   NO → Unknown

6. Is obligation optional/recommended?
   YES → Optional

7. Requirement is required.
   Can satisfaction be evaluated?
   NO → Unknown

8. Satisfaction true?
   YES → Satisfied

9. Satisfaction false?
   YES → Missing
```

---

# 108. Canonical truth table

```text
Applicability  Obligation  Satisfaction   Result

unknown        *           *              Unknown

false          *           *              Not applicable

true           unknown     *              Unknown

true           optional    *              Optional

true           recommended *              Optional + Recommended

true           required    unknown        Unknown

true           required    satisfied      Satisfied

true           required    unsatisfied    Missing
```

---

# 109. Optional + completed

Optionality must not disappear when user completes the item.

Example:

```text
Portfolio
Optional
Completed
```

Do not replace it with:

```text
Required/Satisfied
```

Internal:

```text
obligation = optional
satisfaction = satisfied
```

UI:

```text
Optional · Completed
```

---

# 110. Recommended + completed

Internal:

```text
obligation = recommended
satisfaction = satisfied
```

UI:

```text
Optional · Recommended · Completed
```

Exact visual treatment belongs to Design System.

---

# 111. `Satisfied`

Only return `Satisfied` when all are true:

```text
rule is reliable
requirement applies
requirement is required
Ekho has sufficient evidence
satisfaction expression evaluates true
```

---

# 112. Satisfied does NOT mean university verified

Possible basis:

```text
Based on your information
```

versus:

```text
Externally verified
```

must remain distinguishable.

---

# 113. `Missing`

Only return `Missing` when:

```text
rule reliable
AND
applies = true
AND
obligation = required
AND
satisfaction can be evaluated
AND
satisfaction = false
```

---

# 114. Absolute Missing rule

Forbidden:

```text
no answer
→ Missing
```

Correct:

```text
no answer
→ usually Unknown
```

---

# 115. Explicit absence can produce Missing

Example:

Rule:

```text
SAT or ACT required
```

Applicant says:

```text
I have not taken either.
```

Then:

```text
Missing
```

because absence is known.

---

# 116. Unknown test history

Same requirement.

Applicant has never answered anything about SAT/ACT.

Then:

```text
Unknown
```

not:

```text
Missing
```

---

# 117. `Optional`

Return when official policy says:

```text
optional
```

or:

```text
recommended
```

with appropriate qualifier.

Never infer optionality from:

```text
other universities usually don't require this
```

---

# 118. `Unknown`

Unknown is a correct and expected result.

It means:

> Ekho does not currently have enough reliable information to make a stronger statement.

This is not an error by itself.

---

# 119. Unknown reason codes

Every Unknown must have a machine-readable reason.

Minimum:

```text
missing_user_data
missing_application_context

source_unknown
source_not_found
source_not_published

source_conflict
source_stale

unsupported_rule
cannot_compare

individual_assessment_required
pending_evidence
```

---

# 120. Missing user data

Example:

```text
Academic requirement:
IB >= 36

qualification = IB
score = unknown
```

Result:

```text
Unknown
```

Reason:

```text
missing_user_data
```

Next Action may be:

```text
Add your predicted or final IB score
```

---

# 121. Missing application context

Example:

```text
requirement differs by program
program = null
```

Result:

```text
Unknown
```

Reason:

```text
missing_application_context
```

Next Action:

```text
Choose your program
```

---

# 122. Source unknown

Ekho does not have a verified rule.

Result:

```text
Unknown
```

Do not ask the applicant questions that cannot resolve the data gap.

---

# 123. Source conflict

Result:

```text
Unknown
```

Reason:

```text
source_conflict
```

UI:

```text
We found conflicting official information.
```

---

# 124. Source stale

Result:

```text
Unknown
```

when the stale rule cannot safely support a confident current conclusion.

Reason:

```text
source_stale
```

---

# 125. Individual assessment

UvA explicitly says applicants whose international prior education/country is not covered by its diploma finder can still apply and will be assessed individually.

Correct Ekho result:

```text
Unknown
```

Reason:

```text
individual_assessment_required
```

Bad:

```text
Missing
```

Bad:

```text
Not eligible
```

---

# 126. Nonbinding external tools

uni-assist describes its admission-check information as orientation rather than a binding certification decision.

Therefore results from such tools must not automatically become:

```text
University confirms you're eligible
```

Preserve source limitation.

---

# 127. `Not applicable`

Internal/user-secondary state.

Example:

```text
Portfolio required for Architecture applicants
```

User applies to Economics.

Result:

```text
Not applicable
```

Prefer hiding irrelevant requirements from the primary list.

---

# 128. Not applicable vs Optional

Critical distinction:

```text
Not applicable:
This requirement does not apply to you.

Optional:
This requirement applies, but you are not required to complete it.
```

Never mix them.

---

# 129. Missing reason codes

Useful machine-readable reasons:

```text
not_started
known_absent

score_below_minimum
subscore_below_minimum

grade_below_minimum
required_subject_missing

insufficient_count

document_not_ready

expired_evidence

required_action_incomplete
```

---

# 130. Satisfied reason codes

Examples:

```text
threshold_met
all_thresholds_met

accepted_qualification

accepted_alternative

official_exemption_applies

required_document_ready

required_count_met

workspace_action_completed
```

---

# 131. Optional reason codes

Examples:

```text
officially_optional
officially_recommended
conditional_optional
```

Do not invent `optional` because Ekho is uncertain.

Uncertainty is:

```text
Unknown
```

---

# 132. Example — required standardized test

Suppose verified rule:

```text
SAT OR ACT required
```

Yale currently requires first-year and transfer applicants to include ACT or SAT scores.

### Case A

Applicant:

```text
SAT = 1480
```

Result for test-submission requirement:

```text
Satisfied
```

Basis:

```text
user_self_report
```

---

# 133. SAT example — no test

Applicant explicitly says:

```text
SAT: not taken
ACT: not taken
```

Result:

```text
Missing
```

---

# 134. SAT example — unanswered

Applicant has given no testing information.

Result:

```text
Unknown
```

Reason:

```text
missing_user_data
```

Next question:

```text
Have you taken the SAT or ACT?
```

---

# 135. English requirement example

Yale's current rule requires an English-proficiency test for non-native English speakers who have not completed two or more years at an English-medium school.

The rule depends on:

```text
language background
+
English-medium education duration
```

Not merely:

```text
citizenship
```

---

# 136. English example — insufficient context

Applicant:

```text
citizenship = Brazil
```

Nothing else known.

Result:

```text
Unknown
```

Ekho must not conclude:

```text
English test required
```

from Brazilian citizenship alone.

---

# 137. English example — rule applies

Applicant states:

```text
non-native English speaker
english_medium_years = 0
```

Then applicability:

```text
applies
```

If no English-test information exists:

```text
Unknown
```

until Ekho knows whether the applicant has qualifying evidence.

---

# 138. English example — known no test

Applicant explicitly:

```text
no accepted English test taken
```

Requirement:

```text
required
```

Result:

```text
Missing
```

---

# 139. Important threshold distinction

Yale currently publishes scores described as typical of its "most competitive applicants" for English tests rather than presenting those numbers as a universal minimum requirement on that page.

Ekho must **not** convert:

```text
competitive applicants typically score X
```

into:

```text
minimum required score = X
```

This is exactly the kind of false certainty the system must prevent.

---

# 140. Requirement vs guidance

Canonical data must distinguish:

```text
required_threshold
recommended_threshold
competitive_reference
informational_value
```

Never compare the applicant against a recommended/reference number as though it were required.

---

# 141. Academic qualification example

UvA tells applicants with international prior education to identify the specific requirements for their diploma/context.

Possible rule inputs:

```text
qualification
qualification_country
program
cycle
```

Not:

```text
citizenship only
```

---

# 142. Qualification accepted

Rule:

```text
IB Diploma accepted
```

Applicant:

```text
qualification = IB Diploma
```

If the requirement is simply:

```text
accepted secondary qualification
```

Result may be:

```text
Satisfied
```

---

# 143. Qualification score unknown

Rule:

```text
IB >= 36
```

Applicant:

```text
qualification = IB
score = unknown
```

Result:

```text
Unknown
```

Not:

```text
Missing
```

---

# 144. Qualification below threshold

Applicant:

```text
IB = 32 final
```

Rule:

```text
IB >= 36
```

Result:

```text
Missing
```

Reason:

```text
grade_below_minimum
```

---

# 145. Predicted score nuance

Applicant:

```text
IB predicted = 38
```

Rule is:

```text
final IB >= 36
```

Do not automatically declare final condition:

```text
Satisfied
```

unless source says predicted grades can satisfy the relevant **current-stage** requirement.

---

# 146. Requirement stages

Every rule should support:

```text
stage
```

Examples:

```text
discovery
application_submission
admission_evaluation
post_offer
enrollment
```

---

# 147. Why stage matters

Example:

Before graduation:

```text
Final diploma required before enrollment
```

The applicant has not graduated yet.

It would be misleading to show:

```text
Missing
```

as an urgent current application blocker.

The requirement can exist but belong to:

```text
post_offer/enrollment
```

---

# 148. Stage-aware workflow

Requirement can be:

```text
required
unsatisfied
```

but its action should not outrank immediate application work when it is not yet actionable.

Next Action ranking uses:

```text
stage
deadline
actionability
```

---

# 149. Deadline state is separate

Do not encode:

```text
deadline passed
```

inside:

```text
Missing
```

as the only information.

Store separate:

```text
deadline_state
```

Possible:

```text
future
due_soon
due_today
passed
unknown
```

---

# 150. Requirement after deadline

Example:

```text
Required essay
Missing
Deadline passed
```

Do not convert it to:

```text
Unknown
```

just because deadline passed.

---

# 151. Alternative requirements example

Rule:

```text
English proof:
IELTS OR TOEFL OR DET
```

Applicant:

```text
DET satisfies verified rule
IELTS absent
TOEFL absent
```

Final result:

```text
Satisfied
```

Do not show:

```text
IELTS — Missing
TOEFL — Missing
```

as blockers.

---

# 152. Alternative options UI

Display conceptually:

```text
English proficiency — Satisfied

Satisfied with:
Duolingo English Test
```

Then optionally:

```text
Other accepted options:
IELTS
TOEFL
...
```

---

# 153. Required subjects

Example rule:

```text
Mathematics required
```

If qualification uses subject-level rules:

```text
qualification = IB
Math AA HL >= 4
```

Evaluation must use qualification-specific subject structure.

---

# 154. Missing subject data

Applicant:

```text
IB
subjects unknown
```

Result:

```text
Unknown
```

---

# 155. Explicitly missing required subject

Applicant provides full subject list and required Math subject is absent.

Result:

```text
Missing
```

Reason:

```text
required_subject_missing
```

---

# 156. Do not infer subject absence from partial list

If user has entered only:

```text
English
Economics
```

do not conclude:

```text
Math missing
```

unless the user indicates that the list is complete.

---

# 157. Completeness metadata

Collections that can be partial need:

```text
completeness_state
```

Example:

```text
partial
complete
unknown
```

Applies to:

* subject list;
* test attempts;
* education history;
* documents where relevant.

---

# 158. Partial data principle

```text
not present in partial data
≠
known absent
```

Critical invariant.

---

# 159. Applicant question generation

Questions must come from unresolved rule dependencies.

Conceptual:

```text
Requirement rule
↓
needs fields [qualification, score]
↓
qualification known
score unknown
↓
ask score only
```

---

# 160. No hardcoded onboarding sequence

Do not implement:

```text
Question 1 nationality
Question 2 GPA
Question 3 SAT
Question 4 IELTS
...
```

for everyone.

---

# 161. Question priority

When several applicant fields are missing, rank questions by:

```text
1. unlocks deadline/application route
2. unlocks multiple critical requirements
3. unlocks core academic eligibility information
4. unlocks one required requirement
5. unlocks optional information
```

---

# 162. Information gain

Prefer:

```text
one answer unlocking 8 requirements
```

over:

```text
one answer unlocking 1 optional requirement
```

---

# 163. Question batching

If multiple tightly related values are naturally entered together:

Example:

```text
Add your IELTS result
```

can ask:

```text
test date
overall
required subscores
```

in one compact interaction.

Do not split this into six unnecessary modal steps.

---

# 164. Question explanation

For potentially non-obvious fields, explain why.

Example:

```text
Where was your qualification issued?

We need this because this university evaluates international qualifications by the country of the certificate.
```

This follows Ekho's trust principle.

---

# 165. "Why are you asking?"

Every meaningful profile question should be traceable to:

```text
affected application(s)
affected requirement(s)
```

---

# 166. `Not sure`

Support when legitimate:

```text
Not sure
```

This stores:

```text
unknown
```

not a guessed value.

---

# 167. Skip

Optional profiling questions can support:

```text
Skip
```

If skipped:

```text
affected result remains Unknown
```

Do not block application saving.

---

# 168. Don't ask impossible questions

If uncertainty comes from:

```text
source_not_found
source_conflict
source_stale
```

do not ask the applicant for unrelated profile information.

The user cannot solve Ekho's data problem.

---

# 169. Reuse applicant data

Once:

```text
qualification = IB Diploma
```

is known globally, every relevant application should reuse it.

Do not repeatedly ask the same question.

---

# 170. Multiple qualifications

Applicant can have:

```text
qualification_records[]
```

not one global qualification string.

Examples:

```text
Russian secondary diploma
IB Diploma
A Levels
```

The rule determines which one is relevant.

---

# 171. Do not choose qualification arbitrarily

If multiple possible qualifications could satisfy the rule:

Evaluate all compatible records.

If multiple interpretations remain materially different:

ask user or produce:

```text
Unknown
```

---

# 172. Multiple education records

Never assume the most recent record is always the relevant one.

The rule may depend on:

* secondary education;
* prior university study;
* specific entrance exam;
* transfer coursework.

---

# 173. Multiple test attempts

Evaluate according to university policy.

Possible:

```text
best valid attempt
single attempt
official superscore
all attempts required
```

Never choose a strategy because it gives the user the best result unless that strategy is officially accepted.

---

# 174. Evidence validity

Requirements may include:

```text
evidence_validity_expression
```

Example:

```text
test_date >= allowed cutoff
```

Only apply validity windows supported by authoritative policy.

---

# 175. Expired evidence

If an otherwise qualifying test is known to fall outside a verified accepted validity period:

```text
Missing
```

Reason:

```text
expired_evidence
```

Potential Next Action:

```text
Take an accepted English test
```

---

# 176. Unknown test date

If score is sufficient but the test date is necessary to verify validity and date is unknown:

```text
Unknown
```

not:

```text
Satisfied
```

---

# 177. Application-route rules

Rule can depend on:

```text
application_route
```

Examples may include:

* Common App;
* UCAS;
* Studielink;
* uni-assist;
* direct university portal.

Do not assume documents/tasks are identical across routes.

---

# 178. Country-specific document rules

uni-assist states that additional documents can depend on country of origin and chosen course.

Therefore country-specific document requirements should be represented as conditional rules.

Do not hardcode them globally.

---

# 179. Recommendation optionality

If university says:

```text
0 required
1 optional
```

result:

```text
Optional
```

Do not create:

```text
Missing recommendation
```

---

# 180. Test-optional logic

Never store at institution level simply as:

```text
test_optional = true
```

Common App itself notes that testing requirements vary by college, while broader policy may also depend on applicant context.

Rule scope should support:

```text
cycle
applicant type
program
applicant context
```

where required.

---

# 181. Conditional requirements

Example:

```text
IF application_round = Early Decision
THEN early_decision_agreement required
```

This is:

```text
applicability rule
```

not:

```text
optional requirement
```

---

# 182. Changes to application context

If user changes:

```text
program
round
intake
route
```

Ekho must:

```text
identify affected rules
↓
re-evaluate applicability
↓
re-evaluate obligation
↓
re-evaluate satisfaction
↓
update actions
```

---

# 183. Changes to applicant data

Same recomputation applies after:

```text
qualification change
new grade
new test result
citizenship update
new document
etc.
```

---

# 184. Dependency graph

Do not recompute everything blindly if avoidable.

Conceptually maintain:

```text
field
↓
dependent rules
↓
affected applications
```

Example:

```text
IELTS score
↓
English requirements
↓
6 applications
```

---

# 185. Cross-application recomputation

New IELTS result may update:

```text
Application A → Satisfied
Application B → Satisfied
Application C → Missing
Application D → Unknown
```

because rules differ.

Do not store:

```text
IELTS requirement satisfied = true
```

globally.

---

# 186. Global fact vs application conclusion

Correct:

```text
Profile:
IELTS = 7.0

Application A:
English = Satisfied

Application B:
English = Missing

Application C:
English = Optional
```

Same user data.

Different rules.

---

# 187. Evaluation must be deterministic

Given:

```text
same rule version
same application context
same applicant data
same evidence
same source state
```

the result must be identical.

---

# 188. Evaluation record

For important requirements store or reproducibly generate:

```text
evaluation_id

application_id
requirement_id

rule_id
rule_version

result
reason_code

input_snapshot/reference

evaluated_at

source_ids
```

---

# 189. Explainability

Ekho must be able to answer:

> Why is this Missing?

Example:

```text
Required:
IELTS 7.0 overall

Your information:
IELTS 6.5

Result:
Missing

Source:
Official university admissions page
```

---

# 190. Avoid black-box explanation

Bad:

```text
AI determined that you do not meet this requirement.
```

That is unacceptable for critical admissions information.

---

# 191. Result explanation pattern

Every important result can conceptually expose:

```text
Requirement
Why it applies
Official requirement
Your information used
Result
Source
Last verified
```

Progressive disclosure can hide most of this by default.

---

# 192. Unknown explanation

Unknown must explain **what is unknown**.

Bad:

```text
Unknown
```

Better:

```text
We need your qualification to check this requirement.
```

or:

```text
The university has not published enough information for us to determine this.
```

---

# 193. Unknown must distinguish ownership

Two major categories:

## User-resolvable

```text
missing_user_data
missing_application_context
pending_evidence
```

## Ekho/data-resolvable

```text
source_unknown
source_conflict
source_stale
unsupported_rule
```

---

# 194. Next Action integration

User-resolvable Unknown may produce:

```text
Next Action
```

Example:

```text
Add your qualification
```

Data-resolvable Unknown generally should **not** ask the applicant to fix it.

---

# 195. Missing → actions

Examples:

```text
Missing English test
→ Register for an accepted test

Missing transcript
→ Request transcript

Missing recommendation
→ Invite recommender

Missing essay
→ Complete essay
```

Action selection belongs to Core User Flows, not the rule result itself.

---

# 196. Satisfied → no blocker action

Satisfied requirements do not create required Next Actions unless another workflow stage requires something new.

Example:

```text
self-reported SAT satisfied
```

may later generate:

```text
Send official score
```

only if a separate later-stage rule requires official verification.

---

# 197. Stage-specific duplicate-looking requirements

Correct architecture can contain:

```text
SAT score for application
```

and later:

```text
Official SAT verification before enrollment
```

as separate stage requirements.

Do not mutate the first requirement into a completely different meaning.

---

# 198. Requirement grouping

UI may group:

```text
Academic
Language
Tests
Documents
Writing
Recommendations
Interview/Audition
Application
Financial Aid
```

But grouping must not affect rule evaluation.

---

# 199. Completion progress

Required progress counts only requirements that are:

```text
applies
+
required
```

Optional items do not increase denominator.

---

# 200. Unknown denominator handling

Do not hide unresolved critical requirements from readiness.

Example:

```text
7 / 9 known required items satisfied
2 unresolved
```

Better than:

```text
78% complete
```

when critical scope is uncertain.

---

# 201. Ready-state rule

Application must not appear fully ready when:

```text
critical Unknown > 0
```

even if:

```text
all known required = Satisfied
```

---

# 202. Non-critical Unknown

Some informational optional items may remain Unknown without blocking application readiness.

Therefore requirements need:

```text
criticality
```

or equivalent rule metadata.

Do not make every Unknown a full application blocker.

---

# 203. Criticality

Potential values:

```text
blocking
important
non_blocking
```

Do not infer criticality from UI category.

Use admissions meaning.

---

# 204. Rule precedence

Potential conflict:

```text
Institution:
SAT optional

Program:
SAT required
```

If program rule is verified and explicitly overrides institution rule:

Use program rule.

But this relationship must be encoded.

Never let frontend decide:

```text
program seems more specific so probably required
```

---

# 205. Override relation

Rule model may support:

```text
overrides_rule_id
```

or an explicit scope precedence mechanism.

Data Pipeline owns extraction/validation.

Personalization owns deterministic evaluation.

---

# 206. Conflicting rules without valid override

If two current authoritative rules disagree and no verified precedence exists:

```text
Unknown
```

Reason:

```text
source_conflict
```

---

# 207. Applicant-context conflict

If user data itself conflicts:

Example:

```text
Profile:
qualification = IB

Uploaded document:
qualification = A Levels
```

Do not pick one silently.

Result depending on qualification:

```text
Unknown
```

until resolved.

---

# 208. User data freshness

Some applicant data changes naturally.

Examples:

```text
current grades
predicted grades
test scores
application round
```

Keep:

```text
updated_at
```

where relevant.

---

# 209. Do not request re-entry unnecessarily

If a field is old but still inherently stable:

```text
citizenship
```

do not automatically ask every month.

If a field is time-sensitive:

```text
predicted grade
```

refreshing may be more reasonable.

Exact freshness belongs to field policy.

---

# 210. Deletion

If user deletes a Profile value that affects requirements:

```text
delete
↓
dependent evaluations recompute
↓
potentially Satisfied/Missing → Unknown
```

Do not keep hidden derived conclusions based on deleted data.

---

# 211. Privacy by architecture

Do not send the entire Profile to every requirement evaluation when only one field is needed.

Rule engine should access/evaluate only necessary structured inputs where practical.

---

# 212. Analytics privacy

Do not send values such as:

```text
family income
passport number
document content
exact sensitive profile values
```

into generic product analytics.

---

# 213. Personalization analytics events

Minimum:

```text
personalization_question_shown
personalization_question_answered
personalization_question_skipped

requirement_evaluated
requirement_recomputed

requirement_result_changed

personalization_compute_failed
```

---

# 214. Requirement analytics properties

Safe useful properties:

```text
requirement_type
result
reason_code

rule_id
rule_version

application_id
institution_id

trigger
```

Avoid raw private applicant values.

---

# 215. Recompute trigger

Potential:

```text
profile_updated
application_context_updated
rule_updated
source_reverified
evidence_updated
action_completed
```

---

# 216. Personalization metrics

Track:

## Unknown rate

```text
Unknown / evaluated requirements
```

---

# 217. Unknown reason distribution

Break down:

```text
missing_user_data
missing_application_context
source_gap
source_conflict
source_stale
individual_assessment
```

This tells Ekho whether problems come from:

```text
UX
```

or:

```text
data coverage
```

---

# 218. Questions to first value

Measure:

```text
number of applicant questions
before first personalized useful result
```

Goal:

```text
as low as possible
```

Not:

```text
maximize profile completeness
```

---

# 219. Question usefulness

Track whether answering a question changes:

```text
Unknown
→
Satisfied / Missing / Optional / Not applicable
```

Questions that rarely unlock useful results should be reconsidered.

---

# 220. Never optimize Unknown rate by guessing

Bad KPI behavior:

```text
Unknown too high
→ infer missing values
```

Correct:

```text
Unknown too high
→ improve verified rules/data
or
ask fewer, better questions
```

---

# 221. Core user fields — v1 final

These are **supported fields**, not mandatory onboarding fields.

## Application Context

```text
applicant_type
degree_level
program
intake
academic_cycle
application_route
application_round
point_of_entry where needed
```

## Identity / geography

```text
citizenships[]
residence_country when required
residency_status when required
```

## Education

```text
education_records[]
school_country
qualification_country
curriculum
qualification
qualification_status
graduation_date / expected date
instruction_languages
```

## Academic

```text
overall grades
predicted grades
final grades
subject-level grades
grading scale
```

## Tests

```text
SAT
ACT
IELTS
TOEFL
DET
Cambridge
PTE
other verified admissions tests
```

with:

```text
date
variant
overall
subscores
evidence basis
```

## Evidence/workflow

```text
documents
recommendations
essays
portfolios
interviews
external forms
```

---

# 222. Conditional fields — only when needed

Potential:

```text
first_language
English-medium duration

visa/residency status

prior university attendance

entrance exam history

specific subject history

aid intent

financial information
```

---

# 223. Explicitly not universal

Do NOT create mandatory onboarding fields for:

```text
GPA
SAT
ACT
IELTS
DET
TOEFL

citizenship

date of birth

parent education

family income

intended major
```

The fact that a field can matter somewhere does not justify asking everyone.

---

# 224. Question dependency registry

Every profile field should document:

```text
field_id

why it exists

which rule types can require it

sensitivity

reusable_scope

allowed values

validation

deletion behavior
```

---

# 225. Field example

```text
field_id:
qualification

purpose:
Evaluate qualification-specific academic requirements

scope:
global reusable applicant data

required_on_signup:
false

sensitivity:
normal personal/education data
```

---

# 226. English medium field example

```text
field_id:
education_instruction_language

purpose:
Evaluate verified language-test exemption rules

required_on_signup:
false

requested_when:
an active application rule requires it
```

---

# 227. Financial field example

```text
field_id:
household_income

purpose:
Financial aid / net-cost calculation only

required_for_admissions_personalization:
false

required_on_signup:
false
```

---

# 228. UI Profile page

Profile may exist as a secondary management surface.

But it is **not**:

```text
Complete your profile to 100%
```

Do not gamify completeness.

---

# 229. Profile UI principle

Show useful groups:

```text
Education
Tests
Personal details
```

and perhaps:

```text
Used by 4 applications
```

Do not create dozens of empty required fields.

---

# 230. Optional profile pre-filling

Power users may choose to enter data proactively.

Allowed.

But:

```text
proactive input
≠
mandatory onboarding
```

---

# 231. Personalization question location

Prefer contextual location:

```text
Application Detail
Requirements section
Next Action
```

Not constant redirect to:

```text
Profile
```

---

# 232. First personalization flow

Canonical:

```text
Application created
↓
Rules evaluated
↓
5 requirements known
3 unresolved because qualification missing
↓
Ekho identifies qualification as highest-value missing field
↓
"Which qualification are you completing?"
↓
User answers
↓
Save globally
↓
Recompute
↓
requirements update immediately
```

---

# 233. Multi-application leverage

Example:

Applicant has:

```text
Oxford
UvA
Bocconi
```

All need qualification context.

User answers qualification once.

Ekho recomputes all three.

This is one of the strongest reasons for Profile reuse.

---

# 234. Example final outputs

### Example A

```text
Requirement:
SAT/ACT

Policy:
Required

Applicant:
SAT 1480

Result:
Satisfied
```

---

# 235. Example B

```text
Requirement:
SAT/ACT

Policy:
Required

Applicant:
Explicitly no SAT or ACT

Result:
Missing
```

---

# 236. Example C

```text
Requirement:
SAT/ACT

Policy:
Required

Applicant:
Testing data unknown

Result:
Unknown

Reason:
missing_user_data
```

---

# 237. Example D

```text
Requirement:
Portfolio

Policy:
Optional

Applicant:
No portfolio

Result:
Optional
```

---

# 238. Example E

```text
Requirement:
Architecture portfolio

Applicant program:
Economics

Result:
Not applicable
```

---

# 239. Example F

```text
Requirement:
IB score >= 36

Applicant:
IB
score unknown

Result:
Unknown
```

---

# 240. Example G

```text
Requirement:
IB score >= 36

Applicant:
IB 32 final

Result:
Missing
```

---

# 241. Example H

```text
Requirement:
English proficiency

Accepted:
IELTS >= X
OR TOEFL >= Y
OR official exemption

Applicant:
Official exemption conditions met

Result:
Satisfied

Reason:
official_exemption_applies
```

---

# 242. Example I

```text
Requirement:
Qualification equivalency

Applicant:
Qualification not covered
University says individual assessment required

Result:
Unknown

Reason:
individual_assessment_required
```

---

# 243. Example J

```text
Requirement:
English score

University source:
"Competitive applicants typically score 120 DET"

No official minimum published.

Applicant:
DET 115

Result:
NOT automatically Missing
```

The competitive reference must not be converted into a minimum.

---

# 244. Example K — source conflict

```text
Admissions page:
IELTS 7.0

Program page:
IELTS 6.5

No verified precedence
```

Result:

```text
Unknown
```

Reason:

```text
source_conflict
```

---

# 245. Example L — stale cycle

```text
Rule:
SAT optional

Verified for:
2025-2026

Application:
2027-2028

No verified carry-forward policy
```

Result:

```text
Unknown
```

not:

```text
Optional
```

---

# 246. Rule-engine function

Conceptually:

```text
evaluateRequirement(
  rule,
  applicationContext,
  applicantProfile,
  evidence,
  sourceState
)
```

returns:

```text
{
  applicability,
  obligation,
  satisfaction,

  result,
  reasonCode,

  missingInputs,

  evidenceBasis,

  ruleVersion,
  sourceIds
}
```

---

# 247. Evaluation must be server-authoritative

Frontend can optimistically render simple changes where safe.

But canonical requirement evaluation should live in shared/server domain logic.

Do not duplicate the rule engine separately in:

```text
React component A
React component B
mobile component C
```

---

# 248. One canonical evaluator

All surfaces consume the same result:

```text
Home
Applications
Application Detail
Requirements
Updates
```

No screen-specific admissions logic.

---

# 249. Cache behavior

Personalized evaluations may be cached.

But cache key must account for:

```text
application context version
applicant data version
rule version
source state version
```

Never serve stale personalized conclusions after underlying inputs changed.

---

# 250. Recompute failure

If user input saves but recomputation fails:

```text
input remains saved
```

UI:

```text
Your information was saved, but requirements couldn't refresh.
```

Do not display newly fabricated results.

---

# 251. Idempotency

Running the same recomputation twice should not create duplicate requirements/actions/events.

---

# 252. Historical evaluation

When a rule changes:

Do not lose ability to understand:

```text
why was this Satisfied yesterday?
```

At minimum retain rule version/change provenance.

---

# 253. Live Admissions Updates integration

Verified rule update:

```text
Rule v3
→ Rule v4
```

causes:

```text
identify affected applications
↓
recompute
↓
compare previous result
↓
show meaningful change
```

---

# 254. Result transition examples

Possible:

```text
Satisfied → Missing
Missing → Satisfied
Unknown → Satisfied
Unknown → Missing
Optional → Required/Missing
Required/Missing → Optional
```

All meaningful transitions caused by source updates should retain provenance.

---

# 255. User-data update transitions

Example:

```text
IELTS:
6.5 → 7.5
```

may produce:

```text
Application A:
Missing → Satisfied

Application B:
Missing → Missing

Application C:
Unknown → Satisfied
```

---

# 256. No acceptance probability

Personalization must never produce:

```text
80% chance of admission
```

from requirement completion.

That is outside this system.

---

# 257. No Reach / Target / Safety from rule completion

Do not derive:

```text
Safety
Target
Reach
```

because:

```text
minimum requirements satisfied
```

Holistic/selective admission is fundamentally different.

---

# 258. No invented contextual admissions

UCAS notes that contextual admissions can alter offers based on personal circumstances.

Ekho must not attempt to infer contextual eligibility unless:

1. the institution publishes sufficiently explicit criteria;
2. the required applicant data is legitimately collected;
3. the resulting rule is reliable.

Otherwise:

```text
Unknown
```

or simply present information without personalized conclusion.

---

# 259. No sensitive-context inference

Never infer contextual eligibility from:

* postcode;
* race;
* disability;
* family circumstances;

unless a future explicitly scoped feature has appropriate verified rules and privacy design.

---

# 260. Requirement coverage indicator

Each application can internally know:

```text
evaluated
unknown_user_context
unknown_data_gap
not_applicable
```

This can support honest completeness reporting.

---

# 261. Personalization readiness

Application personalization can be considered sufficiently resolved when:

```text
all critical applicability rules resolved
AND
all critical required requirements have
Satisfied or Missing
```

Optional unresolved informational data should not necessarily block this.

---

# 262. Requirement visibility

Default primary list:

```text
Satisfied
Missing
Optional
Unknown
```

Hide:

```text
Not applicable
```

unless user expands:

```text
Not applicable
```

or needs an explanation.

---

# 263. Unknown ordering

Within Application Detail:

User-resolvable Unknown requirements can appear relatively high because they block certainty.

Data-gap Unknowns should remain visible but should not repeatedly demand impossible user action.

---

# 264. Missing ordering

Missing does not automatically mean highest priority.

Next Action should consider:

```text
deadline
stage
lead time
dependency
```

from Core User Flows.

---

# 265. Security constraints

Rule input must be validated.

Never trust arbitrary client-provided:

```text
rule_id
evaluation_result
```

as authoritative.

User submits facts/evidence.

Backend computes conclusions.

---

# 266. User cannot set `Satisfied` directly

Bad API:

```text
PATCH requirement
status = satisfied
```

for rules based on academic/test data.

Instead user changes underlying evidence:

```text
Add IELTS result
```

then evaluator calculates status.

---

# 267. Manual completion exception

Workflow requirements can support manual user confirmation:

```text
Mark as completed
```

But internally:

```text
evidence_basis = user_confirmation
```

must remain known.

---

# 268. External verified completion

If future integration confirms submission:

```text
evidence_basis = external_verified
```

This can supersede manual tracking where appropriate.

---

# 269. Required E2E test — missing user context

```text
rule requires qualification
qualification unknown
↓
Unknown
↓
question shown
↓
qualification added
↓
recompute
```

---

# 270. E2E — explicit absence

```text
SAT required
SAT/ACT unknown
→ Unknown

user selects:
"I haven't taken either"

→ Missing
```

---

# 271. E2E — qualifying evidence

```text
required threshold
+
qualifying score
↓
Satisfied
```

---

# 272. E2E — below threshold

```text
required threshold
+
known below-threshold score
↓
Missing
```

---

# 273. E2E — optional

```text
verified optional policy
↓
Optional
```

regardless of whether the user has completed the optional item.

---

# 274. E2E — recommended

```text
verified recommended policy
↓
Optional + Recommended
```

Never Required.

---

# 275. E2E — not applicable

```text
portfolio only applies to architecture
+
economics application
↓
Not applicable
```

---

# 276. E2E — conditional application route

```text
Round A
→ requirement not applicable

change to Round B
→ requirement applies + required
```

Recompute immediately.

---

# 277. E2E — qualification change

```text
qualification A
→ requirement Satisfied

change profile to qualification B
→ affected rule re-evaluates
```

Old status must not remain cached.

---

# 278. E2E — multi-application reuse

```text
3 applications
qualification unknown

answer qualification once

→ all dependent applications recompute
```

---

# 279. E2E — multiple citizenships

```text
citizenships = [A, B]
```

No rule may silently evaluate only citizenship A unless verified logic says so.

---

# 280. E2E — country separation

```text
citizenship = A
school_country = B
qualification_country = C
```

Rule using qualification country must use C.

---

# 281. E2E — partial subject list

```text
subjects completeness = partial
required subject absent from entered list
```

Expected:

```text
Unknown
```

not Missing.

---

# 282. E2E — complete subject list

```text
subjects completeness = complete
required subject absent
```

Expected:

```text
Missing
```

---

# 283. E2E — alternatives

```text
IELTS missing
TOEFL satisfied
```

Parent language requirement:

```text
Satisfied
```

---

# 284. E2E — subscore

```text
overall meets minimum
one required component below minimum
```

Expected:

```text
Missing
```

if verified rule requires every component threshold.

---

# 285. E2E — missing subscore

```text
overall known
required component unknown
```

Expected:

```text
Unknown
```

---

# 286. E2E — superscore prohibited

Two attempts individually fail.

Combined hypothetical score passes.

Policy does not permit combining.

Expected:

```text
Missing
```

---

# 287. E2E — superscore allowed

Verified policy permits relevant combination.

Combination passes.

Expected:

```text
Satisfied
```

with correct rule basis.

---

# 288. E2E — stale rule

```text
critical rule stale
```

Expected:

```text
Unknown
```

when freshness policy requires reverification.

---

# 289. E2E — source conflict

```text
authoritative rules conflict
```

Expected:

```text
Unknown
reason = source_conflict
```

---

# 290. E2E — unsupported qualification

University explicitly requires individual review.

Expected:

```text
Unknown
reason = individual_assessment_required
```

Never ineligible.

---

# 291. E2E — optional completed

```text
optional essay
+
essay completed
```

Expected:

```text
Optional
completion = completed
```

---

# 292. E2E — application stage

```text
final diploma required for enrollment
student still in school
```

Requirement must not incorrectly become current top application blocker.

---

# 293. E2E — deleted applicant field

```text
Satisfied
↓
delete supporting score
↓
Unknown
```

if no other qualifying evidence remains.

---

# 294. E2E — recompute failure

```text
user updates qualification
save succeeds
evaluation fails
```

Expected:

```text
new qualification remains saved
requirements show refresh error
no false new result
```

---

# 295. Acceptance criteria

Personalization Logic is **not ready** until:

* [ ] Application Context is separate from Applicant Profile.
* [ ] Applicant Profile is separate from Evidence/Progress.
* [ ] Program is application-specific.
* [ ] Round is application-specific.
* [ ] Intake is application-specific.
* [ ] Academic cycle scopes requirements correctly.
* [ ] Applicant type is explicit where required.
* [ ] Citizenship supports multiple values.
* [ ] Citizenship is not treated as country of education.
* [ ] Residence is separate from citizenship.
* [ ] Qualification country is separately representable.
* [ ] Curriculum/qualification is structured.
* [ ] Multiple education records are supported.
* [ ] Multiple qualifications are supported.
* [ ] Predicted and final grades are separate.
* [ ] Original grading scale is preserved.
* [ ] Generic GPA conversion is not used for critical evaluation.
* [ ] Subject-level requirements are supported.
* [ ] Partial subject lists cannot create false Missing states.
* [ ] Language background is not inferred from nationality.
* [ ] English-medium education duration can be represented.
* [ ] Multiple test attempts are supported.
* [ ] Test variants are supported.
* [ ] Test dates are supported.
* [ ] Test subscores are supported.
* [ ] Test evidence source is represented.
* [ ] Superscore policy is rule-driven.
* [ ] Ekho never creates its own superscore unless permitted.
* [ ] Documents distinguish possession/upload/submission.
* [ ] Recommendation role/count rules are supported.
* [ ] Conditional supplements are supported.
* [ ] Financial data is not required for normal admissions personalization.
* [ ] Sensitive data is not collected by default.
* [ ] Profile completeness is not a product requirement.
* [ ] `unknown` and `known_none` are different.
* [ ] `pending` is supported where necessary.
* [ ] Applicant collection completeness can be represented.
* [ ] Missing data does not automatically equal Missing requirement.
* [ ] Explicit known absence can produce Missing.
* [ ] Requirement rules are versioned.
* [ ] Requirement rules have source provenance.
* [ ] Requirement rules are cycle/scoped.
* [ ] Applicability is calculated separately.
* [ ] Obligation is calculated separately.
* [ ] Satisfaction is calculated separately.
* [ ] `Satisfied` is deterministically derived.
* [ ] `Missing` requires sufficient evidence.
* [ ] `Optional` requires verified optional/recommended policy.
* [ ] `Unknown` has a reason code.
* [ ] `Not applicable` is separate from Optional.
* [ ] Recommended is not converted to Required.
* [ ] Competitive/reference scores are not converted to minimums.
* [ ] Alternatives use proper `ANY_OF`.
* [ ] Combined requirements support `ALL_OF`.
* [ ] Count requirements are supported.
* [ ] Exemptions are rule-driven.
* [ ] No runtime LLM decides requirement state.
* [ ] No arbitrary `eval()` rule code exists.
* [ ] User cannot directly set academic/test requirements to Satisfied.
* [ ] Self-reported satisfaction is distinguished from official verification.
* [ ] Requirement stage is supported.
* [ ] Deadline state is separate from requirement state.
* [ ] Profile updates recompute affected applications.
* [ ] Application Context updates recompute the application.
* [ ] Rule updates recompute affected applications.
* [ ] Source conflicts produce Unknown.
* [ ] Critical stale rules do not retain false certainty.
* [ ] Deleted applicant inputs invalidate dependent evaluations.
* [ ] Evaluation is deterministic.
* [ ] Evaluation provenance is traceable.
* [ ] Every important result can explain why.
* [ ] User-resolvable Unknown and data-gap Unknown are distinguished.
* [ ] Only user-resolvable uncertainty creates profiling actions.
* [ ] Application readiness does not ignore critical Unknowns.
* [ ] Optional requirements do not block readiness.
* [ ] Not-applicable requirements do not block readiness.
* [ ] Meeting requirements never becomes admission probability.
* [ ] Core personalization analytics contain no unnecessary sensitive values.
* [ ] All required E2E tests pass.

---

# 296. Things Codex must NOT invent

When implementing Personalization Logic, Codex must not independently add:

```text
mandatory full onboarding

profile completion %

one nationality string

nationality → education-system inference

nationality → language inference

IP → residency inference

generic GPA conversion

global test-optional boolean

global English-test boolean

global eligibility boolean

acceptance probability

reach / target / safety

AI-generated admissions rules

runtime LLM requirement evaluation

automatic university-policy assumptions

Unknown → Missing fallback

recommended → required conversion

competitive score → minimum threshold conversion

partial data → known absence

automatic homemade superscoring

self-report → officially verified

application requirement satisfied → admitted/eligible
```

---

# 297. Codex implementation order

## Phase 1 — Applicant data foundation

Implement canonical structures for:

```text
Applicant Profile
Education Records
Qualifications
Academic Results
Test Attempts
Evidence
Application Context
```

---

# 298. Phase 2 — Explicit unknown states

Implement distinction:

```text
unknown
known
known_none
pending
```

and collection completeness where necessary.

Do this **before** rule evaluation.

---

# 299. Phase 3 — Rule model

Implement:

```text
RequirementRule
Rule Scope
Rule Version
Source links
Applicability
Obligation
Satisfaction
```

---

# 300. Phase 4 — Typed rule engine

Minimum support:

```text
all
any
not

eq
neq
in
not_in

exists

gte
gt
lte
lt

count_at_least

duration_at_least

date comparisons
```

---

# 301. Phase 5 — Final result derivation

Implement:

```text
Satisfied
Missing
Optional
Unknown
Not applicable
```

using canonical precedence.

No UI-specific shortcuts.

---

# 302. Phase 6 — Progressive profiling

Implement:

```text
rule missing dependencies
↓
identify highest-value applicant field
↓
ask contextually
↓
persist
↓
recompute
```

---

# 303. Phase 7 — Cross-application recomputation

Applicant Profile update:

```text
dependency graph
↓
affected applications
↓
recompute
```

---

# 304. Phase 8 — Explanation/provenance

Every critical result gains:

```text
reason
source
last verified
applicant information used
```

---

# 305. Phase 9 — Core User Flow integration

Convert personalized requirement results into:

```text
Next Actions
Progress
Home global Next Action
```

Action generation remains a separate domain layer.

---

# 306. Phase 10 — Analytics + edge cases

Implement:

```text
Unknown reasons
question usage
result transitions
compute failures
```

Then test with real launch university rules.

---

# 307. Canonical personalization algorithm

```text
01
Load Application Context

02
Load relevant verified Requirement Rules

03
Reject/exclude rules outside:
program
cycle
intake
route
round
applicant type

04
Check source verification/freshness

05
For each rule:
evaluate applicability

06
If required applicant input missing:
Unknown
+ missing input dependency

07
If does not apply:
Not applicable

08
Determine obligation:
Required / Optional / Recommended

09
If Optional/Recommended:
Optional

10
If Required:
evaluate satisfaction

11
If satisfaction lacks required applicant/evidence data:
Unknown

12
If condition passes:
Satisfied

13
If condition conclusively fails:
Missing

14
Attach:
reason code
rule version
source
evidence basis

15
Aggregate alternatives/groups

16
Recompute application progress

17
Send results to Next Action engine
```

---

# 308. Canonical progressive-profiling algorithm

```text
Personalized evaluations
↓
collect unresolved user-resolvable dependencies
↓
group duplicate fields
↓
score by information gain / criticality
↓
choose highest-value question
↓
ask one contextual question
↓
save answer to correct scope
↓
recompute all dependent applications
↓
repeat only if useful
```

---

# 309. Final locked requirement-state model

```text
SOURCE / RULE
    ↓
Is the rule reliable?
    ├─ NO → Unknown
    ↓ YES

APPLICATION SCOPE
    ↓
Do we know if it applies?
    ├─ NO → Unknown
    ├─ DOES NOT APPLY → Not applicable
    ↓ APPLIES

OBLIGATION
    ├─ UNKNOWN → Unknown
    ├─ OPTIONAL → Optional
    ├─ RECOMMENDED → Optional + Recommended
    ↓ REQUIRED

SATISFACTION
    ├─ INSUFFICIENT DATA → Unknown
    ├─ PASS → Satisfied
    └─ FAIL → Missing
```

---

# 310. Final product rule

Ekho must distinguish four fundamentally different situations:

```text
Satisfied
"We have enough reliable information to conclude that you meet this requirement."

Missing
"We have enough reliable information to conclude that this required item is not currently satisfied."

Optional
"This applies to your application, but the official policy does not require it."

Unknown
"We do not currently have enough reliable information to say."
```

And one additional case:

```text
Not applicable
"This requirement does not apply to your application."
```

The most important invariant of the whole system:

> **Ekho may ask for more information, or it may say Unknown. It must never manufacture certainty.**
