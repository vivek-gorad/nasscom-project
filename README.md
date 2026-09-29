# FinNews AI

FinNews AI is a financial news dashboard that turns complex business headlines into clear, beginner-friendly explanations. Headlines load first, then each AI explanation resolves independently so the first useful result does not wait for the slowest article.

## Features

- Live business and financial headlines from NewsAPI
- Search and category filters
- Duplicate detection before AI processing
- Plain-English explanations with:
  - what happened
  - why it matters
  - simple explanation
  - important terms
  - one-line takeaway
- Concurrent AI summaries with a four-request limit
- In-memory news cache (5 minutes) and summary cache (24 hours)
- Progressive loading, error, empty, and partial-result states
- Original article links and responsive dashboard layout
- API keys remain server-side

## Architecture

- `artifacts/finnews-ai` — React + Vite dashboard
- `artifacts/api-server` — Express API service that handles NewsAPI, Groq, caching, validation, and CORS
- `lib/api-spec/openapi.yaml` — API contract source of truth
- `lib/api-client-react` — generated React Query client
- `lib/api-zod` — generated request and response validation schemas

The managed API service is used instead of exposing provider keys in the browser. The cache is isolated behind a small TTL interface so it can be replaced with Redis later without changing the route contract.

## Prerequisites

- Node.js 20 or newer
- pnpm
- A NewsAPI account and API key
- A Groq account and API key

## Environment

Copy `.env.example` to `.env` for local reference, then add the values through your environment's secret manager:

```env
NEWS_API_KEY=...
GROQ_API_KEY=...
GROQ_MODEL=llama-3.1-8b-instant
ALLOWED_ORIGINS=
```

`GROQ_MODEL` is optional. Use `llama-3.3-70b-versatile` only when that model is available to your Groq account; the default is the smaller fast model.

## Run the app

Install workspace dependencies:

```bash
pnpm install
```

Start the API:

```bash
pnpm --filter @workspace/api-server run dev
```

Start the web app in a second terminal:

```bash
pnpm --filter @workspace/finnews-ai run dev
```

In the managed workspace, the preview opens from the root app route. On Windows PowerShell, the commands are the same:

```powershell
pnpm install
pnpm --filter @workspace/api-server run dev
pnpm --filter @workspace/finnews-ai run dev
```

## API endpoints

- `GET /api/healthz` — service health
- `GET /api/news?category=&query=&limit=` — fetch deduplicated news
- `POST /api/summarize` — summarize one article
- `GET /api/news-and-summaries?category=&query=&limit=` — fetch news and cached summaries

## Performance and reliability

- News results are cached for five minutes.
- Summaries are cached for 24 hours by content fingerprint.
- Up to four Groq requests can run at once.
- News articles are returned independently from AI summaries.
- Provider timeouts prevent stalled requests.
- Invalid provider responses are rejected through generated Zod schemas.
- NewsAPI and Groq failures return useful errors without crashing the server.

## Troubleshooting

### News headlines do not load

Check that `NEWS_API_KEY` is a valid, active NewsAPI key. A revoked or mistyped key returns a provider error while keeping the dashboard available.

### Summaries do not load

Check `GROQ_API_KEY` and `GROQ_MODEL`. Model availability depends on the Groq account; use the default model or choose a model listed in the Groq console.

### API changes

Edit `lib/api-spec/openapi.yaml`, then regenerate the clients:

```bash
pnpm --filter @workspace/api-spec run codegen
```

## Future improvements

- Move TTL caches to Redis for multi-instance deployments
- Store user-selected saved briefings
- Add source-level trust and coverage indicators
- Stream summary tokens over server-sent events