# 🟡 Jetpack Compose Interview Questions — Medium Level

A practical Jetpack Compose interview preparation guide for Android Engineers.

This section focuses on concepts that are commonly asked once the interviewer knows you have practical Compose experience.

---

📚 Topics

13. "remember" vs "rememberSaveable"
14. State vs StateFlow
15. "collectAsStateWithLifecycle"
16. "LaunchedEffect"
17. "SideEffect"
18. "DisposableEffect"
19. "rememberUpdatedState"
20. "derivedStateOf"
21. "snapshotFlow"
22. "CompositionLocal"
23. LazyColumn Keys
24. Stable Keys
25. State Hoisting with ViewModel
26. Compose and ViewModel
27. Compose Navigation
28. Events in Compose
29. Unidirectional Data Flow
30. Compose UI State

---

13. 💾 "remember" vs "rememberSaveable"

"remember"| "rememberSaveable"
Survives recomposition| Survives recomposition
Does not automatically survive Activity recreation| Can survive configuration/process restoration when supported
Simple in-memory Composition state| Saveable state

Example

var text by remember {
    mutableStateOf("")
}

For saveable UI state:

var text by rememberSaveable {
    mutableStateOf("")
}

🧠 Remember

remember
    ↓
Recomposition

rememberSaveable
    ↓
Recomposition + saveable restoration

---

14. 🌊 State vs StateFlow

Compose State

var count by remember {
    mutableStateOf(0)
}

Primarily used for Compose-managed UI state.

StateFlow

private val _uiState =
    MutableStateFlow(UiState())

val uiState =
    _uiState.asStateFlow()

Usually used by ViewModel/application layers to expose observable state.

Typical Architecture

ViewModel
   ↓
StateFlow
   ↓
Composable
   ↓
UI

🧠 Remember

«"State" is a Compose state mechanism.»

«"StateFlow" is a Kotlin Flow state holder.»

---

15. 🌊 "collectAsStateWithLifecycle"

💡 Simple Answer

It collects a Flow/StateFlow as Compose state while respecting the Android lifecycle.

val uiState by viewModel.uiState
    .collectAsStateWithLifecycle()

Then:

when {
    uiState.isLoading -> Loading()
    uiState.error != null -> Error()
    else -> Content(uiState.data)
}

🧠 Remember

For Android UI collecting a Flow:

Flow
 ↓
collectAsStateWithLifecycle()
 ↓
Compose State
 ↓
UI

---

16. 🚀 What is "LaunchedEffect"?

💡 Simple Answer

"LaunchedEffect" runs a coroutine tied to the Composable lifecycle.

LaunchedEffect(Unit) {

    viewModel.loadData()
}

When the key changes, the previous effect is cancelled and a new one starts.

🏠 Example

Screen opens:

Screen enters Composition
        ↓
LaunchedEffect
        ↓
Load data

🧠 Remember

Use it for suspending work triggered by Composition.

---

17. 🔄 What is "SideEffect"?

💡 Simple Answer

"SideEffect" runs after a successful recomposition.

SideEffect {
    analytics.setScreen("Home")
}

Use it when you need to publish Compose state to something outside Compose.

🧠 Remember

Successful recomposition
        ↓
SideEffect

---

18. 🧹 What is "DisposableEffect"?

💡 Simple Answer

Use "DisposableEffect" when you need to set something up and clean it up.

DisposableEffect(Unit) {

    // Register observer

    onDispose {
        // Remove observer
    }
}

🏠 Real-Life Example

Entering a hotel room:

Enter → Register
Leave → Unregister

🧠 Remember

Setup
  ↓
DisposableEffect
  ↓
Cleanup with onDispose

---

19. 🔄 What is "rememberUpdatedState"?

💡 Simple Answer

It allows a long-running effect to use the latest value without restarting the effect.

Example:

val currentOnTimeout by rememberUpdatedState(onTimeout)

LaunchedEffect(Unit) {

    delay(3000)

    currentOnTimeout()
}

🧠 Remember

Use it when:

«The effect should continue running, but it should use the latest callback/value.»

---

20. ⚡ What is "derivedStateOf"?

💡 Simple Answer

"derivedStateOf" creates state derived from other state and helps avoid unnecessary updates when the derived result hasn't actually changed.

Example:

val firstItemVisible by remember {
    derivedStateOf {
        listState.firstVisibleItemIndex == 0
    }
}

🏠 Example

You have:

Scroll position changes:
0 → 1 → 2 → 3 → 4

But you only care:

At top?
YES / NO

"derivedStateOf" can avoid reacting to every raw scroll-position change when the derived result remains the same.

🧠 Remember

«Use "derivedStateOf" when you have frequently changing input state but only care about a derived result changing less often.»

---

21. 🌊 What is "snapshotFlow"?

💡 Simple Answer

"snapshotFlow" converts Compose snapshot state reads into a Flow.

Example:

LaunchedEffect(listState) {

    snapshotFlow {
        listState.firstVisibleItemIndex
    }.collect { index ->

        println("Visible index: $index")
    }
}

🧠 Remember

Compose State
     ↓
snapshotFlow
     ↓
Flow

Useful when Compose state needs to participate in coroutine/Flow processing.

---

22. 🌎 What is CompositionLocal?

💡 Simple Answer

"CompositionLocal" allows data to be available to Composables without explicitly passing it through every function parameter.

Example:

CompositionLocalProvider(
    LocalAppTheme provides theme
) {
    App()
}

A child can read it:

val theme = LocalAppTheme.current

