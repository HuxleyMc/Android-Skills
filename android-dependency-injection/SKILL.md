---
name: android-dependency-injection
description: Implements dependency injection with Hilt, Koin, and Metro. Use when setting up DI, migrating between frameworks, organizing modules, or testing with injected dependencies.
tags: ["android", "dependency-injection", "hilt", "koin", "metro", "dagger", "ksp", "testing"]
difficulty: intermediate
category: architecture
version: "1.0.0"
last_updated: "2025-01-29"
---

# Android Dependency Injection

## Quick Start

Choose your DI framework:

```
┌─────────────┬─────────────────┬─────────────────┬─────────────────┐
│   Feature   │     Hilt        │     Koin        │     Metro       │
├─────────────┼─────────────────┼─────────────────┼─────────────────┤
│ Type        │ Compile-time    │ Runtime         │ Compile-time    │
│ Build Tool  │ KAPT/KSP        │ None            │ KSP             │
│ Startup     │ Slower          │ Faster          │ Fast            │
│ Error Catch │ Build time      │ Runtime         │ Build time      │
│ Jetpack     │ Excellent       │ Good            │ Good            │
│ Learning    │ Steeper         │ Easy            │ Moderate        │
└─────────────┴─────────────────┴─────────────────┴─────────────────┘
```

## Hilt

### Setup

```kotlin
// build.gradle (project)
plugins {
    id("com.google.dagger.hilt.android") version "2.48" apply false
}

// build.gradle (app)
plugins {
    id("com.google.dagger.hilt.android")
    id("com.google.devtools.ksp")
}

dependencies {
    implementation("com.google.dagger:hilt-android:2.48")
    ksp("com.google.dagger:hilt-compiler:2.48")
    
    // Testing
    testImplementation("com.google.dagger:hilt-android-testing:2.48")
    kspTest("com.google.dagger:hilt-compiler:2.48")
}
```

### Application Setup

```kotlin
@HiltAndroidApp
class MyApplication : Application()
```

```xml
<!-- AndroidManifest.xml -->
<application
    android:name=".MyApplication"
    android:label="@string/app_name">
</application>
```

### Basic Injection

```kotlin
// Constructor injection (preferred)
class UserRepository @Inject constructor(
    private val api: UserApi,
    private val dao: UserDao
) {
    suspend fun getUser(id: String): User {
        return dao.getUser(id) ?: api.getUser(id).also {
            dao.insert(it)
        }
    }
}

// Field injection (Android components)
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject lateinit var viewModelFactory: ViewModelProvider.Factory
}

@AndroidEntryPoint
class UserFragment : Fragment() {
    private val viewModel: UserViewModel by viewModels()
}

@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {
    // ...
}
```

### Module Definition

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BuildConfig.BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(MoshiConverterFactory.create())
            .build()
    }
    
    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(HttpLoggingInterceptor())
            .build()
    }
    
    @Provides
    fun provideUserApi(retrofit: Retrofit): UserApi {
        return retrofit.create(UserApi::class.java)
    }
}

@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    
    @Binds
    @Singleton
    abstract fun bindUserRepository(
        impl: UserRepositoryImpl
    ): UserRepository
}
```

### Scoping

```kotlin
@Singleton              // Application scope
@ActivityScoped         // Activity scope (survives config change)
@FragmentScoped         // Fragment scope
@ActivityRetainedScoped // Survives config change, shared across fragments
@ViewModelScoped        // ViewModel scope
@ServiceScoped         // Service scope
```

### Qualifiers

```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class ApiKey

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class BaseUrl

@Module
@InstallIn(SingletonComponent::class)
object ConfigModule {
    
    @Provides
    @ApiKey
    fun provideApiKey(): String = BuildConfig.API_KEY
    
    @Provides
    @BaseUrl
    fun provideBaseUrl(): String = BuildConfig.BASE_URL
}

