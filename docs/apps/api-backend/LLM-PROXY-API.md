# LLM Proxy API (api-backend)

The **api-backend** app is the inference proxy. External clients send chat completion requests here using a Bearer API key. The service validates the key, checks credits, routes to an LLM provider, and deducts usage credits.

| Property | Value |
|----------|-------|
| **Folder** | `apps/api-backend/` |
| **Port** | `4000` |
| **Framework** | Elysia + `@elysiajs/bearer` |

## Directory Structure

```
apps/api-backend/
└── src/
    ├── index.ts       # Chat completions endpoint
    ├── types.ts       # Request/response schemas (Elysia TypeBox)
    └── llms/
        ├── Base.ts    # LlmResponse type and base class
        ├── OpenAi.ts  # OpenAI adapter
        ├── Claude.ts  # Anthropic adapter
        └── Gemini.ts  # Google Gemini adapter
```

## API Endpoint

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/v1/chat/completions` | Bearer API key | Proxy chat request to an LLM provider |

### Request Body

```json
{
  "model": "anthropic/claude-sonnet-4",
  "messages": [
    { "role": "user", "content": "Hello!" }
  ]
}
```

- `model` — slug in `company/provider-model` format
- `messages` — array of `{ role: "user" | "assistant", content: string }`

### Response

Returns a normalized `LlmResponse` with content, token counts, and model metadata from the selected provider adapter.

## Request Processing Flow

1. **Parse model slug** — split on `/` to identify company and model
2. **Validate Bearer token** — look up API key in DB (must not be disabled or deleted)
3. **Check credits** — user must have credits > 0
4. **Lookup model** — find model by slug in the database
5. **Select provider** — pick a random entry from `ModelProviderMapping` for that model
6. **Route to adapter** based on provider name:
   - `"OpenAI"` → `OpenAi.chat()`
   - `"Claude API"` → `Claude.chat()`
   - `"Google API"` / `"Google Vertex"` → `Gemini.chat()`
7. **Calculate cost** — `(inputTokens × inputCost + outputTokens × outputCost) / 10`
8. **Update records** — decrement user credits, increment `apiKey.creditsConsumed`
9. **Return response**

## LLM Adapters

Each adapter in `src/llms/` wraps a provider SDK and normalizes the response into the shared `LlmResponse` shape defined in `Base.ts`.

| Adapter | SDK | Env Variable |
|---------|-----|--------------|
| `OpenAi.ts` | `openai` | `OPENAI_API_KEY` |
| `Claude.ts` | `@anthropic-ai/sdk` | `ANTHROPIC_API_KEY` |
| `Gemini.ts` | `@google/genai` | `GOOGLE_API_KEY` |

## Environment Variables

| Variable | Required | Purpose |
|----------|----------|---------|
| `DATABASE_URL` | Yes | PostgreSQL connection string |
| `OPENAI_API_KEY` | For OpenAI routes | OpenAI API access |
| `ANTHROPIC_API_KEY` | For Claude routes | Anthropic API access |
| `GOOGLE_API_KEY` | For Gemini routes | Google GenAI access |

## Example cURL Request

```bash
curl -X POST http://localhost:4000/api/v1/chat/completions \
  -H "Authorization: Bearer sk-or-v1-YOUR-KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "anthropic/claude-sonnet-4",
    "messages": [{ "role": "user", "content": "Hello!" }]
  }'
```

## Running Locally

```bash
cd apps/api-backend
bun run dev
```

## Notes

- The `Conversation` table exists in the schema for usage logging but is **not yet written to** during chat requests
- Provider selection is **random** among available mappings for the requested model
- Model catalog data (`Company`, `Model`, `Provider`, `ModelProviderMapping`) must exist in the database before requests succeed
