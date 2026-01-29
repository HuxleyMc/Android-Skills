# Apollo GraphQL - Agent Guide

## Quick Start

Use this skill when:
- Setting up Apollo Kotlin client in an Android project
- Writing GraphQL queries, mutations, or subscriptions
- Implementing caching (memory or normalized)
- Adding pagination with connection patterns
- Integrating with authentication (JWT/OAuth)
- Converting GraphQL responses to domain models
- Testing GraphQL operations

## Common Tasks

### 1. Setup Apollo Client

**Input:** "Set up Apollo with Hilt DI"

**Action:**
1. Add Apollo plugin and dependencies
2. Configure code generation in build.gradle
3. Create ApolloModule with Hilt
4. Add auth interceptor

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object ApolloModule {
    @Provides
    @Singleton
    fun provideApolloClient(authInterceptor: AuthInterceptor): ApolloClient {
        return ApolloClient.Builder()
            .serverUrl("https://api.example.com/graphql")
            .addInterceptor(authInterceptor)
            .normalizedCache(MemoryCacheFactory(maxSizeBytes = 10 * 1024 * 1024))
            .build()
    }
}
```

### 2. Write a Query

**Input:** "Create a query to fetch user details"

**Action:**
1. Create .graphql file in src/main/graphql
2. Write query with variables
3. Generate code with build
4. Use in repository

```graphql
# GetUser.graphql
query GetUser($id: ID!) {
    user(id: $id) {
        id
        name
        email
    }
}
```

```kotlin
// Repository
suspend fun getUser(id: String): Result<User> {
    return try {
        val response = apolloClient.query(GetUserQuery(id)).execute()
        Result.success(response.data!!.user.toUser())
    } catch (e: ApolloException) {
        Result.failure(e)
    }
}
```

### 3. Handle Mutations with Cache Update

**Input:** "Update user and refresh cache"

**Action:** Use optimistic updates or refetch queries

```kotlin
apolloClient.mutation(UpdateUserMutation(id, input))
    .fetchPolicy(FetchPolicy.NetworkOnly)
    .execute()
```

### 4. Implement Pagination

**Input:** "Add pagination to user list"

**Action:** Use connection pattern with cursors

```graphql
query GetUsers($first: Int!, $after: String) {
    users(first: $first, after: $after) {
        edges { node { id name } cursor }
        pageInfo { hasNextPage endCursor }
    }
}
```

## Error Handling Patterns

| Scenario | Solution |
|----------|----------|
| Network failure | Catch ApolloNetworkException |
| GraphQL error | Check response.errors |
| Auth failure | Check error extensions for code |
| Partial data | Handle nullable fields |

## Cache Policies

| Policy | Use Case |
|--------|----------|
| CacheFirst | Fast UI, stale data OK |
| NetworkOnly | Fresh data required |
| CacheAndNetwork | Show cached, then update |
| NoCache | Sensitive data |

## Testing

```kotlin
// MockApolloClient for tests
val mockClient = ApolloClient.Builder()
    .serverUrl("https://test.com")
    .addInterceptor(MockInterceptor())
    .build()
```

## Resources

- [Apollo Kotlin Docs](https://www.apollographql.com/docs/kotlin/)
- [Normalized Cache](https://www.apollographql.com/docs/kotlin/caching/normalized-cache)
- [Pagination Guide](https://www.apollographql.com/docs/kotlin/pagination/)
