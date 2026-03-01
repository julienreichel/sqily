# Permissions Matrix

Roles documented here:
- Learner: authenticated member without moderator privileges.
- Moderator: community member with `memberships.moderator = true`.
- Admin: platform admin (`users.admin = true`), including admin namespace access.

Legend:
- Yes: role can perform action directly.
- Conditional: depends on ownership/state/domain-specific condition.
- No: role cannot perform action.

## Community and Membership Governance

| Action | Learner | Moderator | Admin | Notes / Code references |
|---|---|---|---|---|
| View community skills/pages as member | Yes | Yes | Yes | `must_be_membership` guard |
| Edit community metadata | No | Yes | Yes | `can_edit_current_community?` |
| Toggle moderator role on membership | No | Yes | Yes | `MembershipsController#moderator` + `must_be_moderator` usage |
| Manage invitations (create/destroy) | No | Yes | Yes | `InvitationsController` guarded by `must_be_moderator` |
| Accept/reject invitation requests | No | Yes | Yes | `InvitationRequestsController#update/#destroy` with `must_be_moderator` |
| Manage teams | No | Yes | Yes | `Membership::Permissions#create_teams?` |

## Skill Architecture and Progression

| Action | Learner | Moderator | Admin | Notes / Code references |
|---|---|---|---|---|
| Browse skill catalog/tree | Yes | Yes | Yes | `SkillsController#index/show` |
| Subscribe/unsubscribe to a skill | Conditional | Conditional | Conditional | Allowed when `Skill#startable_by?` and visibility constraints pass |
| Mark task done/undone | Yes | Yes | Yes | `TasksController#toggle` |
| Create/edit skill | No (except free-creation owner case) | Yes | Yes | `SkillsController#must_be_moderator_or_free_skill_creation` |
| Delete skill | No | Conditional | Conditional | `User::Permissions#destroy_skill?` requires moderator + `skill.destroyable?` |
| Manage prerequisites | No | Yes | Yes | `PrerequisitesController` + `must_be_authorized_to_edit_current_community_skills` |
| Complete a subscription | Conditional | Yes | Yes | `evaluate_subscription?`: moderator, expert, or auto-evaluation self-case |
| Uncomplete subscription | No | Yes | Yes | `SubscriptionsController#uncomplete` requires `moderator?` |

## Messaging and Collaboration

| Action | Learner | Moderator | Admin | Notes / Code references |
|---|---|---|---|---|
| Post messages in allowed scopes | Yes | Yes | Yes | `MessagesController#create/upload` |
| Edit own message | Yes | Yes | Yes | Author-only (`Message.from_user(current_user)`) |
| Delete message | Conditional | Yes | Yes | Author or moderator |
| Pin message | Conditional | Conditional | Conditional | `Message#pinnable_by?` (community moderator or skill expert/moderator) |
| Vote message | Yes | Yes | Yes | `Vote.toggle` |
| Mark direct message unread | Conditional | Conditional | Conditional | Only recipient (`mark_message_as_unread?`) |
| Download attachment | Conditional | Conditional | Conditional | `Message#viewable_by?` access checks |

## Assessment and Validation

| Action | Learner | Moderator | Admin | Notes / Code references |
|---|---|---|---|---|
| Create/edit own evaluations | Conditional | Yes | Yes | Owner can edit own; moderator can edit community evaluations (`edit_evaluation?`) |
| Disable/enable evaluation | Conditional | Yes | Yes | Same permission as edit |
| Delete evaluation | Conditional | Conditional | Conditional | `destroy_evaluation?` and `evaluation.destroyable?` |
| Start exam from draft | Conditional | Conditional | Conditional | `create_exam_from?` requires active subscription and no ongoing exam |
| Read exam thread | Conditional | Conditional | Conditional | Candidate or examiner only (`read_exam?`) |
| Accept exam | No (unless examiner) | Conditional | Conditional | Only examiner (`can_accept_exam?`) |
| Cancel exam | Conditional | Conditional | Conditional | Candidate-only for active exam (`cancel_exam?`) |
| Evaluate homework (approve/reject) | Conditional | Yes | Yes | Practically evaluator/expert path; moderation flow enables oversight |

## Events and Participation

| Action | Learner | Moderator | Admin | Notes / Code references |
|---|---|---|---|---|
| Register/unregister to event | Conditional | Conditional | Conditional | `Event#registerable?` + capacity/deadline rules |
| Create/edit own event | Conditional | Conditional | Conditional | Owner-only (`can_edit_event?`) |
| Toggle attendance status | No | Conditional | Conditional | Event owner and event already started (`can_toggle_participations_of_event?`) |

## Workspaces

| Action | Learner | Moderator | Admin | Notes / Code references |
|---|---|---|---|---|
| Read workspace | Conditional | Conditional | Conditional | Published or partnership (`read_workspace?`) |
| Edit workspace content | Conditional | Conditional | Conditional | Writer partnership (`update_workspace?`) |
| Approve/reject workspace | Conditional | Conditional | Conditional | Reader + moderation constraints (`approve_workspace?`, `reject_workspace?`) |
| Publish/unpublish workspace | Conditional | Conditional | Conditional | Ownership/moderation + state checks (`publish_workspace?`, `unpublish_workspace?`) |
| Destroy workspace | Conditional | Conditional | Conditional | Owner-only (`destroy_workspace?`) |

## Platform Admin Namespace

| Action | Learner | Moderator | Admin | Notes / Code references |
|---|---|---|---|---|
| Access `/admin/*` pages | No | No | Yes | `Admin::BaseController` + `current_user_must_be_admin` |
| Manage admin communities/pages/requests | No | No | Yes | `namespace :admin` routes |

## Caveats
- This matrix documents backend authorization logic, not UI visibility only.
- Some actions are technically callable but constrained by object ownership/state checks in models and policies.
