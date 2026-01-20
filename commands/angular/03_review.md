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

### Always Do
- Check ALL Critical items (auto-fail if missed)
- Provide specific file:line references
- Include fix code snippets
- Generate Fix Checklist for next phase

### Ask First
- Suggest major refactoring
- Recommend architecture changes

### Never Do
- Skip Critical checks
- Give vague feedback without fix examples
- Approve code with Critical issues

---

## Checks (Priority Order)

### Critical (Auto-Fail)
1. Standalone component used (must be Module-based)?
2. Component -> API direct call (bypassing ComponentStore)?
3. `export default` found?
4. DestroyedService injection (should use component-level destroyed$)?
5. Business logic in presentational component?
6. Missing message pattern in ComponentStore state?
7. (Ionic) Direct controller usage without service wrapper?

### Important
8. Missing component-level `destroyed$` cleanup?
9. Page vs Component separation violated?
10. Store not provided in Module's providers?
11. Guards order wrong? (App -> Auth -> Feature)
12. API methods missing `$` suffix?
13. (Multi-env) LocalStorage keys missing `_${env}` suffix?

### Nice-to-have
14. Could selectors be combined?
15. Effect error handling could be better?
16. Naming could be clearer?

---

## Decision Tree

```
Standalone component? -> Critical: Convert to Module-based
Component -> API direct? -> Critical: Use ComponentStore
export default? -> Critical: Convert to named export
DestroyedService injection? -> Critical: Use component-level destroyed$
Business logic in Component? -> Critical: Move to Page or Store
Missing message pattern? -> Critical: Add message field to state
(Ionic) Direct controller? -> Critical: Use service wrapper
Missing destroyed$ cleanup? -> Important: Add cleanup
Store not in Module providers? -> Important: Add to providers
Wrong Guard order? -> Important: Fix to App -> Auth -> Feature
(Multi-env) Missing _${env}? -> Important: Add suffix
All pass -> Score 10/10
```

---

## Verification Commands

```bash
# {pm} = npm, yarn, pnpm, bun (use project's package manager)
grep -r "standalone: true" src/app/modules/
grep -r "export default" src/app/
grep -r "DestroyedService" src/app/modules/ --include="*.ts"
# (Ionic) Check for direct controller usage
grep -r "modalCtrl\|toastCtrl\|loadingCtrl" src/app/modules/ --include="*.ts"
# (Multi-env) Check for localStorage without env suffix
grep -r "localStorage\|sessionStorage" src/app/ | grep -v "_\${environment"
ng build
{pm} run lint
```

---

## Output (-> fix)

```markdown
### Review: [Feature] - Score: X/10

**Summary:**
- Critical: X issues (must fix)
- Important: X issues (should fix)
- Nice-to-have: X issues (optional)

---

## Critical Issues

### C1: [Issue Title]
- **File:** `src/app/modules/feature/pages/list.page.ts:15`
- **Problem:** DestroyedService injected instead of component-level destroyed$
- **Fix:**
```typescript
// Before
constructor(private destroyed$: DestroyedService) {}

// After
private destroyed$: ReplaySubject<boolean> = new ReplaySubject(1);

ngOnDestroy(): void {
    this.destroyed$.next(true);
    this.destroyed$.complete();
}
```

### C2: [Issue Title]
- **File:** `src/app/modules/feature/components/list.component.ts:8`
- **Problem:** Presentational component has store injection
- **Fix:**
```typescript
// Before
@Component({ selector: 'feature-list' })
export class FeatureListComponent {
    constructor(private store: FeatureStore) {}
}

// After
@Component({ selector: 'feature-list' })
export class FeatureListComponent {
    @Input() items: FeatureView[] = [];
    @Output() selectItem = new EventEmitter<string>();
}
```

---

## Important Issues

### I1: [Issue Title]
- **File:** `src/app/modules/feature/stores/feature.store.ts:5`
- **Problem:** Missing message pattern in state
- **Fix:**
```typescript
// Before
export interface FeatureState {
    items: FeatureView[];
}

// After
export interface FeatureState {
    message: FeatureStoreMessage;
    isFetching: boolean;
    items: FeatureView[];
}
```

---

## Nice-to-have

### N1: [Issue Title]
- **Suggestion:** [Optional improvement]

---

## Fix Checklist (-> /angular:fix)

**Critical (Must Fix):**
- [ ] C1: Replace DestroyedService with destroyed$ (list.page.ts:15)
- [ ] C2: Remove store from presentational component (list.component.ts:8)

**Important (Should Fix):**
- [ ] I1: Add message pattern to store state (feature.store.ts:5)

**Nice-to-have (Optional):**
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
| 8-9 | Minor issues only (Nice-to-have) |
| 6-7 | Important issues |
| 4-5 | Critical issues |
| 0-3 | Major problems, needs rework |
