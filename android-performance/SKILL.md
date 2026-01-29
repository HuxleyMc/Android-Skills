---
name: android-performance
description: Guides Android performance optimizations. Use when app is slow, has memory leaks, ANR issues, battery drain, or janky UI. Covers profiling, memory management, rendering optimizations, and best practices.
tags: ["android", "performance", "optimization", "profiling", "memory", "battery"]
difficulty: advanced
category: performance
version: "1.0.0"
last_updated: "2025-01-29"
---

# Android Performance Optimization

## Quick Start

Common performance issues and solutions:

| Issue | Tool | Solution |
|-------|------|----------|
| Janky UI (dropped frames) | Profile GPU Rendering | Reduce overdraw, optimize layouts |
| Memory leaks | Memory Profiler | Fix lifecycle issues, avoid static refs |
| ANR | ANR traces | Move work off main thread |
| Slow startup | Macrobenchmark | Lazy init, reduce startup work |
| Battery drain | Battery Historian | Batch work, use WorkManager |

## Profiling Tools

### Android Studio Profiler

**CPU Profiler:**
```kotlin
// Add to gradle for method tracing
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt')
        }
    }
}

// Custom trace markers
trace("Bitmap Processing") {
    processBitmap(image)
}
```

**Memory Profiler:**
```kotlin
// Force garbage collection before heap dump
Runtime.getRuntime().gc()

// Check memory before heavy operation
val runtime = Runtime.getRuntime()
val usedMem = (runtime.totalMemory() - runtime.freeMemory()) / 1024 / 1024
Log.d("Memory", "Used: ${usedMem}MB")
```

### Macrobenchmark

```kotlin
@RunWith(AndroidJUnit4::class)
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
```

## Core Patterns

### Rendering Optimization

**Reduce Overdraw:**
```kotlin
// Enable overdraw debugging in Developer Options
// Avoid multiple backgrounds

// Bad: Container with bg + Card with bg + Content with bg
Column(
    modifier = Modifier.background(Color.White)  // Remove if parent already has bg
) {
    Card(
        modifier = Modifier.background(Color.White)  // Redundant
    ) { }
}

// Good: Single background at appropriate level
Column {
    Card { }  // Card handles its own background
}
```

**Optimize RecyclerView:**
```kotlin
class ProductAdapter : ListAdapter<Product, ViewHolder>(DiffCallback()) {
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        // Inflate view
        return ViewHolder(binding)
    }
    
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val item = getItem(position)
        
        // Use setHasStableIds for better performance
        holder.bind(item)
    }
    
    class DiffCallback : DiffUtil.ItemCallback<Product>() {
        override fun areItemsTheSame(old: Product, new: Product) = 
            old.id == new.id
        
        override fun areContentsTheSame(old: Product, new: Product) = 
            old == new
    }
}

// Usage optimizations
recyclerView.apply {
    setHasFixedSize(true)  // If layout size doesn't change
    layoutManager = LinearLayoutManager(context)
    adapter = productAdapter
    
    // Use appropriate prefetch
    (layoutManager as LinearLayoutManager).apply {
        initialPrefetchItemCount = 4
    }
}
```

**ViewHolder Pattern:**
```kotlin
class ProductViewHolder(
    private val binding: ItemProductBinding
) : RecyclerView.ViewHolder(binding.root) {
    
    private val imageLoader = ImageLoader.getInstance()
    
    fun bind(product: Product) {
        binding.name.text = product.name
        
        // Cancel previous load to prevent wrong image showing
        imageLoader.cancel(binding.image)
        imageLoader.load(product.imageUrl, binding.image)
    }
}
```

### Memory Management

**Bitmap Optimization:**
```kotlin
object BitmapUtils {
    
    fun decodeSampledBitmap(
        filePath: String,
        reqWidth: Int,
        reqHeight: Int
    ): Bitmap? {
        return BitmapFactory.Options().run {
            inJustDecodeBounds = true
            BitmapFactory.decodeFile(filePath, this)
            
            inSampleSize = calculateInSampleSize(this, reqWidth, reqHeight)
            inJustDecodeBounds = false
            inPreferredConfig = Bitmap.Config.RGB_565  // Use lower color depth if acceptable
            
            BitmapFactory.decodeFile(filePath, this)
        }
    }
    
    private fun calculateInSampleSize(
        options: BitmapFactory.Options,
        reqWidth: Int,
        reqHeight: Int
    ): Int {
        val (height, width) = options.run { outHeight to outWidth }
        var inSampleSize = 1
        
        if (height > reqHeight || width > reqWidth) {
            val halfHeight = height / 2
            val halfWidth = width / 2
            
            while (halfHeight / inSampleSize >= reqHeight &&
                   halfWidth / inSampleSize >= reqWidth) {
                inSampleSize *= 2
            }
        }
        return inSampleSize
    }
}

// Usage
val bitmap = BitmapUtils.decodeSampledBitmap(
    imagePath,
    imageView.width,
    imageView.height
)
imageView.setImageBitmap(bitmap)
```

