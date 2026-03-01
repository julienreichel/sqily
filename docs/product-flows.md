# Product Flows

This file maps the main functional flows implemented in the Sqily codebase.

Legend:
- Actors: primary users or system actors.
- Trigger: what starts the flow.
- Outputs: key state change or user-visible result.

## 1. Access and Onboarding Flows

| Flow name | Actors | Trigger | Main steps | Outputs | Related routes/classes |
|---|---|---|---|---|---|
| User registration | Visitor | Submit signup form | Open signup page -> validate form -> create `User` -> redirect/login path | New account created | `users#new`, `users#create`; `User` |
| Login | User | Submit login form | Validate credentials -> create session -> redirect to relevant community | Authenticated session | `session#create`, `session#show`; `SessionsController` |
| Logout | User | Click sign out | Destroy session -> redirect | Session closed | `session#destroy` |
| Password reset request | User | Submit email on reset page | Create reset token -> send email | Reset token + email sent | `password_resets#create`; `PasswordReset`, `UserMailer#password_reset` |
| Password reset completion | User | Open reset link/token | Validate token -> show reset form -> update password | Password updated | `password_resets#show`; `PasswordReset` |
| Browse public communities | Visitor | Open `/public` | Fetch public communities -> render list | Public catalog visible | `public/communities#index` |
| Public community view | Visitor | Open `/:permalink` | Resolve community -> list public skills | Public community landing | `public/skills#index` |
| Public skill view | Visitor | Open `/:permalink/public/skills/:id` | Load skill -> render public details | Skill page visible | `public/skills#show` |
| Community request submission | User | Submit new community form | Validate request -> store request -> notify support/admin | Request queued | `community_requests#new/create`; `CommunityRequest`, `UserMailer#community_request_created` |
| Invitation request in community | Visitor | Submit invitation request | Validate email -> create `InvitationRequest` -> moderators can process | Pending invitation request | `invitation_requests#create`; `InvitationRequest` |
| Invitation acceptance by token | Invitee | Open invitation link | Resolve token -> join/create membership -> consume invitation | User admitted to community | `invitations#show`; `Invitation`, `Membership` |

## 2. Admin and Governance Flows

| Flow name | Actors | Trigger | Main steps | Outputs | Related routes/classes |
|---|---|---|---|---|---|
| Admin community CRUD | Admin | Use admin communities UI | List/create/edit/delete community records | Community managed | `admin/communities` |
| Admin community duplication | Admin | Click duplicate in admin | Build duplication params -> clone structure/content | New duplicated community | `admin/communities#duplicate`; `Community::DuplicationJob` |
| Admin community statistics | Admin | Open admin statistics | Aggregate metrics -> render dashboard | Admin analytics view | `admin/communities#statistics` |
| Admin community request moderation | Admin | Accept/reject community request | List requests -> accept or destroy | Request resolved; community possibly created | `admin/community_requests#index/accept/destroy` |
| Admin static page management | Admin | Manage CMS pages | CRUD page by slug/content | Updated static pages | `admin/pages`; `Page` |
| Public static page rendering | User/Visitor | Open `/pages/:slug` | Resolve page by slug -> render | Public/help content displayed | `pages#show`; `Page` |

## 3. Community and Membership Flows

