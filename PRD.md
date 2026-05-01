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

### 3.1 Business Goals

#### MUST
- Build a reusable authoring and evaluation foundation that can scale across future content volume without requiring every assignment question to be configured from scratch.
- Enable high-precision, evidence-backed diagnostic evaluation at the `concept` and `diagnostic-dimension` level for handwritten `math` and `science` final submissions.

#### SHOULD
- Reduce tutor effort over time through structured AI-assisted review, while accepting that early V1 may prioritize evaluation consistency and diagnosis quality over immediate review-time reduction.
- Improve the quality and specificity of remediation recommendations by linking tutor-approved concept and diagnostic-dimension weaknesses to targeted next-step assignments.

#### COULD
- Establish directional operating metrics and baseline instrumentation for later KPI-setting, while deferring numeric success thresholds to a future operating plan.

### 3.2 User Outcome Goals

#### Tutors
#### MUST
- Tutors must receive AI-generated draft evaluations that are specific enough to review and finalize at the `concept` and `diagnostic-dimension` level, with linked question evidence.
- Tutors must retain full control to review, edit, override, and finalize all official evaluation outputs.

#### SHOULD
- Tutors should be able to complete review with greater consistency and less manual reconstruction of student reasoning than a fully manual workflow.

#### Students
#### MUST
- Students must receive tutor-approved, targeted diagnostic feedback that identifies what went wrong at the concept and diagnostic-dimension level.
- Students must receive more targeted next-step remediation than broad chapter-level restudy guidance.

#### SHOULD
- Student-facing outputs should make weaknesses actionable without exposing internal AI uncertainty or non-finalized system judgments.

#### Parents
#### SHOULD
- Parents should receive tutor-approved chapter-level and concept-level visibility into weak areas, remarks, and remediation rationale.

### 3.3 Product Outcome Goals

#### MUST
- The product must support both `authoring outcomes` and `evaluation outcomes` as first-class goals in V1.
- Authoring outcomes must ensure that published assignments contain question-level concept mappings and diagnostic-dimension rubric definitions that are complete enough for downstream evaluation.
- Evaluation outcomes must ensure that each final submission can produce tutor-reviewable concept judgments, diagnostic-dimension findings, and remediation suggestions backed by question evidence.
- When evidence quality is weak, ambiguous, or incomplete, the product must surface uncertainty clearly and preserve tutor control rather than silently asserting overconfident judgments.

#### SHOULD
- The product should improve downstream reuse by allowing finalized authoring structures to inform future AI suggestions and governed reusable templates.
- The product should make remediation generation a reliable downstream consequence of diagnosis quality, not an isolated workflow.

### 3.4 Non-Goals

#### MUST NOT
- V1 must not function as a fully autonomous grading or evaluation system.
- V1 must not finalize official evaluation outputs without tutor review.
- V1 must not support iterative draft review or work-in-progress feedback loops.
- V1 must not provide real-time step-by-step tutoring or interactive problem-solving assistance.
- V1 must not expand beyond handwritten `math` and `science` in this release.
- V1 must not include institution-wide analytics or broad administrative reporting as a release goal.
- V1 must not include adaptive learning-path orchestration beyond targeted remediation suggestion and assignment.

#### SHOULD NOT
- V1 should not optimize primarily for immediate tutor time savings if doing so weakens diagnostic precision or remediation quality.
- V1 should not treat parent visibility as a primary release success criterion ahead of tutor workflow quality and diagnosis quality.

### 3.5 Release Positioning

#### MUST
- V1 must be positioned primarily as a `diagnostic intelligence product`.

Positioning clarification:
- The tutor-governed workflow is an enabling control layer that ensures quality and trust.
- Remediation is a downstream action layer that depends on the quality of tutor-approved diagnosis.
- The core differentiated value in V1 is evidence-backed concept and diagnostic-dimension evaluation of handwritten submissions.

### 3.6 Assumptions

