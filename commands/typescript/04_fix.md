# /typescript:04_fix

Fix issues identified in review phase, prioritized by severity.

**Agents:** tak-typescript-expert, code-simplicity-reviewer

---

## Pre-Fix

**From /typescript:03_review output, load:**
1. Fix Checklist (Critical → Important → Nice-to-have)
2. Specific file:line references
3. Fix code snippets provided

---

## Boundaries

### ✅ Always Do
- Fix ALL 🔴 Critical issues (no exceptions)
- Run verification after each fix
- Mark completed items in checklist
- Keep fixes minimal and focused

### ⚠️ Ask First
- Skip 🟡 Important issues
- Refactor beyond fix scope
- Add new features while fixing

### 🚫 Never Do
- Skip 🔴 Critical issues
- Change logic beyond the fix
- Introduce new patterns
- Leave verification failing

---

## Workflow

1. **Load (1min)**: Get Fix Checklist from review
2. **Fix Critical (varies)**: Fix ALL 🔴 issues, verify each
3. **Fix Important (varies)**: Fix 🟡 issues if time permits
4. **Verify (2min)**: Run all verification commands
5. **Report (1min)**: Generate completion report

---

## Fix Priority

```
🔴 Critical (MUST fix - no exceptions)
    │
    ├─ export default → named export
    ├─ any → proper types
    ├─ function → arrow function
    ├─ unhandled null → add guards
    └─ missing barrel → add index.ts
    │
    ▼
🟡 Important (SHOULD fix)
    │
    ├─ deep nesting → early returns
    ├─ unnamed conditions → named variables
    └─ testability improvements
    │
    ▼
🟢 Nice-to-have (OPTIONAL)
    │
    └─ simplification, naming, type inference
```

---

## Common Fixes

**Default Export → Named Export**
```typescript
// Before
export default validateEmail;

// After
export const validateEmail = (email: string): boolean => { };

// Update ALL imports
import validateEmail from './validation';  // Before
import { validateEmail } from './validation';  // After
```

**any → Proper Type**
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

**function → Arrow Function**
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

**Deep Nesting → Early Returns**
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

**Complex Condition → Named Variable**
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

**Add Barrel Export**
```typescript
// src/utils/index.ts
export * from './validation';
export * from './formatting';
export * from './helpers';
```

---

## Verification Commands

```bash
# {pm} = npm, yarn, pnpm, bun (use project's package manager)
# After each fix
{pm} run lint    # if available

# After all fixes
{pm} test        # if available
{pm} run build   # if available
```

---

## Output

```markdown
### ✅ Fixed: [Feature]

**Fixes Applied:**
- [x] 🔴 C1: Converted default export → named (validation.ts:45)
- [x] 🔴 C2: Added proper types (api.ts:23)
- [x] 🔴 C3: Converted function → arrow (helpers.ts:12)
- [x] 🟡 I1: Flattened nesting (user.ts:34)

**Verification:**
- [x] `{pm} run lint` ✓ (if available)
- [x] `{pm} test` ✓ (if available)
- [x] `{pm} run build` ✓ (if available)

**Skipped (with reason):**
- [ ] 🟢 N1: [Reason for skip]

**Final Score:** X/10 → 10/10
```

---

## Re-Review Criteria

After completing fixes, determine if re-review is needed based on these criteria:

| Condition | Action |
|-----------|--------|
| Had Critical issues | → Run `/typescript:03_review` again (required) |
| Had only Important issues | → Self-check and complete |
| Final score 8/10 or higher | → Complete |
| Final score below 8/10 | → Run `/typescript:03_review` again |

---

## Checklist

- [ ] ALL 🔴 Critical issues fixed
- [ ] 🟡 Important issues addressed (or justified skip)
- [ ] `{pm} run lint` passes (if available)
- [ ] `{pm} test` passes (if available)
- [ ] `{pm} run build` passes (if available)
- [ ] Fix report generated
- [ ] Re-review decision made
