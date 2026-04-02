---
name: kotlin-05-test
description: "Android/Kotlin test writing and execution. Triggers on 'kotlin test', '코틀린 테스트', 'android test', '안드로이드 테스트', 'kt test', 'android unit test'."
tools: Read, Bash, Glob, Grep, Write, Edit
---

# Kotlin Test

Write and run tests for Android/Kotlin features.

**Agents:** test-strategy-expert, kotlin-android-expert

## Pre-Test

1. Implementation complete? (`/kotlin-02-implement`)
2. Review passed? (`/kotlin-03-review` -> `/kotlin-04-fix`)
3. Files to test identified

---

## Android Test Stack

```
JUnit 5 / JUnit 4               # Test framework
MockK                            # Kotlin-idiomatic mocking
Turbine                          # Flow testing
Kotlin Coroutines Test           # Coroutine testing (runTest, TestDispatcher)
Robolectric                      # Android framework unit tests
Compose UI Test                  # Compose component testing
Espresso                         # Instrumented UI tests (legacy View)
```

**Maintain 100% compatibility with existing test stack and patterns.**

---

## Test Strategy

1. **Analyze implemented files**
   - ViewModel -> [Critical] Required test
   - Repository -> [Critical] Required test
   - UseCase -> [Critical] Required test
   - Mapper/Converter -> [Critical] Required test (pure functions)
   - Screen (Composable) -> [Important] UI test
   - Component (Composable) -> [Nice] Optional

2. **Determine mocking targets**
   - Repository -> MockK mock
   - API Service -> MockK mock or MockWebServer
   - Room DAO -> In-memory Room database
   - DataStore -> Test DataStore
   - Dispatchers -> TestDispatcher

---

## Test Patterns

### ViewModel Test (Required)

```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
class FeatureViewModelTest {
    private val testDispatcher = UnconfinedTestDispatcher()
    private val repository: FeatureRepository = mockk()
    private lateinit var viewModel: FeatureViewModel

    @BeforeEach
    fun setup() {
        Dispatchers.setMain(testDispatcher)
        viewModel = FeatureViewModel(repository)
    }

    @AfterEach
    fun tearDown() {
        Dispatchers.resetMain()
    }

    @Test
    fun `should load items successfully`() = runTest {
        // Arrange
        val items = listOf(FeatureItem(id = "1", name = "Test"))
        coEvery { repository.getItems() } returns Result.success(items)

        // Act
        viewModel.onEvent(FeatureEvent.LoadItems)

        // Assert
        viewModel.uiState.test {
            val state = awaitItem()
            assertThat(state.items).isEqualTo(items)
            assertThat(state.isLoading).isFalse()
        }
    }

    @Test
    fun `should handle error when loading fails`() = runTest {
        // Arrange
        coEvery { repository.getItems() } returns Result.failure(Exception("Network error"))

        // Act
        viewModel.onEvent(FeatureEvent.LoadItems)

        // Assert
        viewModel.uiState.test {
            val state = awaitItem()
            assertThat(state.error).isNotNull()
            assertThat(state.isLoading).isFalse()
        }
    }
}
```

### Repository Test (Required)

```kotlin
class FeatureRepositoryImplTest {
    private val remoteDataSource: FeatureRemoteDataSource = mockk()
    private val localDataSource: FeatureLocalDataSource = mockk()
    private val repository = FeatureRepositoryImpl(remoteDataSource, localDataSource)

    @Test
    fun `should fallback to cache when remote fails`() = runTest {
        // Arrange
        coEvery { remoteDataSource.fetchItems() } throws IOException("Network error")
        val cachedItems = listOf(FeatureEntity(id = "1", name = "Cached"))
        coEvery { localDataSource.getItems() } returns cachedItems

        // Act
        val result = repository.getItems()

        // Assert
        assertThat(result.isSuccess).isTrue()
        assertThat(result.getOrNull()?.first()?.name).isEqualTo("Cached")
    }
}
```

### Compose UI Test (Optional)

```kotlin
@OptIn(ExperimentalTestApi::class)
class FeatureScreenTest {
    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun `should display items when loaded`() {
        // Arrange
        val items = listOf(FeatureItem(id = "1", name = "Test Item"))

        // Act
        composeTestRule.setContent {
            FeatureContent(
                uiState = FeatureUiState(items = items),
                onEvent = {}
            )
        }

        // Assert
        composeTestRule.onNodeWithText("Test Item").assertIsDisplayed()
    }
}
```

---

## Test Writing Rules

### Do
- Behavior-based test naming (backtick style: `` `should [action] when [condition]` ``)
- AAA pattern (Arrange-Act-Assert) required
- Use `runTest` for coroutine tests
- Use Turbine for Flow testing
- Use TestDispatcher for coroutine dispatcher replacement
- Test error/edge cases, not just happy path

### Don't
- Snapshot tests
- Implementation detail tests (verify internal method calls)
- Share state between tests
- Use `delay()` for timing -- use `advanceUntilIdle()` or Turbine
- Test Android framework code in unit tests (use Robolectric if needed)
- Use `Thread.sleep()` -- use `runTest` + `advanceTimeBy()`

---

## Run and Verify

```bash
./gradlew test{Variant}UnitTest --tests "*FeatureName*"
./gradlew test{Variant}UnitTest --info     # verbose
./gradlew jacocoTestReport                  # coverage (if configured)
./gradlew connectedAndroidTest              # instrumented (device needed)
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
-> Failures -> Run `/kotlin-04-fix` again
```
