---
name: android-performance-profiler
description: Analyzes and optimizes Android app performance. Use when identifying UI jank, memory leaks, slow startup, high battery drain, or Compose recomposition issues. Covers profiling tools, benchmarks, and optimization techniques.
tags: ["android", "performance", "profiling", "benchmark", "compose", "memory", "optimization"]
difficulty: advanced
category: performance
version: "1.0.0"
last_updated: "2025-01-29"
---

# Android Performance Profiler

## Quick Start

Identify performance issue type:

```
1. UI Jank / Frame Drops
2. Memory Leaks
3. Slow Startup
4. High Battery Drain
5. Database Slowness
6. Compose Recomposition
7. Network Performance
```

## Profiling Tools

### Android Studio Profiler

```kotlin
// CPU Profiling - identify hotspots
// 1. Build with debuggable true
// 2. Run app with profiler attached
// 3. Record CPU trace during slow operation
// 4. Analyze call stack for long-running methods

// Memory Profiling
// 1. Use Memory Profiler to track allocations
// 2. Look for memory spikes
// 3. Capture heap dump for analysis
// 4. Identify retained objects
```

### Macrobenchmark

```kotlin
// build.gradle
androidTestImplementation("androidx.benchmark:benchmark-macro-junit4:1.2.0")

// Startup benchmark
class StartupBenchmark {
    @get:Rule
    val benchmarkRule = MacrobenchmarkRule()

    @Test
    fun startup() = benchmarkRule.measureRepeated(
        packageName = "com.example.app",
        metrics = listOf(StartupTimingMetric()),
        iterations = 5,
        startupMode = StartupMode.COLD
    ) {
        pressHome()
        startActivityAndWait()
    }
}

// Frame metrics benchmark
class FrameBenchmark {
    @get:Rule
    val benchmarkRule = MacrobenchmarkRule()

    @Test
    fun scrollList() = benchmarkRule.measureRepeated(
        packageName = "com.example.app",
        metrics = listOf(FrameTimingMetric(), JankMetric()),
        iterations = 5
    ) {
        val recycler = device.findObject(By.res("recycler"))
        recycler.setGestureMargin(device.displayWidth / 5)
        recycler.fling(Direction.DOWN, 5000)
    }
}
```

### Compose Metrics

```kotlin
// Enable in build.gradle
android {
    buildTypes {
        debug {
            composeOptions {
                metricsDestination = layout.buildDirectory.dir("compose_metrics")
            }
        }
    }
}

// Runtime tracking
class CompositionLogger : CompositionObserver {
    override fun onComposition(content: @Composable () -> Unit) {
        val start = System.nanoTime()
        content()
        val duration = (System.nanoTime() - start) / 1_000_000
        if (duration > 16) {
            Log.w("Composition", "Slow composition: ${duration}ms")
        }
    }
}
```

## Performance Patterns

### UI Jank Prevention

```kotlin
// Bad: Heavy work on main thread
@Composable
fun BadList() {
    val items = remember {
        heavyComputation() // ❌ Blocks composition
    }
}

// Good: Offload to background
@Composable
fun GoodList() {
    val items by produceState<List<Item>>(initialValue = emptyList()) {
        value = withContext(Dispatchers.Default) {
            heavyComputation()
        }
    }
}

// Good: Lazy evaluation
LazyColumn {
    items(data) { item ->
        ListItem(item)
    }
}
```

### Compose Recomposition Optimization

```kotlin
// Bad: Unstable parameter causes recomposition
@Composable
fun UserCard(user: User) { // User is unstable
    Column {
        Text(user.name)
        Text(user.email)
    }
}

// Good: Make data class immutable and stable
@Immutable
data class User(
    val id: String,
    val name: String,
    val email: String
)

// Good: Use stable keys
LazyColumn {
    items(
        items = users,
        key = { it.id }
    ) { user ->
        UserCard(user = user)
    }
}

// Good: Remember expensive calculations
@Composable
fun ExpensiveCalculation(data: List<Int>) {
    val sum = remember(data) {
        data.sum()
    }
    Text("Sum: $sum")
}

// Good: Derive state to minimize recompositions
@Composable
fun SearchResults(query: String, items: List<Item>) {
    val filteredItems by remember(query, items) {
        derivedStateOf {
            items.filter { it.name.contains(query, ignoreCase = true) }
        }
    }
    
    LazyColumn {
        items(filteredItems) { item ->
            ListItem(item)
        }
    }
}
```

### Memory Leak Prevention

```kotlin
// Bad: Holding Activity reference in ViewModel
class BadViewModel : ViewModel() {
    var activity: MainActivity? = null // ❌ Memory leak
}

// Good: Use Application context
class GoodRepository(application: Application) {
    private val context = application.applicationContext
}

// Bad: Not cleaning up callbacks
class BadManager {
    fun register(listener: Listener) {
        listeners.add(listener) // Never removed!
    }
}

// Good: Proper cleanup with lifecycle awareness
class GoodManager {
    fun register(owner: LifecycleOwner, listener: Listener) {
        owner.lifecycle.addObserver(object : DefaultLifecycleObserver {
            override fun onDestroy(owner: LifecycleOwner) {
                listeners.remove(listener)
            }
        })
        listeners.add(listener)
    }
}

// Good: Use WeakReference when necessary
class WeakCallback<T>(callback: Callback<T>) {
    private val weakRef = WeakReference(callback)
    
    fun invoke(value: T) {
        weakRef.get()?.onResult(value)
    }
}
```

### Startup Optimization

