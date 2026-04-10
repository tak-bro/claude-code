---
name: angular-02-implement
description: "Angular feature implementation. NgRx ComponentStore + Module-based. Triggers on 'angular implement', '앵귤러 구현', 'angular coding', 'ng implement', 'angular 개발'."
tools: Read, Bash, Glob, Grep, Write, Edit, Agent
---

# Angular Implement

Implement Angular features using NgRx ComponentStore pattern.

**Agents:** angular-componentstore-expert, tak-typescript-expert, best-practices-researcher

**Refer to code patterns in `references/` directory:**
- `references/componentstore-pattern.md` — ComponentStore detailed implementation
- `references/ionic-patterns.md` — IonBasePage, IonicRouteStrategy, service wrappers
- `references/tanstack-query-pattern.md` — TanStack Query Angular pattern

---

## Pre-Implementation

**Before coding, verify from plan output:**
1. Plan approved?
2. Files to create/modify identified?
3. Implementation Checklist ready?

---

(see `_shared/implement-common.md`)

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

### [Warning] Ask First
- Adding new dependencies
- Modifying shared services
- Creating new patterns not in codebase
- Using `providedIn: 'root'` for ComponentStore

### 🚫 Never Do
- `export default` (except App entrypoint, config)
- Standalone components (Module-based only)
- Component → API direct calls (must go through ComponentStore)
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

## Verification Commands
(see `_shared/verification-commands.md`)

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

## --team Mode
(see `_shared/team-mode.md`)

### Team Composition

| Role | Responsibility | Files |
|------|---------------|-------|
| Store Agent | ComponentStore, state, effects | `stores/` |
| Page Agent | Smart components, orchestration | `pages/` |
| Component Agent | Presentational components | `components/` |
| Service Agent | API services, types | `services/`, `types/` |

---

## Output
(see `_shared/output-templates.md`)

→ Next: `/simplify` → `/angular-03-review`
