---
name: android-mvvm
description: Guides implementation of Model-View-ViewModel (MVVM) architecture for Android. Use when structuring Android apps, creating ViewModels, implementing data binding, or separating UI logic from business logic. Covers ViewModel, LiveData, StateFlow, and UI layer patterns.
tags: ["android", "mvvm", "architecture", "viewmodel", "livedata", "stateflow", "ui"]
difficulty: intermediate
category: architecture
version: "1.0.0"
last_updated: "2025-01-29"
---

# Android MVVM Architecture

## Quick Start

Add ViewModel dependency to `build.gradle`:

```kotlin
dependencies {
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.6.2")
    implementation("androidx.lifecycle:lifecycle-livedata-ktx:2.6.2")
    implementation("androidx.activity:activity-ktx:1.8.0")
}
```

Enable ViewBinding:

```kotlin
android {
    buildFeatures {
        viewBinding = true
    }
}
```

## Core Patterns

### Basic MVVM Structure

```
ui/
├── UserViewModel.kt      # Exposes data, handles UI logic
├── UserFragment.kt       # Observes data, handles UI events
└── UserAdapter.kt        # RecyclerView adapter

data/
├── UserRepository.kt     # Single source of truth
└── remote/UserApi.kt     # Data sources
```

### ViewModel with StateFlow

```kotlin
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {

    private val _uiState = MutableStateFlow(UserUiState())
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()

    fun loadUser(userId: String) {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            
            try {
                val user = repository.getUser(userId)
                _uiState.update { 
                    it.copy(user = user, isLoading = false, error = null) 
                }
            } catch (e: Exception) {
                _uiState.update { 
                    it.copy(error = e.message, isLoading = false) 
                }
            }
        }
    }
}

data class UserUiState(
    val user: User? = null,
    val isLoading: Boolean = false,
    val error: String? = null
)
```

### Fragment/Activity observing ViewModel

```kotlin
class UserFragment : Fragment() {

    private val viewModel: UserViewModel by viewModels()
    private var _binding: FragmentUserBinding? = null
    private val binding get() = _binding!!

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // Collect StateFlow
        viewLifecycleOwner.lifecycleScope.launch {
            viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    updateUI(state)
                }
            }
        }

        // Send events to ViewModel
        binding.refreshButton.setOnClickListener {
            viewModel.loadUser("123")
        }
    }

    private fun updateUI(state: UserUiState) {
        binding.progressBar.isVisible = state.isLoading
        binding.errorText.isVisible = state.error != null
        binding.errorText.text = state.error
        state.user?.let { binding.userName.text = it.name }
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null  // Prevent memory leaks
    }
}
```

### Repository Pattern

```kotlin
class UserRepository(
    private val remoteDataSource: UserApi,
    private val localDataSource: UserDao
) {
    suspend fun getUser(id: String): User {
        return localDataSource.getUser(id) 
            ?: remoteDataSource.fetchUser(id).also {
                localDataSource.insert(it)
            }
    }

    fun getUsers(): Flow<List<User>> = localDataSource.getAllUsers()
}
```

## Common Patterns

### Single UI State (Recommended)

One StateFlow holding entire UI state:

```kotlin
data class NewsUiState(
    val articles: List<Article> = emptyList(),
    val isLoading: Boolean = false,
    val errorMessage: String? = null,
    val selectedArticle: Article? = null
)

class NewsViewModel(private val repository: NewsRepository) : ViewModel() {
    
    private val _uiState = MutableStateFlow(NewsUiState())
    val uiState: StateFlow<NewsUiState> = _uiState.asStateFlow()

    fun loadNews() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true, errorMessage = null) }
            
            try {
                val articles = repository.fetchNews()
                _uiState.update { 
                    it.copy(articles = articles, isLoading = false) 
                }
            } catch (e: Exception) {
                _uiState.update { 
                    it.copy(errorMessage = e.message, isLoading = false) 
                }
            }
        }
    }

    fun selectArticle(article: Article) {
        _uiState.update { it.copy(selectedArticle = article) }
    }
}
```

### Multiple LiveData/StateFlow (Legacy)

Separate streams for different data types:

```kotlin
class LegacyViewModel : ViewModel() {
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users

    private val _loading = MutableLiveData<Boolean>()
    val loading: LiveData<Boolean> = _loading

    private val _error = MutableLiveData<String>()
    val error: LiveData<String> = _error
}
```

### Shared ViewModel

Share between Fragment and parent Activity:

```kotlin
// Activity-scoped
class MasterFragment : Fragment() {
    private val sharedViewModel: SharedViewModel by activityViewModels()
}

// Same instance in detail fragment
class DetailFragment : Fragment() {
    private val sharedViewModel: SharedViewModel by activityViewModels()
}
```

### ViewModel Factory

