# /angular:implement

Implement Angular features using ComponentStore-as-Facade pattern.

**Agents:** angular-componentstore-expert, framework-docs-researcher, tak-typescript-expert

---

## Workflow

1. Setup (3min): Verify Module-based, identify ComponentStore scope, plan wrappers
2. **Parallel** (20min): ComponentStore | API Service | Component | Wrappers
3. Integration (5min): Connect layers, verify DestroyedService, test Guards, check env suffixes
4. Validation (2min): Lint, verify NO defaults, NO Ionic controllers, component-scoped

---

## Critical Patterns

**ComponentStore** (Facade + State)
```typescript
@Injectable()
export class OrderStore extends ComponentStore<OrderState> {
  readonly orders$ = this.select(state => state.orders);
  readonly setOrders = this.updater((state, orders) => ({ ...state, orders }));
  readonly loadOrders$ = this.effect((trigger$) => trigger$.pipe(...));
}
```

**Component** (UI only, component-scoped)
```typescript
@Component({ selector: 'app-orders', providers: [OrderStore, DestroyedService] })
export class OrdersComponent {
  readonly orders$ = this.store.orders$;
  constructor(public store: OrderStore, private destroyed$: DestroyedService) {}
  ngOnInit() { this.store.loadOrders$(); }
}
```

---

## Checklist

- [ ] ComponentStore = Facade (no separate Facade)
- [ ] Component-scoped providers
- [ ] Component → ComponentStore → API only
- [ ] Service wrappers (NOT Ionic controllers)
- [ ] LocalStorage: `_${env}` suffix
- [ ] Module-based (NOT standalone)
- [ ] Named exports ONLY

---

## Output

```markdown
### ✅ Implementation: [Feature]

**Architecture**
ComponentStore: [Store] | API: [Service] | Component: [Component]

**Patterns**
✅ ComponentStore as Facade | Service wrappers | DestroyedService | Env-suffixed keys
```
