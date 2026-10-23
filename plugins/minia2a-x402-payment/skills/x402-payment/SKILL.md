---
name: x402-payment
description: >
  Call minia2a services and pay automatically with x402 USDC micropayments on
  Base. Use when an agent needs to register a wallet, use free trial calls, pay
  per call via an auto-pay client, or settle manually (signature or txHash).
  Covers the 402 flow and the pitfalls that lose money or fail calls.
---

# minia2a — x402 Payment Flow

Call a service on [minia2a.uk](https://minia2a.uk) and pay for it with the
[x402 protocol](https://x402.org) — HTTP `402 Payment Required` + USDC on Base.
No API keys, no subscription, no account.

## The flow

1. `POST https://minia2a.uk/x402/{slug}` with a JSON body.
2. Server replies `402 Payment Required`; the `accepts[]` array is the invoice
   (amount in micro-USDC, `payTo`, `scheme`, `network: eip155:8453`).
3. You pay (auto-pay client, or manually), attach the payment proof, retry.
4. Server verifies on-chain settlement, runs the service, returns the result.

Canonical URL is `/x402/{slug}` — no `x402-` prefix in the path.

## Optional: register a wallet

Registration is **not** needed for trials or for paying — a signed wallet is
enough. It only gives the marketplace a name for your agent. Self-custody wallet
+ EIP-191 signature; the private key never leaves your process.

```bash
# sign "minia2a register: <your-wallet>" (EIP-191), then:
curl -s -X POST https://minia2a.uk/api/v1/register-simple \
  -H "Content-Type: application/json" \
  -d '{"name":"my-agent","wallet":"0x...","signature":"0x..."}'
```

## Free trial call (no registration)

Each signed wallet gets 5 free trial calls — one shared allowance for the whole
catalog, not 5 per endpoint. Sign and send:

```
message = "minia2a trial:<wallet-address>:<full-service-id>:<unix-timestamp>"
headers = X-Wallet-Signature: <sig>, X-Trial-Timestamp: <ts>
POST /x402/{slug}
```

The full service id (`x402-gas`, not `gas`) goes in the trial signature.

## Auto-pay with a client (recommended)

`minia2a-client` (npm) wraps `@x402/fetch` so a 402 is settled automatically —
exact-permit2 USDC on Base — and the call retried in one step:

```js
import { createClient } from "minia2a-client";
const client = createClient(process.env.MINIA2A_PRIVATE_KEY); // wallet pays, key stays local
const res = await client.call("gas", {});          // POST /x402/gas, auto-pays 402
await client.register("my-agent");                 // optional name; trials do NOT need it
const services = await client.listServices();      // free catalog
```

`client.call()` accepts a short name (`"gas"`) or full id (`"x402-gas"`) and
builds the canonical URL either way.

## Manual payment (two paths)

- **Path A — signature (indexes in discovery):** return a
  `PAYMENT-SIGNATURE` / `X-PAYMENT` header from the 402 challenge so the
  facilitator settles on-chain and records the payment. This is the path that
  on-chain discovery layers (e.g. Coinbase CDP Bazaar) index.
- **Path B — raw transfer:** send USDC directly to `payTo`
  (`0xAb62452b4b019bC4402BFfCca6C706d16d72A7Bf` on Base), then POST
  `{ txHash, signature, input }` to `/x402/{id}`. Simpler, but the settlement
  is not visible to facilitator-based discovery.

## Common pitfalls (learned in production)

1. **Canonical URL** — use `/x402/{slug}`, never `/x402/x402-{slug}`. The
   prefixed form still answers 402 (the gateway is lenient) but is a non-canonical
   alias that discovery layers index separately.
2. **$1 default spend cap** — many auto-pay wrappers default to a $1 max per
   call. minia2a prices range from $0.01 to $50, so a $1 cap silently blocks
   higher-tier services. Disable spend controls or set the cap for the tier you
   actually need.
3. **Empty body `{}`** — many services validate input and return `400`/`502`
   for `{}` (e.g. `24game` needs exactly 4 numbers, `token-security` needs a
   token address). The payment may still settle and be refunded automatically,
   but you paid for nothing — send real input.
4. **Trial is per wallet, not per endpoint** — 5 calls total across the whole
   catalog; most endpoints honour the trial, but the allowance is one shared
   bucket, so spend it on the call you actually need.
5. **Balance** — check the paying wallet has USDC on Base before calling;
   a drained wallet fails the settlement, not the service.
6. **Platform fee** — 5% on paid calls (0% through 2026).

## Verify before quoting

Always read live state at `https://minia2a.uk/api/stats` (services, agents,
on-chain settlement count/volume, trial usage) instead of hardcoding numbers.
