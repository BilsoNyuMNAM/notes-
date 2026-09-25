---
title: UNDERSTANDING THE BENEFIT OF DOCKER THOUGH AN EXAMPLE
description: example explaination on why you should use docker
type: NOTE
year: 2026
tags: [docker, fundamentals]

---
# There are 2 way to run MongoDb
 1. using a connection string and running on MongoDb provider
 2. running it locally 

### Why we would run locally ? 
You are building and testing. You write bad code. It deletes all users.
If that happened on Atlas(MongoDb provider) — **real data is gone.**
With mongoDb running locally you can Break it. Reset it. Mess it up. Nobody cares.
Also , when there is no internet , you cannot use Atlas, but with running locally you can run it even without the internet 
![more on why you should run database locally:](https://studio3t.com/knowledge-base/articles/5-reasons-you-should-install-a-local-mongodb-database/)
---
### Why docker ? 
To run MongoDB locally, you'd normally have to **install it** on your computer.
The problem with installing:
- Different projects need different versions of MongoDb
- Hard to remove cleanly
- You have to configure so that MongoDb works,
- It also takes up a large amount of storage space on your computer
- Sits on your computer forever
---

**Docker lets you run MongoDB without installing it.** 

You just run 
```
docker run -p 27017:27017 mongo 

```
![more on the -p:](https://docs.docker.com/engine/network/port-publishing/)

MongoDb is started in a container , 


***See, there was no need for installing , confiuguration etc***
---
## Connecting mongoDb using MongoDb Compass 
 - When you are using database from a provider for your backend you will get the connection string , paste it in the env and use it , 
- But we already have mongoDb running in a container at `localhost:27017` , we just needs to connects to it,
- but we dont have a backend yet so we use mongoDb compass, here it will acts as a backend server, we paste the `mongodb://localhost:27017` and hit connect and we are connected to the mongoDb locally, without using it from the internet 
- when you have backend, use this url in the env it will connect to the mongoDb running locally provided the container is running  


