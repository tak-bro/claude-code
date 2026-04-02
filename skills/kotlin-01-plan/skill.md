---
name: kotlin-01-plan
description: "Android/Kotlin project implementation planning. Triggers on 'kotlin plan', 'android plan', '코틀린 계획', '안드로이드 계획', 'android feature', 'kt plan'. MVVM, Jetpack Compose, Hilt."
tools: Read, Bash, Glob, Grep, Agent
---

# Kotlin Plan

Android/Kotlin-specific implementation planning. **Base: Reference `/plan` skill for same output structure.**

## CRITICAL: STOP AFTER PLAN
Stop after planning is complete. Wait for the user's next command.
- **Plan file path:** `.claude/{YYYYMMDD}/PLAN-{HHMMSS}.md`
- Output: `Plan complete -> Next: /kotlin-02-implement`

---

## Phase 0: Explore First
Read the code first. Parallel exploration via sub-agents:
- `codebase-researcher`: Analyze existing ViewModel patterns, Module structure
- `best-practices-researcher`: Verify Android official documentation (Jetpack, Compose)
- `kotlin-android-expert`: Check architecture patterns, DI setup

## Phase 0.5: Rethink
For new features: "Is this really what the user wants?" Redefine (skip for bug fixes/infrastructure)

---

## Knowledge Sources
- `.claude/llms.txt`: [Project architecture -- generated via `/generate-codebase-context`. Reference if present]
- `./CLAUDE.md`: [patterns found]
- `[reference file]`: [patterns extracted]

---

## Android-Specific Planning

### Phase 1 Additional Analysis

1. **Module Structure**
   - Which feature module does this belong to?
   - Need to create new module? (`:feature:xxx`)
   - Shared dependencies from `:core:*` modules?

2. **Navigation Design**
   - NavGraph registration (Compose Navigation / Navigation Component)
   - Deep link support needed?
   - Arguments passing (type-safe with Safe Args or Compose)

3. **ViewModel + State Design**
   - UiState sealed interface/data class definition
   - UiEvent (user actions) definition
   - UiEffect (one-shot side effects: Snackbar, Navigation)
   - Flow/StateFlow selection

4. **Data Layer Design**
   - Repository interface + implementation
   - Room Entity / DAO needed?
   - Remote data source (Retrofit/Ktor) endpoints
   - Offline-first strategy needed?

### Phase 2 Additional Research
- Analyze existing ViewModel patterns (codebase-researcher)
- Check Hilt/Dagger module setup
- Verify Compose theme/design system patterns

---

## Android-Specific Boundaries

### Do
- MVVM + Clean Architecture (or MVI if project uses it)
- Jetpack Compose for new UI (unless project is View-based)
- Hilt for dependency injection
- Plan Screen(smart) / Component(presentational) separation
- State design with UiState + UiEvent + UiEffect

### [Warning] Ask First
- Creating new Gradle module
- Adding new library dependencies
- Modifying core/common modules
- WorkManager / background tasks

### Don't
- Activity-based navigation for new features (use single-activity)
- Direct network calls from ViewModel
- Mutable state exposed from ViewModel
- God Activity/Fragment antipattern

---

## Android-Specific Checklist Items

- [ ] Create/modify feature module
- [ ] Register in Navigation graph
- [ ] Define UiState sealed interface
- [ ] Define UiEvent sealed interface
- [ ] Define UiEffect sealed class (if one-shot events needed)
- [ ] Screen(smart) / Component(presentational) separation
- [ ] Repository interface + implementation
- [ ] Hilt module setup (@Module, @InstallIn)
- [ ] Room entities/DAOs (if local DB needed)
- [ ] ProGuard/R8 rules (if new library)

---

## Review Focus Additional Items

- [ ] ViewModel state managed via StateFlow (not LiveData for new code)
- [ ] No business logic in Composable functions
- [ ] Repository pattern: interface in domain, impl in data
- [ ] Hilt scoping correct (Singleton, ViewModelScoped, etc.)
- [ ] Coroutine scope: viewModelScope for ViewModel, lifecycleScope for UI

## Plan Mode Usage
- Enter Plan Mode with Shift+Tab -> iterate 2-3 times
- Use `think hard` for state design decisions

-> Next: `/kotlin-02-implement`
