# 🟢 Jetpack Compose Interview Questions — Easy Level

A practical Jetpack Compose interview preparation guide for Android Engineers.

The goal is not to memorize every Compose API.

The goal is to understand:

How Compose works
       ↓
How State works
       ↓
How Recomposition works
       ↓
How Side Effects work
       ↓
How Compose communicates with ViewModel
       ↓
How to build performant & maintainable UI

---

📚 Topics

1. What is Jetpack Compose?
2. What is a Composable function?
3. What is Composition?
4. What is Recomposition?
5. What is State in Compose?
6. "remember"
7. "mutableStateOf"
8. State Hoisting
9. Modifier
10. Column, Row and Box
11. LazyColumn vs Column
12. Why should Composables be side-effect free?

---

1. 🧩 What is Jetpack Compose?

💡 Simple Answer

Jetpack Compose is Android's modern declarative UI toolkit.

Instead of telling Android:

«"Find this TextView and change its text."»

we describe:

«"If the state is X, show this UI."»

Compose then updates the UI when the state changes.

🏠 Real-Life Example

Think about a digital counter.

Count = 0

UI:
Count: 0

User clicks the button:

Count = 1

UI:
Count: 1

We change the state, and Compose updates the UI.

💻 Example

@Composable
fun Counter(count: Int) {

    Text(
        text = "Count: $count"
    )
}

🧠 Remember

State
  ↓
UI

Compose is declarative.

---

2. 🧩 What is a Composable Function?

💡 Simple Answer

A Composable function is a function that can describe UI.

It is marked with:

@Composable

💻 Example

@Composable
fun WelcomeScreen() {

    Column {
        Text("Hello")
        Button(
            onClick = {}
        ) {
            Text("Login")
        }
    }
}

🧠 Remember

Composable functions:

- Describe UI
- Can call other Composable functions
- Can be recomposed
- Should ideally be side-effect free

---

3. 🧩 What is Composition?

💡 Simple Answer

Composition is the process where Compose executes Composable functions and builds the UI representation.

Think:

Composable functions
        ↓
Composition
        ↓
UI

🏠 Real-Life Example

You give a restaurant your order.

Order
 ↓
Kitchen processes order
 ↓
Food is prepared

Similarly:

Composable
 ↓
Compose processes it
 ↓
UI is produced

🧠 Remember

Composition = building the UI from Composables.

---

4. 🔄 What is Recomposition?

💡 Simple Answer

Recomposition means Compose runs affected Composable functions again when the state they read changes.

It does not mean the entire application is redrawn.

💻 Example

@Composable
fun Counter() {

    var count by remember {
        mutableStateOf(0)
    }

    Button(
        onClick = {
            count++
        }
    ) {
        Text("Count: $count")
    }
}

When:

count = 0

and then:

count = 1

Compose recomposes the UI that depends on "count".

🧠 Remember

State changes
      ↓
Affected UI recomposes

🎯 Senior Follow-up

Does recomposition redraw the entire UI?

No.

Compose tracks state reads and can recompose the parts affected by changed state.

---

5. 🧠 What is State in Compose?

💡 Simple Answer

State is data that can change over time and affects the UI.

Examples:

Loading
User name
Selected tab
Counter value
Search text
Login status

💻 Example

var count by remember {
    mutableStateOf(0)
}

When "count" changes, Compose knows that UI reading "count" may need recomposition.

🧠 Remember

State changes
     ↓
UI may change

---

6. 🧠 What is "remember"?

💡 Simple Answer

"remember" stores a value across recompositions.

Without "remember", a normal local variable can be recreated when the Composable runs again.

Example

var count by remember {
    mutableStateOf(0)
}

The value survives recomposition.

🧠 Remember

remember
   ↓
Keep value across recompositions

---

7. 🧠 What is "mutableStateOf"?

💡 Simple Answer

"mutableStateOf()" creates observable Compose state.

val count = mutableStateOf(0)

When the value changes, Compose can detect it.

Usually we write:

var count by remember {
    mutableStateOf(0)
}

🧠 Remember

mutableStateOf
      ↓
Observable Compose state

---

8. 🔼 What is State Hoisting?

💡 Simple Answer

State hoisting means moving state to the caller so the Composable becomes reusable and easier to test.

Instead of:

@Composable
fun SearchBox() {

    var text by remember {
        mutableStateOf("")
    }
}

we can do:

@Composable
fun SearchBox(
    text: String,
    onTextChange: (String) -> Unit
) {

    TextField(
        value = text,
        onValueChange = onTextChange
    )
}

🏠 Real-Life Example

Instead of the child keeping the family budget, the parent manages the budget and tells the child what it is.

🧠 Remember

State belongs to the lowest common owner
that needs it.

---

9. 🛠️ What is Modifier?

💡 Simple Answer

"Modifier" is used to configure or decorate Composables.

Examples:

Padding
Size
Background
Click
FillMaxWidth
Alignment

💻 Example

Text(
    text = "Hello",
    modifier = Modifier
        .fillMaxWidth()
        .padding(16.dp)
)

🧠 Remember

Modifier describes how a UI element should behave or look.

---

10. 📐 Column, Row and Box

Column

Places children vertically.

Column {
    Text("A")
    Text("B")
}

Row

Places children horizontally.

Row {
    Text("A")
    Text("B")
}

Box

Places children on top of each other.

Box {
    Image(...)
    Text("Hello")
}

🧠 Remember

Column → Vertical
Row    → Horizontal
Box    → Stack / Overlay

---

11. 📜 LazyColumn vs Column

Column

Creates all children.

Column {
    items.forEach {
        Text(it.name)
    }
}

LazyColumn

Creates items lazily as required.

LazyColumn {
    items(items) {
        Text(it.name)
    }
}

🧠 Remember

For a large list:

Large list → LazyColumn
Small fixed content → Column

---

12. ⚠️ Why should Composables be side-effect free?

A Composable can be executed many times due to recomposition.

Therefore, don't do things like:

@Composable
fun Screen() {

    database.save(...)
}

because this could execute unexpectedly.

Instead use appropriate side-effect APIs:

LaunchedEffect
SideEffect
DisposableEffect

🧠 Remember

«Composable = describe UI»

«Side effect = use the correct Effect API»

---

🧠 Easy Level Quick Revision

Compose
   ↓
Declarative UI

Composable
   ↓
Function that describes UI

Composition
   ↓
Build UI

Recomposition
   ↓
Update affected UI when state changes

State
   ↓
Data that can change

remember
   ↓
Keep value across recomposition

mutableStateOf
   ↓
Observable Compose state

State Hoisting
   ↓
Move state to appropriate owner

Modifier
   ↓
Configure UI

Column
   ↓
Vertical

Row
   ↓
Horizontal

Box
   ↓
Overlay

LazyColumn
   ↓
Large/lazy lists

Effects
   ↓
Handle side effects correctly