**Memory Leak Prevention:**
```kotlin
// Bad: Static reference to Activity
object BadCache {
    var activity: Activity? = null  // Memory leak!
}

// Good: Use ApplicationContext and WeakReference
class ImageCache(context: Context) {
    private val context = context.applicationContext
    private val cache = LruCache<String, Bitmap>(
        (Runtime.getRuntime().maxMemory() / 1024 / 8).toInt()
    )
    
    fun put(key: String, bitmap: Bitmap) {
        cache.put(key, bitmap)
    }
    
    fun get(key: String): Bitmap? = cache.get(key)
}

// Lifecycle-aware cleanup
class ImageLoader(private val lifecycle: Lifecycle) {
    private val activeLoads = mutableListOf<Job>()
    
    init {
        lifecycle.addObserver(object : DefaultLifecycleObserver {
            override fun onDestroy(owner: LifecycleOwner) {
                activeLoads.forEach { it.cancel() }
                activeLoads.clear()
            }
        })
    }
}
```

**Object Pooling:**
```kotlin
class PaintPool {
    private val pool = ArrayDeque<Paint>(10)
    
    fun obtain(): Paint {
        return pool.removeFirstOrNull() ?: Paint().apply {
            isAntiAlias = true
        }
    }
    
    fun recycle(paint: Paint) {
        if (pool.size < 10) {
            pool.addLast(paint)
        }
    }
}

// Usage in custom view
override fun onDraw(canvas: Canvas) {
    val paint = paintPool.obtain()
    try {
        paint.color = Color.RED
        canvas.drawCircle(x, y, radius, paint)
    } finally {
        paintPool.recycle(paint)
    }
}
```

### Threading Optimization

**Coroutine Dispatchers:**
```kotlin
// CPU-intensive: Use Default
coroutineScope.launch(Dispatchers.Default) {
    val result = heavyComputation()
}

// IO operations: Use IO
suspend fun fetchData(): Data = withContext(Dispatchers.IO) {
    api.fetchData()
}

// UI updates: Main (default for ViewModel scope)
viewModelScope.launch {
    val data = withContext(Dispatchers.IO) { repository.fetch() }
    updateUI(data)  // Back on Main
}
```

**WorkManager for Deferred Work:**
```kotlin
class SyncWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        return try {
            repository.syncData()
            Result.success()
        } catch (e: Exception) {
            if (runAttemptCount < 3) {
                Result.retry()
            } else {
                Result.failure()
            }
        }
    }
}

// Schedule work
val syncWork = PeriodicWorkRequestBuilder<SyncWorker>(
    15, TimeUnit.MINUTES
).setConstraints(
    Constraints.Builder()
        .setRequiredNetworkType(NetworkType.UNMETERED)
        .setRequiresBatteryNotLow(true)
        .build()
).build()

WorkManager.getInstance(context).enqueueUniquePeriodicWork(
    "sync",
    ExistingPeriodicWorkPolicy.KEEP,
    syncWork
)
```

### Battery Optimization

**Batch Network Requests:**
```kotlin
class BatchedUploader(context: Context) {
    private val queue = mutableListOf<Event>()
    private val scope = CoroutineScope(SupervisorJob() + Dispatchers.IO)
    
    fun addEvent(event: Event) {
        queue.add(event)
        scheduleUpload()
    }
    
    private fun scheduleUpload() {
        scope.launch {
            delay(5000)  // Wait 5s to batch
            val batch = queue.toList()
            queue.clear()
            
            if (batch.isNotEmpty()) {
                uploadBatch(batch)
            }
        }
    }
}
```

**Avoid Wake Locks:**
```kotlin
// Bad: Keeping device awake
wakeLock.acquire(10*60*1000L)  // Don't do this

// Good: Use WorkManager or FCM high priority
val workRequest = OneTimeWorkRequestBuilder<UploadWorker>()
    .setExpedited(OutOfQuotaPolicy.RUN_AS_NON_EXPEDITED_WORK_REQUEST)
    .build()
```

