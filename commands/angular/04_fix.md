# /angular:fix

Fix issues identified in review phase, prioritized by severity.

**Agents:** angular-componentstore-expert, tak-typescript-expert

---

## Pre-Fix

**From /review output, load:**
1. Fix Checklist (Critical → Important → Nice-to-have)
2. Specific file:line references
3. Fix code snippets provided

---

## Boundaries

### ✅ Always Do
- Fix ALL 🔴 Critical issues (no exceptions)
- Run verification after each fix
- Mark completed items in checklist
- Keep fixes minimal and focused

### ⚠️ Ask First
- Skip 🟡 Important issues
- Refactor beyond fix scope
- Add new features while fixing

### 🚫 Never Do
- Skip 🔴 Critical issues
- Change logic beyond the fix
- Introduce new patterns
- Leave verification failing

---

## Workflow

1. **Load (1min)**: Get Fix Checklist from review
2. **Fix Critical (varies)**: Fix ALL 🔴 issues, verify each
3. **Fix Important (varies)**: Fix 🟡 issues if time permits
4. **Verify (2min)**: Run all verification commands
5. **Report (1min)**: Generate completion report

---

## Fix Priority

```
🔴 Critical (MUST fix - no exceptions)
    │
    ├─ Separate Facade → merge into ComponentStore
    ├─ providedIn: 'root' → component-scoped
    ├─ Standalone → Module-based
    ├─ Direct Ionic controller → service wrapper
    └─ export default → named export
    │
    ▼
🟡 Important (SHOULD fix)
    │
    ├─ Missing _${env} suffix
    ├─ Missing DestroyedService
    ├─ Wrong Guard order
    └─ Business logic in component
    │
    ▼
🟢 Nice-to-have (OPTIONAL)
    │
    └─ Selector optimization, naming
```

---

## Common Fixes

**Remove Separate Facade**
```typescript
// Delete: orders.facade.ts

// Move all facade methods to ComponentStore
@Injectable()
export class OrderStore extends ComponentStore<OrderState> {
  // Methods that were in facade now live here
  readonly loadOrders$ = this.effect(...);
}
```

**providedIn: 'root' → Component-scoped**
```typescript
// Before (store)
@Injectable({ providedIn: 'root' })
export class OrderStore extends ComponentStore<OrderState> {}

// After (store)
@Injectable()
export class OrderStore extends ComponentStore<OrderState> {}

// After (component)
@Component({
  selector: 'app-orders',
  providers: [OrderStore, DestroyedService]
})
```

**Standalone → Module-based**
```typescript
// Before
@Component({
  standalone: true,
  imports: [CommonModule, IonicModule]
})
export class OrdersComponent {}

// After
@Component({
  selector: 'app-orders'
})
export class OrdersComponent {}

@NgModule({
  declarations: [OrdersComponent],
  imports: [CommonModule, IonicModule]
})
export class OrdersModule {}
```

**Direct Ionic Controller → Service Wrapper**
```typescript
// Before
constructor(private modalCtrl: ModalController) {}

async showModal() {
  const modal = await this.modalCtrl.create({ component: MyModal });
  await modal.present();
}

// After
constructor(private modalService: ModalService) {}

async showModal() {
  await this.modalService.present(MyModal);
}
```

**Add _${env} Suffix**
```typescript
// Before
this.storage.set('user_theme', theme);

// After
import { environment } from '@env/environment';
this.storage.set(`user_theme_${environment.env}`, theme);
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
### ✅ Fixed: [Feature]

**Fixes Applied:**
- [x] 🔴 C1: Removed separate Facade (orders.facade.ts deleted)
- [x] 🔴 C2: Component-scoped providers (orders.store.ts)
- [x] 🟡 I1: Added _${env} suffix (storage.service.ts)

**Verification:**
- [x] `ng build` ✓
- [x] `{pm} run lint` ✓
- [x] `ng test` ✓
- [x] `ng build --prod` ✓

**Skipped (with reason):**
- [ ] 🟢 N1: [Reason for skip]

**Final Score:** X/10 → 10/10
```

---

## Checklist

- [ ] ALL 🔴 Critical issues fixed
- [ ] 🟡 Important issues addressed (or justified skip)
- [ ] `ng build` passes
- [ ] `{pm} run lint` passes
- [ ] `ng test` passes
- [ ] `ng build --configuration=production` passes
- [ ] Fix report generated
