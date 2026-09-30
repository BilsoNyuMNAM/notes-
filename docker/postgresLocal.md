---
title: RUNNING A LOCAL POSTGRES CONTAINER FOR BACKEND DEVELOPMENT
description: How to run a local PostgreSQL container, map ports, configure the database URL for local backend and migrations, and inspect tables inside the container.
type: NOTE
year: 2026
tags: [docker, postgres, database, prisma, fundamentals]
---

# Running a Local PostgreSQL Container for Backend Development

When developing locally, you do not need a cloud database. You can spin up a PostgreSQL instance inside a Docker container and connect your local backend or migration tools (e.g., Prisma) directly to it.

---

## 1. Command to Run PostgreSQL Container

Run this command in your terminal:

```bash
docker run --name some-postgres \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -p 5432:5432 \
  -d postgres
```

### Breakdown of the Command

| Flag / Option | What It Does | Default When Omitted |
| :--- | :--- | :--- |
| `--name some-postgres` | Names the running container | Random generated name |
| `-e POSTGRES_PASSWORD=...` | Sets the password for the superuser | **Required** |
| `-p 5432:5432` | Maps host port `5432` to container port `5432` | **None** *(Host cannot access DB without this)* |
| `-d` | Runs the container in the background (detached mode) | Runs in foreground |
| `postgres` | Docker image to download and run | Uses `latest` tag |

> [!NOTE]
> If you do not pass `-e POSTGRES_USER` or `-e POSTGRES_DB`:
> - **Username** defaults to `postgres`.
> - **Database name** defaults to `postgres`.

---

## 2. Why Port Mapping is Required & How the URL Connects

Docker containers run in an **isolated network namespace**. By default, applications running on your machine (like your Express server or `prisma migrate`) cannot access services inside a container.

To bridge this gap, you must use **port mapping**:

👉 **Refer to the port mapping note:** [Understanding Port Mapping](./portMapping.md) (or [Web Note](/notes/docker/portMapping/))

### Diagram: Host to Container Port Mapping

![Docker Port Mapping](https://www.acte.in/wp-content/uploads/2025/02/Docker-Port-Mapping-1568x854.png)

```text
Host Machine (Your Computer)                    PostgreSQL Container
┌──────────────────────────────┐              ┌─────────────────────────────┐
│ Backend App / Prisma         │              │ PostgreSQL Server           │
│                              │              │                             │
│ Connects to:                 │ -p 5432:5432 │ Listens on:                 │
│ localhost:5432 ──────────────┼─────────────►│ container:5432              │
└──────────────────────────────┘              └─────────────────────────────┘
      ▲                                              ▲
      │                                              │
  HOST PORT                                    CONTAINER PORT
```

- `-p <HOST_PORT>:<CONTAINER_PORT>` tells Docker: *"Any traffic sent to `localhost:<HOST_PORT>` on my machine must be forwarded to `<CONTAINER_PORT>` inside the container."*
- If port `5432` is already in use on your computer, map a different host port (e.g., `-p 5433:5432`). In that case, your URL host becomes `localhost:5433`.

---

## 3. The PostgreSQL Connection URL

### Generic Format
```text
postgresql://USER:PASSWORD@HOST:PORT/DATABASE_NAME?schema=public
```

### URL For the Container Started Above
For the command `docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 -d postgres`:

```env
DATABASE_URL="postgresql://postgres:mysecretpassword@localhost:5432/postgres?schema=public"
```

### Component Mapping

| URL Component | Value | Where It Comes From |
| :--- | :--- | :--- |
| **Protocol** | `postgresql://` | Standard PostgreSQL driver protocol |
| **USER** | `postgres` | Default value when `POSTGRES_USER` is omitted |
| **PASSWORD** | `mysecretpassword` | Set by `-e POSTGRES_PASSWORD=mysecretpassword` |
| **HOST** | `localhost` | Points to your machine (exposed via port mapping) |
| **PORT** | `5432` | Left number in `-p 5432:5432` (the host port) |
| **DATABASE_NAME** | `postgres` | Default value when `POSTGRES_DB` is omitted |
| **Query Param** | `?schema=public` | Default PostgreSQL schema |

---

## 4. How to Get Inside the Container and Inspect Tables

Once migrations run, verify the database schema directly inside the container using `docker exec` and `psql`.

### Step 1: Open `psql` Inside the Container

Using the container name:
```bash
docker exec -it some-postgres psql -U postgres -d postgres
```

*(Or using the container ID, e.g. `docker exec -it c6ab498a40cc psql -U postgres -d postgres`)*

- `-it`: Allocates an interactive terminal.
- `psql`: The PostgreSQL command-line client.
- `-U postgres`: Specifies the database user.
- `-d postgres`: Specifies the database name.

### Step 2: Inspect Tables

Once connected (`postgres=#` prompt):

1. **List all tables:**
   ```sql
   \dt
   ```

2. **Describe table schema & columns:**
   ```sql
   \d "User"
   ```
   *(Use double quotes `"User"` because ORMs like Prisma create table names with matching case).*

3. **Query records:**
   ```sql
   SELECT * FROM "User";
   ```

4. **Exit `psql`:**
   ```text
   \q
   ```
