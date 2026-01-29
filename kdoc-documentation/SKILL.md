---
name: kdoc-documentation
description: Guides Kotlin documentation with KDoc for public and internal APIs. Use when documenting code, writing library docs, explaining complex logic, or creating API references. Covers KDoc syntax, best practices, tags, and documentation patterns.
tags: ["kotlin", "kdoc", "documentation", "api", "dokka", "comments"]
difficulty: beginner
category: documentation
version: "1.0.0"
last_updated: "2025-01-29"
---

# KDoc Documentation

## Quick Start

KDoc is Kotlin's documentation format, similar to Javadoc but with Markdown support.

**Basic structure:**

```kotlin
/**
 * Brief description of what this does.
 *
 * Detailed explanation with **markdown** support.
 * Can include [links] and code blocks.
 *
 * @param name Description of parameter
 * @return Description of return value
 * @throws IllegalArgumentException When input is invalid
 * @since 1.2.0
 * @see RelatedClass
 */
fun process(name: String): Result {
    // implementation
}
```

**Generate docs with Dokka:**

```kotlin
// build.gradle.kts
plugins {
    id("org.jetbrains.dokka") version "1.9.10"
}

dependencies {
    dokkaPlugin("org.jetbrains.dokka:android-documentation-plugin:1.9.10")
}
```

**Run Dokka:**
```bash
./gradlew dokkaHtml
./gradlew dokkaHtmlMultiModule  // For multi-module projects
```

## Core Concepts

### KDoc Syntax

**Basic documentation:**

```kotlin
/**
 * Calculates the area of a rectangle.
 *
 * This function multiplies width by height to determine
 * the total area in square units.
 *
 * @param width The width of the rectangle (must be positive)
 * @param height The height of the rectangle (must be positive)
 * @return The calculated area
 * @throws IllegalArgumentException if width or height is negative
 *
 * @sample samples.RectangleSamples.calculateAreaSample
 */
fun calculateArea(width: Double, height: Double): Double {
    require(width > 0) { "Width must be positive" }
    require(height > 0) { "Height must be positive" }
    return width * height
}
```

**Markdown formatting:**

```kotlin
/**
 * Processes user data for the **analytics system**.
 *
 * This function handles:
 * - *Validation* of input data
 * - *Transformation* to internal format
 * - *Storage* in the database
 *
 * ## Usage
 *
 * ```kotlin
 * val result = processUserData(
 *     userId = "123",
 *     data = userInput
 * )
 * ```
 *
 * ## Important Notes
 *
 * > This operation is **not reversible**.
 * > Ensure backups exist before processing.
 *
 * @param userId The unique identifier for the user
 * @param data The raw user data to process
 * @return [ProcessingResult] containing the outcome
 * @throws UserNotFoundException if the user doesn't exist
 *
 * @author John Doe
 * @since 2.1.0
 * @see [UserDataProcessor] for the full processing pipeline
 * @see <a href="https://docs.example.com">Full Documentation</a>
 */
fun processUserData(userId: String, data: UserData): ProcessingResult {
    // implementation
}
```

### Documentation Elements

**Classes and interfaces:**

```kotlin
/**
 * Manages user authentication and session state.
 *
 * This class provides a complete authentication solution including:
 * - Login with credentials
 * - Token refresh
 * - Session management
 * - Logout handling
 *
 * ## Thread Safety
 *
 * This class is thread-safe and can be accessed from multiple threads.
 * All operations are synchronized internally.
 *
 * ## Example
 *
 * ```kotlin
 * val authManager = AuthManager(
 *     tokenRepository = tokenRepo,
 *     userRepository = userRepo
 * )
 *
 * val result = authManager.login("user", "pass")
 * if (result.isSuccess) {
 *     val session = result.getOrThrow()
 *     // Use session
 * }
 * ```
 *
 * @property tokenRepository Repository for token storage
 * @property userRepository Repository for user data
 * @constructor Creates a new AuthManager instance
 *
 * @author Jane Smith
 * @since 1.0.0
 */
class AuthManager(
    private val tokenRepository: TokenRepository,
    private val userRepository: UserRepository
) {
    // class implementation
}
```

**Properties:**

```kotlin
/**
 * The current user session.
 *
 * This property returns the active session if one exists,
 * or null if no user is currently logged in.
 *
 * The session is automatically refreshed if needed.
 */
var currentSession: Session? = null
    private set

/**
 * Maximum number of retry attempts for failed operations.
 *
 * Default value is 3. Set to 0 to disable retries.
 */
var maxRetries: Int = 3
```

