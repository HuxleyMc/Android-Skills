---
name: android-test-generator
description: Generates comprehensive test suites for Android projects. Use when creating unit tests for ViewModels/Repositories, UI tests for Compose, integration tests, or property-based tests. Covers Kotest, JUnit, MockK, and coroutine testing.
tags: ["android", "testing", "kotest", "junit", "mockk", "compose-testing", "unit-test", "coroutines"]
difficulty: intermediate
category: testing
version: "1.0.0"
last_updated: "2025-01-29"
---

# Android Test Generator

## Quick Start

Generate tests based on component type:

```
1. ViewModel Tests (StateFlow, coroutines)
2. Repository Tests (data sources, caching)
3. UseCase Tests (business logic)
4. Compose UI Tests
5. Integration Tests
6. Property-Based Tests
```

## Test Templates

### ViewModel Tests

**Complete Test Class:**
```kotlin
@ExperimentalCoroutinesApi
class UserViewModelTest : FunSpec({

    coroutineTestScope = true

    lateinit var repository: UserRepository
    lateinit var viewModel: UserViewModel

    beforeEach {
        repository = mockk()
        viewModel = UserViewModel(repository)
    }

    context("loadUser") {

        test("emits loading then success state") {
            val user = User("1", "John", "john@test.com")
            coEvery { repository.getUser("1") } coAnswers {
                delay(100)
                user
            }

            viewModel.loadUser("1")

            viewModel.uiState.value.isLoading shouldBe true
            advanceTimeBy(100)

            assertSoftly(viewModel.uiState.value) {
                isLoading shouldBe false
                this.user?.name shouldBe "John"
                error shouldBe null
            }
        }

        test("emits error state on failure") {
            coEvery { repository.getUser(any()) } throws 
                IOException("Network error")

            viewModel.loadUser("1")
            advanceUntilIdle()

            assertSoftly(viewModel.uiState.value) {
                isLoading shouldBe false
                user shouldBe null
                error shouldBe "Network error"
            }
        }

        test("cancels previous load on new request") {
            coEvery { repository.getUser("1") } coAnswers {
                delay(1000)
                User("1", "Old")
            }
            coEvery { repository.getUser("2") } returns 
                User("2", "New")

            viewModel.loadUser("1")
            viewModel.loadUser("2")
            advanceUntilIdle()

            viewModel.uiState.value.user?.name shouldBe "New"
        }
    }

    context("search") {

        test("debounces search query") {
            val users = listOf(User("1", "John"))
            coEvery { repository.searchUsers(any()) } returns users

            viewModel.setQuery("j")
            viewModel.setQuery("jo")
            viewModel.setQuery("john")

            coVerify(exactly = 0) { repository.searchUsers(any()) }
            advanceTimeBy(300)
            coVerify(exactly = 1) { repository.searchUsers("john") }
        }
    }
})
```

### Repository Tests

**With MockK:**
```kotlin
class UserRepositoryTest : BehaviorSpec({

    val api = mockk<UserApi>()
    val dao = mockk<UserDao>()
    val repository = UserRepository(api, dao)

    given("getUser") {
        val userId = "123"
        val cachedUser = User(userId, "Cached User")
        val networkUser = User(userId, "Network User")

        `when`("user exists in cache") {
            coEvery { dao.getUser(userId) } returns cachedUser

            then("returns cached user without network call") {
                val result = repository.getUser(userId)

                result shouldBe cachedUser
                coVerify(exactly = 0) { api.fetchUser(any()) }
            }
        }

        `when`("user not in cache") {
            coEvery { dao.getUser(userId) } returns null
            coEvery { api.fetchUser(userId) } returns networkUser
            coEvery { dao.insert(networkUser) } just Runs

            then("fetches from network and caches") {
                val result = repository.getUser(userId)

                result shouldBe networkUser
                coVerify { api.fetchUser(userId) }
                coVerify { dao.insert(networkUser) }
            }
        }

        `when`("network fails") {
            coEvery { dao.getUser(userId) } returns null
            coEvery { api.fetchUser(any()) } throws IOException()

            then("throws exception") {
                shouldThrow<IOException> {
                    repository.getUser(userId)
                }
            }
        }
    }
})
```

### Compose UI Tests

