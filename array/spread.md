---
title: HOW SPREAD OPERATOR WORKS 
description: explaination with example on how spread operator works 
type: NOTE
year: 2026
tags: [array, array method, spread operator]
---

# What does spread operator(...) means ? 
it means "copy everything from this array/object and put it in [the new object/array]"

## How does it work ?
Think of it like **unpacking a box** 📦

- `...oldList` opens the array and **pours out** every item
- Those items are now free to sit **next to** new items
- Everything is then packed into a **new array**

```javascript
const oldList = ["pen", "book"]
//                 ↑       ↑
//            item 0    item 1

const newList = [...oldList, "pencil"]
//               ↑             ↑
//         "pen", "book"    new item

// React does this internally:
// step 1 — open oldList  →  "pen", "book"
// step 2 — add "pencil"  →  "pen", "book", "pencil"
// step 3 — wrap in []    →  ["pen", "book", "pencil"]
```

> ⚠️ It does NOT change `oldList` — it always creates a **brand new array**

---

## Why `[...]` for arrays ?

Because `[]` is how you **create an array** in JavaScript.
The `...` just fills it with items from another array.

```javascript
// [] = create a new array
// ... = pour items into it

const newList = [...oldList]   // ✅ array  → use []
const newObj  = {...oldObj}    // ✅ object → use {}
```

| What you are copying | Syntax |
|---|---|
| Array | `[...oldArray]` |
| Object | `{...oldObject}` |

---

## For example: 
```javascript
const oldList = ["pen", "book"]
const newList = [...oldList, "pencil"]
console.log(newList) // ["pen", "book", "pencil"]
```
"copy everything from `oldList` array and put it in `newList`"

---

## What if the array is an array of objects ?

Works the **same way** — each object is copied into the new array.

```javascript
const todos = [
  { task: "Buy milk", check: false },
  { task: "Walk dog", check: false }
]

const newTodos = [...todos, { task: "Read book", check: false }]

// newTodos = [
//   { task: "Buy milk",  check: false },
//   { task: "Walk dog",  check: false },
//   { task: "Read book", check: false }  ← new object added
// ]
```

This is exactly what happens in your **todo app**:

```javascript
setTodos((prev) => [...prev, { task: input, check: false }])
//                    ↑              ↑
//              old todo items    new todo added at end
```

> ⚠️ Note: The objects inside are **not deep copied** — they are references.
> For a simple todo app this is fine.

---

## How to add new things after spreading the array

Just write them after the spread, separated by a comma.

### 1. Add at the End
```javascript 
const oldList = ["pen", "book"]
const newList = [...oldList, "pencil"]
// ["pen", "book", "pencil"]  ← pencil added at end
```

### 2. Add at the Beginning
```javascript
const newList = ["pencil", ...oldList]
// ["pencil", "pen", "book"]  ← pencil added at start
```

### 3. Add an Object (like in your todo app)
```javascript
const newList = [...oldList, { task: "Buy milk", check: false }]
// ["pen", "book", { task: "Buy milk", check: false }]
```

---

## Simple Rule to Remember
```
[  ...oldList,  newThing  ]
      ↑             ↑
  old items      new item
```