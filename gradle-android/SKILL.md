---
name: gradle-android
description: Guides Gradle configuration and usage for Android projects. Use when configuring builds, managing dependencies, setting up build variants, optimizing build performance, or troubleshooting Gradle issues. Covers Groovy and Kotlin DSL, plugins, tasks, and Android-specific configurations.
tags: ["gradle", "android", "build", "dependencies", "kotlin-dsl", "groovy"]
difficulty: intermediate
category: build-tools
version: "1.0.0"
last_updated: "2025-01-29"
---

# Gradle for Android

## Quick Start

Basic Android project structure:

```
project/
├── build.gradle(.kts)          # Root build script
├── settings.gradle(.kts)       # Project settings
├── gradle.properties           # Build properties
├── local.properties            # Local SDK paths (gitignored)
└── app/
    ├── build.gradle(.kts)      # Module build script
    └── src/
```

**Choose DSL:**

| Groovy DSL | Kotlin DSL |
|------------|------------|
| `build.gradle` | `build.gradle.kts` |
| `id 'plugin'` | `id("plugin")` |
| `key value` | `key = value` |
| `classpath 'x:y:z'` | `classpath("x:y:z")` |

**Root build.gradle.kts:**

```kotlin
plugins {
    id("com.android.application") version "8.2.0" apply false
    id("com.android.library") version "8.2.0" apply false
    id("org.jetbrains.kotlin.android") version "1.9.20" apply false
}

allprojects {
    repositories {
        google()
        mavenCentral()
    }
}
```

**settings.gradle.kts:**

```kotlin
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "MyApp"
include(":app")
include(":core")
include(":feature:home")
```

## Core Patterns

### Module Build Configuration

**Application module:**

```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("com.google.dagger.hilt.android")
}

android {
    namespace = "com.example.myapp"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.myapp"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        
        // Build config fields
        buildConfigField("String", "API_URL", "\"https://api.example.com/\"")
        
        // Resource values
        resValue("string", "app_name", "MyApp")
        
        // Vector drawables
        vectorDrawables {
            useSupportLibrary = true
        }
    }

    buildTypes {
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            signingConfig = signingConfigs.getByName("release")
        }
        
        debug {
            isDebuggable = true
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-debug"
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
    }

    buildFeatures {
        buildConfig = true
        viewBinding = true
        compose = true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.8"
    }

    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
        }
    }
}

dependencies {
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.appcompat:appcompat:1.6.1")
}
```

**Library module:**

```kotlin
plugins {
    id("com.android.library")
    id("org.jetbrains.kotlin.android")
}

android {
    namespace = "com.example.core"
    compileSdk = 34

    defaultConfig {
        minSdk = 24
        
        consumerProguardFiles("consumer-rules.pro")
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```

### Dependency Management

**Basic dependencies:**

```kotlin
dependencies {
    // Core Android
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.appcompat:appcompat:1.6.1")
    implementation("com.google.android.material:material:1.11.0")
    
    // Architecture Components
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.6.2")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.6.2")
    
    // Navigation
    implementation("androidx.navigation:navigation-fragment-ktx:2.7.6")
    implementation("androidx.navigation:navigation-ui-ktx:2.7.6")
    
    // Compose
    implementation(platform("androidx.compose:compose-bom:2023.10.01"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.material3:material3")
    
    // Networking
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
    
    // DI
    implementation("com.google.dagger:hilt-android:2.48")
    kapt("com.google.dagger:hilt-compiler:2.48")
    
    // Image loading
    implementation("io.coil-kt:coil-compose:2.5.0")
    
    // Database
    implementation("androidx.room:room-runtime:2.6.1")
    kapt("androidx.room:room-compiler:2.6.1")
    implementation("androidx.room:room-ktx:2.6.1")
    
    // Testing
    testImplementation("junit:junit:4.13.2")
    testImplementation("io.mockk:mockk:1.13.8")
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
}
```

**Version catalogs (recommended):**

```toml
# gradle/libs.versions.toml
[versions]
agp = "8.2.0"
kotlin = "1.9.20"
compose-bom = "2023.10.01"
hilt = "2.48"

[libraries]
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version = "1.12.0" }
androidx-lifecycle-viewmodel = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-ktx", version = "2.6.2" }

compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }

hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }

retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version = "2.9.0" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
```

```kotlin
// build.gradle.kts
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.hilt)
}

dependencies {
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.viewmodel)
    
    implementation(platform(libs.compose.bom))
    implementation(libs.compose.ui)
    implementation(libs.compose.material3)
    
    implementation(libs.hilt.android)
    kapt(libs.hilt.compiler)
    
    implementation(libs.retrofit)
}
```

**Dependency configurations:**

