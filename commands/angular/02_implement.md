# /angular:implement

Implement Angular features using NgRx ComponentStore pattern with Module-based architecture.

**Agents:** angular-componentstore-expert, tak-typescript-expert, framework-docs-researcher

---

## Tech Stack

```
@ngrx/store                        # Global state (router, app-wide)
@ngrx/component-store              # Feature-level state (primary)
@ngrx/effects                      # Side effects (router only)
@ngrx/entity                       # Entity collections
@ngrx/operators                    # tapResponse, etc.
@tanstack/angular-query-experimental  # Server state (optional)
```

---

## Pre-Implementation

**Before coding, verify from /plan output:**
1. Plan approved?
2. Files to create/modify identified?
3. Implementation Checklist ready?

---

## Boundaries

### Always Do
- Use plan's Implementation Checklist
- Component-level `destroyed$` for cleanup (NOT DestroyedService)
- Page (smart) vs Component (presentational) separation
- Message pattern in ComponentStore state
- Store provided in Module's providers array
- Named exports only (`export const`, `export class`)
- Run verification commands before done

### Ask First
- Adding new dependencies
- Modifying shared services
- Creating new patterns not in codebase
- Using `providedIn: 'root'` for ComponentStore

### Never Do
- `export default`
- Standalone components (Module-based only)
- Component -> API direct calls (must go through ComponentStore)
- Business logic in presentational components
- Missing message pattern in store state
- (Ionic) Direct controller usage without service wrapper
- (Ionic) Missing `extends IonBasePage` for pages
- (Ionic) Missing `super.ionViewWillEnter()` call
- (Multi-env) LocalStorage keys without `_${env}` suffix

---

## Architecture

```
Page (Smart Component - orchestrates stores)
    |
ComponentStore (State + Business Logic + Effects)
    |-- Selectors (derived state)
    |-- Updaters (sync mutations)
    |-- Effects (async operations)
        |
API Service (HTTP only, Observable<T>)
    |
Backend

Component (Presentational - @Input/@Output only)
```

**Core Module (app-wide state):**
```
Manager (orchestrates state + API)
    |
State (BehaviorSubject holder)
    |
API Service
```

---

## Folder Structure

```
src/app/modules/
  [feature]/
    [feature].module.ts
    [feature]-routing.module.ts
    pages/           # Smart components
    components/      # Presentational components
    services/        # API services
    stores/          # ComponentStore instances
    types/           # TypeScript interfaces
    pipes/           # Feature-specific pipes
```

---

## Patterns

**ComponentStore (with Message Pattern)**
```typescript
// types/feature.type.ts
export type FeatureStoreMessage = { type: FeatureStoreMessageType; data?: any };

// stores/feature.store.ts
export interface FeatureState {
    message: FeatureStoreMessage;
    isFetching: boolean;
    items: FeatureView[];
    selectedItem: FeatureView | null;
}

const DEFAULT_STATE: FeatureState = {
    message: { type: 'Default' },
    isFetching: false,
    items: [],
    selectedItem: null,
};

@Injectable()  // Module provides it
export class FeatureStore extends ComponentStore<FeatureState> {
    // Selectors
    readonly message$ = this.select(({ message }) => message);
    readonly isFetching$ = this.select(({ isFetching }) => isFetching);
    readonly items$ = this.select(({ items }) => items);

    constructor(private api: FeatureApiService) {
        super(DEFAULT_STATE);
    }

    // Effects
    fetchItems$ = this.effect<void>(trigger$ =>
        trigger$.pipe(
            tap(() => this.patchState({ isFetching: true })),
            switchMap(() => this.api.fetchItems$().pipe(
                tap(items => this.patchState({ items, message: { type: 'SuccessToFetchItems' } })),
                finalize(() => this.patchState({ isFetching: false })),
                catchError(error => {
                    this.patchState({ message: { type: 'FailedToRequest', data: { errorMessage: error.message } } });
                    return EMPTY;
                })
            ))
        )
    );
}
```

**Page (Smart Component)**
```typescript
@Component({
    selector: 'feature-list-page',
    templateUrl: './list.page.html',
})
export class FeatureListPage implements OnInit, OnDestroy {
    private destroyed$: ReplaySubject<boolean> = new ReplaySubject(1);

    readonly items$ = this.store.items$;
    readonly isFetching$ = this.store.isFetching$;

    constructor(
        private readonly store: FeatureStore,
        private readonly toastService: ToastService
    ) {}

    ngOnInit(): void {
        this.setupStoreListener();
        this.store.fetchItems$();
    }

    ngOnDestroy(): void {
        this.destroyed$.next(true);
        this.destroyed$.complete();
    }

    private setupStoreListener(): void {
        this.store.message$
            .pipe(takeUntil(this.destroyed$))
            .subscribe(message => {
                if (message.type === 'FailedToRequest') {
                    this.toastService.showToast(message.data?.errorMessage);
                }
            });
    }
}
```

