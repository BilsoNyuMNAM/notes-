---
title: DOCKER BUILD ISSUE IN VIRTUAL MACHINE (docker compose up -d)
description: When you are starting containers in a virtual machine using docker compose or just docker you have see a compatability issue when trying to run the image 
type: NOTE
year: 2026
tags: [docker, docker-compose,virtual_machine, bug]
---

![Compose up issue](./assets/composeup.png)
 when you run the command :

 ```bash
    sudo docker compose up -d
 ```
you see the above issue !!! \
- Notice the error mentions platform: linux/amd64.
- Your Mac uses Apple Silicon (arm64), but your EC2 machine uses linux/amd64.

when you were creating the ec2 instance , you had selected `linux` as the operating system
![operating system](./assets/ec2opeartinsystem.png)

### How to fix this issue: 
- well you have to flag and tell docker which operating system and architecture to build for and it will fix this issue . 
`--platform linux/amd64`

```bash
docker build  --platform linux/amd64 -t  image_name <docker_file_path> .
```