
---
title: localStorage
description: how to use localStoarge in a project 
type: NOTE
year: 2026
tags: [frontend, browser, react, javascript]
---
##
![localstorage](./assets/EAED18F6-C078-487A-8E6D-3475D9FDA612.png)
## BASIC POINTS: 
- it is in the form of `key:value` pair 
- value: it is only text , (not other data like array or objects ), IF YOU have to store them then  use `JSON.stringify(object)`
- when you do `getItem` convert those (if you have stored object or array) back using `JSON.parse(key)`
- if `getItem` the item is not present it return `null`

## CORE IDEA OF localStorage IN ONE SENTENCE :
> “The website remembers something about you, without asking the server.”
This is what makes it different from a normal variable — a normal variable disappears when you close the page. localStorage does not

## EXAMPLE: 
```javascript
// User turns ON dark mode
localStorage.setItem("theme", "dark");

// Next day, user opens the website again...
let savedTheme = localStorage.getItem("theme");

if (savedTheme === "dark") {
    document.body.style.background = "black";
    document.body.style.color = "white";
}
```


## SAVING A DATA :
![](./assets/A80A1B25-E6DA-4056-BA8C-6BDDAE1DA07D.png)

## DELETING A DATA:
```javascript
localStorage.removeItem("username");// Remove one item
localStorage.clear();// Remove everything

```

### NOTE:
> Two keys with the same name can exist — but they won’t. The second one will silently destroy the first.

## example:
```javascript
localStorage.setItem("color", "blue");
localStorage.setItem("color", "red");  // same key!

console.log(localStorage.getItem("color")); // Output: "red"

```
- “blue” is gone forever. No error, no warning — it just gets replaced.