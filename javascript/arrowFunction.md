---
title: Arrow function
decription: arrow function basics 
type: NOTE
year: 2026
tags: [array, fundamentals, map,filter,  javascript]

---

---

## 1. What is a function?
A function takes something in, does work, and gives something back.

function double(x) {
  return x * 2
}

Give it x → it gives back x * 2

---

## 2. Arrow function is the same thing, written shorter

(x) => {
  return x * 2
}

That's it. Same work, less writing.

---

## 3. Two ways to write the body

Way 1 — With {} (you must write return)
(x) => { return x * 2 }

Way 2 — Without {} (return happens automatically)
(x) => x * 2

Simple rule: No {} = value returns by itself. With {} = you must type return.

---

## 4. Returning an Object — The Tricky Part

Problem: After =>, JavaScript always reads { as "start of code block", NOT as an object.

- JS thinks this is a code block — returns undefined
() => { id: 1 }

-  Wrap in () to tell JS it's an object
() => ({ id: 1 })

### Why does () fix it?
The ( tells JavaScript — "what comes next is a value, not a code block"
so it reads { id: 1 } as an object.

See the difference clearly:

const a = () => { id: 1 }              // undefined  (code block)
const b = () => ({ id: 1 })            // { id: 1 }  (object)
const c = () => { return { id: 1 } }   // { id: 1 }  (explicit return)  > remember the syntax of using return 

Quick rule:

| What you want to return   | How to write it              |
|---------------------------|------------------------------|
| Number / String / Array   | () => x + 1                  |
| Object                    | () => ({ key: value })       |
| Multiple lines            | () => { ...; return value }  |

---

## 5. Parameters

// No params — () is required
() => 1

// One param — () is optional
x => x

// Two or more params — () is required
(a, b) => a + b

---

## 6. Common Mistakes

// WRONG — can't use return without {}
x => return x * 2

// WRONG — object without ()
x => { a: 1 }

// WRONG — if is not an expression
if (a) ? b : c

// CORRECT — use ternary instead of if
a ? b : c

---

## 7. Practice — Rewrite as arrow functions

// 1. function add(a, b) { return a + b }
// 2. function square(n) { return n * n }
// 3. function makeObj(id) { return { id: id } }
// 4. function getWeight(s) { return s.weight }
// 5. function first(arr) { return arr[0] }
// 6. function nothing() { return null }

---

## 8. Answers

// 1.
(a, b) => a + b

// 2.
n => n * n

// 3.
id => ({ id })

// 4.
s => s.weight

// 5.
arr => arr[0]

// 6.
() => null

---

## 9. One Question to Always Ask Yourself

"What does this return?"

If you're not sure — your body form is probably wrong. Test it.