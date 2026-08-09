# 🔴 Hard — Jetpack Compose

> Recomposition internals, performance, and senior-level architecture.
---

### 31. How does Recomposition actually work?

Compose tracks **which state each composable reads** during composition; when a state's value changes, only the composables that actually read it are scheduled for recomposition.

---

### 32. Compose Runtime

The underlying engine (separate from `compose-ui`) that manages the **Composition, Slot Table, and Snapshot state system** — it's what makes Compose's incremental updates possible.

---

### 33. Slot Table

The Runtime's internal **data structure that stores the UI tree** and associated state — it's what lets Compose diff and patch efficiently instead of rebuilding everything.

🏠 Like a seating chart that tracks exactly who sits where, so updating one seat doesn't require redoing the whole chart.

---

### 34. Compose Snapshot System

An MVCC-style (multi-version) system that tracks state reads/writes in **isolated snapshots**, so Compose knows exactly which reads depend on which writes.

```kotlin
Snapshot.withMutableSnapshot {
    stateA.value = 1
    stateB.value = 2 // both changes seen atomically
}
```

---

### 35. State Reads and Writes

A composable only recomposes if it **reads** a state value directly during its execution (e.g. `state.value`) — writing to state elsewhere doesn't trigger recomposition in composables that never read it.

```kotlin
val count = remember { mutableStateOf(0) }
Text("Static") // doesn't read count -> never recomposes
Text("${count.value}") // reads count -> recomposes on change
```

---

### 36. What is Stability in Compose?

A type is **stable** if Compose can guarantee its equality check reliably reflects whether it changed — stable parameters let Compose **skip** unnecessary recomposition.

---

### 37. Stable vs Unstable Types

`val` primitives, `String`, and immutable data classes are usually stable. `var` properties, mutable `List`/`Map`, or classes without `@Immutable`/`@Stable` are treated as **unstable** by default.

```kotlin
@Immutable
data class User(val id: String, val name: String) // stable

data class MutableUser(var name: String) // unstable — var breaks stability
```

---

### 38. What is Strong Skipping?

A compiler mode (Compose 1.5.4+) that makes **more parameter types skippable by default**, including unstable ones (using `equals` at runtime), reducing unnecessary recompositions without manual annotations.

---

### 39. Skipping Recomposition

If all of a composable's parameters are **stable and unchanged**, Compose skips re-running it entirely — a key performance optimization built into the Runtime.

```kotlin
@Composable
fun UserRow(user: User) { // stable params -> skipped if user is unchanged
    Text(user.name)
}
```

---

### 40. How do you optimize Recomposition?

- Keep state as close as possible to where it's used
- Use stable keys in lists
- Avoid reading state you don't need in a composable
- Use `derivedStateOf` for computed values
- Keep composables small and focused

```kotlin
// ❌ whole screen recomposes on scroll
@Composable
fun Screen(scrollState: ScrollState) { /* everything here */ }

// ✅ isolate the part that actually needs scrollState
@Composable
fun ScrollAwareHeader(scrollState: ScrollState) { /* only this recomposes */ }
```

---

### 41. Large LazyColumn Performance

Use **stable keys**, avoid heavy work inside item composables, use `contentType` for mixed item layouts, and avoid reading unrelated state inside list items.

```kotlin
LazyColumn {
    items(items, key = { it.id }, contentType = { it.type }) { item ->
        ItemRow(item)
    }
}
```

---

### 42. `derivedStateOf` vs `remember`

`remember` just caches a value across recomposition. `derivedStateOf` additionally **recalculates when its inputs change but only notifies dependents if the output actually differs**.

```kotlin
val isScrolled by remember { derivedStateOf { scrollState.value > 0 } }
// vs plain remember, which would recompute+notify on every scroll pixel
```

---

### 43. `LaunchedEffect` vs `rememberCoroutineScope`

`LaunchedEffect` runs automatically **tied to the Composition lifecycle**. `rememberCoroutineScope` gives you a scope to launch coroutines **manually**, e.g. from a click handler.

```kotlin
val scope = rememberCoroutineScope()
Button(onClick = { scope.launch { snackbarHostState.showSnackbar("Saved") } }) {
    Text("Save")
}
```

---

### 44. Compose + Flow Architecture

Repository exposes `Flow` → ViewModel converts to `StateFlow` (single source of truth) → Compose collects it with `collectAsStateWithLifecycle`.

```kotlin
class ProfileViewModel(repo: ProfileRepository) : ViewModel() {
    val uiState = repo.observeProfile()
        .map { ProfileState(user = it) }
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), ProfileState())
}
```

---

### 45. Where should State live?

As **low as possible** while still being shared by everything that needs it — screen-level state in the ViewModel, purely visual/transient state (like a scroll position) can stay local with `remember`.

---

### 46. Compose + Clean Architecture

Compose sits at the **UI layer** only — it reads `UiState` from the ViewModel (presentation layer), which calls UseCases (domain layer), which call Repositories (data layer). Compose never touches domain/data directly.

```
UI (Compose) → ViewModel → UseCase → Repository → Data Source
```

---

### 47. Compose + Multi-Module Architecture

Common split: a `:core-ui` module for shared design-system composables, `:feature-x` modules owning their own screens/ViewModels, and `:core-navigation` wiring routes between features.

---

### 48. How do you debug Compose Performance?

Use **Layout Inspector** to see recomposition counts/skips, enable the Compose compiler's `@Stable`/`@Immutable` reports, and use `Modifier.composed { }` or manual logging in `SideEffect` to trace unexpected recompositions.

---

### 49. Compose Testing

`ComposeTestRule` lets you set content and assert on nodes using semantics — `onNodeWithText`, `performClick`, `assertIsDisplayed`, etc.

```kotlin
@get:Rule val composeRule = createComposeRule()

@Test
fun clickIncrementsCounter() {
    composeRule.setContent { Counter() }
    composeRule.onNodeWithText("Add").performClick()
    composeRule.onNodeWithText("Count: 1").assertIsDisplayed()
}
```

---

### 50. Real-World Senior Scenario

**"A LazyColumn of 5,000 chat messages is janky while scrolling — how do you fix it?"**

Answer: add stable `key`s per message, avoid unstable lambda/list params to item composables, move expensive formatting (e.g. timestamp parsing) out of the composable into the ViewModel/mapper, and use `contentType` if message layouts vary (text vs image).

```kotlin
LazyColumn {
    items(messages, key = { it.id }, contentType = { it.type }) { message ->
        MessageRow(message) // message.formattedTime precomputed upstream
    }
}
```

---
