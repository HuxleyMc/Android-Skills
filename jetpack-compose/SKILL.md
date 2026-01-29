---
name: jetpack-compose
description: Guides implementation of Android UI with Jetpack Compose. Use when building Android UIs, creating composable functions, managing state in Compose, implementing navigation, or theming apps. Covers Compose fundamentals, state management, layouts, and Material Design 3.
tags: ["android", "compose", "ui", "kotlin", "material-design", "state"]
difficulty: intermediate
category: ui
version: "1.0.0"
last_updated: "2025-01-29"
---

# Jetpack Compose

## Quick Start

Add dependencies to `build.gradle`:

```kotlin
dependencies {
    implementation("androidx.compose.ui:ui:1.5.4")
    implementation("androidx.compose.material3:material3:1.1.2")
    implementation("androidx.compose.ui:ui-tooling-preview:1.5.4")
    implementation("androidx.activity:activity-compose:1.8.0")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.6.2")
    debugImplementation("androidx.compose.ui:ui-tooling:1.5.4")
}
```

Enable Compose:

```kotlin
android {
    buildFeatures {
        compose = true
    }
    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.8"
    }
}
```

Basic Activity:

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyAppTheme {
                Surface {
                    Greeting("Android")
                }
            }
        }
    }
}

@Composable
fun Greeting(name: String) {
    Text(text = "Hello $name!")
}
```

## Core Patterns

### State Management

**Remember vs Remember Saveable:**
```kotlin
@Composable
fun Counter() {
    // Survives recomposition, lost on config change
    var count by remember { mutableIntStateOf(0) }
    
    // Survives config change (process death)
    var text by rememberSaveable { mutableStateOf("") }
    
    Button(onClick = { count++ }) {
        Text("Count: $count")
    }
}
```

**ViewModel + Compose:**
```kotlin
@Composable
fun UserScreen(viewModel: UserViewModel = viewModel()) {
    val uiState by viewModel.uiState.collectAsState()
    
    when {
        uiState.isLoading -> CircularProgressIndicator()
        uiState.error != null -> ErrorMessage(uiState.error)
        uiState.user != null -> UserProfile(uiState.user)
    }
}
```

**State Hoisting:**
```kotlin
// Stateless - reusable
@Composable
fun SearchBar(
    query: String,
    onQueryChange: (String) -> Unit,
    onSearch: () -> Unit
) {
    TextField(
        value = query,
        onValueChange = onQueryChange,
        trailingIcon = {
            IconButton(onClick = onSearch) {
                Icon(Icons.Default.Search, "Search")
            }
        }
    )
}

// State holder
@Composable
fun SearchScreen(viewModel: SearchViewModel = viewModel()) {
    val query by viewModel.query.collectAsState()
    
    SearchBar(
        query = query,
        onQueryChange = viewModel::setQuery,
        onSearch = viewModel::search
    )
}
```

### Layouts

**Column/Row/Box:**
```kotlin
Column(
    modifier = Modifier
        .fillMaxSize()
        .padding(16.dp),
    horizontalAlignment = Alignment.CenterHorizontally,
    verticalArrangement = Arrangement.spacedBy(8.dp)
) {
    Text("Title", style = MaterialTheme.typography.headlineMedium)
    Text("Subtitle", style = MaterialTheme.typography.bodyMedium)
    Button(onClick = { }) { Text("Action") }
}
```

**Lazy Lists:**
```kotlin
LazyColumn(
    modifier = Modifier.fillMaxSize(),
    contentPadding = PaddingValues(16.dp),
    verticalArrangement = Arrangement.spacedBy(8.dp)
) {
    items(items = products, key = { it.id }) { product ->
        ProductCard(product)
    }
    
    item {
        if (isLoading) CircularProgressIndicator()
    }
}
```

**ConstraintLayout (complex layouts):**
```kotlin
ConstraintLayout(modifier = Modifier.fillMaxWidth()) {
    val (avatar, name, date) = createRefs()
    
    Image(
        painter = painterResource(R.drawable.avatar),
        contentDescription = null,
        modifier = Modifier.constrainAs(avatar) {
            start.linkTo(parent.start)
            top.linkTo(parent.top)
        }
    )
    
    Text(
        text = userName,
        modifier = Modifier.constrainAs(name) {
            start.linkTo(avatar.end, margin = 8.dp)
            top.linkTo(avatar.top)
        }
    )
}
```

### Side Effects

**LaunchedEffect:**
```kotlin
@Composable
fun UserProfile(userId: String, viewModel: UserViewModel = viewModel()) {
    // Runs when userId changes
    LaunchedEffect(userId) {
        viewModel.loadUser(userId)
    }
    
    // UI content
}
```

**DisposableEffect (cleanup):**
```kotlin
@Composable
fun SensorListener() {
    val context = LocalContext.current
    
    DisposableEffect(context) {
        val sensorManager = context.getSystemService(Context.SENSOR_SERVICE) as SensorManager
        val listener = object : SensorEventListener {
            override fun onSensorChanged(event: SensorEvent) { }
            override fun onAccuracyChanged(sensor: Sensor, accuracy: Int) { }
        }
        
        sensorManager.registerListener(listener, sensor, SensorManager.SENSOR_DELAY_NORMAL)
        
        onDispose {
            sensorManager.unregisterListener(listener)
        }
    }
}
```

**derivedStateOf (expensive calculations):**
```kotlin
@Composable
fun FilteredList(items: List<Item>, query: String) {
    val filtered by remember(query, items) {
        derivedStateOf {
            items.filter { it.name.contains(query, ignoreCase = true) }
        }
    }
    
    LazyColumn {
        items(filtered) { item ->
            ListItem(item)
        }
    }
}
```

### Theming

**Material 3 Theme:**
```kotlin
@Composable
fun MyAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }
    
    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}