| Flow name | Actors | Trigger | Main steps | Outputs | Related routes/classes |
|---|---|---|---|---|---|
| Community profile update | Moderator | Edit community | Load community -> update description/config/public settings | Community settings changed | `communities#edit/update` |
| Community state/progression view | Member | Open state/progression pages | Query progression indicators -> render charts/tables | Progress visibility | `communities#state`, `communities#progression` |
| Community tree view | Member | Open tree page | Load skill hierarchy -> render tree UI | Learning map visible | `communities#tree` |
| Community-level duplication | Moderator | Start duplication flow | Open duplication form -> submit duplication request | Duplicated community artifact | `communities#duplication_form`, `communities#duplicate` |
| Community discussion access | Member | Open community discussion | Load filtered messages for community | Discussion timeline | `communities#messages`, `messages#index` |
| Membership creation | Moderator | Add member | Resolve user/invitation -> create `Membership` | Member added | `memberships#create`; `Membership` |
| Membership update | Moderator | Edit membership | Update attributes/team/public flags | Membership updated | `memberships#update` |
| Moderator promotion | Moderator | Toggle moderator role | Update membership moderator flag | Access rights elevated | `memberships#moderator` |
| Team management | Moderator | Create/edit/delete team | Maintain team records -> assign memberships by context | Team structure maintained | `teams#new/create/edit/update/delete`; `Team` |
| User directory browsing | Member | Open users list/profile | Query community users -> show cards/sidebar/details | Discoverable members | `users#index`, `users#show`, `users#sidebar` |
| User removal | Moderator | Remove user | Delete membership/user link (contextual) | User removed from community | `users#destroy` |
| Avatar removal | User/Moderator | Remove avatar | Clear avatar file reference | Avatar deleted | `users#destroy_avatar` |

## 4. Skill Catalog and Tree Flows

| Flow name | Actors | Trigger | Main steps | Outputs | Related routes/classes |
|---|---|---|---|---|---|
| Skill catalog browsing | Member | Open skills page | List skills by scope/team/filter | Skill list shown | `skills#index` |
| Skill creation | Moderator/authorized member | Submit skill form | `SkillForm#create` -> create skill + tasks + metadata | New skill node | `skills#new/create`; `SkillForm`, `Skill` |
| Skill update | Moderator/authorized member | Submit edit form | `SkillForm.update` -> persist updates | Skill updated | `skills#edit/update`; `SkillForm` |
| Skill deletion | Authorized actor | Delete skill | Authorization check -> run delete job -> redirect | Skill removed | `skills#destroy`; `Skill::DeleteJob` |
| Skill details and context | Member | Open skill page | Load evaluations/subscription/draft status | Detailed skill workspace | `skills#show` |
| Skill description page | Member | Open description tab | Load descriptive content/help | Learning guidance shown | `skills#description` |
| Skill progression page | Member/Moderator | Open progression tab | Aggregate learners progression data | Progress matrix by users | `skills#progression` |
| Skill-specific messaging | Subscriber | Open skill messages | Resolve subscription -> load skill messages | Skill thread | `skills#messages`, `messages#index` |
| Skill pinning | Subscriber | Pin skill | Toggle subscription pinned timestamp | Skill prioritized in UI | `skills#pin` |
| Skill subscription | Member | Click subscribe | Create `Subscription` for user+skill | Learner enrolled | `skills#subscribe`; `Subscription` |
| Skill unsubscription | Member | Click unsubscribe | Remove subscription | Learner unenrolled | `skills#unsubscribe` |
| Skill auto-evaluation trigger | Member/Moderator | Click auto-evaluation | Trigger skill auto-evaluation logic | Evaluation signal/state updated | `skills#auto_evaluation` |
| Prerequisite creation | Moderator | Add prerequisite edge | Create `Prerequisite` between skills | Skill dependency added | `prerequisites#create`; `Prerequisite` |
| Prerequisite deletion | Moderator | Remove prerequisite edge | Delete `Prerequisite` | Dependency removed | `prerequisites#destroy` |
| Mandatory prerequisite toggle | Moderator | Toggle mandatory | Update prerequisite flag | Constraint changed | `prerequisites#toggle_mandatory` |
| Task completion toggle | Subscriber | Toggle task done/undone | Create/delete `DoneTask` | Personal task status updated | `tasks#toggle`; `DoneTask` |
| Task deletion | Moderator | Delete task | Remove task from skill | Task removed | `tasks#destroy`; `Task` |
| Subscription completion | Learner/validator | Mark subscription complete | Set completion state/date | Skill marked completed | `subscriptions#complete` |
| Subscription uncomplete | Learner/validator | Revert completion | Clear completion state/date | Completion reverted | `subscriptions#uncomplete` |

