<div align="center">

# 🔁 Jetpack Compose — Predict the Recomposition

**15 scenario-based questions.** Read the code, guess what recomposes (and how many times), then reveal the answer.

![Compose](https://img.shields.io/badge/Jetpack-Compose-4285F4?logo=jetpackcompose&logoColor=white)
![Format](https://img.shields.io/badge/Format-Predict%20the%20Recomposition-blueviolet)
![Level](https://img.shields.io/badge/Level-Basic%20→%20Advanced-orange)

</div>

---

## 💡 Why this file is different

Compose interview questions rarely stop at *"what is recomposition?"* — the real test is: **look at this code, tell me exactly what redraws and what doesn't, and why.**

Every question here:

1. Shows a small piece of Compose code with **simple, clear comments**
2. Asks you to predict **what recomposes** — before you look at the answer
3. Reveals the answer **and the reason**, in plain words, once you click

> 🧠 Try to answer out loud first, like you're in an interview. Then click to check yourself.

---

## 🟢 Level 1 — Basic

### Q1

```kotlin
@Composable
fun Counter() {
    // "count" is remembered — Compose keeps this value
    // alive across recompositions of THIS function
    var count by remember { mutableStateOf(0) }

    Column {
        // this Text reads "count", so it recomposes
        // whenever "count" changes
        Text("Count: $count")

        // this Text does NOT read "count" at all —
        // it never has a reason to recompose
        Text("This never changes")

        Button(onClick = { count++ }) {
            Text("Add")
        }
    }
}
```

**Question:** When you tap the button 3 times, how many times does each `Text` recompose?

<details>
<summary>📤 Reveal Answer</summary>

- `Text("Count: $count")` → recomposes **3 times** (once per click, since it reads `count`)
- `Text("This never changes")` → recomposes **0 times** (it never reads `count`, so Compose has no reason to re-run it)

**Why:** Compose only recomposes a composable if it actually **reads** a piece of state that changed. Reading state is tracked function-by-function, not "the whole screen at once" — this is why fine-grained recomposition is possible.

</details>

---

### Q2

```kotlin
@Composable
fun Screen() {
    // count is declared WITHOUT remember —
    // this is a common beginner mistake
    var count = 0

    Column {
        Text("Count: $count")
        Button(onClick = {
            count++ // this changes the local variable...
            // ...but Compose doesn't know this variable
            // even exists, so nothing tells it to recompose
        }) {
            Text("Add")
        }
    }
}
```

**Question:** Does clicking the button ever update the text on screen?

<details>
<summary>📤 Reveal Answer</summary>

**No — the text never updates**, no matter how many times you click.

**Why:** `count` here is a plain `Int`, not a Compose `State`. Compose has no way of knowing it changed, so there's nothing to trigger recomposition. Also, since `remember` isn't used, `count` would even reset to `0` on every recomposition anyway. You need `remember { mutableStateOf(0) }` for Compose to both track the value AND know when it changes.

</details>

---

### Q3

```kotlin
@Composable
fun Parent() {
    var text by remember { mutableStateOf("Hello") }

    Column {
        // this Text reads "text" directly, so it recomposes
        // whenever "text" changes
        Text(text)

        Child() // Child does not receive "text" as a parameter at all

        Button(onClick = { text = "Updated" }) {
            Text("Change text")
        }
    }
}

@Composable
fun Child() {
    // Child has no parameters, and doesn't read
    // any state from Parent — it's completely unrelated
    Text("I am the child")
}
```

**Question:** When you click the button, does `Child()` recompose?

<details>
<summary>📤 Reveal Answer</summary>

**No, `Child()` does not recompose.**

**Why:** `Child()` doesn't take any parameters and doesn't read `text` (or any other changed state) inside its own body. Compose only recomposes functions that actually depend on the state that changed — being physically nearby in the UI tree doesn't matter, only what state is actually *read*.

</details>

---

### Q4

```kotlin
@Composable
fun Greeting() {
    // remember WITHOUT rememberSaveable —
    // this survives normal recomposition,
    // but NOT a configuration change like screen rotation
    var name by remember { mutableStateOf("") }

    Column {
        TextField(value = name, onValueChange = { name = it })
        Text("Hello, $name")
    }
}
```

**Question:** If the user types "Rehan" and then rotates the screen, what happens to the text field?

<details>
<summary>📤 Reveal Answer</summary>

**The text field resets back to empty ("").**

**Why:** `remember` only keeps a value alive across normal recompositions — it does **not** survive a configuration change like rotation, because the Activity (and the whole Composition) is recreated. To survive rotation, you'd need `rememberSaveable` instead, which saves the value into the saved-instance-state bundle.

</details>

---

## 🟡 Level 2 — Intermediate

### Q5

```kotlin
@Composable
fun ParentScreen() {
    var counter by remember { mutableStateOf(0) }

    Column {
        Button(onClick = { counter++ }) {
            Text("Increment")
        }

        // counter is passed down as a plain Int parameter
        ChildDisplay(value = counter)
    }
}

@Composable
fun ChildDisplay(value: Int) {
    // Int is a STABLE type, so Compose can safely skip
    // recomposing this function if "value" hasn't changed
    Text("Value: $value")
}
```

**Question:** Does `ChildDisplay` recompose every time `ParentScreen` recomposes, or only when `value` actually changes?

<details>
<summary>📤 Reveal Answer</summary>

**Only when `value` actually changes.**

**Why:** `Int` is a stable, immutable type. When `ParentScreen` recomposes, Compose checks whether `ChildDisplay`'s parameters actually changed — since `value` is stable and comparable with `==`, Compose can safely **skip** recomposing `ChildDisplay` if the value passed in is the same as last time.

</details>

---

### Q6

```kotlin
// A plain data class, but WITHOUT @Immutable or @Stable,
// and it uses a mutable list — this makes it UNSTABLE
data class UserListHolder(val users: MutableList<String>)

@Composable
fun UserList(holder: UserListHolder) {
    // Compose can't safely tell if "holder" changed,
    // because a MutableList can be modified without
    // Compose being told about it
    Column {
        holder.users.forEach { Text(it) }
    }
}
```

**Question:** If the parent recomposes but `holder` is "the same object" (same reference, list mutated in place), does `UserList` skip recomposition?

<details>
<summary>📤 Reveal Answer</summary>

**No — `UserList` will NOT be skipped; it recomposes every time the parent recomposes.**

**Why:** `MutableList` is an unstable type from Compose's point of view, because its contents can change without Compose being notified. Since `holder` contains a mutable list, the whole `UserListHolder` class is treated as unstable too — so Compose can't safely assume "same reference = same content," and it recomposes `UserList` every time just to be safe.

**Fix:** use an immutable `List<String>` and mark the class `@Immutable`, so Compose can trust it and skip recomposition when nothing actually changed.

</details>

---

### Q7

```kotlin
@Composable
fun ParentScreen() {
    var count by remember { mutableStateOf(0) }

    Column {
        Text("Count: $count")

        // this lambda is created FRESH every single time
        // ParentScreen recomposes — it's a brand new object
        // in memory each time, even though it "looks the same"
        ChildButton(onClick = { count++ })
    }
}

@Composable
fun ChildButton(onClick: () -> Unit) {
    Button(onClick = onClick) {
        Text("Add")
    }
}
```

**Question:** Does passing a lambda like this prevent `ChildButton` from being skipped during recomposition?

<details>
<summary>📤 Reveal Answer</summary>

**It depends on the Compose compiler version — but historically, yes, it often prevented skipping.**

**Why:** A lambda that captures variables from its surrounding scope is normally treated as unstable, because a "new" lambda object is created on every recomposition, even if its behavior is identical. Older Compose compilers would then treat `ChildButton`'s `onClick` parameter as "changed" every time, forcing it to recompose.

**Good news:** with newer Compose compilers, lambdas passed directly to composables are automatically remembered/stabilized under the hood — and "Strong Skipping" mode (Compose 1.5.4+) makes this kind of case skippable by default even without manual fixes. Still, it's a very common thing interviewers probe, because it used to be a real performance trap.

</details>

---

### Q8

```kotlin
@Composable
fun SearchScreen(items: List<String>) {
    var query by remember { mutableStateOf("") }

    // this filtering runs on EVERY recomposition of SearchScreen,
    // even if "query" hasn't actually changed since last time —
    // for example, if some unrelated state changes and causes
    // a recomposition anyway
    val filtered = items.filter { it.contains(query) }

    Column {
        TextField(value = query, onValueChange = { query = it })
        filtered.forEach { Text(it) }
    }
}
```

**Question:** Is there a performance problem here, and how would you fix it?

<details>
<summary>📤 Reveal Answer</summary>

**Yes — `items.filter { ... }` re-runs on every recomposition of `SearchScreen`, even when neither `items` nor `query` changed.**

**Why:** A plain `val` computed inside a composable body is recalculated every time the function re-runs — Compose doesn't know it can skip this line specifically. This wastes CPU on unnecessary filtering.

**Fix:** wrap it in `remember`, keyed on the inputs it depends on:
```kotlin
val filtered = remember(items, query) {
    items.filter { it.contains(query) }
}
```
Now it only recalculates when `items` or `query` actually change.

</details>

---

### Q9

```kotlin
@Composable
fun ScrollAwareHeader(listState: LazyListState) {
    // this reads listState.firstVisibleItemIndex directly —
    // that value changes on EVERY pixel of scroll,
    // so this Text recomposes constantly while scrolling
    val isScrolled = listState.firstVisibleItemIndex > 0

    Text(if (isScrolled) "Scrolled" else "At top")
}
```

**Question:** How often does this `Text` recompose while the user is scrolling, and how would you reduce that?

<details>
<summary>📤 Reveal Answer</summary>

**It recomposes very frequently — on almost every scroll position change**, even though the actual displayed text ("Scrolled" vs "At top") only changes at ONE specific point (crossing index 0).

**Why:** `isScrolled` is recalculated as a plain `val` every recomposition, and it changes value on every tiny scroll movement, so `Text` keeps recomposing even when the boolean result would be identical most of the time.

**Fix:** wrap it in `derivedStateOf`, which only notifies dependents when the **derived result** actually changes:
```kotlin
val isScrolled by remember {
    derivedStateOf { listState.firstVisibleItemIndex > 0 }
}
```
Now `Text` only recomposes at the exact moment `isScrolled` flips between `true` and `false`.

</details>

---

### Q10

```kotlin
@Composable
fun UserRows(users: List<User>) {
    // no "key" is provided here — Compose falls back to
    // using the ITEM'S POSITION in the list as its identity
    LazyColumn {
        items(users) { user ->
            UserRow(user)
        }
    }
}
```

**Question:** If a new user is inserted at the **top** of the list, what problem can happen here?

<details>
<summary>📤 Reveal Answer</summary>

**Every row below the insertion point can recompose unnecessarily, and any local UI state stored per-row (like an expanded/collapsed toggle) can attach itself to the wrong user.**

**Why:** without a stable `key`, Compose identifies each row by its **position** in the list (index 0, 1, 2...), not by the actual item's identity. When a new item is inserted at the top, everything shifts down by one position — so Compose thinks "the item at position 1 changed" for every row that shifted, even though those users didn't actually change at all. It also means any `remember`ed state tied to a row can end up associated with the wrong user after the shift.

**Fix:** always provide a stable, unique key:
```kotlin
items(users, key = { it.id }) { user -> UserRow(user) }
```
Now Compose tracks each row by the user's actual `id`, so insertions/removals don't cause unrelated rows to shuffle their state.

</details>

---

## 🔴 Level 3 — Advanced

### Q11

```kotlin
@Composable
fun ProfileScreen(viewModel: ProfileViewModel) {
    // collectAsState reads the CURRENT value of the StateFlow
    // and turns it into Compose State — this composable
    // recomposes whenever a NEW value is emitted
    val state by viewModel.uiState.collectAsState()

    Column {
        // this only reads state.name — but since "state" itself
        // is a single object, ANY change inside the whole
        // UiState data class causes a NEW state object,
        // which causes THIS composable to recompose
        Text("Name: ${state.name}")

        // this only reads state.age
        Text("Age: ${state.age}")
    }
}
```

**Question:** If only `age` changes in the ViewModel (name stays the same), does the `Text("Name: ...")` recompose too?

<details>
<summary>📤 Reveal Answer</summary>

**Yes — both `Text` composables recompose, even though only `age` changed.**

**Why:** `state` is a single `UiState` object. When any field inside it changes, a brand-new `UiState` instance is emitted from the `StateFlow`, and `collectAsState()` produces a new `State` value. Since **both** `Text` calls are inside the same parent composable that reads this single `state` object, the whole `Column` recomposes together — Compose doesn't automatically know that one `Text` "only cares about name."

**To reduce this:** split the UI into smaller composables that each take just the specific field they need as a parameter (e.g. `NameText(name: String)`), so Compose can skip the ones whose specific field didn't change.

</details>

---

### Q12

```kotlin
@Composable
fun Timer() {
    var seconds by remember { mutableStateOf(0) }

    // LaunchedEffect(Unit) means: start this coroutine once,
    // when Timer first enters the Composition, and DON'T
    // restart it just because Timer recomposes
    LaunchedEffect(Unit) {
        while (true) {
            delay(1000)
            seconds++ // this triggers recomposition of Timer
        }
    }

    Text("Seconds: $seconds")
}
```

**Question:** Does the `LaunchedEffect` restart every time `Timer` recomposes (i.e., every second, when `seconds` changes)?

<details>
<summary>📤 Reveal Answer</summary>

**No — the `LaunchedEffect` does NOT restart. It keeps running the same coroutine, counting up forever.**

**Why:** `LaunchedEffect` is keyed on `Unit`, which never changes. Compose only restarts a `LaunchedEffect` when its key(s) change — since `Unit` is always the same value, the effect starts once (when `Timer` first appears) and keeps running across every recomposition of `Timer`, completely independent of how often `Text` redraws.

**If you'd written `LaunchedEffect(seconds)` instead**, it WOULD restart every second — cancelling and relaunching the coroutine on every tick, which would actually break this timer.

</details>

---

### Q13

```kotlin
@Composable
fun ProductCard(product: Product, onAddToCart: (Product) -> Unit) {
    // this lambda captures "product" from the outer scope —
    // it's created fresh on every recomposition of the PARENT
    // that calls ProductCard, unless the parent guards against it
    Button(onClick = { onAddToCart(product) }) {
        Text("Add ${product.name} to cart")
    }
}

@Composable
fun ProductList(products: List<Product>, onAddToCart: (Product) -> Unit) {
    Column {
        // a NEW lambda { onAddToCart(product) } is created
        // for EVERY product, on EVERY recomposition of ProductList
        products.forEach { product ->
            ProductCard(product = product, onAddToCart = onAddToCart)
        }
    }
}
```

**Question:** What's the subtle recomposition cost hiding in `ProductList`, and how would a senior engineer improve it?

<details>
<summary>📤 Reveal Answer</summary>

**Nothing is dramatically broken here** (the lambda in `ProductCard` is created fresh each time `ProductCard` itself recomposes, which is normal), **but using `forEach` inside a `Column` instead of `LazyColumn` means every single `ProductCard` is composed immediately, all at once — even ones that aren't visible on screen.**

**Why:** `Column` composes all of its children eagerly, regardless of whether they're visible in the viewport. For a long or unbounded `products` list, this wastes composition work and memory on off-screen items.

**Fix:** use `LazyColumn` with a stable key, so only visible items are actually composed:
```kotlin
LazyColumn {
    items(products, key = { it.id }) { product ->
        ProductCard(product = product, onAddToCart = onAddToCart)
    }
}
```

</details>

---

### Q14

```kotlin
@Immutable
data class UiState(
    val isLoading: Boolean = false,
    val items: List<String> = emptyList()
)

@Composable
fun ScreenContent(state: UiState) {
    // Because UiState is marked @Immutable, Compose TRUSTS
    // that if the reference is the same, the content is the same —
    // it can safely SKIP recomposing this function
    if (state.isLoading) {
        LoadingSpinner()
    } else {
        ItemList(state.items)
    }
}
```

**Question:** What would change about this if `@Immutable` were removed from `UiState`?

<details>
<summary>📤 Reveal Answer</summary>

**Without `@Immutable`, Compose treats `UiState` as unstable by default — because it contains a `List`, which Compose can't prove is truly immutable at compile time (a `List` interface could technically be backed by a mutable implementation).**

**Effect:** `ScreenContent` would no longer be automatically skippable just because the same `UiState` reference is passed in — Compose has to be more conservative and may recompose it more often than necessary, even when nothing actually changed.

**Why `@Immutable` fixes it:** it's a promise you make to the Compose compiler — "trust me, once created, this object's contents never change." Compose then treats it as stable and can safely skip recomposition based on reference/equality checks.

</details>

---

### Q15 — Senior-Level Scenario

```kotlin
@Composable
fun ChatScreen(messages: List<Message>) {
    LazyColumn {
        items(messages, key = { it.id }) { message ->
            // formatting the timestamp is done INSIDE the
            // composable, every time this row composes
            val formattedTime = formatTimestamp(message.timestamp)

            MessageRow(text = message.text, time = formattedTime)
        }
    }
}
```

**Question:** With 5,000 chat messages, this screen is janky while scrolling. What's the recomposition-related issue, and how do you fix it?

<details>
<summary>📤 Reveal Answer</summary>

**The problem:** `formatTimestamp(message.timestamp)` runs inside the composable body, meaning it's recalculated every time that row composes — including every time it scrolls back into view, since `LazyColumn` disposes and recomposes off-screen items as they re-enter the viewport. For 5,000 messages, this repeated formatting work adds up and c
