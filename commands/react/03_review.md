# /react:review

Review React code against Feature Library pattern and team standards.

**Agents:** react-figma-ui-engineer, tak-typescript-reviewer, code-simplicity-reviewer, design-implementation-reviewer (if UI)

---

## Workflow

1. Setup (2min): Identify scope, check if UI review needed
2. **Parallel** (10min): Feature Library | Export/Import | TypeScript | TanStack Query | UI/Component | State
3. **Analysis** (5min): Complexity review
4. Final (3min): Figma comparison (if UI), consolidate report

---

## Critical Checks (Priority Order)

1. **EXPORTS/IMPORTS** - ANY default exports/imports? (auto-fail)
2. **FEATURE LIBRARY** - Structure correct? Barrel exports? Clean imports?
3. **DATA FLOW** - Component → Hook → TanStack Query → API?
4. **TANSTACK QUERY** - Proper hooks? Mutations invalidate?
5. **COMPONENT** - Shadcn UI? cn()? Props typed?

---

## Decision Tree

```
ANY default exports? → 🔴 Convert ALL to named
ANY default imports? → 🔴 Convert ALL to named
Barrel exports missing? → 🔴 Add index.ts to all folders
Component → API directly? → 🔴 Use TanStack Query hooks
Relative imports across libs? → 🔴 Use @{projectName}/*
useState for shared state? → 🔴 Use Zustand
Inline styles? → 🟡 Use Tailwind + cn()
✅ All checks pass
```

---

## Verification

```bash
# Should find NONE
grep -r "export default" libs/{feature}/
grep -r "import .* from" libs/{feature}/ | grep -v "import {"
```

---

## Output

```markdown
### 🔍 Review: [Feature]

**Score**: X/10

**Export/Import** ✅ Named only | Barrel exports | Ordered
**Feature Library** ✅ Structure | Data flow | Clean imports
**TanStack Query** ✅ Hooks | Mutations | Keys
**Component** ✅ Shadcn UI | cn() | Typed props

**Critical** 🔴 | **Important** 🟡 | **Nice-to-have** 🟢
```
