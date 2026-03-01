# Product Backlog

This backlog is derived from the current codebase features (routes, models, controllers, jobs, and views) to describe what the product already covers.

## Feature 1 - User Registration and Authentication

**User Story**
As a new or returning user,
I want to create an account and securely sign in/out,
so that I can access my community workspace.

**Acceptance Criteria**
```gherkin
Scenario: User signs in with valid credentials
  Given a user account exists with email "alice@example.org"
  When the user submits the login form with valid credentials
  Then the user session is created and the user is redirected to the community area
```
Examples / Test Cases:

| email | password | expected_result |
|---|---|---|
| alice@example.org | ValidPassword123 | Session created |

## Feature 2 - Password Reset

**User Story**
As a user who forgot my password,
I want to request a reset link,
so that I can regain access to my account.

**Acceptance Criteria**
```gherkin
Scenario: Password reset request is created
  Given a registered user with email "bob@example.org"
  When the user requests a password reset
  Then a password reset token is generated and reset instructions are sent
```
Examples / Test Cases:

| email | token_generated | email_sent |
|---|---|---|
| bob@example.org | true | true |

## Feature 3 - Community Discovery and Join Requests

**User Story**
As a prospective learner,
I want to browse public communities and request new ones,
so that I can join the right learning environment.

**Acceptance Criteria**
```gherkin
Scenario: User submits a new community request
  Given a signed-in user wants a new community
  When the user submits a community request with a name and description
  Then the request is saved for admin review
```
Examples / Test Cases:

| community_name | description | result |
|---|---|---|
| Digital Fabrication 2026 | Need a collaborative space for prototyping skills | Request stored |

## Feature 4 - Community Administration and Duplication

**User Story**
As a community administrator,
I want to manage community settings and duplicate an existing community,
so that I can scale proven learning structures.

**Acceptance Criteria**
```gherkin
Scenario: Admin duplicates a community
  Given an admin has access to community "hep-vd"
  When the admin triggers duplication
  Then a new community copy is created with the selected configuration
```
Examples / Test Cases:

| source_permalink | duplicated | result |
|---|---|---|
| hep-vd | true | New community created |

## Feature 5 - Invitations and Invitation Requests

**User Story**
As a moderator,
I want to invite people directly or process invitation requests,
so that community access is controlled and trackable.

**Acceptance Criteria**
```gherkin
Scenario: Moderator sends an invitation
  Given a moderator is on the invitations page
  When the moderator invites "charlie@example.org"
  Then an invitation token is created and can be used for onboarding
```
Examples / Test Cases:

| invited_email | token_created | invitation_listed |
|---|---|---|
| charlie@example.org | true | true |

## Feature 6 - Membership and Role Management

**User Story**
As a moderator,
I want to manage memberships and moderator rights,
so that governance responsibilities are delegated properly.

**Acceptance Criteria**
```gherkin
Scenario: Promote a member to moderator
  Given a membership exists for user "diane"
  When a moderator enables moderator mode on that membership
  Then the membership gains moderator privileges
```
Examples / Test Cases:

| membership_id | previous_moderator | new_moderator |
|---|---|---|
| 42 | false | true |

## Feature 7 - Team Management

**User Story**
As a moderator,
I want to create and maintain teams,
so that members can be organized into meaningful groups.

**Acceptance Criteria**
```gherkin
Scenario: Create a team
  Given a moderator has team management access
  When the moderator submits a new team name
  Then the team is created and available for member assignment
```
Examples / Test Cases:

| team_name | created |
|---|---|
| Apprentices Group A | true |

## Feature 8 - Skill Catalog and Skill Tree Management

**User Story**
As a pedagogue,
I want to create, edit, and organize skills,
so that learners can follow competency-based pathways.

**Acceptance Criteria**
```gherkin
Scenario: Create a new skill in a community
  Given the user has skill creation rights
  When the user creates skill "SQL Window Functions"
  Then the skill appears in the community skill list and tree
```
Examples / Test Cases:

| skill_title | visible_in_list | visible_in_tree |
|---|---|---|
| SQL Window Functions | true | true |

## Feature 9 - Skill Prerequisites

**User Story**
As a skill designer,
I want to add prerequisite links (optional or mandatory),
so that learning progression remains coherent.

**Acceptance Criteria**
```gherkin
Scenario: Mark prerequisite as mandatory
  Given a prerequisite link exists between two skills
  When the author toggles mandatory mode
  Then the prerequisite is saved as mandatory
```
Examples / Test Cases:

| from_skill | to_skill | mandatory |
|---|---|---|
| SQL Basics | SQL Window Functions | true |

## Feature 10 - Skill Subscription and Task Completion

