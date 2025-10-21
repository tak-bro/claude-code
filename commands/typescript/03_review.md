# /typescript:review

Multi-perspective code review against team standards.

**Agents:** tak-typescript-reviewer, code-simplicity-reviewer, design-implementation-reviewer (if UI)

---

## Workflow

1. Setup (2min): Determine review types
2. **Parallel** (10min): tak-typescript-reviewer | design (if UI) | Security+Performance
3. Analysis (5min): code-simplicity-reviewer
4. Report (3min): Consolidate, prioritize

---

## Checks (Priority Order)

1. **EXPORTS/IMPORTS** - `export default`? (auto-fail)
2. **TYPE SAFETY** - `any` without justification? Unhandled null?
3. **MODERN PATTERNS** - `function` keyword? Deep nesting?
4. **TESTABILITY** - Hard to test = poor structure

---

## Output

```markdown
### 🔍 [Feature] - X/10

🔴 **Critical**
- [Issue]: [Fix] (File:line)

🟡 **Important**
- [Issue]: [Suggestion]

🟢 **Nice-to-have**
- [Enhancement]
```

**Scores:** 10=Perfect | 8-9=Minor | 6-7=Important | 4-5=Critical | 0-3=Major
