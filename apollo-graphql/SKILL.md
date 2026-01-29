---
name: apollo-graphql
description: Implements Apollo GraphQL in Android apps. Use when setting up GraphQL client, writing queries/mutations/subscriptions, implementing caching, pagination, or integrating with Coroutines/Flow.
tags: ["android", "graphql", "apollo", "networking", "coroutines", "flow", "cache", "pagination"]
difficulty: intermediate
category: networking
version: "1.0.0"
last_updated: "2025-01-29"
---

# Apollo GraphQL

## Quick Start

### Setup

```kotlin
// build.gradle (project)
plugins {
    id("com.apollographql.apollo3").version("3.8.2")
}

// build.gradle (app)
plugins {
    id("com.apollographql.apollo3")
}

dependencies {
    implementation("com.apollographql.apollo3:apollo-runtime:3.8.2")
    implementation("com.apollographql.apollo3:apollo-coroutines-support:3.8.2")
    implementation("com.apollographql.apollo3:apollo-cache-sqlite:3.8.2")
}

// Configure code generation
apollo {
    service("service") {
        packageName.set("com.example.graphql")
        srcDir("src/main/graphql")
    }
}
```

### Create Apollo Client

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object ApolloModule {

    @Provides
    @Singleton
    fun provideApolloClient(
        @ApplicationContext context: Context,
        authInterceptor: AuthInterceptor
    ): ApolloClient {
        return ApolloClient.Builder()
            .serverUrl("https://api.example.com/graphql")
            .addInterceptor(authInterceptor)
            .addInterceptor(LoggingInterceptor())
            .normalizedCache(
                MemoryCacheFactory(maxSizeBytes = 10 * 1024 * 1024)
                    .chain(SqlNormalizedCacheFactory(context, "apollo_cache.db"))
            )
            .build()
    }
}

// Authentication Interceptor
class AuthInterceptor @Inject constructor(
    private val tokenProvider: TokenProvider
) : ApolloInterceptor {
    override fun <D : Operation.Data> intercept(
        request: ApolloRequest<D>,
        chain: ApolloInterceptorChain
    ): Flow<ApolloResponse<D>> {
        val token = tokenProvider.getToken()
        val newRequest = request.newBuilder()
            .addHttpHeader("Authorization", "Bearer $token")
            .build()
        return chain.proceed(newRequest)
    }
}
```

## Core Patterns

### Queries with Coroutines

```kotlin
// GraphQL query (src/main/graphql/GetUser.graphql)
query GetUser($id: ID!) {
    user(id: $id) {
        id
        name
        email
        avatar {
            url
        }
    }
}

// Repository implementation
class UserRepository @Inject constructor(
    private val apolloClient: ApolloClient
) {
    suspend fun getUser(id: String): Result<User> {
        return try {
            val response = apolloClient
                .query(GetUserQuery(id))
                .execute()
            
            response.data?.user?.toUser()
                ?.let { Result.success(it) }
                ?: Result.failure(GraphQLError("User not found"))
        } catch (e: ApolloException) {
            Result.failure(e)
        }
    }
}

// ViewModel with StateFlow
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {
    
    private val _uiState = MutableStateFlow(UserUiState())
    val uiState = _uiState.asStateFlow()
    
    fun loadUser(id: String) {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            
            repository.getUser(id)
                .onSuccess { user ->
                    _uiState.update { it.copy(user = user, isLoading = false) }
                }
                .onFailure { error ->
                    _uiState.update { it.copy(error = error.message, isLoading = false) }
                }
        }
    }
}
```

### Mutations

```kotlin
// GraphQL mutation (src/main/graphql/UpdateUser.graphql)
mutation UpdateUser($id: ID!, $input: UpdateUserInput!) {
    updateUser(id: $id, input: $input) {
        id
        name
        email
    }
}