// Custom colors
private val LightColorScheme = lightColorScheme(
    primary = Purple40,
    secondary = PurpleGrey40,
    tertiary = Pink40
)
```

**Custom Composables with Theme:**
```kotlin
@Composable
fun PrimaryButton(
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    content: @Composable RowScope.() -> Unit
) {
    Button(
        onClick = onClick,
        modifier = modifier,
        colors = ButtonDefaults.buttonColors(
            containerColor = MaterialTheme.colorScheme.primary
        ),
        content = content
    )
}
```

## Common Patterns

### Navigation

```kotlin
@Composable
fun AppNavigation() {
    val navController = rememberNavController()
    
    NavHost(navController = navController, startDestination = "home") {
        composable("home") {
            HomeScreen(
                onNavigateToDetail = { id ->
                    navController.navigate("detail/$id")
                }
            )
        }
        composable(
            "detail/{id}",
            arguments = listOf(navArgument("id") { type = NavType.StringType })
        ) { backStackEntry ->
            val id = backStackEntry.arguments?.getString("id")
            DetailScreen(id = id)
        }
    }
}
```

### Animation

**AnimatedVisibility:**
```kotlin
var visible by remember { mutableStateOf(true) }

AnimatedVisibility(
    visible = visible,
    enter = fadeIn() + slideInVertically(),
    exit = fadeOut() + slideOutVertically()
) {
    Card { Text("Content") }
}
```

**Animate As State:**
```kotlin
var expanded by remember { mutableStateOf(false) }
val size by animateDpAsState(
    targetValue = if (expanded) 200.dp else 100.dp,
    animationSpec = tween(durationMillis = 300)
)

Box(
    modifier = Modifier.size(size),
    contentAlignment = Alignment.Center
) {
    Button(onClick = { expanded = !expanded }) {
        Text(if (expanded) "Shrink" else "Expand")
    }
}
```

### Forms

```kotlin
@Composable
fun LoginForm(viewModel: LoginViewModel = viewModel()) {
    var email by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }
    var isLoading by remember { mutableStateOf(false) }
    var error by remember { mutableStateOf<String?>(null) }
    
    Column(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        OutlinedTextField(
            value = email,
            onValueChange = { email = it },
            label = { Text("Email") },
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Email),
            singleLine = true,
            modifier = Modifier.fillMaxWidth()
        )
        
        OutlinedTextField(
            value = password,
            onValueChange = { password = it },
            label = { Text("Password") },
            visualTransformation = PasswordVisualTransformation(),
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Password),
            singleLine = true,
            modifier = Modifier.fillMaxWidth()
        )
        
        error?.let {
            Text(it, color = MaterialTheme.colorScheme.error)
        }
        
        Button(
            onClick = {
                isLoading = true
                viewModel.login(email, password)
            },
            enabled = !isLoading,
            modifier = Modifier.fillMaxWidth()
        ) {
            if (isLoading) {
                CircularProgressIndicator(
                    modifier = Modifier.size(20.dp),
                    color = MaterialTheme.colorScheme.onPrimary
                )
            } else {
                Text("Login")
            }
        }
    }
}
```

### Preview

```kotlin
@Preview(showBackground = true)
@Composable
fun GreetingPreview() {
    MyAppTheme {
        Greeting("Android")
    }
}

