---
name: material-design-3
description: Guides Material Design 3 (M3) implementation for Android apps. Use when theming apps, choosing colors, implementing components, creating adaptive layouts, or following Material design principles. Covers color schemes, typography, components, tokens, and adaptive design.
tags: ["android", "material-design", "m3", "compose", "ui", "theming", "components"]
difficulty: intermediate
category: ui
version: "1.0.0"
last_updated: "2025-01-29"
---

# Material Design 3 for Android

## Quick Start

Add M3 dependencies:

```kotlin
dependencies {
    implementation("androidx.compose.material3:material3:1.2.0")
    implementation("androidx.compose.material3:material3-window-size-class:1.2.0")
}
```

Basic M3 theme setup:

```kotlin
@Composable
fun MyAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) 
            else dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}

private val LightColorScheme = lightColorScheme(
    primary = md_theme_light_primary,
    onPrimary = md_theme_light_onPrimary,
    primaryContainer = md_theme_light_primaryContainer,
    onPrimaryContainer = md_theme_light_onPrimaryContainer,
    // ... other colors
)

private val DarkColorScheme = darkColorScheme(
    primary = md_theme_dark_primary,
    onPrimary = md_theme_dark_onPrimary,
    // ... other colors
)
```

## Core Concepts

### Color System

**M3 color roles:**

| Role | Usage | Token |
|------|-------|-------|
| Primary | Main brand color, key actions | `primary` |
| Secondary | Less prominent actions | `secondary` |
| Tertiary | Contrasting accents | `tertiary` |
| Error | Errors, warnings | `error` |
| Surface | Backgrounds, cards | `surface` |
| Inverse | Opposite theme elements | `inversePrimary` |

**Surface variants:**
```kotlin
// Hierarchy of surfaces
surface (main background)
surfaceVariant (switches, tracks)
inverseSurface (snackbars, dialogs)

// Containers
primaryContainer / onPrimaryContainer
secondaryContainer / onSecondaryContainer
tertiaryContainer / onTertiaryContainer
errorContainer / onErrorContainer
```

**Dynamic color (Android 12+):**

```kotlin
// Use system wallpaper colors automatically
val colorScheme = if (darkTheme) {
    dynamicDarkColorScheme(context)
} else {
    dynamicLightColorScheme(context)
}

// Fallback for older Android versions
val colorScheme = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
    dynamicDarkColorScheme(context)
} else {
    DarkColorScheme // Your custom fallback
}
```

**Custom color scheme:**

```kotlin
val LightColorScheme = lightColorScheme(
    primary = Color(0xFF006C4C),
    onPrimary = Color(0xFFFFFFFF),
    primaryContainer = Color(0xFF89F8C7),
    onPrimaryContainer = Color(0xFF002114),
    secondary = Color(0xFF4D6357),
    onSecondary = Color(0xFFFFFFFF),
    secondaryContainer = Color(0xFFCFE9D9),
    onSecondaryContainer = Color(0xFF0A1F16),
    tertiary = Color(0xFF3D6373),
    onTertiary = Color(0xFFFFFFFF),
    tertiaryContainer = Color(0xFFC1E8FB),
    onTertiaryContainer = Color(0xFF001F29),
    error = Color(0xFFBA1A1A),
    onError = Color(0xFFFFFFFF),
    errorContainer = Color(0xFFFFDAD6),
    onErrorContainer = Color(0xFF410002),
    surface = Color(0xFFFBFDF9),
    onSurface = Color(0xFF191C1A),
    surfaceVariant = Color(0xFFDBE5DD),
    onSurfaceVariant = Color(0xFF404943),
    outline = Color(0xFF707973),
    inverseSurface = Color(0xFF2E312F),
    inverseOnSurface = Color(0xFFEFF1ED),
    inversePrimary = Color(0xFF6CDBAC),
    surfaceTint = Color(0xFF006C4C),
    outlineVariant = Color(0xFFBFC9C2),
    scrim = Color(0xFF000000),
)
```

**Using theme colors:**

```kotlin
@Composable
fun MyComponent() {
    val colorScheme = MaterialTheme.colorScheme
    
    Card(
        colors = CardDefaults.cardColors(
            containerColor = colorScheme.primaryContainer
        )
    ) {
        Text(
            text = "Hello",
            color = colorScheme.onPrimaryContainer
        )
    }
}
```

### Typography

**M3 type scale:**