## 5. Messaging and Collaboration Flows

| Flow name | Actors | Trigger | Main steps | Outputs | Related routes/classes |
|---|---|---|---|---|---|
| Post text message | Member | Submit message form | Validate target (`to_user`/community/skill/workspace) -> create `Message::Text` | Message created | `messages#create`; `Message::Text`, `MessagesController` |
| Upload attachment message | Member | Upload file | Store file via storage concern -> create `Message::Upload` | Upload message created | `messages#upload`; `Message::Upload`, `AwsFileStorage` |
| Read message timeline | Member | Open discussion pane | Filter/paginate messages (`before/after/pinned/hash_tag`) | Timeline rendered | `messages#index`; `RespondMessages#filter_messages` |
| Edit message | Author | Submit edit | Ensure ownership -> update text + `edited_at` | Message edited | `messages#update` |
| Delete message | Author/Moderator | Delete action | Soft/hard delete based on model/controller behavior | Message removed/hidden | `messages#destroy` |
| Mark message unread | Eligible user | Click unread | Permission check -> toggle `read_at` | Unread state updated | `messages#unread`; `User::Permissions`, `Message#mark_as_unread` |
| Pin/unpin message | Eligible user | Click pin | Authorization -> toggle `pinned_at` | Message pinned/unpinned | `messages#pin`; `Message#toggle_pinned_at` |
| Vote on message | Member | Click vote | `Vote.toggle` create/delete | Vote count changed | `messages#vote`; `Vote.toggle` |
| Download attachment | Member | Click download | `viewable_by?` check -> increment count -> redirect to file URL | File delivered + count incremented | `messages#download`; `Message#viewable_by?` |
| Search messages | Member | Submit search query | Full-text/file search in scope -> render results | Search results list | `messages#search_form`, `messages#search`; `Message.search` |
| Discussions digest view | Member | Open discussions page | Build discussion overview by context | Consolidated discussion page | `discussions#index` |
| Votes overview | Member/Moderator | Open votes page | Aggregate voted messages/users | Voting activity view | `votes#index` |

## 6. Poll and Event Flows

| Flow name | Actors | Trigger | Main steps | Outputs | Related routes/classes |
|---|---|---|---|---|---|
| Poll creation | Moderator/authorized member | Create poll from UI | Create poll + choices + publish in discussion | New poll active | `polls#create`; `Poll`, `PollChoice` |
| Poll response | Member | Submit poll answer | Validate eligibility -> persist `PollAnswer` | Vote recorded | `poll_answers#create`; `PollAnswer` |
| Poll viewing | Member | Open poll | Load question, options, aggregated answers | Poll details visible | `polls#show` |
| Poll deletion | Creator/Moderator | Delete poll | Remove poll and related UI references | Poll removed | `polls#destroy` |
| Event creation | Organizer | Submit event form | Validate schedule/capacity -> create event | Event published | `events#new/create`; `Event` |
| Event update | Organizer | Save event edits | Update event data | Event updated | `events#update` |
| Event deletion/cancellation | Organizer/System | Delete event or cancellation workflow | Delete event and notify participants as needed | Event removed + possible emails | `events#destroy`; `CancelEventJob`, `UserMailer#event_cancelled` |
| Event registration | Member | Register to event | Check capacity/time window -> add participation | User registered | `events#register`; `Participation` |
| Event unregistration | Member | Unregister | Remove participation | Slot freed | `events#unregister` |
| Attendance marking | Organizer | Toggle absent/present | Update participation presence flag | Attendance status changed | `events/participations#toggle`; `WaitingParticipation` |
| Event listing/consultation | Member | Open events pages | List and show event details | Event agenda visible | `events#index`, `events#show` |

## 7. Evaluation and Assessment Flows