🧠 Remember

Use it for values that are conceptually ambient/contextual, such as theme-related values.

Don't use it as a replacement for normal state passing everywhere.

---

23. 📜 What are LazyColumn Keys?

When displaying dynamic lists:

LazyColumn {

    items(
        items = users,
        key = { user -> user.id }
    ) {
        UserItem(it)
    }
}

The key helps Compose identify the identity of each item.

🧠 Remember

List item identity
        ↓
Stable key
        ↓
Better item tracking

---

24. 🔑 Why are stable keys important?

Imagine:

A
B
C

Now insert:

X
A
B
C

Without proper identity, Compose may have difficulty understanding that the existing items simply moved.

With stable keys:

A → id=1
B → id=2
C → id=3

Compose can track item identity.

🎯 Senior Follow-up

Stable keys become especially important when list items:

- Move
- Are inserted
- Are removed
- Maintain local state
- Contain expensive UI

---

25. 🔼 State Hoisting with ViewModel

A common architecture is:

             ViewModel
                 │
                 │ StateFlow
                 ▼
             Composable
                 │
                 │ Events
                 ▼
             ViewModel

Example:

data class LoginUiState(
    val username: String = "",
    val isLoading: Boolean = false
)

ViewModel:

class LoginViewModel : ViewModel() {

    private val _uiState =
        MutableStateFlow(LoginUiState())

    val uiState =
        _uiState.asStateFlow()

    fun login() {
        // Business logic
    }
}

UI:

@Composable
fun LoginScreen(
    viewModel: LoginViewModel
) {

    val state by viewModel.uiState
        .collectAsStateWithLifecycle()

    LoginContent(
        state = state,
        onLogin = viewModel::login
    )
}

---

26. 🧠 Compose and ViewModel

A common pattern:

Composable
     │
     │ events
     ▼
 ViewModel
     │
     │ state
     ▼
Composable

Example

@Composable
fun HomeScreen(
    viewModel: HomeViewModel
) {

    val state by viewModel.uiState
        .collectAsStateWithLifecycle()

    HomeContent(
        state = state,
        onRefresh = viewModel::refresh
    )
}

🧠 Remember

UI should primarily:

Render State
+
Send Events

ViewModel handles:

State management
Business orchestration
Events

---

27. 🧭 Compose Navigation

Navigation Compose allows navigation between destinations.

Example:

NavHost(
    navController = navController,
    startDestination = "home"
) {

    composable("home") {
        HomeScreen()
    }

    composable("details/{id}") { entry ->

        val id = entry.arguments
            ?.getString("id")

        DetailsScreen(id)
    }
}

🧠 Remember

Navigation should be treated as a UI concern, while the ViewModel/business layer shouldn't become tightly coupled to the "NavController".

A useful pattern is to pass navigation events/callbacks rather than passing "NavController" deep into reusable UI components.

---

28. 📣 Events in Compose

Instead of a child directly modifying ViewModel state:

Button(
    onClick = {
        viewModel.deleteUser()
    }
)

A reusable UI component can expose an event:

@Composable
fun UserItem(
    user: User,
    onDelete: () -> Unit
) {

    Button(
        onClick = onDelete
    ) {
        Text("Delete")
    }
}

🧠 Remember

UI Event
   ↓
Callback
   ↓
Screen / ViewModel

This improves reusability and testability.

---

29. 🔄 What is Unidirectional Data Flow?

UDF means data moves in one direction.

        State
          ↓
         UI
          ↓
        Event
          ↓
      ViewModel
          ↓
      New State
          ↓
         UI

Example

User clicks Like
      ↓
onLikeClicked()
      ↓
ViewModel
      ↓
isLiked = true
      ↓
StateFlow emits new state
      ↓
Compose recomposes
      ↓
Like icon changes

🧠 Remember

«State goes down. Events go up.»

---

30. 📦 What is UI State?

Instead of exposing many independent values:

val isLoading: Boolean
val users: List<User>
val error: String?

we can group them:

data class UserUiState(
    val isLoading: Boolean = false,
    val users: List<User> = emptyList(),
    val error: String? = null
)

Then:

ViewModel
   ↓
UserUiState
   ↓
Compose

🧠 Remember

A UI state object should represent:

«Everything the UI needs to render the current screen.»

---

🧠 Medium Level Quick Revision

    remember
      ↓
    Recomposition

    rememberSaveable
      ↓
    Saveable/restorable UI state

    State
      ↓
    Compose state

    StateFlow
      ↓
    Kotlin Flow state holder

    collectAsStateWithLifecycle()
      ↓
    Flow → Compose State

    LaunchedEffect
      ↓
    Coroutine triggered by Composition

    SideEffect
      ↓
    Runs after successful recomposition

    DisposableEffect
      ↓
    Setup + cleanup

    rememberUpdatedState
      ↓
    Latest value without restarting effect

    derivedStateOf
      ↓
    Derived state

    snapshotFlow
      ↓
    Compose State → Flow

    CompositionLocal
      ↓
    Ambient/contextual data

    LazyColumn key
      ↓
    Item identity

    ViewModel
      ↓
    Screen state + events

    UDF
     ↓
    State down + Events up

    UiState
     ↓
    Everything UI needs to render

---

🎯 Medium-Level Interview Pattern

When answering a Medium-level Compose question, use:

What is it?
    ↓
Why do we need it?
    ↓
Simple example
    ↓
When should we use it?
    ↓
What should we avoid?

This makes your answer clear and structured during an interview.