| Style | Size | Weight | Usage |
|-------|------|--------|-------|
| displayLarge | 57sp | Regular | Hero text |
| displayMedium | 45sp | Regular | Large headers |
| displaySmall | 36sp | Regular | Medium headers |
| headlineLarge | 32sp | Regular | Screen titles |
| headlineMedium | 28sp | Regular | Section headers |
| headlineSmall | 24sp | Regular | Subsection headers |
| titleLarge | 22sp | Medium | Card titles |
| titleMedium | 16sp | Medium | App bar, dialogs |
| titleSmall | 14sp | Medium | List items |
| bodyLarge | 16sp | Regular | Primary body text |
| bodyMedium | 14sp | Regular | Secondary body text |
| bodySmall | 12sp | Regular | Captions |
| labelLarge | 14sp | Medium | Buttons |
| labelMedium | 12sp | Medium | Input labels |
| labelSmall | 11sp | Medium | Overlines |

**Custom typography:**

```kotlin
val Typography = Typography(
    displayLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 57.sp,
        lineHeight = 64.sp,
        letterSpacing = (-0.25).sp
    ),
    displayMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 45.sp,
        lineHeight = 52.sp,
        letterSpacing = 0.sp
    ),
    // ... other styles
    
    // Custom font family
    bodyLarge = TextStyle(
        fontFamily = FontFamily(
            Font(R.font.roboto_regular, FontWeight.Normal),
            Font(R.font.roboto_medium, FontWeight.Medium),
            Font(R.font.roboto_bold, FontWeight.Bold)
        ),
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp
    )
)

// Usage
Text(
    text = "Title",
    style = MaterialTheme.typography.headlineMedium
)

Text(
    text = "Body",
    style = MaterialTheme.typography.bodyLarge
)
```

### Shapes

**M3 shape scale:**

| Token | Corner Radius | Usage |
|-------|---------------|-------|
| extraSmall | 4dp | Chips, small buttons |
| small | 8dp | Buttons |
| medium | 12dp | Cards |
| large | 16dp | Dialogs |
| extraLarge | 28dp | Bottom sheets |

**Custom shapes:**

```kotlin
val Shapes = Shapes(
    extraSmall = RoundedCornerShape(4.dp),
    small = RoundedCornerShape(8.dp),
    medium = RoundedCornerShape(12.dp),
    large = RoundedCornerShape(16.dp),
    extraLarge = RoundedCornerShape(28.dp)
)

// Apply in theme
MaterialTheme(
    colorScheme = colorScheme,
    typography = Typography,
    shapes = Shapes,
    content = content
)

// Usage
Card(
    shape = MaterialTheme.shapes.medium
) { }

Button(
    shape = MaterialTheme.shapes.small
) { }
```

## Components

### Buttons

**Filled button (primary action):**

```kotlin
Button(
    onClick = { /* action */ },
    modifier = Modifier.fillMaxWidth()
) {
    Icon(Icons.Default.Send, contentDescription = null)
    Spacer(Modifier.width(8.dp))
    Text("Send")
}
```

**Outlined button (secondary action):**

```kotlin
OutlinedButton(
    onClick = { /* action */ },
    modifier = Modifier.fillMaxWidth()
) {
    Text("Cancel")
}
```

**Text button (low emphasis):**

```kotlin
TextButton(
    onClick = { /* action */ }
) {
    Text("Learn more")
}
```

**Elevated button:**

```kotlin
ElevatedButton(
    onClick = { /* action */ }
) {
    Text("Elevated")
}
```

**Tonal button (secondary container color):**

```kotlin
FilledTonalButton(
    onClick = { /* action */ }
) {
    Text("Tonal")
}
```

**Icon button:**

```kotlin
IconButton(
    onClick = { /* action */ }
) {
    Icon(Icons.Default.Favorite, contentDescription = "Favorite")
}

// Filled icon button
FilledIconButton(
    onClick = { /* action */ }
) {
    Icon(Icons.Default.Settings, contentDescription = "Settings")
}

// Tonal icon button
FilledTonalIconButton(
    onClick = { /* action */ }
) {
    Icon(Icons.Default.Edit, contentDescription = "Edit")
}

// Outlined icon button
OutlinedIconButton(
    onClick = { /* action */ }
) {
    Icon(Icons.Default.Share, contentDescription = "Share")
}
```

**FAB variants:**

