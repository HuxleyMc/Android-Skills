---
name: android-code-review
description: Performs thorough code reviews on Android Kotlin/Java code. Use when reviewing pull requests, analyzing code quality, checking architecture patterns, or validating Android best practices. Covers MVVM/MVI patterns, Compose, Coroutines, security, and performance.
tags: ["android", "kotlin", "code-review", "quality", "security", "performance", "mvvm", "compose", "coroutines"]
difficulty: intermediate
category: quality-assurance
version: "1.0.0"
last_updated: "2025-01-29"
---

# Android Code Review

## Quick Start

Review code systematically across these areas:

```
1. Architecture & Patterns
2. Kotlin Idioms & Conventions
3. Compose UI Best Practices
4. Coroutines & Flow
5. Performance & Memory
6. Security
7. Testing Coverage
```

## Review Categories

### Architecture & Patterns

**MVVM Pattern Check:**
```kotlin
// Good: ViewModel delegates to UseCases/Repositories
class UserViewModel(
    private val getUserUseCase: GetUserUseCase,
    private val saveUserUseCase: SaveUserUseCase
) : ViewModel() {
    
    private val _uiState = MutableStateFlow(UserUiState())
    val uiState = _uiState.asStateFlow()
    
    fun loadUser(id: String) {
        viewModelScope.launch {
            getUserUseCase(id)
                .onSuccess { _uiState.update { it.copy(user = user) } }
                .onFailure { _uiState.update { it.copy(error = it.message) } }
        }
    }
}

// Bad: Business logic in ViewModel
class BadViewModel : ViewModel() {
    fun calculateDiscount(price: Double, userType: String): Double {
        // Business logic should be in UseCase
        return when (userType) {
            "VIP" -> price * 0.8
            else -> price
        }
    }
}
```

### Kotlin Idioms

**Prefer Kotlin conventions:**
```kotlin
// Good
val name: String = user?.name ?: "Unknown"

// Bad
val name: String = if (user != null) user.name else "Unknown"

// Good
users.filter { it.isActive }
     .sortedBy { it.name }
     .map { it.toUiModel() }
```

### Jetpack Compose

**State Management:**
```kotlin
// Good: Hoist state appropriately
@Composable
fun UserScreen(
    viewModel: UserViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    
    UserContent(
        user = uiState.user,
        isLoading = uiState.isLoading,
        onRefresh = viewModel::refresh
    )
}

@Composable
private fun UserContent(
    user: User?,
    isLoading: Boolean,
    onRefresh: () -> Unit
) {
    // Stateless component, easy to preview and test
}
```

### Coroutines & Flow

**Proper Scope Usage:**
```kotlin
// Good: Use viewModelScope for ViewModel
class UserViewModel : ViewModel() {
    fun loadData() {
        viewModelScope.launch {
            // Coroutine cancelled when ViewModel cleared
        }
    }
}

// Bad: GlobalScope - never use in production
GlobalScope.launch { } // ❌
```

## Examples (Input → Output)

### PR Review Example

**Input:** Review this ViewModel code

```kotlin
class UserViewModel : ViewModel() {
    var user: User? = null
    val loading = MutableLiveData<Boolean>()
    
    fun load() {
        viewModelScope.launch {
            loading.value = true
            user = repository.getUser()
            loading.value = false
        }
    }
}
```

**Output:**

```markdown
## Review Comments

**[Critical]** MutableLiveData exposure
- Line 3: `loading` is mutable but should be immutable
- Fix: `val loading: LiveData<Boolean> = _loading`

**[Important]** State management
- Lines 2-3: Mixed state types (nullable + LiveData)
- Suggestion: Use StateFlow for consistency
```

## Resources

- [Android Kotlin Style Guide](https://developer.android.com/kotlin/style-guide)
- [Compose Guidelines](https://developer.android.com/jetpack/compose/guidelines)
