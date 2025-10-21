# /angular:implement

## Purpose
Implement Angular features using ComponentStore-as-Facade pattern, Module-based architecture, and Ionic 8 components.

## Agents
- **angular-componentstore-expert**: ComponentStore patterns and Angular best practices
- **framework-docs-researcher**: Angular/Ionic framework reference
- **tak-typescript-expert**: TypeScript implementation checklist

---

## Workflow

### Phase 1: Setup (3 min)
```bash
1. Read requirements/plan
2. Verify Module-based architecture (NOT standalone)
3. Identify ComponentStore scope (component vs module)
4. Plan service wrappers needed (Modal, Toast, Loader)
```

### Phase 2: Parallel Implementation 🔀 (20 min)
```bash
# If components are independent
Lane 1: ComponentStore (Facade + State + Effects)
Lane 2: API Service (HTTP calls only)
Lane 3: Component (UI only, providers: [Store, DestroyedService])
Lane 4: Service Wrappers (if needed)
```

### Phase 3: Integration (5 min)
```bash
4. Connect Component → ComponentStore → API
5. Verify DestroyedService subscription cleanup
6. Test Guards (App → Auth → Feature)
7. Verify LocalStorage keys have _${env} suffix
```

### Phase 4: Validation (2 min)
```bash
8. Run lint/tests
9. Verify NO default exports
10. Verify NO direct Ionic controller usage
11. Verify ComponentStore is component-scoped
```

---

## Critical Patterns

### ComponentStore Structure
```typescript
@Injectable()
export class FeatureStore extends ComponentStore<FeatureState> {
  constructor(
    private api: FeatureApiService,
    private modalService: ModalService,
    private toastService: ToastService
  ) {
    super(initialState);
  }

  // 1. Selectors
  readonly items$ = this.select(state => state.items);

  // 2. Updaters
  readonly setItems = this.updater((state, items: Item[]) => ({
    ...state,
    items
  }));

  // 3. Effects
  readonly loadItems$ = this.effect((trigger$: Observable<void>) =>
    trigger$.pipe(
      switchMap(() => this.api.getItems$().pipe(
        tap(items => this.setItems(items)),
        catchError(error => {
          this.toastService.error('Failed');
          return EMPTY;
        })
      ))
    )
  );
}
```

### Component Pattern
```typescript
@Component({
  selector: 'app-feature',
  templateUrl: './feature.component.html',
  providers: [FeatureStore, DestroyedService]  // Component-scoped
})
export class FeatureComponent implements OnInit {
  readonly items$ = this.store.items$;

  constructor(
    public store: FeatureStore,
    private destroyed$: DestroyedService
  ) {}

  ngOnInit(): void {
    this.store.loadItems$();
  }
}
```

### API Service Pattern
```typescript
@Injectable({ providedIn: 'root' })
export class FeatureApiService {
  constructor(private endpointService: EndpointService) {}

  getItems$(): Observable<Item[]> {
    return from(
      this.endpointService
        .getApi('feature')
        .get<ListResponse<Item>>('/items')
    ).pipe(
      map(response => response.data.items),
      catchError(this.handleError)
    );
  }
}
```

---

## Anti-Patterns to Avoid

```typescript
// ❌ Component calling API directly
this.api.getItems$().subscribe();

// ❌ Separate Facade class
export class FeatureFacade { }

// ❌ ComponentStore providedIn: 'root'
@Injectable({ providedIn: 'root' })
export class FeatureStore extends ComponentStore<State> { }

// ❌ Using Ionic controllers directly
const modal = await this.modalCtrl.create({ ... });

// ❌ Missing environment suffix
this.storage.set('theme', 'dark');

// ❌ Standalone components
@Component({ standalone: true })

// ❌ Default exports
export default FeatureComponent;
```

---

## Checklist Before Submit

- [ ] Component depends ONLY on ComponentStore (no direct API)
- [ ] ComponentStore = Facade (no separate Facade class)
- [ ] ComponentStore provided at component level
- [ ] DestroyedService used for cleanup
- [ ] LocalStorage keys have `_${env}` suffix
- [ ] Service wrappers used (NOT Ionic controllers)
- [ ] API service returns Observable<T>
- [ ] Module-based (NOT standalone)
- [ ] Named exports ONLY
- [ ] const + arrow functions
- [ ] All null/undefined handled

---

## Output Structure

```markdown
### ✅ Implementation: [Feature]

**Architecture**
- ComponentStore: [FeatureStore] (component-scoped)
- API Service: [FeatureApiService]
- Component: [FeatureComponent]
- Module: [FeatureModule]

**Files**
- ✨ New: [paths]
- 🔧 Modified: [paths]

**Patterns Applied**
✓ ComponentStore as Facade
✓ Module-based architecture
✓ Service wrappers
✓ DestroyedService cleanup
✓ Environment-suffixed keys

**Tests**
✓ [test] (passed)
```
