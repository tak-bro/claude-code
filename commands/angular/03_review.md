# /angular:review

## Purpose
Review Angular code against ComponentStore-as-Facade pattern, Module-based architecture, and team standards.

## Agents
- **angular-componentstore-expert**: Verify ComponentStore patterns
- **tak-typescript-reviewer**: Type safety and TypeScript standards
- **code-simplicity-reviewer**: YAGNI violations and complexity

---

## Workflow

### Phase 1: Sequential (2 min)
```bash
1. Identify review scope
2. Check for Angular-specific patterns
```

### Phase 2: Parallel Reviews 🔀 (10 min)
```bash
Lane 1: ComponentStore Pattern Review
  - ComponentStore = Facade (no separate Facade)
  - Component-scoped providers
  - Effects → API pattern

Lane 2: Architecture Review
  - Module-based (NOT standalone)
  - Component → ComponentStore → API flow
  - Service wrappers (NOT Ionic controllers)

Lane 3: TypeScript Review (tak-typescript-reviewer)
  - Named exports ONLY
  - Type safety
  - null/undefined handling
```

### Phase 3: Parallel Analysis 🔀 (5 min)
```bash
Lane 1: Environment & Storage
  - LocalStorage keys have _${env} suffix
  - Environment config usage

Lane 2: Subscription Cleanup
  - DestroyedService pattern
  - takeUntil usage

Lane 3: Complexity (code-simplicity-reviewer)
  - Unnecessary abstractions
  - YAGNI violations
```

### Phase 4: Sequential (3 min)
```bash
3. Generate consolidated report
4. Prioritize issues (Critical → Important → Nice-to-have)
```

---

## Review Checklist

### ComponentStore Pattern
- [ ] ComponentStore acts as Facade (no separate Facade class)
- [ ] ComponentStore provided at component/module level (NOT root)
- [ ] Has Selectors, Updaters, Effects
- [ ] Effects call API services
- [ ] Effects handle errors with Toast/Loader services

### Architecture
- [ ] Component depends ONLY on ComponentStore
- [ ] Component does NOT call API directly
- [ ] Module-based (NOT standalone components)
- [ ] Guards ordered: App → Auth → Feature

### Service Patterns
- [ ] API services return Observable<T>
- [ ] Service wrappers used (ModalService, ToastService, LoaderService)
- [ ] NO direct Ionic controller usage (modalCtrl, toastCtrl, etc.)

### Subscription Management
- [ ] DestroyedService provided at component level
- [ ] Subscriptions use takeUntil(destroyed$)
- [ ] No memory leaks

### Storage & Environment
- [ ] LocalStorage keys have `_${env}` suffix
- [ ] Environment config used correctly

### TypeScript Standards
- [ ] Named exports ONLY (NO default exports)
- [ ] const + arrow functions (NO function keyword)
- [ ] All null/undefined cases handled
- [ ] No `any` types (or justified)

---

## Critical Anti-Patterns

```typescript
// 🔴 CRITICAL: Component calling API directly
@Component({ ... })
export class FeatureComponent {
  ngOnInit() {
    this.api.getItems$().subscribe();  // ❌ Bypass ComponentStore
  }
}

// 🔴 CRITICAL: Separate Facade class
export class FeatureFacade {  // ❌ ComponentStore IS the Facade
  constructor(private store: FeatureStore) {}
}

// 🔴 CRITICAL: ComponentStore in root
@Injectable({ providedIn: 'root' })  // ❌ Should be component-scoped
export class FeatureStore extends ComponentStore<State> { }

// 🔴 CRITICAL: Ionic controllers directly
const modal = await this.modalCtrl.create({ ... });  // ❌ Use ModalService

// 🔴 CRITICAL: Missing env suffix
this.storage.set('theme', 'dark');  // ❌ Need _${env}

// 🔴 CRITICAL: Standalone component
@Component({ standalone: true })  // ❌ Use Module-based

// 🔴 CRITICAL: Default export
export default FeatureComponent;  // ❌ Use named export
```

---

## Review Output Structure

```markdown
### 🔍 Review: [Feature/Component]

**Score**: [score] / 10

**Architecture Review**
✅ ComponentStore as Facade
✅ Component-scoped providers
✅ Service wrappers used
❌ Issue found: [description]

**Pattern Compliance**
✅ Module-based architecture
✅ Guards properly ordered
✅ DestroyedService cleanup
❌ Issue found: [description]

**TypeScript Standards**
✅ Named exports only
✅ Type safety
❌ Issue found: [description]

**Issues Found**
🔴 Critical: [issue] (fix: [solution])
  - Impact: [impact]
  - File: [path:line]

🟡 Important: [issue]
  - Suggestion: [solution]
  - File: [path:line]

🟢 Nice-to-have: [issue]
  - Enhancement: [solution]

**Recommendations**
1. [Priority 1 fix]
2. [Priority 2 improvement]
3. [Future enhancement]
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
Is Component calling API directly?
  → YES: 🔴 CRITICAL - Must use ComponentStore
  → NO: Continue

Is there a separate Facade class?
  → YES: 🔴 CRITICAL - Remove it, ComponentStore IS the Facade
  → NO: Continue

Is ComponentStore providedIn: 'root'?
  → YES: 🔴 CRITICAL - Should be component-scoped
  → NO: Continue

Using Ionic controllers directly?
  → YES: 🔴 CRITICAL - Use service wrappers
  → NO: Continue

LocalStorage keys missing _${env}?
  → YES: 🔴 CRITICAL - Add environment suffix
  → NO: Continue

Using default exports?
  → YES: 🔴 CRITICAL - Use named exports
  → NO: ✅ PASS
```
