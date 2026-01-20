# /angular:implement

Implement Angular features using ComponentStore-as-Facade pattern.

**Agents:** angular-componentstore-expert, tak-typescript-expert, framework-docs-researcher

---

## Pre-Implementation

**Before coding, verify from /plan output:**
1. ✅ Plan approved?
2. ✅ Files to create/modify identified?
3. ✅ Implementation Checklist ready?

---

## Boundaries

### ✅ Always Do
- Use plan's Implementation Checklist
- ComponentStore = Facade (no separate Facade class)
- Component-scoped providers (NOT providedIn: 'root')
- Service wrappers for Ionic controllers
- LocalStorage keys with `_${env}` suffix
- Named exports only (`export const`)
- Run verification commands before done

### ⚠️ Ask First
- Adding new dependencies
- Modifying shared services
- Creating new patterns not in codebase

### 🚫 Never Do
- `export default`
- Standalone components (Module-based only)
- Direct Ionic controller usage (`modalCtrl.create()`)
- `providedIn: 'root'` for ComponentStore
- Component → API direct calls (must go through ComponentStore)

---

## Workflow

1. **Setup (3min)**: Verify Module-based, identify scope, plan wrappers
2. **Parallel (20min)**: ComponentStore | API Service | Component | Module
3. **Integration (5min)**: Connect layers, verify DestroyedService, test Guards
4. **Validation (2min)**: Run verification commands, check env suffixes

---

## Architecture

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

---

## Patterns

**ComponentStore** (Facade + State)
```typescript
@Injectable()
export class OrderStore extends ComponentStore<OrderState> {
  constructor(
    private api: OrderApiService,
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

**Component** (UI only, component-scoped)
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

**Module** (NOT standalone)
```typescript
@NgModule({
  declarations: [OrdersComponent],
  imports: [CommonModule, IonicModule],
  exports: [OrdersComponent]
})
export class OrdersModule {}
```

---

## Verification Commands

```bash
# {pm} = npm, yarn, pnpm, bun (use project's package manager)
ng build                       # No build errors
{pm} run lint                  # No lint errors
ng test --watch=false          # Tests pass
```

---

## Checklist

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
- [ ] Verification commands pass

---

## Output (→ review 단계 입력)

```markdown
### ✅ Implemented: [Feature]

**Files Created/Modified:**
- ✨ `src/app/features/orders/orders.store.ts`
- ✨ `src/app/features/orders/orders.component.ts`
- ✨ `src/app/features/orders/orders.module.ts`
- 🔧 `src/app/services/orders-api.service.ts`

**Checklist Completed:**
- [x] ComponentStore = Facade
- [x] Component-scoped providers
- [x] Service wrappers used

**Verification:**
- [x] `ng build` ✓
- [x] `{pm} run lint` ✓
- [x] `ng test` ✓

**Notes for Review:**
- [Implementation decisions]
- [Areas needing attention]
```
