# OpenRouter Documentation

Documentation for the **openrouter** monorepo — an OpenRouter-style LLM gateway with user management, API keys, credit billing, and multi-provider chat routing.

## Documentation Index

| Folder | Document | Description |
|--------|----------|-------------|
| [overview](./overview/) | [PROJECT-OVERVIEW.md](./overview/PROJECT-OVERVIEW.md) | What the project is, key features, and user journey |
| [architecture](./architecture/) | [SYSTEM-ARCHITECTURE.md](./architecture/SYSTEM-ARCHITECTURE.md) | How apps and packages fit together |
| [apps/primary-backend](./apps/primary-backend/) | [MANAGEMENT-API.md](./apps/primary-backend/MANAGEMENT-API.md) | Auth, API keys, models, and payments API |
| [apps/api-backend](./apps/api-backend/) | [LLM-PROXY-API.md](./apps/api-backend/LLM-PROXY-API.md) | Chat completions proxy and LLM routing |
| [apps/dashboard-frontend](./apps/dashboard-frontend/) | [DASHBOARD-UI.md](./apps/dashboard-frontend/DASHBOARD-UI.md) | React dashboard and frontend routes |
| [packages/database](./packages/database/) | [DATABASE-SCHEMA.md](./packages/database/DATABASE-SCHEMA.md) | Prisma schema and data models |
| [setup](./setup/) | [GETTING-STARTED.md](./setup/GETTING-STARTED.md) | Environment variables and local development |

## Quick Start

```bash
bun install
# Configure DATABASE_URL, JWT_SECRET, and LLM API keys (see setup doc)
bun run dev
```

Default ports:

- **Dashboard** — `http://localhost:3001`
- **Management API** — `http://localhost:3000`
- **LLM Proxy API** — `http://localhost:4000`
