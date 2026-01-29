---
name: jetpack-compose-animations
description: Guides Jetpack Compose animations for Android UI. Use when adding motion to UI, creating transitions between screens, animating state changes, or building interactive gestures. Covers AnimatedVisibility, animate*AsState, transitions, and gesture animations.
tags: ["android", "compose", "animation", "ui", "motion", "gestures", "transitions"]
difficulty: intermediate
category: ui
version: "1.0.0"
last_updated: "2025-01-29"
---

# Jetpack Compose Animations

## Quick Start

Add animation dependencies:

```kotlin
dependencies {
    implementation("androidx.compose.animation:animation:1.5.4")
    implementation("androidx.compose.ui:ui:1.5.4")
}
```

Basic animation types:

| Animation | Use For | API |
|-----------|---------|-----|
| Visibility | Show/hide content | `AnimatedVisibility` |
| Single value | Animate size, color, position | `animate*AsState` |
| Content change | Switch between composables | `Crossfade`, `AnimatedContent` |
| Gesture driven | Swipe, drag animations | `Modifier.draggable`, `animateDecay` |
| Complex transitions | Multi-property animations | `updateTransition` |
| Layout changes | List reordering | `LazyList` + `animateItemPlacement` |

## Core Patterns

### Visibility Animations

**AnimatedVisibility:**
```kotlin
var visible by remember { mutableStateOf(true) }

AnimatedVisibility(
    visible = visible,
    enter = fadeIn() + expandVertically(),
    exit = fadeOut() + shrinkVertically()
) {
    Card {
        Text("Content that animates in and out")
    }
}
```

**Custom enter/exit:**
```kotlin
AnimatedVisibility(
    visible = visible,
    enter = slideInHorizontally(
        initialOffsetX = { -it },  // Slide from left
        animationSpec = tween(300)
    ) + fadeIn(),
    exit = slideOutHorizontally(
        targetOffsetX = { it },  // Slide to right
        animationSpec = spring(stiffness = Spring.StiffnessMedium)
    ) + fadeOut()
) {
    Content()
}
```

**Animate content size:**
```kotlin
Column {
    Button(onClick = { expanded = !expanded }) {
        Text(if (expanded) "Show Less" else "Show More")
    }
    
    AnimatedVisibility(
        visible = expanded,
        enter = expandVertically(),
        exit = shrinkVertically()
    ) {
        Column {
            Text("Additional content line 1")
            Text("Additional content line 2")
            Text("Additional content line 3")
        }
    }
}
```

### Value Animations

**animateDpAsState (size, position):**
```kotlin
var expanded by remember { mutableStateOf(false) }
val size by animateDpAsState(
    targetValue = if (expanded) 200.dp else 100.dp,
    animationSpec = spring(
        dampingRatio = Spring.DampingRatioMediumBouncy,
        stiffness = Spring.StiffnessLow
    )
)

Box(
    modifier = Modifier.size(size),
    contentAlignment = Alignment.Center
) {
    Button(onClick = { expanded = !expanded }) {
        Text("Toggle")
    }
}
```

**animateColorAsState:**
```kotlin
val isSelected by viewModel.isSelected.collectAsState()
val backgroundColor by animateColorAsState(
    targetValue = if (isSelected) 
        MaterialTheme.colorScheme.primary 
    else 
        MaterialTheme.colorScheme.surface,
    animationSpec = tween(durationMillis = 300)
)

Card(
    modifier = Modifier.clickable { viewModel.toggle() },
    colors = CardDefaults.cardColors(
        containerColor = backgroundColor
    )
) {
    Text("Selectable Card")
}
```

**animateFloatAsState (rotation, alpha):**
```kotlin
var isRotated by remember { mutableStateOf(false) }
val rotation by animateFloatAsState(
    targetValue = if (isRotated) 360f else 0f,
    animationSpec = tween(1000, easing = FastOutSlowInEasing)
)

IconButton(onClick = { isRotated = !isRotated }) {
    Icon(
        imageVector = Icons.Default.Refresh,
        contentDescription = "Rotate",
        modifier = Modifier.rotate(rotation)
    )
}
```

**animateOffsetAsState (position):**
```kotlin
var moved by remember { mutableStateOf(false) }
val offset by animateOffsetAsState(
    targetValue = if (moved) Offset(100f, 100f) else Offset.Zero,
    animationSpec = spring(stiffness = Spring.StiffnessHigh)
)

Box(
    modifier = Modifier
        .offset { IntOffset(offset.x.toInt(), offset.y.toInt()) }
        .size(50.dp)
        .background(Color.Blue)
        .clickable { moved = !moved }
)
```

