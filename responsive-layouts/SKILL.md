---
name: responsive-layouts
description: Guides responsive UI development for phones, tablets, foldables, and desktop. Use when supporting multiple screen sizes, implementing adaptive layouts, handling foldable devices, or optimizing for large screens. Covers window size classes, multi-pane layouts, foldable support, and input adaptations.
tags: ["android", "responsive", "adaptive", "tablet", "foldable", "desktop", "compose", "large-screens"]
difficulty: intermediate
category: ui
version: "1.0.0"
last_updated: "2025-01-29"
---

# Responsive Layouts

## Quick Start

Add dependencies:

```kotlin
dependencies {
    // Window size classes
    implementation("androidx.compose.material3:material3-window-size-class:1.2.0")
    
    // Foldable support
    implementation("androidx.window:window:1.2.0")
    
    // Navigation for large screens
    implementation("androidx.navigation:navigation-compose:2.7.6")
}
```

Device categories:

| Device | Width | Height | Layout Strategy |
|--------|-------|--------|-----------------|
| Phone (compact) | <600dp | <480dp | Single pane, bottom nav |
| Phone (medium) | 600-840dp | 480-900dp | Single/dual pane adaptive |
| Tablet (expanded) | 840-1200dp | 900dp+ | Dual pane, side nav |
| Desktop/large tablet | >1200dp | >900dp | Multi-pane, permanent nav |
| Foldable (folded) | Similar to phone | - | Single pane |
| Foldable (unfolded) | Similar to tablet | - | Dual pane |

## Core Patterns

### Window Size Classes

**Calculate size class:**

```kotlin
@Composable
fun MyApp() {
    val windowSizeClass = calculateWindowSizeClass()
    
    // Use size class for adaptive behavior
    ResponsiveLayout(windowSizeClass = windowSizeClass)
}

@Composable
fun ResponsiveLayout(windowSizeClass: WindowSizeClass) {
    // Compact = phone portrait
    // Medium = phone landscape / small tablet
    // Expanded = tablet / desktop
    val widthClass = windowSizeClass.widthSizeClass
    val heightClass = windowSizeClass.heightSizeClass
    
    when (widthClass) {
        WindowWidthSizeClass.Compact -> PhoneLayout()
        WindowWidthSizeClass.Medium -> TabletLayout()
        WindowWidthSizeClass.Expanded -> DesktopLayout()
    }
}
```

**Adaptive content:**

```kotlin
@Composable
fun ListDetailScreen(
    items: List<Item>,
    selectedItem: Item?,
    onItemSelected: (Item) -> Unit
) {
    val windowSizeClass = calculateWindowSizeClass()
    
    when (windowSizeClass.widthSizeClass) {
        WindowWidthSizeClass.Compact -> {
            // Phone: Single pane with navigation
            if (selectedItem == null) {
                ItemListScreen(items, onItemSelected)
            } else {
                ItemDetailScreen(selectedItem)
            }
        }
        else -> {
            // Tablet/Desktop: Dual pane
            Row {
                ItemListPane(
                    items = items,
                    selectedItem = selectedItem,
                    onItemSelected = onItemSelected,
                    modifier = Modifier.weight(1f)
                )
                ItemDetailPane(
                    item = selectedItem,
                    modifier = Modifier.weight(2f)
                )
            }
        }
    }
}
```

### Multi-Pane Layouts

**List-detail layout:**