**Enum classes:**

```kotlin
/**
 * Represents the various states of a network request.
 *
 * @property description Human-readable description of the state
 */
enum class RequestState(val description: String) {
    /**
     * Initial state before any request is made.
     */
    IDLE("Waiting to start"),

    /**
     * Request is actively being processed.
     */
    LOADING("In progress"),

    /**
     * Request completed successfully with data.
     */
    SUCCESS("Completed successfully"),

    /**
     * Request failed with an error.
     *
     * @property error The exception that caused the failure
     */
    ERROR("Failed")
}
```

## Common Patterns

### Standard Tags

| Tag | Purpose | Example |
|-----|---------|---------|
| `@param` | Describe parameter | `@param userId The user identifier` |
| `@return` | Describe return value | `@return The user object` |
| `@throws` | Document exceptions | `@throws IllegalArgumentException` |
| `@see` | Reference related code | `@see UserRepository` |
| `@since` | Version introduced | `@since 2.1.0` |
| `@author` | Author attribution | `@author John Doe` |
| `@sample` | Include code sample | `@sample samples.Usage` |
| `@deprecated` | Mark as deprecated | `@deprecated Use newMethod()` |
| `@suppress` | Exclude from docs | `@suppress Internal API` |

### Function Documentation

**Simple function:**

```kotlin
/**
 * Returns the sum of two integers.
 *
 * @param a The first number
 * @param b The second number
 * @return The sum of a and b
 */
fun add(a: Int, b: Int): Int = a + b
```

**Complex function:**

```kotlin
/**
 * Fetches user data from the remote API with caching support.
 *
 * This function will:
 * 1. Check the local cache first
 * 2. Return cached data if fresh (within [cacheDuration])
 * 3. Fetch from network if cache is stale or missing
 * 4. Update cache with fresh data
 *
 * ## Performance
 *
 * - Cache lookup: O(1)
 * - Network call: ~200-500ms
 * - Cache write: O(1)
 *
 * ## Threading
 *
 * This function is safe to call from any thread. Network operations
 * are performed on a background thread automatically.
 *
 * ## Example
 *
 * ```kotlin
 * // Basic usage with default cache
 * val user = fetchUserData("user-123")
 *
 * // With custom cache duration
 * val user = fetchUserData(
 *     userId = "user-123",
 *     cacheDuration = Duration.ofMinutes(5)
 * )
 * ```
 *
 * @param userId The unique identifier of the user to fetch
 * @param cacheDuration How long to consider cached data valid.
 *                      Default is 5 minutes.
 * @return The user data, or null if user not found
 * @throws IllegalArgumentException if [userId] is blank
 * @throws NetworkException if the network request fails
 * @throws TimeoutException if the request exceeds 30 seconds
 *
 * @sample samples.UserDataSamples.fetchUserDataSample
 * @see UserCache for cache implementation details
 * @since 1.5.0
 */
suspend fun fetchUserData(
    userId: String,
    cacheDuration: Duration = Duration.ofMinutes(5)
): User? {
    require(userId.isNotBlank()) { "userId cannot be blank" }
    // implementation
}
```

### Class Hierarchy Documentation

**Base class:**

```kotlin
/**
 * Base class for all data sources in the application.
 *
 * Data sources are responsible for:
 * - Fetching data from external sources
 * - Caching data locally
 * - Synchronizing data changes
 *
 * Implementations should be thread-safe and handle errors gracefully.
 *
 * @param T The type of data this source provides
 * @constructor Creates a new data source with the specified configuration
 *
 * @see RemoteDataSource
 * @see LocalDataSource
 * @see CachedDataSource
 */
abstract class DataSource<T> {
    /**
     * Fetches data from this source.
     *
     * @param forceRefresh If true, bypass cache and fetch fresh data
     * @return Result containing data or error
     */
    abstract suspend fun fetch(forceRefresh: Boolean = false): Result<T>

    /**
     * Clears any cached data.
     */
    abstract fun clearCache()
}
```

**Implementation class:**

```kotlin
/**
 * Data source that fetches from remote API with local caching.
 *
 * This implementation combines network fetching with local database
 * storage for optimal performance and offline support.
 *
 * ## Caching Strategy
 *
 * - Cache hit: Return immediately (no network call)
 * - Cache miss/stale: Fetch from network, update cache
 * - Network error: Return stale cache if available
 *
 * @param T The type of data
 * @property api The remote API client
 * @property database The local database
 * @property cacheDuration How long cache is considered fresh
 *
 * @author John Doe
 * @since 1.0.0
 */
class CachedDataSource<T>(
    private val api: Api<T>,
    private val database: Database<T>,
    private val cacheDuration: Duration
) : DataSource<T>() {
    // implementation
}
```