```kotlin
// Small FAB
SmallFloatingActionButton(
    onClick = { /* action */ }
) {
    Icon(Icons.Default.Add, "Add")
}

// Standard FAB
FloatingActionButton(
    onClick = { /* action */ },
    shape = MaterialTheme.shapes.large
) {
    Icon(Icons.Default.Create, "Create")
}

// Large FAB
LargeFloatingActionButton(
    onClick = { /* action */ }
) {
    Icon(Icons.Default.Add, "Add")
}

// Extended FAB
ExtendedFloatingActionButton(
    onClick = { /* action */ },
    icon = { Icon(Icons.Default.Add, "Add") },
    text = { Text("Compose") }
)
```

### Cards

**Elevated card (default):**

```kotlin
ElevatedCard(
    onClick = { /* action */ },
    modifier = Modifier.fillMaxWidth()
) {
    Column(
        modifier = Modifier.padding(16.dp)
    ) {
        Text(
            text = "Title",
            style = MaterialTheme.typography.titleLarge
        )
        Spacer(Modifier.height(8.dp))
        Text(
            text = "Supporting text",
            style = MaterialTheme.typography.bodyMedium
        )
    }
}
```

**Filled card:**

```kotlin
Card(
    modifier = Modifier.fillMaxWidth(),
    colors = CardDefaults.cardColors(
        containerColor = MaterialTheme.colorScheme.surfaceVariant
    )
) {
    // Content
}
```

**Outlined card:**

```kotlin
OutlinedCard(
    onClick = { /* action */ },
    modifier = Modifier.fillMaxWidth()
) {
    // Content
}
```

### Lists

**Single-line list item:**

```kotlin
ListItem(
    headlineContent = { Text("Headline") },
    leadingContent = {
        Icon(Icons.Default.Person, contentDescription = null)
    },
    trailingContent = {
        Icon(Icons.Default.ChevronRight, contentDescription = null)
    }
)
```

**Two-line list item:**

```kotlin
ListItem(
    headlineContent = { Text("Headline") },
    supportingContent = { Text("Supporting text") },
    leadingContent = {
        Icon(Icons.Default.Email, contentDescription = null)
    },
    trailingContent = { Text("10m") }
)
```

**Three-line list item:**

```kotlin
ListItem(
    headlineContent = { Text("Headline") },
    overlineContent = { Text("Overline") },
    supportingContent = { Text("Longer supporting text that extends to multiple lines") },
    leadingContent = {
        Box(
            modifier = Modifier
                .size(40.dp)
                .background(MaterialTheme.colorScheme.primaryContainer, CircleShape)
        )
    }
)
```

### Navigation

**Navigation bar (bottom):**

```kotlin
var selectedItem by remember { mutableIntStateOf(0) }
val items = listOf("Home", "Search", "Library")

NavigationBar {
    items.forEachIndexed { index, item ->
        NavigationBarItem(
            icon = { 
                Icon(
                    if (selectedItem == index) Icons.Filled.Home else Icons.Outlined.Home,
                    contentDescription = item
                )
            },
            label = { Text(item) },
            selected = selectedItem == index,
            onClick = { selectedItem = index }
        )
    }
}
```

**Navigation rail (side):**

```kotlin
NavigationRail {
    items.forEachIndexed { index, item ->
        NavigationRailItem(
            icon = { Icon(Icons.Default.Home, contentDescription = item) },
            label = { Text(item) },
            selected = selectedItem == index,
            onClick = { selectedItem = index }
        )
    }
}
```

**Navigation drawer:**

```kotlin
ModalNavigationDrawer(
    drawerContent = {
        ModalDrawerSheet {
            Spacer(Modifier.height(12.dp))
            NavigationDrawerItem(
                icon = { Icon(Icons.Default.Home, contentDescription = null) },
                label = { Text("Home") },
                selected = true,
                onClick = { /* Handle click */ }
            )
            NavigationDrawerItem(
                icon = { Icon(Icons.Default.Settings, contentDescription = null) },
                label = { Text("Settings") },
                selected = false,
                onClick = { /* Handle click */ }
            )
        }
    }
) {
    // Screen content
}
```

### App Bars

**Top app bar (center-aligned):**

```kotlin
CenterAlignedTopAppBar(
    title = { Text("Title") },
    navigationIcon = {
        IconButton(onClick = { /* Handle back */ }) {
            Icon(Icons.Default.ArrowBack, "Back")
        }
    },
    actions = {
        IconButton(onClick = { /* Handle search */ }) {
            Icon(Icons.Default.Search, "Search")
        }
        IconButton(onClick = { /* Handle more */ }) {
            Icon(Icons.Default.MoreVert, "More")
        }
    }
)
```

