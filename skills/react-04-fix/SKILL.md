---
name: react-04-fix
description: "React 리뷰 이슈 수정. Triggers on 'react fix', '리액트 수정', 'react 이슈 해결'."
tools: Read, Bash, Glob, Grep, Write, Edit
---

# React Fix

Fix issues discovered in review by severity order.

**Agents:** tak-typescript-expert, code-simplicity-reviewer

**Fix pattern references:**
- `references/common-fixes.md` — Before/After code for all common React fixes
- `../react-02-implement/references/feature-library-pattern.md`

---

## Pre-Fix

**From review output, load:**
1. Fix Checklist (Critical → Important → Nice-to-have)
2. Specific file:line references
3. Fix code snippets provided

---

## Boundaries

### Always Do
- Fix ALL [Critical] issues (no exceptions)
- Run verification after each fix
- Regression testing required for all bug fixes

### 🚫 Never Do
- Skip [Critical] issues
- Change logic beyond the fix

---

## Fix Priority

```
[Critical] (MUST fix)
    ├─ export default → named export (except App/config entrypoint)
    ├─ missing barrel exports → add index.ts
    ├─ direct API calls → use hooks
    └─ any without justification → add types
[Important] (SHOULD fix)
    ├─ useState → Zustand for shared
    ├─ props explosion
    └─ component too large
[Nice-to-have] (OPTIONAL)
```

## Common Fixes

**Default Export → Named Export**
```typescript
// Before
export default Component
// After
export const Component = () => { }
// Update imports everywhere
import { Component } from './Component'
```

**Add Barrel Export**
```typescript
// libs/feature/index.ts
export * from './apis';
export * from './hooks';
export * from './types';
export * from './consts';
```

**Direct API → Hook**
```typescript
// Before (in component)
const [data, setData] = useState();
useEffect(() => { fetchData().then(setData); }, []);
// After
const { data } = useFeatureData();  // TanStack Query hook
```

---

## Checklist

- [ ] ALL [Critical] issues fixed
- [ ] [Important] issues addressed (or justified skip)
- [ ] `{pm} run lint` passes (if available)
- [ ] `{pm} test` passes (if available)
- [ ] `{pm} run build` passes (if available)
- [ ] Fix report generated
- [ ] Re-review decision made

## Verification Commands
(see `_shared/verification-commands.md`)

## Doc Sync & Self-Verification (Claude Philosophy)
- Check README/CLAUDE.md/CHANGELOG drift
- Self-verify before completion: Tests pass? Debug code removed? Conventions followed?

## Re-Review Criteria

| Condition | Action |
|-----------|--------|
| Had Critical issues | → `/react-03-review` again |
| Final score 8/10+ | → Complete |
| Below 8/10 | → `/react-03-review` again |

## Output
(see `_shared/output-templates.md` — Fix Output)

→ Next: `/react-03-review` (re-review) or `/qa` or `/ship`
