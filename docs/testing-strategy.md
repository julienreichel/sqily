# Testing Strategy

## Goals
- Preserve behavior of core learning, collaboration, and moderation flows.
- Catch regressions early in controller/domain interactions.
- Maintain fast developer feedback and high confidence before release.

## Current test stack
- Framework: Minitest (`rails test`)
- Coverage: SimpleCov with minimum threshold `93.63%` (configured in `test/test_helper.rb`)
- Fixtures: global fixtures enabled (`fixtures :all`)
- Mocks/stubs: `mocha/minitest`

## Current suite composition
Based on test file inventory:
- Total test files: **117**
- Controllers: **38**
- Models: **57**
- Jobs: **9**
- Mailers: **3**
- Helpers: **4**
- Lib: **6**
- Integration: **0**

## Pyramid interpretation (current)
- Base (unit/model/lib): strong
- Middle (controller/request-ish): strong
- Top (integration/end-to-end): weak/non-existent in `test/integration`

## Critical path coverage map

### Covered well
- Skills/subscriptions/prerequisites/tasks:
  - `test/controllers/skills_controller_test.rb`
  - `test/controllers/prerequisites_controller_test.rb`
  - `test/controllers/tasks_controller_test.rb`
  - `test/models/skill_test.rb`, `test/models/subscription_test.rb`
- Messaging and collaboration:
  - `test/controllers/messages_controller_test.rb`
  - `test/controllers/discussions_controller_test.rb`
  - `test/models/message_test.rb`, `test/models/vote_test.rb`
- Evaluation/exam/homework:
  - `test/controllers/evaluations_controller_test.rb`
  - `test/controllers/evaluations/exams_controller_test.rb`
  - `test/controllers/evaluations/notes_controller_test.rb`
  - `test/controllers/homeworks_controller_test.rb`
  - `test/models/evaluation_test.rb`, `test/models/evaluation/exam_test.rb`, `test/models/homework_test.rb`
- Events/polls:
  - `test/controllers/events_controller_test.rb`
  - `test/controllers/events/participations_controller_test.rb`
  - `test/controllers/polls_controller_test.rb`
  - `test/controllers/poll_answers_controller_test.rb`
- Workspaces and partnerships:
  - `test/controllers/workspaces_controller_test.rb`
  - `test/controllers/workspaces/partnerships_controller_test.rb`
  - `test/models/workspace_test.rb`, `test/models/workspace/*_test.rb`
- Scheduled/async concerns:
  - `test/jobs/*`

### Coverage gaps / risks
1. No true integration/system tests (`test/integration` currently empty).
2. JavaScript behavior (polling, dynamic forms, toggles) is mostly untested at browser level.
3. End-to-end security and permission matrix scenarios are only partially exercised across cross-controller flows.
4. Runtime configuration behavior (SMTP/object storage environment permutations) is not deeply tested.

## Fixture strategy (current and recommended)

### Current
- Global fixtures are loaded for all tests (`fixtures :all`).
- This keeps setup easy but can couple tests to broad shared data.

### Recommended evolution
- Keep fixtures for core static reference data.
- Prefer narrower fixture subsets or factories for high-variance scenarios.
- Add helper builders for complex workflow setup (exam + draft + homework + notifications).

## Execution commands

### Fast local loop
```bash
rails test
```

### Project CI equivalent
```bash
bin/ci
```
This currently executes:
- `rails test`
- `bundle exec standardrb`
- `bin/brakeman`

### Targeted runs
```bash
rails test test/models/subscription_test.rb
rails test test/controllers/messages_controller_test.rb
rails test test/jobs/daily_summary_job_test.rb
```

## Recommended additions for delivery speed
1. Add integration tests for 5 highest-value user paths:
- Skill subscribe -> task completion -> evaluation start
- Message post/edit/pin/vote/search
- Homework submit -> review -> completion
- Event register/waitlist/promotion
- Workspace publish/approve/reject
2. Add permission matrix tests (role/action tables) for moderator/admin boundaries.
3. Add contract tests for JSON polling payloads (`messages#index`).
4. Add failure-mode tests for callback side effects (mail/notification/message triggering).

## Quality gates proposal
- Required on PR:
  - `rails test`
  - lint (`standardrb`)
  - security static check (`brakeman`)
- Recommended:
  - changed-files targeted integration scenarios
  - coverage threshold kept at or above current minimum
