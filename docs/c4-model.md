# C4 Model Diagrams

The diagrams below describe the Sqily architecture at four C4 zoom levels, from ecosystem context down to code structure.

## 1) Context Diagram - Where does this system fit in the world?

Architectural question answered: **Who uses Sqily and why, and which external systems does it depend on?**

```mermaid
C4Context
  title System Context - Sqily

  Person(student, "Learner", "Subscribes to skills, submits evaluations/homework, collaborates with peers")
  Person(moderator, "Moderator / Teacher", "Manages communities, skills, invitations, assessments, events")
  Person(admin, "Platform Admin", "Manages community requests and static pages in admin backoffice")

  System(sqily, "Sqily Platform", "School communication and competency-validation platform")

  System_Ext(mail, "SMTP / Email Provider", "Delivers transactional and digest emails")
  System_Ext(storage, "Object Storage (AWS S3)", "Stores uploaded files and attachments")
  System_Ext(obs, "RorVsWild", "Application monitoring and performance metrics")

  Rel(student, sqily, "Uses")
  Rel(moderator, sqily, "Uses")
  Rel(admin, sqily, "Administrates")

  Rel(sqily, mail, "Sends emails", "SMTP")
  Rel(sqily, storage, "Uploads/downloads files", "S3 API")
  Rel(sqily, obs, "Reports metrics/errors")
```

## 2) Container Diagram - What are the main building blocks?

Architectural question answered: **How is Sqily deployed and how do runtime containers communicate?**

```mermaid
C4Container
  title Container Diagram - Sqily

  Person(user, "User", "Learner, moderator, admin")

  System_Boundary(s1, "Sqily") {
    Container(web, "Rails Web Application", "Ruby on Rails 7", "MVC app: communities, skills, messaging, evaluations, workspaces, admin")
    ContainerDb(db, "PostgreSQL", "PostgreSQL 16", "Persistent data: users, memberships, skills, messages, evaluations, events, notifications")
    Container(scheduler, "Cron Scheduler", "Whenever + rake tasks", "Triggers hourly/daily jobs and maintenance tasks")
    Container(jobs, "Background Job Runtime", "ActiveJob (in-process)", "Runs summary, reminder, badge, poll and lifecycle jobs")
  }

  System_Ext(s3, "Object Storage", "AWS S3 or local public storage")
  System_Ext(smtp, "SMTP Server", "Outgoing email")
  System_Ext(rorvswild, "RorVsWild", "Monitoring")

  Rel(user, web, "Uses via browser", "HTTPS")
  Rel(web, db, "Reads/writes", "ActiveRecord/SQL")
  Rel(scheduler, jobs, "Triggers", "rake / ActiveJob")
  Rel(web, jobs, "Invokes jobs", "ActiveJob")
  Rel(web, s3, "Stores and serves files", "S3 API / public URLs")
  Rel(web, smtp, "Sends notifications and summaries", "SMTP")
  Rel(web, rorvswild, "Sends metrics and errors")
```

## 3) Component Diagram - What lives inside the backend container?

Architectural question answered: **What are the major internal responsibilities inside the Rails backend?**

