# Golazo Backend

REST API for Golazo — a goal-tracking community app where members set goals and post weekly progress updates.

**Stack:** Bun + Elysia + TypeScript + Drizzle ORM + SQLite + JWT + Cloudflare R2

---

## Prerequisites

- [Bun](https://bun.sh) v1.2+
- A Cloudflare R2 bucket (for image uploads)

---

## Setup

### 1. Install dependencies

```bash
bun install
```

### 2. Configure environment variables

Copy the example env file and fill in your values:

```bash
cp .env.example .env
```

| Variable | Description |
|---|---|
| `JWT_SECRET` | Secret used to sign JWTs — use a long random string |
| `R2_ACCOUNT_ID` | Cloudflare account ID |
| `R2_ACCESS_KEY_ID` | R2 API token access key |
| `R2_SECRET_ACCESS_KEY` | R2 API token secret |
| `R2_BUCKET_NAME` | Name of your R2 bucket |
| `R2_PUBLIC_URL` | Public base URL for the bucket (e.g. `https://pub-xxx.r2.dev`) |

To generate a strong `JWT_SECRET`:
```bash
openssl rand -base64 32
```

### 3. Initialize the database

Push the schema to create the SQLite database and tables:

```bash
bun run db:push
```

This creates `golazo.db` in the project root.

### 4. Start the server

```bash
bun run dev
```

The server starts on `http://localhost:3000`.

---

## Database Scripts

| Command | Description |
|---|---|
| `bun run db:push` | Push schema changes directly to the database (good for development) |
| `bun run db:generate` | Generate migration files from schema changes |
| `bun run db:migrate` | Apply pending migrations |
| `bun run db:studio` | Open Drizzle Studio (visual database browser) |

---

## API Overview

### Auth (public)
| Method | Path | Description |
|---|---|---|
| `POST` | `/auth/register` | Create account, returns JWT |
| `POST` | `/auth/login` | Verify credentials, returns JWT |

### Users
| Method | Path | Description |
|---|---|---|
| `GET` | `/users/me` | Get own profile |
| `PUT` | `/users/me` | Update own profile |

### Goals
| Method | Path | Description |
|---|---|---|
| `GET` | `/goals` | List own goals |
| `POST` | `/goals` | Create a goal |
| `GET` | `/goals/:goalId` | Get a goal |
| `PUT` | `/goals/:goalId` | Update a goal |
| `DELETE` | `/goals/:goalId` | Delete a goal |

### Weekly Updates
| Method | Path | Description |
|---|---|---|
| `GET` | `/goals/:goalId/updates` | List updates for a goal |
| `POST` | `/goals/:goalId/updates` | Create a weekly update |
| `GET` | `/goals/:goalId/updates/:id` | Get an update |
| `PUT` | `/goals/:goalId/updates/:id` | Update an update |
| `DELETE` | `/goals/:goalId/updates/:id` | Delete an update |

### Uploads
| Method | Path | Description |
|---|---|---|
| `POST` | `/uploads` | Upload an image to R2, returns `{ url }` |

### Admin only
| Method | Path | Description |
|---|---|---|
| `GET` | `/admin/users` | List all users |
| `GET` | `/admin/users/:id` | Get a user |
| `GET` | `/admin/users/:id/goals` | Get a user's goals |
| `GET` | `/admin/users/:id/goals/:goalId/updates` | Get a user's updates for a goal |

All routes except `/auth/*` require a `Authorization: Bearer <token>` header. Admin routes additionally require the user to have `role: 'admin'`.

---

## Image Uploads

To attach an image to a weekly update:

1. `POST /uploads` with `multipart/form-data`, field name `image` — returns `{ url: "https://..." }`
2. Include that `url` as `image_url` when creating or updating a weekly update

---

## Project Structure

```
src/
├── index.ts              # App entry point
├── db/
│   ├── index.ts          # Drizzle instance
│   └── schema.ts         # Table definitions
├── lib/
│   └── r2.ts             # R2 client + upload helper
├── middleware/
│   └── auth.ts           # JWT + role guard plugins
└── routes/
    ├── auth.ts           # /auth/*
    ├── uploads.ts        # /uploads
    ├── users.ts          # /users/me
    ├── goals.ts          # /goals/* + nested updates
    └── admin.ts          # /admin/*
drizzle/                  # Generated migration files
drizzle.config.ts         # Drizzle Kit config
.env.example              # Environment variable template
```
