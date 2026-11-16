---
title: How 2 different method of javascript is used as a method chaning 
decription: Each top heading represents a Method Chaining
type: NOTE
year: 2026
tags: [array, array method, methodchaning, javascript]

---


# Learning 1: 
```javascript
const scoredCombinations: { combination: string[]; penalty: number }[] = []

const tiedCombinations = scoredCombinations
        .filter((item) => item.penalty === minPenalty)
        .map((item) => item.combination)
```
- we can divided the code into 2 steps where next step depends onthe previous, 
### Part 1: the `filter` part 
```javascript
scoredCombinations.filter((item) => item.penalty === minPenalty)
```
- here , the code is doing : give me all the element in scoredCombination where each element penalty key value is equal to minPenalty 

### Part2: the `map` part
```javascript 
.map((item) => item.combination)
```
- the part 1 returns elements which satisfy the condition, we know that filter return an array , so here 
.map is also only works on array, so it automatically means this method is applying to the filtered array, 
- This is why i said , the next step is related to the previous one , 
```javascript
(item) => item.combination
```
- if you have read about `arrow function` , this code block means , return item.combination for each filtered array, 

#### So what is tiedCombination holding ???



# Learning 2: 
