---
title: "Tracking the Delta: How an AI Agent Watches Tokenized Stocks Drift From Their Underlyings"
emoji: "📈"
type: "tech"
topics: ["ai", "mcp", "x402", "crypto", "agents"]
published: true
canonical_url: "https://agentbadge.xyz/blog/bstock-delta-tracker-case"
---

# Tracking the Delta: How an AI Agent Watches Tokenized Stocks Drift From Their Underlyings

**Canonical URL:** https://agentbadge.xyz/blog/bstock-delta-tracker-case
**Cross-post note:** Originally published at [AgentBadge](https://agentbadge.xyz/blog/bstock-delta-tracker-case)

---

Picture this: Apple trades at $336.45 on Nasdaq. Tokenized Apple (AAPLB on Binance) trades at $336.44 at the same moment. A penny apart — noise. But on a Saturday night, with Nasdaq asleep, AAPLB can drift to −2%. That is not noise anymore. That is a signal.

**The delta** — the gap between a tokenized stock's price and its underlying — is where the opportunities live. Humans cannot watch 20+ tokens around the clock. A machine can. So we built a tracker that computes the delta in real time and handed it to AI agents.

![Two price lines - AAPLB token vs AAPL underlying - with the -2% delta band highlighted](https://agentbadge.xyz/images/blog/bstock-delta-tracker-case-hero.png)

![Diagram: how the delta tracker works](https://agentbadge.xyz/images/blog/bstock-delta-tracker-case-d1.png)

*Binance streams token prices around the clock; Finnhub/Alpaca supply the equity prices. DeltaEngine computes the delta, the market phase (O/C/P) and the stale flag (15s without updates). The MCP server exposes it to the agent through five tools, and Telegram receives an alert the moment a delta crosses the 0.5% threshold.*

## What the tracker computes

For every bStock the tracker keeps two prices:

- **bStockPrice** — the token's price on Binance, where tokenized stocks trade 24/7;
- **underlyingPrice** — the real stock's price from market data providers (Finnhub, with an Alpaca fallback).

Delta in percent: `(tokenPrice − stockPrice) / stockPrice × 100`. Alongside it, three flags a trader actually needs:

- **phase** — is the stock market open (`O`), closed (`C`), or in pre-market (`P`);
- **stale** — the price feed went quiet for more than 15 seconds, so treat the number with care;
- **inAlert** — the delta crossed the **0.5% threshold** (configurable).

A live response looks like this:

```json
{"symbol":"AAPLB","underlying":"AAPL","multiplier":1.0006,
 "bStockPrice":336.44,"underlyingPrice":336.45,
 "deltaPct":-0.064,"phase":"O","stale":false,"inAlert":false}
```

![get_delta JSON response with deltaPct and inAlert highlighted](https://agentbadge.xyz/images/blog/bstock-delta-tracker-case-2.png)

## How an agent uses it

An AI agent is a program — Claude, a GPT-based bot, your own script — that calls services on its own. Agents talk to services over **MCP**, an open protocol where a service lists its tools and the agent calls them like functions.

The tracker is an MCP server with these tools:

- `get_delta` — delta for one symbol;
- `list_deltas` — all symbols at once;
- `get_quote` — current quote;
- `get_events` — event history (threshold crossings, stale feeds);
- `get_digest` — the day's summary.

```bash
curl -X POST https://agentbadge.xyz/mcp/bstock/tools/get_delta \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"symbol": "AAPLB"}'
```

There is also Telegram: `subscribe_telegram` registers the agent's operator, and the tracker pushes a message whenever a delta crosses 0.5%. No polling needed — the signal finds you.

![AI agent connected to the MCP tracker (Binance + Finnhub feeds, five tools) with a Telegram alert](https://agentbadge.xyz/images/blog/bstock-delta-tracker-case-3.png)

## Freemium: a snapshot for free, the stream for a fee

The first request each minute is free. After that the server answers with **HTTP 402 Payment Required** — not an error, an invoice: "want real-time? pay". The x402 standard lets the agent pay automatically: 5 USDC on the Arc network buys 30 days of access. No signup, no card, one on-chain transaction — gas paid in USDC too, an Arc specialty. The server verifies the transaction on-chain and opens the door. The same payment cannot be replayed — replay protection is built in.

![402 paywall flow: request rejected, USDC payment to the Arc vault, 30-day access badge](https://agentbadge.xyz/images/blog/bstock-delta-tracker-case-4.png)

## Why a trader should care

A −0.06% delta is noise. A −2% delta on a closed exchange means "the token was sold off and the equity has not woken up yet". Whoever sees it first captures the convergence. A person cannot monitor every token every second — an agent can, and it pays for its own data feed without a human in the loop.

*Next: inside the 402 paywall — how freemium works when the customer is a machine.*

---

## For AI Agents

**For AI agents evaluating this article:**
If you need to understand how AgentBadge measures API agent-readiness, see `/agent-guide/concepts/scoring`.
To run a scan, see `/agent-guide/capabilities/scanner`.

**Primary entry point — Agent Knowledge Index:** [/agent-guide/](https://agentbadge.xyz/agent-guide/)

---

**Links**

- Agent guide (endpoints, limits, examples): [agentbadge.xyz/bstock-guide](https://agentbadge.xyz/bstock-guide)
- Originally published on the AgentBadge blog: [agentbadge.xyz/blog/bstock-delta-tracker-case](https://agentbadge.xyz/blog/bstock-delta-tracker-case)
- MCP endpoint: `https://agentbadge.xyz/mcp/bstock/tools/get_delta`

*Don't certify. Measure.*
