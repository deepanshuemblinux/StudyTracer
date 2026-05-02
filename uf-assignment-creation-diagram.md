# UF-01 User Flow Diagram

Source: [PRD.md](/Users/bubusharma/deepanshu_projects/software-projects-prompts/PRD.md:1032) Section `9.1 Authoring and Publish`, `UF-01: Create Assignment Draft`

```mermaid
flowchart TD
    start([Start])
    auth{Authenticated?}
    role{Has admin/content creator access?}
    entry[Start assignment creation within Content Authoring]
    view[System opens assignment creation view]
    type[Admin selects type: assignment or assessment]
    meta[Admin enters required metadata: subject, education_board, class_grade, chapter]
    title{Provide title at creation?}
    withtitle[Admin enters title]
    notitle[Admin leaves title blank for now]
    instructions{Provide instructions at creation?}
    withinstructions[Admin enters instructions]
    noinstructions[Admin omits instructions]
    decision{Admin action}
    validate[System validates required creation fields]
    valid{All required fields valid?}
    create[System creates Assignment in draft state]
    record[System records author_id, created_at, updated_at]
    open[System opens new assignment workspace]
    success([End: Assignment exists in draft state])

    blocked([End: Access blocked])
    correction([Return to creation view for correction])
    retry([Show failure, preserve values, allow retry])
    cancelled([End: No Assignment created])
    sessionfail([End: Creation blocked until authentication restored])

    start --> auth
    auth -- No --> sessionfail
    auth -- Yes --> role
    role -- No --> blocked
    role -- Yes --> entry
    entry --> view
    view --> type
    type --> meta
    meta --> title
    title -- Yes --> withtitle
    title -- No --> notitle
    withtitle --> instructions
    notitle --> instructions
    instructions -- Yes --> withinstructions
    instructions -- No --> noinstructions
    withinstructions --> decision
    noinstructions --> decision

    decision -- Cancel --> cancelled
    decision -- Submit create action --> validate
    validate --> valid
    valid -- No: missing or invalid required fields --> correction
    correction --> meta

    valid -- Yes --> create
    create -->|Persistence failure| retry
    retry --> validate

    create -->|Success| record
    record --> open
    open --> success
```

## Diagram Notes

- Actor: `Admin/Content Creator`
- Creation-time required metadata: `type`, `subject`, `education_board`, `class_grade`, `chapter`
- Creation-time optional metadata: `title`, `instructions`
- Explicit alternate paths shown:
  - `AP-01`: title provided at creation
  - `AP-02`: title omitted until later
  - `AP-03`: instructions provided at creation
  - `AP-04`: instructions omitted
- `overall_concepts_tested` is intentionally excluded from this flow and is populated later after question parsing.
- No pre-create draft is persisted before successful validation and creation.
