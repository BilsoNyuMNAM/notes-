---

title: TypeScript — Record<KeyType, ValueType>
description: A clean way to type an object where every key and every value follow the same shape
type: NOTE
year: 2026
tags: [typescript, types, objects, beginner]

---


# Record<KeyType, ValueType>

## What is it?

A `Record` is a TypeScript type for an object where:
- every **key** is the same type
- every **value** is the same type

Think of it like a **register at a hotel**.
Every room has a number (key), and every room holds a guest (value).
The shape is always the same — just the data changes.

```typescript
Record<KeyType, ValueType>
```


## Examples

---

### Example 1 — Month number to list of exercises

Each month (1–12) holds a list of exercises planned for that month.

```typescript
const monthlyPlan: Record<number, string[]> = {
  1: ["squat", "bench press"],
  2: ["deadlift", "row"],
  3: [],
}
```

- Key → `number` (the month)
- Value → `string[]` (list of exercise names)

---

### Example 2 — Username to score

Each player has a username, and each username maps to their score.

```typescript
const leaderboard: Record<string, number> = {
  ali: 1200,
  ben: 950,
  cara: 1100,
}
```

- Key → `string` (username)
- Value → `number` (score)

---

### Example 3 — Status code to message

Each HTTP status code maps to a human-readable message.

```typescript
const statusMessages: Record<number, string> = {
  200: "OK",
  404: "Not Found",
  500: "Internal Server Error",
}
```

- Key → `number` (status code)
- Value → `string` (message)

---

## The analogy that preserves the most meaning

> A `Record` is like a **dictionary**.
> Every word (key) has exactly one definition (value).
> All words are strings. All definitions are strings.
> The shape never changes — only the content does.

```typescript
const dictionary: Record<string, string> = {
  "hypertrophy": "the growth of muscle through exercise",
  "fatigue":     "tiredness caused by hard training",
}
```

This is the best analogy because:
- A dictionary enforces one key → one value, just like `Record`
- All keys are the same type (words)
- All values are the same type (definitions)
- You look something up by its key and get back its value


## Quick syntax reminder

```typescript
// object where keys are numbers and values are arrays of strings
Record<number, string[]>

// object where keys are strings and values are numbers
Record<string, number>

// object where keys are numbers and values are custom objects
Record<number, Exercise[]>
```