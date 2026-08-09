# 🟡 Medium — Jetpack Compose

> Side effects, ViewModel integration, and unidirectional data flow.
---

### 13. `remember` vs `rememberSaveable`

`remember` loses its value on **configuration change** (e.g. rotation). `rememberSaveable` survives it by saving into the instance state bundle.

```kotlin
var name by rememberSaveable { mutableStateOf("") } // survives rotation
```

---

### 14. State vs StateFlow

Compose `State` is the UI-observable value; `StateFlow` is a **coroutine-based hot stream** — typically the ViewModel exposes `StateFlow`, and Compose converts it to `State` for reading.

```kotlin
val uiState by viewModel.uiState.collectAsState()
```

---

### 15. `collectAsStateWithLifecycle`

Like `collectAsState`, but **pauses collection** when the UI isn't visible (e.g. app backgrounded) — avoids wasted work.

```kotlin
val uiState by viewModel.uiState.collectAsStateWithLifecycle()
```

---

### 16. `LaunchedEffect`

Runs a **suspend block** tied to the Composition — starts when it enters, cancels when it leaves or its key changes.

🏠 Like starting a kettle the moment you walk into the kitchen, and turning it off automatically when you leave.

```kotlin
LaunchedEffect(userId) {
    viewModel.loadUser(userId) // re-runs if userId changes
}
```

---

### 17. `SideEffect`

Runs a block on **every successful recomposition** — used to sync Compose state with non-Compose code (e.g. an analytics SDK).

```kotlin
SideEffect {
    analytics.setUserProperty("screen", "Profile")
}
```

---

### 18. `DisposableEffect`

Like `LaunchedEffect`, but for **non-suspending setup/cleanup** — you must provide an `onDispose {}` block.

🏠 Registering for a newsletter (setup) and unsubscribing when you leave (cleanup).

```kotlin
DisposableEffect(Unit) {
    val listener = registerListener()
    onDispose { unregisterListener(listener) }
}
```

---

### 19. `rememberUpdatedState`

Keeps a **lambda/value always up to date** inside a long-running effect, without restarting the effect itself.

```kotlin
val currentOnTimeout by rememberUpdatedState(onTimeout)
LaunchedEffect(Unit) {
    delay(5000)
    currentOnTimeout() // always calls the latest lambda
}
```

---

### 20. `derivedStateOf`

Computes a value from other state, but only **triggers recomposition when the derived result actually changes** — not on every input change.

```kotlin
val showButton by remember {
    derivedStateOf { listState.firstVisibleItemIndex > 0 }
}
```

---

### 21. `snapshotFlow`

Converts Compose `State` reads into a **cold Flow** — useful for reacting to state changes with Flow operators like `debounce`.

```kotlin
LaunchedEffect(Unit) {
    snapshotFlow { scrollState.value }
        .debounce(300)
        .collect { position -> onScroll(position) }
}
```

---

### 22. What is CompositionLocal?

A way to pass data **implicitly down the tree** (like theme colors) without threading it through every function parameter.

```kotlin
val LocalUserName = compositionLocalOf { "Guest" }

CompositionLocalProvider(LocalUserName provides "Rehan") {
    Greeting() // can read LocalUserName.current inside
}
```

---

### 23. LazyColumn Keys

A stable `key` per item tells Compose **which item is which** across list updates — without it, reordering/insertion can cause wrong recompositions or lost state.

```kotlin
LazyColumn {
    items(users, key = { it.id }) { user -> UserRow(user) }
}
```

---

### 24. Stable Keys

Keys should be **unique and unchanging** for the same logical item (like a DB id) — using list index as key breaks state when items are inserted/removed.

```kotlin
// ❌ index changes when list reorders
items(list) { item -> Row(item) }

// ✅ stable identity
items(list, key = { it.id }) { item -> Row(item) }
```

---

### 25. State Hoisting with ViewModel

The `ViewModel` owns the state (survives rotation); the composable just **reads and reports events up** — the classic hoisting pattern applied to real apps.

```kotlin
@Composable
fun ProfileScreen(viewModel: ProfileViewModel) {
    val state by viewModel.uiState.collectAsStateWithLifecycle()
    ProfileContent(state, onSave = viewModel::saveProfile)
}
```

---

### 26. Compose and ViewModel

`ViewModel` survives recomposition and configuration changes; Compose reads its exposed `State`/`StateFlow` and sends events back via lambdas.

```kotlin
class ProfileViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(ProfileState())
    val uiState: StateFlow<ProfileState> = _uiState.asStateFlow()
}
```

---

### 27. Compose Navigation

`NavHost` + `NavController` manage a **back stack of composable screens**, defined by routes.

```kotlin
NavHost(navController, startDestination = "home") {
    composable("home") { HomeScreen(onNavigate = { navController.navigate("details") }) }
    composable("details") { DetailsScreen() }
}
```

---

### 28. Events in Compose

UI events (clicks, text input) flow **up** as lambda calls — the composable never mutates state directly, it just reports "this happened."

```kotlin
@Composable
fun LoginButton(onLoginClick: () -> Unit) {
    Button(onClick = onLoginClick) { Text("Login") } // reports event upward
}
```

---

### 29. What is Unidirectional Data Flow?

**State flows down**, **events flow up** — a one-way cycle that keeps UI predictable: ViewModel → State → UI → Event → ViewModel.

```
State  ──────────▶  UI
  ▲                  │
  └──── Event ◀──────┘
```

---

### 30. What is UI State?

A single **immutable data class** representing everything the screen needs to render (loading, data, error) — makes state predictable and easy to test.

```kotlin
data class ProfileState(
    val isLoading: Boolean = false,
    val user: User? = null,
    val error: String? = null
)
```

---
