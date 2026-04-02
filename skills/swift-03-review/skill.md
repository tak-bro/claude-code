---
name: swift-03-review
description: "iOS/Swift code review. Triggers on 'swift review', '스위프트 리뷰', 'ios review', 'iOS 리뷰', 'ios code review', 'swiftui review'."
context: fork
tools: Read, Bash, Glob, Grep
---

# Swift Review

Review iOS/Swift code and generate Fix Checklist for next phase.

**Agents:** swift-ios-expert, code-simplicity-reviewer

**Context: fork** -- Review in a separate context from implementation (Test-Time Compute: same model but separated context = better results).

**Smart Review Routing:** UI code -> design-reviewer, security-related -> `/cso`

---

## Pre-Review

**From plan and implement outputs:**
1. Review Focus items from plan
2. Implementation notes from implement
3. Files created/modified list

---

## Boundaries

### Always Do
- Check ALL Critical items (auto-fail if missed)
- Provide specific file:line references
- Include fix code snippets
- Generate Fix Checklist for next phase

### [Warning] Ask First
- Suggest major refactoring
- Recommend architecture changes

### Never Do
- Skip Critical checks
- Give vague feedback without fix examples
- Approve code with Critical issues

---

## Checks (Priority Order)

### [Critical] (Auto-Fail)
1. Business logic in SwiftUI View body?
2. Force unwrapping (`!`) without guard/safety?
3. Direct API/DB calls from ViewModel (bypassing Repository)?
4. Missing DI (concrete dependency instead of protocol)?
5. Completion handler in new async code (should be async/await)?
6. Retain cycle (missing [weak self] in escaping closure)?
7. Mutable published state that should be private(set)?
8. Missing `@MainActor` on ViewModel (state mutations from background)?
9. Untracked `Task {}` in ViewModel (never cancelled, concurrent duplicates)?

### [Important]
10. Missing ViewState/Action pattern?
11. View vs Component separation violated?
12. Repository not using protocol (concrete class in domain)?
13. Task not cancelled on view disappear?
14. Missing error handling (unhandled throws)?
15. Hardcoded strings in UI (no localization)?
16. Missing accessibility labels?

### [Nice-to-have]
17. Could state be simplified?
18. Naming could be clearer?
19. View body too complex (should extract subviews)?

---

## Decision Tree

```
Business logic in View? -> [Critical] Move to ViewModel
Force unwrap? -> [Critical] Use guard/if-let/nil-coalescing
Direct API call from ViewModel? -> [Critical] Use Repository
Missing DI protocol? -> [Critical] Create protocol, inject
Completion handler? -> [Critical] Convert to async/await
Retain cycle? -> [Critical] Add [weak self]
Public mutable state? -> [Critical] Use private(set)
Missing @MainActor? -> [Critical] Add @MainActor to ViewModel
Untracked Task? -> [Critical] Store task ref, cancel on deinit/new request
Missing ViewState? -> [Important] Add state enum/struct
No View/Component separation? -> [Important] Extract components
No Repository protocol? -> [Important] Create protocol
Task not cancelled? -> [Important] Add .task or onDisappear
No error handling? -> [Important] Add do-catch/Result
Hardcoded strings? -> [Important] Use localization
Missing accessibility? -> [Important] Add labels
All pass -> Score 10/10
```

---

## Scoring
(see `_shared/scoring-table.md`)

## Verification Commands

```bash
xcodebuild build -scheme {Scheme} -sdk iphonesimulator
xcodebuild test -scheme {Scheme} -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 16'
swiftlint lint --strict                # if configured
swiftformat --lint .                    # if configured
```

---

## --team Mode
(see `_shared/team-mode.md`)

### Team Composition

| Role | Focus | Agent |
|------|-------|-------|
| iOS Expert | MVVM, SwiftUI, Concurrency patterns | swift-ios-expert |
| Code Quality | Complexity, naming, YAGNI | code-simplicity-reviewer |

---

## Output
(see `_shared/output-templates.md` -- Review Output)

-> If issues exist: `/swift-04-fix`, otherwise: `/qa` or `/ship`
