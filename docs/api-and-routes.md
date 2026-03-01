# API and Routes Catalog

This catalog documents the server routes by domain, with key payloads, auth requirements, and response shapes.

## Auth model used in this document
- Public: no login required.
- Authenticated: user must be logged in.
- Member: logged-in user with membership in `:permalink` community (`must_be_membership`).
- Moderator: member with moderator privileges (`must_be_moderator` or equivalent permission checks).
- Admin: platform admin (`current_user_must_be_admin`) for `/admin/*` namespace.

## Response conventions
- HTML redirect/render: default for most actions.
- XHR/partial HTML: used by messaging/toggles.
- JSON: primarily `messages#index` polling endpoint.
- `head(:ok)` / `head(:unprocessable_entity)` / `head(:locked)` used for some async endpoints.

## 1) Public, Access, and Onboarding

| Method | Path | Auth | Payload (key params) | Response shape |
|---|---|---|---|---|
| GET | `/` | Public | - | HTML page index |
| GET | `/public` | Public | - | HTML public communities |
| GET | `/:permalink` | Public | - | HTML public skills list |
| GET | `/:permalink/public/skills/:id` | Public | - | HTML public skill details |
| GET | `/pages/:slug` | Public | `slug` | HTML static page |
| GET | `/session` | Public | - | HTML login form |
| POST | `/session` | Public | session credentials | Redirect (success/failure) |
| DELETE | `/session` | Authenticated | - | Redirect |
| GET | `/users/new` | Public | optional invitation context | HTML signup form |
| POST | `/users` | Public | `user[name,email,password]` | Redirect / render errors |
| GET | `/password_resets` | Public | - | HTML reset form |
| POST | `/password_resets` | Public | reset request attrs | Redirect |
| GET | `/password_resets/:id` | Public | token/id | HTML reset completion form |

## 2) Community and Membership Domain

| Method | Path | Auth | Payload (key params) | Response shape |
|---|---|---|---|---|
| GET | `/:permalink/description` | Member | - | HTML |
| GET | `/:permalink/discussion` | Member | - | HTML |
| GET | `/:permalink/tree` | Public/member (action-specific) | optional `community_tree` | HTML |
| GET | `/:permalink/edit` | Moderator/authorized | - | HTML form |
| PATCH | `/:permalink` | Moderator/authorized | `community[name,description,permalink,free_skill_creation,public,registration_code,duplicate_evaluations]` | Redirect/render |
| GET | `/:permalink/state` | Member | - | HTML |
| POST | `/:permalink/duplicate` | Conditional | duplication params | Redirect |
| GET | `/:permalink/progression` | Member | optional filters | HTML |
| GET | `/:permalink/users` | Member | optional filters | HTML |
| GET | `/:permalink/users/:id` | Authenticated member | `id` | HTML |
| DELETE | `/:permalink/users/:id` | Moderator | `id` | Redirect |
| DELETE | `/:permalink/users/:id/avatar` | Owner/moderator flow | `id` | Redirect |
| POST | `/:permalink/memberships` | Public/auth depending flow | `registration_code` etc. | Redirect |
| PATCH | `/:permalink/memberships/:id` | Member (self context) | `membership[description]`, `user[...]` | Redirect/render |
| PUT | `/:permalink/memberships/:id/moderator` | Moderator | `id` | Redirect |

## 3) Invitations and Requests

| Method | Path | Auth | Payload (key params) | Response shape |
|---|---|---|---|---|
| GET | `/:permalink/invitations` | Moderator | - | HTML |
| POST | `/:permalink/invitations` | Moderator | `invitation[email]` (multi-line) | Redirect/render |
| GET | `/:permalink/invitations/:token` | Public | `token` | Redirect (join/signup flow) |
| DELETE | `/:permalink/invitations/:token` | Moderator | `token` | Redirect |
| GET | `/:permalink/invitation_requests` | Public non-member | - | HTML form |
| POST | `/:permalink/invitation_requests` | Public non-member | `invitation_request[email]` | Render/redirect |
| PUT | `/:permalink/invitation_requests/:id` | Moderator | `id` | Redirect |
| DELETE | `/:permalink/invitation_requests/:id` | Moderator | `id` | Redirect |

## 4) Team Domain

| Method | Path | Auth | Payload (key params) | Response shape |
|---|---|---|---|---|
| GET | `/:permalink/teams/new` | Moderator-equivalent (`create_teams?`) | - | HTML form |
| POST | `/:permalink/teams` | Moderator-equivalent | `team[name]`, `user_ids[]` | Redirect/render |
| GET | `/:permalink/teams/edit/:id` | Moderator-equivalent | `id` | HTML form |
| PATCH | `/:permalink/teams/:id` | Moderator-equivalent | `team[name]`, `user_ids[]` | Redirect/render |
| DELETE | `/:permalink/teams/:id` | Moderator-equivalent | `id` | Redirect |

