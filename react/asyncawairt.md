---

title: HOW TO USE ASYNC/AWAIT INSIDE USEEFFECT 
description: explanation with example on how to correctly use async/await inside useEffect, the 2 ways to do it, and why making the callback itself async causes problems 
type: NOTE
year: 2026
tags: [react, useEffect, async await, practical, fetch, callback function]
---
## The Rule

`useEffect` expects its callback to return **nothing** or a **cleanup function**.

```ts
useEffect(() => {
  // do something
  return () => { /* optional cleanup */ }
}, [])
```

---

## CALLBACK FUNCTION
callback simply means the function you give to another function.
### for example:
```javascript
useEffect(() => {
  console.log("Component rendered");
}, []);

// ()=>{} is the callback function
//why??
//Because you are giving the function to useEffect, and React will call it later.
```

## The Problem

An `async` function **always returns a Promise** — even without a `return` keyword.

So if you do this:

```ts
// ❌ WRONG
useEffect(async () => {
  const result = await fetch("http://localhost:8080")
  const data = await result.json()
  setTodos(data.alltodos)
}, [])
```

The callback returns a **Promise**. React does not expect that.\
React will **silently ignore** the Promise — but this can cause memory leaks or state updates after the component is gone.

---

## CORRECT WAY — Why This Works

```ts
// ✅ CORRECT
useEffect(() => {
  async function fetchAlltodos() {
    const result = await fetch("http://localhost:8080")
    const data = await result.json()
    setTodos(data.alltodos)
  }

  fetchAlltodos()
}, [])
```

Why does this work, even though `fetchAlltodos()` returns a Promise?

Because the **outer callback** does NOT use the `return` keyword.

### Rule 1 — A function returns nothing by default
```javascript
function example() {
  doSomething() // no "return" keyword
}
```
This function returns undefined. Not the result of doSomething().

### Rule 2 — async function always returns a Promise
```javascript
async function fetchAlltodos() { ... }

fetchAlltodos() // this gives you a Promise
```

### Now put it together:
```javascript
useEffect(() => {        // outer callback
  fetchAlltodos()       // Promise is created, but NOT returned
})
```
The outer callback calls fetchAlltodos() — yes, a Promise is created.

But the outer callback never says return.

So the outer callback returns undefined — which is exactly what React expects

```ts
useEffect(() => {       // outer callback — NOT async
  fetchAlltodos()      // Promise is created, but NOT returned
})
```

So the outer callback returns `undefined` — which is exactly what React expects. ✅

The Promise from `fetchAlltodos()` is created, but it is "floating" — the outer callback never passes it back to React.

---

## 2 Ways to Use Async Inside useEffect

### Way 1 — Define the async function inside the effect

```ts
useEffect(() => {
  async function fetchAlltodos() {
    const result = await fetch("http://localhost:8080")
    const data = await result.json()
    setTodos(data.alltodos)
  }

  fetchAlltodos()
}, [])
```

### Way 2 — Define the async function outside the effect, then call it inside

```ts
async function fetchAlltodos() {
  const result = await fetch("http://localhost:8080")
  const data = await result.json()
  setTodos(data.alltodos)
}

useEffect(() => {
  fetchAlltodos()  // just call it — no return keyword
}, [])
```

Both ways work. The outer callback stays non-async. React receives `undefined`. ✅

---

## Quick Summary

|  | Returns | React happy? |
| --- | --- | --- |
| `async` callback | Promise | ❌ No |
| Normal callback that calls async function | `undefined` | ✅ Yes |