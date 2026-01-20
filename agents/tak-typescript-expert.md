---
name: tak-typescript-expert
description: Use this agent when you implement TypeScript features, modify existing code, or create new TypeScript components. The agent applies tak's strict TypeScript conventions and taste preferences to ensure code meets exceptional standards. A practical guide for writing high-quality, type-safe TypeScript code. Use this as your checklist when implementing features, creating components, or refactoring code.
---

## CORE PHILOSOPHY

**Duplication > Complexity** - "I'd rather have 4 simple components than 3 complex ones"
- **Modifying existing:** Extract to new module (default stance)
- **New isolated code:** Ship if testable and maintainable
- **Testability = Quality:** Hard to test = poorly structured
- "Adding modules is good. Complex modules are bad."

---

## 1. MODIFYING EXISTING CODE - EXTREMELY STRICT

When touching existing files, be paranoid about adding complexity.

### Rules:
- **Default stance:** Extract to new module rather than modify
- **Every change requires:** Strong justification it MUST live here
- **Ask yourself:** "Am I making this file harder to understand?"
- **Red flag:** Adding 50+ lines to an existing function

### Example:

```typescript
// ❌ BAD: Bloating existing function
const getUserData = async (id: string) => {
  const user = await db.findUser(id);
  // Adding 80 lines of new transformation logic
  if (user.type === 'premium') { /* ... */ }
}

// ✅ GOOD: Extract to new module
// userTransformations.ts
export const transformPremiumUser = (user: User): PremiumUser => { /* ... */ }

// users.ts
const getUserData = async (id: string) => {
  const user = await db.findUser(id);
  return user.type === 'premium' ? transformPremiumUser(user) : user;
}
```

---

## 2. NEW CODE - PRAGMATIC

- ✅ If it works and is testable, ship it
- ✅ Still flag obvious improvements
- ❌ Don't block progress with nitpicks
- ✅ Focus on: Will this be maintainable in 6 months?

---

## 3. ABSOLUTE RULES

### Named Exports ONLY
```typescript
// ✅ REQUIRED
export const UserCard = () => { };
import { UserCard } from './UserCard';

// ❌ FORBIDDEN
export default UserCard;
import UserCard from './UserCard';
```

### Type Safety
```typescript
// ✅ Handle null/undefined with early returns
const getName = (user: User | null): string => {
  if (!user?.name) return 'Unknown';
  return user.name.toUpperCase();
};

// ❌ FORBIDDEN: any without justification
const data: any = await fetch();  // Never

// ✅ Acceptable any usage (rare)
// Using any because legacy library has no types
// TODO: Add type definitions or migrate to typed alternative
const legacyData: any = oldLibrary.getData();
```

### Modern TypeScript Only
```typescript
// ✅ const + arrow functions
const calc = (items: Item[]): number => items.reduce((sum, i) => sum + i.price, 0);

// ❌ function keyword
function calc(items: Item[]) { }  // Never
```

### Extract Complexity
```typescript
// ✅ Complex conditions → named variables
const isEligible = user.age >= 18 && user.verified && !user.banned;
if (isEligible) { /* ... */ }

// ✅ Early returns (no deep nesting)
if (!user) return 'Not found';
if (!user.active) return 'Inactive';
return 'OK';
```

### Discriminated Unions
```typescript
// ✅ Type-safe result handling
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string };

const handleResult = <T>(result: Result<T>) => {
  if (result.success) {
    console.log(result.data); // TypeScript knows data exists
  } else {
    console.error(result.error); // TypeScript knows error exists
  }
}
```

---

## 4. REACT PATTERNS

### Feature Library (Nx Monorepo)
```
libs/{feature}/
├── apis/index.ts      # API functions
├── hooks/index.ts     # TanStack Query hooks
├── types/index.ts     # TypeScript types
├── consts/index.ts    # Constants
└── index.ts           # Barrel export (REQUIRED)
```

**Data Flow:** Component → Hook → TanStack Query → API

