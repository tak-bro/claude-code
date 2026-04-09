---
name: swift-02-implement
description: "iOS/Swift feature implementation. MVVM + SwiftUI + Swift Concurrency. Triggers on 'swift implement', '스위프트 구현', 'ios implement', 'iOS 구현', 'ios coding', 'swiftui implement'."
tools: Read, Bash, Glob, Grep, Write, Edit, Agent
---

# Swift Implement

Implement iOS features using MVVM + SwiftUI + Swift Concurrency.

**Agents:** swift-ios-expert, best-practices-researcher

**Refer to code patterns in `references/` directory:**
- `references/mvvm-swiftui-pattern.md` -- MVVM + SwiftUI detailed implementation

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
- MVVM: ViewModel holds state, View observes
- View (smart) vs Component (presentational) separation
- ViewState + Action pattern in ViewModel
- Repository pattern: protocol in domain, impl in data
- Protocol-based DI for all dependencies
- Run verification commands before done

### [Warning] Ask First
- Adding new SPM dependencies
- Modifying core/shared modules
- Creating new targets/packages
- Using iOS version-specific APIs

### Never Do
- Business logic in SwiftUI View body
- Force unwrapping without guard (`!`)
- Completion handler callbacks in new code (use async/await)
- Direct API/DB calls from ViewModel (must go through Repository)
- var for published state when val suffices
- Retain cycles (missing [weak self] in escaping closures)
- Hardcoded strings in UI (use Localizable)

---

## Architecture

```
View (Smart SwiftUI View - observes state, dispatches actions)
    |
ViewModel (@Observable / ObservableObject - State + Logic)
    |-- ViewState (published state)
    |-- Action handling (send/onAction)
    |-- Task management
        |
UseCase / Interactor (optional, for complex business logic)
        |
Repository (protocol in domain, impl in data)
    |-- RemoteDataSource (URLSession/Alamofire)
    |-- LocalDataSource (SwiftData/CoreData/UserDefaults)
```

---

## Folder Structure

### Feature-based (recommended)
```
Sources/
  App/
    {App}App.swift
    Navigation/
    DI/
  Features/
    {Feature}/
      Views/
        {Feature}View.swift        # Smart view
        Components/                 # Presentational views
      ViewModels/
        {Feature}ViewModel.swift
      Models/
        {Feature}ViewState.swift    # State + Action definitions
      Domain/
        Repositories/               # Repository protocols
        UseCases/                   # UseCase protocols (optional)
      Data/
        Repositories/               # Repository implementations
        Network/                    # API service, DTOs
        Local/                      # SwiftData models, local storage
        Mappers/                    # DTO <-> Domain mappers
  Core/
    Network/                        # HTTP client setup
    Storage/                        # Persistence setup
    Common/                         # Shared utilities
    DesignSystem/                   # Theme, shared components
```

---

## Tech Stack

```
SwiftUI                            # UI framework
Swift Concurrency (async/await)    # Async operations
@Observable (iOS 17+)              # State observation
Combine (iOS 13+, legacy/interop)  # Reactive (when needed)
URLSession / Alamofire             # HTTP client
SwiftData / Core Data              # Local persistence
Swift Package Manager              # Dependencies
Kingfisher / SDWebImage            # Image loading
```

---

## Verification Commands

```bash
# Xcode build
xcodebuild -workspace {Name}.xcworkspace -scheme {Scheme} -sdk iphonesimulator build
# or
xcodebuild -project {Name}.xcodeproj -scheme {Scheme} -sdk iphonesimulator build

# Tests
xcodebuild test -workspace {Name}.xcworkspace -scheme {Scheme} -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 16'

# Swift Package
swift build
swift test

# SwiftLint (if configured)
swiftlint lint --strict

# SwiftFormat (if configured)
swiftformat --lint .
```

---

## Checklist

- [ ] MVVM architecture (ViewModel + @Observable/@ObservableObject)
- [ ] ViewState struct/enum defined
- [ ] Action enum defined
- [ ] View(smart) vs Component(presentational) separation
- [ ] Repository protocol in domain, impl in data
- [ ] DI configured (protocol-based)
- [ ] async/await for all async operations (no completion handlers)
- [ ] Task cancellation handled (onDisappear, .task modifier)
- [ ] No business logic in View body
- [ ] No force unwrapping (use guard/if let/nil coalescing)
- [ ] No retain cycles ([weak self] in escaping closures)
- [ ] Accessibility (accessibilityLabel, accessibilityHint)
- [ ] Localization (String Catalogs / NSLocalizedString)
- [ ] Error handling (do-catch, Result, throws)
- [ ] Verification commands pass

---

## --team Mode
(see `_shared/team-mode.md`)

### Team Composition

| Role | Responsibility | Files |
|------|---------------|-------|
| UI Agent | SwiftUI views, components | `Views/` |
| ViewModel Agent | ViewModels, state, actions | `ViewModels/`, `Models/` |
| Data Agent | Repository, API, Storage | `Data/`, `Domain/` |
| Core Agent | Shared utilities, DI | `Core/`, `DI/` |

---

## Output
(see `_shared/output-templates.md`)

-> Next: `/swift-03-review`