```kotlin
@Composable
fun ListDetailLayout(
    items: List<Email>,
    selectedId: String?,
    onSelect: (String) -> Unit,
    onBack: () -> Unit
) {
    val windowSizeClass = calculateWindowSizeClass()
    val isExpanded = windowSizeClass.widthSizeClass != WindowWidthSizeClass.Compact
    
    if (isExpanded) {
        // Dual pane for large screens
        Row(modifier = Modifier.fillMaxSize()) {
            EmailList(
                emails = items,
                selectedId = selectedId,
                onSelect = onSelect,
                modifier = Modifier.width(360.dp)
            )
            
            VerticalDivider()
            
            val selectedEmail = items.find { it.id == selectedId }
            if (selectedEmail != null) {
                EmailDetail(
                    email = selectedEmail,
                    modifier = Modifier.weight(1f)
                )
            } else {
                EmptyDetailPane(modifier = Modifier.weight(1f))
            }
        }
    } else {
        // Single pane for phones
        val selectedEmail = items.find { it.id == selectedId }
        
        if (selectedEmail == null) {
            EmailList(
                emails = items,
                selectedId = selectedId,
                onSelect = onSelect
            )
        } else {
            EmailDetail(
                email = selectedEmail,
                onBack = onBack
            )
        }
    }
}
```

**Feed with supporting pane:**

```kotlin
@Composable
fun FeedLayout(posts: List<Post>) {
    val windowSizeClass = calculateWindowSizeClass()
    val showSupportingPane = windowSizeClass.widthSizeClass == WindowWidthSizeClass.Expanded
    
    Row(modifier = Modifier.fillMaxSize()) {
        // Main content
        PostFeed(
            posts = posts,
            modifier = if (showSupportingPane) Modifier.weight(2f) else Modifier.fillMaxSize()
        )
        
        // Supporting pane only on large screens
        if (showSupportingPane) {
            VerticalDivider()
            SupportingPane(
                modifier = Modifier.weight(1f)
            )
        }
    }
}
```

**Adaptive navigation:**

```kotlin
@Composable
fun AdaptiveNavigation(
    destinations: List<Destination>,
    selected: Destination,
    onSelect: (Destination) -> Unit
) {
    val windowSizeClass = calculateWindowSizeClass()
    
    when (windowSizeClass.widthSizeClass) {
        WindowWidthSizeClass.Compact -> {
            // Bottom nav for phones
            NavigationBar {
                destinations.forEach { destination ->
                    NavigationBarItem(
                        icon = { Icon(destination.icon, null) },
                        label = { Text(destination.label) },
                        selected = destination == selected,
                        onClick = { onSelect(destination) }
                    )
                }
            }
        }
        else -> {
            // Side rail for tablets
            NavigationRail {
                destinations.forEach { destination ->
                    NavigationRailItem(
                        icon = { Icon(destination.icon, null) },
                        label = { Text(destination.label) },
                        selected = destination == selected,
                        onClick = { onSelect(destination) }
                    )
                }
            }
        }
    }
}
```

### Foldable Support

**WindowInfoTracker:**

```kotlin
class FoldableHelper(context: Context) {
    private val windowInfoTracker = WindowInfoTracker.getOrCreate(context)
    
    @OptIn(ExperimentalMaterial3WindowSizeClassApi::class)
    fun getWindowLayoutInfo(activity: Activity): Flow<WindowLayoutInfo> {
        return windowInfoTracker.windowLayoutInfo(activity)
    }
}

@Composable
fun FoldableAwareLayout(activity: Activity) {
    val windowLayoutInfo by rememberUpdatedState(
        WindowInfoTracker.getOrCreate(LocalContext.current)
            .windowLayoutInfo(activity)
            .collectAsState(initial = null).value
    )
    
    val foldingFeature = windowLayoutInfo?.displayFeatures
        ?.filterIsInstance<FoldingFeature>()
        ?.firstOrNull()
    
    when {
        foldingFeature == null -> {
            // Regular device
            NormalLayout()
        }
        foldingFeature.state == FoldingFeature.State.FLAT &&
        foldingFeature.orientation == FoldingFeature.Orientation.HORIZONTAL -> {
            // Unfolded horizontal fold (tabletop mode)
            TableTopLayout(foldingFeature = foldingFeature)
        }
        foldingFeature.state == FoldingFeature.State.HALF_OPENED &&
        foldingFeature.orientation == FoldingFeature.Orientation.HORIZONTAL -> {
            // Half-open horizontal (book mode)
            BookModeLayout(foldingFeature = foldingFeature)
        }
        else -> {
            // Other fold states
            FoldableLayout(foldingFeature = foldingFeature)
        }
    }
}
```