@Preview(showBackground = true, uiMode = Configuration.UI_MODE_NIGHT_YES)
@Composable
fun GreetingPreviewDark() {
    MyAppTheme {
        Greeting("Android")
    }
}
```

## Examples (Input → Output)

### Complete Screen

**Input request:** "Create a product list screen with search and loading state"

**Output:**
```kotlin
@Composable
fun ProductListScreen(viewModel: ProductViewModel = viewModel()) {
    val uiState by viewModel.uiState.collectAsState()
    var searchQuery by remember { mutableStateOf("") }
    
    Scaffold(
        topBar = {
            SearchBar(
                query = searchQuery,
                onQueryChange = { 
                    searchQuery = it
                    viewModel.search(it)
                },
                modifier = Modifier.padding(16.dp)
            )
        }
    ) { padding ->
        Box(
            modifier = Modifier
                .fillMaxSize()
                .padding(padding)
        ) {
            when {
                uiState.isLoading -> CircularProgressIndicator(
                    modifier = Modifier.align(Alignment.Center)
                )
                uiState.error != null -> ErrorView(
                    message = uiState.error,
                    onRetry = viewModel::retry
                )
                uiState.products.isEmpty() -> EmptyView()
                else -> ProductList(
                    products = uiState.products,
                    onProductClick = { /* navigation */ }
                )
            }
        }
    }
}

@Composable
fun ProductList(
    products: List<Product>,
    onProductClick: (Product) -> Unit
) {
    LazyColumn(
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(products, key = { it.id }) { product ->
            ProductCard(
                product = product,
                onClick = { onProductClick(product) }
            )
        }
    }
}

@Composable
fun ProductCard(product: Product, onClick: () -> Unit) {
    Card(
        onClick = onClick,
        modifier = Modifier.fillMaxWidth()
    ) {
        Row(
            modifier = Modifier.padding(16.dp),
            horizontalArrangement = Arrangement.spacedBy(16.dp)
        ) {
            AsyncImage(
                model = product.imageUrl,
                contentDescription = null,
                modifier = Modifier.size(80.dp)
            )
            Column {
                Text(
                    text = product.name,
                    style = MaterialTheme.typography.titleMedium
                )
                Text(
                    text = "$${product.price}",
                    style = MaterialTheme.typography.bodyMedium,
                    color = MaterialTheme.colorScheme.primary
                )
            }
        }
    }
}
```

### Converting XML to Compose

**Input request:** "Convert this RecyclerView item layout to Compose"

**Given XML:**
```xml
<androidx.cardview.widget.CardView>
    <LinearLayout android:orientation="horizontal">
        <ImageView android:id="@+id/avatar" />
        <LinearLayout android:orientation="vertical">
            <TextView android:id="@+id/name" android:textStyle="bold" />
            <TextView android:id="@+id/email" />
        </LinearLayout>
    </LinearLayout>
</androidx.cardview.widget.CardView>
```

**Output:**
```kotlin
@Composable
fun UserListItem(user: User, onClick: () -> Unit) {
    Card(
        onClick = onClick,
        modifier = Modifier.fillMaxWidth()
    ) {
        Row(
            modifier = Modifier.padding(16.dp),
            horizontalArrangement = Arrangement.spacedBy(16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Image(
                painter = painterResource(R.drawable.avatar),
                contentDescription = null,
                modifier = Modifier.size(48.dp)
            )
            Column {
                Text(
                    text = user.name,
                    style = MaterialTheme.typography.titleMedium,
                    fontWeight = FontWeight.Bold
                )
                Text(
                    text = user.email,
                    style = MaterialTheme.typography.bodyMedium
                )
            }
        }
    }
}
```

## Best Practices

1. **State hoisting**: Keep composables stateless, hoist state to parent or ViewModel
2. **Remember keys**: Use `remember(key)` or `rememberSaveable` appropriately
3. **Use key in lazy lists**: `items(list, key = { it.id })` for proper recomposition
4. **Avoid business logic in composables**: Delegate to ViewModel
5. **Use MaterialTheme**: Don't hardcode colors, reference theme
6. **Preview everything**: Add @Preview for all UI components
7. **Modifier chaining**: Order matters (size → padding → click → then effects)
8. **CollectAsStateWithLifecycle**: Use `collectAsStateWithLifecycle()` instead of `collectAsState()` for lifecycle awareness
9. **Recomposition optimization**: Use `derivedStateOf` for expensive calculations
10. **Immutable data classes**: Use immutable state for better recomposition tracking

## Resources

- [Compose documentation](https://developer.android.com/jetpack/compose)
- [Material Design 3 in Compose](https://developer.android.com/jetpack/compose/designsystems/material3)
- [Compose samples](https://github.com/android/compose-samples)
- [Accompanist libraries](https://google.github.io/accompanist/)
