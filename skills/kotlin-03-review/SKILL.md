---
name: kotlin-03-review
model: sonnet
description: "Android/Kotlin code review. Triggers on 'kotlin review', '코틀린 리뷰', 'android review', '안드로이드 리뷰', 'android code review', 'kt review'."
context: fork
tools: Read, Bash, Glob, Grep
---

# Kotlin Review

Review Android/Kotlin code and generate Fix Checklist for next phase.

**Agents:** kotlin-android-expert, code-simplicity-reviewer

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
1. MutableStateFlow/MutableState exposed publicly from ViewModel?
2. Business logic in Composable function?
3. Direct API/DB calls from ViewModel (bypassing Repository)?
4. Missing Hilt injection (manual instantiation)?
5. Blocking call on Main thread (no withContext/IO)?
6. Mutable data class used for UiState?
7. var used for observable state?
8. Activity/Context reference held in ViewModel (memory leak)?

### [Important]
9. Missing UiState/UiEvent/UiEffect pattern?
10. Screen vs Component separation violated?
11. Repository not using interface (concrete class in domain)?
12. Coroutine scope misuse (GlobalScope, no cancellation)?
13. Missing error handling in Repository/ViewModel?
14. Hardcoded strings in UI (no string resources)?
15. Missing `remember` / `derivedStateOf` for expensive computation in Compose?
16. Effect Channel using default capacity (should be BUFFERED)?

### [Nice-to-have]
17. Could state be simplified?
18. Naming could be clearer?
19. Unnecessary recomposition?

---

## Decision Tree

```
MutableStateFlow exposed? -> [Critical] Use private _state + public state
Business logic in Composable? -> [Critical] Move to ViewModel
Direct API call from ViewModel? -> [Critical] Use Repository
Missing Hilt? -> [Critical] Add @Inject/@Module
Blocking Main thread? -> [Critical] Use withContext(Dispatchers.IO)
Mutable UiState? -> [Critical] Use immutable data class
var for state? -> [Critical] Use val + StateFlow
Context in ViewModel? -> [Critical] Never hold Activity/Context ref
Missing UiState pattern? -> [Important] Add sealed interface
No Screen/Component separation? -> [Important] Extract composables
No Repository interface? -> [Important] Create interface in domain
GlobalScope usage? -> [Important] Use viewModelScope
No error handling? -> [Important] Add try-catch/Result
Hardcoded strings? -> [Important] Use stringResource()
All pass -> Score 10/10
```

---

## Scoring
(see `_shared/scoring-table.md`)

## Verification Commands

```bash
./gradlew lint{Variant}
./gradlew test{Variant}UnitTest
./gradlew assembleDebug
./gradlew detekt              # if configured
./gradlew ktlintCheck         # if configured
```

---

## --team Mode
(see `_shared/team-mode.md`)

### Team Composition

| Role | Focus | Agent |
|------|-------|-------|
| Android Expert | MVVM, Compose, Hilt patterns | kotlin-android-expert |
| Code Quality | Complexity, naming, YAGNI | code-simplicity-reviewer |

---

## Output
(see `_shared/output-templates.md` -- Review Output)

-> If issues exist: `/kotlin-04-fix`, otherwise: `/qa` or `/ship`