```mermaid
C4Component
  title Component Diagram - Rails Web Application Container

  Container_Boundary(backend, "Rails Web Application") {
    Component(router, "Routes", "config/routes.rb", "Maps HTTP endpoints to controllers")
    Component(ctrl, "Controllers", "ActionController", "Request orchestration, auth checks, view model preparation")
    Component(authz, "AuthN/AuthZ", "Controller concerns + permissions", "Current user resolution and role-based access checks")
    Component(domain, "Domain Models", "ActiveRecord models", "Business entities: Community, Skill, Evaluation, Message, Workspace, etc.")
    Component(forms, "Form/Service Objects", "app/lib/*_form.rb", "Complex creation/update workflows (e.g., SkillForm, WorkspaceForm)")
    Component(jobs, "Jobs", "ActiveJob", "Scheduled/background processes and lifecycle operations")
    Component(mailers, "Mailers", "ActionMailer", "Email composition and delivery")
    Component(files, "File Storage Adapters", "AwsFileStorage/PublicFileStorage", "Upload/download and file URL generation")
    Component(views, "Views + JS/CSS assets", "ERB + Sprockets", "HTML rendering and client-side interactions")
  }

  ContainerDb(pg, "PostgreSQL", "Main relational data store")
  System_Ext(s3x, "Object storage", "S3/local file store")
  System_Ext(smtpx, "SMTP", "Email delivery")

  Rel(router, ctrl, "Dispatches requests to")
  Rel(ctrl, authz, "Applies")
  Rel(ctrl, forms, "Uses for complex writes")
  Rel(ctrl, domain, "Reads/writes")
  Rel(domain, pg, "Persists to")
  Rel(ctrl, views, "Renders")
  Rel(domain, files, "Delegates file persistence")
  Rel(files, s3x, "Stores/retrieves blobs")
  Rel(jobs, domain, "Processes domain workflows")
  Rel(jobs, mailers, "Triggers emails")
  Rel(mailers, smtpx, "Sends")
```

## 4) Code Diagrams - What's under the hood?

Architectural question answered: **How are the major backend components implemented at file/class level?**

### 4.1 Controllers Component (HTTP orchestration)

```mermaid
classDiagram
  direction LR

  class ApplicationController {
    +authenticate_user()
    +must_be_membership()
    +must_be_moderator()
    +current_community()
    +current_membership()
  }

  class SkillsController {
    +index()
    +show()
    +create()
    +update()
    +subscribe()
    +unsubscribe()
  }

  class MessagesController {
    +index()
    +create()
    +upload()
    +search()
    +pin()
    +vote()
  }

  class WorkspacesController {
    +show()
    +create()
    +update()
    +publish()
    +approve()
  }

  class EvaluationsController {
    +index()
    +create()
    +update()
    +disable()
    +enable()
  }

  ApplicationController <|-- SkillsController
  ApplicationController <|-- MessagesController
  ApplicationController <|-- WorkspacesController
  ApplicationController <|-- EvaluationsController
```

### 4.2 AuthN/AuthZ Component (access rules and identity context)

```mermaid
classDiagram
  direction LR

  class CurrentUser {
    +current_user()
  }

  class CurrentInvitation {
    +current_invitation()
  }

  class PermissionsHelper {
    +admin?()
    +moderator?()
  }

  class UserPermissions {
    +destroy_skill?()
    +mark_message_as_unread?()
    +edit_workspace?()
  }

  class ApplicationController {
    +authenticate_user()
    +must_be_membership()
    +must_be_moderator()
  }

  class User {
    +permissions()
    +membership_for(community)
  }

  ApplicationController ..> CurrentUser : includes
  ApplicationController ..> CurrentInvitation : includes
  ApplicationController ..> PermissionsHelper : includes
  User --> UserPermissions : builds
  ApplicationController --> User : checks context and rights
```

### 4.3 Domain Models Component (business entities)

```mermaid
classDiagram
  direction TB

  class Community
  class Membership
  class User
  class Skill
  class Subscription
  class Evaluation
  class Message
  class Workspace
  class Poll
  class Event
  class Notification

  Community "1" --> "*" Membership
  User "1" --> "*" Membership
  Community "1" --> "*" Skill
  User "1" --> "*" Subscription
  Skill "1" --> "*" Subscription
  Skill "1" --> "*" Evaluation
  User "1" --> "*" Message : from_user
  Skill "1" --> "*" Message : to_skill
  Workspace "1" --> "*" Message : to_workspace
  Community "1" --> "*" Poll
  Community "1" --> "*" Event
  Membership "1" --> "*" Notification
```

### 4.4 Form/Service Objects Component (complex write workflows)

