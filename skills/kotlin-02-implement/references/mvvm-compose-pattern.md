# MVVM + Jetpack Compose Pattern Reference

## UiState Pattern

```kotlin
// Sealed interface for type-safe state
sealed interface FeatureUiState {
    data object Loading : FeatureUiState
    data class Success(
        val items: List<FeatureItem> = emptyList(),
        val isRefreshing: Boolean = false,
    ) : FeatureUiState
    data class Error(val message: String) : FeatureUiState
}

// Alternative: Single data class with loading/error fields
data class FeatureUiState(
    val items: List<FeatureItem> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null,
    val isRefreshing: Boolean = false,
)
```

## UiEvent Pattern

```kotlin
sealed interface FeatureEvent {
    data object LoadItems : FeatureEvent
    data object Refresh : FeatureEvent
    data class SelectItem(val id: String) : FeatureEvent
    data class DeleteItem(val id: String) : FeatureEvent
    data class SearchQueryChanged(val query: String) : FeatureEvent
}
```

## UiEffect Pattern (One-shot side effects)

```kotlin
sealed class FeatureEffect {
    data class ShowSnackbar(val message: String) : FeatureEffect()
    data class NavigateToDetail(val id: String) : FeatureEffect()
    data object NavigateBack : FeatureEffect()
}
```

## ViewModel Pattern

```kotlin
@HiltViewModel
class FeatureViewModel @Inject constructor(
    private val repository: FeatureRepository,
    private val savedStateHandle: SavedStateHandle,
) : ViewModel() {

    // Private mutable, public immutable
    private val _uiState = MutableStateFlow(FeatureUiState())
    val uiState: StateFlow<FeatureUiState> = _uiState.asStateFlow()

    // One-shot effects via Channel
    private val _effect = Channel<FeatureEffect>()
    val effect: Flow<FeatureEffect> = _effect.receiveAsFlow()

    init {
        onEvent(FeatureEvent.LoadItems)
    }

    fun onEvent(event: FeatureEvent) {
        when (event) {
            is FeatureEvent.LoadItems -> loadItems()
            is FeatureEvent.Refresh -> refresh()
            is FeatureEvent.SelectItem -> selectItem(event.id)
            is FeatureEvent.DeleteItem -> deleteItem(event.id)
            is FeatureEvent.SearchQueryChanged -> updateSearch(event.query)
        }
    }

    private fun loadItems() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true, error = null) }
            repository.getItems()
                .onSuccess { items ->
                    _uiState.update { it.copy(items = items, isLoading = false) }
                }
                .onFailure { error ->
                    _uiState.update { it.copy(error = error.message, isLoading = false) }
                    _effect.send(FeatureEffect.ShowSnackbar(error.message ?: "Unknown error"))
                }
        }
    }

    private fun deleteItem(id: String) {
        viewModelScope.launch {
            repository.deleteItem(id)
                .onSuccess {
                    _uiState.update { state ->
                        state.copy(items = state.items.filter { it.id != id })
                    }
                    _effect.send(FeatureEffect.ShowSnackbar("Deleted successfully"))
                }
                .onFailure { error ->
                    _effect.send(FeatureEffect.ShowSnackbar(error.message ?: "Delete failed"))
                }
        }
    }
}
```

## Screen (Smart Composable) Pattern

```kotlin
@Composable
fun FeatureScreen(
    viewModel: FeatureViewModel = hiltViewModel(),
    onNavigateToDetail: (String) -> Unit,
    onNavigateBack: () -> Unit,
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val snackbarHostState = remember { SnackbarHostState() }

    // Collect one-shot effects
    LaunchedEffect(Unit) {
        viewModel.effect.collect { effect ->
            when (effect) {
                is FeatureEffect.ShowSnackbar -> {
                    snackbarHostState.showSnackbar(effect.message)
                }
                is FeatureEffect.NavigateToDetail -> {
                    onNavigateToDetail(effect.id)
                }
                is FeatureEffect.NavigateBack -> {
                    onNavigateBack()
                }
            }
        }
    }

    FeatureContent(
        uiState = uiState,
        snackbarHostState = snackbarHostState,
        onEvent = viewModel::onEvent,
    )
}
```

## Content (Presentational Composable) Pattern

