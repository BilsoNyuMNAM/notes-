---
title: How to add secrets to a ci/cd workflows such as passowrd, datbase url , ssh private password etc
description: 
type: NOTE
year: 2026
tags: [ci-cd, how to, ]
---

### There are 2 ways to add secrets to a ci/cd file , \
#### 1. Hardcoding the values inside the file itself . 
You just hard coded the values in  you `.yml` file 

#### 2. Using it dynamically through code 
![](./assets/05A14E3E-B956-4D08-87B4-DFF5C2BC922A_1_105_c.jpeg)
![](./assets/2791C8D1-5E95-47E3-9436-3EE361F56647.png)
You add the secrets as a `New repository secret` and then import it in the file 

##### The syntax to import the secrets is: 
```bash
${{ secrets.YOUR_SECRET_NAME }}
```
###### For example, from the above image if you want to import `DOCKER_PASSWORD`it would be :
```
${ secrets.DOCKER_PASSWORD }
```

### Key things to remember:

- The secret name in GitHub must match exactly what you write in secrets.YOUR_NAME
- GitHub will hide the value in logs automatically — it will show as ***
- Secrets are only available to workflows in the same repo