For ViewModels with constructor parameters:

```kotlin
class UserViewModelFactory(
    private val repository: UserRepository
) : ViewModelProvider.Factory {
    
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(UserViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return UserViewModel(repository) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}

// Usage
val viewModel: UserViewModel by viewModels {
    UserViewModelFactory(repository)
}
```

### Saved State Handle

Persist data across process death:

```kotlin
class SearchViewModel(
    savedStateHandle: SavedStateHandle,
    private val repository: SearchRepository
) : ViewModel() {

    private val _query = savedStateHandle.getStateFlow("query", "")
    val query: StateFlow<String> = _query

    fun setQuery(newQuery: String) {
        savedStateHandle["query"] = newQuery
    }
}
```

## Examples (Input → Output)

### Complete MVVM Setup

**Input request:** "Create a complete MVVM setup for a product list screen with loading and error states"

**Output:**

```kotlin
// Data layer
class ProductRepository(private val api: ProductApi) {
    suspend fun getProducts(): List<Product> = api.fetchProducts()
}

// UiState
data class ProductsUiState(
    val products: List<Product> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)

// ViewModel
class ProductsViewModel(
    private val repository: ProductRepository
) : ViewModel() {

    private val _uiState = MutableStateFlow(ProductsUiState())
    val uiState: StateFlow<ProductsUiState> = _uiState.asStateFlow()

    init {
        loadProducts()
    }

    fun loadProducts() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true, error = null) }
            
            try {
                val products = repository.getProducts()
                _uiState.update { it.copy(products = products, isLoading = false) }
            } catch (e: Exception) {
                _uiState.update { it.copy(error = e.message, isLoading = false) }
            }
        }
    }
}

// Fragment
class ProductsFragment : Fragment() {

    private val viewModel: ProductsViewModel by viewModels()
    private lateinit var binding: FragmentProductsBinding
    private lateinit var adapter: ProductAdapter

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding = FragmentProductsBinding.bind(view)
        
        setupRecyclerView()
        observeViewModel()
        
        binding.retryButton.setOnClickListener {
            viewModel.loadProducts()
        }
    }

    private fun observeViewModel() {
        viewLifecycleOwner.lifecycleScope.launch {
            viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    binding.progressBar.isVisible = state.isLoading
                    binding.errorGroup.isVisible = state.error != null
                    binding.recyclerView.isVisible = state.error == null && !state.isLoading
                    
                    adapter.submitList(state.products)
                    state.error?.let { binding.errorText.text = it }
                }
            }
        }
    }
}
```

### Converting from MVP to MVVM

**Input request:** "Convert this MVP Presenter to MVVM ViewModel"

**Given MVP code:**
```kotlin
class UserPresenter(
    private val view: UserView,
    private val repository: UserRepository
) {
    fun loadUser(id: String) {
        view.showLoading()
        scope.launch {
            try {
                val user = repository.getUser(id)
                view.showUser(user)
            } catch (e: Exception) {
                view.showError(e.message)
            }
        }
    }
}
```

**Output:**
```kotlin
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {

    private val _uiState = MutableStateFlow(UserUiState())
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()

    fun loadUser(id: String) {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true, error = null) }
            
            try {
                val user = repository.getUser(id)
                _uiState.update { 
                    it.copy(user = user, isLoading = false) 
                }
            } catch (e: Exception) {
                _uiState.update { 
                    it.copy(error = e.message, isLoading = false) 
                }
            }
        }
    }
}

data class UserUiState(
    val user: User? = null,
    val isLoading: Boolean = false,
    val error: String? = null
)
```

## Best Practices

1. **Single UI State**: Use one StateFlow for entire screen state (easier to manage, test)
2. **Unidirectional Data Flow**: UI emits events → ViewModel updates state → UI observes state
3. **No Android framework in ViewModel**: Don't hold Context, View, or Lifecycle in ViewModel
4. **ViewModel survives config changes**: Put logic that should survive rotation in ViewModel
5. **Expose immutable types**: Use `StateFlow` (not `MutableStateFlow`) for public properties
6. **Use `repeatOnLifecycle`**: Always collect flows in `STARTED` state to prevent waste
7. **Clear bindings in Fragment**: Set binding to null in `onDestroyView()` to prevent leaks
8. **ViewModel shouldn't know about UI**: No references to Fragments, Activities, or Views
9. **Repository pattern**: Single source of truth, abstracts data sources from ViewModel
10. **Test ViewModels**: Business logic in ViewModel = easy unit testing

## Resources

- [Guide to app architecture](https://developer.android.com/topic/architecture)
- [ViewModel overview](https://developer.android.com/topic/libraries/architecture/viewmodel)
- [StateFlow and SharedFlow](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow)
- [UI Layer guide](https://developer.android.com/topic/architecture/ui-layer)
