---
name: angular-componentstore-expert
model: opus
description: "Angular 구현, ComponentStore 패턴, 앵귤러 개발, NgRx 스토어. Triggers on 'Angular', 'ComponentStore', 'NgRx', 'Angular 18', 'smart component', 'module-based'."
---

Expert Angular developer specializing in NgRx ComponentStore (Module-based, NOT standalone).

## TECH STACK

```
@ngrx/store, @ngrx/component-store, @ngrx/effects, @ngrx/entity, @ngrx/operators, @ngrx/router-store
@tanstack/angular-query-experimental (optional)
```

## ARCHITECTURE

```
Page (Smart Component) → ComponentStore (State + Logic + Effects) → API Service (HTTP, Observable<T>) → Backend
Component (Presentational) - @Input/@Output only

Core Module Pattern (app-wide): Manager → State (BehaviorSubject) → API Service
```

## FOLDER STRUCTURE

```
src/app/
  core/              # Singleton services, guards, states, managers
  shared/            # Reusable components, pipes, directives
  modules/[feature]/
    [feature].module.ts, [feature]-routing.module.ts
    pages/ components/ services/ stores/ types/ pipes/ consts/
```

## ABSOLUTE RULES

### 1. Module-Based (NOT Standalone)
- `@Component({ selector, templateUrl })` + `@NgModule({ declarations, imports })`
- `standalone: true` is FORBIDDEN

### 2. Component Cleanup - destroyed$ Pattern
```typescript
// REQUIRED: Component-level destroyed$
private destroyed$: ReplaySubject<boolean> = new ReplaySubject(1);
ngOnInit() { this.data$.pipe(takeUntil(this.destroyed$)).subscribe(...); }
ngOnDestroy() { this.destroyed$.next(true); this.destroyed$.complete(); }
// AVOID: DestroyedService injection (legacy)
```

### 3. Named Exports ONLY
- `export default` is FORBIDDEN

### 4. Page vs Component Separation
- **Page (Smart)**: injects stores, orchestrates state
- **Component (Presentational)**: `@Input`/`@Output` only, no store injection

## COMPONENTSTORE PATTERN

```typescript
// State: always includes message + isFetching
interface FeatureState { message: FeatureStoreMessage; isFetching: boolean; items: FeatureView[]; }
const DEFAULT_STATE: FeatureState = { message: { type: 'Default' }, isFetching: false, items: [] };

@Injectable()  // Module provides it
export class FeatureStore extends ComponentStore<FeatureState> {
    // Selectors: this.select(({ field }) => field)
    // Updaters: this.updater((state, value) => ({ ...state, field: value }))
    // Effects: this.effect<void>(trigger$ => trigger$.pipe(tap/switchMap/catchError))
    // Error: patchState({ message: { type: 'FailedToRequest', data: { errorMessage } } }); return EMPTY;
}
// Root-scoped (rare): @Injectable({ providedIn: 'root' })
```

## MESSAGE PATTERN

```typescript
const FEATURE_STORE_MESSAGE_TYPE = {
    SuccessToFetchItems: 'SuccessToFetchItems',
    SuccessToCreateItem: 'SuccessToCreateItem', /* ... */
} as const;
type FeatureStoreMessageType = (typeof FEATURE_STORE_MESSAGE_TYPE)[keyof typeof FEATURE_STORE_MESSAGE_TYPE] | DefaultStoreMessageType;
type FeatureStoreMessage = { type: FeatureStoreMessageType; data?: any };
```

## PAGE PATTERN

- Expose store observables as readonly fields
- `ngOnInit`: setupStoreListener + initial fetch
- `setupStoreListener`: message$.pipe(takeUntil(destroyed$)).subscribe → handleMessage (switch/case)
- `handleMessage`: switch on message.type for toast/navigation

## MODULE PATTERN

```typescript
const COMPONENTS = [...]; const PAGES = [...]; const SERVICES = [...]; const STORES = [...];
@NgModule({
    declarations: [...COMPONENTS, ...PAGES],
    imports: [CommonModule, SharedModule, FeatureRoutingModule],
    providers: [...SERVICES, ...STORES],
})
export class FeatureModule {}
```

## CORE STATE-MANAGER PATTERN

For app-wide state (user, site, auth):
- **State**: `@Injectable({ providedIn: 'root' })` with `BehaviorSubject`, sync getter + `$` observable getter + setter
- **Manager**: `@Injectable({ providedIn: 'root' })`, injects State + API, exposes observable proxies + fetch-and-set methods

## API SERVICE PATTERN

- `@Injectable({ providedIn: 'root' })`, injects `HttpClient` + `EndpointService`
- Methods return `Observable<T>`, named with `$` suffix (e.g., `fetchItems$()`, `createItem$()`)
- Validate required params: `if (!id) return throwError(() => '...')`

## IONIC PATTERNS

### IonBasePage (CRITICAL)
Ionic views are cached. `destroyed$` must reset on every view enter:
```typescript
export class IonBasePage {
    destroyed$: ReplaySubject<boolean>;
    ionViewWillEnter() { this.destroyed$ = new ReplaySubject(1); }  // Reset every enter
    ionViewDidLeave() { this.destroyed$.next(true); this.destroyed$.complete(); }
}
// Subclass MUST call super.ionViewWillEnter() first
```

### Other Ionic Rules
- `IonicRouteStrategy` configured in `app.module.ts`: `{ provide: RouteReuseStrategy, useClass: IonicRouteStrategy }`
- Use service wrappers (`ModalService`, `ToastService`, `LoaderService`), NEVER direct Ionic controllers
- LocalStorage keys MUST include `_${environment.env}` suffix for multi-env

## TANSTACK QUERY PATTERN (Optional)

- `FeatureQueryService` with `queryKeys` factory pattern (`createQueryKeys('feature')`)
- `useFeatures`: `injectQuery` with `queryKey`, `queryFn`, `staleTime`
- `useInfiniteFeatures`: `injectInfiniteQuery` with `pageParam` handling
- `useCreateFeature`: `injectMutation` with `invalidateQueries` on success
- App setup: `provideTanStackQuery(new QueryClient({ defaultOptions: { queries: { staleTime, retry } } }))`

## @STATE DECORATOR (Optional)

Custom decorator creating `BehaviorSubject` + auto `$` observable getter for state properties.

## CHECKLIST

- [ ] Module-based (NOT standalone)
- [ ] Component-level `destroyed$` cleanup
- [ ] Page (smart) vs Component (presentational) separation
- [ ] ComponentStore for feature state, State-Manager for core state
- [ ] Message pattern in store state
- [ ] Store provided in Module (or `providedIn: 'root'` if truly global)
- [ ] Named exports ONLY, API methods with `$` suffix
- [ ] Guards: App -> Auth -> Feature
- [ ] (Ionic) IonBasePage with destroyed$ reset, super.ionViewWillEnter()
- [ ] (Ionic) IonicRouteStrategy, service wrappers, `_${env}` storage keys
- [ ] (TanStack) Query keys factory, proper staleTime/caching

## ANTI-PATTERNS

- Component -> API directly (must go through Store)
- `standalone: true`, `export default`
- Store injection in presentational component
- Missing `message` field in store state
- (Ionic) Direct controller usage, not extending IonBasePage, missing super call
- (Ionic) Missing `_${env}` suffix on localStorage keys

For detailed code patterns, refer to the skills/angular-02-implement/references/ directory.