- High-quality reusable authoring structures will improve both future content creation efficiency and downstream evaluation consistency.
- Early tutor adoption is more likely if the system produces diagnostically useful draft outputs, even if tutors still make frequent edits in V1.
- More precise concept and diagnostic-dimension diagnosis will produce more useful remediation recommendations than chapter-level weakness labeling.
- Numeric success targets can be defined later once baseline operational data exists.

### 3.7 Risks

- If tutor editing remains too frequent for too long, the reusable foundation may be strong but workflow adoption may stall.
- If authoring quality is inconsistent, evaluation precision and remediation quality will degrade regardless of AI capability.
- If uncertainty signaling is too weak, tutors may overtrust incorrect AI judgments.
- If uncertainty signaling is too aggressive, tutors may lose confidence in the system's usefulness.
- If remediation suggestions are specific but low quality, the product may appear diagnostically strong but instructionally weak.

### 3.8 Unresolved Decisions

- Numeric KPI thresholds for tutor efficiency, evaluation precision, and remediation usefulness are deferred to a future operating plan.
- The exact measurement method for `diagnostic precision` is not yet defined and may require a tutor-agreement or benchmark-review framework.
- The exact measurement method for `remediation quality` is not yet defined and may require a later completion/outcome loop.

## 4. Users and Personas

### 4.1 Actor Model

Official V1 actors:
- `admin/content creator`
- `tutor`
- `student`
- `parent`

Role-model decisions locked for V1:
- `admin` and `content creator` are a single persona in V1.
- `tutor` is a single role in V1; no tutor sub-roles exist in the release scope.
- Each submission has exactly `one assigned tutor`.
- Parent access is `read-only`.
- One parent account may be linked to `multiple students`.
- Additional actors such as `school admin`, `internal QA reviewer`, `content approver` as a separate role, and `operations/support staff` are out of scope as formal V1 personas.

### 4.2 Persona Priority

Persona priority for V1:
1. `Tutor` as the primary persona
2. `Admin/Content Creator` as secondary but operationally critical
3. `Student` as downstream end user
4. `Parent` as read-only visibility stakeholder

This priority order should drive workflow design, requirement tradeoffs, and future diagram extraction.

### 4.3 Persona Definitions and JTBD

#### Persona: Tutor

Primary role in V1:
- Review AI output efficiently and finalize accurate evaluation.

Primary jobs-to-be-done:
- Review AI-generated concept and diagnostic-dimension evaluations against question evidence.
- Edit, override, and finalize official submission outcomes.
- Approve, modify, or assign targeted remediation based on tutor-approved diagnosis.

Permissions:
- View and assign all published inventory in V1 unless a future business access constraint is introduced.
- Review student submissions assigned to them.
- Edit question-level and concept-level evaluation outputs during tutor review.
- Finalize submission-level evaluation outputs.
- Approve and assign student-specific remediation assignments.

Constraints:
- Tutors cannot publish assignments or assessments to inventory.
- Tutors cannot create or publish new official versions of published assignments.
- Tutors resolve poor AI output or ambiguous evidence within the review workflow; no separate escalation role exists in V1.

Success criteria for this persona:
- Can review AI output with sufficient specificity to finalize accurate concept and diagnostic-dimension judgments.
- Can preserve tutor control when AI confidence is weak or evidence is ambiguous.
- Can assign targeted remediation without leaving the core review workflow.

#### Persona: Admin/Content Creator

Primary role in V1:
- Own both publishable content creation and reusable template/rubric quality.

Primary jobs-to-be-done:
- Create high-quality publishable assignments and assessments.
- Review and finalize question-level concept mappings and rubric definitions before publish.
- Maintain the governed reusable template and diagnostic-dimension scaffold library.
- Create and publish new official versions when evaluation-critical changes are required.

