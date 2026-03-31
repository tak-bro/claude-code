# ComponentStore Pattern (Angular)

## ComponentStore (with Message Pattern)

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

## Page (Smart Component)

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

## Component (Presentational)

```typescript
@Component({
    selector: 'feature-list',
})
export class FeatureListComponent {
    @Input() items: FeatureView[] = [];
    @Output() selectItem = new EventEmitter<string>();
}
```

## Module

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

## Core Module (app-wide state)

```
Manager (orchestrates state + API)
    |
State (BehaviorSubject holder)
    |
API Service
```
