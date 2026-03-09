# /typescript:03_review

Review TypeScript code and generate actionable fix checklist for next phase.

**Agents:** tak-typescript-reviewer, code-simplicity-reviewer

---

## Pre-Review

**From /common:plan and /typescript:02_implement outputs:**
1. Review Focus items from plan
2. Implementation notes from implement
3. Files created/modified list

---

## Boundaries

### ✅ Always Do
- Check ALL Critical items (auto-fail if missed)
- Provide specific file:line references
- Include fix code snippets
- Generate Fix Checklist for next phase

### ⚠️ Ask First
- Suggest major refactoring
- Recommend architecture changes

### 🚫 Never Do
- Skip Critical checks
- Give vague feedback without fix examples
- Approve code with Critical issues

---

## Workflow

1. **Setup (2min)**: Load plan's Review Focus, implement's Notes
2. **Parallel (10min)**: Exports/Imports | Type Safety | Patterns | Testability
3. **Analysis (5min)**: Complexity review, simplification opportunities
4. **Final (3min)**: Generate Fix Checklist, consolidate score

---

## Checks (Priority Order)

### 🔴 Critical (Auto-Fail)
1. `export default` found? → Must convert to named
2. `any` without justification comment?
3. `function` keyword used? (should be arrow)
4. Unhandled null/undefined?
5. Missing barrel exports (index.ts)?

### 🟡 Important
6. Deep nesting? (should use early returns)
7. Complex conditions not named?
8. Hard to test structure?

### 🟢 Nice-to-have
10. Could be simpler?
11. Naming could be clearer (5-second rule)?
12. Missing type inference opportunities?

---

## Decision Tree

```
export default? → 🔴 Convert to named export
any without comment? → 🔴 Add types or justification
function keyword? → 🔴 Convert to arrow function
Unhandled null/undefined? → 🔴 Add guards/early returns
Missing barrel exports? → 🔴 Add index.ts
Deep nesting? → 🟡 Use early returns
Complex unnamed condition? → 🟡 Extract to named variable
Hard to test? → 🟡 Restructure for testability
✅ All pass → Score 10/10
```

---

## Verification Commands

```bash
# {pm} = npm, yarn, pnpm, bun (use project's package manager)
grep -r "export default" src/
grep -r ": any" src/ --include="*.ts"
grep -r "^function " src/ --include="*.ts"
{pm} run lint    # if available
```

---

## Output (→ /typescript:04_fix 입력)

```markdown
### 🔍 Review: [Feature] - Score: X/10

**Summary:**
- 🔴 Critical: X issues (must fix)
- 🟡 Important: X issues (should fix)
- 🟢 Nice-to-have: X issues (optional)

---

## 🔴 Critical Issues

### C1: Default Export
- **File:** `src/utils/validation.ts:45`
- **Problem:** `export default validateEmail`
- **Fix:**
```typescript
// Before
export default validateEmail

// After
export const validateEmail = (email: string): boolean => { }
```

### C2: Untyped Any
- **File:** `src/services/api.ts:23`
- **Problem:** `const data: any = await fetch()`
- **Fix:**
```typescript
// Before
const data: any = await fetchData();

// After
const data: UserResponse = await fetchData<UserResponse>();
```

### C3: Function Keyword
- **File:** `src/utils/helpers.ts:12`
- **Problem:** `function calculateTotal()`
- **Fix:**
```typescript
// Before
function calculateTotal(items: Item[]): number { }

// After
const calculateTotal = (items: Item[]): number => { }
```

---

## 🟡 Important Issues

### I1: Deep Nesting
- **File:** `src/services/user.ts:34`
- **Problem:** 4 levels of nesting
- **Fix:** Use early returns

### I2: Unnamed Complex Condition
- **File:** `src/components/Form.tsx:56`
- **Problem:** Long inline condition
- **Fix:** Extract to named variable

---

## 🟢 Nice-to-have

### N1: [Issue Title]
- **Suggestion:** [Optional improvement]

---

## Fix Checklist (→ /typescript:04_fix 입력)

**🔴 Critical (Must Fix):**
- [ ] C1: Convert default export → named (validation.ts:45)
- [ ] C2: Add proper types (api.ts:23)
- [ ] C3: Convert function → arrow (helpers.ts:12)

**🟡 Important (Should Fix):**
- [ ] I1: Flatten nesting with early returns (user.ts:34)
- [ ] I2: Extract condition to named variable (Form.tsx:56)

**🟢 Nice-to-have (Optional):**
- [ ] N1: [Description]

---

**Verification After Fix:**
```bash
{pm} run lint    # if available
{pm} test        # if available
```
```

---

## Scoring

| Score | Meaning |
|-------|---------|
| 10 | Perfect - no issues |
| 8-9 | Minor issues only (🟢) |
| 6-7 | Important issues (🟡) |
| 4-5 | Critical issues (🔴) |
| 0-3 | Major problems, needs rework |
