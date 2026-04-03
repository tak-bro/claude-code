# MVVM + SwiftUI Pattern Reference

## ViewState Pattern

```swift
// Struct-based state (simple, recommended for most cases)
struct FeatureViewState: Equatable {
    var items: [FeatureItem] = []
    var isLoading: Bool = false
    var error: String? = nil
    var searchQuery: String = ""

    var filteredItems: [FeatureItem] {
        guard !searchQuery.isEmpty else { return items }
        return items.filter { $0.name.localizedCaseInsensitiveContains(searchQuery) }
    }
}

// Enum-based state (for distinct loading/success/error states)
enum FeatureViewState: Equatable {
    case idle
    case loading
    case loaded(items: [FeatureItem])
    case error(message: String)
}
```

## Action Pattern

```swift
enum FeatureAction {
    case loadItems
    case refresh
    case selectItem(id: String)
    case deleteItem(id: String)
    case searchQueryChanged(String)
}
```

## ViewModel Pattern (iOS 17+ with @Observable)

```swift
@Observable
@MainActor
final class FeatureViewModel {
    // State
    private(set) var state = FeatureViewState()

    // Dependencies
    private let repository: FeatureRepositoryProtocol

    init(repository: FeatureRepositoryProtocol) {
        self.repository = repository
    }

    // Track tasks for cancellation
    private var loadTask: Task<Void, Never>?

    // Action handler
    func send(_ action: FeatureAction) {
        switch action {
        case .loadItems:
            loadTask?.cancel()
            loadTask = Task { await loadItems() }
        case .refresh:
            loadTask?.cancel()
            loadTask = Task { await refresh() }
        case .selectItem(let id):
            selectItem(id: id)
        case .deleteItem(let id):
            Task { await deleteItem(id: id) }
        case .searchQueryChanged(let query):
            state.searchQuery = query
        }
    }

    private func loadItems() async {
        state.isLoading = true
        state.error = nil

        do {
            let items = try await repository.getItems()
            state.items = items
        } catch {
            state.error = error.localizedDescription
        }

        state.isLoading = false
    }

    private func deleteItem(id: String) async {
        do {
            try await repository.deleteItem(id: id)
            state.items.removeAll { $0.id == id }
        } catch {
            state.error = error.localizedDescription
        }
    }
}
```

## ViewModel Pattern (iOS 14-16 with ObservableObject)

```swift
@MainActor
final class FeatureViewModel: ObservableObject {
    @Published private(set) var state = FeatureViewState()

    private let repository: FeatureRepositoryProtocol
    private var loadTask: Task<Void, Never>?

    init(repository: FeatureRepositoryProtocol) {
        self.repository = repository
    }

    deinit {
        loadTask?.cancel()
    }

    func send(_ action: FeatureAction) {
        switch action {
        case .loadItems:
            loadTask = Task { await loadItems() }
        case .deleteItem(let id):
            Task { await deleteItem(id: id) }
        default:
            break
        }
    }

    private func loadItems() async {
        state.isLoading = true
        do {
            let items = try await repository.getItems()
            guard !Task.isCancelled else { return }
            state.items = items
        } catch {
            guard !Task.isCancelled else { return }
            state.error = error.localizedDescription
        }
        state.isLoading = false
    }
}
```

## View (Smart) Pattern

```swift
struct FeatureView: View {
    @State private var viewModel: FeatureViewModel  // iOS 17+ @Observable

    init(repository: FeatureRepositoryProtocol) {
        _viewModel = State(initialValue: FeatureViewModel(repository: repository))
    }

    var body: some View {
        FeatureContent(
            state: viewModel.state,
            onAction: viewModel.send
        )
        .task {
            viewModel.send(.loadItems)
        }
    }
}

// iOS 14-16 version
struct FeatureView: View {
    @StateObject private var viewModel: FeatureViewModel

    init(repository: FeatureRepositoryProtocol) {
        _viewModel = StateObject(wrappedValue: FeatureViewModel(repository: repository))
    }

    var body: some View {
        FeatureContent(
            state: viewModel.state,
            onAction: viewModel.send
        )
        .task {
            viewModel.send(.loadItems)
        }
    }
}
```

## Content (Presentational) Pattern

```swift
struct FeatureContent: View {
    let state: FeatureViewState
    let onAction: (FeatureAction) -> Void

    var body: some View {
        NavigationStack {
            Group {
                if state.isLoading {
                    ProgressView()
                } else if let error = state.error {
                    ErrorView(message: error) {
                        onAction(.loadItems)
                    }
                } else {
                    itemList
                }
            }
            .navigationTitle("Features")
        }
    }

    private var itemList: some View {
        List {
            ForEach(state.filteredItems) { item in
                FeatureItemRow(item: item)
                    .onTapGesture {
                        onAction(.selectItem(id: item.id))
                    }
            }
            .onDelete { indexSet in
                for index in indexSet {
                    let item = state.filteredItems[index]
                    onAction(.deleteItem(id: item.id))
                }
            }
        }
        .searchable(text: Binding(
            get: { state.searchQuery },
            set: { onAction(.searchQueryChanged($0)) }
        ))
        .refreshable {
            onAction(.refresh)
        }
    }
}
```

