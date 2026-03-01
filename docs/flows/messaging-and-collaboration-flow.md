# Messaging and Collaboration - Detailed Flow

## Scope
This flow covers posting/reading messages (community, skill, workspace, direct), live refresh, and key collaboration actions (pin, vote, unread, file download, search).

## End-to-end implementation
1. UI entry points
- `app/views/messages/index.html.erb` renders timeline container and message form.
- `app/views/messages/_form.html.erb` posts text messages and provides modals for file upload/poll/event.
- `app/assets/javascripts/sqily/message_form.js` intercepts submit and sends AJAX for non-archive view.
- `app/assets/javascripts/sqily/message/puller.js` polls every 5s for new/edited messages.

2. Message creation path
- UI submit -> `MessagesController#create`.
- `before_action`: `authenticate_user`, `must_be_membership`, `must_have_a_subscription` (only if skill-targeted).
- Controller creates `Message::Text.create!(message_attributes.merge(from_user: current_user))`.
- On XHR request, returns partial HTML; client appends to timeline (`afterSend`).
- On non-XHR request, redirects based on target (`to_user`/`to_skill`/`to_workspace`/community).

3. Message read and refresh path
- Initial list via `MessagesController#index` and `filter_messages` from `RespondMessages` concern.
- Scope selected by context:
  - direct: `.between(current_user, user)` and mark incoming unread as read,
  - skill: `.to_skill(skill)` and touch subscription `last_read_at`,
  - workspace: `.to_workspace(workspace)`,
  - community: `.to_community(current_community)` and touch membership `last_read_at`.
- Poller hits same endpoint with JSON accept header; response updates:
  - edited messages,
  - unread skill activity markers,
  - newly arrived messages.

4. Collaboration actions
- `pin`: `MessagesController#pin` -> `Message#pinnable_by?` -> `toggle_pinned_at`.
- `vote`: `MessagesController#vote` -> `Vote.toggle(current_user, message)`.
- `unread`: `MessagesController#unread` gated by `current_user.permissions.mark_message_as_unread?`.
- `update`: only author scope (`Message.from_user(current_user).find(params[:id])`).
- `destroy`: author or moderator.
- `download`: `Message#viewable_by?(current_user)` + increments `download_count`.

5. Search path
- `MessagesController#search` runs separate queries for `Message::Text` and `Message::Upload` using `Message.search` and full-text/file matching (`text_search`).

## Validations, checks, and rules
- `Message` validations:
  - recipient must exist (`to_user` or `to_community` or `to_skill` or `to_workspace`),
  - sender required,
  - sender cannot target self in direct message.
- Skill messaging requires active subscription (`must_have_a_subscription`).
- Non-moderators only see `not_deleted` scope in filtered results.
- Hash tags are extracted on text save (`watch_hash_tags_on :text`).

## Side effects and storage
- Persistent storage: `messages`, `votes`, `messages_users`, `hash_tags`.
- File messages (`Message::Upload`) use `AwsFileStorage` concern and save blobs to S3/local URL.
- Reading in skill/community updates read timestamps (`subscriptions.last_read_at` / `memberships.last_read_at`).
- Download action increments `messages.download_count`.

## Sequence diagram
```mermaid
sequenceDiagram
  autonumber
  actor U as User
  participant UI as Messages UI + Sqily.MessageForm
  participant MC as MessagesController
  participant AC as ApplicationController guards
  participant M as Message::Text / Message
  participant RM as RespondMessages
  participant V as Vote
  participant DB as PostgreSQL
  participant P as Sqily.Message.Puller

  U->>UI: Type message and press Enter
  UI->>UI: Sqily.MessageForm.submit(event)
  alt Reading archives
    UI->>MC: Normal POST /messages
  else Live mode
    UI->>MC: AJAX POST /messages (X-Requested-With)
  end

  MC->>AC: authenticate_user + must_be_membership
  MC->>AC: must_have_a_subscription (if to_skill_id present)
  MC->>M: Message::Text.create!(attrs + from_user)
  M->>DB: INSERT messages (+ hashtag side effects)

  alt XHR
    MC-->>UI: Render new message partial
    UI->>UI: Append message to #messages and scroll
  else HTML
    MC-->>UI: Redirect based on target context
  end

  loop every 5s
    P->>MC: GET /messages (JSON)
    MC->>RM: filter_messages(scope)
    RM->>DB: Apply before/after/pinned/not_deleted/hash_tag filters
    MC->>DB: Load new_messages + edited_messages + unread skill ids
    MC-->>P: JSON payload
    P->>UI: Patch edited nodes + append new messages
  end

  U->>UI: Click pin / vote / unread / download
  alt pin
    UI->>MC: POST /messages/:id/pin
    MC->>M: pinnable_by?(current_user)
    M->>DB: UPDATE pinned_at
  else vote
    UI->>MC: POST /messages/:id/vote
    MC->>V: Vote.toggle(current_user, message)
    V->>DB: INSERT or DELETE vote row
  else unread
    UI->>MC: POST /messages/:id/unread
    MC->>M: mark_as_unread (permission-checked)
    M->>DB: UPDATE read_at
  else download
    UI->>MC: GET /messages/:id/download
    MC->>M: viewable_by?(current_user)
    M->>DB: UPDATE download_count
    MC-->>UI: Redirect to file URL
  end
```