| Flow name | Actors | Trigger | Main steps | Outputs | Related routes/classes |
|---|---|---|---|---|---|
| Evaluation definition | Examiner/Moderator | Create evaluation on skill | Build evaluation challenge with description/file | Evaluation available | `evaluations#new/create`; `Evaluation` |
| Evaluation editing | Examiner/Moderator | Edit evaluation | Update evaluation metadata/content | Evaluation updated | `evaluations#edit/update` |
| Evaluation listing/view | Learner/Examiner | Open evaluations pages | Load evaluations for skill or specific evaluation | Evaluation context visible | `evaluations#index`, `evaluations#show` |
| Evaluation deletion | Owner/Moderator | Delete evaluation | Remove evaluation | Evaluation removed | `evaluations#destroy` |
| Evaluation enable/disable | Owner/Moderator | Toggle availability | Set/clear `disabled_at` | Evaluation availability updated | `evaluations#disable`, `evaluations#enable` |
| Draft creation | Learner | Save draft | Persist `Evaluation::Draft` bound to subscription/evaluation | Draft stored | `evaluations/drafts#create`; `Evaluation::Draft` |
| Draft submission | Learner | Submit draft | Change draft to submitted workflow state | Draft sent for review | `evaluations/drafts#submit` |
| Exam creation | Examiner | Start exam for evaluation | Create exam instance tied to subscription/evaluation/examiner | Exam opened | `evaluations/exams#create`; `Evaluation::Exam` |
| Exam listing and viewing | Examiner/Learner | Open exams pages | Query exams and show details | Exam status visibility | `evaluations/exams#index`, `evaluations/exams#show` |
| Exam cancellation | Examiner | Cancel exam | Mark as canceled/stop workflow | Exam canceled | `evaluations/exams#cancel` |
| Exam resume | Examiner | Resume canceled exam | Re-enable active exam | Exam resumed | `evaluations/exams#resume` |
| Examiner reassignment | Authorized actor | Change examiner | Update examiner relationship | Ownership transferred | `evaluations/exams#change_examiner` |
| Evaluation note submission | Examiner | Submit note/decision | Create note with accepted/rejected flags | Decision recorded | `evaluations/notes#create`; `Evaluation::Note` |
| Homework upload | Learner | Upload homework file | Persist file + bind to evaluation/subscription | Homework artifact stored | `homeworks#upload`; `Homework` |
| Homework evaluation | Examiner | Evaluate homework | Mark approved/rejected -> notify learner | Homework decision + notifications | `homeworks#evaluate`; `Notification::*`, `UserMailer` |
| Homework deletion | Authorized actor | Delete homework | Remove homework artifact/record | Homework removed | `homeworks#destroy` |

## 8. Workspace and Portfolio Flows

| Flow name | Actors | Trigger | Main steps | Outputs | Related routes/classes |
|---|---|---|---|---|---|
| Workspace creation | Author | Submit workspace form | `WorkspaceForm` validates and creates workspace | Workspace draft created | `workspaces#create`; `WorkspaceForm`, `Workspace` |
| Workspace editing | Author | Save workspace edits | Update title/content/metadata | Workspace updated | `workspaces#edit/update` |
| Workspace viewing | Member/Public profile viewer | Open workspace page | Load workspace with access controls | Workspace displayed | `workspaces#show`, `profile/workspaces#show` |
| Workspace deletion | Author/Moderator | Delete workspace | Remove workspace and related references | Workspace removed | `workspaces#destroy` |
| Workspace publication | Author | Publish workspace | Set published state and emit related messages | Workspace published | `workspaces#publish`; `Message::WorkspacePublished*` |
| Workspace unpublication | Author/Moderator | Unpublish workspace | Revert publication state | Workspace hidden from published scope | `workspaces#unpublish` |
| Workspace approval | Moderator | Approve workspace | Set approval state + notify | Workspace approved | `workspaces#approve`; `Message::WorkspaceApprovedInternal` |
| Workspace rejection | Moderator | Reject workspace | Set rejection state + notify | Workspace rejected | `workspaces#reject`; `Message::WorkspaceRejectedInternal` |
| Workspace partnerships add/remove | Workspace owner | Manage partners | Create/destroy `Workspace::Partnership` | Collaboration members changed | `workspaces/partnerships#create/destroy`; `Workspace::Partnership` |
| Public profile visibility toggle | Member | Toggle public/private profile | Update membership/public visibility flags | Profile exposure updated | `profile/memberships#public/private` |
| Hidden profile items management | Member | Hide/unhide item | Create/delete `HiddenProfileItem` | Portfolio item hidden/visible | `profile/hidden_items#create/destroy`; `HiddenProfileItem` |
| Public membership profile view | Visitor/Member | Open profile URL | Resolve membership -> render allowed data | Public profile page | `profile/memberships#show` |