### Content Animations

**Crossfade:**
```kotlin
var currentPage by remember { mutableStateOf("home") }

Crossfade(
    targetState = currentPage,
    animationSpec = tween(500)
) { page ->
    when (page) {
        "home" -> HomeScreen()
        "profile" -> ProfileScreen()
        "settings" -> SettingsScreen()
    }
}
```

**AnimatedContent:**
```kotlin
var count by remember { mutableStateOf(0) }

AnimatedContent(
    targetState = count,
    transitionSpec = {
        // Slide up for increase, down for decrease
        if (targetState > initialState) {
            slideInVertically { height -> height } + fadeIn() with
            slideOutVertically { height -> -height } + fadeOut()
        } else {
            slideInVertically { height -> -height } + fadeIn() with
            slideOutVertically { height -> height } + fadeOut()
        }
    }
) { targetCount ->
    Text(
        text = "$targetCount",
        style = MaterialTheme.typography.headlineLarge
    )
}
```

### Multi-Property Transitions

**updateTransition:**
```kotlin
enum class BoxState { Collapsed, Expanded }

var boxState by remember { mutableStateOf(BoxState.Collapsed) }
val transition = updateTransition(boxState, label = "box transition")

// Animate multiple properties together
val size by transition.animateDp(label = "size") { state ->
    when (state) {
        BoxState.Collapsed -> 64.dp
        BoxState.Expanded -> 200.dp
    }
}

val color by transition.animateColor(label = "color") { state ->
    when (state) {
        BoxState.Collapsed -> MaterialTheme.colorScheme.primary
        BoxState.Expanded -> MaterialTheme.colorScheme.secondary
    }
}

val cornerRadius by transition.animateDp(label = "corner") { state ->
    when (state) {
        BoxState.Collapsed -> 4.dp
        BoxState.Expanded -> 24.dp
    }
}

Box(
    modifier = Modifier
        .size(size)
        .background(color, RoundedCornerShape(cornerRadius))
        .clickable { 
            boxState = if (boxState == BoxState.Collapsed) 
                BoxState.Expanded else BoxState.Collapsed 
        }
)
```

**Transition with spec:**
```kotlin
val transition = updateTransition(targetState, label = "transition")

val alpha by transition.animateFloat(
    transitionSpec = {
        if (targetState == State.Visible) {
            tween(300)
        } else {
            tween(100)
        }
    },
    label = "alpha"
) { state ->
    if (state == State.Visible) 1f else 0f
}
```

### Gesture Animations

**Draggable:**
```kotlin
val maxWidth = 300.dp
val minWidth = 50.dp
var currentWidth by remember { mutableStateOf(minWidth) }

val density = LocalDensity.current
val minWidthPx = with(density) { minWidth.toPx() }
val maxWidthPx = with(density) { maxWidth.toPx() }

val draggableState = rememberDraggableState { delta ->
    val newWidth = currentWidth + with(density) { delta.toDp() }
    currentWidth = newWidth.coerceIn(minWidth, maxWidth)
}

Box(
    modifier = Modifier
        .width(currentWidth)
        .height(50.dp)
        .background(Color.Blue)
        .draggable(
            state = draggableState,
            orientation = Orientation.Horizontal,
            onDragStopped = { velocity ->
                // Animate to nearest bound
                val targetWidth = if (currentWidth < (minWidth + maxWidth) / 2) 
                    minWidth else maxWidth
                currentWidth = targetWidth
            }
        )
)
```

**Swipeable (swipe to dismiss):**
```kotlin
@OptIn(ExperimentalMaterialApi::class)
@Composable
fun SwipeableCard(
    onDismiss: () -> Unit,
    content: @Composable () -> Unit
) {
    val width = 300.dp
    val swipeWidth = with(LocalDensity.current) { width.toPx() }
    
    val swipeableState = rememberSwipeableState(0)
    val anchors = mapOf(0f to 0, swipeWidth to 1)
    
    Box(
        modifier = Modifier
            .swipeable(
                state = swipeableState,
                anchors = anchors,
                thresholds = { _, _ -> FractionalThreshold(0.3f) },
                orientation = Orientation.Horizontal
            )
    ) {
        Box(
            modifier = Modifier
                .offset { IntOffset(swipeableState.offset.value.toInt(), 0) }
        ) {
            content()
        }
    }
    
    // Trigger dismiss when swiped far enough
    LaunchedEffect(swipeableState.currentValue) {
        if (swipeableState.currentValue == 1) {
            onDismiss()
        }
    }
}
```

