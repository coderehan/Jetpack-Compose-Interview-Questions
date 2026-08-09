<div align="center">

# 🚀 Jetpack Compose — Android Interview Preparation

**50 interview questions.** Simple words. One real-life example. One Kotlin snippet. No fluff.

![Compose](https://img.shields.io/badge/Jetpack-Compose-4285F4?logo=jetpackcompose&logoColor=white)
![Android](https://img.shields.io/badge/Android-Interview-3DDC84?logo=android&logoColor=white)
![Level](https://img.shields.io/badge/Level-Easy%20→%20Hard-orange)
![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen)

</div>

---

## 💡 Why this repo

Most Compose guides either dump the entire official documentation on you, or skip the "why" and just show code. This repo does neither:

> **Question → Simple-words answer → 1 real-life analogy → 1 runnable Kotlin snippet.**

That's it. Nothing to memorize, nothing to skim past.

---

## 🗂️ What's inside

| Level | File | Questions | Focus |
|:---:|---|:---:|---|
| 🟢 Easy | [`01-Easy.md`](01-Easy.md) | 12 | Composables, State, `remember`, Modifiers, layout basics |
| 🟡 Medium | [`02-Medium.md`](02-Medium.md) | 18 | Side effects (`LaunchedEffect`, `DisposableEffect`...), ViewModel, navigation, unidirectional data flow |
| 🔴 Hard | [`03-Hard.md`](03-Hard.md) | 20 | Recomposition internals, stability, performance, architecture |

---

## 📖 Sample format

Every question in this repo looks like this:

> **What is `derivedStateOf`?**
>
> Computes a value from other state, but only triggers recomposition when the derived result actually changes — not on every input change.
>
> ```kotlin
> val showButton by remember {
>     derivedStateOf { listState.firstVisibleItemIndex > 0 }
> }
> ```

---

## 🧭 How to use this repo

1. **Read** — question, answer, analogy.
2. **Type** the snippet yourself — don't copy-paste.
3. **Close the file** and explain it out loud, like an interviewer is listening.
4. Move to the next question.

Go in order: `Easy → Medium → Hard`. Each level builds on the last.

---

## 🤝 Contributing

Found a question that's still confusing, or want to add one? PRs are welcome — keep the same format: **simple words + real-life example + Kotlin snippet.**

---

<div align="center">

⭐ **If this helped you prepare, star the repo — it helps others find it too.**

</div>
