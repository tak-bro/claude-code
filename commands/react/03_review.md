# /react:review

## Purpose
Review React code against Feature Library pattern, TanStack Query best practices, and team standards.

## Agents
- **react-figma-ui-engineer**: Verify React patterns and UI implementation
- **tak-typescript-reviewer**: Type safety and TypeScript standards
- **code-simplicity-reviewer**: YAGNI violations and complexity
- **design-implementation-reviewer**: UI vs Figma (if applicable)

---

## Workflow

### Phase 1: Sequential (2 min)
```bash
1. Identify review scope
2. Check for React-specific patterns
3. Determine if UI review needed (Figma)
```

### Phase 2: Parallel Reviews 🔀 (10 min)
```bash
Lane 1: Feature Library Pattern Review
  - libs/{feature}/ structure
  - Barrel exports (index.ts)
  - Data flow: Component → Hook → API

Lane 2: Export/Import Review (CRITICAL)
  - Named exports ONLY (NO default)
  - Import organization
  - @{projectName}/* aliases

Lane 3: TypeScript Review (tak-typescript-reviewer)
  - Type safety
  - null/undefined handling
  - Modern patterns
```

### Phase 3: Parallel Analysis 🔀 (5 min)
```bash
Lane 1: TanStack Query Patterns
  - Query hooks structure
  - Mutation hooks with invalidation
  - Query key management

Lane 2: UI/Component Review
  - Shadcn UI usage
  - cn() utility for Tailwind
  - Component structure

Lane 3: State Management
  - Zustand (global) vs useState (local)
  - TanStack Query (server state)

Lane 4: Complexity (code-simplicity-reviewer)
  - Unnecessary abstractions
  - YAGNI violations
```

### Phase 4: Sequential (3 min)
```bash
4. If UI: design-implementation-reviewer (Figma comparison)
5. Generate consolidated report
6. Prioritize issues (Critical → Important → Nice-to-have)
```

---

## Review Checklist

