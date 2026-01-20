# /react:fix

Fix issues identified in review phase, prioritized by severity.

**Agents:** tak-typescript-expert, code-simplicity-reviewer

---

## Pre-Fix

**From /review output, load:**
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
    ├─ import order
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
{pm} run typecheck
{pm} run lint

# After all fixes
{pm} test -- --watchAll=false
{pm} run build
```

---

## Output

```markdown
### ✅ Fixed: [Feature]

**Fixes Applied:**
- [x] 🔴 C1: Converted default export → named (Component.tsx:42)
- [x] 🔴 C2: Added barrel export (index.ts)
- [x] 🟡 I1: Fixed import order (Component.tsx)

**Verification:**
- [x] `{pm} run typecheck` ✓
- [x] `{pm} run lint` ✓
- [x] `{pm} test` ✓
- [x] `{pm} run build` ✓

**Skipped (with reason):**
- [ ] 🟢 N1: [Reason for skip - e.g., "minimal impact, time constraint"]

**Final Score:** X/10 → 10/10
```

---

## Checklist

- [ ] ALL 🔴 Critical issues fixed
- [ ] 🟡 Important issues addressed (or justified skip)
- [ ] `{pm} run typecheck` passes
- [ ] `{pm} run lint` passes
- [ ] `{pm} test` passes
- [ ] `{pm} run build` passes
- [ ] Fix report generated
