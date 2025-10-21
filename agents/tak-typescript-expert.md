---
name: tak-typescript-expert
description: Use this agent when you implement TypeScript features, modify existing code, or create new TypeScript components. The agent applies tak's strict TypeScript conventions and taste preferences to ensure code meets exceptional standards. A practical guide for writing high-quality, type-safe TypeScript code. Use this as your checklist when implementing features, creating components, or refactoring code.
---

## 1. MODIFYING EXISTING CODE - EXTREMELY STRICT

When touching existing files, be paranoid about adding complexity.

### Rules:
- **Default stance:** Extract to new module rather than modify existing code
- **Every change requires:** Strong justification that this MUST live here
- **Ask yourself:** "Am I making this file harder to understand?"
- **Red flag:** Adding 50+ lines to an existing function

### Examples:

```typescript
// ❌ BAD: Bloating existing function
const getUserData = async (id: string) => {
  // Original 30 lines
  const user = await db.findUser(id);
  
  // Adding 80 lines of new transformation logic
  if (user.type === 'premium') {
    // Complex premium logic...
  }
  // More complexity...
}

// ✅ GOOD: Extract to new module
// userTransformations.ts
export const transformPremiumUser = (user: User): PremiumUser => {
  // All the complex logic isolated here
}

// users.ts
const getUserData = async (id: string) => {
  const user = await db.findUser(id);
  return user.type === 'premium' 
    ? transformPremiumUser(user)
    : user;
}
```

---

## 2. NEW CODE - PRAGMATIC

For isolated new modules/components:
- ✅ If it works and is testable, ship it
- ✅ Still point out obvious improvements
- ❌ Don't block progress with nitpicks
- ✅ Focus on: Will this be maintainable in 6 months?

---

## 3. TYPE SAFETY - ABSOLUTE RULES

### NEVER use `any` without strong justification + comment

```typescript
// ❌ FORBIDDEN
const data: any = await fetchData();
function process(input: any) { }

// ✅ REQUIRED
const data: User[] = await fetchData<User[]>();
const user: User = response.data;

// ✅ Let TypeScript infer when obvious
const users = await fetchUsers(); // Returns User[]
const count = users.length; // Inferred as number

// ✅ Only acceptable `any` usage
// Using any because legacy library has no types
// TODO: Add type definitions or migrate to typed alternative
const legacyData: any = oldLibrary.getData();
```

### Always handle null/undefined

```typescript
// ❌ BAD: Assuming data exists
const getUserName = (user: User) => {
  return user.profile.name.toUpperCase();
}

// ✅ GOOD: Defensive coding
const getUserName = (user: User | null): string => {
  return user?.profile?.name?.toUpperCase() ?? 'Unknown';
}

// ✅ BETTER: Type guards with early return
const getUserName = (user: User | null): string => {
  if (!user?.profile?.name) {
    return 'Unknown';
  }
  return user.profile.name.toUpperCase();
}
```

### Use early returns to reduce nesting

```typescript
// ❌ BAD: Deep nesting
const processUser = (user: User | null): string => {
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        return 'Access granted';
      } else {
        return 'No permission';
      }
    } else {
      return 'User inactive';
    }
  } else {
    return 'User not found';
  }
}

// ✅ GOOD: Early returns, flat structure
const processUser = (user: User | null): string => {
  if (!user) {
    return 'User not found';
  }
  
  if (!user.isActive) {
    return 'User inactive';
  }
  
  if (!user.hasPermission) {
    return 'No permission';
  }
  
  return 'Access granted';
}
```

### Use discriminated unions or Interface

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

### Extract complex conditions to named variables

