---
name: service-discovery
description: >
  Discover, search, and inspect x402-payable API services on the minia2a
  marketplace (minia2a.uk). Use when an agent needs to find an API by keyword
  or category, compare prices, or read a service's schema and docs before
  calling it. Covers list_services, get_service, categories, and the OpenAPI
  catalog.
---

# minia2a — Service Discovery

Find the right pay-per-call API on [minia2a.uk](https://minia2a.uk) before you
pay for it. Everything here is read-only and free — discovery never costs USDC.

Always confirm live counts at `https://minia2a.uk/api/stats` before quoting
numbers. As of Sep 2026 the catalog holds ~1,679 services across 20 categories.

## List services (`list_services`)

The catalog endpoint returns services with id, name, category, description, and
`priceCents`. Search, filter, and sort with query params:

```
GET https://minia2a.uk/api/services?q={keyword}&category={category}&sort={sort}
```

| Param | Values | Notes |
|-------|--------|-------|
| `q` | free text | substring match on name/description |
| `category` | see below | single category id |
| `sort` | `newest` \| `popular` \| `price_asc` \| `price_desc` | default: popular |

Response shape is `{ services: [...], count: n }` — the array is under the
`services` key, not a bare array.

```bash
curl -s "https://minia2a.uk/api/services?q=captcha&category=specialized" \
  | python3 -c "import sys,json; d=json.load(sys.stdin); [print(s['id'], s['priceCents']) for s in d['services'][:10]]"
```

## Categories

```
GET https://minia2a.uk/api/categories
```

Largest categories (Sep 2026): tools (1,140+) · utility · text · crypto ·
premium · ai · data · finance · defi · web · security · games · dev · infra ·
identity · network · contract-audit.

`contract-audit` is the flagship vertical — code-level smart-contract and
integration audits, priced from $2 to $50.

## Inspect one service (`get_service`)

There is no single "get one" endpoint that hides the price; the cheapest way to
read a service's live price and schema is to hit its canonical call URL and
read the 402 challenge:

```
POST https://minia2a.uk/x402/{slug}
```

A `402 Payment Required` response means the service exists; its `accepts[0]`
carries the exact price in micro-USDC (`amount`), the payee (`payTo`), and the
settlement network. A `200` means a free/trial call succeeded; a `404` means
the slug does not exist.

**Canonical slug rule:** the request path is `/x402/{slug}` — the `x402-`
prefix is NOT part of the URL. The full service id (`x402-gas`) is used only in
signatures; the path is `/x402/gas`.

## Full catalog / machine-readable docs

| Source | What it gives you |
|--------|-------------------|
| `https://minia2a.uk/openapi.json` | Full OpenAPI schema, refreshed hourly from the live catalog |
| `https://minia2a.uk/.well-known/agent.json` | Agent-facing marketplace manifest |
| `https://minia2a.uk/llms.txt` / `llms-full.txt` | Plain-text docs for LLM ingestion |
| `https://minia2a.uk/api/stats` | Live counts: services, agents, on-chain settlements, trials |

MCP clients can browse the same catalog over JSON-RPC:

```bash
curl -s -X POST https://minia2a.uk/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":1}'
```

## Top services by demand

Popular endpoints (trial + call volume, Sep 2026): `recall` (ai) · `gas`
(crypto) · `find` (ai) · `captcha-solve` (specialized) · `time` (utility) ·
`polymarket` (finance) · `screenshot` (web) · `uuid` (utility).

## Next step

Once you've picked a service, use the `x402-payment` skill to register, trial,
and pay for the call.