### Interface Documentation

```kotlin
/**
 * Contract for authentication operations.
 *
 * Implementations handle the actual authentication mechanism
 * (OAuth, basic auth, token-based, etc.).
 *
 * ## Security
 *
 * - Never log credentials
 * - Use secure storage for tokens
 * - Clear sensitive data on logout
 *
 * @see OAuthAuthenticator
 * @see BasicAuthenticator
 * @see TokenAuthenticator
 */
interface Authenticator {
    /**
     * Authenticates with the provided credentials.
     *
     * @param credentials The authentication credentials
     * @return [AuthResult.Success] with token on success,
     *         [AuthResult.Failure] with error on failure
     * @throws IllegalArgumentException if credentials are invalid
     */
    suspend fun authenticate(credentials: Credentials): AuthResult

    /**
     * Refreshes an expired authentication token.
     *
     * @param token The expired token
     * @return New valid token, or null if refresh fails
     */
    suspend fun refreshToken(token: Token): Token?

    /**
     * Logs out the current user and invalidates the token.
     */
    suspend fun logout()
}
```

### Extension Functions

```kotlin
/**
 * Formats this [Instant] as a user-friendly relative time string.
 *
 * Examples:
 * - "just now" for times within the last minute
 * - "5 minutes ago" for times within the last hour
 * - "2 hours ago" for times within the last day
 * - "Yesterday" for yesterday
 * - "Jan 15" for dates within this year
 * - "Jan 15, 2023" for older dates
 *
 * @param context Android context for string resources
 * @return Formatted relative time string
 * @see java.time.format.DateTimeFormatter for custom formatting
 */
fun Instant.toRelativeTime(context: Context): String {
    // implementation
}
```

### Type Aliases

```kotlin
/**
 * Type alias for user ID strings.
 *
 * Using this type alias instead of raw [String] makes code more
 * self-documenting and prevents mixing up different ID types.
 *
 * Example:
 * ```kotlin
 * fun fetchUser(userId: UserId): User
 * fun fetchOrder(orderId: OrderId): Order
 * ```
 */
typealias UserId = String

/**
 * Callback invoked when a user is selected.
 *
 * @param user The selected user
 * @param isLongPress True if triggered by long press
 */
typealias UserSelectedCallback = (user: User, isLongPress: Boolean) -> Unit
```

## Examples (Input → Output)

### Documenting a Repository

**Input request:** "Document a user repository with all CRUD operations"

**Output:**