Permissions:
- Create, edit, and publish assignments and assessments to inventory.
- Review AI-generated and reusable-template-generated authoring suggestions.
- Accept, edit, merge, or ignore authoring suggestions before finalizing question setup.
- Explicitly promote finalized patterns into reusable template libraries.
- Create and publish new official versions of content.

Constraints:
- Shared reusable templates are governed assets and remain under admin/content control in V1.
- Publish eligibility depends on completed question-level mapping and rubric setup, not on tutor action.

Success criteria for this persona:
- Can prepare production-ready content that supports reliable downstream evaluation.
- Can improve reuse quality over time without losing assignment-specific accuracy.

#### Persona: Student

Primary role in V1:
- Submit handwritten work reliably.

Primary jobs-to-be-done:
- Upload handwritten answers per question with correct answer-to-question association.
- Mark questions as `not attempted` when applicable.
- Review tutor-approved concept-status-driven feedback and assigned remediation.
- Complete assigned remediation submissions.

Permissions:
- Access assigned assignments and remediation assignments intended for them.
- Upload, reorder, replace, move, and submit per-question answer images before final submit.
- View tutor-approved concept status, tutor-approved feedback, and assigned remediation.

Constraints:
- Students may submit only one final submission per assignment in V1.
- Students do not see non-finalized AI outputs.
- Students do not see internal uncertainty handling or non-approved diagnostic drafts.

Success criteria for this persona:
- Can submit answers accurately from a mobile-first workflow.
- Can understand tutor-approved weaknesses through structured concept status plus tutor-approved feedback.
- Can identify next-step remediation assigned to them.

#### Persona: Parent

Primary role in V1:
- Monitor weak areas and tutor remarks.

Primary jobs-to-be-done:
- Review tutor-approved chapter-level and concept-level weak areas for linked students.
- Review tutor-approved remarks, remediation rationale, and assigned remediation visibility.

Permissions:
- Read-only access to linked students' approved outputs.
- Visibility into chapter-level weaknesses, concept-level weaknesses, tutor remarks, remediation rationale, and assigned remediation items.

Constraints:
- Parents cannot edit evaluation outputs, submit work, or assign remediation.
- Parents do not see question-level raw evidence or internal AI review states.

Success criteria for this persona:
- Can understand where a student is struggling.
- Can see what remediation has been assigned without entering the tutor workflow.

### 4.4 Role-Based Visibility and Permission Summary

| Persona | Create/Publish Content | Assign Content | Submit Work | Review/Finalize Evaluation | View Student Outputs | View Remediation |
|---|---|---|---|---|---|---|
| `Admin/Content Creator` | Yes | No | No | No | Limited, only as needed for content/admin context | No direct learner workflow role |
| `Tutor` | No | Yes | No | Yes | Yes, for assigned students/submissions | Yes, can approve/assign |
| `Student` | No | No | Yes | No | Yes, own tutor-approved outputs only | Yes, own assigned remediation |
| `Parent` | No | No | No | No | Yes, linked students' approved chapter/concept outputs only | Yes, linked students' assigned remediation visibility |

### 4.5 Role-Based Behavior Rules

- Tutor is the primary operating persona for the evaluation workflow.
- Admin/content creator is the primary operating persona for the authoring and publishing workflow.
- Students are downstream participants in submission and remediation workflows, not in evaluation approval workflows.
- Parents are visibility-only stakeholders in V1.
- Tutor-owned evaluation and admin/content-owned publishing are intentionally separated to preserve governance and quality control.

### 4.6 Assumptions

- A single-tutor ownership model is sufficient for V1 review accountability.
- Parent read-only visibility is enough to support parent trust and follow-up in the first release.
- Tutor access to all published inventory is acceptable in V1 because no institutional access segmentation is currently defined.
- Formal personas outside the core four actors are not required to make V1 workable.

### 4.7 Risks

- If tutor permissions are too broad in future multi-organization contexts, inventory access may need to be constrained later.
- If parent expectations extend beyond visibility into action-taking or messaging, the read-only model may feel incomplete.
- If content governance needs separation of duties later, the combined admin/content persona may need to split into multiple roles.
- If one-tutor ownership proves operationally limiting, later collaborative review flows may be required.

