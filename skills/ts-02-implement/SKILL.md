---
name: ts-02-implement
description: "TypeScript 기능 구현. tak 컨벤션. Triggers on 'typescript implement', 'ts implement', '타입스크립트 구현', 'ts coding'."
tools: Read, Bash, Glob, Grep, Write, Edit, Agent
---

# TypeScript Implement

Implement features following tak's TypeScript conventions.

**Agents:** tak-typescript-expert, best-practices-researcher

---

(see `_shared/implement-common.md`)

---

## Boundaries

### Always Do
- Named exports only (`export const`)
- Barrel exports (index.ts)
- const + arrow functions (no `function` keyword)
- Handle null/undefined with early returns
- Run verification commands before done

### 🚫 Never Do
- `export default` (except App entrypoint, config)
- `function` keyword
- `any` without justification comment
- Deep nesting

---

## Patterns

**Named Exports Only**
```typescript
export const UserCard = () => { };
export const validateEmail = (email: string): boolean => { };
```

**const + Arrow Functions**
```typescript
const calculateTotal = (items: Item[]): number => {
  return items.reduce((sum, item) => sum + item.price, 0);
};
```

**Type Safety**
```typescript
const getUserName = (user: User | null): string => {
  if (!user?.name) return 'Unknown';
  return user.name.toUpperCase();
};

type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string };
```

**Early Returns**
```typescript
const processUser = (user: User | null): string => {
  if (!user) return 'Not found';
  if (!user.active) return 'Inactive';
  if (!user.hasPermission) return 'No permission';
  return 'Access granted';
};
```

**Complex Conditions → Named Variables**
```typescript
const isEligible = user.age >= 18 && user.verified && !user.banned;
if (isEligible) { /* ... */ }
```

---

## Checklist

- [ ] Named exports ONLY
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

## --team Mode
(see `_shared/team-mode.md`)

| Role | Responsibility | Files |
|------|---------------|-------|
| Core Logic Agent | Business logic, algorithms | `core/`, `utils/` |
| Types Agent | TypeScript types, interfaces | `types/` |
| Service Agent | External integrations, APIs | `services/` |
| Test Agent | Unit tests | `__tests__/`, `*.test.ts` |

---

## Output
(see `_shared/output-templates.md`)

→ Next: `/simplify` → `/ts-03-review`