```kotlin
/**
 * Repository for user data operations.
 *
 * This repository provides a clean API for user data access,
 * abstracting the underlying data sources (network, database, cache).
 *
 * All operations return [Result] types for explicit error handling.
 *
 * ## Thread Safety
 *
 * This class is thread-safe. All operations can be called from
 * any thread or coroutine context.
 *
 * ## Error Handling
 *
 * Operations may fail with:
 * - [UserNotFoundException] - User doesn't exist
 * - [NetworkException] - Network request failed
 * - [ValidationException] - User data is invalid
 * - [DuplicateUserException] - User already exists
 *
 * ## Example
 *
 * ```kotlin
 * val repository = UserRepository(
 *     remoteDataSource = api,
 *     localDataSource = database
 * )
 *
 * // Create user
 * val result = repository.createUser(
 *     email = "user@example.com",
 *     name = "John Doe"
 * )
 *
 * when (result) {
 *     is Result.Success -> println("Created: ${result.data}")
 *     is Result.Failure -> println("Error: ${result.exception}")
 * }
 * ```
 *
 * @property remoteDataSource Source for network operations
 * @property localDataSource Source for local database operations
 *
 * @author Jane Smith
 * @since 1.0.0
 */
class UserRepository(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource
) {
    /**
     * Creates a new user.
     *
     * @param email The user's email address (must be unique)
     * @param name The user's display name
     * @return [Result] containing the created [User] or an exception
     * @throws ValidationException if email is invalid or name is blank
     * @throws DuplicateUserException if email already exists
     * @throws NetworkException if the network request fails
     *
     * @sample samples.UserRepositorySamples.createUserSample
     */
    suspend fun createUser(
        email: String,
        name: String
    ): Result<User> {
        // implementation
    }

    /**
     * Retrieves a user by their unique ID.
     *
     * This method checks local cache first, then falls back to
     * network if not cached or cache is stale.
     *
     * @param userId The unique identifier of the user
     * @param forceRefresh If true, bypass cache and fetch from network
     * @return [Result] containing the [User] or null if not found
     * @throws UserNotFoundException if user doesn't exist
     * @throws NetworkException if network request fails and no cache available
     */
    suspend fun getUser(
        userId: String,
        forceRefresh: Boolean = false
    ): Result<User?> {
        // implementation
    }

    /**
     * Updates an existing user's information.
     *
     * Only non-null fields are updated. Null fields retain their current values.
     *
     * @param userId The ID of the user to update
     * @param email New email (null to keep current)
     * @param name New name (null to keep current)
     * @param avatarUrl New avatar URL (null to keep current)
     * @return [Result] containing the updated [User]
     * @throws UserNotFoundException if user doesn't exist
     * @throws ValidationException if any field is invalid
     * @throws DuplicateUserException if email change conflicts with existing user
     */
    suspend fun updateUser(
        userId: String,
        email: String? = null,
        name: String? = null,
        avatarUrl: String? = null
    ): Result<User> {
        // implementation
    }

    /**
     * Deletes a user permanently.
     *
     * > **Warning:** This operation is irreversible. The user's data
     * > will be permanently removed from all data sources.
     *
     * @param userId The ID of the user to delete
     * @return [Result] indicating success or failure
     * @throws UserNotFoundException if user doesn't exist
     * @throws NetworkException if deletion fails on remote
     */
    suspend fun deleteUser(userId: String): Result<Unit> {
        // implementation
    }

    /**
     * Lists all users with optional filtering and pagination.
     *
     * @param query Search query to filter users (null for all)
     * @param page Page number (0-indexed)
     * @param pageSize Number of users per page
     * @return [Result] containing list of [User] objects
     * @throws NetworkException if the request fails
     */
    suspend fun listUsers(
        query: String? = null,
        page: Int = 0,
        pageSize: Int = 20
    ): Result<List<User>> {
        // implementation
    }
}
```

### Documenting Internal APIs

**Input request:** "Document internal classes that shouldn't be public"

**Output:**

```kotlin
/**
 * Internal utility for database migration operations.
 *
 * > **Note:** This is an internal API and should not be used outside
 * > of the persistence module. It may change or be removed without notice.
 *
 * @suppress
 */
@InternalApi
internal class MigrationHelper(
    private val database: SQLiteDatabase
) {
    /**
     * Executes a series of migration steps within a transaction.
     *
     * If any step fails, the entire transaction is rolled back.
     *
     * @param steps The migration steps to execute
     * @return True if all migrations succeeded
     * @throws MigrationException if any step fails
     */
    internal fun runMigrations(steps: List<MigrationStep>): Boolean {
        // implementation
    }
}

/**
 * Marker annotation for internal APIs.
 *
 * APIs marked with this annotation are internal implementation details
 * and should not be used by external code.
 *
 * @suppress
 */
@RequiresOptIn(
    level = RequiresOptIn.Level.ERROR,
    message = "This is an internal API that may change without notice"
)
@Retention(AnnotationRetention.BINARY)
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
annotation class InternalApi
```

## Best Practices

1. **Document public APIs**: Every public class, function, property needs KDoc
2. **Start with summary**: First sentence should be a brief summary (appears in indexes)
3. **Use Markdown**: Bold, italics, lists, code blocks for readability
4. **Document parameters**: Every `@param` should explain purpose and constraints
5. **Document exceptions**: List all possible exceptions with conditions
6. **Include examples**: Use `@sample` or inline code blocks for complex APIs
7. **Document thread safety**: Note if class is thread-safe or not
8. **Use `@since`**: Track when APIs were introduced
9. **Mark deprecated APIs**: Use `@deprecated` with migration path
10. **Hide internals**: Use `@suppress` for internal implementation details
11. **Document nullability**: Explain when null can be returned or passed
12. **Link related items**: Use `@see` and `[ClassName]` for cross-references
13. **Keep it current**: Update docs when changing code
14. **Be concise but complete**: Don't state the obvious, explain the non-obvious
15. **Use imperative mood**: "Returns the user" not "Returns the user"

## Resources

- [KDoc reference](https://kotlinlang.org/docs/kotlin-doc.html)
- [Dokka documentation](https://kotlinlang.org/docs/dokka-introduction.html)
- [Markdown in KDoc](https://kotlinlang.org/docs/kotlin-doc.html#markdown-syntax)
- [API documentation best practices](https://kotlinlang.org/docs/jvm-api-guidelines-introduction.html)
