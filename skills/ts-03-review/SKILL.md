---
name: ts-03-review
description: "TypeScript 코드 리뷰. Triggers on 'typescript review', 'ts review', '타입스크립트 리뷰', 'ts 검토'."
context: fork
tools: Read, Bash, Glob, Grep
---

# TypeScript Review

Review TypeScript code and generate actionable fix checklist for next phase.

## Scope
Review ONLY changed files. Run `git diff --name-only` first to identify targets.
Do NOT scan or review the entire codebase.
**Agents:** tak-typescript-reviewer, code-simplicity-reviewer

**Context: fork** — Review in separate context (Test-Time Compute: separated context = better results).

---

## Pre-Review

**From /plan and /ts-02-implement outputs:**
1. Review Focus items from plan
2. Implementation notes from implement
3. Files created/modified list

---

## Boundaries

### Always Do
- Check ALL [Critical] items (auto-fail if missed)
- Provide specific file:line references
- Include fix code snippets
- Generate Fix Checklist for next phase

### [Warning] Ask First
- Suggest major refactoring
- Recommend architecture changes

### Never Do
- Skip [Critical] checks
- Give vague feedback without fix examples
- Approve code with [Critical] issues

---

## Checks (Priority Order)

### [Critical] (Auto-Fail)
1. `export default` found? (except App/config entrypoint) -> Must convert to named
2. `any` without justification comment?
3. `function` keyword used? (should be arrow)
4. Unhandled null/undefined?
5. Missing barrel exports (index.ts)?

### [Important]
6. Deep nesting? (should use early returns)
7. Complex conditions not named?
8. Hard to test structure?

### [Nice-to-have]
9. Could be simpler?
10. Naming could be clearer (5-second rule)?
11. Missing type inference opportunities?

---

## Decision Tree

```
export default? -> [Critical] Convert to named export
any without comment? -> [Critical] Add types or justification
function keyword? -> [Critical] Convert to arrow function
Unhandled null/undefined? -> [Critical] Add guards/early returns
Missing barrel exports? -> [Critical] Add index.ts
Deep nesting? -> [Important] Use early returns
Complex unnamed condition? -> [Important] Extract to named variable
Hard to test? -> [Important] Restructure for testability
All pass -> Score 10/10
```

---

## Verification Commands
(see `_shared/verification-commands.md`)

```bash
# TypeScript-specific checks
grep -r "export default" src/
grep -r ": any" src/ --include="*.ts"
grep -r "^function " src/ --include="*.ts"
{pm} run lint    # if available
```

---

## Scoring
(see `_shared/scoring-table.md`)

---

## --team Mode (Agent Teams)
(see `_shared/team-mode.md`)

| Role | Focus | Agent |
|------|-------|-------|
| Code Quality | TypeScript, patterns, exports | tak-typescript-reviewer |
| Simplicity | Over-engineering, complexity | code-simplicity-reviewer |

### Coordination

1. **Parallel Review**: Each agent reviews independently
2. **Issue Sharing**: Agents share findings via messages
3. **Deduplication**: Remove duplicate issues across agents
4. **Priority Discussion**: Agents discuss issue severity
5. **Unified Output**: Main agent consolidates Fix Checklist

### When to Use

**Use --team:**
- Critical utility/service code
- Complex algorithms
- Code requiring deep quality review

**Skip --team:**
- Small changes
- Simple utilities
- Quick iteration cycles

---

## Output
(see `_shared/output-templates.md` -- Review Output)

-> If issues: `/ts-04-fix`, if no issues: `/qa` or `/ship`