**Avoid fold crease:**

```kotlin
@Composable
fun AvoidFoldCrease(
    foldingFeature: FoldingFeature,
    content: @Composable () -> Unit
) {
    val foldBounds = foldingFeature.bounds
    
    Box(
        modifier = Modifier
            .fillMaxSize()
            .padding(
                top = if (foldBounds.top > 0) foldBounds.height.dp else 0.dp,
                bottom = if (foldBounds.bottom < LocalConfiguration.current.screenHeightDp) 
                    foldBounds.height.dp else 0.dp
            )
    ) {
        content()
    }
}
```

**Dual screen with fold:**

```kotlin
@Composable
fun DualScreenReader(
    article: Article,
    foldingFeature: FoldingFeature
) {
    Row(modifier = Modifier.fillMaxSize()) {
        // Left pane: Article list
        Box(
            modifier = Modifier
                .weight(1f)
                .padding(end = if (foldingFeature.isSeparating) 0.dp else 16.dp)
        ) {
            ArticleList(article = article)
        }
        
        // Visual separator at fold
        if (foldingFeature.isSeparating) {
            VerticalDivider()
        }
        
        // Right pane: Reading content
        Box(
            modifier = Modifier.weight(1f)
        ) {
            ArticleContent(article = article)
        }
    }
}
```

### Adaptive Grids

**Responsive grid:**

```kotlin
@Composable
fun AdaptiveGrid(
    items: List<Product>,
    modifier: Modifier = Modifier
) {
    val windowSizeClass = calculateWindowSizeClass()
    
    val columns = when (windowSizeClass.widthSizeClass) {
        WindowWidthSizeClass.Compact -> 2
        WindowWidthSizeClass.Medium -> 3
        WindowWidthSizeClass.Expanded -> 4
    }
    
    LazyVerticalGrid(
        columns = GridCells.Fixed(columns),
        contentPadding = PaddingValues(16.dp),
        horizontalArrangement = Arrangement.spacedBy(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp),
        modifier = modifier
    ) {
        items(items, key = { it.id }) { product ->
            ProductCard(product = product)
        }
    }
}
```

**Adaptive staggered grid:**

```kotlin
@Composable
fun AdaptiveStaggeredGrid(
    photos: List<Photo>,
    modifier: Modifier = Modifier
) {
    val windowSizeClass = calculateWindowSizeClass()
    val configuration = LocalConfiguration.current
    
    // Consider both width and height
    val columns = when {
        windowSizeClass.widthSizeClass == WindowWidthSizeClass.Compact -> 2
        windowSizeClass.widthSizeClass == WindowWidthSizeClass.Medium -> 3
        windowSizeClass.widthSizeClass == WindowWidthSizeClass.Expanded -> 4
        else -> 2
    }
    
    LazyVerticalStaggeredGrid(
        columns = StaggeredGridCells.Fixed(columns),
        contentPadding = PaddingValues(16.dp),
        horizontalArrangement = Arrangement.spacedBy(16.dp),
        verticalItemSpacing = 16.dp,
        modifier = modifier
    ) {
        items(photos, key = { it.id }) { photo ->
            PhotoCard(photo = photo)
        }
    }
}
```

### Responsive Typography & Spacing

**Responsive text:**

```kotlin
@Composable
fun ResponsiveTitle(text: String) {
    val windowSizeClass = calculateWindowSizeClass()
    
    val style = when (windowSizeClass.widthSizeClass) {
        WindowWidthSizeClass.Compact -> MaterialTheme.typography.headlineSmall
        WindowWidthSizeClass.Medium -> MaterialTheme.typography.headlineMedium
        WindowWidthSizeClass.Expanded -> MaterialTheme.typography.headlineLarge
    }
    
    Text(
        text = text,
        style = style
    )
}
```

