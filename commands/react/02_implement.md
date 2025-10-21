# /react:implement

## Purpose
Implement React features using Feature Library pattern (Nx monorepo), TanStack Query, and Shadcn UI components.

## Agents
- **react-figma-ui-engineer**: UI implementation from Figma designs
- **framework-docs-researcher**: React/TanStack Query reference
- **tak-typescript-expert**: TypeScript implementation checklist

---

## Workflow

### Phase 1: Setup (3 min)
```bash
1. Read requirements/plan
2. Identify feature library structure (libs/{feature}/)
3. Plan data flow: Component → Hook → TanStack Query → API
4. Check existing Shadcn UI components
```

### Phase 2: Parallel Implementation 🔀 (20 min)
```bash
# If components are independent
Lane 1: API Functions (libs/{feature}/src/apis/)
Lane 2: TanStack Query Hooks (libs/{feature}/src/hooks/)
Lane 3: Components (using hooks + Shadcn UI)
Lane 4: Types & Constants (libs/{feature}/src/types/, consts/)
```

### Phase 3: Integration (5 min)
```bash
4. Create barrel exports (index.ts) for all folders
5. Test data flow: Component → Hook → API
6. Verify imports use @{projectName}/* aliases
7. Apply cn() utility for Tailwind classes
```

### Phase 4: Validation (2 min)
```bash
8. Run lint/tests
9. Verify ALL exports are named exports (NO default)
10. Verify barrel exports exist
11. Verify import order (external → internal → relative → types)
```

---

## Critical Patterns

### Feature Library Structure
```
libs/feature-name/
├── src/
│   ├── apis/           # API functions
│   │   └── index.ts
│   ├── hooks/          # TanStack Query hooks
│   │   └── index.ts
│   ├── types/          # TypeScript types
│   │   └── index.ts
│   ├── consts/         # Constants
│   │   └── index.ts
│   └── index.ts        # Root barrel export (REQUIRED)
```

### API Functions Pattern
```typescript
// libs/feature/src/apis/index.ts
import { webCore } from '@{projectName}/web-core';
import type { Item } from '../types';

export const fetchItems = async (params: Params): Promise<Item[]> => {
  const { data } = await webCore
    .buildSignedRequest({ method: 'GET', baseURL: '/items' })
    .setParams(params)
    .execute();
  return data;
};

export const createItem = async (body: CreateItemDto): Promise<Item> => {
  const { data } = await webCore
    .buildSignedRequest({ method: 'POST', baseURL: '/items' })
    .setBody(body)
    .execute();
  return data;
};
```

### TanStack Query Hooks Pattern
```typescript
// libs/feature/src/hooks/index.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { createQueryKeys, useCustomMutation } from '@{projectName}/shared';
import { fetchItems, createItem } from '../apis';
import type { Item, CreateItemDto } from '../types';

export const itemsKeys = createQueryKeys('items');

export const useItems = (params: Params) => {
  return useQuery({
    queryKey: itemsKeys.list(params),
    queryFn: () => fetchItems(params),
    refetchOnWindowFocus: false
  });
};

export const useCreateItem = () => {
  const queryClient = useQueryClient();

  return useCustomMutation(
    (data: CreateItemDto) => createItem(data),
    {
      onSuccess: () => {
        queryClient.invalidateQueries({ queryKey: itemsKeys.lists() });
      }
    }
  );
};
```

### Component Pattern
```typescript
// apps/web/src/features/items/pages/ItemList.tsx
import { useItems, useCreateItem } from '@{projectName}/feature';
import { Button } from '@{projectName}/ui-kit';
import { cn } from '@{projectName}/ui-kit';

export const ItemListPage = () => {
  const { data: items, isLoading } = useItems({ page: 1 });
  const createMutation = useCreateItem();

  const handleCreate = (dto: CreateItemDto) => {
    createMutation.mutate(dto);
  };

  if (isLoading) return <div>Loading...</div>;

  return (
    <div className={cn('flex flex-col gap-4')}>
      {items?.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
      <Button onClick={() => handleCreate({ name: 'New' })}>
        Create
      </Button>
    </div>
  );
};
```

