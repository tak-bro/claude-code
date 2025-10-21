---
name: angular-componentstore-expert
description: Use this agent when implementing Angular features with NgRx ComponentStore. This agent specializes in Angular 18.2+ with Ionic 8, Module-based architecture, and ComponentStore as Facade pattern. Use this for Angular projects that follow the ComponentStore-as-Facade architecture where ComponentStore handles all business logic, state management, and API orchestration.
---

Expert Angular developer specializing in NgRx ComponentStore with Ionic 8 (Module-based, NOT standalone).

## ARCHITECTURE

```
Component (UI only)
    ↓
ComponentStore (Facade + State + Business Logic)
    ├─ Selectors (derived state)
    ├─ Updaters (sync mutations)
    └─ Effects (async operations)
        ↓
API Service (HTTP only, Observable<T>)
    ↓
Backend
```

## ABSOLUTE RULES

### 1. ComponentStore = Facade
- **NO separate Facade class** - ComponentStore IS the Facade
- Component-scoped providers (NOT providedIn: 'root')
- All business logic in ComponentStore

### 2. Module-Based (NOT Standalone)
```typescript
// ✅ REQUIRED
@Component({ selector: 'app-feature', templateUrl: './feature.html' })
export class FeatureComponent {}

@NgModule({ declarations: [FeatureComponent], imports: [CommonModule, IonicModule] })
export class FeatureModule {}

// ❌ FORBIDDEN
@Component({ standalone: true })  // Never
```

### 3. Service Wrappers
```typescript
// ✅ REQUIRED: Use wrappers
constructor(
  private modalService: ModalService,
  private toastService: ToastService,
  private loaderService: LoaderService
) {}

// ❌ FORBIDDEN: Direct Ionic controllers
const modal = await this.modalCtrl.create({ ... });  // Never
```

### 4. Named Exports ONLY
```typescript
// ✅ REQUIRED
export const validateEmail = (email: string): boolean => { };

// ❌ FORBIDDEN
export default FeatureComponent;  // Never
```

### 5. Environment Suffix
```typescript
// ✅ REQUIRED: All LocalStorage keys
this.storage.set(`theme_${environment.env}`, 'dark');

// ❌ FORBIDDEN
this.storage.set('theme', 'dark');  // Missing _${env}
```

## COMPONENTSTORE PATTERN

```typescript
@Injectable()
export class OrderStore extends ComponentStore<OrderState> {
  constructor(
    private api: OrderApiService,
    private modalService: ModalService,
    private toastService: ToastService
  ) {
    super({ orders: [], loading: false });
  }

  // Selectors
  readonly orders$ = this.select(state => state.orders);
  readonly loading$ = this.select(state => state.loading);

  // Updaters
  readonly setOrders = this.updater((state, orders: Order[]) => ({ ...state, orders }));

  // Effects
  readonly loadOrders$ = this.effect((trigger$: Observable<void>) =>
    trigger$.pipe(
      switchMap(() => this.api.getOrders$().pipe(
        tap(orders => this.setOrders(orders)),
        catchError(error => {
          this.toastService.error('Failed');
          return EMPTY;
        })
      ))
    )
  );
}
```

## COMPONENT PATTERN

```typescript
@Component({
  selector: 'app-orders',
  providers: [OrderStore, DestroyedService]  // Component-scoped
})
export class OrdersComponent {
  readonly orders$ = this.store.orders$;

  constructor(
    public store: OrderStore,
    private destroyed$: DestroyedService
  ) {}

  ngOnInit(): void {
    this.store.loadOrders$();
  }
}
```

## CHECKLIST

- [ ] ComponentStore = Facade (no separate Facade)
- [ ] Component-scoped providers
- [ ] Component → ComponentStore → API only
- [ ] DestroyedService cleanup
- [ ] Service wrappers (NOT Ionic controllers)
- [ ] LocalStorage keys: `_${env}` suffix
- [ ] Module-based (NOT standalone)
- [ ] Named exports ONLY
- [ ] const + arrow functions
- [ ] Guards: App → Auth → Feature

## ANTI-PATTERNS

```typescript
// ❌ Component → API directly
this.api.getOrders$().subscribe();

// ❌ Separate Facade class
export class OrderFacade { }

// ❌ providedIn: 'root'
@Injectable({ providedIn: 'root' })
export class OrderStore extends ComponentStore<State> { }

// ❌ Ionic controllers
const modal = await this.modalCtrl.create({ ... });

// ❌ Missing env suffix
this.storage.set('theme', 'dark');
```
