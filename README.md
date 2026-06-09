# StudentHub 🎓

> **Where students shine brighter**

A full-stack Student Management System for managing student records, tracking academic performance (CGPA) and viewing real-time analytics — all in one place.

🔗 **Live Demo:** [studenthub-flash.vercel.app](https://studenthub-flash.vercel.app)

---

## About the Project

StudentHub is a production-ready full-stack web application built to manage students efficiently. It supports full CRUD operations on student records, tracks CGPA and academic progress, and provides an analytics dashboard powered by raw PostgreSQL queries for accurate aggregate insights.

The frontend is deployed on **Vercel** and communicates with a **Dockerized Express backend** deployed on **Render**. Real-time features are handled via **Socket.IO**. Authentication is managed by **Better Auth** with middleware-level route protection.

---

## Features

- 📋 **Student Management** — View, add, edit, and delete student records
- 📈 **Performance Tracking** — Monitor CGPA and academic progress per student
- 📊 **Analytics Dashboard** — Aggregate insights powered by raw `pg` queries
- ⚡ **Real-time Updates** — Socket.IO integration for live data sync
- 🔐 **Authentication** — Email/password auth via Better Auth
- 🛡️ **Route Protection** — Middleware proxy guards protected pages with `callbackUrl` redirect
- 🚀 **Quick Add** — Simple form to onboard new students fast

---

## Tech Stack

### Frontend
| Tool | Purpose |
|------|---------|
| Next.js (App Router) | React framework, SSR, routing |
| React | UI components |
| TypeScript | Type safety across the codebase |
| Better Auth (`better-auth/minimal`) | Auth client — `signIn`, `signUp`, `signOut`, `useSession` |

### Backend
| Tool | Purpose |
|------|---------|
| Express.js | REST API server |
| TypeScript | Typed backend logic |
| Socket.IO | Real-time WebSocket communication |
| Prisma 7 | ORM for schema management and migrations |
| `pg` (node-postgres) | Raw SQL queries for analytics aggregations |
| Better Auth | Server-side session management |
| Redis (Upstash) | Session caching / fast data layer |

### Database & Infrastructure
| Tool | Purpose |
|------|---------|
| Neon PostgreSQL | Serverless PostgreSQL database |
| Docker | Containerized backend server |
| Render | Backend hosting (Docker-based deployment) |
| Vercel | Frontend hosting |

---

## Key Topics Covered

- **Dockerized Express backend** deployed to Render with a custom `Dockerfile`
- **Prisma 7** setup with `prisma.config.ts` for `url`/`directUrl` (breaking change from v6)
- **`@prisma/adapter-neon`** for serverless-compatible Neon connections
- **`better-auth/minimal`** import to avoid Turbopack ESM errors in Next.js 16
- **Raw `pg` queries** for dashboard analytics (aggregations, averages, counts)
- **Redis (Upstash)** for caching via REST-based `@upstash/redis` SDK
- **Socket.IO** with a split architecture: REST on Vercel, sockets on Render
- **Next.js Middleware** proxy for route protection with `callbackUrl` support
- **Neon pooled vs direct URLs** — pooled for runtime, direct for Prisma CLI

---

## Environment Variables

### Frontend (`.env.local`)
```env
DATABASE_URL="postgresql://user:pass@ep-xxx-pooler.region.aws.neon.tech/dbname?sslmode=require"
DIRECT_URL="postgresql://user:pass@ep-xxx.region.aws.neon.tech/dbname?sslmode=require"
BETTER_AUTH_SECRET="your-secret-min-32-chars"
BETTER_AUTH_URL="http://localhost:3000"
NEXT_PUBLIC_BETTER_AUTH_URL="http://localhost:3000"
NEXT_PUBLIC_BACKEND_URL="http://localhost:5000"
```

### Backend (`.env`)
```env
DATABASE_URL="postgresql://user:pass@ep-xxx-pooler.region.aws.neon.tech/dbname?sslmode=require"
REDIS_URL="your-upstash-redis-rest-url"
REDIS_TOKEN="your-upstash-redis-rest-token"
PORT=5000
```

---

## Getting Started

### 1. Clone & Install

```bash
git clone https://github.com/yourusername/studenthub
cd studenthub/frontend && npm install
cd ../backend && npm install
```

### 2. Setup Database

```bash
cd frontend
npx prisma generate
npx prisma db push
```

### 3. Run Locally

```bash
# Frontend
cd frontend && npm run dev

# Backend
cd backend && npm run dev
```

### 4. Run Backend with Docker

```bash
cd backend
docker build -t nihalxofficial/socket-server .
docker run -p 5000:5000 --env-file .env nihalxofficial/socket-server
```

---

## Middleware — Route Protection

Routes are protected via Next.js middleware. Unauthenticated users are redirected to `/login` with a `callbackUrl` so they land back on their intended page after signing in.

```
Protected routes: /add-student, /dashboard/*, /students/:id
Auth routes:      /login, /signup  (redirect away if already logged in)
```

---

## Deployment

| Service | What's deployed |
|---------|----------------|
| Vercel | Next.js frontend |
| Render | Dockerized Express + Socket.IO backend |
| Neon | PostgreSQL database |
| Upstash | Redis |
| DockerHub (server) | [nihalxofficial/socket-server](https://hub.docker.com/r/nihalxofficial/socket-server) |
| DockerHub (client) | [nihalxofficial/socket-client](https://hub.docker.com/r/nihalxofficial/socket-client) |

**Render build command:**
```bash
npm install && npx prisma generate && npm run build
```

**Render start command:**
```bash
npx prisma migrate deploy && node dist/index.js
```

---

## Better Auth + Prisma 7 + Neon Setup Guide

> Stack: **Better Auth 1.6** · **Prisma 7** · **Neon PostgreSQL** · **Next.js 16**

### Step 1 — Install Packages

```bash
npm install better-auth @prisma/client @prisma/adapter-neon dotenv
npm install -D prisma
```

---

### Step 2 — Environment Variables (`.env`)

Go to **Neon Console → your project → Connect** and copy the connection string.

```env
# Pooled — has -pooler in hostname (used by app at runtime)
DATABASE_URL="postgresql://user:pass@ep-xxx-pooler.region.aws.neon.tech/dbname?sslmode=require"

# Direct — remove -pooler from hostname (used by Prisma CLI)
DIRECT_URL="postgresql://user:pass@ep-xxx.region.aws.neon.tech/dbname?sslmode=require"

BETTER_AUTH_SECRET="any-long-random-string-minimum-32-chars"
BETTER_AUTH_URL="http://localhost:3000"
NEXT_PUBLIC_BETTER_AUTH_URL="http://localhost:3000"
```

> **Tip:** `DIRECT_URL` is just `DATABASE_URL` with `-pooler` removed from the hostname.

---

### Step 3 — Initialize Prisma

```bash
npx prisma init --output ../src/generated/prisma
```

---

### Step 4 — `prisma/schema.prisma`

> In Prisma 7: provider is `prisma-client`, `output` is required, and `url`/`directUrl` are moved to `prisma.config.ts`.

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated/prisma"
}

datasource db {
  provider = "postgresql"
}

model User {
  id            String    @id
  name          String
  email         String    @unique
  emailVerified Boolean
  image         String?
  createdAt     DateTime
  updatedAt     DateTime
  sessions      Session[]
  accounts      Account[]
}

model Session {
  id        String   @id
  expiresAt DateTime
  token     String   @unique
  createdAt DateTime
  updatedAt DateTime
  ipAddress String?
  userAgent String?
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model Account {
  id                    String    @id
  accountId             String
  providerId            String
  userId                String
  user                  User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  accessToken           String?
  refreshToken          String?
  idToken               String?
  accessTokenExpiresAt  DateTime?
  refreshTokenExpiresAt DateTime?
  scope                 String?
  password              String?
  createdAt             DateTime
  updatedAt             DateTime
}

model Verification {
  id         String    @id
  identifier String
  value      String
  expiresAt  DateTime
  createdAt  DateTime?
  updatedAt  DateTime?
}
```

---

### Step 5 — `prisma.config.ts` (root of project)

```ts
import "dotenv/config";
import { defineConfig, env } from "prisma/config";

export default defineConfig({
  schema: "prisma/schema.prisma",
  migrations: {
    path: "prisma/migrations",
  },
  datasource: {
    url: env("DIRECT_URL"), // CLI uses direct connection
  },
});
```

---

### Step 6 — `src/lib/prisma.ts`

```ts
import "dotenv/config";
import { PrismaClient } from "@/generated/prisma/client";
import { PrismaNeon } from "@prisma/adapter-neon";

const adapter = new PrismaNeon({
  connectionString: process.env.DATABASE_URL!,
});

const globalForPrisma = global as unknown as { prisma: PrismaClient };

const prisma = globalForPrisma.prisma || new PrismaClient({ adapter });

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;

export default prisma;
```

---

### Step 7 — `src/lib/auth.ts`

> Use `better-auth/minimal` with Prisma — it excludes Kysely and fixes the Turbopack ESM error.

```ts
import { betterAuth } from "better-auth/minimal";
import { prismaAdapter } from "better-auth/adapters/prisma";
import prisma from "@/lib/prisma";

export const auth = betterAuth({
  database: prismaAdapter(prisma, {
    provider: "postgresql",
  }),
  emailAndPassword: {
    enabled: true,
  },
});
```

---

### Step 8 — `src/app/api/auth/[...all]/route.ts`

```ts
import { auth } from "@/lib/auth";
import { toNextJsHandler } from "better-auth/next-js";

export const { POST, GET } = toNextJsHandler(auth);
```

---

### Step 9 — `src/lib/auth-client.ts`

```ts
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_BETTER_AUTH_URL || "http://localhost:3000",
});

export const { signIn, signUp, signOut, useSession } = authClient;
```

---

### Step 10 — Generate & Push to Database

```bash
npx prisma generate
npx prisma db push
npm run dev
```

---

### Usage in Components

```ts
// Sign up
await signUp.email({ name, email, password });

// Sign in
await signIn.email({ email, password });

// Sign out
await signOut();

// Get session (in a Client Component)
const { data: session } = useSession();
console.log(session?.user);
```

---

### When to Run Prisma Commands

| Situation | Commands |
|-----------|----------|
| Changed `schema.prisma` | `npx prisma db push` → `npx prisma generate` |
| Changed TypeScript only | Nothing needed |
| Want migration history (production) | `npx prisma migrate dev` → `npx prisma generate` |

> **Rule:** Always run both commands in order whenever `schema.prisma` changes.

---

### Why `better-auth/minimal`?

Next.js 16 uses Turbopack by default. `better-auth` (full) imports `@better-auth/kysely-adapter` which causes a Turbopack ESM export error. `better-auth/minimal` skips Kysely entirely — it is the correct import when using Prisma or Drizzle adapters.

---

### Why Two Database URLs?

| Variable | Used by | Why |
|----------|---------|-----|
| `DATABASE_URL` | Your app (runtime) | Pooled connection handles serverless concurrency |
| `DIRECT_URL` | Prisma CLI (`db push`, `migrate`) | Schema changes need a direct connection |