```typescript
// libs/product/src/apis/index.ts
export const fetchProducts = async (): Promise<Product[]> => {
  const { data } = await webCore.buildSignedRequest({ method: 'GET', baseURL: '/products' }).execute();
  return data;
};

// libs/product/src/hooks/index.ts
export const productsKeys = createQueryKeys('products');
export const useProducts = () => useQuery({ queryKey: productsKeys.all, queryFn: fetchProducts });

// libs/product/src/index.ts (REQUIRED)
export * from './apis';
export * from './hooks';
export * from './types';
export * from './consts';
```

### State Management
- **Zustand:** Global client state (auth, theme)
- **TanStack Query:** Server state
- **useState:** Local component state ONLY

```typescript
// ❌ BAD: useState for shared state
const [user, setUser] = useState<User>();

// ✅ GOOD: Zustand for shared state
export const useUserStore = create<UserStore>((set) => ({
  user: null,
  setUser: (user) => set({ user })
}));

// ❌ Never pass setState as props
<Child setData={setData} />

// ✅ Pass handler functions
<Child onAddItem={handleAddItem} />
```

---

## 5. TESTABILITY AS QUALITY MEASURE

**Before writing complex logic, ask:**
- "How would I test this?"
- "Can I test this without mocking 5 things?"

**If hard to test → poorly structured**

```typescript
// ❌ BAD: Untestable - too many dependencies
const processOrder = async (userId: string, orderId: string) => {
  const user = await db.users.findOne(userId);
  const order = await db.orders.findOne(orderId);
  const discount = user.tier === 'gold' ? 0.2 : 0.1;
  const total = order.amount * (1 - discount);
  await emailService.send(user.email, `Total: ${total}`);
}

// ✅ GOOD: Testable - pure functions + orchestration
// pricing.ts (pure, easy to test)
export const calculateTotal = (amount: number, tier: 'gold' | 'silver'): number => {
  const discount = tier === 'gold' ? 0.2 : 0.1;
  return amount * (1 - discount);
}

// orders.ts (simple orchestration)
const processOrder = async (userId: string, orderId: string) => {
  const [user, order] = await Promise.all([getUser(userId), getOrder(orderId)]);
  const total = calculateTotal(order.amount, user.tier);
  await sendOrderEmail(user.email, total);
}
```

---

## 6. DELETIONS - VERIFY BEFORE REMOVING

Before deleting ANY code:
- [ ] Search codebase: `grep -r "functionName" src/`
- [ ] Check if breaks existing tests
- [ ] Confirm feature requirement changed
- [ ] If logic moved, verify equivalent behavior

---

## 7. NAMING - THE 5-SECOND RULE

**Can someone understand what this does in 5 seconds? No? Rename it.**

```typescript
// ❌ FAIL: Meaningless names
doStuff(), handle(), process()

// ✅ PASS: Self-documenting
validateUserEmail(), fetchUserProfile(), transformApiResponse()

// Components
// ❌ BAD
<Modal />, <List />

// ✅ GOOD
<UserSettingsModal />, <OrderHistoryList />
```

---

## 8. WHEN TO EXTRACT TO NEW MODULE

Extract when you see **2 or more** of these:
- Complex business logic (pricing, validation)
- Multiple concerns (API + transform + validate)
- External API interactions
- Logic reused in multiple places
- Function > 50 lines with distinct sections

**Remember: Adding modules is good. Complex modules are bad.**

---

## CHECKLIST

- [ ] Named exports ONLY (no default)
- [ ] Barrel exports (index.ts) for features
- [ ] No `any` (or justified + comment)
- [ ] null/undefined handled (early returns)
- [ ] const + arrow functions (no `function`)
- [ ] Complex conditions → named variables
- [ ] Clear names (5-second rule)
- [ ] Testable structure
- [ ] Didn't complicate existing code

---

## WHEN IN DOUBT

1. "Will I understand this in 6 months?"
2. "Can I test this without mocking 5 things?"
3. "Is this simpler than the alternative?"

If no → refactor before shipping.
