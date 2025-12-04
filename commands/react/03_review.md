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
6. **OVER-ENGINEERING** - Mega-components? Props explosion (10+)? Forced abstraction?

---

## Decision Tree

```
Default exports/imports? → 🔴 Convert to named
Barrel exports missing? → 🔴 Add index.ts
Component → API direct? → 🔴 Use TanStack Query hooks
Relative imports across libs? → 🔴 Use @{projectName}/*
useState for shared state? → 🔴 Use Zustand
Inline styles? → 🟡 Use Tailwind + cn()

# Anti-Over-Engineering (WET > DRY)
Component > ~200 lines? → 🟡 Consider splitting into self-contained components
Props > 10? → 🟡 Props explosion - consider splitting (~5 business props is healthy)
Multiple conditional renders? → 🟡 Prefer separate simple components
Forced reuse hurting clarity? → 🔴 Allow intentional duplication (natural reuse only)
Mega-component pattern? → 🔴 Garden over Pyramid - split into simple, self-contained components

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
**Simplicity:** ✅ ~200 lines | ~5 business props | Self-contained | Natural reuse

🔴 Critical | 🟡 Important | 🟢 Nice-to-have
```
