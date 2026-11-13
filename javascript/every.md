---

title: every method in javascript
description: notes about every method, chek out some method notes also 
type: NOTE
year: 2026
tags: [javascript, method, fundamentals]

---

## Key points:
- method used on array 
- Answers: “do ALL elements pass?”,  return `true` or `false`

### Example:
```javascript
const nums = [2, 4, 6, 8];

const allEven = nums.every(n => n % 2 === 0); 
//if all the element satisfy this condition it returns true

const allPositive = nums.every(n => n > 5);
// false  (2 and 4 fail)

```

> it is like AND gate