## 5) Skills, Prerequisites, Tasks, Subscriptions

| Method | Path | Auth | Payload (key params) | Response shape |
|---|---|---|---|---|
| GET | `/:permalink/skills` | Member | optional `community_tree` | HTML |
| POST | `/:permalink/skills` | Moderator or free-skill-creation | `skill[name,description,help,minimum_prerequisites,auto_evaluation,parent_id,mandatory,published_at]`, `tasks[][id,title,position,file]` | Redirect/render |
| PATCH | `/:permalink/skills/:id` | Moderator or free-skill-creation | same as create | Redirect/render |
| DELETE | `/:permalink/skills/:id` | Conditional (`destroy_skill?`) | - | Redirect |
| GET | `/:permalink/skills/:id` | Member | - | HTML |
| GET | `/:permalink/skills/:id/messages` | Member/subscriber context | paging filters | HTML |
| POST | `/:permalink/skills/:id/subscribe` | Member | - | Redirect |
| DELETE | `/:permalink/skills/:id/unsubscribe` | Member | - | Redirect |
| POST | `/:permalink/skills/:id/pin` | Subscriber | - | Redirect |
| POST | `/:permalink/skills/:id/auto_evaluation` | Member/moderator | - | Redirect/HTML |
| POST | `/:permalink/skills/:skill_id/prerequisites` | Moderator/authorized | `prerequisite[from_skill_id]` | Partial HTML |
| PATCH | `/:permalink/skills/:skill_id/prerequisites/:id/toggle_mandatory` | Moderator/authorized | - | `head(:ok)` |
| DELETE | `/:permalink/skills/:skill_id/prerequisites/:id` | Moderator/authorized | - | `head(:ok)` |
| POST | `/:permalink/skills/:skill_id/tasks/:id/toggle` | Member | - | `head(:ok)` |
| DELETE | `/:permalink/skills/:skill_id/tasks/:id` | Moderator/authorized flow | - | `head(:ok)` |
| POST | `/:permalink/subscription/:id/complete` | Conditional (`evaluate_subscription?`) | - | Redirect |
| POST | `/:permalink/subscription/:id/uncomplete` | Moderator | - | Redirect |

## 6) Messaging Domain

| Method | Path | Auth | Payload (key params) | Response shape |
|---|---|---|---|---|
| GET | `/:permalink/messages` | Member | context params: `user_id`, `skill_id`, `workspace_id`, pagination (`before`,`after`,`from`,`to`,`pinned`,`hash_tag`) | HTML, XHR partial HTML, or JSON |
| POST | `/:permalink/messages` | Member (+subscription if to_skill) | `message[to_user_id,to_community_id,to_skill_id,to_workspace_id,text]` | XHR partial HTML or redirect |
| PATCH | `/:permalink/messages/:id` | Author | `message[text]` | Partial HTML |
| POST | `/:permalink/messages/upload` | Member | `upload[to_user_id,to_community_id,to_skill_id,to_workspace_id,text,file]` | Redirect |
| DELETE | `/:permalink/messages/:id` | Author or moderator | - | Redirect |
| POST | `/:permalink/messages/:id/unread` | Recipient only | - | Redirect |
| POST | `/:permalink/messages/:id/pin` | Conditional | - | Redirect |
| POST | `/:permalink/messages/:id/vote` | Member | - | Redirect |
| GET | `/:permalink/messages/:id/download` | Conditional visibility | - | Redirect to file URL / 404 |
| GET | `/:permalink/messages/search_form` | Member | - | Partial HTML |
| GET | `/:permalink/messages/search` | Member | `query` | Partial HTML results |

### `messages#index` JSON response keys
- `new_messages`: rendered HTML chunk or `null`
- `next_url`: polling cursor URL
- `previous_url`: backward cursor URL
- `skill_ids_with_unread_messages`: integer array
- `edited_messages[]`: `{ id, html }`

## 7) Poll and Event Domain

| Method | Path | Auth | Payload (key params) | Response shape |
|---|---|---|---|---|
| POST | `/:permalink/polls` | Member | `poll[title,community_id,skill_id,workspace_id,finished_at,single_answer]` + `choices[]` | Redirect/render |
| GET | `/:permalink/polls/:id` | Member with editability constraints | `id` | HTML/404 |
| DELETE | `/:permalink/polls/:id` | Creator/moderation flow | - | Redirect |
| POST | `/:permalink/polls/answers` | Member | `choice_ids[]` | Redirect |
| GET | `/:permalink/events` | Member | optional `registered=true` | HTML |
| POST | `/:permalink/events` | Member | `event[title,max_participations,scheduled_at,registration_finished_at,file,description,skill_id]` | Redirect/render |
| PATCH | `/:permalink/events/:id` | Event owner | same as create | Redirect/render |
| DELETE | `/:permalink/events/:id` | Event owner | - | Redirect |
| POST | `/:permalink/events/:id/register` | Registerable member | - | Redirect |
| DELETE | `/:permalink/events/:id/unregister` | Participant/waitlisted member | - | Redirect |
| POST | `/:permalink/events/:event_id/participations/:id/absent` | Event owner after start | - | JSON participation object |

