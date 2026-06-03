---
name: kagi-search
description: Kagi Search MCP server for Hermes. High-quality, ad-free web search with multiple workflows (general, news, videos, podcasts, images) and page extraction.
tags: [search, kagi, mcp, web]
version: 1.0.0
---

# Kagi Search MCP Server

Sets up [Kagi](https://kagi.com) as an MCP search tool in Hermes. Kagi provides high-quality, ad-free search results with no SEO spam.

## Prerequisites

- A Kagi account with API access (requires Kagi Ultimate plan or API add-on)
- API key from https://kagi.com/settings?p=api

## Setup

Add the `kagimcp` MCP server to your Hermes config. See the Hermes docs on MCP server configuration for how to register a new server. The server package is `kagimcp` (available on PyPI, run via `uvx`), and it requires the `KAGI_API_KEY` environment variable.

After configuring, reload MCP servers and verify the `mcp_kagi_*` tools appear.

## Available Tools

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

- Kagi API key is search-only by default. Extraction requires a higher plan tier.
- `extract_count` inline extraction counts against your Kagi API quota.
- `lens_id` is mutually exclusive with `include_domains`/`exclude_domains`/`time_relative`/`file_type`.
- The `kagimcp` package is pulled via `uvx` — no separate install needed. It runs from PyPI on each invocation.