### Barrel Exports (REQUIRED)
```typescript
// libs/feature/src/index.ts (Root barrel)
export * from './apis';
export * from './hooks';
export * from './types';
export * from './consts';

// Usage - Clean imports
import { useItems, fetchItems, type Item } from '@{projectName}/feature';
```

---

## Anti-Patterns to Avoid

```typescript
// ❌ Default exports
export default ItemCard;  // NEVER

// ❌ Default imports
import ItemCard from './ItemCard';  // NEVER

// ✅ Named exports ONLY
export const ItemCard = () => { };
import { ItemCard } from './ItemCard';

// ❌ Direct imports (bypassing barrel)
import { useItems } from '@{projectName}/feature/src/hooks/useItems';

// ✅ Barrel imports
import { useItems } from '@{projectName}/feature';

// ❌ Component calling API directly
const fetchData = async () => {
  const data = await axios.get('/items');  // ❌ Bypass hooks
};

// ✅ Use TanStack Query hooks
const { data } = useItems();

// ❌ Relative imports across libraries
import { useProducts } from '../../../product/src/hooks';

// ✅ Workspace aliases
import { useProducts } from '@{projectName}/product';

// ❌ Mixed import order
import { ItemCard } from './ItemCard';
import { useState } from 'react';

// ✅ Organized imports
import { useState } from 'react';
import { ItemCard } from './ItemCard';

// ❌ Inline styles
<div style={{ color: 'red' }}>

// ✅ Tailwind classes with cn()
<div className={cn('text-red-500')}>

// ❌ useState for shared state
const [user, setUser] = useState<User | null>(null);

// ✅ Zustand for shared state
const { user } = useUserStore();
```

---

## Checklist Before Submit

- [ ] **CRITICAL:** All exports are named exports (NO default)
- [ ] **CRITICAL:** All imports are named imports (NO default)
- [ ] Barrel exports (index.ts) in ALL folders
- [ ] Feature library structure: apis, hooks, types, consts
- [ ] Data flow: Component → Hook → TanStack Query → API
- [ ] Imports organized: external → internal → relative → types
- [ ] Using @{projectName}/* aliases (not relative paths across libs)
- [ ] Tailwind classes with cn() utility
- [ ] Shadcn UI components used
- [ ] const + arrow functions (no function keyword)
- [ ] All null/undefined handled

---

## State Management Rules

```typescript
// ✅ Zustand - Global client state
const { isAuthenticated, profile } = useWebCoreStore();

// ✅ TanStack Query - Server state
const { data: items } = useItems();

// ✅ useState - Local component state only
const [isOpen, setIsOpen] = useState(false);

// ❌ useState for shared state
const [user, setUser] = useState<User>();  // Should use Zustand

// ❌ Passing setState as props
<Child setData={setData} />  // Should pass handler function
```

---

## Output Structure

```markdown
### ✅ Implementation: [Feature]

**Feature Library**
- Library: libs/[feature-name]/
- Structure: apis, hooks, types, consts
- Barrel exports: ✓ All folders

**Files**
- ✨ New: [paths]
- 🔧 Modified: [paths]

**Data Flow**
✓ Component → Custom Hook → TanStack Query → API Function

**Patterns Applied**
✓ Feature Library pattern
✓ Named exports ONLY
✓ Barrel exports everywhere
✓ TanStack Query for server state
✓ Shadcn UI components
✓ cn() utility for Tailwind

**Tests**
✓ [test] (passed)

**Import Examples**
```typescript
// Clean feature imports
import { useItems, fetchItems, type Item } from '@{projectName}/feature';
import { Button } from '@{projectName}/ui-kit';
```
```
