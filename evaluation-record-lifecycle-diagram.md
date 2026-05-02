# Evaluation Record Lifecycle Diagram

Source: `PRD.md` Section 8, `Lifecycle and State Model`

```mermaid
stateDiagram-v2
    [*] --> not_started
    not_started --> ai_generated: system generates draft
    ai_generated --> in_tutor_review: tutor opens review
    in_tutor_review --> finalized: tutor finalizes

    note right of not_started
      Guard:
      final submission must exist
    end note

    note right of in_tutor_review
      Guard:
      required tutor-facing
      finalization fields complete
    end note

    note right of finalized
      Official system truth.
      Student/parent visibility
      only here.
    end note
```