```mermaid
classDiagram
  direction LR

  class SkillsController {
    +create()
    +update()
  }

  class WorkspacesController {
    +create()
    +update()
  }

  class CommunityRequestsController {
    +create()
  }

  class SkillForm {
    +create(params)
    +self.update(skill, params)
    +skill
  }

  class WorkspaceForm {
    +create(params)
    +self.update(workspace, params)
  }

  class CommunityRequestForm {
    +create(params)
  }

  class Skill
  class Workspace
  class CommunityRequest

  SkillsController --> SkillForm
  WorkspacesController --> WorkspaceForm
  CommunityRequestsController --> CommunityRequestForm
  SkillForm --> Skill
  WorkspaceForm --> Workspace
  CommunityRequestForm --> CommunityRequest
```

### 4.5 Jobs Component (scheduled/background execution)

```mermaid
classDiagram
  direction LR

  class DailySummaryJob {
    +self.perform_now_for_all_users()
    +perform(user)
  }

  class WeeklySummaryJob {
    +self.perform_for_all_membeships()
    +perform(membership)
  }

  class EventReminderNotificationJob {
    +perform()
  }

  class CancelEventJob {
    +perform(event_id)
  }

  class UserMailer {
    +daily_summary(summary)
    +weekly_summary(summary)
    +event_reminder(event, user)
    +event_cancelled(event, user)
  }

  class User
  class Membership
  class Event

  DailySummaryJob --> User : iterate targets
  WeeklySummaryJob --> Membership : iterate targets
  EventReminderNotificationJob --> Event : query upcoming
  CancelEventJob --> Event : load and delete
  DailySummaryJob --> UserMailer : deliver_now
  WeeklySummaryJob --> UserMailer : deliver_now
  EventReminderNotificationJob --> UserMailer : deliver_now
  CancelEventJob --> UserMailer : deliver_now
```

### 4.6 Mailers Component (email composition)

```mermaid
classDiagram
  direction LR

  class ApplicationMailer

  class UserMailer {
    +invitation()
    +password_reset()
    +daily_summary()
    +weekly_summary()
    +event_reminder()
    +unread_notifications()
  }

  class ExportMailer
  class ExamMailer

  ApplicationMailer <|-- UserMailer
  ApplicationMailer <|-- ExportMailer
  ApplicationMailer <|-- ExamMailer
```

### 4.7 File Storage Adapters Component (binary/file persistence)

```mermaid
classDiagram
  direction LR

  class AwsFileStorage {
    +file=(uploaded_file)
    +save_file()
    +file_url()
    +bucket()
  }

  class PublicFileStorage {
    +file=(uploaded_file)
    +save_file()
    +file_url()
    +file_system_path()
  }

  class MessageUpload
  class Homework
  class Evaluation
  class Event

  MessageUpload ..> AwsFileStorage : include
  Homework ..> AwsFileStorage : include
  Evaluation ..> AwsFileStorage : include
  Event ..> AwsFileStorage : include
```

### 4.8 Views + Assets Component (presentation layer)

```mermaid
flowchart LR
  C["Controller action"] --> V["ERB views (app/views/*)"]
  V --> L["Layouts/partials"]
  V --> J["Sprockets JS modules (app/assets/javascripts/sqily/*)"]
  V --> S["CSS stylesheets (app/assets/stylesheets/*)"]
  J --> B["Browser DOM interactions"]
  S --> B
```

### Scope note
- Level 4 now includes one diagram per major component from the Level 3 backend view.
- Representative file references:
  - Controllers: `app/controllers/application_controller.rb`, `app/controllers/messages_controller.rb`, `app/controllers/skills_controller.rb`
  - AuthN/AuthZ: `app/controllers/concerns/current_user.rb`, `app/helpers/permissions_helper.rb`, `app/lib/user/permissions.rb`
  - Domain: `app/models/*.rb`
  - Forms/services: `app/lib/skill_form.rb`, `app/lib/workspace_form.rb`, `app/lib/community_request_form.rb`
  - Jobs: `app/jobs/*.rb`
  - Mailers: `app/mailers/*.rb`
  - File storage: `app/models/concerns/aws_file_storage.rb`, `app/models/concerns/public_file_storage.rb`
  - Views/assets: `app/views/**/*`, `app/assets/javascripts/**/*`, `app/assets/stylesheets/**/*`
