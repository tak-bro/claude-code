---
name: swift-01-plan
model: sonnet
description: "iOS/Swift project implementation planning. Triggers on 'swift plan', 'ios plan', '스위프트 계획', 'iOS 계획', 'ios feature', 'swiftui plan'. MVVM, SwiftUI, Swift Concurrency."
tools: Read, Bash, Glob, Grep, Agent
---

# Swift Plan

iOS/Swift-specific implementation planning. **Base: Reference `/plan` skill for same output structure.**

## CRITICAL: STOP AFTER PLAN
Stop after planning is complete. Wait for the user's next command.
- **Plan file path:** `.claude/{YYYYMMDD}/PLAN-{HHMMSS}.md`
- Output: `Plan complete -> Next: /swift-02-implement`

---

## Phase 0: Explore First
Read the code first. Parallel exploration via sub-agents:
- `codebase-researcher`: Analyze existing ViewModel patterns, module structure
- `best-practices-researcher`: Verify Apple official documentation (SwiftUI, Combine, Swift Concurrency)
- `swift-ios-expert`: Check architecture patterns, DI setup

## Phase 0.5: Rethink
For new features: "Is this really what the user wants?" Redefine (skip for bug fixes/infrastructure)

---

## Knowledge Sources
- `.claude/llms.txt`: [Project architecture -- generated via `/generate-codebase-context`. Reference if present]
- `./CLAUDE.md`: [patterns found]
- `[reference file]`: [patterns extracted]

---

## iOS-Specific Planning

### Phase 1 Additional Analysis

1. **Module/Target Structure**
   - Which feature module/target does this belong to?
   - Need to create new Swift Package / framework target?
   - Shared dependencies from core modules?

2. **Navigation Design**
   - NavigationStack / NavigationSplitView (SwiftUI)
   - Coordinator pattern (if UIKit hybrid)
   - Deep link support needed?

3. **ViewModel + State Design**
   - @Observable class (iOS 17+) or ObservableObject (iOS 16-)
   - ViewState enum/struct definition
   - Action enum (user interactions)
   - Side effects handling

4. **Data Layer Design**
   - Repository protocol + implementation
   - SwiftData / Core Data models needed?
   - Network service (URLSession / Alamofire) endpoints
   - Offline-first strategy needed?

### Phase 2 Additional Research
- Analyze existing ViewModel patterns (codebase-researcher)
- Check DI container setup (Swinject, Factory, manual)
- Verify SwiftUI theme/design system patterns

---

## iOS-Specific Boundaries

### Do
- MVVM + Clean Architecture
- SwiftUI for new UI (unless project is UIKit-based)
- Protocol-based dependency injection
- Plan View(smart) / Component(presentational) separation
- State design with ViewState + Action

### [Warning] Ask First
- Creating new Swift Package / framework target
- Adding new SPM dependencies
- Modifying core/shared modules
- Background tasks / push notifications

### Don't
- Massive ViewController antipattern
- Direct network calls from View or ViewModel
- Force unwrapping (`!`) without guard
- Singleton abuse (prefer DI)
- Storyboard for new views (use SwiftUI or programmatic UIKit)

---

## iOS-Specific Checklist Items

- [ ] Create/modify feature module
- [ ] Register in Navigation (NavigationStack or Coordinator)
- [ ] Define ViewState (enum or struct)
- [ ] Define Action enum
- [ ] View(smart) / Component(presentational) separation
- [ ] Repository protocol + implementation
- [ ] DI registration
- [ ] SwiftData/CoreData models (if local storage needed)
- [ ] Accessibility labels and traits
- [ ] Localization (NSLocalizedString / String Catalog)

---

## Review Focus Additional Items

- [ ] @Observable or ObservableObject used correctly
- [ ] No business logic in SwiftUI View body
- [ ] Repository pattern: protocol in domain, impl in data
- [ ] Proper use of async/await (no completion handlers in new code)
- [ ] Task cancellation handled
- [ ] Memory management: no retain cycles (weak self in closures)

## Plan Mode Usage
- Enter Plan Mode with Shift+Tab -> iterate 2-3 times
- Use `think hard` for state design decisions

-> Next: `/swift-02-implement`
