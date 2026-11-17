---

title: Prisma shape visualising 
description: A step by step on how to visualise the result of a prisma query
type: Note
Date: 2026-11-17
year: 2026
tags: [prisma, database]

---


## Model:
```prisma
model Order {
  id          Int      @id @default(autoincrement())
  name        String?
  status      String   @default("pending")
  items       OrderItem[]
  
}

model OrderItem {
  id         Int    @id @default(autoincrement())
  orderId    Int
  price      Int
  order      Order    @relation(fields: [orderId], references: [id])
 
}

```
- in the above model Order has 1:M relationship with OrderItem, 

## Run this query:
```
order.findFirst({
        where:{
            id:1
        },
        include:{
            items:true  
        }
    })

```

## Visualising the data:
- when the `where ` is met, prisma return the row in a object format :
```javascript
{id:1, name:"order1" ,status:"pending" }
```

- the include part means `give me all the rows related to the id`
- we know that when a condition is return the row in objet format, so in this case it is returning many object, so prisma put all of them in an array , 
- but since it is inside the 1st query , the array needs a key so `key`becomes the `relation name` which is `items`

## Final Shape:
```javascript
{
    id:1,
    name:"order1",
    status:"pending"
    items:[{id: ,orderId, price}, {id: ,orderId, price}, {id: ,orderId, price}]
}
```

## 1. What if no items exist?
- The items key still appears, but as an empty array — not null:
```javascript
{ id: 1, name: "order1", status: "pending", items: [] }
```
## 2. Nested includes follow the same mental model
- If OrderItem had its own relation (say, to a Product), you can nest:

```javascript
include: {
  items: {
    include: { product: true }
  }
}

```
- Each item object just grows a product: {} key — same rule, same shape.

```javascript
{ id: 1, name: "order1", status: "pending", items: [{id: ,orderId, price, product:[{}, {}, {}]}, {}, {}] }
```
- if you dont add `include` then produce will not be appear

> One row → one object. Many rows → array of objects. The key is always the relation field name.