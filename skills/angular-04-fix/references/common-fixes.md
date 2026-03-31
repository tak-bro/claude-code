# Angular Common Fix Patterns

## (Ionic) Add IonBasePage Extension
```typescript
// Before
@Component({ selector: 'feature-list-page' })
export class FeatureListPage {
    private destroyed$ = new ReplaySubject(1);  // Won't reset on view reuse!

    ngOnInit(): void {
        this.setupListeners();
    }
}

// After
import { IonBasePage } from '@shared/pages/ion-base/ion-base.page';

@Component({ selector: 'feature-list-page' })
export class FeatureListPage extends IonBasePage {
    ionViewWillEnter(): void {
        super.ionViewWillEnter();  // CRITICAL: Must call super first
        this.setupListeners();
        this.fetchData();
    }

    private setupListeners(): void {
        this.items$ = this.store.items$.pipe(takeUntil(this.destroyed$));
    }
}
```

## (Ionic) Add IonicRouteStrategy
```typescript
// Before (app.module.ts)
@NgModule({
    providers: [],
})
export class AppModule {}

// After
import { RouteReuseStrategy } from '@angular/router';
import { IonicRouteStrategy } from '@ionic/angular';

@NgModule({
    providers: [
        { provide: RouteReuseStrategy, useClass: IonicRouteStrategy },
    ],
})
export class AppModule {}
```

## Component-level destroyed$ (Non-Ionic)
```typescript
// For non-Ionic projects
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

## Standalone -> Module-based
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

## Add Message Pattern to Store
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

## Remove Store from Presentational Component
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

## Move Store to Module Providers
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

## (Ionic) Direct Controller -> Service Wrapper
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

## (Multi-env) Add Environment Suffix
```typescript
// Before
this.storage.set('user_theme', theme);
this.storage.get('user_theme');

// After
import { environment } from '@env/environment';

this.storage.set(`user_theme_${environment.env}`, theme);
this.storage.get(`user_theme_${environment.env}`);
```

## (TanStack Query) Add Query Keys Factory
```typescript
// Before
@Injectable({ providedIn: 'root' })
export class FeatureQueryService {
    useFeatures = () => {
        return injectQuery(() => ({
            queryKey: ['features'],  // Hard-coded key
            queryFn: () => this.api.fetchFeatures$(),
        }));
    };
}

// After
@Injectable({ providedIn: 'root' })
export class FeatureQueryService {
    featureKeys = this.queryService.createQueryKeys('feature');

    constructor(
        private readonly queryService: TanstackQueryService,
        private readonly queryClient: QueryClient,
        private readonly api: FeatureApiService
    ) {}

    useFeatures = () => {
        return injectQuery(() => ({
            queryKey: this.featureKeys.lists(),
            queryFn: () => this.api.fetchFeatures$(),
            staleTime: 1000 * 60 * 5,
        }));
    };
}
```
