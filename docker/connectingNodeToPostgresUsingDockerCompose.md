---
title: CONNECTING A NODEJS BACKEND PROJECT TO POSTGRES THROUGH PRISMA USING DOCKER COMPOSE
description: End-to-end guide on connecting an Express TypeScript backend to a PostgreSQL container using Prisma and Docker Compose.
type: NOTE
year: 2026
tags: [docker, docker-compose, postgres, prisma, nodejs]
---

## 1. Project Overview & Folder Structure

This project is a simple Node.js backend application that connects to a PostgreSQL database using Prisma ORM.

It exposes only **two routes**:
- `GET /` — Fetches all user records from the database.
- `POST /` — Generates and inserts a new user record into the database.

That is it.

### File and Folder Structure

```text
week-27-docker-compose/
├── Contribute.md
├── Dockerfile
├── docker-compose.yml
├── package.json
├── package-lock.json
├── tsconfig.json
├── prisma/
│   └── schema.prisma
└── src/
    └── index.ts
```
[Link to the repo](https://github.com/100xdevs-cohort-3/week-27-docker-compose/)

### The Application Code (`src/index.ts`)

```typescript
import { PrismaClient } from "@prisma/client";
import express from "express";

const app = express();
const prismaClient = new PrismaClient();

app.get("/", async (req, res) => {
    const data = await prismaClient.user.findMany();
    res.json({ data });
});

app.post("/", async (req, res) => {
    await prismaClient.user.create({
        data: {
            username: Math.random().toString(),
            password: Math.random().toString()
        }
    });
    res.json({ message: "post endpoint" });
});

// Listens on 0.0.0.0 so Docker can route traffic from host
app.listen(3000, "0.0.0.0", () => {
    console.log("Application listening on port 3000");
});
```

> [!TIP]
> For why `0.0.0.0` is required instead of `localhost`, see [Bug 3 in the Bug Log](./dockerComposePrismaBugs.md#3-bug-3-express-listening-on-127001-instead-of-0000).

---

## 2. Why a Dockerfile is Needed & Connection to `build: .`

A `Dockerfile` is needed because our application is custom source code. Unlike PostgreSQL, which has a pre-built image on Docker Hub (`image: postgres`), Docker does not know how to install our npm dependencies, generate Prisma client types, or compile our TypeScript code.

In `docker-compose.yml`, the line:

```yaml
application:
  build: .
```

tells Docker Compose: **"Look at the `Dockerfile` in the current directory (`.`), execute its instructions, and build the container image for the `application` service."**

### The Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY ./package.json .
COPY ./package-lock.json .

RUN npm install

COPY . .

# Generate Prisma client before compiling TypeScript
RUN npx prisma generate
RUN npm run build

CMD ["npm", "run", "dev:docker"]
```

> [!IMPORTANT]
> `RUN npx prisma generate` must come before `RUN npm run build`. Otherwise, TypeScript compilation fails. See [Bug 4 in the Bug Log](./dockerComposePrismaBugs.md#4-bug-4-dockerfile-build-fails-due-to-order-of-prisma-generate-and-build).

---

## 3. Docker Compose (`docker-compose.yml`)

The `docker-compose.yml` file manages both services: the PostgreSQL database and our custom Node.js application.

```yaml
services:
  postgres:
    image: postgres
    environment:
      POSTGRES_PASSWORD: userpassword
    ports:
      - "5432:5432"

  application:
    build: .
    environment:
      DATABASE_URL: "postgresql://postgres:userpassword@postgres:5432/postgres"
    ports:
      - "3000:3000"
    depends_on:
      - postgres
```

### Breakdown of Key Fields:

1. **`postgres` service:**
   * Uses the official `postgres` image from Docker Hub.
   * Sets `POSTGRES_PASSWORD: userpassword` — the password is created by you here on the first start.
   * `ports: - "5432:5432"` maps container port 5432 to your Mac host so you can connect local tools like TablePlus or run migrations from your host terminal.

2. **`application` service:**
   * `build: .` builds the image using the local `Dockerfile`.
   * `ports: - "3000:3000"` forwards your laptop's `localhost:3000` to the container's port `3000`.
   * `depends_on: - postgres` ensures the database container starts before the application container.

> [!WARNING]
> When running `docker compose up`, omitting the hyphen `-` before `"3000:3000"` causes:
> ```text
> validating docker-compose.yml: services.application.ports must be a array
> ```
> See [Bug 1 in the Bug Log](./dockerComposePrismaBugs.md#1-bug-1-docker-compose-syntax-error--ports-must-be-an-array) for the side-by-side comparison of `postgres` vs `application`.

3. **Database URL & DNS Resolution:**
   ```text
   DATABASE_URL: "postgresql://postgres:userpassword@postgres:5432/postgres"
                                                   ↑
                                          Docker Service Name
   ```
   Inside Docker Compose, containers communicate through their service names. The app talks to `postgres:5432`, **not** `localhost:5432`. *(See [Bug 5 in the Bug Log](./dockerComposePrismaBugs.md#5-bug-5-hostname-confusion-in-database_url-host-vs-container)).*

---

## 4. Starting the Application (`package.json` Startup Script)

In `package.json`, the container startup command is:

```json
"scripts": {
  "build": "tsc -b",
  "dev:docker": "npx prisma db push && node dist/index.js"
}
```

### Why `prisma db push` Instead of `prisma migrate dev`?
* The project has no `prisma/migrations/` folder.
* Running `prisma migrate dev` inside Docker freezes the container because it stops and waits for a user to type a migration name in the terminal.
* `prisma db push` directly syncs `schema.prisma` with PostgreSQL without asking questions, allowing `node dist/index.js` to start immediately.

> [!WARNING]
> For details on why `prisma migrate dev` causes the container to freeze and why `--name` doesn't solve it on restarts, read [Bug 2 in the Bug Log](./dockerComposePrismaBugs.md#2-bug-2-container-freezes-on-startup-due-to-prisma-migrate-dev).

---

## 5. Running the Project

To start both services in detached mode:

```bash
docker compose up -d --build
```

To test the endpoints:

```bash
# Insert a user
curl -X POST http://localhost:3000/

# Fetch all users
curl http://localhost:3000/
```

To view logs:
```bash
docker compose logs -f application
```

---

## 6. Verifying Files Inside the Running Container (`docker exec`)

To inspect what was actually built and copied into the container's working directory (`/app`), open an interactive shell:

```bash
docker exec -it <container_id> /bin/sh
# or using compose service name:
docker compose exec application sh
```

Running `ls` inside `/app` confirms all files:

![Files inside container](./assets/docker-exec-container-files.png)

Inside `/app`, we can verify:
- `dist/`: Contains compiled JavaScript files generated by `RUN npm run build`.
- `node_modules/`: Contains installed dependencies and the generated `@prisma/client`.
- `prisma/`: Contains `schema.prisma`.
- Source and config files: `src/`, `package.json`, `tsconfig.json`, `Contribute.md`.

---

## Reference & Troubleshooting
- [Common Mistakes & Bug Log](./dockerComposePrismaBugs.md)
