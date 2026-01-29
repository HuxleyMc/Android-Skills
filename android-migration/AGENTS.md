# Android Migration - Agent Guide

## Quick Start

Use this skill when:
- Migrating Java code to Kotlin
- Upgrading Gradle or Android Gradle Plugin
- Moving from Support Library to AndroidX
- Converting XML layouts to Jetpack Compose
- Replacing RxJava with Kotlin Coroutines/Flow
- Migrating from Dagger to Hilt
- Upgrading target/compile SDK versions

## Common Tasks

### 1. Java to Kotlin Migration

**Input:** "Convert this Java class to Kotlin"

**Action:**
1. Use Android Studio's automatic converter
2. Clean up to idiomatic Kotlin
3. Apply scope functions and null safety

```kotlin
// Auto-converted (cleanup needed)
val user = User()
user.name = name
user.email = email

// Idiomatic Kotlin
val user = User(name = name, email = email)
```

### 2. Gradle/AGP Upgrade

**Input:** "Upgrade from AGP 7.4 to 8.2"

**Action:**
1. Check compatibility matrix
2. Update gradle wrapper
3. Update plugin versions
4. Fix breaking changes (namespace, non-transitive R class)

### 3. XML to Compose Migration

**Input:** "Convert this RecyclerView to Compose"

**Action:**
1. Add Compose dependencies
2. Create equivalent Composable
3. Replace in layout or use ComposeView bridge

```kotlin
// XML RecyclerView
<RecyclerView
    android:id="@+id/list"
    app:layoutManager="LinearLayoutManager" />

// Compose equivalent
LazyColumn {
    items(data, key = { it.id }) { item ->
        ListItem(item)
    }
}
```

### 4. RxJava to Coroutines

**Input:** "Convert this RxJava chain to Flow"

**Action:**
- `Observable<T>` → `Flow<T>`
- `Single<T>` → `suspend` function returning `T`
- `Completable` → `suspend` function returning `Unit`
- `zip/combine` → `combine()` flow operator
- `subscribeOn` → `flowOn(Dispatchers.IO)`

## Migration Checklist

```markdown
□ Update dependencies/versions
□ Apply code changes
□ Update imports
□ Fix compilation errors
□ Run tests
□ Verify functionality
□ Update documentation
```

## Breaking Changes Quick Reference

| Migration | Common Issues |
|-----------|--------------|
| AGP 7→8 | Namespace required, nonTransitiveRClass |
| SDK 33→34 | Notification permissions, foreground services |
| Hilt 2.44→2.48 | KSP support, @HiltViewModel |
| Compose 1.4→1.5 | AnimatedContent API changes |

## Resources

- [AndroidX Migration](https://developer.android.com/jetpack/androidx/migrate)
- [Compose Interop](https://developer.android.com/jetpack/compose/migrate/interoperability-apis)
