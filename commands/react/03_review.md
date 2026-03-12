# /react:03_review

Review React code and generate actionable fix checklist for next phase.

**Agents:** tak-typescript-reviewer, code-simplicity-reviewer, design-implementation-reviewer (if UI)

---

## Pre-Review

**From /common:plan and /react:02_implement outputs:**
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
2. **Parallel (10min)**: Feature Library | Exports/Imports | TypeScript | State
3. **Analysis (5min)**: Complexity review, over-engineering check
4. **Final (3min)**: Generate Fix Checklist, consolidate score

---

## Checks (Priority Order)

### 🔴 Critical (Auto-Fail)
1. `export default` found? → Must convert to named
2. Missing barrel exports (index.ts)?
3. Component → API direct call (bypassing hooks)?
4. `any` without justification?

### 🟡 Important
5. `useState` for shared state? (should use Zustand)
6. Props > 10? (props explosion)
7. Component > 200 lines?

### 🟢 Nice-to-have
9. Could be simpler?
10. Missing cn() for Tailwind?
11. Naming could be clearer?

---

## Decision Tree

```
Default exports/imports? → 🔴 Convert to named
Barrel exports missing? → 🔴 Add index.ts
Component → API direct? → 🔴 Use TanStack Query hooks
any without comment? → 🔴 Add types or justification
Relative imports across libs? → 🟡 Use @{projectName}/*
useState for shared state? → 🟡 Use Zustand
Component > 200 lines? → 🟡 Consider splitting
Props > 10? → 🟡 Props explosion - split component
Inline styles? → 🟢 Use Tailwind + cn()
✅ All pass → Score 10/10
```

---

## Verification Commands

```bash
# {pm} = npm, yarn, pnpm, bun (use project's package manager)
grep -r "export default" libs/{feature}/
grep -r ": any" libs/{feature}/ --include="*.ts" --include="*.tsx"
{pm} run lint    # if available
```

---

## Output (→ run /react:04_fix)

```markdown
### 🔍 Review: [Feature] - Score: X/10

**Summary:**
- 🔴 Critical: X issues (must fix)
- 🟡 Important: X issues (should fix)
- 🟢 Nice-to-have: X issues (optional)

---

## 🔴 Critical Issues

### C1: [Issue Title]
- **File:** `libs/feature/Component.tsx:42`
- **Problem:** `export default Component`
- **Fix:**
```typescript
// Before
export default Component

// After
export const Component = () => { }
```

### C2: [Issue Title]
- **File:** `libs/feature/hooks/index.ts:15`
- **Problem:** Missing barrel export
- **Fix:** Add to `libs/feature/index.ts`

---

## 🟡 Important Issues

### I1: [Issue Title]
- **File:** `libs/feature/Component.tsx`
- **Problem:** [Description]
- **Suggestion:** [How to improve]

---

## 🟢 Nice-to-have

### N1: [Issue Title]
- **Suggestion:** [Optional improvement]

---

## Fix Checklist (→ run /react:04_fix)

**🔴 Critical (Must Fix):**
- [ ] C1: Convert default export → named (Component.tsx:42)
- [ ] C2: Add barrel export (index.ts)

**🟡 Important (Should Fix):**
- [ ] I1: [Description] (File.tsx:line)

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

---

## --team Mode (Agent Teams)

Use `--team` flag for comprehensive multi-perspective review.

**Cost:** 3x tokens vs standard mode. Use for critical features only.

### Team Composition

| Role | Focus | Agent |
|------|-------|-------|
| Code Quality | TypeScript, patterns, exports | tak-typescript-reviewer |
| Simplicity | Over-engineering, complexity | code-simplicity-reviewer |
| UI/Design | Figma match, visual QA | design-implementation-reviewer |

### Coordination

1. **Parallel Review**: Each agent reviews independently
2. **Issue Sharing**: Agents share findings via messages
3. **Deduplication**: Remove duplicate issues across agents
4. **Priority Discussion**: Agents discuss issue severity
5. **Unified Output**: Main agent consolidates Fix Checklist

### Workflow

```
1. Main agent identifies files to review
2. Spawn 3 parallel agents (Quality, Simplicity, UI)
3. Each agent:
   - Reviews from their perspective
   - Creates issues with severity
   - Shares findings with team
4. Main agent:
   - Collects all issues
   - Removes duplicates
   - Resolves severity conflicts
   - Generates unified Fix Checklist
```

### When to Use

✅ **Use --team:**
- Critical feature launch
- Complex UI with Figma designs
- Code requiring deep quality review

❌ **Skip --team:**
- Small changes
- Non-UI features (skip design reviewer)
- Quick iteration cycles
