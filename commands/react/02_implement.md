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

## --team Mode (Agent Teams)

Use `--team` flag for large features requiring parallel implementation.

**Cost:** 3-5x tokens vs standard mode. Use for complex features only.

### Team Composition

| Role | Responsibility | Files |
|------|---------------|-------|
| API Agent | API functions, endpoints | `apis/` |
| Hook Agent | TanStack Query hooks, Zustand | `hooks/` |
| Component Agent | React components, UI | `components/` |
| Types Agent | TypeScript types, constants | `types/`, `consts/` |

### Coordination

1. **Shared Task List**: All agents use TaskCreate/TaskUpdate for progress
2. **Interface Changes**: When types change, notify other agents via messages
3. **No File Conflicts**: Each agent owns distinct folders
4. **Integration Point**: Main agent coordinates final barrel exports

### Workflow

```
1. Main agent creates Task List from Implementation Checklist
2. Spawn 4 parallel agents (API, Hook, Component, Types)
3. Each agent:
   - Claims tasks via TaskUpdate (owner, in_progress)
   - Implements assigned files
   - Marks tasks completed
   - Notifies if interface changes affect others
4. Main agent:
   - Monitors progress via TaskList
   - Handles integration (barrel exports)
   - Runs verification commands
```

### When to Use

✅ **Use --team:**
- Feature with 4+ files across folders
- Complex data flow needing parallel work
- Time-sensitive large features

❌ **Skip --team:**
- Single component/hook
- Small bug fixes
- Features touching 1-2 files

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