### 4.8 Unresolved Decisions

- No blocking unresolved persona decisions remain for V1.
- Future organizational access segmentation for tutors may need to be revisited if the product expands to multi-institution deployment.

## 5. Scope

### Locked Foundation Relevant to Scope
- Only `admin/content creators` can publish assignments/assessments to inventory in V1.
- Tutors can assign published worksheets/assignments/assessments to students.
- Each question must be fully prepared before publish with confirmed `concept mapping` and question-level `diagnostic-dimension rubric definitions` for the supported evaluation dimensions.
- Assignments and assessments share the same structure in V1 and differ only by label/workflow context.

### 5.1 Release Boundary

V1 is a `full closed-loop` release that includes:
- authoring and publishing of assignments/assessments
- reusable-template and diagnostic-dimension scaffold governance
- tutor assignment of published work
- student handwritten submission
- AI-assisted evaluation
- tutor review and finalization
- remediation suggestion, approval, and assignment
- student remediation submission
- remediation evaluation and tutor finalization
- student and parent visibility into tutor-approved outputs

The release is positioned as a tutor-governed `diagnostic intelligence product` with downstream remediation execution, not as a general-purpose learning platform or autonomous grading system.

### 5.2 In-Scope

#### Authoring and Publishing
- Admin/content creator creation of assignments and assessments for handwritten `math` and `science`.
- Assignment-level metadata entry required for authoring context.
- AI-assisted question parsing, concept mapping, rubric suggestion, and reusable-template suggestion.
- Human review, edit, merge, and finalization of question mappings and rubric definitions before publish.
- Publishing of prepared assignments/assessments to shared inventory.
- Creation of new official content versions by `admin/content creator` when evaluation-critical changes are required.

#### Reusable Foundation and Governance
- Reusable question-template retrieval during authoring.
- Diagnostic-dimension template retrieval during authoring.
- Admin/content governance of reusable templates and diagnostic-dimension scaffolds.
- Explicit promotion of finalized patterns into reusable libraries.
- Lightweight operational visibility for admin/content users into draft readiness and incomplete setup states.

#### Tutor Workflow
- Tutor assignment of published inventory to students.
- Tutor review of AI-generated concept-level and question-level evaluation outputs.
- Tutor editing, override, save-progress, and submission-level finalization.
- Tutor review, approval, edit, and assignment of remediation.
- Lightweight tutor follow-up visibility across prior submission/remediation context sufficient to support continuity.

#### Student Workflow
- Mobile-first per-question handwritten answer submission.
- Multiple-image answer upload per question, reorder, replace, delete, and move-before-submit behavior.
- Explicit `not attempted` handling per question.
- Submission of one final assignment response per assignment in V1.
- Submission of assigned remediation work.
- Visibility into tutor-approved concept status, tutor-approved feedback, and assigned remediation.

#### Parent Workflow
- Read-only visibility into tutor-approved chapter-level and concept-level outputs.
- Visibility into tutor-approved remarks, remediation rationale, and assigned remediation items.
- Lightweight chronological history of tutor-approved assignments and remediation items with their concept statuses.

#### Evaluation and Remediation
- AI-assisted question-level evidence generation.
- AI-assisted concept-level and diagnostic-dimension evaluation generation.
- Tutor-governed finalization of all official evaluation outputs.
- AI-assisted remediation suggestion based on tutor-reviewed concept and diagnostic-dimension weaknesses.
- Assembly of student-specific remediation assignment instances from published inventory.
- Evaluation and tutor finalization of remediation submissions using the same concept-first diagnostic model.

#### Graceful Handling Within Scope
- Graceful handling of common real-world input quality problems, including poor handwriting, ambiguous evidence, missing answer evidence, low-confidence interpretation, and answer-to-question ambiguity.
- Uncertainty signaling and advisory warnings for tutors when evidence quality is weak.
- Rejection or prevention of unsupported subjects outside handwritten `math` and `science`.

