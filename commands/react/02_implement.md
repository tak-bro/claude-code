# /react:implement

Implement React features using Feature Library pattern (Nx monorepo).

**Agents:** react-figma-ui-engineer, framework-docs-researcher, tak-typescript-expert

---

## Workflow
1. Setup (3min): **Create lib**, identify feature, plan data flow, check Shadcn UI
2. **Parallel** (20min): APIs | Hooks | Components | Types/Consts
3. Integration (5min): Barrel exports, test flow, verify aliases, cn()
4. Validation (2min): Lint, named exports, barrel exports, import order

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

## Anti-Over-Engineering (WET > DRY)

- **2 simple > 1 complex** - Prefer two simple components over one mega-component with conditionals
- **Intentional duplication OK** - Duplication is acceptable if it preserves clarity and allows independent evolution
- **Self-contained** - Each component should be understandable in isolation
- **Props awareness** - ~5 business props is healthy; 10+ is a smell, consider splitting
- **~200 lines guideline** - If component exceeds 200 lines, consider splitting
- **Natural vs Forced reuse** - Reuse when patterns naturally emerge, not to satisfy DRY dogma
- **Question abstraction** - "Is this serving the code or my ego? Clarity outlives cleverness."
- **Garden over Pyramid** - Simple, self-contained components that grow independently

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

## Checklist
- [ ] **Create lib:** `npx nx g @nx/js:lib {feature} --directory=libs/{feature}`
- [ ] Named exports ONLY
- [ ] Barrel exports ALL folders
- [ ] Imports: external → internal → relative → types
- [ ] @{projectName}/* aliases
- [ ] cn() for Tailwind
- [ ] Component → Hook → TanStack Query → API
- [ ] Zustand (global) | TanStack Query (server) | useState (local)
- [ ] **Simplicity:** ~200 lines guideline, ~5 business props healthy
- [ ] **Natural reuse only:** No forced abstraction, intentional duplication OK

---

## Output

```markdown
### ✅ [Feature]

**Library:** libs/[feature] (apis, hooks, types, consts)
**Command:** `npx nx g @nx/js:lib {feature} --directory=libs/{feature}`
**Flow:** Component → Hook → TanStack Query → API
**Standards:** Named exports | Barrel exports | Shadcn UI | cn()
**Simplicity:** ~200 lines | ~5 business props | Self-contained | Natural reuse
```