**Component (Presentational)**
```typescript
@Component({
    selector: 'feature-list',
})
export class FeatureListComponent {
    @Input() items: FeatureView[] = [];
    @Output() selectItem = new EventEmitter<string>();
}
```

**Module**
```typescript
const COMPONENTS = [FeatureListComponent, FeatureDetailComponent];
const PAGES = [FeatureListPage, FeatureDetailPage];
const SERVICES = [FeatureApiService];
const STORES = [FeatureStore];

@NgModule({
    declarations: [...COMPONENTS, ...PAGES],
    imports: [CommonModule, SharedModule, FeatureRoutingModule],
    providers: [...SERVICES, ...STORES],
})
export class FeatureModule {}
```

---

## Ionic Patterns

**IonBasePage (CRITICAL for Ionic projects)**
Ionic views are cached and reused. destroyed$ must be **reset on every view enter**:

```typescript
// shared/pages/ion-base/ion-base.page.ts
@Component({ selector: 'app-ion-base-page', template: `` })
export class IonBasePage {
    destroyed$: ReplaySubject<boolean>;

    ionViewWillEnter(): void {
        this.resetDestroyedSubject();  // CRITICAL: Reset on every enter
    }

    ionViewDidLeave(): void {
        this.destroyed$.next(true);
        this.destroyed$.complete();
    }

    private resetDestroyedSubject(): void {
        this.destroyed$ = new ReplaySubject(1);
    }
}
```

**Feature Page (Ionic)**
```typescript
export class FeatureListPage extends IonBasePage {
    ionViewWillEnter(): void {
        super.ionViewWillEnter();  // MUST call super first
        this.setupListeners();
        this.fetchData();
    }

    private setupListeners(): void {
        this.items$ = this.store.items$.pipe(takeUntil(this.destroyed$));
    }
}
```

**IonicRouteStrategy (app.module.ts)**
```typescript
import { RouteReuseStrategy } from '@angular/router';
import { IonicRouteStrategy } from '@ionic/angular';

@NgModule({
    providers: [
        { provide: RouteReuseStrategy, useClass: IonicRouteStrategy },
    ],
})
export class AppModule {}
```

---

## TanStack Query Pattern (Optional)

For server state management alongside ComponentStore:

```typescript
// services/queries/feature-query.service.ts
@Injectable({ providedIn: 'root' })
export class FeatureQueryService {
    featureKeys = this.queryService.createQueryKeys('feature');

    constructor(
        private readonly queryService: TanstackQueryService,
        private readonly queryClient: QueryClient,
        private readonly featureApiService: FeatureApiService
    ) {}

    useFeatures = () => {
        return injectQuery(() => ({
            queryKey: this.featureKeys.lists(),
            queryFn: () => this.featureApiService.fetchFeatures$(),
            staleTime: 1000 * 60 * 5,
        }));
    };

    useInfiniteFeatures = (params: Signal<Params>) => {
        return injectInfiniteQuery(() => ({
            queryKey: this.featureKeys.list(params()),
            queryFn: ({ pageParam }) =>
                this.featureApiService.fetchFeatures$({ ...params(), page: pageParam || 0 }),
            initialPageParam: 0,
            getNextPageParam: (lastPage, allPages) => {
                const loadedCount = allPages.reduce((acc, page) => acc + page.list.length, 0);
                return loadedCount < lastPage.total ? allPages.length : undefined;
            },
        }));
    };
}
```

---

## Verification Commands

```bash
# {pm} = npm, yarn, pnpm, bun (use project's package manager)
ng build              # if available
{pm} run lint         # if available
ng test --watch=false # if available
```

---

## Checklist

- [ ] Module-based (NOT standalone)
- [ ] Component-level `destroyed$` cleanup
- [ ] Page (smart) vs Component (presentational) separation
- [ ] Message pattern in store state
- [ ] Store provided in Module's providers
- [ ] Component -> ComponentStore -> API only
- [ ] Named exports ONLY
- [ ] API methods end with `$` suffix
- [ ] Guards: App -> Auth -> Feature
- [ ] (Ionic) Pages extend IonBasePage
- [ ] (Ionic) super.ionViewWillEnter() called first
- [ ] (Ionic) IonicRouteStrategy configured
- [ ] (Ionic) Service wrappers for controllers
- [ ] (Multi-env) LocalStorage keys with `_${env}` suffix
- [ ] (TanStack Query) Query keys factory pattern
- [ ] Verification commands pass

---

## Output (-> review)

```markdown
### Implemented: [Feature]

**Files Created/Modified:**
- `src/app/modules/feature/stores/feature.store.ts`
- `src/app/modules/feature/pages/list/list.page.ts`
- `src/app/modules/feature/components/list/list.component.ts`
- `src/app/modules/feature/feature.module.ts`

**Checklist Completed:**
- [x] Module-based architecture
- [x] Component-level destroyed$
- [x] Page/Component separation
- [x] Message pattern in store

**Verification:** (if available)
- [x] `ng build`
- [x] `{pm} run lint`
- [x] `ng test`

**Notes for Review:**
- [Implementation decisions]
- [Areas needing attention]
```