### 5.3 Out of Scope

- Autonomous evaluation finalization without tutor review.
- Iterative draft review or work-in-progress feedback.
- Real-time tutoring, interactive solving help, or student study-assistant workflows.
- Support for subjects beyond handwritten `math` and `science`.
- Institution-wide analytics dashboards or administrative reporting surfaces.
- Parent messaging, acknowledgements, approval actions, or other parent-interactive workflows.
- Tutor collaboration, co-review, or multi-reviewer evaluation workflows.
- Broad tutor case-management or intervention-planning workspaces.
- Student progress workspaces, study planning, or broad learning-history features beyond assigned work and approved feedback.
- Advanced trend analytics beyond lightweight chronological history.
- Advanced content governance such as approval chains, bulk editing, or governance dashboards.
- Broad automation such as AI auto-approval, auto-publish, or low-risk auto-finalization.
- Adaptive learning-path orchestration beyond targeted remediation suggestion and assignment.

### 5.4 Release Boundary Clarifications

- `Assignments` and `assessments` are both in scope and share one common structural model in V1.
- `Remediation` is not suggestion-only; the full remediation assignment lifecycle is in scope through tutor-approved evaluation and closure.
- `Parent history` in scope means chronological visibility into prior tutor-approved assignments/remediation and their concept statuses, not advanced trend analysis or predictive progress scoring.
- `Tutor follow-up visibility` in scope means lightweight continuity context, not a full longitudinal case-management product.
- `Admin/content operational visibility` in scope means basic readiness and incomplete-setup visibility, not advanced governance analytics.
- `Assistive AI` in scope means suggestion, evaluation drafting, retrieval, and uncertainty signaling only; AI does not become the final authority for official outputs.

### 5.5 Scope Assumptions

- A full closed loop is necessary to validate whether diagnosis quality meaningfully improves remediation quality.
- Lightweight historical visibility is sufficient for parents and tutors in V1 without requiring deeper progress analytics.
- Content governance can remain lightweight in V1 while still protecting quality through role restrictions and publish gates.
- Common input-quality problems must be treated as normal operating conditions, not exceptional edge cases outside the product boundary.

### 5.6 Scope Risks

- Full closed-loop scope increases delivery complexity because remediation introduces a second evaluation cycle.
- Lightweight historical visibility may create stakeholder demand for deeper progress analytics sooner than planned.
- Broad tutor inventory access may become misaligned if future deployments introduce institutional segmentation.
- Graceful handling of poor-quality inputs may still create review burden if ambiguity frequency is high.

### 5.7 Unresolved Scope Decisions

- No blocking scope decisions remain for V1.
- Future expansion decisions remain open for organizational access segmentation, advanced trend analytics, and broader tutor or parent workspaces.

## 6. Information Architecture

### 6.0 Product Workspaces and Navigation Hierarchy

V1 uses a `hybrid workspace model` with role-based top-level product areas and function-specific modules within those areas.

Official top-level workspaces in V1:
- `Content Authoring`
- `Template Library`
- `Tutor Workspace`
- `Student Workspace`
- `Parent Workspace`

#### Content Authoring
Primary modules:
- `Assignments List`
- `Assignment Metadata Setup`
- `Question Parsing and Review`
- `Question Mapping and Rubric Editor`
- `Assignment Summary and Publish`
- `New Version Draft`

Purpose:
- Used by `admin/content creator` to create, prepare, version, and publish assignments/assessments.

#### Template Library
Primary modules:
- `Question Templates`
- `Diagnostic-Dimension Templates`

Purpose:
- Used by `admin/content creator` to govern reusable authoring assets.

#### Tutor Workspace
Primary modules:
- `Assignments to Students`
- `Submissions Queue`
- `Submission Review Dashboard`
- `Remediation Review and Assignment`
- `Student History / Follow-up`

