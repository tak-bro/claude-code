# /angular:fix

Fix issues identified in review phase, prioritized by severity.

**Agents:** angular-componentstore-expert, tak-typescript-expert

---

## Pre-Fix

**From /review output, load:**
1. Fix Checklist (Critical -> Important -> Nice-to-have)
2. Specific file:line references
3. Fix code snippets provided

---

## Boundaries

### Always Do
- Fix ALL Critical issues (no exceptions)
- Run verification after each fix
- Mark completed items in checklist
- Keep fixes minimal and focused

### Ask First
- Skip Important issues
- Refactor beyond fix scope
- Add new features while fixing

### Never Do
- Skip Critical issues
- Change logic beyond the fix
- Introduce new patterns
- Leave verification failing

---

## Fix Priority

```
Critical (MUST fix - no exceptions)
    |
    |-- Standalone -> Module-based
    |-- Component -> API direct -> use ComponentStore
    |-- export default -> named export
    |-- DestroyedService -> component-level destroyed$
    |-- Business logic in Component -> move to Page/Store
    |-- Missing message pattern -> add to state
    |-- (Ionic) Direct controller -> service wrapper
    |
    v
Important (SHOULD fix)
    |
    |-- Missing destroyed$ cleanup
    |-- Page/Component separation
    |-- Store not in Module providers
    |-- Wrong Guard order
    |-- (Multi-env) Missing _${env} suffix
    |
    v
Nice-to-have (OPTIONAL)
    |
    |-- Selector optimization, naming
```

---

## Common Fixes

**DestroyedService -> Component-level destroyed$**
```typescript
// Before
import { DestroyedService } from '@core/services';

@Component({
    providers: [DestroyedService]
})
export class FeaturePage {
    constructor(private destroyed$: DestroyedService) {}
}

// After
import { ReplaySubject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

@Component({})
export class FeaturePage implements OnDestroy {
    private destroyed$: ReplaySubject<boolean> = new ReplaySubject(1);

    ngOnDestroy(): void {
        this.destroyed$.next(true);
        this.destroyed$.complete();
    }

    private setupListener(): void {
        this.data$.pipe(takeUntil(this.destroyed$)).subscribe(...);
    }
}
```

**Standalone -> Module-based**
```typescript
// Before
@Component({
    standalone: true,
    imports: [CommonModule]
})
export class FeatureComponent {}

// After
@Component({
    selector: 'app-feature'
})
export class FeatureComponent {}

// In module
@NgModule({
    declarations: [FeatureComponent],
    imports: [CommonModule, SharedModule]
})
export class FeatureModule {}
```

**Add Message Pattern to Store**
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

const DEFAULT_STATE: FeatureState = {
    message: { type: 'Default' },
    isFetching: false,
    items: [],
};

// Add selectors
readonly message$ = this.select(({ message }) => message);
readonly isFetching$ = this.select(({ isFetching }) => isFetching);
```

**Remove Store from Presentational Component**
```typescript
// Before
@Component({ selector: 'feature-list' })
export class FeatureListComponent {
    constructor(private store: FeatureStore) {}

    onClick(id: string): void {
        this.store.selectItem(id);
    }
}

// After
@Component({ selector: 'feature-list' })
export class FeatureListComponent {
    @Input() items: FeatureView[] = [];
    @Output() selectItem = new EventEmitter<string>();

    onClick(id: string): void {
        this.selectItem.emit(id);
    }
}

// In Page template
<feature-list [items]="items$ | async" (selectItem)="onSelectItem($event)"></feature-list>
```

**Move Store to Module Providers**
```typescript
// Before - component-scoped (incorrect for this project)
@Component({
    providers: [FeatureStore]
})
export class FeaturePage {}

// After - module-scoped (correct)
@NgModule({
    providers: [FeatureStore]
})
export class FeatureModule {}

@Component({})  // No providers here
export class FeaturePage {
    constructor(private store: FeatureStore) {}
}
```

**(Ionic) Direct Controller -> Service Wrapper**
```typescript
// Before
constructor(private modalCtrl: ModalController) {}

async showModal(): Promise<void> {
    const modal = await this.modalCtrl.create({
        component: MyModalComponent
    });
    await modal.present();
}

// After
constructor(private modalService: ModalService) {}

async showModal(): Promise<void> {
    await this.modalService.present(MyModalComponent);
}
```

**(Multi-env) Add Environment Suffix**
```typescript
// Before
this.storage.set('user_theme', theme);
this.storage.get('user_theme');

// After
import { environment } from '@env/environment';

this.storage.set(`user_theme_${environment.env}`, theme);
this.storage.get(`user_theme_${environment.env}`);
```

---

## Verification Commands

```bash
# {pm} = npm, yarn, pnpm, bun (use project's package manager)
# After each fix
ng build
{pm} run lint

# After all fixes
ng test --watch=false
ng build --configuration=production
```

---

## Output

```markdown
### Fixed: [Feature]

**Fixes Applied:**
- [x] C1: Replaced DestroyedService with destroyed$ (list.page.ts)
- [x] C2: Removed store from presentational component (list.component.ts)
- [x] I1: Added message pattern to store (feature.store.ts)

**Verification:**
- [x] `ng build`
- [x] `{pm} run lint`
- [x] `ng test`
- [x] `ng build --configuration=production`

**Skipped (with reason):**
- [ ] N1: [Reason for skip]

**Final Score:** X/10 -> 10/10
```

---

## Checklist

- [ ] ALL Critical issues fixed
- [ ] Important issues addressed (or justified skip)
- [ ] `ng build` passes
- [ ] `{pm} run lint` passes
- [ ] `ng test` passes
- [ ] `ng build --configuration=production` passes
- [ ] Fix report generated
