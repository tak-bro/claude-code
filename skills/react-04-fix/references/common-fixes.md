# React Common Fix Patterns

## Default Export -> Named Export
```typescript
// Before
export default Component

// After
export const Component = () => { }

// Update imports everywhere
import Component from './Component'       // Before
import { Component } from './Component'   // After
```

## Add Barrel Export
```typescript
// libs/feature/index.ts
export * from './apis';
export * from './hooks';
export * from './types';
export * from './consts';
export * from './Component';
```

## Direct API -> Hook
```typescript
// Before (in component)
const [data, setData] = useState();
useEffect(() => {
  fetchData().then(setData);
}, []);

// After
const { data } = useFeatureData();  // TanStack Query hook
```

## any -> Proper Type
```typescript
// Before
const data: any = await fetchData();

// After
const data: FeatureData[] = await fetchData();
```

## useState -> Zustand (shared state)
```typescript
// Before - local state shared via props drilling
const [user, setUser] = useState<User | null>(null);
// passed down 3+ levels...

// After - Zustand store
// stores/useUserStore.ts
interface UserState {
  user: User | null;
  setUser: (user: User) => void;
}

export const useUserStore = create<UserState>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));

// In component
const { user } = useUserStore();
```

## Props Explosion -> Composition
```typescript
// Before - too many props
<UserCard
  name={user.name}
  avatar={user.avatar}
  bio={user.bio}
  stats={user.stats}
  isOnline={user.isOnline}
  onEdit={handleEdit}
  onDelete={handleDelete}
/>

// After - pass domain object
<UserCard user={user} onEdit={handleEdit} onDelete={handleDelete} />
```

## Component Too Large -> Split
```typescript
// Before - 300+ line component with multiple concerns

// After - split by responsibility
// UserProfile.tsx (container/smart)
export const UserProfile = () => {
  const { data: user } = useUser();
  return (
    <>
      <UserHeader user={user} />
      <UserStats stats={user.stats} />
      <UserActions userId={user.id} />
    </>
  );
};

// UserHeader.tsx, UserStats.tsx, UserActions.tsx (presentational)
```
