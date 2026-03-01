# Evaluation and Validation - Detailed Flow

## Scope
This flow covers evaluation lifecycle on a skill: draft auto-save, exam creation, discussion notes, accept/reject decision, and subscription completion.

## End-to-end implementation
1. UI entry points
- Skill page renders evaluation zone in `app/views/skills/_evaluations.html.erb`.
- Draft editor form is in `app/views/evaluations/drafts/_form.html.erb` with `data-ariato="Sqily.Evaluation.DraftForm"`.
- Exam conversation page is `app/views/evaluations/exams/show.html.erb`.

2. Draft auto-save and submission
- Frontend module `app/assets/javascripts/sqily/evaluation/draft_form.js` watches trix changes.
- It enables submit only when draft content is non-empty and activates `FormAutoSave`.
- Auto-save POSTs to `Evaluations::DraftsController#create` with `subscription_id`, `evaluation_id`, `content`.
- Controller checks ownership and skill consistency:
  - subscription must belong to current user,
  - evaluation must belong to that subscription skill.
- Then it upserts `Evaluation::Draft` (`find_or_initialize_by(...).update(...)`).

3. Exam creation
- Submit button posts to `Evaluations::ExamsController#create`.
- Controller resolves evaluation + current user subscription.
- Checks ongoing exam guard:
  - if any ongoing exam exists -> redirect to active exam with alert.
- Else calls `Evaluation#start(subscription, draft_content)`:
  - transaction,
  - selects examiner via `pick_examiner_for` (load balancing + team preference + no immediate repeat),
  - resumes existing canceled exam or creates new `Evaluation::Exam`,
  - creates first `Evaluation::Note` from candidate draft content.
- On success: `ExamMailer.created(exam).deliver_now` and redirect to exam page.

4. Exam conversation and decision
- `Evaluations::ExamsController#show` loads exam + notes and enforces `current_user.permissions.read_exam?`.
- User posts note via `Evaluations::NotesController#create` -> `Evaluation::Exam#add_note`.
- `add_note` business logic:
  - if user lacks permission to accept exam, force `is_accepted=false` and `is_rejected=false`,
  - save note,
  - if accepted note, call `subscription.complete(user)` in transaction.
- If note persisted, controller redirects and triggers `note.send_email`.

5. Lifecycle operations
- Candidate can cancel/resume exam (`ExamsController#cancel/#resume`) with sibling-active checks.
- Candidate can change examiner (`#change_examiner`) by canceling and re-starting with prior note content.
- Evaluation owners can enable/disable evaluation (`EvaluationsController#enable/#disable`) and edit metadata.

## Validations, checks, and rules
- `Evaluation` requires `skill_id`, `user_id`, `description`.
- `Evaluation::Exam` requires `examiner`.
- `Evaluation::Note` requires `content` unless accepting.
- Draft submittability rule: `Evaluation::Draft#submittable?` requires `evaluation.skill.experts.any?`.
- Access controls:
  - membership required (`must_be_membership`),
  - permission checks for editing evaluations and reading exams.

## Side effects and storage
- Persistent storage: `evaluations`, `evaluation_drafts`, `evaluation_exams`, `evaluation_notes`, `subscriptions`.
- Accepting an exam can mark subscription complete and propagate completion hierarchy.
- Email side effects:
  - `ExamMailer.created` on exam start,
  - `Evaluation::Note#send_email` for reply/rejection notifications.

## Sequence diagram
```mermaid
sequenceDiagram
  autonumber
  actor U as Candidate User
  participant UI as Skill Evaluation UI (draft/exam)
  participant DF as Sqily.Evaluation.DraftForm
  participant DC as Evaluations::DraftsController
  participant EC as Evaluations::ExamsController
  participant E as Evaluation
  participant EX as Evaluation::Exam
  participant NC as Evaluations::NotesController
  participant N as Evaluation::Note
  participant Sub as Subscription
  participant DB as PostgreSQL
  participant Mail as ExamMailer

  U->>UI: Type draft in trix editor
  UI->>DF: trix-change
  DF->>DF: updateSubmitButtonState()
  DF->>DC: Auto-save POST /evaluations/drafts
  DC->>DB: Verify subscription belongs to current_user
  DC->>DB: Verify evaluation belongs to subscription.skill
  DC->>DB: Upsert Evaluation::Draft
  DC-->>UI: 200 OK

  U->>UI: Click submit draft
  UI->>EC: POST /evaluations/:evaluation_id/exams
  EC->>DB: Load evaluation + subscription
  EC->>DB: Check subscription.exams.ongoing.first
  alt Active exam exists
    EC-->>UI: Redirect to existing exam with alert
  else No active exam
    EC->>E: start(subscription, draft.content)
    E->>DB: pick_examiner_for(subscription)
    E->>EX: create/resume exam
    EX->>N: create first note with candidate content
    N->>DB: INSERT evaluation_notes
    EX->>DB: INSERT/UPDATE evaluation_exams
    EC->>Mail: ExamMailer.created(exam).deliver_now
    EC-->>UI: Redirect to exam show
  end

  U->>UI: Post note / accept / reject
  UI->>NC: POST /exams/:id/notes
  NC->>EX: add_note(note_params)
  EX->>EX: Permission gate on acceptance flags
  EX->>N: new note + save
  N->>DB: INSERT evaluation_notes
  alt Note is accepted
    EX->>Sub: complete(current_user)
    Sub->>DB: UPDATE subscriptions.completed_at/validator_id
  end
  NC->>N: send_email()
  N->>Mail: deliver_now (created/rejected depending state)
  NC-->>UI: Redirect exam list/show
```
