# Permissions Matrix (Implemented Snapshot)

This matrix reflects the current code implementation only.

Roles used:
- Learner: authenticated user, typically non-moderator community member.
- Moderator: user with `current_membership.moderator = true` in the active community.
- Admin: user with `users.admin = true`.

Legend:
- Yes: explicitly allowed by current code path.
- Conditional: depends on object ownership/state/context checks.
- No: explicitly denied by current code path.
- Unknown: endpoint lacks explicit authorization guard; behavior depends on runtime context and may fail or be bypassable.

## Community and Membership Governance

| Action | Learner | Moderator | Admin | Implemented basis |
|---|---|---|---|---|
| Access community member pages | Yes | Yes | Conditional | `must_be_membership` on many community controllers |
| Edit community metadata | No | Yes | Yes | `can_edit_current_community?` includes `admin?` |
| Toggle moderator on membership | Unknown | Unknown | Unknown | `MembershipsController#moderator` has no explicit auth guard |
| Manage invitations | No | Yes | Conditional | `InvitationsController` guarded by `must_be_moderator` |
| Accept/reject invitation requests | No | Yes | Conditional | `InvitationRequestsController#update/#destroy` guarded by `must_be_moderator` |
| Create/edit/delete teams | No | Yes | Conditional | Requires `current_membership.permissions.create_teams?` |

## Skill Architecture and Progression

| Action | Learner | Moderator | Admin | Implemented basis |
|---|---|---|---|---|
| Browse skills | Yes | Yes | Conditional | `SkillsController` requires membership/auth |
| Subscribe/unsubscribe skill | Conditional | Conditional | Conditional | `Skill#startable_by?` and routing context |
| Toggle task done | Unknown | Unknown | Unknown | `TasksController` has no explicit auth/membership guard |
| Delete task | Unknown | Unknown | Unknown | Same as above |
| Create/edit skill | No (except free-creation case) | Yes | Conditional | `must_be_moderator_or_free_skill_creation` (no admin-specific branch) |
| Delete skill | No | Conditional | Conditional | `User::Permissions#destroy_skill?` (moderator + destroyable) |
| Manage prerequisites | No | Yes | Yes | `can_edit_current_community_skills?` includes `admin?` |
| Complete subscription | Conditional | Yes | Conditional | `evaluate_subscription?` has no explicit admin branch |
| Uncomplete subscription | No | Yes | Conditional | `SubscriptionsController#uncomplete` uses `moderator?` |

## Messaging and Collaboration

| Action | Learner | Moderator | Admin | Implemented basis |
|---|---|---|---|---|
| Create/upload message | Yes | Yes | Conditional | `MessagesController` requires membership/auth |
| Edit message text | Conditional | Conditional | Conditional | Author-only query scope |
| Delete message | Conditional | Yes | Conditional | Author or `moderator?` |
| Pin message | Conditional | Conditional | Conditional | `Message#pinnable_by?` logic |
| Vote message | Yes | Yes | Yes | No role check in `Vote.toggle` |
| Mark unread | Conditional | Conditional | Conditional | Recipient-only via `mark_message_as_unread?` |
| Download attachment | Conditional | Conditional | Conditional | `Message#viewable_by?` |

## Assessment and Validation

| Action | Learner | Moderator | Admin | Implemented basis |
|---|---|---|---|---|
| Create evaluation | Yes (member route) | Yes | Conditional | `EvaluationsController#create` requires membership, no role check |
| Edit/enable/disable evaluation | Conditional | Yes | Conditional | `edit_evaluation?` owner or community moderator |
| Delete evaluation | Conditional | Conditional | Conditional | `destroy_evaluation?` and `destroyable?` |
| Start exam | Conditional | Conditional | Conditional | `create_exam_from?`/subscription context |
| Read exam | Conditional | Conditional | Conditional | Candidate or examiner (`read_exam?`) |
| Accept exam note | No (unless examiner) | Conditional | Conditional | `can_accept_exam?` examiner-only |
| Cancel exam | Conditional | Conditional | Conditional | Candidate-only (`cancel_exam?`) |
| Evaluate homework | Unknown | Unknown | Unknown | `HomeworksController#evaluate` has no explicit auth guard |

## Events and Participation

| Action | Learner | Moderator | Admin | Implemented basis |
|---|---|---|---|---|
| Register/unregister event | Conditional | Conditional | Conditional | `Event#registerable?` + deadline/capacity |
| Edit/delete own event | Conditional | Conditional | Conditional | `Event#editable_by?` owner-only |
| Toggle attendance presence | No | Conditional | Conditional | `can_toggle_participations_of_event?` (owner + started event) |

## Workspaces

| Action | Learner | Moderator | Admin | Implemented basis |
|---|---|---|---|---|
| Read workspace | Conditional | Conditional | Conditional | Published or partnership (`read_workspace?`) |
| Edit workspace | Conditional | Conditional | Conditional | Writer partnership (`update_workspace?`) |
| Approve/reject workspace | Conditional | Conditional | Conditional | `approve_workspace?`, `reject_workspace?` |
| Publish/unpublish workspace | Conditional | Conditional | Conditional | `publish_workspace?`, `unpublish_workspace?` |
| Destroy workspace | Conditional | Conditional | Conditional | Owner-only (`destroy_workspace?`) |

## Platform Admin Namespace

| Action | Learner | Moderator | Admin | Implemented basis |
|---|---|---|---|---|
| Access `/admin/*` | No | No | Yes | `Admin::BaseController` + `current_user_must_be_admin` |
| Manage admin communities/pages/requests | No | No | Yes | `namespace :admin` routes |

## Notes
- Admin is not a global bypass in most community-scoped controllers.
- Rows marked `Unknown` are implementation gaps where explicit authorization is not enforced in controller actions.
