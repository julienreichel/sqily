# Assessment Orchestration (Moderator/Teacher) - Detailed Flow

## Scope
This flow focuses on teacher/expert moderation of assessment: evaluation authoring, exam assignment/orchestration, note-based decisioning, and completion propagation.

## End-to-end implementation
1. UI entry points
- Evaluation controls in skill page (`app/views/skills/_evaluations.html.erb`, `app/views/evaluations/exams/_evaluation.html.erb`).
- Exam conversation and actions in `app/views/evaluations/exams/show.html.erb`.
- Teacher actions include activate/deactivate evaluation, reply/reject/accept note, and tracking ongoing exams.

2. Evaluation authoring and lifecycle
- `EvaluationsController#create` creates evaluation linked to skill and current user.
- `can_update_evaluation` uses `current_user.permissions.edit_evaluation?` for `edit/update/enable/disable`.
- `disable` sets `disabled_at`; `enable` clears it.

3. Exam orchestration
- Exam creation is initiated by learner, but assignment/orchestration logic is central for teacher flow:
  - `Evaluations::ExamsController#create` calls `Evaluation#start(subscription, content)`.
  - `Evaluation#start` transaction:
    - chooses examiner with `pick_examiner_for` (prefers less busy experts, same team, avoids previous examiner),
    - resumes canceled matching exam or creates a new `Evaluation::Exam`,
    - creates first `Evaluation::Note` from candidate content.
- Teacher sees assigned exams in `Evaluations::ExamsController#index` using scopes:
  - `in_community`, `of_user`, `order_by_last_note`.

4. Review and decision loop
- Teacher posts note via `Evaluations::NotesController#create`.
- `note_params` maps UI buttons into flags (`is_accepted`, `is_rejected`).
- `Evaluation::Exam#add_note`:
  - enforces acceptance permission (`can_accept_exam?`) by resetting accept/reject flags if unauthorized,
  - persists note,
  - if accepted, marks subscription complete (`subscription.complete(user)`) in transaction.
- On persisted note, controller calls `@note.send_email` for roundtrip notifications.

5. Operational controls
- `cancel` and `resume` for exam lifecycle.
- `change_examiner` cancels current exam and starts a new one with initial note content transferred.
- `show` denies access unless `current_user.permissions.read_exam?(@exam)`.

## Validations, checks, and rules
- `Evaluation`: must have `skill_id`, `user_id`, and `description`.
- `Evaluation::Exam`: examiner required.
- `Evaluation::Note`: content required unless accepted.
- Exam state guard: only one ongoing exam per candidate context (`subscription.exams.ongoing`).
- Authorization:
  - edit/destroy evaluation (`edit_evaluation?`, `destroy_evaluation?`),
  - read exam (`read_exam?`),
  - accept exam decision (`can_accept_exam?`).

## Side effects and storage
- Persistent storage: `evaluations`, `evaluation_exams`, `evaluation_notes`, `subscriptions`.
- Side effects:
  - `ExamMailer.created` on exam creation,
  - `Evaluation::Note#send_email` after note creation,
  - subscription completion updates learner status and may propagate to parent subscriptions.

## Sequence diagram
```mermaid
sequenceDiagram
  autonumber
  actor T as Teacher/Examiner
  actor L as Learner
  participant UI as Evaluation/Exam UI
  participant EVC as EvaluationsController
  participant EXC as Evaluations::ExamsController
  participant NC as Evaluations::NotesController
  participant E as Evaluation
  participant EX as Evaluation::Exam
  participant N as Evaluation::Note
  participant P as User::Permissions
  participant Sub as Subscription
  participant DB as PostgreSQL
  participant Mail as ExamMailer

  T->>UI: Create or edit evaluation definition
  UI->>EVC: POST/PATCH /skills/:skill_id/evaluations or /evaluations/:id
  EVC->>P: edit_evaluation?(evaluation) for update/enable/disable
  EVC->>DB: INSERT/UPDATE evaluations (disabled_at toggle when needed)
  EVC-->>UI: Redirect skill page

  L->>UI: Submit draft as exam request
  UI->>EXC: POST /evaluations/:evaluation_id/exams
  EXC->>DB: Load evaluation + learner subscription
  EXC->>DB: Check ongoing exam
  alt ongoing exists
    EXC-->>UI: Redirect with already_in_progress alert
  else no ongoing
    EXC->>E: start(subscription, draft_content)
    E->>DB: pick_examiner_for(...) + create/resume exam + first note
    E-->>EXC: persisted exam
    EXC->>Mail: created(exam).deliver_now
    EXC-->>UI: Redirect to exam
  end

  T->>UI: Open assigned exam and submit decision note
  UI->>NC: POST /exams/:id/notes (accept/reject/send)
  NC->>EX: add_note(note_params)
  EX->>P: can_accept_exam?(exam)
  alt unauthorized accept attempt
    EX->>EX: Force is_accepted=false and is_rejected=false
  end
  EX->>N: Save note
  N->>DB: INSERT evaluation_notes
  alt accepted note
    EX->>Sub: complete(teacher)
    Sub->>DB: UPDATE subscriptions.completed_at
  end
  NC->>N: send_email()
  N->>Mail: deliver_now
  NC-->>UI: Redirect exams list

  T->>UI: Cancel/resume/change examiner if needed
  UI->>EXC: DELETE cancel / POST resume / POST change_examiner
  EXC->>DB: Update exam state or recreate with new examiner
  EXC-->>UI: Redirect exam/skill
```
