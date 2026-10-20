---
title: A guide on how to install github plugin or skills in antigravity and opencode 
description: 
type: NOTE
year: 2026
tags: [ai-agents, how to, opencode, antigravity]
---


- to install ecc, skills for a specific project , at the project root run 
`npx ecc-universal install --profile minimal --target antigravity`
- It copies the skills, rules, and workflows into your project's .agents/ folder so Antigravity can discover and use them.

After you have run the command , if you dont `.agents` at the root , you will see  a folder now with the above skill installed for the project 


## opencode : 
in v2 , opencode , adding the plugin in 
`plugins` keys at opencode.json does not work anymore, you will error: `PLUGINS failed to load`

### INSTALLING ECC PLUGIN IN OPENCODE : 
ECC has a specific install path for OpenCode. Here’s what to do, step by step:
Step 1 — Clone the repo
```bash
git clone https://github.com/affaan-m/ECC.git
cd ECC
```
Step 2 — Build and install for OpenCode
```bash
npm install && npm run build:opencode && ./install.sh --profile full --target opencode --enable-hooks
```

> OpenCode V2 needs a proper plugin export (id + setup), not just raw files. So the npm run build:opencode step compiles the plugin into the correct V2 shape before installing.

After install, verify it worked:
```bash
opencode plugin list
```