## Repository Pattern

```swift
// Domain layer - protocol
protocol FeatureRepositoryProtocol: Sendable {
    func getItems() async throws -> [FeatureItem]
    func getItem(id: String) async throws -> FeatureItem
    func createItem(body: CreateFeatureBody) async throws -> FeatureItem
    func deleteItem(id: String) async throws
}

// Data layer - implementation
final class FeatureRepositoryImpl: FeatureRepositoryProtocol {
    private let remoteDataSource: FeatureRemoteDataSourceProtocol
    private let localDataSource: FeatureLocalDataSourceProtocol

    init(
        remoteDataSource: FeatureRemoteDataSourceProtocol,
        localDataSource: FeatureLocalDataSourceProtocol
    ) {
        self.remoteDataSource = remoteDataSource
        self.localDataSource = localDataSource
    }

    func getItems() async throws -> [FeatureItem] {
        do {
            let dtos = try await remoteDataSource.fetchItems()
            let items = dtos.map { $0.toDomain() }
            try await localDataSource.saveItems(items)
            return items
        } catch {
            // Fallback to cache
            let cached = try await localDataSource.getItems()
            if cached.isEmpty { throw error }
            return cached
        }
    }
}
```

## Network Service Pattern

```swift
// API Service
protocol FeatureAPIServiceProtocol: Sendable {
    func fetchItems() async throws -> [FeatureDTO]
    func createItem(body: CreateFeatureBody) async throws -> FeatureDTO
}

final class FeatureAPIService: FeatureAPIServiceProtocol {
    private let httpClient: HTTPClientProtocol

    init(httpClient: HTTPClientProtocol) {
        self.httpClient = httpClient
    }

    func fetchItems() async throws -> [FeatureDTO] {
        let request = URLRequest(url: URL(string: "\(baseURL)/features")!)
        return try await httpClient.send(request)
    }
}
```

## DI Pattern (Factory-based)

```swift
// Protocol-based DI container
@MainActor
final class DependencyContainer {
    // Network
    lazy var httpClient: HTTPClientProtocol = URLSessionHTTPClient()

    // API Services
    lazy var featureAPIService: FeatureAPIServiceProtocol =
        FeatureAPIService(httpClient: httpClient)

    // Repositories
    lazy var featureRepository: FeatureRepositoryProtocol =
        FeatureRepositoryImpl(
            remoteDataSource: featureAPIService,
            localDataSource: FeatureLocalDataSource()
        )

    // ViewModels
    func makeFeatureViewModel() -> FeatureViewModel {
        FeatureViewModel(repository: featureRepository)
    }
}

// Usage in App
@main
struct MyApp: App {
    @State private var container = DependencyContainer()

    var body: some Scene {
        WindowGroup {
            FeatureView(repository: container.featureRepository)
        }
    }
}
```

## Navigation Pattern

```swift
// Type-safe navigation with NavigationStack
enum AppRoute: Hashable {
    case featureList
    case featureDetail(id: String)
    case settings
}

struct AppNavigator: View {
    @State private var path = NavigationPath()

    var body: some View {
        NavigationStack(path: $path) {
            FeatureListView()
                .navigationDestination(for: AppRoute.self) { route in
                    switch route {
                    case .featureList:
                        FeatureListView()
                    case .featureDetail(let id):
                        FeatureDetailView(id: id)
                    case .settings:
                        SettingsView()
                    }
                }
        }
    }
}
```

## Mapper Pattern

```swift
// DTO -> Domain
extension FeatureDTO {
    func toDomain() -> FeatureItem {
        FeatureItem(
            id: id,
            name: name,
            description: description,
            createdAt: ISO8601DateFormatter().date(from: createdAt) ?? .now
        )
    }
}

// Domain -> SwiftData Entity
extension FeatureItem {
    func toEntity() -> FeatureEntity {
        FeatureEntity(
            id: id,
            name: name,
            description: description,
            createdAt: createdAt
        )
    }
}
```

## Error Handling Pattern

```swift
enum AppError: LocalizedError {
    case network(underlying: Error)
    case notFound
    case unauthorized
    case serverError(statusCode: Int)
    case unknown

    var errorDescription: String? {
        switch self {
        case .network(let error):
            return "Network error: \(error.localizedDescription)"
        case .notFound:
            return "Resource not found"
        case .unauthorized:
            return "Please sign in again"
        case .serverError(let code):
            return "Server error (code: \(code))"
        case .unknown:
            return "An unexpected error occurred"
        }
    }
}
```