Purpose:
- Used by tutors to assign work, review AI output, finalize evaluation, and assign remediation.

#### Student Workspace
Primary modules:
- `My Work`
- `Work Detail / Submission`
- `Feedback and Results`

Information grouping rules:
- `My Work` is a unified work list that includes `assignment`, `assessment`, and `remediation` items using explicit `work_type` labels.
- A separate remediation-only student area is not required in V1.

Purpose:
- Used by students to complete assigned work, submit answers, and review tutor-approved outputs.

#### Parent Workspace
Primary modules:
- `Student Selector`
- `Current Results Summary`
- `Remediation Visibility`
- `Chronological History`

Purpose:
- Used by parents for read-only visibility into approved results and prior work history for linked students.

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
- Reusable question-template candidates should be shown only if they cross a fixed global minimum match threshold in V1.
- If no strong reusable candidate is found, the system shows only the fresh AI-generated mapping/rubric suggestion.
- Retrieval runs automatically and also supports manual re-run by the creator.
- Retrieval runs again if assignment metadata is changed.
- Retrieval runs again if the parsed question changes or the uploaded file is replaced.
- For an uploaded question, the system derives a comparable question representation from the uploaded image/PDF; the creator does not manually provide a retrieval pattern.
- The minimum match threshold should be system-controlled and not user-configurable in V1.

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
- Diagnostic-dimension-template retrieval should also use thresholding in V1 so that weak scaffold matches do not influence the AI-generated rubric suggestion.

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

In V1, the system should store a derived `assignment concept coverage summary`.
- This is a stored derived summary, not an independently authored object.
- The creator may review it, but may not edit it directly.
- Minimum structure:
  - `assignment_id`
  - `subject`
  - `education_board`
  - `class_grade`
  - `chapter`
  - `concept_summaries[]`
- Each `concept_summary` should contain:
  - `concept_name` or `concept_id`
  - `linked_question_ids[]`
  - `linked_question_count`
  - `covered_diagnostic_dimensions[]`
  - optional `summary_notes`

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
- After tutor approval, the system should create a single student-specific `work instance` with `work_type = remediation` that may contain selected questions from multiple source assignments.
- A remediation `work instance` is always student-specific in V1 and is not promoted into the shared inventory.
- Source provenance for each remediation question must be preserved internally even when the student receives one combined remediation work instance.
- In V1, question order inside a remediation work instance has no special pedagogical semantics; the student-facing order should preserve the final assembled order approved by the tutor/system.
- In V1, students do not need to see why each remediation question was selected.
- In V1, parents should see a short tutor-approved remediation rationale at the remediation level.

#### Work Instance Foundation

The system should model a first-class student-specific `work instance` object for assigned work in V1.

This common model should represent:
- normal assigned `assignments`
- normal assigned `assessments`
- student-specific `remediation`

Purpose:
- A `published assignment/assessment` is the reusable inventory object.
- A `work instance` is the student-specific assigned occurrence of that content.
- A `submission` is the student's final answer package tied to that work instance.

Relationship rules:
- One `work instance` may reference one or more source assignments depending on `work_type`.
- One `work instance` supports at most `one final submission` in V1.
- `assignment`, `assessment`, and `remediation` remain visible as explicit `work_type` labels even though they share one common student-facing model.

#### Work Instance Fields

