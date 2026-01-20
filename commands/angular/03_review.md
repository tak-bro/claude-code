# /angular:review

Review Angular code and generate actionable fix checklist for next phase.

**Agents:** angular-componentstore-expert, tak-typescript-reviewer, code-simplicity-reviewer

---

## Pre-Review

**From /plan and /implement outputs:**
1. Review Focus items from plan
2. Implementation notes from implement
3. Files created/modified list

---

## Boundaries

### ✅ Always Do
- Check ALL Critical items (auto-fail if missed)
- Provide specific file:line references
- Include fix code snippets
- Generate Fix Checklist for next phase

### ⚠️ Ask First
- Suggest major refactoring
- Recommend architecture changes

### 🚫 Never Do
- Skip Critical checks
- Give vague feedback without fix examples
- Approve code with Critical issues

---

## Workflow

1. **Setup (2min)**: Load plan's Review Focus, implement's Notes
2. **Parallel (10min)**: ComponentStore | Architecture | TypeScript | Storage
3. **Analysis (5min)**: Cleanup patterns, env suffixes, complexity
4. **Final (3min)**: Generate Fix Checklist, consolidate score

---

## Checks (Priority Order)

### 🔴 Critical (Auto-Fail)
1. Separate Facade class exists? (ComponentStore IS Facade)
2. `providedIn: 'root'` for ComponentStore?
3. Component → API direct call (bypassing ComponentStore)?
4. Standalone component used (must be Module-based)?
5. Direct Ionic controller (`modalCtrl.create()`)?
6. `export default` found?

### 🟡 Important
7. Missing `_${env}` suffix on LocalStorage keys?
8. Missing DestroyedService cleanup?
9. Guards order wrong? (App → Auth → Feature)
10. Component has business logic? (should be in ComponentStore)

### 🟢 Nice-to-have
11. Could selectors be combined?
12. Effect error handling could be better?
13. Naming could be clearer?

---

## Decision Tree

```
Separate Facade class? → 🔴 Remove, use ComponentStore
providedIn: 'root' on Store? → 🔴 Component-scoped providers
Component → API direct? → 🔴 Use ComponentStore
Standalone component? → 🔴 Convert to Module-based
Ionic controller direct? → 🔴 Use service wrapper
export default? → 🔴 Convert to named export
Missing _${env} suffix? → 🟡 Add environment suffix
Missing DestroyedService? → 🟡 Add cleanup
Wrong Guard order? → 🟡 Fix to App → Auth → Feature
✅ All pass → Score 10/10
```

---

## Verification Commands

```bash
# {pm} = npm, yarn, pnpm, bun (use project's package manager)
grep -r "providedIn: 'root'" src/app/features/
grep -r "export default" src/app/
grep -r "modalCtrl\|toastCtrl\|loadingCtrl" src/app/ --include="*.ts"
grep -r "localStorage\|sessionStorage" src/app/ | grep -v "_${environment.env}"
ng build
{pm} run lint
```

---

## Output (→ fix 단계 입력)

```markdown
### 🔍 Review: [Feature] - Score: X/10

**Summary:**
- 🔴 Critical: X issues (must fix)
- 🟡 Important: X issues (should fix)
- 🟢 Nice-to-have: X issues (optional)

---

## 🔴 Critical Issues

### C1: [Issue Title]
- **File:** `src/app/features/orders/orders.facade.ts`
- **Problem:** Separate Facade class exists
- **Fix:** Remove facade, move logic to ComponentStore
```typescript
// Delete orders.facade.ts entirely
// Move all methods to OrderStore
```

### C2: [Issue Title]
- **File:** `src/app/features/orders/orders.store.ts:5`
- **Problem:** `@Injectable({ providedIn: 'root' })`
- **Fix:**
```typescript
// Before
@Injectable({ providedIn: 'root' })
export class OrderStore extends ComponentStore<OrderState> {}

// After
@Injectable()  // Component provides it
export class OrderStore extends ComponentStore<OrderState> {}

// In component
@Component({
  providers: [OrderStore, DestroyedService]  // Add here
})
```

---

## 🟡 Important Issues

### I1: [Issue Title]
- **File:** `src/app/services/storage.service.ts:42`
- **Problem:** Missing `_${env}` suffix
- **Fix:**
```typescript
// Before
this.storage.set('user_theme', theme);

// After
this.storage.set(`user_theme_${environment.env}`, theme);
```

---

## 🟢 Nice-to-have

### N1: [Issue Title]
- **Suggestion:** [Optional improvement]

---

## Fix Checklist (→ /angular:fix 입력)

**🔴 Critical (Must Fix):**
- [ ] C1: Remove separate Facade class (orders.facade.ts)
- [ ] C2: Remove providedIn: 'root' (orders.store.ts:5)

**🟡 Important (Should Fix):**
- [ ] I1: Add _${env} suffix (storage.service.ts:42)

**🟢 Nice-to-have (Optional):**
- [ ] N1: [Description]

---

**Verification After Fix:**
```bash
ng build
{pm} run lint
ng test --watch=false
```
```

---

## Scoring

| Score | Meaning |
|-------|---------|
| 10 | Perfect - no issues |
| 8-9 | Minor issues only (🟢) |
| 6-7 | Important issues (🟡) |
| 4-5 | Critical issues (🔴) |
| 0-3 | Major problems, needs rework |