```kotlin
@HiltAndroidTest
class UserListScreenTest {

    @get:Rule
    val hiltRule = HiltAndroidRule(this)

    @get:Rule
    val composeTestRule = createAndroidComposeRule<MainActivity>()

    @Test
    fun userList_displaysUsers() {
        composeTestRule.setContent {
            AppTheme {
                UserListScreen()
            }
        }

        composeTestRule.onNodeWithText("John Doe").assertExists()
        composeTestRule.onNodeWithText("Jane Smith").assertExists()
    }

    @Test
    fun userList_clickNavigatesToDetail() {
        composeTestRule.setContent {
            AppTheme {
                UserListScreen(onUserClick = { id ->
                    assertEquals("1", id)
                })
            }
        }

        composeTestRule.onNodeWithText("John Doe")
            .performClick()
    }

    @Test
    fun userList_scrollToBottom() {
        composeTestRule.setContent {
            AppTheme {
                UserListScreen()
            }
        }

        composeTestRule.onNodeWithTag("user_list")
            .performScrollToNode(hasText("User 50"))
    }
})
```

### Property-Based Tests

```kotlin
class CalculatorPropertyTest : StringSpec({

    "addition is commutative" {
        checkAll<Int, Int> { a, b ->
            a + b shouldBe b + a
        }
    }

    "reversing a string twice returns original" {
        checkAll<String> { str ->
            str.reversed().reversed() shouldBe str
        }
    }

    "list size after adding element increases by 1" {
        checkAll(Arb.list(Arb.int()), Arb.int()) { list, element ->
            (list + element).size shouldBe list.size + 1
        }
    }
})
```

### Data-Driven Tests

```kotlin
class CalculatorTest : FunSpec({

    context("addition") {
        withData(
            Pair(1, 1) to 2,
            Pair(2, 3) to 5,
            Pair(0, 0) to 0,
            Pair(-1, 1) to 0
        ) { (input, expected) ->
            val (a, b) = input
            a + b shouldBe expected
        }
    }

    context("division") {
        withData(
            nameFn = { "${it.first} / ${it.second} = ${it.third}" },
            Triple(10, 2, 5),
            Triple(9, 3, 3),
            Triple(100, 10, 10)
        ) { (a, b, expected) ->
            a / b shouldBe expected
        }
    }
})
```

## Test Data Builders

```kotlin
class UserBuilder {
    var id: String = "1"
    var name: String = "John Doe"
    var email: String = "john@test.com"
    var age: Int = 30

    fun build() = User(id, name, email, age)
}

fun user(block: UserBuilder.() -> Unit = {}) = 
    UserBuilder().apply(block).build()

// Usage in tests
val user = user {
    name = "Jane"
    email = "jane@test.com"
}
```

## Examples (Input → Output)

### Generate ViewModel Tests

**Input:** "Create tests for this ViewModel"

```kotlin
class ProductViewModel(
    private val repository: ProductRepository
) : ViewModel() {
    
    private val _products = MutableStateFlow<List<Product>>(emptyList())
    val products = _products.asStateFlow()
    
    fun loadProducts() {
        viewModelScope.launch {
            _products.value = repository.getProducts()
        }
    }
}
```

**Output:**

```kotlin
class ProductViewModelTest : FunSpec({

    coroutineTestScope = true

    lateinit var repository: ProductRepository
    lateinit var viewModel: ProductViewModel

    beforeEach {
        repository = mockk()
        viewModel = ProductViewModel(repository)
    }

    test("loadProducts updates state") {
        val products = listOf(
            Product("1", "Product 1", 10.0),
            Product("2", "Product 2", 20.0)
        )
        coEvery { repository.getProducts() } returns products

        viewModel.loadProducts()
        advanceUntilIdle()

        viewModel.products.value shouldBe products
    }

    test("loadProducts handles error") {
        coEvery { repository.getProducts() } throws 
            IOException("Network error")

        viewModel.loadProducts()
        advanceUntilIdle()

        viewModel.products.value shouldBe emptyList()
    }
})
```

## Resources

- [Kotest Documentation](https://kotest.io/)
- [MockK Documentation](https://mockk.io/)
- [Compose Testing](https://developer.android.com/jetpack/compose/testing)
