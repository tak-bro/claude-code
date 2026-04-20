---
name: kotlin-04-fix
description: "Android/Kotlin review issue fixes. Triggers on 'kotlin fix', '코틀린 수정', 'android fix', '안드로이드 수정', 'android 이슈 해결', 'kt fix'."
tools: Read, Bash, Glob, Grep, Write, Edit
model: sonnet

# Kotlin Fix

Fix issues found in review, prioritized by severity.

**Agents:** kotlin-android-expert

**Fix pattern references:**
- `references/common-fixes.md` -- Before/After code for all common Android fixes
- `../kotlin-02-implement/references/mvvm-compose-pattern.md`

---

## Pre-Fix

**From review output, load:**
1. Fix Checklist (Critical -> Important -> Nice-to-have)
2. Specific file:line references
3. Fix code snippets provided

---

## Boundaries

### Always Do
- Fix ALL [Critical] issues (no exceptions)
- Run verification after each fix
- Mark completed items in checklist
- Keep fixes minimal and focused
- Regression testing required for all bug fixes (Claude Philosophy)

### [Warning] Ask First
- Skip [Important] issues
- Refactor beyond fix scope
- Add new features while fixing

### Never Do
- Skip [Critical] issues
- Change logic beyond the fix
- Introduce new patterns
- Leave verification failing

---

## Fix Priority

```
[Critical] (MUST fix - no exceptions)
    |
    |-- MutableStateFlow exposed -> private _state + public state
    |-- Business logic in Composable -> move to ViewModel
    |-- Direct API/DB calls -> use Repository
    |-- Missing Hilt -> add @Inject/@Module
    |-- Blocking Main -> withContext(Dispatchers.IO)
    |-- Mutable UiState -> immutable data class
    |-- var for state -> val + StateFlow
    |-- Activity/Context ref in ViewModel -> never hold, use Application context
    |
    v
[Important] (SHOULD fix)
    |
    |-- Missing UiState/UiEvent pattern
    |-- Screen/Component separation
    |-- No Repository interface
    |-- GlobalScope -> viewModelScope
    |-- Missing error handling
    |-- Hardcoded strings -> string resources
    |
    v
[Nice-to-have] (OPTIONAL)
```

---

## Doc Sync (Claude Philosophy)
After fixing, verify:
- README drift check
- CLAUDE.md needs update?
- CHANGELOG needs update?

## Self-Verification (Stop hook integration)
Before completion, verify:
- [ ] Tests pass?
- [ ] Debug code removed?
- [ ] Conventions followed?

---

## Checklist

- [ ] ALL [Critical] issues fixed
- [ ] [Important] issues addressed (or justified skip)
- [ ] `./gradlew lint` passes (if available)
- [ ] `./gradlew test` passes (if available)
- [ ] `./gradlew assembleDebug` passes (if available)
- [ ] Fix report generated
- [ ] Re-review decision made

## Verification Commands

```bash
./gradlew lint{Variant}
./gradlew test{Variant}UnitTest
./gradlew assembleDebug
```

## Re-Review Criteria

| Condition | Action |
|-----------|--------|
| Had Critical issues | -> Run `/kotlin-03-review` again (required) |
| Had only Important issues | -> Self-check and complete |
| Final score 8/10 or higher | -> Complete |
| Final score below 8/10 | -> Run `/kotlin-03-review` again |

---

## Output
(see `_shared/output-templates.md` -- Fix Output)

-> Next: `/kotlin-03-review` (re-review) or `/qa` or `/ship`
