# Review and Correct Extracted Questions User Flow Diagram

Source: [PRD.md](/Users/bubusharma/deepanshu_projects/software-projects-prompts/PRD.md:1267) Section `9.1 Authoring and Publish`, `UF-03: Review and Correct Extracted Questions`

```mermaid
flowchart TD
    start([Start])
    auth{Authenticated?}
    role{Has admin/content creator access?}
    draft{Assignment exists and is in draft?}
    extraction{Extraction status completed and candidate question set exists?}
    open[Admin opens draft assignment after extraction completion]
    available[System surfaces question correction as available]
    status[System shows correction-step status]
    enter[Admin enters extracted-question review and correction flow]
    inprogress[System sets correction-step status to in_progress]
    review[Admin reviews boundaries, numbering, order, and content]
    action{Correction action needed}
    split[Split incorrectly combined question unit]
    merge[Merge incorrectly separated question units]
    reorder[Reorder questions]
    renumber[Renumber questions]
    delete[Delete false-positive extracted question]
    add[Add missing question manually]
    edit[Manually edit question content]
    persist[System persists working changes]
    update[System updates corrected working question set]
    saveexit[Admin saves progress and exits]
    confirm[Admin requests final confirmation]
    validate{Corrected question set structurally valid?}
    complete[System sets correction-step status to completed]
    preserve[System preserves confirmed corrected question set as authoritative]
    nextstep[System surfaces next authoring flow]
    success([End: Corrected question set confirmed])

    blocked([End: Access or prerequisite blocked])
    invalidconfirm([End: Confirmation blocked until issues resolved])
    persistfail([End: Persistence failure surfaced])
    warning[System warns corrected working set will be discarded]
    reextract[Admin confirms return to re-extraction]
    discard[System discards current corrected working set as active]
    backtoextract([End: Return to source replacement/re-extraction])
    paused([End: Progress saved, correction remains in progress])

    start --> auth
    auth -- No --> blocked
    auth -- Yes --> role
    role -- No --> blocked
    role -- Yes --> draft
    draft -- No --> blocked
    draft -- Yes --> extraction
    extraction -- No --> blocked
    extraction -- Yes --> open
    open --> available
    available --> status
    status --> enter
    enter --> inprogress
    inprogress --> review
    review --> action

    action -- Split --> split
    action -- Merge --> merge
    action -- Reorder --> reorder
    action -- Renumber --> renumber
    action -- Delete false positive --> delete
    action -- Add missing question --> add
    action -- Edit question content --> edit
    action -- Save progress and exit --> saveexit
    action -- Return to re-extraction --> warning
    action -- Final confirm corrected set --> confirm

    split --> persist
    merge --> persist
    reorder --> persist
    renumber --> persist
    delete --> persist
    add --> persist
    edit --> persist

    persist -->|Failure| persistfail
    persist -->|Success| update
    update --> review

    saveexit --> paused

    warning --> reextract
    reextract --> discard
    discard --> backtoextract

    confirm --> validate
    validate -- No --> invalidconfirm
    invalidconfirm --> review
    validate -- Yes --> complete
    complete --> preserve
    preserve --> nextstep
    nextstep --> success
```

## Diagram Notes

- Actor: `Admin/Content Creator`
- Preconditions enforced in the diagram:
  - authenticated user
  - `admin/content creator` access
  - target `Assignment` exists and is in `draft`
  - extraction status is `completed`
  - candidate extracted `Question` set exists
- Supported correction actions:
  - split
  - merge
  - reorder
  - renumber
  - delete false positives
  - add missing questions
  - manually edit question content
- Working changes persist during editing and support save-progress.
- Manual content edits become authoritative question content.
- Final confirmation requires structural validity:
  - at least one question exists
  - each question has authoritative content
  - each question has final number/order for this stage
  - no unresolved split/merge placeholder artifacts remain
- All questions remain `incomplete` after this flow.
- Returning to re-extraction is destructive to the active corrected working set and requires explicit warning.