**Small top app bar:**

```kotlin
TopAppBar(
    title = { Text("Title") },
    navigationIcon = {
        IconButton(onClick = { }) {
            Icon(Icons.Default.Menu, "Menu")
        }
    }
)
```

**Medium top app bar:**

```kotlin
MediumTopAppBar(
    title = { Text("Title") },
    navigationIcon = {
        IconButton(onClick = { }) {
            Icon(Icons.Default.ArrowBack, "Back")
        }
    }
)
```

**Large top app bar:**

```kotlin
LargeTopAppBar(
    title = { Text("Title") },
    navigationIcon = {
        IconButton(onClick = { }) {
            Icon(Icons.Default.ArrowBack, "Back")
        }
    }
)
```

### Chips

**Assist chip (refines search/filter):**

```kotlin
AssistChip(
    onClick = { /* action */ },
    label = { Text("Assist") },
    leadingIcon = {
        Icon(Icons.Default.Search, contentDescription = null)
    }
)
```

**Filter chip (toggle filters):**

```kotlin
var selected by remember { mutableStateOf(false) }

FilterChip(
    selected = selected,
    onClick = { selected = !selected },
    label = { Text("Filter") },
    leadingIcon = if (selected) {
        {
            Icon(
                imageVector = Icons.Filled.Done,
                contentDescription = "Selected",
                modifier = Modifier.size(FilterChipDefaults.IconSize)
            )
        }
    } else {
        null
    }
)
```

**Input chip (user input):**

```kotlin
InputChip(
    onClick = { },
    label = { Text("Input chip") },
    avatar = {
        Icon(Icons.Default.Person, contentDescription = null)
    },
    trailingIcon = {
        Icon(Icons.Default.Close, contentDescription = "Remove")
    }
)
```

**Suggestion chip (action prompt):**

```kotlin
SuggestionChip(
    onClick = { /* action */ },
    label = { Text("Suggestion") }
)
```

## Examples (Input → Output)

### Complete M3 Theme

**Input request:** "Create a complete M3 theme with custom colors and typography"

**Output:**

```kotlin
// ui/theme/Color.kt
val md_theme_light_primary = Color(0xFF006C4C)
val md_theme_light_onPrimary = Color(0xFFFFFFFF)
val md_theme_light_primaryContainer = Color(0xFF89F8C7)
val md_theme_light_onPrimaryContainer = Color(0xFF002114)
val md_theme_light_secondary = Color(0xFF4D6357)
val md_theme_light_onSecondary = Color(0xFFFFFFFF)
val md_theme_light_secondaryContainer = Color(0xFFCFE9D9)
val md_theme_light_onSecondaryContainer = Color(0xFF0A1F16)
val md_theme_light_tertiary = Color(0xFF3D6373)
val md_theme_light_onTertiary = Color(0xFFFFFFFF)
val md_theme_light_tertiaryContainer = Color(0xFFC1E8FB)
val md_theme_light_onTertiaryContainer = Color(0xFF001F29)
val md_theme_light_error = Color(0xFFBA1A1A)
val md_theme_light_onError = Color(0xFFFFFFFF)
val md_theme_light_errorContainer = Color(0xFFFFDAD6)
val md_theme_light_onErrorContainer = Color(0xFF410002)
val md_theme_light_surface = Color(0xFFFBFDF9)
val md_theme_light_onSurface = Color(0xFF191C1A)
val md_theme_light_surfaceVariant = Color(0xFFDBE5DD)
val md_theme_light_onSurfaceVariant = Color(0xFF404943)
val md_theme_light_outline = Color(0xFF707973)
val md_theme_light_inverseSurface = Color(0xFF2E312F)
val md_theme_light_inverseOnSurface = Color(0xFFEFF1ED)
val md_theme_light_inversePrimary = Color(0xFF6CDBAC)

// Dark theme colors
val md_theme_dark_primary = Color(0xFF6CDBAC)
val md_theme_dark_onPrimary = Color(0xFF003826)
// ... etc

// ui/theme/Type.kt
val Typography = Typography(
    displayLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 57.sp,
        lineHeight = 64.sp
    ),
    headlineLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 32.sp,
        lineHeight = 40.sp
    ),
    titleLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 22.sp,
        lineHeight = 28.sp
    ),
    bodyLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp
    ),
    labelLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 14.sp,
        lineHeight = 20.sp
    )
)

// ui/theme/Shape.kt
val Shapes = Shapes(
    extraSmall = RoundedCornerShape(4.dp),
    small = RoundedCornerShape(8.dp),
    medium = RoundedCornerShape(12.dp),
    large = RoundedCornerShape(16.dp),
    extraLarge = RoundedCornerShape(28.dp)
)

// ui/theme/Theme.kt
@Composable
fun MyAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) 
            else dynamicLightColorScheme(context)
        }
        darkTheme -> darkColorScheme(
            primary = md_theme_dark_primary,
            onPrimary = md_theme_dark_onPrimary,
            // ... etc
        )
        else -> lightColorScheme(
            primary = md_theme_light_primary,
            onPrimary = md_theme_light_onPrimary,
            // ... etc
        )
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        shapes = Shapes,
        content = content
    )
}
```

