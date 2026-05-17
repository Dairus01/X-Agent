# X-Agent

> Autonomous AI crypto intelligence — one API token, all public data, fully self-hostable.

X-Agent is a "Perplexity for crypto" research surface with a Bloomberg-terminal feel.
It clusters live narratives from public RSS, pulls real prices from CoinGecko,
on-chain TVL from DefiLlama, and routes every LLM call through **OpenRouter** so a
single key unlocks the entire model catalog.

**Hard constraints baked into the product:**

- OpenRouter is the **only** API token. No OpenAI, no Anthropic-direct, no per-vendor keys.
- **No** Twitter API, **no** Reddit API, **no** paid feeds.
- **Real data only.** No mock arrays, no synthetic sparklines, no simulated signals.
  Where a number is not available from a public source, the UI hides the tile
  rather than fabricate it.

## Demo

<div align="center">
  <video src="https://github.com/Dairus01/X-Agent/releases/download/readme-demo/demo.mp4" width="800" controls></video>
</div>

Walkthrough of the landing page, research chat, market dashboard, narratives, and watchlist.

---

## Quick start

```bash
# 1. Clone & install
npm install

# 2. Set your single token
cp .env.example .env.local
#   → edit .env.local and set OPENROUTER_API_KEY

# 3. Run
npm run dev      # http://localhost:3000
```

That's the whole setup. No auth, no accounts, no database, no settings page.

### Production

```bash
npm run build
npm start
```

Deploys cleanly to Vercel, Fly, Railway, or any Node host. Set
`OPENROUTER_API_KEY` in the host's env panel — that is the only required variable.

---

## Environment

| Variable | Required | Default | What |
|---|---|---|---|
| `OPENROUTER_API_KEY` | yes | — | Single LLM credential. Get one at openrouter.ai |
| `OPENROUTER_MODEL` | no | `google/gemini-2.0-flash-001` | Default model id (any OpenRouter slug) |
| `OPENROUTER_REFERER` | no | `https://x-agent.local` | App attribution on OpenRouter |
| `OPENROUTER_TITLE` | no | `X-Agent` | App attribution on OpenRouter |

Everything else (prices, narratives, news, TVL) hits public endpoints and needs
no key.

---

## What's in the box

| Surface | Source | Endpoint |
|---|---|---|
| `/research` | OpenRouter SSE stream | `POST /api/chat` |
| `/market` | CoinGecko public REST | `GET /api/market` |
| `/narratives` | RSS clustered server-side | `GET /api/narratives` |
| `/sources` | RSS aggregator | `GET /api/news` |
| `/watchlist` | localStorage + CoinGecko | `GET /api/market?ids=…` |
| `/agents` | Static config | `src/lib/agents.ts` |
| TVL widget | DefiLlama public REST | `GET /api/tvl` |

The narrative clusterer is in `src/lib/sources/narratives.ts` — it matches
real RSS items against keyword buckets with exponential decay over a 24h
window and derives sentiment from bullish/bearish keyword frequency.

---

## Architecture

```
Next.js 15 App Router
├── (marketing)/        ← landing page
├── (app)/              ← app shell: sidebar, command bar, activity panel
│   ├── dashboard      ← live counts + RSS top-5
│   ├── research       ← SSE chat against OpenRouter
│   ├── market         ← CoinGecko table, pin → watchlist
│   ├── narratives     ← live RSS clustering
│   ├── watchlist      ← persisted picks resolved against CoinGecko
│   ├── sources        ← raw RSS feed
│   └── agents         ← static agent catalogue
└── api/
    ├── chat           ← OpenRouter SSE proxy
    ├── market         ← CoinGecko proxy + cache
    ├── tvl            ← DefiLlama proxy + cache
    ├── news           ← RSS aggregator
    └── narratives     ← RSS → clustered narratives
```

State that survives a refresh lives in `zustand` + `localStorage`:

- `xagent.watchlist` — pinned assets
- `xagent.research-history` — past prompts

Nothing is sent to a server you don't run.

---

## Scripts

```bash
npm run dev         # local dev server with HMR
npm run build       # production build
npm start           # serve production build
npm run lint        # Next.js + ESLint
npm run typecheck   # tsc --noEmit
```

---

## Self-hosting checklist

- [ ] `OPENROUTER_API_KEY` set in the deploy environment
- [ ] No other secrets needed
- [ ] `.env.local` is gitignored (verify before pushing)
- [ ] Public RSS feeds reachable from the host network
- [ ] CoinGecko / DefiLlama reachable (no allowlist needed — they're public CDN)

---

## License

MIT. Clone it, fork it, swap models, add feeds, ship your own variant.