**Responsive padding:**

```kotlin
@Composable
fun ResponsivePadding(content: @Composable () -> Unit) {
    val windowSizeClass = calculateWindowSizeClass()
    
    val padding = when (windowSizeClass.widthSizeClass) {
        WindowWidthSizeClass.Compact -> 16.dp
        WindowWidthSizeClass.Medium -> 24.dp
        WindowWidthSizeClass.Expanded -> 32.dp
    }
    
    Box(modifier = Modifier.padding(padding)) {
        content()
    }
}
```

## Common Patterns

### Drag and Drop (Desktop/Multi-window)

```kotlin
@Composable
fun DraggableItem(
    item: Item,
    onDragStart: () -> Unit,
    onDragEnd: (DropTarget) -> Unit
) {
    var isDragging by remember { mutableStateOf(false) }
    
    Box(
        modifier = Modifier
            .pointerInput(item) {
                detectDragGestures(
                    onDragStart = { onDragStart() },
                    onDragEnd = { onDragEnd(calculateDropTarget()) }
                ) { change, dragAmount ->
                    change.consume()
                    // Update position
                }
            }
            .alpha(if (isDragging) 0.5f else 1f)
    ) {
        ItemContent(item)
    }
}

@Composable
fun DropTarget(
    onItemDropped: (Item) -> Unit,
    content: @Composable () -> Unit
) {
    var isHovered by remember { mutableStateOf(false) }
    
    Box(
        modifier = Modifier
            .border(
                width = if (isHovered) 2.dp else 0.dp,
                color = MaterialTheme.colorScheme.primary,
                shape = MaterialTheme.shapes.medium
            )
            .pointerInput(Unit) {
                // Handle drop events
            }
    ) {
        content()
    }
}
```

### Keyboard & Mouse Support

**Keyboard shortcuts:**

```kotlin
@Composable
fun KeyboardShortcutsHandler(
    onNewDocument: () -> Unit,
    onSave: () -> Unit,
    onSearch: () -> Unit
) {
    val focusManager = LocalFocusManager.current
    
    Box(
        modifier = Modifier
            .onPreviewKeyEvent { event ->
                if (event.type == KeyEventType.KeyDown) {
                    when {
                        event.isCtrlPressed && event.key == Key.N -> {
                            onNewDocument()
                            true
                        }
                        event.isCtrlPressed && event.key == Key.S -> {
                            onSave()
                            true
                        }
                        event.isCtrlPressed && event.key == Key.F -> {
                            onSearch()
                            true
                        }
                        else -> false
                    }
                } else {
                    false
                }
            }
            .focusable()
            .focusRequester(FocusRequester())
    )
}
```

**Right-click context menu:**

```kotlin
@Composable
fun ContextMenuItem(
    item: Item,
    onEdit: () -> Unit,
    onDelete: () -> Unit
) {
    var showMenu by remember { mutableStateOf(false) }
    
    Box(
        modifier = Modifier.pointerInput(Unit) {
            detectTapGestures(
                onLongPress = { showMenu = true },
                onSecondaryTap = { showMenu = true }  // Right-click
            )
        }
    ) {
        ItemContent(item)
        
        DropdownMenu(
            expanded = showMenu,
            onDismissRequest = { showMenu = false }
        ) {
            DropdownMenuItem(
                text = { Text("Edit") },
                onClick = {
                    onEdit()
                    showMenu = false
                }
            )
            DropdownMenuItem(
                text = { Text("Delete") },
                onClick = {
                    onDelete()
                    showMenu = false
                }
            )
        }
    }
}
```

**Hover effects:**