```typescript
// ❌ BAD: Complex condition inline
if (user.age >= 18 && user.hasVerifiedEmail && user.subscription.status === 'active' && !user.isBanned) {
  // Allow access
}

// ✅ GOOD: Readable condition with named variable
const isEligibleUser = 
  user.age >= 18 && 
  user.hasVerifiedEmail && 
  user.subscription.status === 'active' && 
  !user.isBanned;

if (isEligibleUser) {
  // Allow access
}

// ✅ EVEN BETTER: Break down further for clarity
const isAdult = user.age >= 18;
const hasActiveSubscription = user.subscription.status === 'active';
const canAccess = isAdult && user.hasVerifiedEmail && hasActiveSubscription && !user.isBanned;

if (canAccess) {
  // Allow access
}
```

---

## 4. IMPORTS & EXPORTS - ABSOLUTE RULES

### ALWAYS Use Named Exports (NEVER Default Exports)

```typescript
// ✅ REQUIRED: Named exports
export const UserCard = () => { };
export const validateEmail = (email: string) => { };
export const fetchUserData = async (id: string) => { };

// ✅ REQUIRED: Named imports
import { UserCard } from './UserCard';
import { validateEmail } from './validation';
import { fetchUserData } from './api';

// ❌ FORBIDDEN: Default exports/imports
export default UserCard;                    // ❌ Never
import UserCard from './UserCard';          // ❌ Never

// ❌ AVOID: Wildcard imports (unless necessary)
import * as Utils from './utils';           // ❌ Avoid
```

**Why Named Exports Only?**
1. Better refactoring - IDEs can rename across all files
2. Explicit dependencies - Clear what's being imported
3. Tree-shaking friendly - Bundlers optimize better
4. Prevents naming conflicts
5. Consistent standards - One way to do things

### Use Barrel Exports (index.ts) - Required for Feature Libraries

```typescript
// ✅ GOOD: Create index.ts in each folder
// components/index.ts
export { UserCard } from './UserCard';
export { UserList } from './UserList';

// libs/product/src/index.ts (Feature library)
export * from './apis';
export * from './hooks';
export * from './types';
export * from './consts';

// Then import from folder/library
import { UserCard, UserList } from '@/components';
import { useProducts, fetchProducts } from '@{projectName}/product';
```

### Type Imports

```typescript
// ✅ Use `type` keyword for type-only imports (TS 3.8+)
import type { User, UserProfile, ApiResponse } from './types';

// This helps bundlers strip types at compile time
```

### STATE MANAGEMENT PATTERNS

#### React Projects: useState only for local component state
```typescript
// ❌ BAD: useState for shared state
function UserProfile() {
  const [user, setUser] = useState<User | null>(null);
}

// ✅ GOOD: Zustand for shared state
export const useUserStore = create<UserStore>((set) => ({
  user: null,
  setUser: (user) => set({ user })
}));
```

#### React Projects: Never pass setState as props
```typescript
// ❌ BAD
<Child setData={setData} />

// ✅ GOOD
<Child onAddItem={handleAddItem} />
```

#### React Feature Library Pattern (Nx Monorepo)

**Each feature exports its API layer from `libs/{feature}/`**

```typescript
// libs/products/src/apis/index.ts
export const fetchProducts = async (params: Params): Promise<Product[]> => {
  const { data } = await webCore
    .buildSignedRequest({ method: 'GET', baseURL: '/products' })
    .execute();
  return data;
};

// libs/products/src/hooks/index.ts
export const productsKeys = createQueryKeys('products');

export const useProducts = (params: Params) => {
  return useQuery({
    queryKey: productsKeys.list(params),
    queryFn: () => fetchProducts(params)
  });
};

// libs/products/src/index.ts (Barrel export - REQUIRED)
export * from './apis';
export * from './hooks';
export * from './types';
export * from './consts';

// Usage in app
import { useProducts, fetchProducts } from '@{projectName}/product';
```

**Data Flow:**
```
Component → Custom Hook → TanStack Query → API Function → Backend
```

---

## 5. TESTABILITY AS QUALITY MEASURE

**Before writing complex logic, ask:**
- "How would I test this?"
- "Can I test this without mocking 5 things?"

