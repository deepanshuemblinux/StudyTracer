# Work Instance Lifecycle Diagram

Source: `PRD.md` Section 8, `Lifecycle and State Model`

```mermaid
stateDiagram-v2
    [*] --> assigned
    assigned --> in_progress: student starts work/upload
    in_progress --> submitted: student submits final answers
    submitted --> evaluated: tutor finalizes evaluation
    evaluated --> closed: system auto-closes

    note right of in_progress
      Guard:
      submission completeness rules
      must be satisfied
    end note

    note right of submitted
      Guard:
      tutor review must reach
      finalization readiness
    end note

    note right of closed
      Cannot reopen in V1.
      New follow-up requires
      new work instance.
    end note
```
