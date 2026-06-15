# Database Package (db)

The **db** package is the shared data layer for both backends. It exports a singleton Prisma client connected to PostgreSQL.

| Property | Value |
|----------|-------|
| **Folder** | `packages/db/` |
| **ORM** | Prisma 7 |
| **Database** | PostgreSQL |
| **Adapter** | `@prisma/adapter-pg` + `pg` |

## Directory Structure

```
packages/db/
├── index.ts                  # Exports prisma singleton
├── prisma/
│   ├── schema.prisma         # Data model definitions
│   ├── migrations/           # Migration history
│   └── prisma.config.ts      # Prisma configuration
└── generated/prisma/         # Generated Prisma client (output)
```

## Usage in Backends

Both `primary-backend` and `api-backend` import the shared client:

```ts
import { prisma } from 'db'
```

## Entity Relationship Diagram

```mermaid
erDiagram
    User ||--o{ ApiKey : has
    User ||--o{ OnrampTransaction : has
    User ||--o{ Conversation : has
    ApiKey ||--o{ Conversation : has
    Company ||--o{ Model : has
    Model ||--o{ ModelProviderMapping : has
    Provider ||--o{ ModelProviderMapping : has
    ModelProviderMapping ||--o{ Conversation : has
```

## Models

### User

| Field | Type | Notes |
|-------|------|-------|
| `id` | Int | Primary key |
| `email` | String | Unique |
| `password` | String | Hashed with Bun.password |
| `credits` | Int | Default 1000 on signup |

### ApiKey

| Field | Type | Notes |
|-------|------|-------|
| `id` | Int | Primary key |
| `userId` | Int | FK → User |
| `name` | String | User-defined label |
| `apiKey` | String | Unique, format `sk-or-v1-...` |
| `disabled` | Boolean | Default false |
| `deleted` | Boolean | Soft delete flag |
| `lastUsed` | DateTime? | Not yet updated by chat flow |
| `creditsConsumed` | Int | Running total of credits used |

### Company

Represents an LLM company (e.g. Anthropic, OpenAI). Has many `Model` records.

### Model

A specific model offered on the platform. Identified by `slug` (used in chat requests). Belongs to a `Company`.

### Provider

An infrastructure provider that serves a model (e.g. "OpenAI", "Claude API", "Google API").

### ModelProviderMapping

Links a `Model` to a `Provider` with pricing:

| Field | Type | Notes |
|-------|------|-------|
| `inputTokenCost` | Int | Cost per input token |
| `outputTokenCost` | Int | Cost per output token |

Used by `api-backend` to select a provider and calculate credit deductions.

### OnrampTransaction

Records credit purchase transactions from the mock onramp endpoint.

### Conversation

Schema for logging chat usage. Fields include input/output text, token counts, and relations to User, ApiKey, and ModelProviderMapping. **Not yet populated** by the chat endpoint.

## Migrations

Run migrations from the `packages/db` directory:

```bash
cd packages/db
bunx prisma migrate dev
```

## Environment Variables

| Variable | Required | Purpose |
|----------|----------|---------|
| `DATABASE_URL` | Yes | PostgreSQL connection string |

Example:

```
DATABASE_URL="postgresql://user:password@localhost:5432/openrouter"
```

## Seeding

There is **no seed script** in the repository. Before chat requests work, you must manually insert:

- `Company` records
- `Model` records (with slugs matching request format)
- `Provider` records
- `ModelProviderMapping` records (linking models to providers with pricing)

Without this data, model lookups in `api-backend` will fail.
