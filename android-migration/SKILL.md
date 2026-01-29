---
name: android-migration
description: Guides Android project migrations between technologies and versions. Use when migrating Java to Kotlin, upgrading Gradle/AGP, moving to AndroidX, converting XML Views to Compose, RxJava to Coroutines, or Dagger to Hilt.
tags: ["android", "migration", "kotlin", "java", "gradle", "androidx", "compose", "coroutines", "hilt"]
difficulty: advanced
category: migration
version: "1.0.0"
last_updated: "2025-01-29"
---

# Android Migration

## Quick Start

Identify migration type and follow the specific guide:

```
1. Java → Kotlin
2. Gradle/AGP Upgrade
3. AndroidX Migration
4. XML Views → Compose
5. RxJava → Coroutines/Flow
6. Dagger → Hilt
7. API Level Upgrade
```

## Migration Types

### Java to Kotlin

**Automatic Conversion:**
```kotlin
// Tools → Kotlin → Convert Java File to Kotlin
// Or: Cmd+Shift+Alt+K (Mac) / Ctrl+Shift+Alt+K (Win)
```

**Post-Conversion Cleanup:**
```kotlin
// Before (auto-converted)
val user = User()
user.name = "John"
user.email = "john@test.com"

// After (idiomatic Kotlin)
val user = User(
    name = "John",
    email = "john@test.com"
)

// Before
val names = ArrayList<String>()
for (user in users) {
    names.add(user.name)
}

// After
val names = users.map { it.name }
```

### Gradle & AGP Upgrade

**Step-by-Step Upgrade:**
```gradle
// 1. Check compatibility matrix
// https://developer.android.com/studio/releases/gradle-plugin#updating-gradle

// 2. Update gradle/wrapper/gradle-wrapper.properties
distributionUrl=https\://services.gradle.org/distributions/gradle-8.2-bin.zip

// 3. Update build.gradle (project level)
plugins {
    id 'com.android.application' version '8.2.0' apply false
    id 'org.jetbrains.kotlin.android' version '1.9.20' apply false
}

// 4. Update build.gradle (app level) - new plugin syntax
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'com.google.dagger.hilt.android'
}
```

**Breaking Changes Handling:**
```kotlin
// Namespace instead of applicationId in manifest
android {
    namespace 'com.example.app'
    
    // Compile SDK bump
    compileSdk 34
    
    defaultConfig {
        targetSdk 34
    }
}
```

### AndroidX Migration

**Migration Steps:**
```gradle
// 1. Add to gradle.properties
android.useAndroidX=true
android.enableJetifier=true

// 2. Use Refactor → Migrate to AndroidX in Android Studio

// 3. Replace support library imports:
// Before
import android.support.v7.app.AppCompatActivity
import android.support.v7.widget.RecyclerView

// After
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.RecyclerView
```

### XML Views to Compose

**Incremental Migration Strategy:**
```kotlin
// 1. Add Compose dependencies
implementation platform('androidx.compose:compose-bom:2023.10.01')
implementation 'androidx.compose.ui:ui'
implementation 'androidx.compose.material3:material3'

// 2. Create ComposeView in XML layout
<androidx.compose.ui.platform.ComposeView
    android:id="@+id/compose_view"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />

// 3. Use in Activity/Fragment
binding.composeView.setContent {
    MaterialTheme {
        UserProfile(user = viewModel.user)
    }
}
```

**View to Compose Mapping:**
```kotlin
// XML TextView → Compose Text
<TextView
    android:text="@string/hello"
    android:textSize="16sp"
    android:textColor="@color/primary" />

// Becomes
Text(
    text = stringResource(R.string.hello),
    fontSize = 16.sp,
    color = MaterialTheme.colorScheme.primary
)

// XML RecyclerView → LazyColumn
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/list"
    app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager" />

// Becomes
LazyColumn {
    items(data) { item ->
        ListItem(item)
    }
}
```

### RxJava to Coroutines/Flow

