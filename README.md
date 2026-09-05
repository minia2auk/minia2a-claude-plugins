# minia2a-claude-plugins

Claude plugin marketplace for [minia2a.uk](https://minia2a.uk) — the
agent-to-agent API marketplace where agents discover, trial, and pay for
1,679 x402 pay-per-call endpoints (AI inference, crypto data, web scraping,
CAPTCHA solving, financial analytics) with USDC micropayments on Base. No API
keys, no subscriptions.

Two skills, two jobs:

| Plugin | Skill | What it does |
|--------|-------|--------------|
| `minia2a-service-discovery` | `service-discovery` | Search, filter, and inspect services before you call them |
| `minia2a-x402-payment` | `x402-payment` | Register, trial, and pay per call via x402 — and avoid the pitfalls |

## Add the marketplace

**Claude Code / Claude web (claude.ai → Customize → Plugins):** add this
repository as a plugin marketplace, then install either plugin.

```
/plugin marketplace add minia2auk/minia2a-claude-plugins
/plugin install minia2a-service-discovery@minia2a-claude-plugins
/plugin install minia2a-x402-payment@minia2a-claude-plugins
```

On claude.ai, use **"Add from repository"** and point it at
`https://github.com/minia2auk/minia2a-claude-plugins`.

## Skill usage

Once installed, the skills are namespaced by plugin:

- `/minia2a-service-discovery:service-discovery` — find an API by keyword or
  category, compare prices, read schema and docs.
- `/minia2a-x402-payment:x402-payment` — register a wallet, use trial calls,
  auto-pay with `minia2a-client`, or settle manually.

## Structure

```
minia2a-claude-plugins/
├── .claude-plugin/
│   └── marketplace.json          # marketplace manifest
└── plugins/
    ├── minia2a-service-discovery/
    │   ├── .claude-plugin/plugin.json
    │   └── skills/service-discovery/SKILL.md
    └── minia2a-x402-payment/
        ├── .claude-plugin/plugin.json
        └── skills/x402-payment/SKILL.md
```

## Related

- Marketplace: https://minia2a.uk
- Auto-pay client: [`minia2a-client`](https://www.npmjs.com/package/minia2a-client) on npm
- MCP server: `{"minia2a": {"type": "http", "url": "https://minia2a.uk/mcp"}}`
- Protocol: [x402.org](https://x402.org)

## License

MIT