class ApiClient @Inject constructor(
    @ApiKey private val apiKey: String,
    @BaseUrl private val baseUrl: String
)
```

## Koin

### Setup

```kotlin
// build.gradle (app)
dependencies {
    implementation("io.insert-koin:koin-android:3.5.0")
    implementation("io.insert-koin:koin-androidx-compose:3.5.0")
    
    // Testing
    testImplementation("io.insert-koin:koin-test:3.5.0")
    testImplementation("io.insert-koin:koin-test-junit4:3.5.0")
}
```

### Application Setup

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidLogger()
            androidContext(this@MyApplication)
            modules(appModule, networkModule, dataModule)
        }
    }
}
```

### Module Definition

```kotlin
val networkModule = module {
    single { 
        OkHttpClient.Builder()
            .addInterceptor(HttpLoggingInterceptor())
            .build() 
    }
    
    single {
        Retrofit.Builder()
            .baseUrl(BuildConfig.BASE_URL)
            .client(get())
            .addConverterFactory(MoshiConverterFactory.create())
            .build()
    }
    
    single { get<Retrofit>().create(UserApi::class.java) }
}

val repositoryModule = module {
    single<UserRepository> { UserRepositoryImpl(get(), get()) }
    single { UserRepositoryImpl(get(), get()) } bind UserRepository::class
}

val viewModelModule = module {
    viewModel { UserViewModel(get()) }
    viewModel { (userId: String) -> DetailViewModel(userId, get()) }
}
```

### Injection

```kotlin
// Constructor injection
class UserRepository(
    private val api: UserApi,
    private val dao: UserDao
)

// Field injection in Android
class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModel()
    private val sharedViewModel: SharedViewModel by sharedViewModel()
}

// Compose injection
@Composable
fun UserScreen() {
    val viewModel: UserViewModel = koinViewModel()
    val userId = "123"
    val detailViewModel: DetailViewModel = koinViewModel { parametersOf(userId) }
}

// Direct injection
val repository: UserRepository by inject()
val api: UserApi by inject()
```

### Scoping

```kotlin
module {
    single { }           // Singleton (application scope)
    factory { }          // New instance each time
    scoped { }           // Scope-specific (activity, fragment)
    viewModel { }        // ViewModel scope
}

// Custom scope
val activityScope = scope<MainActivity> {
    scoped { ActivityDependency() }
}
```

### Qualifiers

```kotlin
val appModule = module {
    single(named("api_key")) { BuildConfig.API_KEY }
    single(named("base_url")) { BuildConfig.BASE_URL }
    
    single { ApiClient(get(named("api_key")), get(named("base_url"))) }
}

// Usage
val apiKey: String by inject(named("api_key"))
```

## Metro (Kotlin Inject)

### Setup

```kotlin
// build.gradle (app)
plugins {
    id("com.google.devtools.ksp") version "1.9.20-1.0.14"
}

dependencies {
    implementation("dev.zacsweers.metro:runtime:0.1.0")
    ksp("dev.zacsweers.metro:compiler:0.1.0")
}
```

### Component Definition

```kotlin
// Metro is KSP-based and generates code at compile time
@DependencyGraph
interface AppGraph {
    val userRepository: UserRepository
    val userViewModel: UserViewModel
    
    @DependencyGraph.Factory
    fun interface Factory {
        fun create(@ApplicationContext context: Context): AppGraph
    }
}

// Generated usage
val graph = AppGraph.Factory.create(context)
val repository = graph.userRepository
```

### Provider Functions

```kotlin
@DependencyGraph
interface AppGraph {
    // Metro generates implementations based on these providers
}

@Provides
fun provideOkHttpClient(): OkHttpClient {
    return OkHttpClient.Builder()
        .addInterceptor(HttpLoggingInterceptor())
        .build()
}

@Provides
fun provideRetrofit(client: OkHttpClient): Retrofit {
    return Retrofit.Builder()
        .baseUrl(BuildConfig.BASE_URL)
        .client(client)
        .build()
}

@Provides
@Singleton
fun provideUserApi(retrofit: Retrofit): UserApi {
    return retrofit.create(UserApi::class.java)
}
```

### Assisted Injection

```kotlin
// Factory for runtime parameters
@AssistedFactory
interface UserViewModelFactory {
    fun create(userId: String): UserViewModel
}

class UserViewModel @AssistedInject constructor(
    @Assisted private val userId: String,
    private val repository: UserRepository
) : ViewModel()
```