## 9. Notifications, Analytics, and Scheduled Flows

| Flow name | Actors | Trigger | Main steps | Outputs | Related routes/classes |
|---|---|---|---|---|---|
| In-app notifications center | Member | Open notifications page | Fetch membership notifications -> mark/read via UI interactions | Notification feed | `notifications#index`; `Notification` |
| Community statistics dashboard | Moderator | Open statistics pages | Aggregate activity/progression metrics | Community KPIs | `statistics#index`, `statistics#skills` |
| Daily summary email flow | Scheduler/System | Daily cron (`sqily:daily`) | Iterate eligible users -> compile summary -> send mail if content | Daily digest emails | `DailySummaryJob`, `UserMailer#daily_summary`, `lib/tasks/sqily.rake` |
| Weekly summary email flow | Scheduler/System | Weekly condition in daily task | Iterate memberships -> compile weekly data -> send | Weekly digest emails | `WeeklySummaryJob`, `UserMailer#weekly_summary` |
| Event reminder flow | Scheduler/System | Job execution before event | Find tomorrow events -> email participants | Reminder emails | `EventReminderNotificationJob`, `UserMailer#event_reminder` |
| Event cancellation notification flow | System | Event canceled/background job | Notify all non-owner participants -> destroy event | Cancellation emails + cleanup | `CancelEventJob`, `UserMailer#event_cancelled` |
| Badge awarding flow | Scheduler/System | Hourly cron (`sqily:hourly`) | Trigger badge rule processors | Badges attached to memberships | `Badge::*`, `lib/tasks/sqily.rake` |
| Poll finished notification flow | Scheduler/System | Hourly cron (`sqily:hourly`) | Trigger finished poll notifier | Poll-completion notifications | `Notification::PollFinished`, `lib/tasks/sqily.rake` |
| Deployment maintenance flow | DevOps/System | Post-deploy task | Run migrations -> assets precompile -> update crontab -> sync translations | Runtime updated and scheduled | `sqily:after_deploy`, `sqily:update_crontab`, `tolk:sync` |

## 10. Cross-Cutting Technical Flows

| Flow name | Actors | Trigger | Main steps | Outputs | Related routes/classes |
|---|---|---|---|---|---|
| Authorization guard flow | Any authenticated request | Controller before_actions | Resolve current user/community/membership -> enforce role checks | Protected access boundaries | `ApplicationController`, `PermissionsHelper`, `User::Permissions` |
| File upload persistence flow | Any file upload (message/homework/etc.) | File attached in forms | Assign `file_node` -> save file via storage concern -> generate URL | File persisted and retrievable | `AwsFileStorage`, `PublicFileStorage`, `Message::Upload`, `Homework` |
| Mention/hash-tag extraction flow | Message text save | Save `Message::Text` | Parse text for tags/mentions -> persist `HashTag` -> enable filtering/notifications | Tag metadata + searchable content | `HashTaggable`, `HashTag`, `Message` |
| Monitoring instrumentation flow | Scheduled tasks and runtime | Code path wrapped with metrics | `RorVsWild.measure_code` and error capture around jobs | Performance/error telemetry | `RorVsWild`, `lib/tasks/sqily.rake`, jobs |

## Notes
- This flow map is intentionally business-flow oriented and grouped by domain.
- Some flows share underlying endpoints/controllers but are separated here by intent.
