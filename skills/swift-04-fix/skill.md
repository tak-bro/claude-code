---
name: swift-04-fix
description: "iOS/Swift review issue fixes. Triggers on 'swift fix', '스위프트 수정', 'ios fix', 'iOS 수정', 'ios 이슈 해결'."
tools: Read, Bash, Glob, Grep, Write, Edit
---

# Swift Fix

Fix issues found in review, prioritized by severity.

**Agents:** swift-ios-expert

**Fix pattern references:**
- `references/common-fixes.md` -- Before/After code for all common iOS fixes
- `../swift-02-implement/references/mvvm-swiftui-pattern.md`

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
    |-- Business logic in View -> move to ViewModel
    |-- Force unwrap -> guard/if-let/nil-coalescing
    |-- Direct API calls -> use Repository
    |-- Missing DI protocol -> create protocol
    |-- Completion handler -> async/await
    |-- Retain cycle -> [weak self]
    |-- Public mutable state -> private(set)
    |-- Missing @MainActor -> add @MainActor to ViewModel
    |-- Untracked Task -> store task ref, cancel on deinit/new request
    |
    v
[Important] (SHOULD fix)
    |
    |-- Missing ViewState/Action pattern
    |-- View/Component separation
    |-- No Repository protocol
    |-- Task not cancelled
    |-- Missing error handling
    |-- Hardcoded strings -> localization
    |-- Missing accessibility
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
- [ ] `xcodebuild build` passes (if available)
- [ ] `xcodebuild test` passes (if available)
- [ ] `swiftlint` passes (if configured)
- [ ] Fix report generated
- [ ] Re-review decision made

## Verification Commands

```bash
xcodebuild build -scheme {Scheme} -sdk iphonesimulator
xcodebuild test -scheme {Scheme} -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 16'
swiftlint lint --strict
```

## Re-Review Criteria

| Condition | Action |
|-----------|--------|
| Had Critical issues | -> Run `/swift-03-review` again (required) |
| Had only Important issues | -> Self-check and complete |
| Final score 8/10 or higher | -> Complete |
| Final score below 8/10 | -> Run `/swift-03-review` again |

---

## Output
(see `_shared/output-templates.md` -- Fix Output)

-> Next: `/swift-03-review` (re-review) or `/qa` or `/ship`
