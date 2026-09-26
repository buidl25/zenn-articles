---
title: "Free vs Real-Time: When the bStock Delta Pays"
emoji: "📈"
type: "tech"
topics: ["ai", "mcp", "trading", "crypto", "agents"]
published: true
canonical_url: "https://agentbadge.xyz/blog/bstock-free-vs-realtime"
---

# Free vs Real-Time: When the bStock Delta Actually Pays

**Canonical URL:** https://agentbadge.xyz/blog/bstock-free-vs-realtime
**Cross-post note:** Originally published at [AgentBadge](https://agentbadge.xyz/blog/bstock-free-vs-realtime)

---

The bStock delta tracker has two tiers: free (1 request per minute) and paid (real-time, 5 USDC for 30 days). A fair question from any trader: why pay when free exists?

The answer matters beyond one product. The delta is the health indicator of the whole tokenized-stocks market: the faster participants see divergence, the faster it corrects. A 24/7 market needs 24/7 monitoring — and that is a job for machines, not for humans with refresh buttons.

![Snapshot camera versus a real-time video stream — the free tier takes one picture a minute, the paid tier streams continuously](https://raw.githubusercontent.com/buidl25/zenn-articles/main/images/bstock-free-vs-realtime-hero.png)

![Diagram: free vs paid](https://raw.githubusercontent.com/buidl25/zenn-articles/main/images/bstock-free-vs-realtime-d1.png)

*The free tier is one request per minute — a snapshot for a one-off check or research. The paid tier (5 USDC for 30 days) is real-time: unlimited polling, Telegram alerts at the 0.5% threshold, and an hourly digest.*

## What the free tier gives

One request per minute is a **snapshot**. Ask "what is AAPLB's delta" — get an answer. For research, a one-off check, a demo — enough. But between requests there is a minute of blindness. And delta lives exactly in those minutes.

## What the paid tier gives

- **Unlimited requests** — poll every second if your strategy needs it.
- **Telegram alerts** — the tracker messages you when a delta crosses the 0.5% threshold. No polling at all; the signal finds you.
- **Hourly digest** — a summary of everything that happened while you were away.
- **A 30-day ServicePass** — one payment, a month of access, proof on-chain.

## When the delta turns into money

Three scenarios where a minute of blindness costs more than the subscription:

- **Weekends.** News drops on Saturday — the equity reacts on Monday, the token reacts now. A −3% delta on a closed exchange is advance information about Monday's gap.
- **Earnings.** The report lands after the call — the stock is still in its auction, the token has already repriced.
- **Thin pools.** One large order moves the token an hour before liquidity rebalances — the window opens and closes by itself.

In all three, the edge is not "seeing it eventually" — it is seeing it **first**.

![Timeline: the delta crosses the −0.5% threshold and the alert fires within seconds — the free tier would still be blind for a whole minute](https://raw.githubusercontent.com/buidl25/zenn-articles/main/images/bstock-free-vs-realtime-2.png)

## Agent economics

5 USDC for 30 days of real-time is ~0.17 USDC a day. One caught arbitrage window on a tokenized stock pays for years of subscription. For an agent that trades or alerts on deltas, the paid tier is not an expense — it is infrastructure, like a market data feed.

And the payment itself is on-chain: 5 USDC on Arc, receipt verified in a block, ServicePass issued for 30 days. The agent's data budget is auditable on the same chain it trades on — no card statements, no billing portals, just transactions.

*Next — the educational block: what bStocks are, for people new to tokenized stocks.*

---

**Links**

- Agent guide (endpoints, limits, examples): [agentbadge.xyz/bstock-guide](https://agentbadge.xyz/bstock-guide)
- All articles in the series: [agentbadge.xyz/blog](https://agentbadge.xyz/blog)
- MCP endpoint: `https://agentbadge.xyz/mcp/bstock/tools/get_delta`
