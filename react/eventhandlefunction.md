---
title: React Event Handler — When to Add "()" and When Not To
description: A quick note to remember the difference between passing a function and calling a function inside React event handlers like onClick
type: NOTE
year: 2026
tags: [react, javascript, event-handler, onclick, common-mistakes, fundamentals]
---

# React Event Handler — When to Add `()` and When Not To

## The Core Rule

> **`onClick` needs a function — not the result of calling a function.**

---

## The 3 Patterns

###  Pattern 1 — Pass the function directly
```jsx
onClick={handleAddTodo}
```
- React holds the function and **calls it when user clicks**
- Simple and clean — use this when you have **no arguments** to pass

#### One common mistake: 

```javascript
onClick={handleAddTodo()}
- You are calling the function immediately — it runs when the page loads, not when the user clicks.

```

---

###  Pattern 2 — Wrap in arrow function and call it
```jsx
onClick={() => { handleAddTodo() }}
// or shorter:
onClick={() => handleAddTodo()}
```
- Arrow function runs on click, then **calls** `handleAddTodo()`
- Use this when you need to **pass arguments** or run extra logic

```jsx
// Example with argument:
onClick={() => handleAddTodo(id)}
```

---

###  Pattern 3 — Wrap in arrow function but forget the `()`
```jsx
onClick={() => { handleAddTodo }}
```
- Arrow function runs on click, but `handleAddTodo` is **just mentioned — never called**
- **Nothing happens.** This is the most common mistake.


It is like writing: 

```javascript
function() {
  handleAddTodo  // just a name, not called
}
```

---

## Quick Comparison Table

| Pattern | Runs on page load? | Calls the function on click? | Use it? |
|---|---|---|---|
| `onClick={handleAddTodo}` |  No |  Yes |  Yes |
| `onClick={handleAddTodo()}` |  Yes (bad!) |  Not on click |  No |
| `onClick={() => handleAddTodo()}` |  No |  Yes |  Yes |
| `onClick={() => { handleAddTodo }}` |  No |  Never called |  No |

---


Inside `onClick`, you always want a **function to be called later**, not a result now.
## Meaning of not a result right now 
- Every function in JavaScript returns something when you call it. That "something" is called the result.
```javascript
function handleAddTodo() {
  console.log("todo added!");
  // returns nothing → so the result is: undefined
}
```
- When you write handleAddTodo() — you are calling it right now, and JavaScript replaces it with the result:
```javascript
onClick={handleAddTodo()}
// JavaScript sees this as:
onClick={undefined}   // ← the result of calling the function

```
So onClick receives undefined — not a function. It has nothing to call when you click.

When you write handleAddTodo — you are giving the function itself:
```javascript
onClick={handleAddTodo}
// JavaScript sees this as:
onClick={function() { console.log("todo added!") }}  // ← the function itself
```
So onClick receives the actual function and calls it later when you click



---

## When to Use Which

| Situation | Use |
|---|---|
| No arguments needed | `onClick={handleAddTodo}` |
| Need to pass arguments | `onClick={() => handleAddTodo(id)}` |
| Need to run multiple things | `onClick={() => { doThis(); doThat(); }}` |

---

## The Mistake to Avoid

```jsx
// ❌ WRONG — handleAddTodo is never called
onClick={() => { handleAddTodo }}

// ✅ RIGHT — handleAddTodo is called on click
onClick={() => { handleAddTodo() }}
```

The only difference is `()` — but it changes everything.