---

title: some method in javascript
description: notes about some method, check out every method notes also 
type: NOTE
year: 2026
tags: [javascript, method, fundamentals]

---
## Key points:
- it a method used on array 
- Answers: “does at least one element pass?” , return true or false
- Also stops early once it finds one match.

### Example 1:
```javascript
const nums = [1, 3, 5, 6, 7];

const hasEven = nums.some(n => n % 2 === 0); //if any element satisfy this condition, it returns true 


const hasNegative = nums.some(n => n < 0);
// false

```

### Example 2:
```javascript
const products = [
  { name: "Shoes", price: 120, inStock: true },
  { name: "Shirt", price: 40,  inStock: false },
  { name: "Hat",   price: 25,  inStock: true },
  { name: "Pants", price: 80,  inStock: true },
];
console.log(products.some(p => !p.inStock));  // Are any products out of stock?

```

> It is like OR gate