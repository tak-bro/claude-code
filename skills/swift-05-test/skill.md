---
name: swift-05-test
description: "iOS/Swift test writing and execution. Triggers on 'swift test', '스위프트 테스트', 'ios test', 'iOS 테스트', 'swiftui test', 'xctest'."
tools: Read, Bash, Glob, Grep, Write, Edit
---

# Swift Test

Write and run tests for iOS/Swift features.

**Agents:** test-strategy-expert, swift-ios-expert

## Pre-Test

1. Implementation complete? (`/swift-02-implement`)
2. Review passed? (`/swift-03-review` -> `/swift-04-fix`)
3. Files to test identified

---

## iOS Test Stack

```
XCTest                           # Test framework
Swift Testing (@Test, iOS 16+)   # Modern test framework
XCTest + async/await             # Async testing
ViewInspector                    # SwiftUI view testing (optional)
Combine Testing                  # Publisher testing
```

**Maintain 100% compatibility with existing test stack and patterns.**

---

## Test Strategy

1. **Analyze implemented files**
   - ViewModel -> [Critical] Required test
   - Repository -> [Critical] Required test
   - UseCase -> [Critical] Required test
   - Mapper/Converter -> [Critical] Required test (pure functions)
   - View (SwiftUI) -> [Important] Preview + UI test
   - Component (SwiftUI) -> [Nice] Optional

2. **Determine mocking targets**
   - Repository -> Protocol mock
   - API Service -> URLProtocol mock or protocol mock
   - Local storage -> In-memory implementation
   - UserDefaults -> Ephemeral UserDefaults suite

---

## Test Patterns

### ViewModel Test (Required)

```swift
@MainActor
final class FeatureViewModelTests: XCTestCase {
    private var sut: FeatureViewModel!
    private var mockRepository: MockFeatureRepository!

    override func setUp() {
        super.setUp()
        mockRepository = MockFeatureRepository()
        sut = FeatureViewModel(repository: mockRepository)
    }

    override func tearDown() {
        sut = nil
        mockRepository = nil
        super.tearDown()
    }

    func test_loadItems_success_updatesState() async {
        // Arrange
        let expectedItems = [FeatureItem(id: "1", name: "Test")]
        mockRepository.getItemsResult = .success(expectedItems)

        // Act
        await sut.send(.loadItems)

        // Assert
        XCTAssertEqual(sut.state.items, expectedItems)
        XCTAssertFalse(sut.state.isLoading)
        XCTAssertNil(sut.state.error)
    }

    func test_loadItems_failure_setsError() async {
        // Arrange
        mockRepository.getItemsResult = .failure(NetworkError.noConnection)

        // Act
        await sut.send(.loadItems)

        // Assert
        XCTAssertTrue(sut.state.items.isEmpty)
        XCTAssertNotNil(sut.state.error)
    }
}
```

### Swift Testing Framework (Xcode 16+ / Swift 6)

```swift
@Suite("FeatureViewModel Tests", .serialized)
@MainActor
struct FeatureViewModelTests {
    // Each test creates its own mock to avoid data races
    @Test("loads items successfully")
    func loadItems_success() async {
        let mockRepository = MockFeatureRepository()
        let viewModel = FeatureViewModel(repository: mockRepository)
        mockRepository.getItemsResult = .success([FeatureItem(id: "1", name: "Test")])

        await viewModel.send(.loadItems)

        #expect(viewModel.state.items.count == 1)
        #expect(!viewModel.state.isLoading)
    }

    @Test("handles load error")
    func loadItems_failure() async {
        let mockRepository = MockFeatureRepository()
        let viewModel = FeatureViewModel(repository: mockRepository)
        mockRepository.getItemsResult = .failure(NetworkError.noConnection)

        await viewModel.send(.loadItems)

        #expect(viewModel.state.error != nil)
    }
}
```

### Repository Test (Required)

```swift
final class FeatureRepositoryImplTests: XCTestCase {
    private var sut: FeatureRepositoryImpl!
    private var mockRemoteDataSource: MockFeatureRemoteDataSource!
    private var mockLocalDataSource: MockFeatureLocalDataSource!

    override func setUp() {
        super.setUp()
        mockRemoteDataSource = MockFeatureRemoteDataSource()
        mockLocalDataSource = MockFeatureLocalDataSource()
        sut = FeatureRepositoryImpl(
            remoteDataSource: mockRemoteDataSource,
            localDataSource: mockLocalDataSource
        )
    }

    func test_getItems_returnsRemoteData() async throws {
        // Arrange
        let expected = [FeatureDTO(id: "1", name: "Remote")]
        mockRemoteDataSource.fetchItemsResult = .success(expected)

        // Act
        let result = try await sut.getItems()

        // Assert
        XCTAssertEqual(result.count, 1)
        XCTAssertEqual(result.first?.name, "Remote")
    }

    func test_getItems_fallbackToCache_whenRemoteFails() async throws {
        // Arrange
        mockRemoteDataSource.fetchItemsResult = .failure(NetworkError.noConnection)
        mockLocalDataSource.cachedItems = [FeatureItem(id: "1", name: "Cached")]

        // Act
        let result = try await sut.getItems()

        // Assert
        XCTAssertEqual(result.first?.name, "Cached")
    }
}
```

### Mock Pattern

```swift
final class MockFeatureRepository: FeatureRepositoryProtocol {
    var getItemsResult: Result<[FeatureItem], Error> = .success([])
    var getItemCallCount = 0

    func getItems() async throws -> [FeatureItem] {
        getItemCallCount += 1
        return try getItemsResult.get()
    }
}
```

---

## Test Writing Rules

### Do
- Behavior-based test naming (`test_[action]_[condition]_[expected]`)
- AAA pattern (Arrange-Act-Assert) required
- Use `async/await` for async tests
- Use protocol-based mocks (no third-party mocking library needed)
- Test error/edge cases, not just happy path
- Use `@MainActor` for ViewModel tests

### Don't
- Snapshot tests (fragile)
- Implementation detail tests
- Share state between tests
- Use `sleep()` / `Task.sleep()` for timing -- use expectations
- Test private methods directly
- Use XCTAssert with complex boolean expressions (prefer specific assertions)

---

## Run and Verify

```bash
# Xcode project
xcodebuild test -workspace {Name}.xcworkspace -scheme {Scheme} -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 16' -resultBundlePath TestResults.xcresult

# Swift Package
swift test --verbose
swift test --filter "FeatureViewModelTests"

# Coverage
xcodebuild test -scheme {Scheme} -enableCodeCoverage YES
xcrun xccov view --report TestResults.xcresult
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

-> All tests pass -> Complete
-> Failures -> Run `/swift-04-fix` again
```