## Testing

### Hilt Testing

```kotlin
@HiltAndroidTest
class UserRepositoryTest {
    
    @get:Rule
    var hiltRule = HiltAndroidRule(this)
    
    @Inject
    lateinit var repository: UserRepository
    
    @Before
    fun init() {
        hiltRule.inject()
    }
    
    @Test
    fun testGetUser() = runTest {
        val user = repository.getUser("1")
        assertNotNull(user)
    }
}

// With mocks
@Module
@TestInstallIn(
    components = [SingletonComponent::class],
    replaces = [NetworkModule::class]
)
object TestNetworkModule {
    @Provides
    fun provideMockApi(): UserApi {
        return mockk {
            coEvery { getUser(any()) } returns User("1", "Test")
        }
    }
}
```

### Koin Testing

```kotlin
class UserRepositoryTest : KoinTest {
    
    private val repository: UserRepository by inject()
    
    @get:Rule
    val koinTestRule = KoinTestRule.create {
        modules(testModule)
    }
    
    @Test
    fun testGetUser() = runTest {
        val user = repository.getUser("1")
        assertNotNull(user)
    }
}

// Test module
val testModule = module {
    single { mockk<UserApi>() }
    single<UserRepository> { UserRepositoryImpl(get(), get()) }
}
```

## Multi-Module Projects

### Hilt

```kotlin
// :core:network
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit { }
}

// :feature:user (uses network, doesn't need to know implementation)
@Module
@InstallIn(ViewModelComponent::class)
abstract class UserModule {
    @Binds
    abstract fun bindRepository(impl: UserRepositoryImpl): UserRepository
}
```

### Koin

```kotlin
// :core:network
val networkModule = module {
    single { Retrofit.Builder()... }
}

// :feature:user
val userModule = module {
    single<UserRepository> { UserRepositoryImpl(get()) }
    viewModel { UserViewModel(get()) }
}

// :app
startKoin {
    modules(
        networkModule,
        databaseModule,
        userModule,
        settingsModule
    )
}
```

## Migration Guide

### Hilt to Koin

```kotlin
// Before (Hilt)
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel()

// After (Koin)
class UserViewModel(
    private val repository: UserRepository
) : ViewModel()

// Module
val viewModelModule = module {
    viewModel { UserViewModel(get()) }
}
```

### Koin to Hilt

```kotlin
// Before (Koin)
val repositoryModule = module {
    single<UserRepository> { UserRepositoryImpl(get(), get()) }
}

// After (Hilt)
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    @Binds
    @Singleton
    abstract fun bindRepository(
        impl: UserRepositoryImpl
    ): UserRepository
}
```

## Examples (Input → Output)

### Multi-Qualifier Setup

**Input:** "Set up Hilt with multiple API clients (staging and production)"

**Output:**

```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class ProductionApi

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class StagingApi

@Module
@InstallIn(SingletonComponent::class)
object ApiModule {
    
    @Provides
    @ProductionApi
    @Singleton
    fun provideProductionRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .build()
    }
    
    @Provides
    @StagingApi
    @Singleton
    fun provideStagingRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://staging-api.example.com/")
            .build()
    }
    
    @Provides
    @Singleton
    fun provideUserApi(
        @ProductionApi retrofit: Retrofit
    ): UserApi {
        return retrofit.create(UserApi::class.java)
    }
}
```

## Best Practices

1. **Prefer constructor injection** over field injection
2. **Use singleton scope** for stateless dependencies (API, Repository)
3. **Use factory scope** for stateful dependencies (ViewModel)
4. **Create feature modules** in multi-module projects
5. **Avoid component dependencies** when possible (use subcomponents)
6. **Use qualifiers sparingly** - prefer separate types
7. **Test with real DI** - don't manually construct dependencies in tests
8. **Document scoping decisions** - why singleton vs factory?

## Resources

- [Hilt Documentation](https://dagger.dev/hilt/)
- [Koin Documentation](https://insert-koin.io/docs/quickstart/android/)
- [Metro GitHub](https://github.com/ZacSweers/metro)
- [Dependency Injection Guide](https://developer.android.com/training/dependency-injection)