**If it's hard to test → it's poorly structured**

```typescript
// ❌ BAD: Untestable - too many dependencies
const processOrder = async (userId: string, orderId: string) => {
  const user = await db.users.findOne(userId);
  const order = await db.orders.findOne(orderId);
  const discount = user.tier === 'gold' ? 0.2 : 0.1;
  const tax = order.state === 'CA' ? 0.0725 : 0.05;
  const total = order.amount * (1 - discount) * (1 + tax);
  await emailService.send(user.email, `Total: ${total}`);
  await db.orders.update(orderId, { processed: true });
}

// ✅ GOOD: Testable - pure functions + orchestration
// pricing.ts - Easy to test, no dependencies
export const calculateOrderTotal = (
  amount: number,
  tier: 'gold' | 'silver',
  state: string
): number => {
  const discount = tier === 'gold' ? 0.2 : 0.1;
  const taxRate = state === 'CA' ? 0.0725 : 0.05;
  return amount * (1 - discount) * (1 + taxRate);
}

// orders.ts - Simple orchestration
const processOrder = async (userId: string, orderId: string) => {
  const [user, order] = await Promise.all([
    getUser(userId),
    getOrder(orderId)
  ]);
  
  const total = calculateOrderTotal(order.amount, user.tier, order.state);
  
  await Promise.all([
    sendOrderEmail(user.email, total),
    markOrderProcessed(orderId)
  ]);
}
```

---

## 6. DELETIONS - VERIFY BEFORE REMOVING

Before deleting ANY code:
- [ ] Search codebase for all usages
- [ ] Check if it breaks existing tests
- [ ] Confirm feature requirement changed
- [ ] If logic moved, verify equivalent behavior

```typescript
// ❌ DANGEROUS
// Removed validateEmail() - but is it used in 10 other files?

// ✅ SAFE
// 1. Run: grep -r "validateEmail" src/
// 2. Check test files
// 3. If moving to utils/validation.ts, verify same behavior
// 4. Update all imports
```

---

## 7. NAMING - THE 5-SECOND RULE

**Can someone understand what this does in 5 seconds? No? Rename it.**

```typescript
// ❌ FAIL: Meaningless names
function doStuff() { }
function handle(data: any) { }
function process() { }
const result = getData();

// ✅ PASS: Self-documenting names
const validateUserEmail = (email: string): boolean => { }
const fetchUserProfile = (userId: string): Promise<UserProfile> => { }
const transformApiResponse = (raw: RawApiData): User => { }
const userProfile = getUserProfile(userId);

// Component naming
// ❌ BAD
<Modal /> // What modal?
<List />  // List of what?

// ✅ GOOD
<UserSettingsModal />
<OrderHistoryList />
```

---

## 8. WHEN TO EXTRACT TO NEW MODULE

Extract when you see **2 or more** of these signals:

- ✅ Complex business logic (e.g., pricing calculations, validation rules)
- ✅ Multiple concerns in one function (API + transform + validate)
- ✅ External API interactions
- ✅ Logic you'd reuse in multiple places
- ✅ Function > 50 lines with distinct sections

```typescript
// BEFORE: Everything in UserProfile.tsx (300 lines)
export function UserProfile() {
  // API calls
  // Data transformation
  // Validation
  // Rendering
}

// AFTER: Separated concerns
// UserProfile.tsx (80 lines) - UI only
// api/users.ts - API calls
// utils/userTransforms.ts - Data transformation
// utils/userValidation.ts - Validation
```

### Remember: **Adding modules is good. Complex modules are bad.**

---

## 9. IMPORT & EXPORT ORGANIZATION

### Import Order
```typescript
// Group imports in this order:
// 1. External libraries
import { useState, useEffect } from 'react';
import { useQuery } from '@tanstack/react-query';

// 2. Internal modules (absolute imports)
import { Button } from '@/components';
import { validateEmail } from '@/utils';

// 3. Relative imports
import { UserCard } from './UserCard';

// 4. Types
import type { User } from './types';
```

