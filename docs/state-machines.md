# State Machines

Explicit lifecycle transitions for core workflow entities.

## 1) Subscription

State fields:
- `completed_at`
- `validator_id`

```mermaid
stateDiagram-v2
  [*] --> Pending
  Pending: completed_at = null
  Completed: completed_at != null

  Pending --> Completed: complete(validator)\n(authorization required)
  Completed --> Pending: uncomplete()\n(moderator path)

  note right of Completed
    Side effects:
    - set validator_id
    - parent.refresh_completed_at
  end note

  note right of Pending
    On uncomplete:
    - clear validator_id
    - parent.uncomplete cascade
    - completed exams can be canceled
  end note
```

Transition triggers:
- `SubscriptionsController#complete` -> `Subscription#complete`
- `SubscriptionsController#uncomplete` -> `Subscription#uncomplete`
- `Evaluation::Exam#add_note` (accepted note) -> `subscription.complete`
- `Homework#approve` -> `subscription.complete`

## 2) Homework

State fields:
- `file_node`
- `approved_at`
- `rejected_at`

```mermaid
stateDiagram-v2
  [*] --> OpenSlot
  OpenSlot: file_node = null
  PendingReview: file_node != null\napproved_at = null\nrejected_at = null
  Approved: approved_at != null
  Rejected: rejected_at != null

  OpenSlot --> PendingReview: upload(file)
  PendingReview --> Approved: approve(by_user)
  PendingReview --> Rejected: reject()
  Rejected --> OpenSlot: reject_and_keep_open()\ncreates new homework row

  note right of PendingReview
    Side effects on save:
    - Message::HomeworkUploaded trigger
    - Notification::HomeworkPending trigger
  end note

  note right of Approved
    Side effects:
    - subscription.complete(by_user)
    - Notification::HomeworkApproved trigger
  end note

  note right of Rejected
    Side effects:
    - UserMailer.homework_rejected
    - Notification::HomeworkRejected trigger
  end note
```

Transition triggers:
- `HomeworksController#upload`
- `HomeworksController#evaluate`
- `Homework#approve`, `Homework#reject`, `Homework#reject_and_keep_open`

## 3) Evaluation::Exam

State fields:
- `is_canceled`
- accepted note existence (`evaluation_notes.is_accepted = true`)

```mermaid
stateDiagram-v2
  [*] --> Ongoing
  Ongoing: is_canceled = false\nno accepted note
  Canceled: is_canceled = true
  Completed: accepted note exists

  Ongoing --> Canceled: cancel()
  Canceled --> Ongoing: resume() if no active sibling
  Ongoing --> Completed: add_note(is_accepted = true)

  note right of Ongoing
    Created by Evaluation#start
    with initial candidate note
  end note

  note right of Completed
    Side effect:
    - subscription.complete(examiner)
  end note
```

Transition triggers:
- `Evaluations::ExamsController#create/cancel/resume/change_examiner`
- `Evaluations::NotesController#create` -> `Evaluation::Exam#add_note`

## 4) Workspace

State fields:
- `approved_at`
- `published_at`
- `published_once`

```mermaid
stateDiagram-v2
  [*] --> Draft
  Draft: approved_at = null\npublished_at = null
  Approved: approved_at != null\npublished_at = null
  Published: published_at != null

  Draft --> Approved: approve!()
  Approved --> Draft: reject!() if rejectable
  Approved --> Published: publish!()
  Published --> Approved: unpublish!()

  note right of Published
    publish! sets published_once = true
    and emits workspace published messages
  end note

  note right of Draft
    reject! allowed only before first publish
    (rejectable? => approved && !published_once)
  end note
```

Transition triggers:
- `WorkspacesController#approve/#reject/#publish/#unpublish`
- Permission checks in `User::Permissions`

## 5) Participation

State field:
- `confirmed` (`nil`, `true`, `false`)

```mermaid
stateDiagram-v2
  [*] --> Unknown
  Unknown: confirmed = null
  Present: confirmed = true
  Absent: confirmed = false

  Unknown --> Present: toggle_presence
  Present --> Absent: toggle_presence
  Absent --> Unknown: toggle_presence

  note right of Unknown
    Toggle allowed only if
    can_toggle_participations_of_event?
  end note
```

Transition triggers:
- `Events::ParticipationsController#toggle` -> `Participation#toggle_presence`

## Notes
- These are persistence-backed state machines derived from current model/controller behavior.
- Some transitions are guarded by role/ownership checks and by parent object state.