```kotlin
@Composable
fun HoverableCard(
    content: @Composable () -> Unit
) {
    var isHovered by remember { mutableStateOf(false) }
    
    val elevation by animateDpAsState(
        targetValue = if (isHovered) 8.dp else 1.dp
    )
    
    Card(
        elevation = CardDefaults.cardElevation(elevation),
        modifier = Modifier
            .pointerInput(Unit) {
                detectHoverEvents(
                    onEnter = { isHovered = true },
                    onExit = { isHovered = false }
                )
            }
    ) {
        content()
    }
}
```

### Configuration Changes

**Save state across resize:**

```kotlin
@Composable
fun ResponsiveActivity() {
    // Save scroll position across configuration changes
    val listState = rememberLazyListState()
    
    // Save in saved state handle for process death
    val savedStateHandle = rememberSaveable { mutableStateOf(0) }
    
    val windowSizeClass = calculateWindowSizeClass()
    
    // Remember previous size class to detect changes
    var previousSizeClass by remember { mutableStateOf(windowSizeClass.widthSizeClass) }
    
    LaunchedEffect(windowSizeClass) {
        if (previousSizeClass != windowSizeClass.widthSizeClass) {
            // Handle transition (e.g., restore scroll position)
            previousSizeClass = windowSizeClass.widthSizeClass
        }
    }
    
    ResponsiveContent(
        listState = listState,
        windowSizeClass = windowSizeClass
    )
}
```

## Examples (Input → Output)

### Complete Adaptive App

**Input request:** "Create an email app that adapts from phone to tablet to desktop"

**Output:**

```kotlin
@Composable
fun EmailApp() {
    val windowSizeClass = calculateWindowSizeClass()
    val navController = rememberNavController()
    
    // Track selected email for dual-pane layout
    var selectedEmailId by rememberSaveable { mutableStateOf<String?>(null) }
    
    Scaffold(
        topBar = {
            if (windowSizeClass.widthSizeClass == WindowWidthSizeClass.Compact) {
                CenterAlignedTopAppBar(
                    title = { Text("Email") }
                )
            }
        },
        bottomBar = {
            if (windowSizeClass.widthSizeClass == WindowWidthSizeClass.Compact) {
                BottomNavBar(navController)
            }
        }
    ) { padding ->
        Row(
            modifier = Modifier
                .fillMaxSize()
                .padding(padding)
        ) {
            // Side navigation for larger screens
            if (windowSizeClass.widthSizeClass != WindowWidthSizeClass.Compact) {
                PermanentNavigationDrawer(
                    navController = navController,
                    modifier = Modifier.width(280.dp)
                )
                VerticalDivider()
            }
            
            // Main content area
            when (windowSizeClass.widthSizeClass) {
                WindowWidthSizeClass.Compact -> {
                    // Phone: Single pane navigation
                    NavHost(navController, startDestination = "inbox") {
                        composable("inbox") {
                            EmailListScreen(
                                onEmailClick = { emailId ->
                                    navController.navigate("detail/$emailId")
                                }
                            )
                        }
                        composable("detail/{emailId}") { backStack ->
                            val emailId = backStack.arguments?.getString("emailId")
                            EmailDetailScreen(
                                emailId = emailId,
                                onBack = { navController.popBackStack() }
                            )
                        }
                    }
                }
                else -> {
                    // Tablet/Desktop: Dual pane
                    Row {
                        // List pane
                        Box(modifier = Modifier.weight(1f)) {
                            EmailListScreen(
                                selectedId = selectedEmailId,
                                onEmailClick = { emailId ->
                                    selectedEmailId = emailId
                                }
                            )
                        }
                        
                        VerticalDivider()
                        
                        // Detail pane
                        Box(modifier = Modifier.weight(2f)) {
                            if (selectedEmailId != null) {
                                EmailDetailScreen(emailId = selectedEmailId)
                            } else {
                                EmptyDetailPlaceholder()
                            }
                        }
                    }
                }
            }
        }
    }
}

@Composable
fun EmailListScreen(
    selectedId: String? = null,
    onEmailClick: (String) -> Unit
) {
    val emails = remember { getSampleEmails() }
    
    LazyColumn(
        contentPadding = PaddingValues(vertical = 8.dp)
    ) {
        items(emails, key = { it.id }) { email ->
            EmailListItem(
                email = email,
                isSelected = email.id == selectedId,
                onClick = { onEmailClick(email.id) }
            )
        }
    }
}
```

