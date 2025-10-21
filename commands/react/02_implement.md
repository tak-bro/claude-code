# /react:implement

Implement React features using Feature Library pattern (Nx monorepo).

**Agents:** react-figma-ui-engineer, framework-docs-researcher, tak-typescript-expert

---

## Workflow

1. Setup (3min): Identify feature lib structure, plan data flow, check Shadcn UI
2. **Parallel** (20min): API Functions | TanStack Query Hooks | Components | Types/Consts
3. Integration (5min): Create barrel exports, test data flow, verify aliases, apply cn()
4. Validation (2min): Lint, verify ALL named exports, barrel exports, import order

---

## Feature Library Structure

```
libs/{feature}/
├── apis/index.ts      # API functions
├── hooks/index.ts     # TanStack Query hooks
├── types/index.ts     # TypeScript types
├── consts/index.ts    # Constants
└── index.ts           # Barrel export (REQUIRED)
```

**Data Flow:** Component → Hook → TanStack Query → API

## Critical Patterns

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

- [ ] Named exports ONLY (no default)
- [ ] Barrel exports (index.ts) ALL folders
- [ ] Imports: external → internal → relative → types
- [ ] @{projectName}/* aliases (not relative across libs)
- [ ] cn() for Tailwind
- [ ] Component → Hook → TanStack Query → API
- [ ] Zustand (global) | TanStack Query (server) | useState (local)

---

## Output

```markdown
### ✅ Implementation: [Feature]

**Library:** libs/[feature] (apis, hooks, types, consts)

**Data Flow:** ✓ Component → Hook → TanStack Query → API

**Standards:** ✅ Named exports | Barrel exports | TanStack Query | Shadcn UI | cn()
```
