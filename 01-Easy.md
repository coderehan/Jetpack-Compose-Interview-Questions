# 🟢 Easy — Jetpack Compose

> Foundation questions. Every answer = **simple words + real-life example + Kotlin code.**

---

### 1. What is Jetpack Compose?

Android's modern **declarative** UI toolkit — you describe *what* the UI should look like for a given state, and Compose updates it when state changes.

🏠 Like giving a chef a recipe based on the season, instead of manually rearranging the menu board yourself every time.

```kotlin
@Composable
fun Counter(count: Int) {
    Text(text = "Count: $count") // describes UI for the current state
}
```

---

### 2. What is a Composable function?

A function marked `@Composable` that can **emit UI** — it can call other composables and re-run when its inputs change.

```kotlin
@Composable
fun Greeting(name: String) {
    Text(text = "Hello, $name!")
}
```

---

### 3. What is Composition?

The **tree of UI elements** Compose builds by running your composable functions — it's the "current picture" of your screen.

🏠 Like the layout of furniture in a room after you've arranged it once.

```kotlin
setContent {
    Greeting("Rehan") // adds a node to the Composition
}
```

---

### 4. What is Recomposition?

Re-running composable functions to **update the UI** when the state they read changes — only the affected parts re-run, not the whole screen.

🏠 Repainting only the wall that got scuffed, not the entire house.

```kotlin
var count by remember { mutableStateOf(0) }
Text("Count: $count") // re-runs only this Text when count changes
Button(onClick = { count++ }) { Text("Add") }
```

---

### 5. What is State in Compose?

A value that, when it changes, **triggers recomposition** of any composable reading it.

```kotlin
val count = remember { mutableStateOf(0) }
```

---

### 6. `remember`

Stores a value across recompositions **within the same Composition** — without it, the value would reset every time the function re-runs.

```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) } // survives recomposition
    Button(onClick = { count++ }) { Text("$count") }
}
```

---

### 7. `mutableStateOf`

Creates an **observable holder** for a value — reading `.value` inside a composable subscribes it to changes.

```kotlin
val name = remember { mutableStateOf("") }
TextField(value = name.value, onValueChange = { name.value = it })
```

---

### 8. State Hoisting

Moving state **up** to a caller, so a composable stays stateless and reusable — it just receives `value` + `onValueChange`.

🏠 A cashier doesn't decide prices — the manager (caller) hoists that decision up and just hands the cashier the price list.

```kotlin
@Composable
fun NameInput(name: String, onNameChange: (String) -> Unit) {
    TextField(value = name, onValueChange = onNameChange) // stateless
}
```

---

### 9. Modifier

A chainable set of instructions that changes a composable's **size, padding, click behavior, appearance**, etc.

```kotlin
Text(
    text = "Hello",
    modifier = Modifier
        .padding(16.dp)
        .fillMaxWidth()
        .clickable { println("Tapped") }
)
```

---

### 10. Column, Row and Box

`Column` stacks children **vertically**, `Row` **horizontally**, `Box` **overlaps** them (like stacking layers).

```kotlin
Box {
    Column { Text("Top"); Text("Bottom") }
    Row { Text("Left"); Text("Right") }
}
```

---

### 11. LazyColumn vs Column

`Column` renders **all children immediately**. `LazyColumn` only composes items that are **currently visible** — use it for long or unbounded lists.

```kotlin
LazyColumn {
    items(1000) { index -> Text("Item $index") } // only visible rows composed
}
```

---

### 12. Why should Composables be side-effect free?

Compose can call a composable **multiple times, skip it, or reorder it** — code with side effects (like network calls) directly inside a composable body would run unpredictably.

```kotlin
// ❌ side effect directly in body — runs on every recomposition
@Composable
fun Bad() { logAnalytics() }

// ✅ controlled with a proper effect API
@Composable
fun Good() { LaunchedEffect(Unit) { logAnalytics() } }
```

---
