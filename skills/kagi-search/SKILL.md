---
name: kagi-search
description: Kagi Search MCP server for Hermes. High-quality, ad-free web search with multiple workflows (general, news, videos, podcasts, images) and page extraction.
tags: [search, kagi, mcp, web]
version: 1.1.0
---

# Kagi Search MCP Server

Sets up [Kagi](https://kagi.com) as an MCP search tool in Hermes. Kagi provides high-quality, ad-free search results with no SEO spam.

## Prerequisites

- A Kagi account with API access (requires Kagi Ultimate plan or API add-on)
- API key from https://kagi.com/settings?p=api

## Setup

1. **Set your API key in `.env`:**

   ```bash
   echo 'KAGI_API_KEY=your_key_here' >> ~/.hermes/.env
   ```

2. **Add the MCP server:**

   ```bash
   hermes mcp add kagi --command uvx --args kagimcp
   ```

   When prompted:
   - Authentication? → `n` (Kagi uses the env var, not OAuth)
   - Enable all tools? → `Y`

3. **Verify:**

   ```bash
   hermes mcp test kagi
   ```

   Should show 2 tools discovered. Start a new session (`/reset`) to use them.

## Available Tools

Hermes prefixes MCP tool names with `mcp_<server-name>_`, so the Kagi tools appear as:

| Tool | Purpose |
|------|---------|
| `mcp_kagi_kagi_search_fetch` | General web search. Use `workflow` param for: `search` (default), `news`, `videos`, `podcasts`, `images` |
| `mcp_kagi_kagi_extract` | Extract a page's content as markdown from a URL |

### Search Parameters

Key params for `kagi_search_fetch`:
- `query` (required): search query
- `workflow`: `search`, `news`, `videos`, `podcasts`, `images`
- `limit`: max results (default 10)
- `extract_count`: number of top results to fetch full page content for inline (0-10)
- `include_domains` / `exclude_domains`: domain filtering
- `lens_id`: apply a Kagi lens (e.g., `2` for Academic, `15` for Programming, `1` for Forums)
- `time_relative`: `day`, `week`, `month` — restrict to recent results
- `after` / `before`: ISO date filters

### Extraction Limitation

The Kagi API key is **search-only** by default — `kagi_extract` only works if your plan includes extraction. For full-page content, use `web_extract` (Firecrawl) or Brave's `llm_context` as alternatives.

## Search Strategy (Recommended)

If you have multiple search tools, use them for their strengths:

| Use case | Tool |
|----------|------|
| General web search | Kagi (`mcp_kagi_kagi_search_fetch`) |
| Local business/places | Brave (`mcp_brave_brave_local_search`) |
| Image search | Brave (`mcp_brave_brave_image_search`) |
| News-specific queries | Kagi with `workflow: "news"` or Brave News |
| Full page extraction | `web_extract` or Brave `llm_context` |
| Last resort | `web_search` (Firecrawl) |

## Pitfalls

- **Secret redaction eats `--env` values.** Do NOT pass `--env KAGI_API_KEY=your_key` to `hermes mcp add` — Hermes redacts credential-like strings and writes literal `***` to `config.yaml` instead of the real key. Put the key in `~/.hermes/.env` instead, and either omit `--env` (the MCP server inherits from `.env` automatically) or use `--env KAGI_API_KEY=${KAGI_API_KEY}` for an env var reference.
- Kagi API key is search-only by default. Extraction requires a higher plan tier.
- `extract_count` inline extraction counts against your Kagi API quota.
- `lens_id` is mutually exclusive with `include_domains`/`exclude_domains`/`time_relative`/`file_type`.
- The `kagimcp` package is pulled via `uvx` — no separate install needed. It runs from PyPI on each invocation.
