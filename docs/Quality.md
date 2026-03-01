# Code Quality Review

## Overall grade
**B-**

The codebase shows solid domain coverage, explicit business concepts, and broad automated tests, but maintainability is constrained by large classes/controllers, heavy callback usage, and dense orchestration logic.

## Detected code smells (with counts)

Method used for counts: static scan of `app/**/*.rb` with size/structure heuristics.

| Code smell (standard name) | Count | How counted |
|---|---:|---|
| **Large Class** | 16 | Ruby files in `app/` with `>= 100` LOC |
| **Long Method** | 43 | Methods longer than `15` lines (tokenized `def..end` scan) |
| **Long Method (severe)** | 17 | Methods longer than `25` lines |
| **Large/Fat Controller** | 6 | Controllers with `>= 80` LOC |
| **Message Chains (Law of Demeter violation)** | 119 | Lines containing call chains with 3+ chained dots (`a.b.c...`) |
| **Callback Hell** | 40 | `before_*`, `after_*`, `around_*` callbacks in models |
| **Duplicate Code (exact declaration duplicate)** | 1 | Duplicate `belongs_to :from_user...` declaration in `app/models/message.rb` |

## Main issues observed

1. **Controller/service orchestration is too centralized in a few hotspots**
- `MessagesController`, `SkillsController`, `ApplicationController`, `WorkspacesController` carry multiple responsibilities (authorization, branching, domain orchestration, response shape).
- This increases change risk and makes behavioral testing harder.

2. **Domain behavior is spread across callbacks and side effects**
- Model callbacks trigger messaging, notifications, emails, and state propagation.
- This obscures execution order and makes failures/non-happy paths difficult to reason about.

3. **Business rules are often embedded in condition-heavy methods**
- Permission checks and workflow conditions are correct but scattered, producing long methods and repeated branching patterns.
- Extracting explicit policy/command objects would improve readability and test focus.

4. **High use of message chains and query logic inline**
- Deep navigation (`a.b.c`) and SQL fragments in models/controllers create tight coupling and reduce local clarity.
- Query objects / repositories would help isolate persistence concerns.

5. **Consistency debt exists in small quality details**
- At least one concrete duplication bug signal (duplicate association declaration in `Message`), plus style inconsistency across similar flows.
- These are low-cost fixes that improve trust and reduce accidental defects.

## Coaching recommendation (short)

Prioritize a refactor track around three seams:
1. Extract **application services/commands** for high-branch flows (`messages`, `skills`, `evaluations`).
2. Replace implicit callback workflows with explicit domain operations where side effects are orchestrated in one place.
3. Introduce **policy/query objects** to reduce controller branching and inline SQL coupling.

This would realistically move the codebase toward **B+/A- maintainability** without a full rewrite.