```kotlin
dependencies {
    // Available to all configurations that extend implementation
    implementation("androidx.core:core-ktx:1.12.0")
    
    // Available at compile time, not in final APK
    compileOnly("javax.annotation:jsr250-api:1.0")
    
    // Only for tests
    testImplementation("junit:junit:4.13.2")
    
    // Only for Android tests
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
    
    // Annotation processors
    kapt("com.google.dagger:hilt-compiler:2.48")
    
    // Platform constraints
    implementation(platform("androidx.compose:compose-bom:2023.10.01"))
    
    // Module dependencies
    implementation(project(":core"))
    implementation(project(":feature:home"))
}
```

### Build Variants & Flavors

**Product flavors:**

```kotlin
android {
    flavorDimensions += listOf("tier", "market")
    
    productFlavors {
        create("free") {
            dimension = "tier"
            applicationIdSuffix = ".free"
            versionNameSuffix = "-free"
            
            buildConfigField("boolean", "PREMIUM_FEATURES", "false")
        }
        
        create("premium") {
            dimension = "tier"
            applicationIdSuffix = ".premium"
            versionNameSuffix = "-premium"
            
            buildConfigField("boolean", "PREMIUM_FEATURES", "true")
        }
        
        create("playStore") {
            dimension = "market"
        }
        
        create("amazon") {
            dimension = "market"
        }
    }
}
```

**Build variants matrix:**
```
Build Type: debug, release
Flavor: free, premium
Market: playStore, amazon

Results in: freePlayStoreDebug, freePlayStoreRelease, premiumAmazonDebug, etc.
```

**Source sets:**

```
src/
├── main/                    # Common code
├── free/                    # Free flavor only
├── premium/                 # Premium flavor only
├── playStore/               # Play Store market only
├── debug/                   # Debug build type only
└── freePlayStoreDebug/      # Specific variant
```

```kotlin
android {
    sourceSets {
        getByName("main").java.srcDirs("src/main/java")
        getByName("free").res.srcDirs("src/free/res")
    }
}
```

### Signing Configuration

```kotlin
android {
    signingConfigs {
        create("release") {
            storeFile = file("myapp.keystore")
            storePassword = System.getenv("STORE_PASSWORD")
            keyAlias = "release"
            keyPassword = System.getenv("KEY_PASSWORD")
        }
    }
    
    buildTypes {
        getByName("release") {
            signingConfig = signingConfigs.getByName("release")
        }
    }
}
```

