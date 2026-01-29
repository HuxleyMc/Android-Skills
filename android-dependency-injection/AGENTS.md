# Android Dependency Injection - Agent Guide

## Quick Start

Use this skill when:
- Choosing between Hilt, Koin, or Metro for a project
- Setting up DI in a new Android project
- Migrating between DI frameworks
- Organizing modules in multi-module projects
- Testing with dependency injection
- Resolving scoping or qualifier issues

## Framework Selection Guide

| Use Case | Recommendation |
|----------|---------------|
| New large project | Hilt (compile-time safety) |
| Quick prototype | Koin (fast setup) |
| Kotlin Multiplatform | Koin (KMP support) |
| Build performance critical | Metro (KSP, fast) |
| Team new to DI | Koin (simple DSL) |
| Complex Android integration | Hilt (ViewModel, WorkManager) |

## Common Tasks

### 1. Setup Hilt

**Input:** "Set up Hilt with Retrofit and Room"

**Action:**
1. Add Hilt plugin and dependencies
2. Create @HiltAndroidApp
3. Define modules with @Module and @InstallIn
4. Use @Inject in ViewModels and repositories

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit = Retrofit.Builder()...
}

@HiltViewModel
class UserViewModel @Inject constructor(
    repository: UserRepository
) : ViewModel()
```

### 2. Setup Koin

**Input:** "Set up Koin for a simple app"

**Action:**
1. Add Koin dependencies
2. Define modules with module { }
3. Start Koin in Application
4. Use by inject() or by viewModel()

```kotlin
val appModule = module {
    single { Retrofit.Builder()... }
    single<UserRepository> { UserRepositoryImpl(get()) }
    viewModel { UserViewModel(get()) }
}

class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidContext(this@MyApp)
            modules(appModule)
        }
    }
}
```

### 3. Multi-Module Setup

**Input:** "Organize DI in a multi-module app"

**Action:**
- Create :core:di module with shared dependencies
- Each feature module exposes its own module
- App module combines all modules

### 4. Testing with Mocks

**Input:** "Write tests with mocked dependencies"

**Action:**
- Hilt: Use @TestInstallIn to replace modules
- Koin: Use loadKoinModules with test doubles
- Use MockK for creating mocks

## Scoping Quick Reference

| Hilt | Koin | Metro | Use For |
|------|------|-------|---------|
| @Singleton | single | @Singleton | Repositories, APIs |
| @ActivityScoped | scope | - | Activity-bound services |
| @ViewModelScoped | viewModel | @Inject | ViewModels |
| @ActivityRetainedScoped | - | - | Survives config change |
| - | factory | @AssistedInject | New instance needed |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Missing binding | Check module installation and return types |
| Duplicate bindings | Use @Named or custom qualifier |
| Circular dependency | Refactor to break cycle, use Provider<> |
| Build slow | Switch from kapt to ksp |
| Runtime crash (Koin) | Check module list in startKoin |

## Resources

- [Hilt Testing Guide](https://developer.android.com/training/dependency-injection/hilt-testing)
- [Koin Modules](https://insert-koin.io/docs/reference/koin-core/modules/)
- [Android DI Patterns](https://developer.android.com/training/dependency-injection)