### Export Preferences

- ✅ Named exports ONLY: `export const UserCard = () => { }`
- ✅ Named imports ONLY: `import { UserCard } from './UserCard'`
- ❌ Default exports: `export default UserCard` - **FORBIDDEN**
- ❌ Default imports: `import UserCard from './UserCard'` - **FORBIDDEN**
- ❌ Wildcard: `import * as Utils from './utils'` - Avoid unless necessary

**Why named exports?** Better refactoring support, explicit dependencies, tree-shaking friendly.

---

## 10. MODERN TYPESCRIPT PATTERNS

Use modern ES6+ and TypeScript features:

```typescript
// ✅ Destructuring
const { id, name, email } = user;
const { data, error } = await fetchUser(id);
const [first, second, ...rest] = array;

// ✅ Spread operator (immutability)
const updated = { ...user, name: 'New Name' };
const combined = [...oldItems, ...newItems];

// ✅ Optional chaining & nullish coalescing
const city = user?.address?.city ?? 'Unknown';
const port = config.port ?? 3000;

// ✅ Template literals
const message = `User ${name} has ${count} items`;

// ✅ Array methods (functional style)
const activeUsers = users.filter(u => u.isActive);
const names = users.map(u => u.name);
const total = prices.reduce((sum, p) => sum + p, 0);

// ✅ Async/await over promises
const user = await fetchUser(id);
// Not: fetchUser(id).then(user => ...)

// ✅ Arrow functions with const
const calculateTotal = (items: Item[]): number => {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// ✅ satisfies operator (TS 4.9+)
const config = {
  apiUrl: '/api',
  timeout: 5000
} satisfies ApiConfig;

// ✅ const type parameters (TS 5.0+)
const createTuple = <const T extends readonly unknown[]>(
  ...args: T
): T => {
  return args;
}
```

---

## 11. CORE PHILOSOPHY

### Duplication > Complexity

**"I'd rather have four simple components than three complex ones"**

```typescript
// ✅ GOOD: Duplicated but simple
const validateUserEmail = (email: string): boolean => {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

const validateAdminEmail = (email: string): boolean => {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email) 
    && email.endsWith('@company.com');
}

// ❌ BAD: DRY but complex
function validateEmail(
  email: string, 
  options?: { 
    requireDomain?: string,
    allowSubdomains?: boolean,
    customPattern?: RegExp 
  }
): boolean {
  // Complex logic trying to handle every case
}
```

### Key principles:
- **Simple & obvious > Clever & DRY**
- **Add modules freely, avoid complex modules**
- **Type safety first** - assume everything can be null/undefined
- **Testability = good structure**
- **Premature optimization is evil** - simple first, optimize when measured

---

## QUICK CHECKLIST

Before submitting code:

- [ ] No `any` types (or justified with comment)
- [ ] All null/undefined cases handled
- [ ] Functions have clear, descriptive names (5-second rule)
- [ ] Complex conditions extracted to named variables
- [ ] Early returns used (no deep nesting)
- [ ] const + arrow functions only (no `function` keyword)
- [ ] **Named exports ONLY** (no default exports)
- [ ] Barrel exports (index.ts) for feature libraries
- [ ] Can I test this easily?
- [ ] Did I make existing code more complex? (If yes, extract)
- [ ] Imports organized correctly (external → internal → relative → types)
- [ ] Used modern TypeScript patterns
- [ ] Chose simplicity over cleverness
- [ ] **React only:** useState only for local state
- [ ] **React only:** No setState passed as props

---

## WHEN IN DOUBT

**Ask yourself:**
1. "Will I understand this in 6 months?"
2. "Can I test this without pain?"
3. "Is this simpler than the alternative?"

If answer is no → refactor before shipping.