## Examples (Input → Output)

### Optimizing RecyclerView

**Input request:** "My RecyclerView is laggy when scrolling, how do I optimize it?"

**Output:**
```kotlin
class OptimizedProductAdapter : ListAdapter<Product, ViewHolder>(DiffCallback()) {
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val binding = ItemProductBinding.inflate(
            LayoutInflater.from(parent.context), parent, false
        )
        return ViewHolder(binding)
    }
    
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(getItem(position))
    }
    
    class ViewHolder(private val binding: ItemProductBinding) : 
        RecyclerView.ViewHolder(binding.root) {
        
        fun bind(product: Product) {
            binding.name.text = product.name
            
            // Use image loading library with caching
            Glide.with(binding.image)
                .load(product.imageUrl)
                .placeholder(R.drawable.placeholder)
                .centerCrop()
                .into(binding.image)
        }
    }
    
    class DiffCallback : DiffUtil.ItemCallback<Product>() {
        override fun areItemsTheSame(old: Product, new: Product) = 
            old.id == new.id
        override fun areContentsTheSame(old: Product, new: Product) = 
            old == new
    }
}

// Fragment setup
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    binding.recyclerView.apply {
        setHasFixedSize(true)  // Optimization if layout size is fixed
        layoutManager = LinearLayoutManager(context)
        adapter = productAdapter
        
        // Enable view recycling
        recycledViewPool.setMaxRecycledViews(0, 10)
        
        // Use prefetch for smoother scrolling
        (layoutManager as LinearLayoutManager).apply {
            initialPrefetchItemCount = 4
        }
    }
    
    // Submit list (diffutil handles updates efficiently)
    viewModel.products.observe(viewLifecycleOwner) { list ->
        productAdapter.submitList(list)
    }
}
```

### Finding Memory Leak

**Input request:** "App memory keeps growing, how do I find the leak?"

**Output:**
```kotlin
// Step 1: Check for static references
class BadExample {
    companion object {
        // Memory leak - static reference to Activity
        var activityRef: MainActivity? = null
    }
}

// Step 2: Fix with WeakReference or proper lifecycle
class GoodExample(lifecycle: Lifecycle) {
    private val listeners = mutableListOf<WeakReference<Listener>>()
    
    init {
        lifecycle.addObserver(object : DefaultLifecycleObserver {
            override fun onDestroy(owner: LifecycleOwner) {
                listeners.clear()
            }
        })
    }
}

// Step 3: Use LeakCanary for detection
// In Application.onCreate()
if (BuildConfig.DEBUG) {
    LeakCanary.install(this)
}

// Step 4: Common leak patterns to check
// - Anonymous inner classes referencing outer class
// - Handlers without WeakReference
// - Listeners not unregistered
// - RxJava/Coroutines not cancelled

class SafeHandler(
    lifecycle: Lifecycle,
    private val callback: () -> Unit
) : Handler(Looper.getMainLooper()) {
    
    private val runnable = Runnable { callback() }
    
    init {
        lifecycle.addObserver(object : DefaultLifecycleObserver {
            override fun onDestroy(owner: LifecycleOwner) {
                removeCallbacks(runnable)
            }
        })
    }
    
    fun postDelayed(delay: Long) {
        postDelayed(runnable, delay)
    }
}
```

## Best Practices

1. **Profile before optimizing**: Don't guess, measure with Profiler
2. **Use RecyclerView.ViewHolder**: Proper recycling prevents object churn
3. **Image sizing**: Load images at display size, not full resolution
4. **Avoid object allocation in onDraw**: No `new Paint()`, `new Path()` in draw loops
5. **Lazy initialization**: Initialize heavy objects only when needed
6. **Use ConstraintLayout**: Reduces view hierarchy depth vs nested layouts
7. **Background work**: Never block main thread, use coroutines/WorkManager
8. **Memory caches**: Use LruCache for bitmaps and expensive objects
9. **Release resources**: Cancel coroutines, unregister listeners, close cursors
10. **Test on low-end devices**: Optimize for the worst case, not your flagship

## Resources

- [Profiling tools](https://developer.android.com/studio/profile)
- [Macrobenchmark library](https://developer.android.com/topic/performance/benchmarking/macrobenchmark-overview)
- [Memory management](https://developer.android.com/topic/performance/memory)
