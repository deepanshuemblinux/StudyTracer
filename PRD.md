# StudyTracer PRD

## Table of Contents
- [1. Document Control](#1-document-control)
- [2. Context and Problem Statement](#2-context-and-problem-statement)
- [3. Goals and Non-Goals](#3-goals-and-non-goals)
- [4. Users and Personas](#4-users-and-personas)
- [5. Scope](#5-scope)
- [6. Information Architecture](#6-information-architecture)
- [7. User Flow Catalog](#7-user-flow-catalog)
- [8. Functional Requirements](#8-functional-requirements)
- [9. Non-Functional Requirements](#9-non-functional-requirements)
- [10. UX Requirements](#10-ux-requirements)
- [11. Integrations and Dependencies](#11-integrations-and-dependencies)
- [12. Risks and Mitigations](#12-risks-and-mitigations)
- [13. Acceptance Criteria Matrix](#13-acceptance-criteria-matrix)
- [14. Open Questions and Decision Log](#14-open-questions-and-decision-log)

## 1. Document Control

| Field | Value |
|---|---|
| Document | StudyTracer Product Requirements Document |
| Version | Draft v0.1 |
| Date | TBD |
| Owner | TBD |
| Contributors | TBD |
| Status | In Progress |

## 2. Context and Problem Statement

### Background
StudyTracer is an AI-powered platform that evaluates handwritten student submissions in `math` and `science` against structured diagnostic rubrics. Its purpose is to identify specific learning gaps, support tutor-reviewed evaluation, generate targeted remediation assignments, and track measurable progress over time.

### Problem
Tutors do not have the time to manually review every step of handwritten student work at the level needed to distinguish among different error types such as `concept gaps`, `logical-step gaps`, and `calculation/execution errors`. As a result, students receive broad chapter-level feedback instead of precise diagnosis, and remediation remains inefficient.

### Current Failure
The primary failure is `lack of visibility into step-level reasoning and evidence-backed diagnosis`.

This affects stakeholders differently:
- Tutors cannot efficiently produce granular, repeatable evaluations across all submissions.
- Students do not get precise question-level feedback explaining what went wrong.
- Parents do not get a sufficiently clear, tutor-approved concept/chapter-level view of weak areas, remarks, and remediation plans.

### Current Workaround
Tutors rely on overall scores, final answers, or broad chapter weakness to provide feedback. Students then re-study large topic areas instead of focusing on the specific misconceptions or execution issues causing poor performance.

### Trigger and Journey Placement
This problem is triggered whenever a student submits a final handwritten assignment or assessment.

Journey placement:
- Entry point: student submits handwritten work
- Upstream dependency: assignment inventory, question-to-concept mapping, and predefined diagnostic rubrics
- Core evaluation layer: AI produces concept/diagnostic judgments backed by question evidence
- Human control layer: tutor reviews, approves, or overrides AI output
- Downstream consequence: remediation quality depends on diagnosis quality
- Desired exit state: tutor-approved diagnosis, targeted remediation, and visible measurable progress

### Why Now
LLM-based evaluation has become practical enough to support production use in a `tutor-reviewed workflow` for handwritten academic submissions, making structured diagnostic evaluation and remediation generation feasible at scale.

### Cost of Inaction
If unresolved over the next 6 to 12 months:
- students will continue receiving broad rather than targeted remediation,
- tutor capacity will remain constrained by manual review burden,
- parents will continue lacking transparent visibility into concept-level weaknesses and improvement plans,
- incorrect or unfocused study effort will continue reducing learning efficiency and trust.

### Problem Statement
StudyTracer must enable tutor-reviewed, concept-first evaluation of final handwritten `math` and `science` submissions, where AI generates diagnostic judgments backed by question evidence, tutors finalize the official evaluation, and the resulting concept/chapter-level diagnosis, remarks, and remediation are made visible to the appropriate users.

### Assumptions
- AI quality is sufficient to support a tutor-reviewed workflow.
- Tutors will review every AI-generated evaluation before finalization.
- Diagnostic taxonomy will start with `concept gap`, `logical-step gap`, and `calculation/execution error`, and expand later.
- Parent visibility is a required product consideration.
- Before evaluation, each question must already have confirmed `concept mapping` and question-level `rubric definitions` for the supported diagnostic dimensions.

### Risks
- Misdiagnosis can lead to incorrect remediation and trust loss.
- Handwriting variability can reduce evaluation quality.
- Tutor review effort can become too high if AI output quality is inconsistent.
- Rubrics may fail to capture valid alternate solution paths.
- Weakly designed status models could blur meaning across diagnostic dimensions.

### Current Locked Decisions
- V1 scope is limited to handwritten `math` and `science`.
- V1 supports only `final submissions`, not iterative work-in-progress review.
- Tutors are the primary workflow operators.
- Tutor review is mandatory before evaluation finalization.
- Tutor override is available during mandatory review.
- Students and parents see only `tutor-approved outputs`.
- Parents see only `chapter-level` and `concept-level` outputs.
- The evaluation model is `concept-first and diagnostic-dimension-aware with question evidence`.
- Every question maps to one or more concepts before evaluation.
- One question can support multiple concepts.
- AI reasons exist at both `question level` and `concept level`.
- For each evaluated item, the system stores the original AI output, the final tutor-approved output, and the identity of the tutor who finalized it.
- No mandatory override-reason capture in V1.

## 3. Goals and Non-Goals

Pending.

## 4. Users and Personas

Pending.

## 5. Scope

### Locked Foundation Relevant to Scope
- Only `admin/content creators` can publish assignments/assessments to inventory in V1.
- Tutors can assign published worksheets/assignments/assessments to students.
- Each question must be fully prepared before publish with confirmed `concept mapping` and question-level `diagnostic-dimension rubric definitions` for the supported evaluation dimensions.
- Assignments and assessments share the same structure in V1 and differ only by label/workflow context.

## 6. Information Architecture

### 6A. Core Entities and Relationships

#### Authoring and Mapping Foundation
- Assignment-level metadata is human-provided.
- Question-level mapping and rubric setup is AI-assisted and human-reviewed.
- `Question` should be modeled as a first-class object in the system.
  The system should distinguish between:
  - an `assignment-bound question instance`, which is the authoritative production object used for publishing and student evaluation
  - an optional reusable `question template/pattern`, which is used for authoring-time retrieval and reuse support
- Concept mapping and rubric definition are separate objects.
- Question is the primary mapped evidence unit.
  Each question is the lowest-level academic artifact that is explicitly prepared for evaluation. Every question must carry its own confirmed concept mapping and question-level rubric definitions for the supported diagnostic dimensions. When a student submits work, the AI first reads and evaluates the response at the question level, because that is where observable evidence exists.
- Concept and diagnostic judgment are the primary evaluation decision units.
  The system does not stop at question-level correctness. Instead, it converts question-level evidence into tutor-reviewable judgments about:
  - whether a concept is `Met`, `Partially Met`, `Not Met`, or has `Insufficient Evidence`
  - what type of weakness exists within that concept across the supported diagnostic dimensions such as `concept mastery`, `logical-step correctness`, and `calculation/execution accuracy`
  These concept and diagnostic judgments are the official outputs used for reporting, remediation, and progress tracking.
- Evaluation is `concept-first and diagnostic-dimension-aware with question evidence`.
  This means the system evaluates each question using its question-level rubric definitions, produces structured evidence for the linked concepts and diagnostic dimensions, and then aggregates that evidence by concept. The final AI output is not just “this question is right or wrong,” and it is not just “this concept is weak.” It is a concept-level evaluation that is backed by question evidence and explained through diagnostic-dimension findings.
- Reusable rubric model is `reusable base with assignment-level overrides`.
  The platform should support reusable rubric building blocks so similar questions do not need to be authored from scratch every time. However, reuse should not force identical evaluation logic across all assignments. A reusable base definition can be suggested from prior patterns, and the creator can then adjust or override it for the specific assignment question being authored. This preserves reuse efficiency without sacrificing question-specific accuracy.
- Reusable templates are visible and editable only to `admin/content` roles in V1.
  In V1, shared reusable templates are governed assets rather than open-edit objects. Admin/content users can create, review, update, and manage these reusable templates so the shared library stays consistent. Tutors can benefit from AI suggestions and published content, but they do not directly edit the shared reusable template library in V1.

Evaluation model clarification:
- AI evaluates each submitted question against its confirmed question-level rubric definitions for the supported diagnostic dimensions.
- For each question, AI produces structured evidence about:
  - the linked concept(s),
  - `concept mastery`,
  - `logical-step correctness`,
  - `calculation/execution accuracy`,
  - and the reason for the judgment.
- AI then aggregates question-level evidence across all questions linked to the same concept.
- Using that evidence, AI produces a concept-level judgment and diagnostic-dimension findings for that concept.
- The output is therefore not just `concept gap` identification; it is a concept-level evaluation supported by question evidence and explained through diagnostic-dimension findings.
- Tutor review is mandatory before any evaluation becomes final or visible to students and parents.

#### Assignment Creation Flow Foundation
In V1, the creator of an assignment or assessment uploads the actual assignment questions as images or PDF and provides:
- `subject` required
- `education board` required
- `class/grade` required
- `chapter` required
- `assignment/assessment type` required
- `overall concepts tested` optional, but AI-assisted and required when the assignment clearly spans multiple concepts

This assignment-level information gives the system the academic context needed to interpret the uploaded questions more accurately. It also narrows the space for AI-generated question mappings, rubric definitions, and reuse suggestions.

In V1, this metadata must be entered before question upload so it is available when the system begins parsing uploaded questions and retrieving reusable template candidates.

For each question, the system generates:
- AI-suggested `question-to-concept mapping`
- AI-suggested `question-specific rubric/evaluation definition`
- suggested `reusable question pattern/template` matches from prior data
- AI-suggested `assignment-level concept/rubric summary`

Here:
- `AI-suggested question-to-concept mapping` means the system proposes which concept or concepts the uploaded question is testing.
- `AI-suggested question-specific rubric/evaluation definition` means the system proposes how that specific question should be evaluated across the supported diagnostic dimensions.
- `Suggested reusable question pattern/template matches` means the system surfaces prior similar question patterns whose mappings or rubric structures may be reused or adapted.
- `AI-suggested assignment-level concept/rubric summary` means the system derives an overall summary of the concepts covered in the assignment and the diagnostic structure implied by the uploaded questions.

Reusable template retrieval behavior in V1:
- Retrieval begins automatically immediately after assignment upload for all uploaded questions.
- Retrieval uses assignment-level metadata first to narrow candidates using:
  - `subject`
  - `education board`
  - `class/grade`
  - `chapter`
  - `overall concepts tested` when available
  - `question type` when inferable
- After metadata-based narrowing, the system uses AI/RAG-style similarity matching over stored reusable-template question representations to identify the top reusable template candidates for each question.
- The system shows the `top 3` reusable template candidates per question.
- If no strong reusable candidate is found, the system shows only the fresh AI-generated mapping/rubric suggestion.
- Retrieval runs automatically and also supports manual re-run by the creator.
- Retrieval runs again if assignment metadata is changed.
- Retrieval runs again if the parsed question changes or the uploaded file is replaced.
- For an uploaded question, the system derives a comparable question representation from the uploaded image/PDF; the creator does not manually provide a retrieval pattern.

Two retrieval layers operate during question authoring in V1:
- `Reusable question-template retrieval`
  This helps answer: `Have we seen a similar question before?`
  It supports reuse of prior question-level mappings and rubric structures.
- `Diagnostic-dimension-template retrieval`
  This helps answer: `What reusable scaffold should define this diagnostic dimension for this kind of question?`
  It supports construction of dimension-specific rubric definitions for the current question.

These two retrieval layers should run separately but converge into one coherent AI-generated suggestion for the creator.
- The creator should not need to manually manage two different retrieval systems.
- The creator should see one AI suggestion, optionally accompanied by a compact provenance summary.

Reusable template retrieval flow in V1:
1. Creator enters required assignment-level metadata.
2. Creator uploads assignment questions as images or PDF.
3. System parses the uploaded assignment into question units.
4. For each question, the system derives a machine-usable question representation from the uploaded file.
5. The system narrows the reusable-template search space using assignment-level metadata and inferred question type where available.
6. The system performs AI/RAG-style similarity retrieval against stored reusable-template question representations.
7. The system returns the top reusable template candidates for each question.
8. The system also generates a fresh AI mapping/rubric suggestion for each question.
9. The creator reviews reusable suggestions side by side with fresh AI-generated suggestions and can accept, edit, merge, or ignore them.
10. The creator finalizes the question setup before save/publish.

During authoring, the creator sees:
- `AI-generated mapping/rubric suggestion`
- `reused mapping/template suggestion`
- `side-by-side comparison`
- ability to choose one, edit either, merge, or create a fresh version
- ability to review and edit the AI-suggested `assignment-level concept/rubric summary` before save/publish

This means the creator is not forced to accept either the AI-generated structure or the reuse suggestion as-is. The creator can compare both, modify them, combine parts of them, or create a new finalized version when needed. The final saved assignment should therefore contain human-reviewed mappings and rubric definitions, even when AI performed most of the initial drafting.

Assignment-level concept summaries are derived from question-level mappings, while assignment-level diagnostic understanding is inferred from question-level diagnostic-dimension definitions and evidence.

#### Publishing Permissions and Ownership
- In V1, `admin/content creators` can create, edit, and publish assignments/assessments to inventory.
- In V1, `tutors` cannot publish content to inventory.
- In V1, tutors can assign published worksheets/assignments/assessments to students.
- Reusable templates are visible and editable only to `admin/content` roles in V1.

#### Persistence and Reuse Model
- The system persists question mappings and question-specific evaluation definitions at the `assignment question instance` level.
  Here, `question mappings` means the confirmed mapping from a specific uploaded question to one or more concepts.
  `Question-specific evaluation definitions` means the confirmed rubric definitions for the supported diagnostic dimensions for that specific question.
  This data is persisted because it is the official evaluation setup that the AI will later use when student submissions are evaluated against that uploaded assignment.
- `Assignment question instance` means a specific question as it exists inside a specific uploaded assignment or assessment.
  Even if a similar question appears elsewhere, the assignment question instance is the concrete version that belongs to that assignment and carries the final approved mapping and rubric definitions used for production evaluation.
- The system may also persist reusable `question pattern/template` records for future suggestion and reuse.
  These reusable records are a library layer above individual assignment questions. They help the system suggest prior mappings and rubric structures when a new uploaded question looks similar to something seen before.
- Reusable question pattern/template records are not the same as the official assignment question instance.
  The assignment question instance remains the authoritative production object used for student evaluation. Reusable templates exist to improve future authoring efficiency.
- A reusable question pattern/template should be persisted only when `admin/content` explicitly promotes a finalized assignment-question setup into the reusable library.
- Not every finalized assignment question should automatically become a reusable template in V1.
- Reuse detection is assistive, not blocking.
  The system may search for similar prior question patterns and suggest them to the creator, but publish should not depend on finding a reusable match.
- The system should not require exact question-text matching before publish.
  Two questions may be academically equivalent even if wording, numbers, formatting, or images differ. V1 should therefore use reuse suggestions to help creators, but should not force an exact text match against previously stored questions before allowing publish.
- Reuse suggestions should help creators, but publish eligibility depends only on whether the current assignment questions have complete confirmed concept mappings and question-level diagnostic-dimension rubric definitions.
- In V1, reusable template retrieval should first use assignment-level metadata filters such as `subject`, `education board`, `class/grade`, `chapter`, and available concept context, and then use AI similarity matching to surface the top candidate reusable templates during authoring.
- Diagnostic-dimension template retrieval should happen during question authoring, after assignment metadata, parsed question context, and likely concept context are available.
- Diagnostic-dimension template retrieval should be automatic, but the creator should see only a compact provenance summary rather than raw retrieval internals.
- If reusable question-template retrieval and diagnostic-dimension-template retrieval conflict, the diagnostic-dimension scaffold should take precedence for that specific dimension.
- Structured provenance about which reusable question template(s) and diagnostic-dimension template(s) influenced the AI suggestion should be stored internally.

#### Confirmed Assignment-Level Fields

| Field | Purpose | Requirement |
|---|---|---|
| `assignment_id` | Unique assignment record identifier | System-generated |
| `title` | Human-readable assignment name | Required before publish |
| `type` | `assignment` or `assessment` | Required at creation |
| `subject` | Academic subject | Required at creation |
| `education_board` | Board/curriculum identifier | Required at creation |
| `class_grade` | Grade/class level | Required at creation |
| `chapter` | Chapter grouping | Required at creation |
| `overall_concepts_tested` | Assignment-level concept summary | Optional, AI-assisted; required when the assignment spans multiple concepts |
| `author_id` | Creator identity | System-recorded |
| `publisher_id` | Publishing identity | System-recorded on publish |
| `status` | `draft` / `ready_to_publish` / `published` / `archived` | System-managed |
| `instructions` | Student/tutor instructions | Optional in current lock state |
| `question_count` | Number of questions | System-derived |
| `created_at` | Creation timestamp | System-generated |
| `updated_at` | Last update timestamp | System-generated |
| `published_at` | Publish timestamp | System-generated on publish |

Assignment-level mandatory input rules currently locked:
- At creation time, the creator must provide `subject`, `education board`, `class/grade`, `chapter`, and `assignment/assessment type`.
- `overall_concepts_tested` is optional, but AI-assisted and required when the assignment clearly spans multiple concepts.

#### Confirmed Question-Level Authoring Fields

| Field | Purpose | Requirement |
|---|---|---|
| `question_id` | Unique question record identifier | System-generated |
| `assignment_id` | Parent assignment reference | Required |
| `question_number` | Display/order within assignment | Required |
| `question_content` | Canonical question text/content | Required before publish |
| `question_image` | Image representation, if applicable | Optional |
| `subject` | Subject context | Inherited or stored |
| `education_board` | Board/curriculum context | Inherited or stored |
| `chapter` | Chapter context | Inherited or stored |
| `mapped_concepts[]` | Linked concept evidence targets | Required before publish |
| `diagnostic_dimensions[]` | Applicable evaluation dimensions | Required before publish |
| `ai_suggested_mapping` | AI-proposed concept mapping | Stored when AI suggestion exists |
| `ai_suggested_rubric_definition` | AI-proposed question evaluation definition | Stored when AI suggestion exists |
| `selected_source` | `ai` / `reused` / `merged` / `manual` | Required for provenance |
| `final_rubric_definition` | Final author-approved evaluation definition | Required before publish |
| `expected_solution_path` | Reference solution flow | Optional; system should strongly recommend for some question types |
| `alternate_methods_allowed` | Whether alternate methods are allowed | Required |
| `accepted_alternate_methods` | Explicit alternate methods | Optional |
| `status` | `incomplete` / `ready` | System-managed |

Confirmed question-level storage rule:
- The system stores both `AI suggestion` and `final selected version`.

#### Reusable Template Foundation

| Field | Purpose | Requirement |
|---|---|---|
| `template_id` | Unique reusable template identifier | System-generated |
| `template_title` | Human-readable template label | Optional |
| `subject` | Subject classification | Recommended |
| `education_board` | Board/curriculum classification | Recommended |
| `class_grade` | Grade/class classification | Recommended |
| `chapter` | Chapter classification | Recommended |
| `concepts[]` | Template concept targets | Recommended |
| `diagnostic_dimension_templates[]` | Reusable dimension definitions | Recommended |
| `reference_question_content` | Stored text or normalized representation of the template question, used for retrieval | Recommended |
| `reference_question_pattern` | AI-generated normalized academic/solution pattern associated with the template question | Recommended |
| `reference_solution_path` | Reference solving path | Optional |
| `alternate_methods_allowed` | Alternate method rule | Recommended |
| `accepted_alternate_methods` | Explicit alternate method list | Optional |
| `created_by` | Template creator identity | System-recorded |
| `approved_by` | Template approver identity | System-recorded |
| `template_status` | Template lifecycle state | System-managed |

Notes:
- `reference_question_content` is the retrievable representation used for similarity-based reuse matching.
- `reference_question_pattern` is a stored template feature generated or inferred by the system; it is not manually entered by the creator during retrieval.
- When a new question is uploaded, the system derives a comparable question representation from the uploaded file and uses it to retrieve reusable template candidates.

#### Diagnostic Dimension Template Foundation
Reusable `diagnostic_dimension_templates` should be modeled as their own first-class reusable objects in the system.

These templates are not question-ready rubrics. They are reusable evaluation scaffolds for a diagnostic dimension within a meaningful academic scope such as subject or concept family.

Minimum required fields for a reusable diagnostic-dimension template:
- `dimension_name`
- `default_status_vocabulary`
- `default_evaluation_principles[]`
- `default_rules[]`
- optional `template_guidance_text`

Diagnostic-dimension templates should be scoped by `subject` or `concept family`, not treated as one fully global template across all academic contexts.

Persistence trigger for diagnostic-dimension templates in V1:
- A diagnostic-dimension template should be persisted only when `admin/content` explicitly promotes a dimension-level scaffold into the reusable template library.

#### Final Rubric Definition Shape
`final_rubric_definition` lives inside each question object.

In stored form, `final_rubric_definition` should stay lean and rely on the parent question object for question-wide metadata, rather than duplicating outer question fields.

Minimum stored structure for `final_rubric_definition`:
- `concept_entries[]`

Each `concept_entry` should contain:
- `concept_name` or `concept_id`
- `diagnostic_dimension_rubrics[]`
- optional `reference_answer_or_expected_outcome`
- optional `accepted_alternate_approaches`
- optional `evidence_granularity_expectation`

Each `diagnostic_dimension_rubric` should be a mixed structured object with:
- `dimension_name`
- `expected_status_vocabulary`
- `evaluation_cues[]`
- `rules[]`
- optional `guidance_text`

Question-specific rubric definitions may reference which reusable diagnostic-dimension templates they were adapted from.

#### Remediation Inventory and Assignment Foundation
- In V1, remediation uses a single underlying content model with labels such as `worksheet` or `assignment`, rather than separate remediation-specific content types.
- AI should first produce structured remediation targets from evaluation output and then trigger a second retrieval phase against the published content inventory.
- Structured remediation targets in V1 should include:
  - `subject`
  - `education_board`
  - `class_grade`
  - `chapter`
  - `target_concepts[]`
  - `target_diagnostic_dimensions[]`
  - `target_status_or_weakness`
  - `remediation_rationale`
- Remediation retrieval should search the existing published inventory using those structured targets.
- Remediation retrieval should reuse the same published question-level artifacts already required for evaluation authoring, including:
  - concept mappings
  - diagnostic dimensions
  - question-level rubric definitions
  - academic metadata
- Remediation retrieval should first identify candidate source assignments and then filter/rank candidate questions within those assignments against the remediation targets.
- Tutor remediation review should show source assignments grouped separately, with recommended questions preselected within each source assignment.
- In V1, remediation may use questions from multiple source assignments.
- After tutor approval, the system should create a single student-specific `remediation assignment instance` that may contain selected questions from multiple source assignments.
- The remediation assignment instance is always student-specific in V1 and is not promoted into the shared inventory.
- Source provenance for each remediation question must be preserved internally even when the student receives one combined remediation assignment instance.
- In V1, question order inside the remediation assignment instance has no special pedagogical semantics; the student-facing order should preserve the final assembled order approved by the tutor/system.
- In V1, students do not need to see why each remediation question was selected.
- In V1, parents should see a short tutor-approved remediation rationale at the remediation level.

#### Remediation Assignment Instance Fields

| Field | Purpose | Requirement |
|---|---|---|
| `remediation_instance_id` | Unique remediation-assignment instance identifier | System-generated |
| `student_id` | Student receiving remediation | Required |
| `source_assignment_ids[]` | Source assignment references used to assemble remediation | Required |
| `selected_question_ids[]` | Selected source-question references included in remediation | Required |
| `target_concepts[]` | Concepts targeted by the remediation | Required |
| `target_diagnostic_dimensions[]` | Diagnostic weaknesses targeted by the remediation | Required |
| `tutor_approved_rationale` | Tutor-approved remediation rationale | Required for parent visibility |
| `assigned_by` | Tutor identity assigning remediation | Required |
| `assigned_at` | Assignment timestamp | System-generated |
| `status` | Remediation-assignment lifecycle state | System-managed |

### 6B. Entity States and Rules

#### Assignment and Question Lifecycle Rules
Assignment states in V1:
- `draft`
- `ready_to_publish`
- `published`
- `archived`

Question states in V1:
- `incomplete`
- `ready`

Rules:
- An assignment starts in `draft`.
- In `draft`, the creator must provide `subject`, `education board`, `class/grade`, `chapter`, and `assignment/assessment type`.
- An assignment may move to `ready_to_publish` only when every question is `ready`.
- A question is `ready` only when it has:
  - confirmed concept mapping
  - confirmed diagnostic dimensions
  - final rubric definition
  - alternate methods allowed flag
- `expected solution path` is optional and never blocks publish, though the system should strongly recommend it for question types where it improves evaluation reliability.
- An assignment may move to `published` only from `ready_to_publish`.
- Once `published`, evaluation-critical fields must not be edited in place.
- Evaluation-critical fields include:
  - question content
  - concept mapping
  - diagnostic dimensions
  - rubric definitions
- Changes to evaluation-critical fields after publish must create a new draft/version.
- Non-evaluation-critical published fields such as title, instructions, or display formatting may be edited in place.
- `archived` assignments cannot be newly assigned, but must remain available for historical reporting and prior evaluation visibility.
- If any question remains incomplete, the assignment cannot move to `ready_to_publish` or `published`.

#### Publishing and Persistence Rules
- Mappings must exist before an assignment/assessment is published.
- If a question lacks required mapping/evaluation structure, it is not publishable.
- If such a gap remains, that question is blocked from evaluation.
- The system persists the `assignment question instance` and its finalized mappings/definitions.
- Reusable `question pattern/template` data is persisted for future suggestion and reuse.
- Reuse is `suggested`, not silently auto-applied.
- Reusable mappings are saved in-assignment first and can be explicitly promoted to the reusable library.

#### Mandatory Question-Level Setup Before Publish
- `question content`
- `concept mapping`
- `diagnostic dimensions`

#### Optional Question-Level Setup Before Publish
- `expected solution path`
- `explicit accepted alternate methods`

#### Diagnostic Model
V1 evaluation dimensions are:
- `concept mastery`
- `logical-step correctness`
- `calculation/execution accuracy`

The system does not evaluate only for `concept gaps`. During evaluation, AI and tutor must also identify weakness patterns in `logical-step correctness` and `calculation/execution accuracy`.
Each of these three diagnostic dimensions has its own rubric definition, and those rubric definitions can vary by `subject`, `chapter`, `concept`, and `question`.
These dimensions are reusable at the framework level, but each question has its own question-specific evaluation definition for those dimensions.

Concept status vocabulary in V1:
- `Met`
- `Partially Met`
- `Not Met`
- `Insufficient Evidence`

Recommended diagnostic dimension vocabularies:
- `Concept Mastery`: `Met`, `Partially Met`, `Not Met`, `Insufficient Evidence`
- `Logical-Step Correctness`: `Correct`, `Partially Correct`, `Incorrect`, `Not Attempted`, `Insufficient Evidence`
- `Calculation/Execution Accuracy`: `No Issue`, `Minor Error`, `Major Error`, `Not Assessable`

#### Submission and Evaluation Rules
- In V1, each student is allowed only `one final submission` per assignment.
- V1 stores only `AI-interpreted evidence` post-submission, not raw OCR/extracted text.
- AI model version and rubric version are not stored in V1.
- Concept judgments are qualitative and are not score-aggregated in V1.
- No question weightage is assigned in V1.
- Each concept is evaluated independently using linked question evidence.
- The system must generate concept-specific judgments backed by question evidence, not a single assignment-wide met/unmet output.
- The evaluation pipeline is `question-level structured evidence -> AI concept/dimension judgment -> tutor finalization`.
- AI must produce reasons at both `question level` and `concept level`.
- During an AI evaluation session, the system evaluates questions in-session and uses the resulting question-level judgments as evidence for concept-level and diagnostic-dimension judgments.
- The system is not required to persist each question evaluation as an intermediate step during AI reasoning.
- After the AI evaluation session completes, the system persists the structured question-level outputs, concept-level outputs, and assignment-level summary for tutor review and downstream use.

#### Confirmed Submission-Level Fields

| Field | Purpose | Requirement |
|---|---|---|
| `submission_id` | Unique submission identifier | System-generated |
| `assignment_id` | Parent assignment reference | Required |
| `student_id` | Student identity | Required |
| `tutor_id` | Assigned tutor identity | Required |
| `submitted_artifacts` | Uploaded images/PDF/pages | Required |
| `submission_status` | Submission lifecycle state | System-managed |
| `submitted_at` | Submission timestamp | System-generated |
| `evaluation_status` | Evaluation lifecycle state | System-managed |
| `finalized_at` | Finalization timestamp | System-generated on finalization |

#### Confirmed Question Evidence Fields

| Field | Purpose | Requirement |
|---|---|---|
| `submission_question_id` | Submission-specific question evidence record | System-generated |
| `question_id` | Source question reference | Required |
| `ai_question_evaluation` | AI's structured evaluation object for the question across applicable concepts and diagnostic dimensions | Required |
| `final_tutor_question_evaluation` | Tutor-approved structured evaluation object for the question across applicable concepts and diagnostic dimensions | Required before finalization |
| `ai_question_evidence_summary` | AI-generated evidence summary for the question | Required |
| `linked_concepts[]` | Concepts supported by this question evidence | Required |
| `linked_dimensions[]` | Diagnostic dimensions evaluated by this question evidence | Required |
| `tutor_final_question_feedback` | Tutor-approved final feedback for the question | Required before finalization |
| `evidence_status` | Evidence review state | System-managed |

`ai_question_evaluation` and `final_tutor_question_evaluation` are structured objects, not flat scalar fields. They should capture the formal question-level evaluation result used for concept-level aggregation and tutor review.

Recommended question-level evaluation object shape:
- `concept_findings[]`
- `diagnostic_dimension_findings[]`
- `question_remarks`

Recommended diagnostic dimension finding shape:
- `dimension_name`
- `status`
- `evidence_summary`

#### Confirmed Concept Evaluation Fields

| Field | Purpose | Requirement |
|---|---|---|
| `concept_evaluation_id` | Unique concept evaluation record | System-generated |
| `submission_id` | Parent submission reference | Required |
| `concept_name` | Evaluated concept name | Required |
| `ai_concept_status` | AI-suggested concept status | Required |
| `final_concept_status` | Tutor-approved final concept status | Required before finalization |
| `supporting_question_ids[]` | Question evidence linked to this concept | Required |
| `ai_concept_reasoning_summary` | AI concept-level reasoning summary | Required |
| `final_tutor_remarks` | Tutor-approved concept remarks | Required before finalization |
| `recommended_remediation` | Remediation output for concept/dimension weakness | Optional but expected when weakness exists |
| `dimension_findings[]` | Dimension-specific findings under this concept | Required |

#### Tutor-Approved Evaluation Object Shape

| Object | Fields |
|---|---|
| `Assignment Evaluation` | `submission_id`, `student_id`, `assignment_id`, `subject`, `education_board`, `class/grade`, `chapter`, `tutor_id`, `evaluation_status`, `final_overall_summary` |
| `Concept Evaluation` | `concept_id` / `concept_name`, `concept_status`, `supporting_question_ids`, `supporting_question_evidence_summary`, `diagnostic_dimension_findings[]`, `final_tutor_remarks`, `recommended_remediation_ids` or `remediation_summary` |
| `Diagnostic Dimension Finding` | `dimension_name`, `status`, `evidence_summary` |
| `Question Evidence` | `question_id`, `linked_concepts[]`, `linked_dimensions[]`, `ai_question_evaluation`, `final_tutor_question_evaluation`, `ai_evidence_summary`, `tutor_final_question_feedback` |

Example question evidence record:

```json
{
  "submission_question_id": "SQ-101",
  "question_id": "Q-3",
  "linked_concepts": ["Newton's Second Law"],
  "linked_dimensions": [
    "Concept Mastery",
    "Logical-Step Correctness",
    "Calculation/Execution Accuracy"
  ],
  "ai_question_evaluation": {
    "concept_findings": [
      {
        "concept_name": "Newton's Second Law",
        "status": "Partially Met"
      }
    ],
    "diagnostic_dimension_findings": [
      {
        "dimension_name": "Concept Mastery",
        "status": "Partially Met",
        "evidence_summary": "Student selected the correct formula but did not apply net force direction correctly."
      },
      {
        "dimension_name": "Logical-Step Correctness",
        "status": "Incorrect",
        "evidence_summary": "Intermediate setup omitted force decomposition."
      },
      {
        "dimension_name": "Calculation/Execution Accuracy",
        "status": "No Issue",
        "evidence_summary": "No arithmetic issue in the visible steps."
      }
    ],
    "question_remarks": "Student understands the formula but struggles with structured setup."
  },
  "final_tutor_question_evaluation": {
    "concept_findings": [
      {
        "concept_name": "Newton's Second Law",
        "status": "Partially Met"
      }
    ],
    "diagnostic_dimension_findings": [
      {
        "dimension_name": "Concept Mastery",
        "status": "Partially Met",
        "evidence_summary": "Formula selection is correct, but force direction understanding is inconsistent."
      },
      {
        "dimension_name": "Logical-Step Correctness",
        "status": "Partially Correct",
        "evidence_summary": "Setup is incomplete rather than fully incorrect."
      },
      {
        "dimension_name": "Calculation/Execution Accuracy",
        "status": "No Issue",
        "evidence_summary": "No execution error observed."
      }
    ],
    "question_remarks": "Needs practice in setting up net-force problems step by step."
  },
  "ai_question_evidence_summary": "AI identified correct formula usage but weak setup reasoning.",
  "tutor_final_question_feedback": "You chose the right formula, but you need to break the problem into force setup steps before solving.",
  "evidence_status": "finalized"
}
```

## 7. User Flow Catalog

Pending.

## 8. Functional Requirements

Pending.

## 9. Non-Functional Requirements

Pending.

## 10. UX Requirements

### Locked Evaluation and Review Structure

For each concept, AI and tutor should finalize:
- `concept status`
- `supporting question evidence`
- `diagnostic dimension findings`
- `final remarks`

When meaningful weakness exists, the system should also surface `recommended remediation` tied to concept weakness, diagnostic-dimension weakness, or both.

Students should see `question-level feedback` derived from concept evidence.
Parents should see only `chapter-level` and `concept-level` tutor-approved outputs.

Tutor review hierarchy for each submission:
- Submission header with student, assignment, subject, education board, class, chapter, submission date, and finalize status
- Concept review panels
- Expandable supporting question views under each concept

Tutor review is concept-first, with question evidence available underneath.

Locked tutor review interaction model in V1:
- The primary review unit is the `concept panel`.
- Supporting question evidence should remain expandable underneath each concept and should be reviewed when needed rather than by default.
- The tutor review workflow should support `bulk approve all AI suggestions, then edit exceptions`.
- Tutors should also be able to review and edit concepts individually before final publish.
- The official finalization point is at the `submission level`, even though concept-level review happens within the workflow.
- Partial tutor progress should be savable before final publish.
- If AI output is broadly wrong for a submission, the tutor should still be able to edit the evaluation manually in place.
- Low-confidence or unclear-handwriting cases should appear as inline warning flags in the tutor review UI.
- Alternate valid solving methods should be handled through normal tutor review and override, without a separate mandatory workflow in V1.

Tutor-editable objects in V1:
- `final concept status`
- `concept-level diagnostic dimension findings`
- `final concept remarks`
- `final question-level evaluation`
- `final question-level learner-facing feedback`
- `remediation suggestion/assignment`

Each concept review panel should show:
- `Concept Name`
- `AI Suggested Concept Status`
- `Tutor Editable Final Concept Status`
- `Supporting Questions`
- `AI Concept-Level Reasoning Summary`
- `Diagnostic Dimension Findings`
- `Final Tutor Remarks`
- optional `Recommended Remediation`

### Locked Student Submission Model

Student submission in V1 should be `mobile-first`, even though the product is a web app.

Submission model in V1:
- Students submit answers `per question`, not as one unstructured full-assignment packet.
- Each question supports `multiple images` for the answer.
- Upload order is the default page order, but students can reorder images before final submit.
- Students should have an explicit `not attempted` option per question.
- Students should be able to move an uploaded image from one question to another before final submission.
- Each question should have a per-question submission state so answer completeness is visible before final submit.

Recommended per-question submission states in V1:
- `no answer uploaded`
- `uploaded`
- `order not confirmed`
- `ready`
- `not attempted`

Student upload UX requirements in V1:
- Support direct browser `camera capture` per question.
- Support `gallery/file upload` per question.
- Show thumbnail previews for uploaded answer images.
- Allow delete, replace, and reorder actions before final submit.
- Preserve upload order as the default sequence for multi-image answers.
- Require explicit order confirmation when a question has multiple uploaded images.

Cross-device and handoff behavior in V1:
- Student submission UX should be optimized for `mobile-first` use.
- If a student starts on desktop, the product should support optional `desktop-to-phone handoff`.
- Handoff may use a QR code, link, or equivalent session-continuation mechanism.
- Uploaded answers should sync across devices within the same submission session.

Answer-to-question mapping reliability rule in V1:
- Per-question submission is the primary mechanism for exact answer-to-question association.
- The system should avoid relying on inference when the student has already uploaded answers into explicit question slots.
- If the system still detects ambiguity within a question submission, that ambiguity should be surfaced for tutor review rather than silently guessed.

## 11. Integrations and Dependencies

### Locked Dependencies
- Assignment/worksheet inventory
- Question-to-concept mapping layer
- Reusable question/template library
- Handwritten submission ingestion

## 12. Risks and Mitigations

### Locked Risks
- Misdiagnosis may lead to incorrect remediation and loss of trust.
- Handwriting variability may reduce evaluation quality.
- Tutor review burden may become too high if AI output quality is inconsistent.
- Weak or inconsistent mapping/rubric authoring may reduce downstream evaluation quality.

## 13. Acceptance Criteria Matrix

Pending.

## 14. Open Questions and Decision Log

### Open Questions
- Exact lifecycle-state validation rules for assignment-level and question-level fields
- Exact tutor review UI interactions
- Expanded diagnostic taxonomy beyond the V1 dimensions
- Exact remediation inventory structure

### Locked Decisions
- AI suggestions and reusable-template suggestions are shown side by side.
- The creator can pick one, edit, or merge.
- `Expected solution path` is optional, but the system should strongly recommend it for certain question types.
- Only one final student submission per assignment is allowed in V1.
- Only AI-interpreted evidence is stored post-submission in V1.
- AI model version and rubric version are not stored in V1.
- Concept judgments are qualitative, not score-aggregated.
- Each concept is evaluated independently using linked question evidence.
- The system must suggest remediation for `Partially Met` and `Not Met` concepts and for meaningful weakness in diagnostic dimensions.
- Remediation in V1 uses `existing assignment/worksheet from inventory`.
- Recommended remediation combines `linked remediation objects` and `short tutor-approved free-text rationale`.
- AI suggests remediation and the tutor approves or edits it.
- Tutor may choose whether to assign a linked remediation object.
- The system should suggest remediation to the tutor and student for both `Partially Met` and `Not Met` concepts.
- Remediation is not limited to concept clarity; it should also address `logical-step` and `calculation/execution` weaknesses when applicable.

### Remediation Recommendation Object
- `target_type` (`concept` / `dimension` / `both`)
- `target_name`
- `trigger_status`
- `suggested_linked_assignment_ids[]`
- `tutor_approved_linked_assignment_ids[]`
- `tutor_approved_rationale`
- `assignment_decision` (`assigned` / `suggested_only` / `not_assigned`)