### Foldable Optimized Reader

**Input request:** "Create a reading app optimized for foldable devices in tabletop mode"

**Output:**

```kotlin
@Composable
fun BookReader(
    book: Book,
    currentPage: Int,
    onPageChange: (Int) -> Unit
) {
    val activity = LocalContext.current as Activity
    val windowLayoutInfo = WindowInfoTracker.getOrCreate(activity)
        .windowLayoutInfo(activity)
        .collectAsState(initial = null).value
    
    val foldingFeature = windowLayoutInfo?.displayFeatures
        ?.filterIsInstance<FoldingFeature>()
        ?.firstOrNull()
    
    val isTableTopMode = foldingFeature?.let {
        it.state == FoldingFeature.State.HALF_OPENED &&
        it.orientation == FoldingFeature.Orientation.HORIZONTAL
    } ?: false
    
    if (isTableTopMode && foldingFeature != null) {
        // Tabletop mode: Controls on bottom half, content on top
        TableTopLayout(
            book = book,
            currentPage = currentPage,
            onPageChange = onPageChange,
            foldBounds = foldingFeature.bounds
        )
    } else {
        // Normal mode
        StandardReader(
            book = book,
            currentPage = currentPage,
            onPageChange = onPageChange
        )
    }
}

@Composable
fun TableTopLayout(
    book: Book,
    currentPage: Int,
    onPageChange: (Int) -> Unit,
    foldBounds: Rect
) {
    val density = LocalDensity.current
    val foldHeight = with(density) { foldBounds.height.toDp() }
    
    Column(modifier = Modifier.fillMaxSize()) {
        // Top half: Reading content
        Box(
            modifier = Modifier
                .weight(1f)
                .padding(bottom = foldHeight / 2)
        ) {
            PageContent(
                page = book.pages[currentPage],
                modifier = Modifier.fillMaxSize()
            )
        }
        
        // Bottom half: Controls
        Box(
            modifier = Modifier
                .weight(1f)
                .padding(top = foldHeight / 2),
            contentAlignment = Alignment.Center
        ) {
            ReaderControls(
                currentPage = currentPage,
                totalPages = book.pages.size,
                onPrevious = { onPageChange(currentPage - 1) },
                onNext = { onPageChange(currentPage + 1) },
                onPageSelect = onPageChange
            )
        }
    }
}
```

## Best Practices

1. **Use window size classes**: Don't hardcode breakpoints, use standard size classes
2. **Test on real devices**: Emulators don't capture all foldable behaviors
3. **Support all orientations**: Tablets are often used in landscape
4. **Minimum touch targets**: 48dp regardless of screen size
5. **Content first**: Don't just stretch content, adapt layout
6. **Save scroll position**: Restore state when switching layouts
7. **Keyboard navigation**: Support Tab navigation for desktop
8. **Right-click menus**: Add context menus for mouse users
9. **Responsive text**: Scale typography appropriately
10. **Handle configuration changes**: Resize triggers config change

## Resources

- [Large screens guidelines](https://developer.android.com/guide/topics/large-screens)
- [Window size classes](https://developer.android.com/reference/kotlin/androidx/compose/material3/windowsizeclass/package-summary)
- [Foldable devices](https://developer.android.com/guide/topics/large-screens/learn-about-foldables)
- [Jetpack WindowManager](https://developer.android.com/jetpack/androidx/releases/window)
- [Responsive UI samples](https://github.com/android/compose-samples)