**Animating scroll position:**
```kotlin
val listState = rememberLazyListState()
val scope = rememberCoroutineScope()

// Smooth scroll to item
scope.launch {
    listState.animateScrollToItem(index = 10)
}

// Animate scroll by offset
scope.launch {
    listState.animateScrollBy(value = 500f)
}
```

### Layout Animations

**Lazy list animations:**
```kotlin
LazyColumn {
    items(
        items = items,
        key = { it.id }  // Important for proper animations
    ) { item ->
        Row(
            modifier = Modifier
                .animateItemPlacement(
                    animationSpec = spring(
                        stiffness = Spring.StiffnessMediumLow,
                        dampingRatio = Spring.DampingRatioMediumBouncy
                    )
                )
        ) {
            ListItem(item)
        }
    }
}
```

**Shared element transition (navigation):**
```kotlin
// Using Navigation-Compose with shared elements
@OptIn(ExperimentalSharedTransitionApi::class)
@Composable
fun SharedElementExample() {
    var selectedItem by remember { mutableStateOf<Item?>(null) }
    
    SharedTransitionLayout {
        AnimatedContent(
            targetState = selectedItem,
            label = "shared"
        ) { item ->
            if (item == null) {
                // List screen
                ItemList(
                    onItemClick = { selectedItem = it },
                    animatedVisibilityScope = this@AnimatedContent
                )
            } else {
                // Detail screen
                ItemDetail(
                    item = item,
                    onBack = { selectedItem = null },
                    animatedVisibilityScope = this@AnimatedContent
                )
            }
        }
    }
}

@OptIn(ExperimentalSharedTransitionApi::class)
@Composable
fun ItemList(
    onItemClick: (Item) -> Unit,
    animatedVisibilityScope: AnimatedVisibilityScope
) {
    LazyColumn {
        items(items) { item ->
            Row(
                modifier = Modifier
                    .clickable { onItemClick(item) }
            ) {
                Image(
                    painter = painterResource(item.image),
                    contentDescription = null,
                    modifier = Modifier
                        .sharedElement(
                            state = rememberSharedContentState(key = "image-${item.id}"),
                            animatedVisibilityScope = animatedVisibilityScope
                        )
                        .size(80.dp)
                )
            }
        }
    }
}
```

### Repeating/Infinite Animations

**Infinite pulse:**
```kotlin
val infiniteTransition = rememberInfiniteTransition(label = "infinite")
val scale by infiniteTransition.animateFloat(
    initialValue = 1f,
    targetValue = 1.2f,
    animationSpec = infiniteRepeatable(
        animation = tween(1000, easing = FastOutSlowInEasing),
        repeatMode = RepeatMode.Reverse
    ),
    label = "scale"
)

Box(
    modifier = Modifier
        .scale(scale)
        .size(50.dp)
        .background(Color.Red, CircleShape)
)
```

**Loading shimmer:**
```kotlin
@Composable
fun ShimmerLoading() {
    val shimmerColors = listOf(
        Color.LightGray.copy(alpha = 0.6f),
        Color.LightGray.copy(alpha = 0.2f),
        Color.LightGray.copy(alpha = 0.6f)
    )
    
    val transition = rememberInfiniteTransition(label = "shimmer")
    val translateAnim by transition.animateFloat(
        initialValue = 0f,
        targetValue = 1000f,
        animationSpec = infiniteRepeatable(
            animation = tween(1000, easing = LinearEasing),
            repeatMode = RepeatMode.Restart
        ),
        label = "shimmer"
    )
    
    val brush = Brush.linearGradient(
        colors = shimmerColors,
        start = Offset.Zero,
        end = Offset(x = translateAnim, y = translateAnim)
    )
    
    Box(
        modifier = Modifier
            .fillMaxWidth()
            .height(200.dp)
            .background(brush)
    )
}
```

## Examples (Input → Output)

### Expandable Card

**Input request:** "Create an expandable card with smooth height animation"

