---
title: "Freemium for AI Agents: Free vs Paid Tiers via HTTP 402"
emoji: "🤖"
type: "tech"
topics: ["ai", "mcp", "x402", "crypto", "agents"]
published: true
canonical_url: "https://agentbadge.xyz/blog/bstock-freemium-402"
---

# Freemium for AI Agents: Free vs Paid Tiers via HTTP 402 on Arc

**Canonical URL:** https://agentbadge.xyz/blog/bstock-freemium-402
**Cross-post note:** Originally published at [AgentBadge](https://agentbadge.xyz/blog/bstock-freemium-402)

---

Every SaaS has a free tier and a paid tier. But how do you sell the paid tier when the customer is not a person but a program? An agent has no card, cannot fill a checkout form, cannot type a CVV.

We solved it with **HTTP 402 Payment Required** — a status code that waited three decades for its moment. It is not an error. It is an invoice.

Why does this matter beyond our tracker? Tokenized stocks trade around the clock, and the traders who watch them increasingly delegate monitoring to AI agents. An agent that cannot pay for its own data is a crippled market participant. Machine-payable access — priced in USDC, settled on Arc, proven on-chain — turns every agent into a full customer of the market's data infrastructure: no cards, no signups, no humans in the loop.

![A robot inserts a glowing USDC coin into a turnstile marked 402](https://raw.githubusercontent.com/buidl25/zenn-articles/main/images/bstock-freemium-402-hero.png)

![Diagram: the freemium gate](https://raw.githubusercontent.com/buidl25/zenn-articles/main/images/bstock-freemium-402-d1.png)

*Every request passes three doors: a live ServicePass means instant real-time; otherwise the free bucket allows one request per minute (a snapshot); otherwise the server answers 402 with an invoice. One USDC transaction (5 USDC on Arc), an on-chain receipt check — and the agent holds a 30-day ServicePass.*

## How the gate works

Every request to the tracker passes three checks:

1. **Already paid?** If the agent holds a live ServicePass (a 30-day access token) — data flows immediately.
2. **Free allowance?** One request per minute is free. A snapshot, not a stream.
3. **Neither?** The server answers 402 and attaches an invoice to the response: what, to whom, how much.

## Inside the invoice

The 402 response is not just text. The `PAYMENT-REQUIRED` header carries a machine-readable payment description:

```json
{
  "x402Version": 2,
  "accepts": {
    "scheme": "eip3009-client-broadcast",
    "network": "eip155:5042002",
    "asset": "USDC",
    "amount": "5000000",
    "payTo": "0xcdd2...699d",
    "maxTimeoutSeconds": 345600
  }
}
```

![A 402 Payment Required JSON response with the accepts block highlighted](https://raw.githubusercontent.com/buidl25/zenn-articles/main/images/bstock-freemium-402-3.png)

The agent reads it like a price list: the payment scheme (EIP-3009 — a standardized transfer signature), the network (Arc Testnet), the asset (USDC), the amount (5 USDC — the six zeros are decimals, the amount is in base units), the recipient, and the invoice expiry (4 days).

## Why not API keys and billing portals

An API key needs signup, an email, a card, invoices — a human in the loop. 402 + x402 is a *programmable* paywall: the agent sees the price → signs a transfer → pays → gets access. The whole cycle takes seconds. For a machine, this is the native way to buy things.

And because it runs on **Arc** — Circle's blockchain where gas itself is paid in USDC — the agent needs exactly one asset in its wallet. One balance covers the payment and the fee. For a market where bStocks themselves settle in USDC, the plumbing finally matches the asset.

## One payment pays once

After paying, the agent retries the request with the transaction hash attached. The server checks the blockchain — not a claimed signature, but the real receipt from a block — and opens access. The same hash cannot be presented twice: the server atomically claims it, and a second attempt gets `tx_replayed`. Even ten parallel requests with the same hash — exactly one gets through.

A paywall a program can read and pay turns data into a first-class on-chain service. More agents able to buy real-time data means more eyes on tokenized markets — tighter deltas, faster convergence, healthier price discovery for everyone who trades them.

*Next: the payment itself on Arc — and why gas there is paid in USDC.*

---

**Links**

- Agent guide (endpoints, limits, examples): [agentbadge.xyz/bstock-guide](https://agentbadge.xyz/bstock-guide)
- Originally published on the AgentBadge blog: [agentbadge.xyz/blog/bstock-freemium-402](https://agentbadge.xyz/blog/bstock-freemium-402)
- MCP endpoint: `https://agentbadge.xyz/mcp/bstock/tools/get_delta`

