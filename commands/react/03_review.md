# /react:review

Review React code against Feature Library pattern and team standards.

**Agents:** react-figma-ui-engineer, tak-typescript-reviewer, code-simplicity-reviewer, design-implementation-reviewer (if UI)

---

## Workflow

1. Setup (2min): Identify scope, check UI review
2. **Parallel** (10min): Feature Library | Exports/Imports | TypeScript | TanStack Query | State
3. Analysis (5min): Complexity review
4. Final (3min): Figma (if UI), consolidate

---

## Checks (Priority Order)

1. **EXPORTS/IMPORTS** - Default exports/imports? (auto-fail)
2. **FEATURE LIBRARY** - Structure? Barrel exports? Clean imports?
3. **DATA FLOW** - Component → Hook → TanStack Query → API?
4. **TANSTACK QUERY** - Hooks? Mutations invalidate?
5. **COMPONENT** - Shadcn UI? cn()? Props typed?

---

## Decision Tree

```
Default exports/imports? → 🔴 Convert to named
Barrel exports missing? → 🔴 Add index.ts
Component → API direct? → 🔴 Use TanStack Query hooks
Relative imports across libs? → 🔴 Use @{projectName}/*
useState for shared state? → 🔴 Use Zustand
Inline styles? → 🟡 Use Tailwind + cn()
✅ All pass
```

---

## Verification

```bash
grep -r "export default" libs/{feature}/
grep -r "import .* from" libs/{feature}/ | grep -v "import {"
```

---

## Output

```markdown
### 🔍 [Feature] - X/10

**Exports/Imports:** ✅ Named | Barrel | Ordered
**Feature Library:** ✅ Structure | Flow | Imports
**TanStack Query:** ✅ Hooks | Mutations | Keys
**Component:** ✅ Shadcn | cn() | Types

🔴 Critical | 🟡 Important | 🟢 Nice-to-have
```