## 8) Evaluation and Homework Domain

| Method | Path | Auth | Payload (key params) | Response shape |
|---|---|---|---|---|
| GET | `/:permalink/skills/:skill_id/evaluations` | Member | - | HTML |
| POST | `/:permalink/skills/:skill_id/evaluations` | Member (author/moderator constraints in code) | `evaluation[description,title]` | Redirect |
| PATCH | `/:permalink/evaluations/:id` | Conditional (`edit_evaluation?`) | `evaluation[description,title]` | Redirect |
| DELETE | `/:permalink/evaluations/:id` | Conditional (`destroy_evaluation?`) | - | Redirect |
| POST | `/:permalink/evaluations/:id/disable` | Conditional | - | Redirect |
| POST | `/:permalink/evaluations/:id/enable` | Conditional | - | Redirect |
| POST | `/:permalink/evaluations/drafts` | Member | `evaluation_draft[subscription_id,evaluation_id,content]` | `head(:ok)` or `head(:unprocessable_entity)` |
| POST | `/:permalink/evaluations/:evaluation_id/exams` | Member candidate | `evaluation_draft[content]` | Redirect |
| GET | `/:permalink/exams/:id` | Candidate or examiner | - | HTML/404 |
| DELETE | `/:permalink/exams/:id/cancel` | Candidate | - | Redirect |
| POST | `/:permalink/exams/:id/resume` | Candidate | - | Redirect |
| POST | `/:permalink/exams/:id/change_examiner` | Candidate | - | Redirect |
| POST | `/:permalink/exams/:id/notes` | Candidate or examiner | `evaluation_note[content]` + `accept/reject/send` button flags | Redirect/render |
| POST | `/homeworks/:id/upload` | Homework owner | `homework_file` | Redirect |
| DELETE | `/homeworks/:id` | Homework owner | - | Redirect |
| POST | `/homeworks/:id/evaluate` | Evaluator flow | `comment`, optional `file`, `approve` or `reject` | Redirect |

## 9) Workspace and Profile Domain

| Method | Path | Auth | Payload (key params) | Response shape |
|---|---|---|---|---|
| POST | `/:permalink/workspaces` | Member | creation params via `WorkspaceForm` | Redirect |
| GET | `/:permalink/workspaces/:id` | Conditional visibility | optional `version` | HTML |
| PATCH | `/:permalink/workspaces/:id` | Writer with lock | `workspace[title,writing]` | `head(:ok)` or `head(:locked)` |
| DELETE | `/:permalink/workspaces/:id` | Owner | - | Redirect |
| POST | `/:permalink/workspaces/:id/publish` | Conditional | optional `skill_id` | Redirect |
| POST | `/:permalink/workspaces/:id/unpublish` | Conditional | - | Redirect |
| POST | `/:permalink/workspaces/:id/approve` | Conditional | - | Redirect |
| POST | `/:permalink/workspaces/:id/reject` | Conditional | - | Redirect |
| POST | `/:permalink/workspaces/:workspace_id/partnerships` | Workspace manager | `workspace_partnership[user_id,read_only]` | Redirect |
| DELETE | `/:permalink/workspaces/:workspace_id/partnerships/:id` | Conditional (`destroy_partnership?`) | - | Redirect |
| GET | `/:permalink/profile/:id` | Public | - | HTML |
| POST | `/:permalink/profile/:id/public` | Membership owner/moderator flow | - | Redirect |
| POST | `/:permalink/profile/:id/private` | Membership owner/moderator flow | - | Redirect |
| POST | `/:permalink/profile/hidden_items` | Member | `hidden_profile_item[workspace_id,subscription_id]` | Redirect |
| DELETE | `/:permalink/profile/hidden_items/:id` | Member | - | Redirect |

## 10) Statistics, Notifications, and Admin

| Method | Path | Auth | Payload (key params) | Response shape |
|---|---|---|---|---|
| GET | `/:permalink/statistics` | Moderator-equivalent (`read_community_statistics?`) | order/filter params | HTML |
| GET | `/:permalink/statistics/skills` | Moderator-equivalent | order/filter params | HTML |
| GET | `/:permalink/notifications` | Member | optional pagination/filter | HTML |
| ALL | `/admin/*` | Admin only | domain-specific forms | HTML redirect/render |

## References
- Complete route source: `config/routes.rb`
- Permission internals: `app/lib/user/permissions.rb`, `app/helpers/permissions_helper.rb`, `app/lib/membership/permissions.rb`