class UserRepository @Inject constructor(
    private val apolloClient: ApolloClient
) {
    suspend fun updateUser(
        id: String,
        name: String,
        email: String
    ): Result<User> {
        return try {
            val input = UpdateUserInput(name = name, email = email)
            val response = apolloClient
                .mutation(UpdateUserMutation(id, input))
                .execute()
            
            response.data?.updateUser?.toUser()
                ?.let { Result.success(it) }
                ?: Result.failure(GraphQLError("Update failed"))
        } catch (e: ApolloException) {
            Result.failure(e)
        }
    }
}
```

### Subscriptions with Flow

```kotlin
// GraphQL subscription (src/main/graphql/UserUpdates.graphql)
subscription UserUpdates($userId: ID!) {
    userUpdated(userId: $userId) {
        id
        status
        lastSeen
    }
}

class UserRepository @Inject constructor(
    private val apolloClient: ApolloClient
) {
    fun userUpdates(userId: String): Flow<UserUpdate> {
        return apolloClient
            .subscription(UserUpdatesSubscription(userId))
            .toFlow()
            .mapNotNull { response ->
                response.data?.userUpdated?.toUserUpdate()
            }
            .catch { e ->
                logger.e("Subscription error", e)
            }
    }
}

// Collect in ViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {
    
    fun observeUserUpdates(userId: String) {
        repository.userUpdates(userId)
            .onEach { update ->
                _uiState.update { current ->
                    current.copy(user = current.user?.copy(status = update.status))
                }
            }
            .launchIn(viewModelScope)
    }
}
```

### Pagination (Connection Pattern)

```kotlin
// GraphQL query with pagination (src/main/graphql/GetUsers.graphql)
query GetUsers($first: Int!, $after: String) {
    users(first: $first, after: $after) {
        edges {
            node {
                id
                name
                email
            }
            cursor
        }
        pageInfo {
            hasNextPage
            endCursor
        }
    }
}

// Repository with pagination
class UserRepository @Inject constructor(
    private val apolloClient: ApolloClient
) {
    suspend fun getUsers(
        first: Int = 20,
        after: String? = null
    ): Result<PaginatedUsers> {
        return try {
            val response = apolloClient
                .query(GetUsersQuery(first, Optional.presentIfNotNull(after)))
                .execute()
            
            val data = response.data?.users
            Result.success(
                PaginatedUsers(
                    users = data?.edges?.map { it.node.toUser() } ?: emptyList(),
                    hasNextPage = data?.pageInfo?.hasNextPage ?: false,
                    endCursor = data?.pageInfo?.endCursor
                )
            )
        } catch (e: ApolloException) {
            Result.failure(e)
        }
    }
}

// ViewModel with paging
class UserListViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {
    
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users = _users.asStateFlow()
    
    private var hasNextPage = true
    private var endCursor: String? = null
    private var isLoading = false
    
    fun loadMore() {
        if (isLoading || !hasNextPage) return
        
        viewModelScope.launch {
            isLoading = true
            
            repository.getUsers(after = endCursor)
                .onSuccess { paginated ->
                    _users.update { it + paginated.users }
                    hasNextPage = paginated.hasNextPage
                    endCursor = paginated.endCursor
                }
            
            isLoading = false
        }
    }
}
```

### Caching Strategies

```kotlin
// Memory cache configuration
@Provides
@Singleton
fun provideApolloClient(): ApolloClient {
    return ApolloClient.Builder()
        .serverUrl("https://api.example.com/graphql")
        .normalizedCache(
            MemoryCacheFactory(maxSizeBytes = 10 * 1024 * 1024)
        )
        .build()
}

// Disk cache with SQLDelight
@Provides
@Singleton
fun provideApolloClient(
    @ApplicationContext context: Context
): ApolloClient {
    return ApolloClient.Builder()
        .serverUrl("https://api.example.com/graphql")
        .normalizedCache(
            MemoryCacheFactory(maxSizeBytes = 10 * 1024 * 1024)
                .chain(SqlNormalizedCacheFactory(context, "apollo_cache.db"))
        )
        .build()
}

