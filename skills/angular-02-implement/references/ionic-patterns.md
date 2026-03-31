# Ionic Patterns (Angular)

## IonBasePage (CRITICAL for Ionic projects)
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

## Feature Page (Ionic)

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

## IonicRouteStrategy (app.module.ts)

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

## Direct Controller → Service Wrapper

```typescript
// Before ([BAD])
constructor(private modalCtrl: ModalController) {}

async showModal(): Promise<void> {
    const modal = await this.modalCtrl.create({
        component: MyModalComponent
    });
    await modal.present();
}

// After ([GOOD])
constructor(private modalService: ModalService) {}

async showModal(): Promise<void> {
    await this.modalService.present(MyModalComponent);
}
```

## Multi-Environment LocalStorage

```typescript
// Before ([BAD])
this.storage.set('user_theme', theme);
this.storage.get('user_theme');

// After ([GOOD])
import { environment } from '@env/environment';

this.storage.set(`user_theme_${environment.env}`, theme);
this.storage.get(`user_theme_${environment.env}`);
```

## Component-level destroyed$ (Non-Ionic)

```typescript
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

## Standalone → Module-based

```typescript
// Before ([BAD])
@Component({
    standalone: true,
    imports: [CommonModule]
})
export class FeatureComponent {}

// After ([GOOD])
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