### Export/Import Standards (CRITICAL)
- [ ] **ALL exports are named exports** (NO default exports anywhere)
- [ ] **ALL imports are named imports** (NO default imports anywhere)
- [ ] Barrel exports (index.ts) exist in ALL folders
- [ ] Imports organized: external → internal → relative → types
- [ ] Using @{projectName}/* aliases (not relative paths across libs)

### Feature Library Structure
- [ ] Feature library in libs/{feature}/
- [ ] Has apis/, hooks/, types/, consts/ folders
- [ ] Root index.ts exports all submodules
- [ ] Clean imports work: `import { useX } from '@{projectName}/feature'`

### Data Flow
- [ ] Component → Custom Hook → TanStack Query → API
- [ ] Components do NOT call API functions directly
- [ ] No business logic in components (should be in hooks)

### TanStack Query Patterns
- [ ] Query hooks use proper queryKey structure
- [ ] Query keys created with createQueryKeys()
- [ ] Mutations invalidate related queries
- [ ] useCustomMutation used for mutations
- [ ] Proper error handling in hooks

### Component Patterns
- [ ] Using Shadcn UI components from @{projectName}/ui-kit
- [ ] Tailwind classes applied with cn() utility
- [ ] const + arrow functions (NO function keyword)
- [ ] Props interfaces defined separately
- [ ] All null/undefined handled

### State Management
- [ ] Zustand ONLY for global client state (auth, theme)
- [ ] TanStack Query for server state
- [ ] useState ONLY for local component state
- [ ] NO setState passed as props

### TypeScript Standards
- [ ] Named exports ONLY
- [ ] No `any` types (or justified)
- [ ] Type imports use `import type`
- [ ] Modern TypeScript patterns

---

## Critical Anti-Patterns

```typescript
// 🔴 CRITICAL: Default exports
export default ItemCard;  // ❌ FORBIDDEN

// 🔴 CRITICAL: Default imports
import ItemCard from './ItemCard';  // ❌ FORBIDDEN

// 🔴 CRITICAL: Missing barrel exports
// libs/feature/src/apis/products.ts
export const fetchProducts = ...
// But no libs/feature/src/apis/index.ts

// 🔴 CRITICAL: Component calling API directly
const ItemList = () => {
  const [items, setItems] = useState([]);

  useEffect(() => {
    fetchItems().then(setItems);  // ❌ Bypass TanStack Query
  }, []);
};

// 🔴 CRITICAL: Relative imports across libraries
import { useProducts } from '../../../product/src/hooks';  // ❌

// 🔴 CRITICAL: Mixed import order
import { ItemCard } from './ItemCard';
import { useState } from 'react';
import type { Item } from './types';

// 🔴 CRITICAL: useState for shared state
const [user, setUser] = useState<User>();  // ❌ Use Zustand

// 🔴 CRITICAL: Passing setState as props
<Child setData={setData} />  // ❌ Pass handler function

// 🔴 CRITICAL: Inline styles
<div style={{ color: 'red' }}>  // ❌ Use Tailwind

// 🔴 CRITICAL: Not using cn() utility
<div className="text-red-500 bg-white">  // ❌ Should use cn()
```

---

## Review Output Structure

```markdown
### 🔍 Review: [Feature/Component]

**Score**: [score] / 10

**Export/Import Review** (CRITICAL)
✅ All named exports (no defaults)
✅ Barrel exports present
✅ Import organization correct
❌ Issue found: [description]

**Feature Library Review**
✅ Correct structure (apis, hooks, types, consts)
✅ Data flow pattern correct
✅ Clean imports work
❌ Issue found: [description]

**TanStack Query Review**
✅ Query hooks properly structured
✅ Mutations invalidate queries
✅ Error handling present
❌ Issue found: [description]

**Component Review**
✅ Shadcn UI components used
✅ cn() utility for Tailwind
✅ Proper component structure
❌ Issue found: [description]

**TypeScript Standards**
✅ Type safety
✅ Modern patterns
❌ Issue found: [description]

**Issues Found**
🔴 Critical: [issue] (fix: [solution])
  - Impact: [impact]
  - File: [path:line]
  - Example: [code]

🟡 Important: [issue]
  - Suggestion: [solution]
  - File: [path:line]

🟢 Nice-to-have: [issue]
  - Enhancement: [solution]

**Recommendations**
1. [Priority 1 fix]
2. [Priority 2 improvement]
3. [Future enhancement]

**Example Fix**
```typescript
// Before (❌)
export default ItemCard;
import ItemCard from './ItemCard';

// After (✅)
export const ItemCard = () => { };
import { ItemCard } from './ItemCard';
```
```

---

## Score Breakdown

- **10/10**: Perfect - All patterns correct, zero issues
- **8-9/10**: Excellent - Minor style issues only
- **6-7/10**: Good - Some important issues to fix
- **4-5/10**: Needs Work - Critical patterns violated
- **0-3/10**: Major Issues - Architecture problems

---

## Quick Decision Tree

```
Are there ANY default exports?
  → YES: 🔴 CRITICAL - Convert ALL to named exports
  → NO: Continue

Are there ANY default imports?
  → YES: 🔴 CRITICAL - Convert ALL to named imports
  → NO: Continue

Are barrel exports (index.ts) missing?
  → YES: 🔴 CRITICAL - Add barrel exports to all folders
  → NO: Continue

Is Component calling API directly?
  → YES: 🔴 CRITICAL - Use TanStack Query hooks
  → NO: Continue

Using relative imports across libraries?
  → YES: 🔴 CRITICAL - Use @{projectName}/* aliases
  → NO: Continue

Using useState for shared state?
  → YES: 🔴 CRITICAL - Use Zustand
  → NO: Continue

Inline styles present?
  → YES: 🟡 IMPORTANT - Use Tailwind with cn()
  → NO: Continue

Not using cn() for Tailwind classes?
  → YES: 🟡 IMPORTANT - Apply cn() utility
  → NO: ✅ PASS
```

---

## Feature Library Verification

```bash
# Check structure
ls libs/{feature}/src/
# Should see: apis/ hooks/ types/ consts/ index.ts

# Check barrel exports
cat libs/{feature}/src/index.ts
# Should export * from all submodules

# Check imports work
grep "from '@{projectName}/{feature}'" apps/web/src/**/*.tsx
# Should be clean imports, not deep paths

# Check for default exports (should find NONE)
grep -r "export default" libs/{feature}/
# Should return 0 results

# Check for default imports (should find NONE)
grep -r "import .* from" libs/{feature}/ | grep -v "import {"
# Should return 0 results or only package imports
```