**Output:**
```kotlin
@Composable
fun ExpandableCard(
    title: String,
    summary: String,
    details: String
) {
    var expanded by remember { mutableStateOf(false) }
    
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
            .clickable { expanded = !expanded },
        elevation = CardDefaults.cardElevation(4.dp)
    ) {
        Column(
            modifier = Modifier.padding(16.dp)
        ) {
            Text(
                text = title,
                style = MaterialTheme.typography.titleMedium
            )
            
            Spacer(modifier = Modifier.height(8.dp))
            
            Text(
                text = summary,
                style = MaterialTheme.typography.bodyMedium
            )
            
            AnimatedVisibility(
                visible = expanded,
                enter = expandVertically(
                    animationSpec = spring(
                        stiffness = Spring.StiffnessMedium,
                        dampingRatio = Spring.DampingRatioMediumBouncy
                    )
                ) + fadeIn(),
                exit = shrinkVertically() + fadeOut()
            ) {
                Column {
                    Spacer(modifier = Modifier.height(16.dp))
                    Divider()
                    Spacer(modifier = Modifier.height(16.dp))
                    Text(
                        text = details,
                        style = MaterialTheme.typography.bodySmall
                    )
                }
            }
            
            Icon(
                imageVector = if (expanded) 
                    Icons.Default.ExpandLess else Icons.Default.ExpandMore,
                contentDescription = if (expanded) "Collapse" else "Expand",
                modifier = Modifier.align(Alignment.CenterHorizontally)
            )
        }
    }
}
```

### Number Counter Animation

**Input request:** "Create an animated counter that animates when number changes"

**Output:**
```kotlin
@Composable
fun AnimatedCounter(
    count: Int,
    modifier: Modifier = Modifier
) {
    AnimatedContent(
        targetState = count,
        modifier = modifier,
        transitionSpec = {
            // Compare incoming and outgoing numbers
            if (targetState > initialState) {
                // Counting up: slide up
                slideInVertically { height -> height } + fadeIn() with
                slideOutVertically { height -> -height } + fadeOut()
            } else {
                // Counting down: slide down
                slideInVertically { height -> -height } + fadeIn() with
                slideOutVertically { height -> height } + fadeOut()
            }.using(
                SizeTransform(clip = false)
            )
        }
    ) { targetCount ->
        Text(
            text = "$targetCount",
            style = MaterialTheme.typography.displayLarge,
            fontWeight = FontWeight.Bold,
            color = MaterialTheme.colorScheme.primary
        )
    }
}

// Usage with animated background
@Composable
fun CounterWithEffect(count: Int) {
    val backgroundColor by animateColorAsState(
        targetValue = when {
            count > 0 -> Color.Green.copy(alpha = 0.1f)
            count < 0 -> Color.Red.copy(alpha = 0.1f)
            else -> Color.Transparent
        },
        animationSpec = tween(300)
    )
    
    Box(
        modifier = Modifier
            .background(backgroundColor, RoundedCornerShape(8.dp))
            .padding(24.dp),
        contentAlignment = Alignment.Center
    ) {
        AnimatedCounter(count = count)
    }
}
```

## Best Practices

1. **Use appropriate animation spec**:
   - `spring()` for natural motion (physically based)
   - `tween()` for precise duration control
   - `keyframes()` for complex multi-step animations

2. **Add labels to animations**: Helps with debugging in Animation Inspector
   ```kotlin
   animateDpAsState(targetValue, label = "card size")
   ```

3. **Use `animateContentSize()` for auto-height animations**:
   ```kotlin
   Column(modifier = Modifier.animateContentSize()) { }
   ```

4. **Avoid animating heavy calculations**: Do work outside animation

5. **Use `LaunchedEffect` for animation triggers**:
   ```kotlin
   LaunchedEffect(key) {
       // Trigger animation
   }
   ```

6. **Test on different devices**: Low-end devices may struggle with complex animations

7. **Respect user preferences**: Check `WindowAnimationScale` for accessibility
   ```kotlin
   val animationEnabled = Settings.Global.getFloat(
       context.contentResolver,
       Settings.Global.ANIMATOR_DURATION_SCALE
   ) > 0
   ```

8. **Use `key` for proper list animations**: Essential for `LazyColumn` animations

9. **Keep animations under 300ms**: Best for perceived performance

10. **Use `graphicsLayer` for complex transforms**: Better performance than `Modifier.transform`

## Resources

- [Compose Animation documentation](https://developer.android.com/jetpack/compose/animation)
- [Animation cheat sheet](https://developer.android.com/jetpack/compose/animation/cheat-sheet)
- [Motion & animation guidelines](https://m3.material.io/styles/motion/overview)
