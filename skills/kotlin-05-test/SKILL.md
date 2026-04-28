---
name: kotlin-05-test
description: "Android/Kotlin test writing and execution. Triggers on 'kotlin test', '코틀린 테스트', 'android test', '안드로이드 테스트', 'kt test', 'android unit test'."
model: sonnet
tools: Read, Bash, Glob, Grep, Write, Edit
---

# Kotlin Test

Write and run tests for Android/Kotlin features.

**Agents:** test-strategy-expert, kotlin-android-expert

## Pre-Test

1. Implementation complete? (`/kotlin-02-implement`)
2. Review passed? (`/kotlin-03-review` → `/kotlin-04-fix`)
3. Files to test identified

---

## Kotlin Test Stack

```
JUnit 5 + MockK                  # Unit tests
Turbine                           # Flow testing
Robolectric                       # Android framework in JVM
Compose UI Test                   # Composable testing
TestDispatcher                    # Coroutine testing
```

**Maintain 100% compatibility with existing test stack and patterns.**

---

## Test Strategy

1. **Analyze implemented files** — prioritize by criticality:
   - [Critical] ViewModel → state changes, events, error handling
   - [Critical] Repository → data mapping, caching logic, error propagation
   - [Critical] UseCase → business rules, edge cases
   - [Important] Mapper → transformation correctness
   - [Important] Screen (Composable) → UI state rendering, user interactions
   - [Nice] Component (optional)

2. **Mocking targets**:
   - Repository → MockK (`every { }`, `coEvery { }`)
   - API Service → MockK or MockWebServer
   - Room DAO → in-memory database
   - DataStore → test DataStore
   - Dispatchers → TestDispatcher (StandardTestDispatcher or UnconfinedTestDispatcher)

---

## Test Patterns

### ViewModel Test (Required)

```kotlin
class FeatureViewModelTest {
    private val repository: FeatureRepository = mockk()
    private lateinit var viewModel: FeatureViewModel

    @BeforeEach
    fun setup() {
        viewModel = FeatureViewModel(repository)
    }

    @Test
    fun `should load data successfully`() = runTest {
        // Arrange
        coEvery { repository.getData() } returns Result.success(testData)

        // Act
        viewModel.loadData()

        // Assert
        viewModel.uiState.test {
            val state = awaitItem()
            assertEquals(expected, state.data)
        }
    }

    @Test
    fun `should handle error gracefully`() = runTest {
        coEvery { repository.getData() } returns Result.failure(IOException())

        viewModel.loadData()

        viewModel.uiState.test {
            val state = awaitItem()
            assertTrue(state.isError)
        }
    }
}
```

### Repository Test (Required)

```kotlin
class FeatureRepositoryTest {
    private val apiService: ApiService = mockk()
    private val dao: FeatureDao = mockk()
    private lateinit var repository: FeatureRepository

    @BeforeEach
    fun setup() {
        repository = FeatureRepositoryImpl(apiService, dao)
    }

    @Test
    fun `should return cached data when available`() = runTest {
        coEvery { dao.getAll() } returns flowOf(testEntities)

        val result = repository.getData().first()

        assertEquals(expected, result)
        coVerify(exactly = 0) { apiService.fetchData() }
    }
}
```

---

## Test Writing Rules

### Do
- Backtick naming (`` `should [action] when [condition]` ``)
- AAA pattern (Arrange-Act-Assert) required
- `runTest` for all coroutine tests
- Turbine `.test { }` for Flow assertions
- TestDispatcher for coroutine control
- Test error/edge cases (empty list, null, network error)
- `@BeforeEach` setup, clean state per test

### 🚫 Don't
- Snapshot tests
- Implementation detail tests (testing private methods)
- Share state between tests
- `delay()` / `Thread.sleep()` → use `advanceUntilIdle()`
- Test Android framework directly in unit tests (use Robolectric)
- `any` type mocking

---

## Run and Verify

```bash
./gradlew test{Variant}UnitTest --tests "*FeatureName*"
./gradlew jacocoTestReport
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
→ Failures → `/kotlin-04-fix`
```
