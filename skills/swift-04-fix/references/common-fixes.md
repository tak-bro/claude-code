# Common iOS/Swift Fixes

## [Critical] Business Logic in View

```swift
// Before (BAD)
struct FeatureView: View {
    @State private var items: [Item] = []

    var body: some View {
        List(items) { item in ... }
            .task {
                let response = try? await URLSession.shared.data(from: url)  // Logic in View!
                if let data = response?.0 {
                    items = try! JSONDecoder().decode([Item].self, from: data)
                }
            }
    }
}

// After (GOOD)
struct FeatureView: View {
    @State private var viewModel = FeatureViewModel(repository: /* injected */)

    var body: some View {
        FeatureContent(state: viewModel.state, onAction: viewModel.send)
            .task { viewModel.send(.loadItems) }
    }
}
```

## [Critical] Force Unwrapping

```swift
// Before (BAD)
let url = URL(string: urlString)!
let data = try! JSONDecoder().decode(Item.self, from: data)
let item = items.first!

// After (GOOD)
guard let url = URL(string: urlString) else {
    throw AppError.invalidURL
}
let item = try JSONDecoder().decode(Item.self, from: data)
guard let firstItem = items.first else {
    return // or throw
}
```

## [Critical] Direct API Call from ViewModel

```swift
// Before (BAD)
@Observable
final class FeatureViewModel {
    private let apiService: FeatureAPIService  // Concrete dependency!

    func loadItems() async {
        let items = try? await apiService.fetchItems()
    }
}

// After (GOOD)
@Observable
final class FeatureViewModel {
    private let repository: FeatureRepositoryProtocol  // Protocol abstraction

    func loadItems() async {
        do {
            let items = try await repository.getItems()
            state.items = items
        } catch {
            state.error = error.localizedDescription
        }
    }
}
```

## [Critical] Missing DI Protocol

```swift
// Before (BAD)
final class FeatureViewModel {
    private let repository = FeatureRepositoryImpl()  // Concrete + manual init!
}

// After (GOOD)
final class FeatureViewModel {
    private let repository: FeatureRepositoryProtocol  // Protocol

    init(repository: FeatureRepositoryProtocol) {      // Injected
        self.repository = repository
    }
}
```

## [Critical] Completion Handler in New Code

```swift
// Before (BAD)
func fetchItems(completion: @escaping (Result<[Item], Error>) -> Void) {
    URLSession.shared.dataTask(with: url) { data, response, error in
        if let error { completion(.failure(error)); return }
        // ...
    }.resume()
}

// After (GOOD)
func fetchItems() async throws -> [Item] {
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode([Item].self, from: data)
}
```

## [Critical] Retain Cycle

```swift
// Before (BAD)
publisher.sink { value in
    self.handleValue(value)  // Strong reference to self!
}

timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
    self.updateUI()  // Retain cycle!
}

// After (GOOD)
publisher.sink { [weak self] value in
    self?.handleValue(value)
}

timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
    self?.updateUI()
}
```

## [Critical] Public Mutable State

```swift
// Before (BAD)
@Observable
final class FeatureViewModel {
    var state = FeatureViewState()  // Anyone can mutate!
}

// After (GOOD)
@Observable
final class FeatureViewModel {
    private(set) var state = FeatureViewState()  // Only ViewModel mutates
}
```

## [Critical] Missing @MainActor on ViewModel

```swift
// Before (BAD)
@Observable
final class FeatureViewModel { // State mutations from background task -- runtime warning in Swift 6
    private(set) var state = FeatureViewState()

    func loadItems() async {
        state.isLoading = true // May not be on main thread!
    }
}

// After (GOOD)
@Observable
@MainActor
final class FeatureViewModel {
    private(set) var state = FeatureViewState()

    func loadItems() async {
        state.isLoading = true // Guaranteed main thread
    }
}
```

## [Critical] Untracked Task in ViewModel

```swift
// Before (BAD) -- Tasks never cancelled, can duplicate on rapid taps
@Observable @MainActor
final class FeatureViewModel {
    func send(_ action: FeatureAction) {
        switch action {
        case .loadItems:
            Task { await loadItems() } // Untracked! Never cancelled!
        }
    }
}

// After (GOOD) -- Track and cancel
@Observable @MainActor
final class FeatureViewModel {
    private var loadTask: Task<Void, Never>?

    func send(_ action: FeatureAction) {
        switch action {
        case .loadItems:
            loadTask?.cancel()
            loadTask = Task { await loadItems() }
        }
    }
}
```

## [Important] Missing Task Cancellation

```swift
// Before (BAD)
struct FeatureView: View {
    var body: some View {
        Text("Hello")
            .onAppear {
                Task { await viewModel.send(.loadItems) }  // Never cancelled!
            }
    }
}

// After (GOOD)
struct FeatureView: View {
    var body: some View {
        Text("Hello")
            .task {  // Automatically cancelled when view disappears
                viewModel.send(.loadItems)
            }
    }
}
```

## [Important] Missing Error Handling

```swift
// Before (BAD)
func loadItems() async {
    let items = try? await repository.getItems()  // Error silently swallowed
    state.items = items ?? []
}

// After (GOOD)
func loadItems() async {
    state.isLoading = true
    do {
        state.items = try await repository.getItems()
        state.error = nil
    } catch {
        state.error = error.localizedDescription
    }
    state.isLoading = false
}
```

## [Important] Hardcoded Strings

```swift
// Before (BAD)
Text("Loading...")
Text("Error occurred")
Button("Retry") { ... }

// After (GOOD)
Text(String(localized: "feature.loading"))
Text(String(localized: "feature.error"))
Button(String(localized: "common.retry")) { ... }
```

## [Important] Missing Accessibility

```swift
// Before (BAD)
Image(systemName: "trash")
    .onTapGesture { deleteItem() }

// After (GOOD)
Image(systemName: "trash")
    .accessibilityLabel(String(localized: "feature.delete"))
    .accessibilityHint(String(localized: "feature.delete.hint"))
    .onTapGesture { deleteItem() }
```
