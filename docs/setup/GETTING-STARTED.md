# Getting Started

This guide covers prerequisites, environment setup, and running all services locally.

## Prerequisites

| Requirement | Version |
|-------------|---------|
| [Bun](https://bun.sh) | 1.3.3+ (specified in root `package.json`) |
| PostgreSQL | Any recent version |
| LLM API keys | OpenAI, Anthropic, and/or Google (depending on which providers you configure) |

## Installation

```bash
git clone https://github.com/code100x/openrouter.git
cd openrouter
bun install
```

## Environment Variables

Create a `.env` file at the repo root or in each app directory. Bun loads `.env` files automatically.

| Variable | Used By | Required | Description |
|----------|---------|----------|-------------|
| `DATABASE_URL` | `packages/db` | Yes | PostgreSQL connection string |
| `JWT_SECRET` | `primary-backend` | Yes | Secret for signing JWT tokens |
| `OPENAI_API_KEY` | `api-backend` | For OpenAI routes | OpenAI API key |
| `ANTHROPIC_API_KEY` | `api-backend` | For Claude routes | Anthropic API key |
| `GOOGLE_API_KEY` | `api-backend` | For Gemini routes | Google GenAI API key |
| `NODE_ENV` | `dashboard-frontend` | No | `development` for HMR, `production` for build |

Example `.env`:

```env
DATABASE_URL="postgresql://postgres:password@localhost:5432/openrouter"
JWT_SECRET="your-secret-key-here"
OPENAI_API_KEY="sk-..."
ANTHROPIC_API_KEY="sk-ant-..."
GOOGLE_API_KEY="..."
```

## Database Setup

1. Create a PostgreSQL database:

```sql
CREATE DATABASE openrouter;
```

2. Run Prisma migrations:

```bash
cd packages/db
bunx prisma migrate dev
```

3. Seed model catalog data manually. Insert records into `Company`, `Model`, `Provider`, and `ModelProviderMapping` so chat requests can resolve model slugs. There is no automated seed script.

## Running All Services

From the repo root:

```bash
bun run dev
```

This uses Turborepo to start all apps in parallel.

### Running Individual Apps

```bash
# Management API (port 3000)
cd apps/primary-backend && bun run dev

# LLM Proxy API (port 4000)
cd apps/api-backend && bun run dev

# Dashboard (port 3001)
cd apps/dashboard-frontend && bun run dev
```

## Verify Setup

1. Open `http://localhost:3001` — landing page should load
2. Sign up for a new account
3. Create an API key from the API Keys page
4. Add credits from the Credits page (+1,000 via mock onramp)
5. Test the LLM proxy:

```bash
curl -X POST http://localhost:4000/api/v1/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "your-model-slug",
    "messages": [{ "role": "user", "content": "Hello!" }]
  }'
```

## Other Scripts

| Command | Description |
|---------|-------------|
| `bun run build` | Build all apps via Turborepo |
| `bun run lint` | Lint all workspaces |
| `bun run check-types` | Type-check all workspaces |
| `bun run format` | Format code with Prettier |

## Troubleshooting

| Issue | Likely Cause |
|-------|--------------|
| Chat returns model not found | Model catalog not seeded in database |
| 401 on dashboard actions | Not signed in or JWT cookie expired |
| 401 on chat API | Invalid, disabled, or deleted API key |
| Insufficient credits | User credits are 0 — use onramp to add more |
| LLM provider error | Missing or invalid provider API key in `.env` |

For architecture details, see [SYSTEM-ARCHITECTURE.md](../architecture/SYSTEM-ARCHITECTURE.md).
