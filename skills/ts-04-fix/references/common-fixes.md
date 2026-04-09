# TypeScript Common Fix Patterns

## Default Export -> Named Export
```typescript
// Before
export default validateEmail;

// After
export const validateEmail = (email: string): boolean => { };

// Update ALL imports
import validateEmail from './validation';      // Before
import { validateEmail } from './validation';  // After
```

## any -> Proper Type
```typescript
// Before
const data: any = await fetchData();

// After
interface UserResponse {
  id: string;
  name: string;
}
const data: UserResponse = await fetchData<UserResponse>();

// Or with justification (rare)
// Using any because legacy API has inconsistent response shape
// TODO: Define proper types when API is updated
const data: any = await legacyFetch();
```

## function -> Arrow Function
```typescript
// Before
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// After
const calculateTotal = (items: Item[]): number => {
  return items.reduce((sum, item) => sum + item.price, 0);
};
```

## Deep Nesting -> Early Returns
```typescript
// Before
const process = (user: User | null): string => {
  if (user) {
    if (user.active) {
      if (user.verified) {
        return 'OK';
      }
    }
  }
  return 'Error';
};

// After
const process = (user: User | null): string => {
  if (!user) return 'No user';
  if (!user.active) return 'Inactive';
  if (!user.verified) return 'Not verified';
  return 'OK';
};
```

## Complex Condition -> Named Variable
```typescript
// Before
if (user.age >= 18 && user.verified && user.subscription.active && !user.banned) {
  // ...
}

// After
const isEligibleUser =
  user.age >= 18 &&
  user.verified &&
  user.subscription.active &&
  !user.banned;

if (isEligibleUser) {
  // ...
}
```

## Add Barrel Export
```typescript
// src/utils/index.ts
export * from './validation';
export * from './formatting';
export * from './helpers';
```

## Unhandled null/undefined -> Guards
```typescript
// Before
const getUserName = (user: User | null): string => {
  return user.name;  // runtime error if null
};

// After
const getUserName = (user: User | null): string => {
  if (!user) return 'Unknown';
  return user.name;
};
```
