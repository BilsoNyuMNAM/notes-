---
title: DOCKER COMPOSE & PRISMA COMMON MISTAKES AND BUG LOG
description: Troubleshooting guide and bug log for common mistakes encountered when containerizing a Node.js backend with PostgreSQL and Prisma via Docker Compose.
type: NOTE
year: 2026
tags: [docker, docker-compose, prisma, postgres, troubleshooting, bugs, practical]
---

This document logs all the errors, configuration bugs, and misunderstandings encountered while setting up a Node.js + Prisma + PostgreSQL project using Docker Compose, along with their root causes, **Wrong Code**, and **Fixed Code**.

---

## Quick Navigation

1. [Bug 1: Docker Compose Syntax Error — Ports Must Be an Array](#1-bug-1-docker-compose-syntax-error--ports-must-be-an-array)
2. [Bug 2: Container Freezes on Startup Due to `prisma migrate dev`](#2-bug-2-container-freezes-on-startup-due-to-prisma-migrate-dev)
3. [Bug 3: Express Listening on `127.0.0.1` Instead of `0.0.0.0`](#3-bug-3-express-listening-on-127001-instead-of-0000)
4. [Bug 4: Dockerfile Build Fails Due to Order of `prisma generate` and `build`](#4-bug-4-dockerfile-build-fails-due-to-order-of-prisma-generate-and-build)
5. [Bug 5: Hostname Confusion in `DATABASE_URL` (Host vs Container)](#5-bug-5-hostname-confusion-in-database_url-host-vs-container)

---

## 1. Bug 1: Docker Compose Syntax Error — Ports Must Be an Array

### The Command Run
```bash
docker compose up
```

### The Terminal Error Output
```text
validating /Users/bilsonyumnam/Desktop/harkirat/Docker/week-27-docker-compose/docker-compose.yml: services.application.ports must be a array
```

### Where the Issue Arose in `docker-compose.yml`
When running `docker compose up`, Docker Compose validates the `.yml` schema before creating any networks or containers.

Look at how `ports` was defined in both services:

```yaml
services:
  postgres:
    image: postgres
    environment:
      POSTGRES_PASSWORD: userpassword
    ports:
      - "5432:5432"       # ✅ Correct: has a hyphen '-', so it's a list/array

  application:
    build: .
    environment:
      DATABASE_URL: postgresql://postgres:userpassword@postgres:5432/postgres
    ports:
      "3000:3000"         # ❌ WRONG: Missing hyphen '-'! Treated as a string, not an array
    depends_on:
      - postgres
```

### Why it Happened
In YAML:
- `ports:` is expected by the Docker Compose specification to be a **list / array of port mappings**.
- A list item in YAML **must start with a hyphen (`-`)**.
- In `postgres:`, `- "5432:5432"` was correctly parsed as `["5432:5432"]`.
- In `application:`, `"3000:3000"` lacked the `-`, so YAML parsed it as a plain key-value scalar string (`ports: "3000:3000"`). Docker Compose rejected it during schema validation.

### ❌ Wrong Code (`docker-compose.yml`)
```yaml
  application:
    build: .
    environment:
      DATABASE_URL: postgresql://postgres:userpassword@postgres:5432/postgres
    ports:
      "3000:3000"     # ❌ Missing hyphen '-'
    depends_on:
      - postgres
```

### ✅ Fixed Code (`docker-compose.yml`)
```yaml
  application:
    build: .
    environment:
      DATABASE_URL: postgresql://postgres:userpassword@postgres:5432/postgres
    ports:
      - "3000:3000"   # ✅ With hyphen '-', parsed as array item
    depends_on:
      - postgres
```

---

## 2. Bug 2: Container Freezes on Startup Due to `prisma migrate dev`

### The Error
The container status was `Up`, and port `3000` was published, but visiting `http://localhost:3000` in the browser resulted in:
```text
This site can't be reached (ERR_CONNECTION_REFUSED)
```

### Why it Happened
The package script was:
```bash
npx prisma migrate dev && node dist/index.js
```
The project had no `prisma/migrations/` folder. When `prisma migrate dev` runs without existing migrations, it pauses and prompts interactively:
```text
Enter a name for the new migration:
```
Because the Docker container ran in non-interactive background mode, nobody could type an answer. The command hung indefinitely. The `&&` prevented `node dist/index.js` from ever executing. The container was alive, but Express never started.

### ❌ Wrong Code (`package.json`)
```json
{
  "scripts": {
    "dev:docker": "npx prisma migrate dev && node dist/index.js"
  }
}
```

### ✅ Fixed Code (`package.json`)
```json
{
  "scripts": {
    "dev:docker": "npx prisma db push && node dist/index.js"
  }
}
```

> **Why this fixes it:** `prisma db push` synchronizes the database directly from `schema.prisma`. It requires no migration files, asks no interactive prompts, and immediately hands off execution to `node dist/index.js`.

---

## 3. Bug 3: Express Listening on `127.0.0.1` Instead of `0.0.0.0`

### The Error
The Express server starts successfully inside the container, but opening `http://localhost:3000` on the Mac browser fails with connection refused.

### Why it Happened
By default (or when explicitly specified), servers often bind to `127.0.0.1` (the container's private loopback). 
- `127.0.0.1` only accepts requests that originate from **inside** the container.
- Docker's port forwarding hits the container from the outside host network.
- To accept forwarded traffic, Express must listen on `0.0.0.0` (all network interfaces).

### ❌ Wrong Code (`src/index.ts`)
```typescript
// Binds only to internal container loopback
app.listen(3000, () => {
    console.log("Listening on port 3000");
});
```

### ✅ Fixed Code (`src/index.ts`)
```typescript
// Explicitly binds to all network interfaces
app.listen(3000, "0.0.0.0", () => {
    console.log("Application listening on port 3000");
});
```

---

## 4. Bug 4: Dockerfile Build Fails Due to Order of `prisma generate` and `build`

### The Error
During `docker build`:
```text
error TS2307: Cannot find module '@prisma/client' or its corresponding type declarations.
```

### Why it Happened
TypeScript compiling (`tsc -b`) needs the generated types from `@prisma/client` to validate your database queries (`prismaClient.user.findMany()`). If `npm run build` runs before `npx prisma generate`, TypeScript cannot find the model types.

### ❌ Wrong Code (`Dockerfile`)
```dockerfile
COPY . .

RUN npm run build          # ❌ Fails! Prisma client not generated yet
RUN npx prisma generate
```

### ✅ Fixed Code (`Dockerfile`)
```dockerfile
COPY . .

RUN npx prisma generate    # ✅ Generates types in node_modules/@prisma/client
RUN npm run build          # ✅ TypeScript compiles cleanly
```

---

## 5. Bug 5: Hostname Confusion in `DATABASE_URL` (Host vs Container)

### The Error
- Inside the app container: `getaddrinfo ENOTFOUND localhost` or connection refused to PostgreSQL.
- OR on the Mac host: `getaddrinfo ENOTFOUND postgres` when trying to run migrations.

### Why it Happened
The hostname in `DATABASE_URL` depends entirely on **where the code is running**:

```
+-------------------------------------------------------------------------+
| Mac Host Terminal                                                       |
| Connect via: localhost:5432                                             |
|                                                                         |
|   +----------------------------+     +----------------------------+     |
|   | Container: application     |     | Container: postgres        |     |
|   |                            |     |                            |     |
|   | Connect via: postgres:5432 |────>| Port 5432                  |     |
|   +----------------------------+     +----------------------------+     |
+-------------------------------------------------------------------------+
```

1. **Inside the App Container:** It cannot reach Postgres via `localhost` (that refers to itself). It must reach Postgres via Docker's internal DNS using the service name: `postgres`.
2. **On the Mac Terminal:** Your Mac is outside the Docker network. It connects via the port published by Docker Compose: `localhost:5432`.

### ❌ Wrong Code (Inside `docker-compose.yml` for the app)
```yaml
environment:
  DATABASE_URL: "postgresql://postgres:userpassword@localhost:5432/postgres" # ❌ Points to application container, not postgres container
```

### ✅ Fixed Code (Inside `docker-compose.yml` for the app)
```yaml
environment:
  DATABASE_URL: "postgresql://postgres:userpassword@postgres:5432/postgres"  # ✅ Uses service name
```

### ❌ Wrong Code (From Mac Host Terminal)
```bash
DATABASE_URL="postgresql://postgres:userpassword@postgres:5432/postgres" npx prisma migrate dev
# ❌ Mac cannot resolve "postgres" DNS directly
```

### ✅ Fixed Code (From Mac Host Terminal)
```bash
DATABASE_URL="postgresql://postgres:userpassword@localhost:5432/postgres" npx prisma migrate dev
# ✅ Uses Mac localhost port forwarding
```
