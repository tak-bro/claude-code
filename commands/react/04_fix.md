# /react:04_fix

Fix issues identified in review phase, prioritized by severity.

**Agents:** tak-typescript-expert, code-simplicity-reviewer

---

## Pre-Fix

**From /react:03_review output, load:**
1. Fix Checklist (Critical → Important → Nice-to-have)
2. Specific file:line references
3. Fix code snippets provided

---

## Boundaries

### ✅ Always Do
- Fix ALL 🔴 Critical issues (no exceptions)
- Run verification after each fix
- Mark completed items in checklist
- Keep fixes minimal and focused

### ⚠️ Ask First
- Skip 🟡 Important issues
- Refactor beyond fix scope
- Add new features while fixing

### 🚫 Never Do
- Skip 🔴 Critical issues
- Change logic beyond the fix
- Introduce new patterns
- Leave verification failing

---

## Workflow

1. **Load (1min)**: Get Fix Checklist from review
2. **Fix Critical (varies)**: Fix ALL 🔴 issues, verify each
3. **Fix Important (varies)**: Fix 🟡 issues if time permits
4. **Verify (2min)**: Run all verification commands
5. **Report (1min)**: Generate completion report

---

## Fix Priority

```
🔴 Critical (MUST fix - no exceptions)
    │
    ├─ export default → named export
    ├─ missing barrel exports → add index.ts
    ├─ direct API calls → use hooks
    └─ any without justification → add types
    │
    ▼
🟡 Important (SHOULD fix)
    │
    ├─ useState → Zustand for shared
    ├─ props explosion
    └─ component too large
    │
    ▼
🟢 Nice-to-have (OPTIONAL)
    │
    └─ simplification, naming, cn()
```

---

## Common Fixes

**Default Export → Named Export**
```typescript
// Before
export default Component

// After
export const Component = () => { }

// Update imports everywhere
import Component from './Component'  // Before
import { Component } from './Component'  // After
```

**Add Barrel Export**
```typescript
// libs/feature/index.ts
export * from './apis';
export * from './hooks';
export * from './types';
export * from './consts';
export * from './Component';
```

**Direct API → Hook**
```typescript
// Before (in component)
const [data, setData] = useState();
useEffect(() => {
  fetchData().then(setData);
}, []);

// After
const { data } = useFeatureData();  // TanStack Query hook
```

**any → Proper Type**
```typescript
// Before
const data: any = await fetchData();

// After
const data: FeatureData[] = await fetchData();
```

---

## Verification Commands

```bash
# {pm} = npm, yarn, pnpm, bun (use project's package manager)
# After each fix
{pm} run lint    # if available

# After all fixes
{pm} test        # if available
{pm} run build   # if available
```

---

## Output

```markdown
### ✅ Fixed: [Feature]

**Fixes Applied:**
- [x] 🔴 C1: Converted default export → named (Component.tsx:42)
- [x] 🔴 C2: Added barrel export (index.ts)
- [x] 🟡 I1: Fixed useState → Zustand (Component.tsx)

**Verification:**
- [x] `{pm} run lint` ✓ (if available)
- [x] `{pm} test` ✓ (if available)
- [x] `{pm} run build` ✓ (if available)

**Skipped (with reason):**
- [ ] 🟢 N1: [Reason for skip - e.g., "minimal impact, time constraint"]

**Final Score:** X/10 → 10/10
```

---

## Re-Review Criteria

After completing fixes, determine if re-review is needed based on these criteria:

| Condition | Action |
|-----------|--------|
| Had Critical issues | → Run `/react:03_review` again (required) |
| Had only Important issues | → Self-check and complete |
| Final score 8/10 or higher | → Complete |
| Final score below 8/10 | → Run `/react:03_review` again |

---

## Checklist

- [ ] ALL 🔴 Critical issues fixed
- [ ] 🟡 Important issues addressed (or justified skip)
- [ ] `{pm} run lint` passes (if available)
- [ ] `{pm} test` passes (if available)
- [ ] `{pm} run build` passes (if available)
- [ ] Fix report generated
- [ ] Re-review decision made
