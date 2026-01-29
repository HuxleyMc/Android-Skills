# Android Performance Profiler - Agent Guide

## Quick Start

Use this skill when:
- Investigating UI jank or frame drops
- Analyzing memory leaks
- Optimizing app startup time
- Reducing battery consumption
- Debugging Compose recomposition issues
- Improving database query performance

## Common Tasks

### 1. Profile UI Jank

**Input:** "My RecyclerView scrolls with jank"

**Action:**
1. Enable GPU profiling on device
2. Use Android Studio Profiler
3. Check for main thread blocking
4. Look at frame timing metrics

```kotlin
// Check for heavy work on main thread
// Use StrictMode to detect
StrictMode.setThreadPolicy(
    StrictMode.ThreadPolicy.Builder()
        .detectDiskReads()
        .detectDiskWrites()
        .detectNetwork()
        .penaltyLog()
        .build()
)
```

### 2. Detect Memory Leaks

**Input:** "Memory usage keeps growing"

**Action:**
1. Add LeakCanary for automatic detection
2. Analyze heap dumps in Android Studio
3. Check for retained Context references

```kotlin
// Add to build.gradle
debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.12'
```

### 3. Optimize Compose Recomposition

**Input:** "My Compose screen recomposes too often"

**Action:**
1. Enable composition tracing
2. Use `@Immutable` on data classes
3. Add stable keys to Lazy lists
4. Hoist state appropriately
5. Use `remember` and `derivedStateOf`

## Profiling Checklist

```markdown
□ Enable developer options on test device
□ Use release build with debuggable for profiling
□ Set up Macrobenchmark for metrics
□ Profile with real user scenarios
□ Measure before and after optimizations
□ Test on low-end devices
```

## Common Issues Quick Fix

| Issue | Symptom | Solution |
|-------|---------|----------|
| Main thread IO | ANR/Freeze | Move to IO dispatcher |
| Unstable Compose params | Excess recompositions | Add @Immutable |
| Large bitmaps | OOM crashes | Resize before loading |
| N+1 queries | Slow lists | Use JOIN queries |
| Missing keys | List flickering | Add stable keys |
| Early initialization | Slow startup | Use lazy/init providers |

## Resources

- [Android Profiler](https://developer.android.com/studio/profile)
- [Baseline Profiles](https://developer.android.com/topic/performance/baselineprofiles)
- [LeakCanary](https://square.github.io/leakcanary/)
