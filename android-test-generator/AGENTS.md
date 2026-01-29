# Android Test Generator - Agent Guide

## Quick Start

Use this skill when:
- Creating unit tests for ViewModels, Repositories, or UseCases
- Writing UI tests for Compose screens
- Setting up test data builders and fixtures
- Generating property-based tests
- Converting from JUnit to Kotest

## Common Tasks

### 1. Generate ViewModel Tests

**Input:** "Create tests for a LoginViewModel with email/password validation"

**Action:** Generate complete test class with:
- State testing (loading, success, error)
- Validation logic tests
- Coroutine handling with `advanceUntilIdle()`

```kotlin
class LoginViewModelTest : FunSpec({
    coroutineTestScope = true
    
    // Setup mocks
    // Test validation states
    // Test login flow
})
```

### 2. Generate Repository Tests

**Input:** "Create tests for UserRepository with local and remote data sources"

**Action:** Generate tests for:
- Cache hits
- Cache misses (fetch from remote)
- Network errors
- Data persistence

### 3. Generate Compose Tests

**Input:** "Create UI tests for a product detail screen"

**Action:** Generate tests for:
- Display assertions
- User interactions
- Navigation verification
- State changes

## Test Style Selection

| Component | Recommended Style |
|-----------|------------------|
| ViewModel | FunSpec with coroutineTestScope |
| Repository | BehaviorSpec for BDD-style |
| UseCase | FunSpec or StringSpec |
| Compose | JUnit with composeTestRule |

## MockK Patterns

```kotlin
// Suspend functions
coEvery { repository.fetch() } returns data
coVerify { repository.fetch() }

// Regular functions
every { validator.validate(any()) } returns true
verify(exactly = 1) { validator.validate(any()) }

// Argument matching
coEvery { api.getUser(match { it.isNotEmpty() }) } returns user
```

## Test Naming Conventions

```kotlin
// Behavior-focused names
test("emits loading then success state")
test("returns cached data when available")
test("debounces rapid search queries")

// Avoid implementation details
test("sets _isLoading to true") // ❌ Too specific
test("shows loading indicator")  // ✅ Behavior-focused
```

## Resources

- [Kotest Styles](https://kotest.io/docs/framework/testing-styles.html)
- [Coroutine Testing](https://kotest.io/docs/framework/coroutines/test-coroutines.html)