### Custom Component with M3

**Input request:** "Create a status card component using M3 design tokens"

**Output:**

```kotlin
@Composable
fun StatusCard(
    status: Status,
    title: String,
    description: String,
    onAction: () -> Unit,
    modifier: Modifier = Modifier
) {
    val colorScheme = MaterialTheme.colorScheme
    
    val (containerColor, contentColor) = when (status) {
        Status.Success -> colorScheme.primaryContainer to colorScheme.onPrimaryContainer
        Status.Warning -> colorScheme.tertiaryContainer to colorScheme.onTertiaryContainer
        Status.Error -> colorScheme.errorContainer to colorScheme.onErrorContainer
        Status.Info -> colorScheme.secondaryContainer to colorScheme.onSecondaryContainer
    }
    
    val icon = when (status) {
        Status.Success -> Icons.Default.CheckCircle
        Status.Warning -> Icons.Default.Warning
        Status.Error -> Icons.Default.Error
        Status.Info -> Icons.Default.Info
    }
    
    Card(
        modifier = modifier.fillMaxWidth(),
        colors = CardDefaults.cardColors(
            containerColor = containerColor
        ),
        shape = MaterialTheme.shapes.medium
    ) {
        Row(
            modifier = Modifier
                .padding(16.dp)
                .fillMaxWidth(),
            horizontalArrangement = Arrangement.spacedBy(16.dp)
        ) {
            Icon(
                imageVector = icon,
                contentDescription = null,
                tint = contentColor,
                modifier = Modifier.size(24.dp)
            )
            
            Column(
                modifier = Modifier.weight(1f)
            ) {
                Text(
                    text = title,
                    style = MaterialTheme.typography.titleMedium,
                    color = contentColor
                )
                Spacer(Modifier.height(4.dp))
                Text(
                    text = description,
                    style = MaterialTheme.typography.bodyMedium,
                    color = contentColor.copy(alpha = 0.8f)
                )
            }
            
            TextButton(
                onClick = onAction,
                colors = ButtonDefaults.textButtonColors(
                    contentColor = contentColor
                )
            ) {
                Text("Action")
            }
        }
    }
}

enum class Status {
    Success, Warning, Error, Info
}
```

## Best Practices

1. **Use dynamic color when available**: Respects user wallpaper on Android 12+
2. **Always use on-colors**: `onPrimary`, `onSurface` for text/icons on colored backgrounds
3. **Follow type hierarchy**: Use appropriate text styles for consistency
4. **Use elevation appropriately**: Combine with surface colors for hierarchy
5. **Respect touch targets**: Minimum 48dp for interactive elements
6. **Material motion**: Use standard motion durations (quick: 100ms, standard: 300ms)
7. **Accessibility**: Ensure 4.5:1 contrast ratios for text
8. **Component consistency**: Use M3 components rather than custom ones when possible
9. **Adaptive design**: Use `WindowSizeClass` for responsive layouts
10. **Theme consistency**: Apply theme at app root, use `MaterialTheme` tokens everywhere

## Resources

- [Material Design 3 guidelines](https://m3.material.io/)
- [Compose Material 3 components](https://developer.android.com/reference/kotlin/androidx/compose/material3/package-summary)
- [Material Theme Builder](https://m3.material.io/theme-builder)
- [Color system](https://m3.material.io/styles/color/the-color-system/key-colors-tones)
- [Typography scale](https://m3.material.io/styles/typography/type-scale-tokens/overview)