| Field | Purpose | Requirement |
|---|---|---|
| `work_instance_id` | Unique student-specific work instance identifier | System-generated |
| `student_id` | Student receiving remediation | Required |
| `assigned_by` | Tutor identity assigning the work instance | Required |
| `work_type` | `assignment` / `assessment` / `remediation` | Required |
| `source_assignment_ids[]` | Source assignment references used to construct the work instance | Required |
| `selected_question_ids[]` | Selected source-question references included in the work instance | Optional for normal assignment/assessment; required when the work instance is question-selective |
| `assigned_at` | Assignment timestamp | System-generated |
| `due_at` | Optional due timestamp | Optional |
| `assigned_by` | Tutor identity assigning remediation | Required |
| `status` | Work-instance lifecycle state | System-managed |
| `submission_id` | Linked final submission for the work instance | Optional until submitted |
| `target_concepts[]` | Concepts targeted by the work instance when used for remediation | Optional for normal assignment/assessment; required for remediation |
| `target_diagnostic_dimensions[]` | Diagnostic weaknesses targeted by the work instance when used for remediation | Optional for normal assignment/assessment; required for remediation |
| `tutor_approved_rationale` | Tutor-approved rationale for remediation or assignment context | Optional for normal assignment/assessment; required for remediation parent visibility |
| `attempt_completeness` | Attempt completeness marker for the work instance | System-managed |

Notes:
- For normal assignments and assessments, `source_assignment_ids[]` will usually contain exactly one published source assignment.
- For remediation, `source_assignment_ids[]` may contain multiple source assignments and `selected_question_ids[]` becomes essential.
- `assigned_by` should be stored once as the tutor identity responsible for assigning the student-facing work instance.
- The `work instance` is the primary object surfaced in student `My Work`, tutor assignment/follow-up views, and parent chronological history.

#### Evaluation Record Foundation

The system should create a first-class `evaluation record` object for each submitted work instance after evaluation begins.

Purpose:
- The evaluation record is the official evaluated outcome container for one student-specific work instance.
- It provides a single final evaluated artifact linking submission, question evidence, concept evaluations, final summary, and finalization metadata.

Relationship rules:
- One `work instance` may have at most one final `submission` in V1.
- One final `submission` may produce one official `evaluation record`.
- Student, parent, and tutor approved-output surfaces should anchor on the `evaluation record`, not assemble final state ad hoc from scattered lower-level records.

Minimum evaluation record fields:
- `evaluation_record_id`
- `work_instance_id`
- `submission_id`
- `student_id`
- `tutor_id`
- `work_type`
- `evaluation_status`
- `final_overall_summary`
- `finalized_at`
- `linked_concept_evaluation_ids[]`
- `linked_question_evidence_ids[]`

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

#### Work Instance Lifecycle Rules
Work instance states in V1:
- `assigned`
- `in_progress`
- `submitted`
- `evaluated`
- `closed`

Rules:
- `assigned` means the tutor has assigned the work instance and the student has not started work.
- `in_progress` begins only when the student starts answer upload/work activity.
- `submitted` means the student has submitted final answers for the work instance.
- `evaluated` means tutor-approved evaluation of the remediation submission is complete.
- `closed` is reached automatically immediately after tutor-approved evaluation in V1.
- A work instance must not be reopened or reassigned after evaluation/closure.
- If more remediation is needed, the system must create a new student-specific work instance with `work_type = remediation`.

Attempt completeness in V1:
- `attempt_completeness` should be stored as a separate top-level field on the work instance.
- Supported values:
  - `not_attempted`
  - `partially_attempted`
  - `fully_attempted`
- `attempt_completeness` should be system-determined in V1 based on whether assigned questions received student answers or were explicitly left not attempted.

Work instance type rules:
- `assignment`, `assessment`, and `remediation` share one common lifecycle model in V1.
- `work_type` affects source-content structure, rationale requirements, and parent visibility semantics, but does not require a separate top-level lifecycle state model.
- For `remediation`, `target_concepts[]`, `target_diagnostic_dimensions[]`, and `tutor_approved_rationale` are required.
- For normal `assignment` and `assessment`, those remediation-specific fields may be empty.

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
| `work_instance_id` | Parent student-specific work instance reference | Required |
| `assignment_id` | Parent assignment reference | Required |
| `student_id` | Student identity | Required |
| `tutor_id` | Assigned tutor identity | Required |
| `submitted_artifacts` | Uploaded images/PDF/pages | Required |
| `submission_status` | Submission lifecycle state | System-managed |
| `submitted_at` | Submission timestamp | System-generated |
| `evaluation_status` | Evaluation lifecycle state | System-managed |
| `finalized_at` | Finalization timestamp | System-generated on finalization |