**Using local.properties (don't commit):**

```kotlin
// build.gradle.kts
val localProperties = Properties().apply {
    load(rootProject.file("local.properties").inputStream())
}

android {
    signingConfigs {
        create("release") {
            storeFile = file(localProperties.getProperty("storeFile"))
            storePassword = localProperties.getProperty("storePassword")
            keyAlias = localProperties.getProperty("keyAlias")
            keyPassword = localProperties.getProperty("keyPassword")
        }
    }
}
```

## Common Patterns

### Custom Tasks

```kotlin
// Register custom task
tasks.register<Copy>("copyReleaseApk") {
    from("build/outputs/apk/release")
    into("dist")
    include("*.apk")
    dependsOn("assembleRelease")
}

// Task with custom logic
tasks.register("generateVersionFile") {
    doLast {
        val versionFile = file("src/main/assets/version.txt")
        versionFile.parentFile.mkdirs()
        versionFile.writeText(android.defaultConfig.versionName ?: "unknown")
    }
}

// Pre-build task
tasks.named("preBuild").configure {
    dependsOn("generateVersionFile")
}
```

### Build Performance

**gradle.properties:**

```properties
# Gradle daemon
org.gradle.daemon=true

# Parallel execution
org.gradle.parallel=true
org.gradle.configureondemand=true

# Memory settings
org.gradle.jvmargs=-Xmx8192m -XX:MaxMetaspaceSize=512m -XX:+HeapDumpOnOutOfMemoryError

# Android settings
android.useAndroidX=true
android.enableJetifier=true

# Build cache
org.gradle.caching=true
android.enableBuildCache=true

# Configuration cache (experimental)
org.gradle.configuration-cache=true
org.gradle.configuration-cache.problems=warn

# Non-transitive R classes
android.nonTransitiveRClass=true

# Disable unnecessary build features
android.defaults.buildfeatures.buildconfig=true
android.defaults.buildfeatures.aidl=false
android.defaults.buildfeatures.renderscript=false
android.defaults.buildfeatures.resvalues=false
android.defaults.buildfeatures.shaders=false
```

**Module-level optimizations:**

```kotlin
android {
    buildFeatures {
        // Disable unused features
        aidl = false
        renderScript = false
        shaders = false
        
        // Enable as needed
        buildConfig = true
        viewBinding = true
    }
    
    packagingOptions {
        // Exclude duplicate files
        resources {
            excludes += listOf(
                "/META-INF/DEPENDENCIES",
                "/META-INF/LICENSE",
                "/META-INF/LICENSE.txt",
                "/META-INF/NOTICE",
                "/META-INF/NOTICE.txt"
            )
        }
    }
}
```

### ProGuard/R8 Configuration

```proguard
# proguard-rules.pro

# Keep annotations
-keepattributes *Annotation*

# Keep Kotlin metadata
-keep class kotlin.Metadata { *; }

# Keep data classes
-keepclassmembers class com.example.** {
    *** get*();
    void set*(***);
}

# Keep for Retrofit
-keep class retrofit2.** { *; }
-keepattributes Signature
-keepattributes Exceptions

# Keep for Room
-keep class * extends androidx.room.RoomDatabase
-keep @androidx.room.Entity class *

# Keep for Parcelable
-keepclassmembers class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator CREATOR;
}

# Keep for Serializable
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

# Remove logging
-assumenosideeffects class android.util.Log {
    public static boolean isLoggable(java.lang.String, int);
    public static int v(...);
    public static int d(...);
    public static int i(...);
}
```

### Multi-Module Setup

**settings.gradle.kts:**

```kotlin
rootProject.name = "MyApp"

include(":app")
include(":core:network")
include(":core:database")
include(":core:ui")
include(":feature:home")
include(":feature:profile")
include(":feature:settings")
```

**Common convention plugin:**

```kotlin
// buildSrc/src/main/kotlin/AndroidApplicationConventionPlugin.kt
class AndroidApplicationConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) = with(target) {
        with(pluginManager) {
            apply("com.android.application")
            apply("org.jetbrains.kotlin.android")
        }
        
        extensions.configure<ApplicationExtension> {
            compileSdk = 34
            
            defaultConfig {
                minSdk = 24
                targetSdk = 34
            }
            
            compileOptions {
                sourceCompatibility = JavaVersion.VERSION_17
                targetCompatibility = JavaVersion.VERSION_17
            }
        }
    }
}
```

## Examples (Input → Output)

### Adding New Module

**Input request:** "Create a new feature module with Compose and Hilt"

**Output:**

```kotlin
// settings.gradle.kts
include(":feature:products")

// feature/products/build.gradle.kts
plugins {
    id("com.android.library")
    id("org.jetbrains.kotlin.android")
    id("com.google.dagger.hilt.android")
}

android {
    namespace = "com.example.feature.products"
    compileSdk = 34

    defaultConfig {
        minSdk = 24
        
        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        consumerProguardFiles("consumer-rules.pro")
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
    }

    buildFeatures {
        compose = true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.8"
    }
}

dependencies {
    // Module dependencies
    implementation(project(":core:ui"))
    implementation(project(":core:network"))
    
    // Compose
    implementation(platform("androidx.compose:compose-bom:2023.10.01"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.compose.ui:ui-tooling-preview")
    debugImplementation("androidx.compose.ui:ui-tooling")
    
    // Hilt
    implementation("com.google.dagger:hilt-android:2.48")
    kapt("com.google.dagger:hilt-compiler:2.48")
    implementation("androidx.hilt:hilt-navigation-compose:1.1.0")
    
    // ViewModel
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.6.2")
    
    // Testing
    testImplementation("junit:junit:4.13.2")
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
}
```

### Migrating to Version Catalogs

**Input request:** "Convert this Groovy build.gradle to Kotlin DSL with version catalogs"

**Given:**
```groovy
// build.gradle (Groovy)
plugins {
    id 'com.android.application' version '8.2.0' apply false
}

dependencies {
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.6.2'
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    testImplementation 'junit:junit:4.13.2'
}
```

**Output:**

```toml
# gradle/libs.versions.toml
[versions]
agp = "8.2.0"
core-ktx = "1.12.0"
lifecycle = "2.6.2"
retrofit = "2.9.0"
junit = "4.13.2"

[libraries]
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "core-ktx" }
androidx-lifecycle-viewmodel = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-ktx", version.ref = "lifecycle" }
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
junit = { group = "junit", name = "junit", version.ref = "junit" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
```

```kotlin
// build.gradle.kts (Kotlin DSL)
plugins {
    alias(libs.plugins.android.application) apply false
}

// app/build.gradle.kts
dependencies {
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.viewmodel)
    implementation(libs.retrofit)
    testImplementation(libs.junit)
}
```

## Best Practices

1. **Use Kotlin DSL**: Better IDE support, type safety
2. **Use version catalogs**: Centralize dependency versions
3. **Enable build cache**: Faster incremental builds
4. **Disable unused features**: Reduces build time
5. **Non-transitive R class**: Faster builds, smaller R files
6. **Incremental annotation processing**: Enable Kapt incremental mode
7. **Parallel execution**: Enable parallel and configure-on-demand
8. **Memory tuning**: Allocate appropriate heap size (4-8GB)
9. **CI optimization**: Use Gradle remote build cache
10. **Module boundaries**: Separate features into modules

## Resources

- [Android Gradle Plugin docs](https://developer.android.com/studio/build)
- [Gradle Kotlin DSL primer](https://docs.gradle.org/current/userguide/kotlin_dsl.html)
- [Version catalogs](https://developer.android.com/build/migrate-to-catalogs)
- [Build performance](https://developer.android.com/studio/build/optimize-your-build)
