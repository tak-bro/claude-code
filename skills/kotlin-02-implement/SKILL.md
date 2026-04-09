---
name: kotlin-02-implement
description: "Android/Kotlin feature implementation. MVVM + Jetpack Compose + Hilt. Triggers on 'kotlin implement', '코틀린 구현', 'android implement', '안드로이드 구현', 'android coding', 'kt implement'."
tools: Read, Bash, Glob, Grep, Write, Edit, Agent
---

# Kotlin Implement

Implement Android features using MVVM + Jetpack Compose + Hilt.

**Agents:** kotlin-android-expert, tak-typescript-expert (for shared logic), best-practices-researcher

**Refer to code patterns in `references/` directory:**
- `references/mvvm-compose-pattern.md` -- MVVM + Compose detailed implementation

---

## Pre-Implementation

**Before coding, verify from plan output:**
1. Plan approved?
2. Files to create/modify identified?
3. Implementation Checklist ready?

---

## TDD-First Principle (Claude Philosophy)
1. Write failing tests first
2. Write minimal code to pass tests
3. Refactor
- Switch to Auto-Accept Mode with Shift+Tab after plan approval
- For large features, use `use subagents` to separate main context

---

## Boundaries

### Always Do
- Use plan's Implementation Checklist
- MVVM: ViewModel holds state, Composable observes
- Screen (smart) vs Component (presentational) separation
- UiState + UiEvent + UiEffect pattern in ViewModel
- Repository pattern: interface in domain, impl in data
- Hilt DI for all dependencies
- Run verification commands before done

### [Warning] Ask First
- Adding new dependencies (check version catalog)
- Modifying core/common modules
- Creating new Gradle modules
- Using experimental Compose APIs

### Never Do
- Expose MutableStateFlow publicly from ViewModel
- Business logic in Composable functions
- Direct API/DB calls from ViewModel (must go through Repository)
- God Activity/Fragment (single-activity pattern)
- Blocking calls on Main thread
- `var` for state (use `val` + immutable data classes)
- Hardcoded strings in UI (use string resources)

---

## Architecture

```
Screen (Smart Composable - collects state, dispatches events)
    |
ViewModel (State holder + Business Logic orchestration)
    |-- UiState (StateFlow<UiState>)
    |-- UiEvent handling (onEvent)
    |-- UiEffect (Channel<UiEffect>)
        |
UseCase / Interactor (optional, for complex business logic)
        |
Repository (interface in domain, impl in data)
    |-- RemoteDataSource (Retrofit/Ktor)
    |-- LocalDataSource (Room/DataStore)
```

---

## Folder Structure

### Multi-module (recommended)
```
:app                        # Application module
:feature:
  {feature}/
    presentation/
      {Feature}Screen.kt       # Smart composable
      {Feature}ViewModel.kt    # ViewModel
      {Feature}UiState.kt      # State definitions
      components/               # Presentational composables
    domain/
      model/                    # Domain models
      repository/               # Repository interfaces
      usecase/                  # UseCases (optional)
    data/
      repository/               # Repository implementations
      remote/                   # API service, DTOs
      local/                    # Room DAOs, entities
      mapper/                   # DTO <-> Domain mappers
    di/
      {Feature}Module.kt       # Hilt module
:core:
  network/                     # Retrofit/Ktor setup
  database/                    # Room database
  common/                      # Shared utilities
  designsystem/                # Theme, shared composables
```

### Single-module (small projects)
```
app/src/main/java/com/example/
  feature/{feature}/
    {Feature}Screen.kt
    {Feature}ViewModel.kt
    {Feature}UiState.kt
    {Feature}Repository.kt
  core/
    network/
    database/
    di/
```

---

## Tech Stack

```
Jetpack Compose                # UI framework
Hilt (Dagger)                  # Dependency injection
Kotlin Coroutines + Flow       # Async + reactive
Retrofit / Ktor                # HTTP client
Room                           # Local database
Jetpack DataStore              # Preferences storage
Compose Navigation             # Navigation
Coil / Glide                   # Image loading
Material 3                     # Design system
```

---

## Verification Commands

```bash
# Detect build tool
if [ -f "gradlew" ]; then
  ./gradlew lint{Variant}                    # Lint check
  ./gradlew test{Variant}UnitTest            # Unit tests
  ./gradlew connectedAndroidTest             # Instrumented tests (if device available)
  ./gradlew assembleDebug                    # Build check
  ./gradlew detekt                           # Static analysis (if configured)
  ./gradlew ktlintCheck                      # Code style (if configured)
fi
```

---

## Checklist

- [ ] MVVM architecture (ViewModel + StateFlow)
- [ ] UiState sealed interface/data class
- [ ] UiEvent sealed interface
- [ ] UiEffect sealed class (if one-shot events)
- [ ] Screen(smart) vs Component(presentational) separation
- [ ] Repository interface in domain, impl in data
- [ ] Hilt DI configured (@Module, @InstallIn, @Inject)
- [ ] ViewModel uses viewModelScope for coroutines
- [ ] No MutableStateFlow exposed publicly
- [ ] No business logic in Composable functions
- [ ] Immutable data classes for state
- [ ] String resources (no hardcoded strings in UI)
- [ ] Error handling in Repository/ViewModel
- [ ] ProGuard rules for new libraries (if release)
- [ ] Verification commands pass

---

## --team Mode
(see `_shared/team-mode.md`)

### Team Composition

| Role | Responsibility | Files |
|------|---------------|-------|
| UI Agent | Compose screens, components | `presentation/` |
| ViewModel Agent | ViewModels, state, events | `presentation/*ViewModel.kt` |
| Data Agent | Repository, API, Room | `data/`, `domain/` |
| DI Agent | Hilt modules, scoping | `di/` |

---

## Output
(see `_shared/output-templates.md`)

-> Next: `/kotlin-03-review`
