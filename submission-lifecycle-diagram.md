# Submission Lifecycle Diagram

Source: `PRD.md` Section 8, `Lifecycle and State Model`

```mermaid
stateDiagram-v2
    [*] --> draft
    draft --> submitted: student final submission

    note right of draft
      Guard:
      completeness satisfied
      or explicit not-attempted markers
    end note

    note right of submitted
      No reopen or revision
      of the same submission in V1
    end note
```
