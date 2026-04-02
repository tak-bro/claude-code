# Common Android/Kotlin Fixes

## [Critical] MutableStateFlow Exposed

```kotlin
// Before (BAD)
class FeatureViewModel : ViewModel() {
    val uiState = MutableStateFlow(FeatureUiState())
}

// After (GOOD)
class FeatureViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(FeatureUiState())
    val uiState: StateFlow<FeatureUiState> = _uiState.asStateFlow()
}
```

## [Critical] Business Logic in Composable

```kotlin
// Before (BAD)
@Composable
fun FeatureScreen(repository: FeatureRepository) {
    var items by remember { mutableStateOf(emptyList<Item>()) }
    LaunchedEffect(Unit) {
        items = repository.getItems() // Business logic in Composable!
    }
}

// After (GOOD)
@Composable
fun FeatureScreen(viewModel: FeatureViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    FeatureContent(uiState = uiState, onEvent = viewModel::onEvent)
}
```

## [Critical] Direct API Call from ViewModel

```kotlin
// Before (BAD)
@HiltViewModel
class FeatureViewModel @Inject constructor(
    private val apiService: FeatureApiService, // Direct API dependency!
) : ViewModel() {
    fun loadItems() {
        viewModelScope.launch {
            val items = apiService.fetchItems()
        }
    }
}

// After (GOOD)
@HiltViewModel
class FeatureViewModel @Inject constructor(
    private val repository: FeatureRepository, // Repository abstraction
) : ViewModel() {
    fun loadItems() {
        viewModelScope.launch {
            repository.getItems()
                .onSuccess { items -> _uiState.update { it.copy(items = items) } }
                .onFailure { error -> _uiState.update { it.copy(error = error.message) } }
        }
    }
}
```

## [Critical] Missing Hilt Injection

```kotlin
// Before (BAD)
class FeatureViewModel : ViewModel() {
    private val repository = FeatureRepositoryImpl() // Manual instantiation!
}

// After (GOOD)
@HiltViewModel
class FeatureViewModel @Inject constructor(
    private val repository: FeatureRepository,
) : ViewModel()
```

## [Critical] Blocking Main Thread

```kotlin
// Before (BAD)
class FeatureRepositoryImpl @Inject constructor(
    private val api: FeatureApiService,
) : FeatureRepository {
    override suspend fun getItems(): Result<List<Item>> {
        return runCatching { api.fetchItems() } // Runs on calling dispatcher
    }
}

// After (GOOD)
class FeatureRepositoryImpl @Inject constructor(
    private val api: FeatureApiService,
    @IoDispatcher private val dispatcher: CoroutineDispatcher,
) : FeatureRepository {
    override suspend fun getItems(): Result<List<Item>> =
        withContext(dispatcher) {
            runCatching { api.fetchItems() }
        }
}
```

## [Critical] Mutable UiState

```kotlin
// Before (BAD)
data class FeatureUiState(
    var items: MutableList<Item> = mutableListOf(), // Mutable!
    var isLoading: Boolean = false,
)

// After (GOOD)
data class FeatureUiState(
    val items: List<Item> = emptyList(), // Immutable
    val isLoading: Boolean = false,
)
```

## [Important] Missing Error Handling

```kotlin
// Before (BAD)
private fun loadItems() {
    viewModelScope.launch {
        val items = repository.getItems() // No error handling!
        _uiState.update { it.copy(items = items) }
    }
}

// After (GOOD)
private fun loadItems() {
    viewModelScope.launch {
        _uiState.update { it.copy(isLoading = true) }
        repository.getItems()
            .onSuccess { items ->
                _uiState.update { it.copy(items = items, isLoading = false) }
            }
            .onFailure { error ->
                _uiState.update { it.copy(error = error.message, isLoading = false) }
            }
    }
}
```

## [Important] Hardcoded Strings

```kotlin
// Before (BAD)
Text("Loading...")
Text("Error occurred")

// After (GOOD)
Text(stringResource(R.string.loading))
Text(stringResource(R.string.error_occurred))
```

## [Important] GlobalScope Usage

```kotlin
// Before (BAD)
GlobalScope.launch { // No lifecycle awareness!
    repository.syncData()
}

// After (GOOD - in ViewModel)
viewModelScope.launch {
    repository.syncData()
}

// After (GOOD - in Composable for one-shot)
LaunchedEffect(Unit) {
    // coroutine tied to composition lifecycle
}
```

## [Important] Missing remember / derivedStateOf

```kotlin
// Before (BAD)
@Composable
fun ItemList(items: List<Item>, filter: String) {
    val filtered = items.filter { it.name.contains(filter) } // Recalculated every recomposition!
    LazyColumn { items(filtered) { ... } }
}

// After (GOOD)
@Composable
fun ItemList(items: List<Item>, filter: String) {
    val filtered by remember(items, filter) {
        derivedStateOf { items.filter { it.name.contains(filter) } }
    }
    LazyColumn { items(filtered) { ... } }
}
```
