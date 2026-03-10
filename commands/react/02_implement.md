# /react:02_implement

Implement React features using Feature Library pattern (Nx monorepo).

**Agents:** react-feature-library-expert, react-figma-ui-engineer, framework-docs-researcher

**For React Feature Library patterns and examples, follow `react-feature-library-expert` agent guidelines.**

---

## Pre-Implementation

**Before coding, verify from /common:plan output:**
1. ✅ Plan approved?
2. ✅ Files to create/modify identified?
3. ✅ Implementation Checklist ready?

---

## Boundaries

### ✅ Always Do
- Use plan's Implementation Checklist
- Named exports only (`export const`)
- Barrel exports (index.ts) for every folder
- Component → Hook → TanStack Query → API flow
- Run verification commands before done
- Zustand for shared state, useState for local only

### ⚠️ Ask First
- Adding new dependencies
- Modifying shared utilities
- Creating patterns not in codebase

### 🚫 Never Do
- `export default`
- `useState` for shared state
- Direct API calls in components
- Skip barrel exports
- `any` without justification

---

## Workflow

1. **Setup (3min)**: Create lib, identify feature, plan data flow
2. **Parallel (20min)**: APIs | Hooks | Components | Types/Consts
3. **Integration (5min)**: Barrel exports, test flow, verify aliases
4. **Validation (2min)**: Run verification commands

---

## Structure

```bash
# Create feature library
npx nx g @nx/js:lib {feature} --directory=libs/{feature}
```

```
libs/{feature}/
├── apis/index.ts      # API functions
├── hooks/index.ts     # TanStack Query hooks
├── types/index.ts     # Types
├── consts/index.ts    # Constants
└── index.ts           # Barrel (REQUIRED)
```

**Flow:** Component → Hook → TanStack Query → API

---

## Patterns

**API**
```typescript
export const fetchProducts = async (): Promise<Product[]> => {
  const { data } = await webCore.buildSignedRequest({ method: 'GET', baseURL: '/products' }).execute();
  return data;
};
```

**Hook**
```typescript
export const productsKeys = createQueryKeys('products');
export const useProducts = () => useQuery({ queryKey: productsKeys.all, queryFn: fetchProducts });
```

**Component**
```typescript
export const ProductList = () => {
  const { data, isLoading } = useProducts();
  if (isLoading) return <div>Loading...</div>;
  return <div className={cn('flex gap-4')}>{data?.map(...)}</div>;
};
```

---

## Anti-Over-Engineering (WET > DRY)

- **2 simple > 1 complex**: Prefer two simple components over mega-component
- **Intentional duplication OK**: If it preserves clarity
- **~200 lines guideline**: Consider splitting if exceeded
- **~5 business props healthy**: 10+ is a smell

---

## Verification Commands

```bash
# {pm} = npm, yarn, pnpm, bun (use project's package manager)
{pm} run lint    # if available
{pm} test        # if available
```

---

## Checklist

- [ ] **Create lib:** `npx nx g @nx/js:lib {feature} --directory=libs/{feature}`
- [ ] Named exports ONLY
- [ ] Barrel exports ALL folders
- [ ] @{projectName}/* aliases
- [ ] cn() for Tailwind
- [ ] Component → Hook → TanStack Query → API
- [ ] Zustand (global) | TanStack Query (server) | useState (local)
- [ ] ~200 lines guideline, ~5 business props
- [ ] Verification commands pass

---

## Output (→ run /react:03_review)

```markdown
### ✅ Implemented: [Feature]

**Files Created/Modified:**
- ✨ `libs/feature/apis/index.ts`
- ✨ `libs/feature/hooks/index.ts`
- 🔧 `libs/feature/index.ts`

**Checklist Completed:**
- [x] Named exports
- [x] Barrel exports
- [x] Component → Hook → API flow

**Verification:**
- [x] `{pm} run lint` ✓ (if available)
- [x] `{pm} test` ✓ (if available)

**Notes for Review:**
- [Implementation decisions]
- [Areas needing attention]
```
