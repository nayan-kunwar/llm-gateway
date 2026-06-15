# Management API (primary-backend)

The **primary-backend** app is the user-facing management API. It handles authentication, API key lifecycle, model catalog queries, and credit top-ups.

| Property | Value |
|----------|-------|
| **Folder** | `apps/primary-backend/` |
| **npm package name** | `app` |
| **Port** | `3000` |
| **Framework** | Elysia (Bun runtime) |

## Directory Structure

```
apps/primary-backend/
├── src/
│   ├── index.ts       # Server entry — CORS + listen on :3000
│   ├── app.ts         # Composes all route modules; exports typed App
│   └── modules/
│       ├── auth/      # Sign-up, sign-in, profile
│       ├── apiKeys/   # API key CRUD
│       ├── models/    # Model and provider catalog
│       └── payments/  # Credit onramp
└── package.json
```

## API Routes

### Auth (`/auth`)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/auth/sign-up` | No | Register with email + password |
| POST | `/auth/sign-in` | No | Login; sets httpOnly `auth` JWT cookie (7 days) |
| GET | `/auth/profile` | JWT cookie | Returns user profile including `{ credits }` |

Passwords are hashed with `Bun.password.hash` and verified with `Bun.password.verify`.

### API Keys (`/api-keys`)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api-keys/` | JWT | Create a new key (`sk-or-v1-...` format) |
| GET | `/api-keys/` | JWT | List the user's keys |
| PUT | `/api-keys/` | JWT | Enable or disable a key |
| DELETE | `/api-keys/:id` | JWT | Soft-delete a key |

### Models (`/models`)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/models/` | No | List all models with company info |
| GET | `/models/providers` | No | List all providers |
| GET | `/models/:id/providers` | No | Provider mappings and token costs for a model |

### Payments (`/payments`)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/payments/onramp` | JWT | Add 1,000 credits (mock payment) |

## Authentication

- JWT signed with the `JWT_SECRET` environment variable
- Token stored in an httpOnly cookie named `auth`
- CORS configured for `http://localhost:3001` with `credentials: true`

## Frontend Integration

The dashboard imports the exported `App` type from `app.ts` and uses `@elysiajs/eden` treaty for end-to-end type safety:

```ts
// apps/dashboard-frontend/src/providers/Eden.tsx
const client = treaty<App>('http://localhost:3000', { fetch: { credentials: 'include' } })
```

## Environment Variables

| Variable | Required | Purpose |
|----------|----------|---------|
| `DATABASE_URL` | Yes | PostgreSQL connection string |
| `JWT_SECRET` | Yes | Secret for signing JWT tokens |

## Running Locally

```bash
cd apps/primary-backend
bun run dev
```

Or from the repo root:

```bash
bun run dev --filter=app
```