// Cache-first policy
class UserRepository @Inject constructor(
    private val apolloClient: ApolloClient
) {
    suspend fun getUserCached(id: String): Result<User> {
        return try {
            val response = apolloClient
                .query(GetUserQuery(id))
                .fetchPolicy(FetchPolicy.CacheFirst)
                .execute()
            
            response.data?.user?.toUser()
                ?.let { Result.success(it) }
                ?: Result.failure(GraphQLError("User not found"))
        } catch (e: ApolloException) {
            Result.failure(e)
        }
    }
    
    // Network-only for fresh data
    suspend fun getUserFresh(id: String): Result<User> {
        val response = apolloClient
            .query(GetUserQuery(id))
            .fetchPolicy(FetchPolicy.NetworkOnly)
            .execute()
        // ...
    }
}
```

### Error Handling

```kotlin
// Sealed class for GraphQL errors
sealed class GraphQLResult<out T> {
    data class Success<T>(val data: T) : GraphQLResult<T>()
    data class Error(
        val message: String,
        val code: String? = null,
        val extensions: Map<String, Any?>? = null
    ) : GraphQLResult<Nothing>()
    data class NetworkError(val exception: Throwable) : GraphQLResult<Nothing>()
}

// Repository with detailed error handling
class UserRepository @Inject constructor(
    private val apolloClient: ApolloClient
) {
    suspend fun getUser(id: String): GraphQLResult<User> {
        return try {
            val response = apolloClient
                .query(GetUserQuery(id))
                .execute()
            
            // Check for GraphQL errors
            if (response.hasErrors()) {
                val error = response.errors?.first()
                return GraphQLResult.Error(
                    message = error?.message ?: "Unknown error",
                    code = error?.extensions?.get("code") as? String,
                    extensions = error?.extensions
                )
            }
            
            // Check for data
            response.data?.user?.toUser()
                ?.let { GraphQLResult.Success(it) }
                ?: GraphQLResult.Error("User not found")
                
        } catch (e: ApolloNetworkException) {
            GraphQLResult.NetworkError(e)
        } catch (e: ApolloException) {
            GraphQLResult.Error(e.message ?: "Apollo error")
        }
    }
}
```

## Examples (Input → Output)

### Setup with Authentication

**Input:** "Set up Apollo with JWT authentication and caching"

**Output:**

```kotlin
// AuthInterceptor.kt
class AuthInterceptor @Inject constructor(
    private val tokenManager: TokenManager
) : ApolloInterceptor {
    override fun <D : Operation.Data> intercept(
        request: ApolloRequest<D>,
        chain: ApolloInterceptorChain
    ): Flow<ApolloResponse<D>> {
        val token = tokenManager.getToken()
        val newRequest = if (token != null) {
            request.newBuilder()
                .addHttpHeader("Authorization", "Bearer $token")
                .build()
        } else {
            request
        }
        return chain.proceed(newRequest)
    }
}

// ApolloModule.kt
@Module
@InstallIn(SingletonComponent::class)
object ApolloModule {
    @Provides
    @Singleton
    fun provideApolloClient(
        @ApplicationContext context: Context,
        authInterceptor: AuthInterceptor
    ): ApolloClient {
        return ApolloClient.Builder()
            .serverUrl(BuildConfig.GRAPHQL_URL)
            .addInterceptor(authInterceptor)
            .addInterceptor(LoggingInterceptor(level = LoggingInterceptor.Level.BODY))
            .normalizedCache(
                MemoryCacheFactory(maxSizeBytes = 10 * 1024 * 1024)
                    .chain(SqlNormalizedCacheFactory(context))
            )
            .httpEngine(OkHttpEngine {
                okHttpClient {
                    OkHttpClient.Builder()
                        .addNetworkInterceptor(HttpLoggingInterceptor().apply {
                            level = HttpLoggingInterceptor.Level.BODY
                        })
                        .build()
                }
            })
            .build()
    }
}
```

## Best Practices

1. **Use normalized cache** for automatic cache updates after mutations
2. **Handle errors at multiple levels** - network, GraphQL, and parsing errors
3. **Use Flow for subscriptions** and real-time updates
4. **Implement pagination** with connection pattern for lists
5. **Add interceptors** for auth, logging, and request modification
6. **Use Kotlin Result** or sealed classes for type-safe error handling
7. **Test with MockApolloClient** for unit tests
8. **Monitor cache hit rates** to optimize fetch policies

## Resources

- [Apollo Kotlin Documentation](https://www.apollographql.com/docs/kotlin/)
- [GraphQL Best Practices](https://www.apollographql.com/docs/technotes/TN0001-schema-design/)
- [Caching Guide](https://www.apollographql.com/docs/kotlin/caching/normalized-cache)
