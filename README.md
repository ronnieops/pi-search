# pi-search

Unified web search extension for [pi](https://pi.dev) with **9 backend providers** (all working). One `web_search` tool, auto-fallback between backends.

## Installation

```bash
pi install npm:pi-search
```

## Usage

After installing, just ask naturally:

```text
Search for recent AI agent frameworks.
```

```text
What's the latest news on Llama 4?
```

Or call the tool directly via `web_search` — the agent picks the best configured backend automatically.

## Supported Backends

| # | Backend               | Free Tier                | API Key? | How to get key                                                    |
| - | --------------------- | ------------------------ | :------: | ----------------------------------------------------------------- |
| 1 | **DuckDuckGo**        | Unlimited (rate-limited) |  **No**  | Nothing needed                                                    |
| 2 | **Marginalia Search** | Unlimited (rate-limited) | **No**†  | [marginalia.nu](https://www.marginalia.nu/marginalia-search/api/) |
| 3 | **Tavily**            | 1,000 calls/month        |   Yes    | [tavily.com](https://tavily.com)                                  |
| 4 | **Serper** (Google)   | 2,500 queries/month      |   Yes    | [serper.dev](https://serper.dev)                                  |
| 5 | **Brave**             | 2,000 queries/month      |   Yes    | [brave.com/search/api](https://brave.com/search/api)              |
| 6 | **Firecrawl**         | 500 free credits         |   Yes    | [firecrawl.dev](https://www.firecrawl.dev)                        |
| 7 | **Exa**               | 10 QPS rate-limited      |   Yes    | [exa.ai](https://dashboard.exa.ai/api-keys)                       |
| 8 | **LangSearch**        | Genuinely free, no CC    |   Yes    | [langsearch.com](https://langsearch.com)                          |
| 9 | **WebSearchAPI.ai**   | 2,000 free credits       |   Yes    | [websearchapi.ai](https://www.websearchapi.ai)                    |

> † Marginalia Search uses `public` as a shared API key — no registration required, but subject to a shared rate limit.

**Removed:** Stract, UnSearch, BoardReader, EntireWeb, Search1API, FreeAPITools.dev — no longer viable (public API removed, requires payment, or endpoint not implemented).

## Benchmark Results (2026-05-04)

**All 9 backends confirmed working** across 3 test queries. All backends returning results were scored for relevance quality (0-10).

> Latest benchmark run: 2026-05-04T18:34 UTC. Full report in [`benchmark/benchmark-report.md`](benchmark/benchmark-report.md).

**How Quality is scored:** Each result is evaluated for keyword relevance (query words matched in title/snippet), source diversity (penalty for generic search engines), and snippet completeness. The average per-result score is then normalized to a 0–10 scale. Time is shown for reference only — it is not a factor in the quality score.

### 🏆 Working Backends

| Backend               | Avg Time  |  Quality   |               Status               |
| --------------------- | :-------: | :--------: | :--------------------------------: |
| **Tavily**            |   356ms   | **3.7/10** |   ✅ Best quality, rich content    |
| **DuckDuckGo**        |  1158ms   |   3.5/10   |     ✅ Reliable, no key needed     |
| **Serper**            |   667ms   |   3.5/10   |         ✅ Google results          |
| **Firecrawl**         |   644ms   |   3.5/10   |    ✅ Search + crawl + extract     |
| **Brave**             |   460ms   |   3.5/10   |      ✅ Fast (~1 req/s free)       |
| **Exa**               | **137ms** |   3.2/10   |        ✅ AI-native search         |
| **Marginalia Search** |   354ms   |   3.0/10   |     ✅ Fastest no-key backend      |
| **LangSearch**        |  1816ms   |   3.2/10   |   ✅ 10 results/query, free tier   |
| **WebSearchAPI.ai**   |  1323ms   |   3.5/10   | ✅ Google-powered, 2K free credits |

## Configuration

Configure backends globally (all projects) or per-project:

**Global:** `~/.pi/agent/extensions/search.json`
**Project:** `.pi/search.json` (project takes precedence)

```json
{
  "defaultBackend": "auto",
  "backends": {
    "duckduckgo": { "enabled": true },
    "marginalia": { "enabled": true },
    "serper": { "enabled": true, "apiKey": "your-serper-key" },
    "tavily": { "enabled": true, "apiKey": "your-tavily-key" },
    "brave": { "enabled": true, "apiKey": "your-brave-key" },
    "exa": { "enabled": true, "apiKey": "your-exa-key" },
    "firecrawl": { "enabled": true, "apiKey": "your-firecrawl-key" },
    "langsearch": { "enabled": true, "apiKey": "your-langsearch-key" },
    "websearchapi": { "enabled": true, "apiKey": "your-websearchapi-key" }
  }
}
```

See [`search.json.example`](search.json.example) for a full template.

Or use the interactive setup:

```
/search-setup
```

## Commands

| Command          | Description                                              |
| ---------------- | -------------------------------------------------------- |
| `/search-setup`  | Interactive prompt to configure API keys for any backend |
| `/search-status` | Show which backends are active and which have keys       |

## How auto mode works

1. Tries each enabled backend in order from your config
2. If a backend fails (rate limit, auth error, etc.), moves to the next one
3. DuckDuckGo requires no API key and is always included as a safety net
4. Returns results from the first backend that succeeds
5. If all backends fail, reports the collected errors

## Security

- API keys are stored in local config files only (`~/.pi/agent/extensions/search.json` or `.pi/search.json`), never sent to any third party besides the chosen backend
- DuckDuckGo queries are executed via temp-file Python scripts (no shell injection surface)
- All HTTP backends have a 30-second timeout to prevent hanging requests
- Error messages are sanitized — API response bodies are truncated and key-like patterns are redacted before being returned
- The `.pi/` directory is in `.gitignore` — **never commit API keys to version control**

## Testing

```bash
# Run the full benchmark against all backends
node benchmark/benchmark.mjs

# Quick test via curl with your configured key
curl -X POST "https://api.exa.ai/search" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $KEY" \
  -d '{"query": "test", "numResults": 3, "contents": {"text": true}}'
```

## Adding a new backend

Backends are just async functions that return `{ results: [{ title, url, snippet }] }`. See `extensions/search.ts` for examples.

## License

MIT

---

<p align="center">Proudly created with <a href="https://pi.dev">pi</a></p>
