---
title: "x402 Payments on Arc: Agent Pays in USDC, No Facilitator"
emoji: "🤖"
type: "tech"
topics: ["ai", "mcp", "x402", "crypto", "agents"]
published: true
canonical_url: "https://agentbadge.xyz/blog/bstock-arc-x402-payment"
---

# x402 Payments on Arc Testnet: How an Agent Pays in USDC With No Facilitator

**Canonical URL:** https://agentbadge.xyz/blog/bstock-arc-x402-payment
**Cross-post note:** Originally published at [AgentBadge](https://agentbadge.xyz/blog/bstock-arc-x402-payment)

---

In classic x402, a **facilitator** sits between the buyer and the seller: it takes the agent's signature and broadcasts the transaction on its behalf. Convenient — but an extra party to trust, an extra API to wait on, an extra point of failure.

On Arc we skipped it. The scheme is called **self-settle** — "settle it yourself".

![A USDC coin flies from an agent's wallet to a treasury vault on Arc, with a trail of USDC coins paying for gas](https://raw.githubusercontent.com/buidl25/zenn-articles/main/images/bstock-arc-x402-payment-hero.png)

![Diagram: paying on Arc in 6 steps](https://raw.githubusercontent.com/buidl25/zenn-articles/main/images/bstock-arc-x402-payment-d1.png)

*The agent receives the 402 invoice, signs an EIP-3009 authorization (exactly 5 USDC to the treasury) and broadcasts on Arc itself — gas is paid in USDC. The server reads the receipt from the block: the Transfer event reached the treasury — access opens for 30 days.*

## Why Arc makes this possible

Arc is a blockchain built by Circle — the company behind USDC. Its signature feature: **gas is paid in USDC**, not in a separate token. A conventional agent would need to hold two assets: USDC for the payment and ETH for gas. On Arc one balance is enough — USDC covers both the payment and the fee.

For the tokenized-stocks market this closes the loop: bStocks settle in USDC on Binance, the agent's gas is USDC, and the data subscription is USDC. One asset for trading, for fees, and for information — that is what a market built for machines looks like.

## The flow in 6 steps

1. The agent requests data → gets a 402 with the invoice: scheme, network, amount, recipient.
2. It signs an **EIP-3009** authorization — the standard for "transfer with authorization": a signature that permits moving exactly 5 USDC from the agent's wallet to the recipient. Nothing more, no wallet access.
3. It broadcasts the transaction itself (hence "client-broadcast").
4. It waits for confirmation — seconds.
5. It retries the request with the transaction hash attached.
6. The server reads the blockchain: is the tx in a block, did USDC reach the treasury, is the amount right → access for 30 days.

![Six-step pipeline: request, 402 invoice, EIP-3009 signature, broadcast, on-chain receipt, access for 30 days](https://raw.githubusercontent.com/buidl25/zenn-articles/main/images/bstock-arc-x402-payment-2.png)

## What the server actually verifies

Not a signature — a **receipt**. The server asks the chain for `getTransactionReceipt(txHash)` and checks: the transaction is really in a block, it contains a `Transfer` event from the USDC contract to the treasury address, the amount covers the price. This cannot be forged: either the transaction is in a block or it does not exist.

![A block on Arc with a highlighted Transfer log paying 5 USDC to the treasury, inspected by the server](https://raw.githubusercontent.com/buidl25/zenn-articles/main/images/bstock-arc-x402-payment-3.png)

## What this gives the ecosystem

Removing the facilitator removes a point of failure and a trust assumption. Any wallet holding USDC on Arc becomes a payment client: one asset, one signature, one RPC call. For agents, buying data becomes as routine as calling an API.

This is infrastructure for the tokenized-assets ecosystem, not a demo. Real-time delta data is what keeps bStock prices honest — and the agents that buy it settle on Arc in USDC, the same asset the tokens themselves settle in. Every payment is a public, auditable transaction: the market's information layer becomes as transparent as its trading layer.

*Next: when the delta actually pays — free tier vs real-time.*

---

**Links**

- Agent guide (endpoints, limits, examples): [agentbadge.xyz/bstock-guide](https://agentbadge.xyz/bstock-guide)
- Originally published on the AgentBadge blog: [agentbadge.xyz/blog/bstock-arc-x402-payment](https://agentbadge.xyz/blog/bstock-arc-x402-payment)
- MCP endpoint: `https://agentbadge.xyz/mcp/bstock/tools/get_delta`
