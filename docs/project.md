# Project Description

## Purpose
Sqily is a school-oriented platform for competency-based learning and peer validation (Validation mutuelle des competences, VMC). It combines:
- Skill pathways organized as trees
- Collaborative communication (community, skill, workspace, direct)
- Validation workflows (drafts, exams, notes, homework evidence)
- Community governance (invitations, moderation, teams, events)

The product supports the idea that learning depth increases when learners prepare to teach or validate peers.

## Target Users
- Learner: follows skills, collaborates, submits evidence, participates in events.
- Moderator/Teacher: curates skill architecture, validates work, moderates memberships and participation.
- Admin (platform): manages platform-wide community requests, community administration, and static pages.

## Core Value
- Make competencies visible at learner and community level.
- Structure autonomous learning with explicit prerequisites and tasks.
- Operationalize peer/formative validation through exam and feedback loops.
- Keep the learning community active through messaging, events, and recognition.

## Scope Boundaries
In scope:
- Communities, memberships, teams
- Skill tree management and progression
- Messaging, polls, events
- Evaluation lifecycle (draft -> exam -> notes -> completion)
- Homework evidence and review
- Workspaces and partnerships
- Notifications, badges, statistics

Out of scope (current codebase):
- Public API for third-party integration
- Dedicated microservices/event bus architecture
- Real-time WebSocket messaging (current polling-based refresh)
- Multi-tenant hard isolation beyond community-level domain model

## Key Terminology
- Community: collaboration space grouping members, skills, and activity.
- Membership: user affiliation to a community, includes moderator flag.
- Skill: competency node in a tree (root/child) with prerequisites and tasks.
- Subscription: learner enrollment to a skill; completion state marks expertise.
- Evaluation: challenge definition for a skill.
- Exam: evaluation execution instance between candidate and examiner.
- Evaluation note: threaded review message in an exam, may accept/reject.
- Homework: uploaded evidence artifact associated with evaluation/subscription.
- Workspace: portfolio/content artifact with versioning and publication states.
- Partnership: workspace access relationship (owner/writer/reader).
- Message: communication artifact (text, upload, event, poll, system messages).
- Notification: in-app event signal (mention, homework state, poll finished, etc.).

## Product Surface (high level)
- Entry points: public community listing, login/session, per-community skill map.
- Main user loop: discover skill -> subscribe -> collaborate -> submit validation -> receive feedback -> complete.
- Main moderator loop: design skill structure -> govern members/teams -> orchestrate evaluations/events.

## References
- Existing architecture documentation: `docs/c4-model.md`
- Existing flow inventory: `docs/product-flows.md`
- Existing detailed user/moderator flows: `docs/flows/*-flow.md`
