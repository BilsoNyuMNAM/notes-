---
title: How to Create a Toast Box Notification in React
description: A first-principles breakdown of how to build a toast notification that appears on an event and auto-disappears after a set time, using only useState and setTimeout
type: NOTE
year: 2026
tags: [react, useState, setTimeout, conditional-rendering, fundamentals, how to ]
---

# How to Create a Toast Box Notification in React

A toast is a small box that pops up (usually at the corner of the screen), shows a message, and disappears on its own after a few seconds.

No libraries. No magic. Just 3 building blocks wired together.

---

## Things We 
1. the toast box 
2. useState
3. setTimeout

### 1. The Toast Box (JSX + CSS)

This is the visual part — a `<div>` that is **styled to float** over everything else using `position: fixed`.

```jsx
<div style={{
    position: "fixed",
    bottom: "20px",
    right: "20px",
    backgroundColor: "#333",
    color: "white",
    padding: "12px 20px",
    borderRadius: "8px",
    boxShadow: "0 4px 12px rgba(0,0,0,0.3)",
    zIndex: 1000,
    fontSize: "14px"
}}>
    Todos saved successfully! ✅
</div>
```

By itself, this div just sits on the screen forever. It doesn't know when to show or hide. It needs **state** to control that.

---

### 2. useState — To Control Visibility

We need a boolean that tells React: "should the toast be on screen right now?"

```jsx
const [showToast, setShowToast] = useState(false)
```

- `false` → toast is hidden
- `true` → toast is visible

We also need a second state to hold **what message** the toast should display, so we can reuse the same toast for different situations (success, error, etc.):

```jsx
const [toastMessage, setToastMessage] = useState("")
```

Now the toast box can read from these states:

```jsx
{showToast && (
    <div style={{ /* ...toast styles... */ }}>
        {toastMessage}
    </div>
)}
```

> **Why `{showToast && (...)}`?**
> This is conditional rendering. When `showToast` is `false`, the `&&` short-circuits and React renders **nothing**. When it's `true`, React renders the div.

---

### 3. setTimeout — To Make It Disappear

`setTimeout` is a plain JavaScript function. You give it a function and a delay (in milliseconds), and it runs that function **once** after the delay.

```javascript
setTimeout(() => {
    // this runs after 3000ms (3 seconds)
}, 3000)
```

We use it to flip `showToast` back to `false` after a few seconds:

```javascript
setTimeout(() => {
    setShowToast(false)
}, 3000)
```

This is what makes the toast **auto-disappear**. Without this, the toast would stay forever once shown.

---

## The Order of Interconnection

Here is the exact sequence that makes a toast appear and then disappear. Every step depends on the one before it.

### Step-by-step flow

```
User clicks "Save todos"
        │
        ▼
 ┌─────────────────────────────┐
 │  1. Set the message         │
 │  setToastMessage("Saved!✅") │
 └──────────┬──────────────────┘
            │
            ▼
 ┌─────────────────────────────┐
 │  2. Show the toast          │
 │  setShowToast(true)         │
 └──────────┬──────────────────┘
            │
            ▼
 ┌─────────────────────────────┐
 │  3. Schedule the hide       │
 │  setTimeout(() => {         │
 │    setShowToast(false)      │
 │  }, 3000)                   │
 └──────────┬──────────────────┘
            │
            ▼
    React re-renders:
    showToast is true → toast div appears on screen
            │
            ▼
    ... 3 seconds pass ...
            │
            ▼
    setTimeout fires:
    setShowToast(false)
            │
            ▼
    React re-renders:
    showToast is false → toast div is removed from screen
```

### The 3 steps in code

All 3 steps happen **together** inside whatever event triggers the toast (like a successful API call):

```jsx
// after a successful save:
setToastMessage("Todos saved successfully! ✅")  // Step 1: what to say
setShowToast(true)                                // Step 2: show it
setTimeout(() => {                                // Step 3: schedule hiding
    setShowToast(false)
}, 3000)
```

### Why this order matters

| Step | What it does | What happens if you skip it |
|---|---|---|
| `setToastMessage(...)` | Sets the text inside the toast | Toast shows but with old/empty message |
| `setShowToast(true)` | Makes React render the toast div | Toast never appears on screen |
| `setTimeout(...)` | Schedules `setShowToast(false)` after 3s | Toast appears but **never disappears** |

All three are needed. Remove any one and the toast is broken.

---

## Full Working Example

Putting it all together in a component:

```jsx
import { useState } from 'react'

function App() {
  const [showToast, setShowToast] = useState(false)
  const [toastMessage, setToastMessage] = useState("")

  const handleSave = () => {
    // ... do your save logic ...

    // trigger the toast
    setToastMessage("Todos saved successfully! ✅")
    setShowToast(true)
    setTimeout(() => {
        setShowToast(false)
    }, 3000)
  }

  return (
    <div>
      <button onClick={handleSave}>Save todos</button>

      {/* Toast — only renders when showToast is true */}
      {showToast && (
        <div style={{
            position: "fixed",
            bottom: "20px",
            right: "20px",
            backgroundColor: "#333",
            color: "white",
            padding: "12px 20px",
            borderRadius: "8px",
            boxShadow: "0 4px 12px rgba(0,0,0,0.3)",
            zIndex: 1000,
            fontSize: "14px"
        }}>
            {toastMessage}
        </div>
      )}
    </div>
  )
}
```

---

## Quick Summary

| Piece | Role |
|---|---|
| Toast `<div>` | The visual box (CSS makes it float in the corner) |
| `useState(false)` | Controls **if** the toast is on screen |
| `useState("")` | Controls **what** the toast says |
| `setTimeout` | Automatically hides the toast after a delay |
| `{showToast && ...}` | Conditional rendering — connects state to the DOM |

Three pieces. One pattern. That's a toast.
