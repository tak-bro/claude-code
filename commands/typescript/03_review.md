# /typescript:review

Multi-perspective code review against team standards.

**Agents:** tak-typescript-reviewer, code-simplicity-reviewer, design-implementation-reviewer (if UI)

---

## Workflow

1. Setup (2min): Determine review types
2. **Parallel Reviews** (10min): tak-typescript-reviewer | design-reviewer (if UI) | Security+Performance | Style
3. **Parallel Analysis** (5min): code-simplicity-reviewer
4. Report (3min): Consolidate, prioritize issues

---

## Critical Checks (Priority Order)

1. **EXPORTS/IMPORTS** - Any `export default`? (auto-fail)
2. **TYPE SAFETY** - `any` without justification? Unhandled null/undefined?
3. **MODERN PATTERNS** - `function` keyword? Deep nesting?
4. **TESTABILITY** - Hard to test = poor structure

---

## Output

```markdown
### 🔍 Review: [Feature]

**Score**: X/10

**Critical** 🔴
- [Issue]: [Fix] (File: [path:line])

**Important** 🟡
- [Issue]: [Suggestion]

**Nice-to-have** 🟢
- [Enhancement]
```

**Scores:** 10=Perfect | 8-9=Minor | 6-7=Important | 4-5=Critical | 0-3=Major