```kotlin
// Bad: Heavy initialization in Application.onCreate
class BadApp : Application() {
    override fun onCreate() {
        super.onCreate()
        initDatabase()      // ❌ Blocks startup
        initAnalytics()     // ❌ Blocks startup
        initCrashReporter() // ❌ Blocks startup
    }
}

// Good: Lazy initialization
class GoodApp : Application() {
    val database: AppDatabase by lazy {
        Room.databaseBuilder(this, AppDatabase::class.java, "db").build()
    }
    
    override fun onCreate() {
        super.onCreate()
        // Only critical initialization here
        
        // Defer non-critical init
        GlobalScope.launch(Dispatchers.Default) {
            initAnalytics()
            initCrashReporter()
        }
    }
}

// Good: ContentProvider for early init
class AnalyticsInitializer : Initializer<Analytics> {
    override fun create(context: Context): Analytics {
        return Analytics.initialize(context)
    }
    
    override fun dependencies(): List<Class<out Initializer<*>>> {
        return emptyList()
    }
}

// AndroidManifest.xml
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"
    tools:node="merge">
    <meta-data
        android:name="com.example.AnalyticsInitializer"
        android:value="androidx.startup" />
</provider>
```

### Database Optimization

```kotlin
// Bad: Query in loop
fun getUsersWithOrders(userIds: List<String>): List<UserWithOrders> {
    return userIds.map { id ->
        val user = db.userDao().getUser(id)          // ❌ N+1 query
        val orders = db.orderDao().getOrdersForUser(id)
        UserWithOrders(user, orders)
    }
}

// Good: Single query with JOIN
@Query("""
    SELECT * FROM users 
    INNER JOIN orders ON users.id = orders.userId 
    WHERE users.id IN (:userIds)
""
)
suspend fun getUsersWithOrders(userIds: List<String>): Map<User, List<Order>>

// Bad: Loading all data
@Query("SELECT * FROM logs")
suspend fun getAllLogs(): List<Log>

// Good: Pagination
@Query("SELECT * FROM logs ORDER BY timestamp DESC LIMIT :limit OFFSET :offset")
suspend fun getLogsPaged(limit: Int, offset: Int): List<Log>

// Bad: No index on queried columns
@Entity(tableName = "users")
data class User(
    @PrimaryKey val id: String,
    val email: String  // Queried frequently
)

// Good: Add index
@Entity(
    tableName = "users",
    indices = [Index(value = ["email"])]
)
data class User(
    @PrimaryKey val id: String,
    val email: String
)
```

### Battery Optimization

```kotlin
// Bad: Frequent polling
class BadLocationTracker {
    fun startTracking() {
        handler.postDelayed({
            fetchLocation()
            startTracking() // ❌ Continuous polling
        }, 5000)
    }
}

// Good: Use WorkManager for background work
class LocationWorker(context: Context, params: WorkerParameters) : 
    CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        return try {
            fetchLocation()
            Result.success()
        } catch (e: Exception) {
            Result.retry()
        }
    }
    
    companion object {
        fun schedule() {
            val request = PeriodicWorkRequestBuilder<LocationWorker>(
                15, TimeUnit.MINUTES
            ).setConstraints(
                Constraints.Builder()
                    .setRequiredNetworkType(NetworkType.CONNECTED)
                    .setRequiresBatteryNotLow(true)
                    .build()
            ).build()
            
            WorkManager.getInstance().enqueueUniquePeriodicWork(
                "location_work",
                ExistingPeriodicWorkPolicy.KEEP,
                request
            )
        }
    }
}

// Good: Batch network requests
class RequestBatcher<T>(
    private val scope: CoroutineScope,
    private val timeout: Long = 100,
    private val processor: suspend (List<T>) -> Unit
) {
    private val channel = Channel<T>(Channel.UNLIMITED)
    
    init {
        scope.launch {
            val batch = mutableListOf<T>()
            whileSelect {
                channel.onReceive {
                    batch.add(it)
                    batch.size < 100
                }
                onTimeout(timeout) { false }
            }
            if (batch.isNotEmpty()) {
                processor(batch)
            }
        }
    }
    
    fun add(item: T) {
        channel.trySend(item)
    }
}
```

## Examples (Input → Output)

### Analyze Compose Recomposition

**Input:** "This screen has jank during scrolling"

```kotlin
@Composable
fun ProductList(products: List<Product>) {
    LazyColumn {
        items(products) { product ->
            ProductCard(product = product)
        }
    }
}

@Composable
fun ProductCard(product: Product) {
    Card {
        Column {
            AsyncImage(model = product.imageUrl)
            Text(product.name)
            Text("$${product.price}")
            RatingBar(rating = calculateRating(product.reviews))
        }
    }
}
```

**Output:**

```kotlin
// Issues identified:
// 1. Product is likely unstable (mutable or non-@Immutable)
// 2. calculateRating called during composition
// 3. No keys for LazyColumn items

// Optimized version:
@Immutable
data class Product(
    val id: String,
    val name: String,
    val price: Double,
    val imageUrl: String,
    val rating: Float  // Pre-calculated
)

@Composable
fun ProductList(products: List<Product>) {
    LazyColumn {
        items(
            items = products,
            key = { it.id }
        ) { product ->
            ProductCard(product = product)
        }
    }
}

@Composable
fun ProductCard(product: Product) {
    Card {
        Column {
            AsyncImage(
                model = product.imageUrl,
                contentDescription = null
            )
            Text(product.name)
            Text("$${product.price}")
            RatingBar(rating = product.rating)
        }
    }
}
```

## Resources

- [Performance Best Practices](https://developer.android.com/topic/performance)
- [Compose Performance](https://developer.android.com/jetpack/compose/performance)
- [Benchmarking](https://developer.android.com/topic/performance/benchmarking)
- [Inspect CPU Activity](https://developer.android.com/studio/profile/cpu-profiler)
