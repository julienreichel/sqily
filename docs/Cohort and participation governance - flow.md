# Cohort and Participation Governance (Moderator/Teacher) - Detailed Flow

## Scope
This flow covers moderator governance of people and participation: invitations, invitation-request moderation, role/team management, and event participation oversight.

## End-to-end implementation
1. UI entry points
- Invitation management page: `app/views/invitations/index.html.erb`.
- Invitation request management on same area and public request form (`InvitationRequestsController#index/create`).
- Team create/edit screens: `app/views/teams/new.html.erb`, `edit.html.erb`.
- Event participation controls in `app/views/messages/_event_created.html.erb` (avatar toggles for attendance).

2. Invitation and join governance
- `InvitationsController#index/create/destroy` guarded by `must_be_moderator`.
- `create` accepts bulk emails (`split("\n")`) and calls `Invitation.bulk_create`.
- `Invitation.find_or_create` validates each email and uniqueness by community.
- `Invitation#after_create` sends invite email via `UserMailer.invitation`.
- Invitee flow via token (`InvitationsController#show`):
  - if logged in: `invitation.complete(current_user)` -> `community.add_user(user)` + destroy invitation,
  - else store `current_invitation` and redirect to registration.

3. Invitation request moderation
- Non-members can submit requests via `InvitationRequestsController#create`.
- `InvitationRequest.after_create` notifies all moderators (`UserMailer.invitation_request`).
- Moderator accepts with `InvitationRequest#accept!`:
  - creates invitation (`Invitation.find_or_create`) then deletes request.
- Moderator may also destroy request.

4. Membership role and team governance
- Moderator role toggling: `MembershipsController#moderator` -> `membership.toggle!(:moderator)`.
- Team management in `TeamsController` guarded by `current_membership.permissions.create_teams?`.
- On team create/update:
  - persist `Team`,
  - call `Team#update_user_ids(user_ids)` to bulk move/remove memberships using `update_all`.

5. Participation governance for events
- Event registration/unregistration through `EventsController#register/#unregister` and `Event#register/#unregister`.
- Moderator/owner attendance marking:
  - JS `Sqily.Event.Participation` calls `Events::ParticipationsController#toggle`.
  - Permission gate: `current_user.permissions.can_toggle_participations_of_event?(event)`.
  - `Participation#toggle_presence` cycles presence state.

## Validations, checks, and rules
- Invitation email format validation and uniqueness per community.
- Invitation request uniqueness by community/email.
- Team creation requires moderator-level membership permission.
- Event participation toggle permission requires event owner and event already started (`can_toggle_participations_of_event?`).
- Invite acceptance token must resolve inside current community context.

## Side effects and storage
- Persistent storage: `invitations`, `invitation_requests`, `memberships`, `teams`, `participations`, `waiting_participations`.
- Side effects:
  - invitation emails (`UserMailer.invitation`),
  - moderator notifications for invitation requests,
  - waiting-list promotion emails (`UserMailer.waiting_participation_finished`),
  - bulk membership team reassignment through SQL updates.

## Sequence diagram
```mermaid
sequenceDiagram
  autonumber
  actor Mod as Moderator/Teacher
  actor Vis as Visitor/Prospect
  actor Mem as Member
  participant UI as Governance UI (invites/teams/events)
  participant IC as InvitationsController
  participant IRC as InvitationRequestsController
  participant I as Invitation
  participant IR as InvitationRequest
  participant MC as MembershipsController
  participant TC as TeamsController
  participant T as Team
  participant EPC as Events::ParticipationsController
  participant P as Participation
  participant DB as PostgreSQL
  participant Mail as UserMailer

  Mod->>UI: Submit bulk invite emails
  UI->>IC: POST /invitations
  IC->>I: bulk_create(community, emails)
  loop each email
    I->>I: find_or_create
    I->>DB: INSERT invitation (token generated)
    I->>Mail: invitation(invitation).deliver_now
  end
  IC-->>UI: Redirect index (or show invalid emails)

  Vis->>UI: Submit invitation request
  UI->>IRC: POST /invitation_requests
  IRC->>IR: create(email)
  IR->>DB: INSERT invitation_request
  IR->>Mail: invitation_request(...).deliver_now to moderators

  Mod->>UI: Accept invitation request
  UI->>IRC: PUT /invitation_requests/:id
  IRC->>IR: accept!()
  IR->>I: find_or_create(community, email)
  I->>DB: INSERT invitation if absent
  IR->>DB: DELETE invitation_request
  IRC-->>UI: Redirect invitations page

  Mod->>UI: Toggle member moderator role
  UI->>MC: PUT /memberships/:id/moderator
  MC->>DB: UPDATE memberships.moderator = !moderator

  Mod->>UI: Create/update team and assign users
  UI->>TC: POST/PATCH /teams
  TC->>TC: must_be_allowed_to_create_teams
  TC->>DB: INSERT/UPDATE teams
  TC->>T: update_user_ids(user_ids)
  T->>DB: Bulk UPDATE memberships.team_id

  Mod->>UI: Toggle event attendance avatar
  UI->>EPC: POST /events/:event_id/participations/:id/absent
  EPC->>EPC: can_toggle_participations_of_event?(event)
  EPC->>P: toggle_presence()
  P->>DB: UPDATE participations.confirmed
  EPC-->>UI: JSON updated status
```