```kotlin
@Composable
fun FeatureContent(
    uiState: FeatureUiState,
    snackbarHostState: SnackbarHostState,
    onEvent: (FeatureEvent) -> Unit,
    modifier: Modifier = Modifier,
) {
    Scaffold(
        snackbarHost = { SnackbarHost(snackbarHostState) },
        topBar = {
            TopAppBar(title = { Text("Features") })
        },
    ) { padding ->
        when {
            uiState.isLoading -> {
                Box(
                    modifier = Modifier.fillMaxSize().padding(padding),
                    contentAlignment = Alignment.Center,
                ) {
                    CircularProgressIndicator()
                }
            }
            uiState.error != null -> {
                ErrorContent(
                    message = uiState.error,
                    onRetry = { onEvent(FeatureEvent.LoadItems) },
                    modifier = Modifier.padding(padding),
                )
            }
            else -> {
                LazyColumn(
                    modifier = Modifier.padding(padding),
                    contentPadding = PaddingValues(16.dp),
                    verticalArrangement = Arrangement.spacedBy(8.dp),
                ) {
                    items(
                        items = uiState.items,
                        key = { it.id },
                    ) { item ->
                        FeatureItemCard(
                            item = item,
                            onClick = { onEvent(FeatureEvent.SelectItem(item.id)) },
                            onDelete = { onEvent(FeatureEvent.DeleteItem(item.id)) },
                        )
                    }
                }
            }
        }
    }
}
```

## Repository Pattern

```kotlin
// Domain layer - interface
interface FeatureRepository {
    suspend fun getItems(): Result<List<FeatureItem>>
    suspend fun getItem(id: String): Result<FeatureItem>
    suspend fun createItem(body: CreateFeatureBody): Result<FeatureItem>
    suspend fun deleteItem(id: String): Result<Unit>
}

// Data layer - implementation
class FeatureRepositoryImpl @Inject constructor(
    private val remoteDataSource: FeatureRemoteDataSource,
    private val localDataSource: FeatureLocalDataSource,
    private val dispatcher: CoroutineDispatcher,
) : FeatureRepository {

    override suspend fun getItems(): Result<List<FeatureItem>> =
        withContext(dispatcher) {
            runCatching {
                val remoteItems = remoteDataSource.fetchItems()
                val domainItems = remoteItems.map { it.toDomain() }
                localDataSource.insertItems(domainItems.map { it.toEntity() })
                domainItems
            }.recoverCatching {
                // Fallback to cache
                localDataSource.getItems().map { it.toDomain() }
            }
        }
}
```

## Hilt Module Pattern

```kotlin
@Module
@InstallIn(ViewModelComponent::class)
abstract class FeatureModule {

    @Binds
    abstract fun bindFeatureRepository(
        impl: FeatureRepositoryImpl,
    ): FeatureRepository
}

@Module
@InstallIn(SingletonComponent::class)
object FeatureNetworkModule {

    @Provides
    fun provideFeatureApiService(
        retrofit: Retrofit,
    ): FeatureApiService = retrofit.create(FeatureApiService::class.java)
}
```

## Navigation Pattern (Compose)

```kotlin
// Navigation route
object FeatureRoute {
    const val ROUTE = "feature"
    const val DETAIL_ROUTE = "feature/{id}"

    fun detailRoute(id: String) = "feature/$id"
}

// NavGraph extension
fun NavGraphBuilder.featureNavGraph(
    navController: NavHostController,
) {
    composable(FeatureRoute.ROUTE) {
        FeatureScreen(
            onNavigateToDetail = { id ->
                navController.navigate(FeatureRoute.detailRoute(id))
            },
            onNavigateBack = { navController.popBackStack() },
        )
    }
    composable(
        route = FeatureRoute.DETAIL_ROUTE,
        arguments = listOf(navArgument("id") { type = NavType.StringType }),
    ) {
        FeatureDetailScreen(
            onNavigateBack = { navController.popBackStack() },
        )
    }
}
```

## Mapper Pattern

```kotlin
// DTO -> Domain
fun FeatureDto.toDomain(): FeatureItem = FeatureItem(
    id = id,
    name = name,
    description = description,
    createdAt = Instant.parse(createdAt),
)

// Entity -> Domain
fun FeatureEntity.toDomain(): FeatureItem = FeatureItem(
    id = id,
    name = name,
    description = description,
    createdAt = createdAt,
)

// Domain -> Entity
fun FeatureItem.toEntity(): FeatureEntity = FeatureEntity(
    id = id,
    name = name,
    description = description,
    createdAt = createdAt,
)
```
