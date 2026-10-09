---
title: Some issue i faced in the ci/cd when redeploying the `auto-deploy_vm` monorepo
description: Link to the repo: [https://github.com/BilsoNyuMNAM/auto-deploy_vm]
type: NOTE
year: 2026
tags: [ci-cd, how to, bug, virtual_machine, troubleshooting]
---

### Issue 1: The docker hub login issue :
- this happens because the access token you generated has insufficient permissions; 
![Permission issue](./assets/permission_issue.jpeg)
#### Fix: 
![fix](./assets/permissionfix.jpeg)


### Issue 2: The github action ubuntu machine cannot ssh into the virtual machine 
![](./assets/sshpermission.jpeg)
- when i copy the value of the private key i did not include the `BEGIN RSA PRIVATE KEY, BEGIN RSA END KEY `,  i just copied the values only 
### Fix: 
![Explaination 1](./assets/ssh1explaination1.jpeg)
![Explaination 2 ](./assets/sshexplaination2.jpeg)


### Issue 3: 
![](./assets/issue3.png)
- docker was not installed , the fix is to installed docker 
