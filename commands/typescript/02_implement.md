# /typescript:02_implement

Implement TypeScript features following team standards.

**Agents:** tak-typescript-expert, framework-docs-researcher

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
- Barrel exports (index.ts) for modules
- const + arrow functions (no `function` keyword)
- Handle null/undefined with early returns
- Run verification commands before done

### ⚠️ Ask First
- Adding new dependencies
- Modifying shared utilities
- Creating new patterns not in codebase

### 🚫 Never Do
- `export default`
- `function` keyword (use arrow functions)
- `any` without justification comment
- Deep nesting (use early returns)
- Skip type safety

---

## Workflow

1. **Setup (3min)**: Read requirements, identify parallel components
2. **Parallel (15min)**: Core logic | Components/UI | Tests
3. **Integration (3min)**: Integrate, run tests
4. **Validation (2min)**: Standards check, lint/tests

---

## Patterns

**Named Exports Only**
```typescript
// ✅ REQUIRED
export const UserCard = () => { };
export const validateEmail = (email: string): boolean => { };

// ❌ FORBIDDEN
export default UserCard;
```

**const + Arrow Functions**
```typescript
// ✅ REQUIRED
const calculateTotal = (items: Item[]): number => {
  return items.reduce((sum, item) => sum + item.price, 0);
};

// ❌ FORBIDDEN
function calculateTotal(items: Item[]): number { }
```

**Type Safety**
```typescript
// ✅ Handle null/undefined
const getUserName = (user: User | null): string => {
  if (!user?.name) return 'Unknown';
  return user.name.toUpperCase();
};

// ✅ Discriminated unions
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string };
```

**Early Returns**
```typescript
// ✅ GOOD - flat structure
const processUser = (user: User | null): string => {
  if (!user) return 'Not found';
  if (!user.active) return 'Inactive';
  if (!user.hasPermission) return 'No permission';
  return 'Access granted';
};

// ❌ BAD - deep nesting
const processUser = (user: User | null): string => {
  if (user) {
    if (user.active) {
      if (user.hasPermission) {
        return 'Access granted';
      }
    }
  }
  return 'Error';
};
```

**Complex Conditions → Named Variables**
```typescript
// ✅ GOOD
const isEligible = user.age >= 18 && user.verified && !user.banned;
if (isEligible) { /* ... */ }

// ❌ BAD
if (user.age >= 18 && user.verified && !user.banned) { /* ... */ }
```

---

## Verification Commands

```bash
# {pm} = npm, yarn, pnpm, bun (use project's package manager)
{pm} run lint    # if available
{pm} test        # if available
```

---

## Checklist

- [ ] Named exports ONLY (no `export default`)
- [ ] Barrel exports (index.ts)
- [ ] const + arrow (no `function`)
- [ ] No `any` (or justified + comment)
- [ ] null/undefined handled (early returns)
- [ ] Complex conditions → named variables
- [ ] Clear names (5-second rule)
- [ ] Testable structure
- [ ] Didn't complicate existing code
- [ ] Verification commands pass

---

## Output (→ /typescript:03_review 입력)

```markdown
### ✅ Implemented: [Feature]

**Files Created/Modified:**
- ✨ `src/utils/validation.ts`
- ✨ `src/utils/index.ts`
- 🔧 `src/services/user.service.ts`

**Checklist Completed:**
- [x] Named exports
- [x] Barrel exports
- [x] Type safety

**Verification:**
- [x] `{pm} run lint` ✓ (if available)
- [x] `{pm} test` ✓ (if available)

**Notes for Review:**
- [Implementation decisions]
- [Areas needing attention]
```