Submission relationship note:
- `assignment_id` remains useful for source-content lineage, but the primary operational parent of a submission is `work_instance_id`.

#### Confirmed Question Evidence Fields

| Field | Purpose | Requirement |
|---|---|---|
| `submission_question_id` | Submission-specific question evidence record | System-generated |
| `question_id` | Source question reference | Required |
| `submitted_answer_images[]` | Original student-submitted answer images for the question, preserved in submitted order | Required unless the question is explicitly `not attempted` |
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
- Submission summary dashboard
- Concept review panels
- Expandable supporting question views under each concept

Tutor review is concept-first, with question evidence available underneath.

Locked tutor review interaction model in V1:
- The default tutor landing view is a `submission summary dashboard`.
- The submission summary dashboard should surface:
  - overall submission status
  - concept list with AI-suggested statuses
  - flagged low-confidence / unclear-handwriting items
  - unanswered / not-attempted questions
  - remediation suggestion presence
  - submission-level readiness for final publish
- The primary review unit is the `concept panel`.
- The dashboard should recommend a concept review order, but tutors may open any concept in any order.
- Recommended review order should prioritize flagged items first and then weaker concepts by AI-indicated severity.
- Supporting question evidence should remain expandable underneath each concept and should be reviewed when needed rather than by default.
- When a tutor opens a concept panel, flagged supporting questions should be expanded by default and non-flagged supporting questions should remain collapsed.
- The tutor review workflow should support `bulk approve all AI suggestions, then edit exceptions`.
- Tutors should also be able to review and edit concepts individually before final publish.
- The official finalization point is at the `submission level`, even though concept-level review happens within the workflow.
- Partial tutor progress should be savable before final publish.
- Concept-level edits should auto-save as fields are changed.
- Question-level edits inside supporting questions should also auto-save.
- If AI output is broadly wrong for a submission, the tutor should still be able to edit the evaluation manually in place.
- Low-confidence or unclear-handwriting cases should appear as inline warning flags in the tutor review UI.
- Alternate valid solving methods should be handled through normal tutor review and override, without a separate mandatory workflow in V1.
- Low-confidence and unclear-handwriting flags are advisory only in V1 and do not block final publish.
- Concept-level review/approval is required before final publish, but question-level explicit review is not required for every question.
- If question-level evaluation is edited, the system should auto-refresh affected concept-level outputs to keep derived summaries consistent.
- Tutors may directly edit concept-level status or remarks without first editing question-level evidence.
- If final concept-level judgment diverges from currently derived question-level evidence, the system should flag the divergence, but it should remain advisory and must not block final publish.
- Save-progress should allow incomplete review states, including partially reviewed concepts and partial question-level edits.
- Minimum gate for final publish of a tutor-reviewed evaluation:
  - every concept in the submission must have a final concept status
  - required tutor-facing final fields must be present where applicable
  - question-level explicit review is not required for every question
  - advisory flags and divergence warnings do not block publish

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
- `Original Student-Submitted Answer Images` for each supporting question, preserved in submitted order and viewable in larger preview
- `AI Concept-Level Reasoning Summary`
- `Diagnostic Dimension Findings`
- `Final Tutor Remarks`
- optional `Recommended Remediation`

Concept-level completion rules in V1:
- The system should show a concept-level completion indicator.
- A concept is complete for final publish when it has:
  - `final_concept_status`
  - `final_diagnostic_dimension_findings[]`
  - `final_tutor_remarks`
  - `remediation_decision`

Submission-level completion rules in V1:
- The system should show a submission-level readiness indicator derived from concept-level completion.
- The tutor should be able to see which concepts remain incomplete before final publish.

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
