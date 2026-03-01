# Business Rules (Centralized)

This document consolidates key behavioral rules currently distributed across models, controllers, and callbacks.

## 1) Skill and Prerequisite Rules

### Skill validity
- A skill must have `name`, `description`, and `community`.
- Skill names are unique per community (case-insensitive).
- A skill cannot be parent of itself.

### Hierarchy constraints
- Only a root skill without evaluations can have children (`Skill#can_have_children?`).
- Parent assignment is filtered in `SkillForm#filter_param_parent_id`:
  - candidate parent must be a root skill in the same community,
  - candidate parent must still be eligible to have children.

### Prerequisite constraints
- Prerequisite edges are unique (`from_skill_id`, `to_skill_id`).
- `mandatory` flag can be toggled by authorized editor.
- Foreign/out-of-scope prerequisite links are removed after skill changes (`Skill#remove_foreign_prerequisites`).

### Skill startability
- Learner can start (subscribe) only if:
  - completed prerequisite count >= `minimum_prerequisites`, and
  - all mandatory prerequisite skills are completed, and
  - parent skill is startable recursively.

## 2) Subscription and Completion Rules

### Subscription lifecycle
- `pending`: `completed_at = null`
- `completed`: `completed_at != null`

### Completion authority
- A subscription can be completed when user is:
  - the learner and skill is auto-evaluation, or
  - an expert for the same skill, or
  - a moderator of the community.

### Completion propagation
- Completing a subscription may refresh parent completion (`parent.refresh_completed_at`).
- Uncompleting a subscription cascades upward (`parent.uncomplete`).
- If uncompleting and completed exams exist, accepted flags are cleared and exams are canceled.

### Hierarchy reorganization after skill edits
- Changing skill parent/mandatory can trigger:
  - reorganization of subscriptions on impacted parent trees,
  - completion refreshes for impacted subscriptions.

## 3) Evaluation and Exam Rules

### Evaluation definition
- Evaluation requires `skill`, `author`, and `description`.
- Evaluation may be disabled (`disabled_at`) and hidden for non-owner users.
- Evaluation is deletable only if no exams exist.

### Exam creation
- Only one ongoing exam per candidate/subscription context.
- On exam start, examiner is selected by workload and team affinity:
  - prefer less busy experts in same team,
  - avoid previous examiner for same candidate when possible,
  - fallback to global expert pool.

### Exam states
- `ongoing`: not canceled and no accepted note yet.
- `completed`: at least one accepted note.
- `canceled`: `is_canceled = true`.

### Exam note acceptance
- Only examiner can effectively accept/reject; unauthorized accept/reject flags are neutralized.
- Accepted note completes learner subscription.

### Candidate controls
- Candidate can cancel active exam.
- Candidate can resume canceled exam if no active sibling exam exists.
- Candidate can request examiner change by canceling and restarting exam with previous content.

## 4) Homework Rules

### Homework states
- `open slot`: no file
- `pending`: file present, not approved/rejected
- `approved`
- `rejected`

### Submission and review
- Learner uploads artifact to own homework record.
- Reviewer can approve/reject and attach comment/file feedback.
- Approve completes subscription with reviewer as validator.
- Reject marks homework rejected and creates a new open homework for retry.

### Side effects
- Homework upload triggers `Message::HomeworkUploaded` creation (if file present).
- Homework status updates trigger notifications (pending/approved/rejected).
- Rejection sends rejection email.

## 5) Messaging Rules

### Recipient and sender integrity
- Message must have exactly one target context among: user/community/skill/workspace.
- Sender is required.
- Direct message to self is invalid.

### Visibility and moderation
- File download requires `viewable_by?` checks based on destination scope membership/subscription/partnership.
- Pinning rights:
  - no direct-message pinning,
  - community message pinning requires moderator,
  - skill message pinning requires skill expert or community moderator.

### Read state
- Unread mark toggle is restricted to message recipient.

## 6) Event and Participation Rules

### Event validity
- Event must have title, scheduling fields, and positive capacity.
- Registration closes before event start.
- Event belongs to either community or skill context.

### Registration constraints
- Registration blocked after registration deadline.
- For skill-scoped event: user must be subscribed to skill.
- For community-scoped event: user must be member of community.
- If full, user is queued in waiting list.

### Waiting list behavior
- On seat release, oldest waiting participant is promoted (FIFO).
- Promotion sends notification email.

### Attendance marking
- Only event owner can toggle attendance.
- Attendance toggling is only enabled after event time has started.

## 7) Workspace Rules

### Access model
- Read: published workspace or explicit partnership.
- Write: writer partnership.
- Owner has destroy capability.

### Publication and approval
- Publishing requires approved workspace and ownership/admin-like rights.
- Approve/reject depend on workspace reader/moderator context and current state.
- Reject is only allowed before first publication (`!published_once`).

### Versioning and locking
- Edit flow acquires workspace lock.
- New version may be created when external feedback exists after last version.

## 8) Invitation and Membership Rules

### Invitation
- Invitation email must be valid and unique per community.
- Invitation token is generated on create and consumed on acceptance.
- Accepting invitation adds user to community and removes invitation.

### Invitation request
- Request email must be valid and unique per community.
- Request creation notifies moderators.
- Accepting request creates invitation and removes request.

### Membership
- One membership per user/community.
- Moderator role toggle endpoint currently has no explicit controller authorization guard.

## 9) Scheduled Rules and Periodic Automation

- Hourly task:
  - trigger omnipresent badge evaluation,
  - trigger poll finished notifications.
- Daily task:
  - run daily summaries,
  - run weekly summaries on configured weekday.

## References
- Flow-level details: `docs/*-flow.md`
- Domain structure: `docs/domain-model.md`
- Permissions: `docs/permissions-matrix.md`