**User Story**
As a learner,
I want to subscribe to skills and track my tasks,
so that I can monitor my progress.

**Acceptance Criteria**
```gherkin
Scenario: Learner toggles a task as done
  Given the learner is subscribed to a skill with tasks
  When the learner toggles task "Read chapter 3" to done
  Then the task completion is stored for that learner
```
Examples / Test Cases:

| subscription_id | task_name | done |
|---|---|---|
| 128 | Read chapter 3 | true |

## Feature 11 - Community, Skill, and Workspace Messaging

**User Story**
As a member,
I want to exchange messages in communities, skills, and workspaces,
so that collaboration can happen asynchronously.

**Acceptance Criteria**
```gherkin
Scenario: Post a text message in a community discussion
  Given a member has access to community discussion
  When the member posts "Can we review challenge 2 tomorrow?"
  Then the message is visible in the discussion thread
```
Examples / Test Cases:

| target | message_text | visible |
|---|---|---|
| community:hep-vd | Can we review challenge 2 tomorrow? | true |

## Feature 12 - Message Enhancements (Pin, Vote, Unread, Edit, Search, Upload)

**User Story**
As a member,
I want advanced message actions,
so that important information is easier to find and manage.

**Acceptance Criteria**
```gherkin
Scenario: Search messages by keyword
  Given discussion messages contain "exam", "event", and "poll"
  When the user searches for "exam"
  Then only matching messages are returned in results
```
Examples / Test Cases:

| query | messages_found |
|---|---|
| exam | 3 |

## Feature 13 - Poll Creation and Voting

**User Story**
As a moderator or member with rights,
I want to create polls and collect answers,
so that group decisions can be made transparently.

**Acceptance Criteria**
```gherkin
Scenario: User submits a poll answer
  Given a poll has open choices
  When a user selects one choice and submits
  Then the answer is recorded and reflected in poll results
```
Examples / Test Cases:

| poll_question | selected_choice | recorded |
|---|---|---|
| Preferred workshop date? | 2026-04-02 | true |

## Feature 14 - Event Planning and Registration

**User Story**
As an organizer,
I want to schedule events with registration,
so that community sessions are coordinated.

**Acceptance Criteria**
```gherkin
Scenario: Member registers for an event with available seats
  Given event "Peer coaching" has max 20 seats and 12 registrations
  When a member registers
  Then the member is added as participant
```
Examples / Test Cases:

| event_title | max_participations | current_participations | registration_result |
|---|---|---|---|
| Peer coaching | 20 | 12 | success |

## Feature 15 - Attendance Tracking for Event Participations

**User Story**
As an organizer,
I want to mark participation status (present/absent),
so that attendance is tracked for follow-up.

**Acceptance Criteria**
```gherkin
Scenario: Organizer marks participant absent
  Given a participant is registered for event "Sprint review"
  When the organizer toggles participation to absent
  Then participation status is updated
```
Examples / Test Cases:

| event_title | participant_id | absent |
|---|---|---|
| Sprint review | 991 | true |

## Feature 16 - Evaluation Definition per Skill

**User Story**
As an examiner,
I want to define evaluations for a skill,
so that learners have clear validation challenges.

**Acceptance Criteria**
```gherkin
Scenario: Create a new evaluation
  Given an examiner can manage evaluations on a skill
  When the examiner creates evaluation "Build normalized schema"
  Then the evaluation is available in the skill evaluation list
```
Examples / Test Cases:

| skill | evaluation_title | created |
|---|---|---|
| SQL Fundamentals | Build normalized schema | true |

## Feature 17 - Evaluation Drafts and Submission Workflow

**User Story**
As a learner,
I want to draft and submit evaluation content,
so that I can prepare my final attempt before review.

**Acceptance Criteria**
```gherkin
Scenario: Learner submits an evaluation draft
  Given a learner has written draft content
  When the learner submits the draft
  Then the draft state changes to submitted for examiner processing
```
Examples / Test Cases:

| evaluation_id | draft_content_length | submitted |
|---|---|---|
| 75 | 1800 | true |

## Feature 18 - Exam Session Lifecycle

**User Story**
As an examiner,
I want to create, cancel, resume, and reassign exams,
so that assessment logistics are manageable.

**Acceptance Criteria**
```gherkin
Scenario: Change examiner on an existing exam
  Given exam #55 is active and linked to examiner "emma"
  When an authorized user changes examiner to "frank"
  Then exam #55 is now assigned to "frank"
```
Examples / Test Cases:

| exam_id | old_examiner | new_examiner | updated |
|---|---|---|---|
| 55 | emma | frank | true |

## Feature 19 - Evaluation Notes and Decisioning