**Migration Patterns:**
```kotlin
// RxJava Single → Suspend Function
// Before
fun getUser(id: String): Single<User> = api.getUser(id)

// After
suspend fun getUser(id: String): User = api.getUser(id)

// RxJava Observable → Flow
// Before
fun getUpdates(): Observable<Update> = updateSubject.hide()

// After
fun getUpdates(): Flow<Update> = updateSubject.asFlow()

// RxJava zip/combine → Kotlin zip/combine
// Before
Observable.zip(source1, source2) { a, b -> a to b }

// After
combine(source1, source2) { a, b -> a to b }
```

**Repository Layer Migration:**
```kotlin
// Before (RxJava)
class UserRepository(private val api: Api) {
    fun getUsers(): Observable<List<User>> =
        api.getUsers()
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
}

// After (Coroutines + Flow)
class UserRepository(private val api: Api) {
    fun getUsers(): Flow<List<User>> = flow {
        emit(api.getUsers())
    }.flowOn(Dispatchers.IO)
    
    // Or for single-shot operations
    suspend fun getUser(id: String): User = 
        withContext(Dispatchers.IO) {
            api.getUser(id)
        }
}
```

### Dagger to Hilt

**Migration Steps:**
```kotlin
// 1. Update dependencies
plugins {
    id 'com.google.dagger.hilt.android' version '2.48'
}
dependencies {
    implementation 'com.google.dagger:hilt-android:2.48'
    kapt 'com.google.dagger:hilt-compiler:2.48'
}

// 2. Add Application class
@HiltAndroidApp
class MyApplication : Application()

// 3. Update manifest
<application
    android:name=".MyApplication"
    ... />

// 4. Convert Dagger modules to Hilt modules
// Before (Dagger)
@Module
abstract class AppModule {
    @Binds
    abstract fun bindRepository(repo: UserRepositoryImpl): UserRepository
}

@Component(modules = [AppModule::class])
interface AppComponent {
    fun inject(activity: MainActivity)
}

// After (Hilt)
@Module
@InstallIn(SingletonComponent::class)
abstract class AppModule {
    @Binds
    abstract fun bindRepository(repo: UserRepositoryImpl): UserRepository
}

// 5. Update injection points
// Before
class MainActivity : AppCompatActivity() {
    @Inject lateinit var viewModel: MainViewModel
    
    override fun onCreate(savedInstanceState: Bundle?) {
        DaggerAppComponent.create().inject(this)
    }
}

// After
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject lateinit var viewModel: MainViewModel
}
```

## Examples (Input → Output)

### Java to Kotlin Migration

**Input:** Convert this Java Activity to Kotlin

```java
public class MainActivity extends AppCompatActivity {
    private TextView textView;
    private Button button;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        textView = findViewById(R.id.text);
        button = findViewById(R.id.button);
        
        button.setOnClickListener(v -> {
            textView.setText("Clicked!");
        });
    }
}
```

**Output:**

```kotlin
class MainActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityMainBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        binding.button.setOnClickListener {
            binding.text.text = "Clicked!"
        }
    }
}
```

### RxJava to Coroutines Migration

**Input:** Convert this RxJava chain to Coroutines/Flow

```kotlin
fun searchUsers(query: String): Observable<List<User>> {
    return api.searchUsers(query)
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .map { response -> response.users }
        .doOnError { error -> logger.error("Search failed", error) }
}
```

**Output:**

```kotlin
fun searchUsers(query: String): Flow<List<User>> = flow {
    emit(api.searchUsers(query).users)
}.flowOn(Dispatchers.IO)
    .catch { error ->
        logger.error("Search failed", error)
        emit(emptyList())
    }
```

## Resources

- [Migrate to AndroidX](https://developer.android.com/jetpack/androidx/migrate)
- [Java to Kotlin Guide](https://developer.android.com/kotlin/add-kotlin)
- [Compose Migration](https://developer.android.com/jetpack/compose/migrate)
- [Hilt Migration](https://dagger.dev/hilt/migration-guide)
