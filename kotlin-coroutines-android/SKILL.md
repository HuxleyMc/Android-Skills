---
name: kotlin-coroutines-android
description: Guides implementation of Kotlin coroutines for Android asynchronous programming. Use when writing async code, handling background operations, implementing ViewModels with suspend functions, or converting callbacks to coroutines. Covers structured concurrency, main-safety, Flow, and Jetpack integration.
tags: ["kotlin", "coroutines", "android", "async", "viewmodel", "flow"]
difficulty: intermediate
category: architecture
version: "1.0.0"
last_updated: "2025-01-29"
---

# Kotlin Coroutines for Android

## Quick Start

Add dependencies to `build.gradle`:

```kotlin
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.6.2")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.6.2")
}
```

## Core Patterns

### Basic Coroutine Launch

**Launch (fire-and-forget):**
```kotlin
viewModelScope.launch {
    repository.saveData(data)
}
```

**Async/await (return value):**
```kotlin
viewModelScope.launch {
    val user = async { repository.fetchUser(id) }.await()
    updateUI(user)
}
```

### Dispatchers

Use `Dispatchers.IO` for network/database, `Dispatchers.Default` for computation:

```kotlin
suspend fun fetchUser(id: String): User = withContext(Dispatchers.IO) {
    api.getUser(id)  // Main-safe function
}

// Usage: Called from any dispatcher
viewModelScope.launch {
    val user = fetchUser(id)  // Suspends, doesn't block
    updateUI(user)  // Back on Main
}
```

### Scopes

**ViewModel scope** (auto-cancels on clear):
```kotlin
class MyViewModel : ViewModel() {
    fun load() {
        viewModelScope.launch {
            // Work here
        }
    }
}
```

**Lifecycle scope** (Activity/Fragment):
```kotlin
class MyActivity : AppCompatActivity() {
    fun load() {
        lifecycleScope.launch {
            // Work here
        }
    }
}
```

## Common Patterns

### Repository Pattern

```kotlin
class UserRepository(private val api: UserApi, private val dao: UserDao) {
    
    suspend fun getUser(id: String): User = withContext(Dispatchers.IO) {
        dao.getUser(id) ?: api.fetchUser(id).also { dao.insert(it) }
    }
    
    fun getAllUsers(): Flow<List<User>> = dao.getAllUsers()
        .flowOn(Dispatchers.IO)
}
```

### Parallel Decomposition

```kotlin
suspend fun loadDashboard(): Dashboard = coroutineScope {
    val user = async { fetchUser() }
    val orders = async { fetchOrders() }
    val notifications = async { fetchNotifications() }
    
    Dashboard(user.await(), orders.await(), notifications.await())
}
```

### Error Handling

```kotlin
viewModelScope.launch {
    val result = try {
        repository.fetchData()
    } catch (e: IOException) {
        Result.Error("Network failed")
    }
    
    when (result) {
        is Result.Success -> display(result.data)
        is Result.Error -> showError(result.message)
    }
}
```

### Flow + StateFlow

```kotlin
// Repository
fun getUsers(): Flow<List<User>> = flow {
    while (true) {
        emit(api.fetchUsers())
        delay(5000)
    }
}.flowOn(Dispatchers.IO)

// ViewModel
val users: StateFlow<List<User>> = repository.getUsers()
    .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), emptyList())

// UI
lifecycleScope.launch {
    viewModel.users.collect { adapter.submitList(it) }
}
```

### Cancellation

```kotlin
viewModelScope.launch {
    try {
        while (isActive) {  // Check cancellation
            doWork()
        }
    } catch (e: CancellationException) {
        throw e  // Re-throw to propagate
    } finally {
        closeResources()  // Always cleanup
    }
}
```

### Timeout

```kotlin
// Throw on timeout
val result = withTimeout(5000) { fetchData() }

// Null on timeout
val result = withTimeoutOrNull(5000) { fetchData() }
```

## Examples (Input → Output)

### ViewModel with Loading State

**Input request:** "Create a ViewModel that loads news with loading state"

**Output:**
```kotlin
class NewsViewModel(private val repo: NewsRepository) : ViewModel() {
    private val _news = MutableStateFlow<List<Article>>(emptyList())
    val news: StateFlow<List<Article>> = _news.asStateFlow()
    
    private val _loading = MutableStateFlow(false)
    val loading: StateFlow<Boolean> = _loading.asStateFlow()
    
    fun load() {
        viewModelScope.launch {
            _loading.value = true
            try {
                _news.value = repo.fetchNews()
            } catch (e: Exception) {
                // Handle error
            } finally {
                _loading.value = false
            }
        }
    }
}
```

### Callback to Coroutine Conversion

**Input request:** "Convert this callback-based API to suspend function"

**Given callback API:**
```kotlin
fun fetchUser(id: String, callback: (Result<User>) -> Unit)
```

**Output:**
```kotlin
suspend fun fetchUser(id: String): User = suspendCancellableCoroutine { continuation ->
    fetchUser(id) { result ->
        when (result) {
            is Result.Success -> continuation.resume(result.data)
            is Result.Error -> continuation.resumeWithException(result.exception)
        }
    }
    continuation.invokeOnCancellation { cancelRequest(id) }
}
```

## Best Practices

1. **Use structured concurrency**: Always launch in `viewModelScope` or `lifecycleScope`
2. **Create main-safe functions**: Wrap blocking calls in `withContext(Dispatchers.IO)`
3. **Handle exceptions**: Use try-catch in coroutines
4. **Respect cancellation**: Check `isActive` in loops, clean up in `finally`
5. **Use appropriate dispatchers**: Main for UI, IO for network/database, Default for CPU
6. **Use Flow for streams**: Replace callbacks with Flow for reactive data
7. **Never use `runBlocking`** in production (tests only)

## Resources

- [Android Coroutines Guide](https://developer.android.com/kotlin/coroutines)
- [Kotlin Coroutines Overview](https://kotlinlang.org/docs/coroutines-overview.html)
- [Coroutines Best Practices](https://developer.android.com/kotlin/coroutines/coroutines-best-practices)