**User Story**
As an evaluator,
I want to attach notes and mark accepted/rejected outcomes,
so that learners receive explicit assessment feedback.

**Acceptance Criteria**
```gherkin
Scenario: Evaluator rejects an exam with note
  Given an exam exists for a learner submission
  When the evaluator submits a note marked rejected
  Then rejection status is stored with the note content
```
Examples / Test Cases:

| exam_id | is_accepted | is_rejected | note_excerpt |
|---|---|---|---|
| 55 | false | true | Missing data integrity checks |

## Feature 20 - Homework Upload and Review

**User Story**
As a learner and evaluator,
I want homework files to be uploaded and reviewed,
so that evidence artifacts support the evaluation process.

**Acceptance Criteria**
```gherkin
Scenario: Learner uploads homework for an evaluation
  Given a homework slot exists for a subscription
  When the learner uploads file "schema-v2.pdf"
  Then the homework record stores the uploaded file reference
```
Examples / Test Cases:

| homework_id | file_name | upload_saved |
|---|---|---|
| 210 | schema-v2.pdf | true |

## Feature 21 - Workspace Authoring, Versioning, and Publication

**User Story**
As a content author,
I want to create workspaces, publish them, and manage versions/approval states,
so that learning resources can progress from draft to validated publication.

**Acceptance Criteria**
```gherkin
Scenario: Publish a workspace
  Given workspace "Data modeling portfolio" is in draft state
  When the author publishes the workspace
  Then the workspace becomes visible as published and status is logged
```
Examples / Test Cases:

| workspace_id | previous_state | new_state |
|---|---|---|
| 310 | draft | published |

## Feature 22 - Workspace Partnerships

**User Story**
As a workspace owner,
I want to add partner members to a workspace,
so that collaborative delivery is supported.

**Acceptance Criteria**
```gherkin
Scenario: Add a workspace partner
  Given a workspace owner opens partnership settings
  When the owner adds membership #402 as a partner
  Then a workspace partnership is created
```
Examples / Test Cases:

| workspace_id | partner_membership_id | partnership_created |
|---|---|---|
| 310 | 402 | true |

## Feature 23 - Public Profiles and Visibility Controls

**User Story**
As a member,
I want to expose or hide profile items and portfolio elements,
so that I control what others can see publicly.

**Acceptance Criteria**
```gherkin
Scenario: Hide a profile workspace item
  Given a member profile currently shows workspace #310
  When the member hides workspace #310 from profile
  Then the workspace is no longer listed on the public profile page
```
Examples / Test Cases:

| membership_id | hidden_workspace_id | visible_on_public_profile |
|---|---|---|
| 87 | 310 | false |

## Feature 24 - Notifications Center and Digest Emails

**User Story**
As a member,
I want in-app notifications and periodic email summaries,
so that I stay aware of key events (mentions, votes, badges, homework, polls).

**Acceptance Criteria**
```gherkin
Scenario: Mention notification appears for tagged user
  Given user "gina" is mentioned in a message
  When notification jobs are processed
  Then "gina" sees a mention notification in notifications list
```
Examples / Test Cases:

| notification_type | recipient | visible_in_center |
|---|---|---|
| mention | gina | true |

## Feature 25 - Community Statistics and User Analytics

**User Story**
As a moderator,
I want dashboards for community and skill progression metrics,
so that I can monitor engagement and learning outcomes.

**Acceptance Criteria**
```gherkin
Scenario: Open community statistics page
  Given a moderator has access to statistics
  When the moderator opens the statistics dashboard
  Then aggregate community metrics are displayed
```
Examples / Test Cases:

| community | metric_name | value_type |
|---|---|---|
| hep-vd | active_members | integer |

## Feature 26 - Badges and Recognition System

**User Story**
As a member,
I want automatic badge attribution,
so that my contributions and progression are recognized visibly.

**Acceptance Criteria**
```gherkin
Scenario: Badge is awarded when criteria are met
  Given a member reaches criteria for badge "Participant"
  When badge processing runs
  Then the badge is attached to the member profile
```
Examples / Test Cases:

| membership_id | badge_type | awarded |
|---|---|---|
| 87 | Participant | true |

## Feature 27 - Static Pages and FAQ Management

**User Story**
As an admin,
I want to manage static pages (including FAQ content),
so that platform guidance stays current.

**Acceptance Criteria**
```gherkin
Scenario: Admin publishes a page with slug
  Given an admin creates page "Onboarding Guide" with slug "onboarding"
  When the page is saved
  Then users can access it at `/pages/onboarding`
```
Examples / Test Cases:

| title | slug | publicly_reachable |
|---|---|---|
| Onboarding Guide | onboarding | true |
