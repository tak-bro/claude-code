---
name: swift-05-test
description: "iOS/Swift test writing and execution. Triggers on 'swift test', '스위프트 테스트', 'ios test', 'iOS 테스트', 'swiftui test', 'xctest'."
model: sonnet
tools: Read, Bash, Glob, Grep, Write, Edit
---

# Swift Test

Write and run tests for iOS/Swift features.

**Agents:** test-strategy-expert, swift-ios-expert

## Pre-Test

1. Implementation complete? (`/swift-02-implement`)
2. Review passed? (`/swift-03-review` → `/swift-04-fix`)
3. Files to test identified

---

## Swift Test Stack

```
XCTest                            # Unit/Integration
Swift Testing (@Test)             # Modern test framework (Swift 6+)
URLProtocol                       # Network mocking
ViewInspector                     # SwiftUI view testing
```

**Maintain 100% compatibility with existing test stack and patterns.**

---

## Test Strategy

1. **Analyze implemented files** — prioritize by criticality:
   - [Critical] ViewModel → state changes, Published properties, error handling
   - [Critical] Repository → data mapping, caching, error propagation
   - [Critical] UseCase → business rules, edge cases
   - [Important] Mapper → transformation correctness
   - [Important] View (SwiftUI) → Preview + UI state rendering
   - [Nice] Component (optional)

2. **Mocking targets**:
   - Repository → protocol-based mock
   - API Service → URLProtocol mock or protocol mock
   - Local storage → in-memory implementation
   - UserDefaults → ephemeral suite (`UserDefaults(suiteName:)`)
   - Keychain → mock wrapper

---

## Test Patterns

### ViewModel Test (Required)

```swift
@MainActor
final class FeatureViewModelTests: XCTestCase {
    private var repository: MockFeatureRepository!
    private var sut: FeatureViewModel!

    override func setUp() {
        super.setUp()
        repository = MockFeatureRepository()
        sut = FeatureViewModel(repository: repository)
    }

    override func tearDown() {
        sut = nil
        repository = nil
        super.tearDown()
    }

    func test_loadData_success_updatesState() async {
        // Arrange
        repository.stubbedResult = .success(testData)

        // Act
        await sut.loadData()

        // Assert
        XCTAssertEqual(sut.items, expectedItems)
        XCTAssertFalse(sut.isLoading)
    }

    func test_loadData_failure_setsError() async {
        // Arrange
        repository.stubbedResult = .failure(TestError.network)

        // Act
        await sut.loadData()

        // Assert
        XCTAssertNotNil(sut.errorMessage)
        XCTAssertTrue(sut.items.isEmpty)
    }
}
```

### Repository Test (Required)

```swift
final class FeatureRepositoryTests: XCTestCase {
    private var apiService: MockAPIService!
    private var cache: MockCache!
    private var sut: FeatureRepositoryImpl!

    override func setUp() {
        super.setUp()
        apiService = MockAPIService()
        cache = MockCache()
        sut = FeatureRepositoryImpl(api: apiService, cache: cache)
    }

    func test_getData_returnsCachedData_whenAvailable() async throws {
        // Arrange
        cache.stubbedData = testData

        // Act
        let result = try await sut.getData()

        // Assert
        XCTAssertEqual(result, expectedData)
        XCTAssertEqual(apiService.fetchCallCount, 0)
    }
}
```

---

## Test Writing Rules

### Do
- AAA pattern (Arrange-Act-Assert) required
- Naming: `test_[action]_[condition]_[expected]`
- `async/await` for async tests
- Protocol-based mocks (no third-party mocking frameworks)
- `@MainActor` for ViewModel tests
- Test error/edge cases (empty, nil, network error, timeout)
- `setUp` / `tearDown` for clean state per test

### 🚫 Don't
- Snapshot tests (unless explicitly requested)
- Implementation detail tests (testing private methods)
- Share state between tests
- `sleep()` / `Task.sleep()` → use expectations or async await
- Test UIKit/SwiftUI internals directly (use ViewInspector)
- Force unwrap in tests → use `XCTUnwrap`

---

## Run and Verify

```bash
xcodebuild test -workspace {Name}.xcworkspace -scheme {Scheme} -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 16'
swift test --verbose
```

---

## Output

```markdown
### Test Results: [Feature Name]

**Coverage**
| File | Statements | Branches | Functions | Lines |
|------|-----------|----------|-----------|-------|

**Test Count**
- Pass: [N]
- Fail: [N]

→ All tests pass → Complete
→ Failures → `/swift-04-fix`
```
