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

---

## Output

```markdown
### ✅ [Feature]

**Library:** libs/[feature] (apis, hooks, types, consts)
**Command:** `npx nx g @nx/js:lib {feature} --directory=libs/{feature}`
**Flow:** Component → Hook → TanStack Query → API
**Standards:** Named exports | Barrel exports | Shadcn UI | cn()
```
