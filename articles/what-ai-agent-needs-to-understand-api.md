---
title: "AIエージェントがAPIを理解するために必要な8つのコンテキスト層"
emoji: 🤖
type: tech
topics: ai, api, openapi, agent
canonical_url: https://agentbadge.xyz/blog/what-ai-agent-needs-to-understand-api
cover_image: https://agentbadge.xyz/images/blog/what-ai-agent-needs-understand-api/1s.webp
published: true
---

<!-- ZENN PUBLISHING NOTES
- Canonical URL: https://agentbadge.xyz/blog/what-ai-agent-needs-to-understand-api
- Topics: ai, api, openapi, agent (max 5)
- Body: English (unchanged from Hashnode version)
- Local intro: Japanese, 2-3 sentences before the English body
- Zenn: GitHub sync or web editor. No API yet.
- Image: hosted at agentbadge.xyz — Zenn supports external image URLs
- Publish via: WORKING-SMI/Zenn/Scripts/publish.sh what-ai-agent-needs-to-understand-api "Article 6: What Does an AI Agent Actually Need to Understand an API?"
-->

![cover](https://agentbadge.xyz/images/blog/what-ai-agent-needs-understand-api/1s.webp)

# What Does an AI Agent Actually Need to Understand an API?

## AIエージェントがAPIを理解するために実際に必要なもの

> 人間向けに完璧に文書化されたAPIでも、AIエージェントにはほぼ使えないことがあります。OpenAPIはインターフェースを記述しますが、エージェントには「意図レベルの説明」「機械可読な認証」「エラー回復ヒント」「安全性分類」など、より多くのコンテキストが必要です。この記事では、自律型エージェントがAPIを発見・理解・実行するために必要な8つのコンテキスト層を特定します。日本の開発者にとって、OpenAPIの先にある「エージェント可読性」とは何かを考察します。

---

An API can be perfectly documented for humans and still be nearly impossible for an AI agent to use.

OpenAPI describes the interface — paths, methods, schemas. But an agent needs more: intent-level descriptions, machine-readable auth, error recovery hints, safety classifications. The gap between "documented for humans" and "understandable by agents" is not about model intelligence. It's about missing context layers.

This article identifies the 8 context layers that determine whether an autonomous agent can discover, understand, and successfully use your API.

---

## The Agent Context Flow

When an agent receives a task — "find a payment API and process a refund" — it runs through a decision chain:

```text
Agent
  ↓
"Where is the API?"          → Discovery
  ↓
"What can I do here?"        → Capabilities
  ↓
"What do I need to provide?" → Inputs
  ↓
"Do I have permission?"      → Authentication
  ↓
"What does this mean?"       → Semantics
  ↓
"What will I get back?"      → Output
  ↓
"What if something breaks?"  → Errors
  ↓
"Is it safe to do this?"     → Safety
  ↓
SUCCESS / FAILURE
```

Each layer is a potential failure point. A human developer compensates with experience and intuition. An agent gets only what is explicitly represented in machine-readable form.

![Hero — Agent context flow: 8 layers from Discovery to Safety](https://agentbadge.xyz/images/blog/what-ai-agent-needs-understand-api/1s.webp)

---

## 1. Discovery — "What is this API?"

An agent cannot use an API it cannot find. Machine-readable discovery is the first layer.

**Bad:** No `llms.txt`, no `.well-known` endpoints, no `ai-sitemap.xml`. The API is invisible to autonomous discovery. A human might Google it. An agent operating in a pipeline cannot.

**Better:** `llms.txt` at root with API summary. `/.well-known/openapi` or `/.well-known/service-desc` for spec discovery. `ai-sitemap.xml` listing API endpoints. `link rel="service"` from the homepage.

**Why agents care:** Without discovery, the agent stops at step one. It doesn't matter how good your OpenAPI is if the agent can't find it. Discovery is the prerequisite for all subsequent layers.

---

## 2. Capabilities — "What can I do here?"

Agents plan actions at the intent level, not the HTTP method level. `POST /orders` — is that creating, updating, or processing?

**Bad:** Bare endpoint listing. Agent sees HTTP methods but doesn't understand intent. It can call the endpoint but doesn't know what it accomplishes.

**Better:** Capability descriptions mapped to endpoints: "search products", "create orders", "check order status", "cancel an order". Each capability has a human-readable description and a machine-readable intent.

**Why agents care:** Agents decompose tasks into sub-goals. "Process a refund" becomes: find order → check status → issue refund. Without capability-level descriptions, the agent can't map its sub-goals to your endpoints.

---

## 3. Inputs — "What do I need to provide?"

Agents cannot read between the lines. Empty `description: ""` means the agent doesn't know what to send.

**Bad:**

```yaml
customer_id:
  type: string
  description: ""
```

**Better:**

```yaml
customer_id:
  type: string
  format: uuid
  description: "UUID of an existing customer, obtained from GET /customers"
  example: "550e8400-e29b-41d4-a716-446655440000"
```

**Why agents care:** Without descriptions, the agent guesses. It might send a customer email instead of a UUID. It might omit required fields. Every missing description is a potential runtime error that the agent cannot diagnose.

---

## 4. Authentication — "Do I have permission?"

Authentication is one of the top failure causes for agents. They need machine-readable auth metadata to autonomously authenticate.

**Bad:** Human OAuth docs with browser redirect flows. The agent cannot execute browser steps. It gets a 401 and stops.

**Better:** `securitySchemes` in OpenAPI with full flow descriptions. `/.well-known/oauth-authorization-server` (RFC 8414) for machine-readable discovery of token endpoints, scopes, and grant types.

**Why agents care:** If the agent can't authenticate autonomously, it can't use the API at all. Browser-based OAuth flows are designed for humans clicking "Authorize". Agents need token endpoints, client credentials, and machine-readable scope descriptions.

---

## 5. Semantics — "What does this operation actually mean?"

This is critical for autonomous agents: is the operation safe? Can it be retried? Are there side effects? Does it charge money?

**Bad:**

```yaml
POST /api/v2/process:
  summary: "Process"
  description: ""
```

**Better:**

```yaml
POST /api/v2/process:
  x-agent-semantics:
    operation: create
    side-effects: true
    idempotent: false
    charges-money: true
    safe-to-retry: false
```

**Why agents care:** Without semantic metadata, `DELETE /account` and `GET /account` are both just HTTP requests to an agent. But the risk is entirely different. Agents need to know: can I retry this? Will retrying double-charge the customer? Is this destructive?

![Evolution: Human-readable → Machine-readable → Agent-readable](https://agentbadge.xyz/images/blog/what-ai-agent-needs-understand-api/3s.webp)

---

## 6. Output — "What will I get?"

Agents need action chains. Not just "what came back" but "what to do next."

**Bad:**

```yaml
responses:
  '200':
    description: "OK"
    schema:
      type: object
```

**Better:**

```yaml
responses:
  '200':
    description: "Order created successfully"
    schema:
      type: object
      properties:
        id:
          type: string
          format: uuid
          description: "Order ID for tracking"
        status:
          type: string
          enum: [pending, confirmed, shipped]
        next_actions:
          type: array
          items:
            type: object
            properties:
              action:
                type: string
                enum: [confirm, cancel, track]
              endpoint:
                type: string
```

**Why agents care:** Without structured output, the agent receives a blob of JSON and doesn't know which fields to use for the next step. `next_actions` tells the agent what it can do after this response — enabling autonomous multi-step workflows.

---

## 7. Errors — "What if something goes wrong?"

Good agent APIs describe not only how to succeed but how to recover. Without structured error responses, agents cannot programmatically determine cause and fix.

**Bad:**

```json
400 Bad Request
{"error": "invalid_request"}
```

**Better:**

```json
{
  "type": "https://agentbadge.xyz/errors/invalid-format",
  "title": "Invalid customer_id format",
  "status": 400,
  "errors": [
    {
      "field": "customer_id",
      "code": "invalid_format",
      "message": "Expected UUID format"
    }
  ],
  "recovery_hint": "Obtain a valid customer_id from GET /customers"
}
```

**Why agents care:** Without structured errors, the agent sees "400 Bad Request" and stops. It doesn't know which field was wrong or how to fix it. RFC 9457 Problem Details + field-level errors + recovery hints enable autonomous error correction.

![Error recovery flow: 401 → refresh, 403 → request permission, 404 → missing, 429 → retry, 500 → backoff](https://agentbadge.xyz/images/blog/what-ai-agent-needs-understand-api/5s.webp)

---

## 8. Safety — "Is it safe to do this?"

`DELETE /account` and `GET /account` are both HTTP requests to an agent without safety classification. But the risk is entirely different.

**Bad:** No safety classification. Agent treats all operations the same. It might retry a destructive operation because it got a timeout.

**Better:**

```yaml
x-agent-safety:
  risk-level: financial
  reversible: false
  requires-confirmation: true
  warning: "This action permanently deletes the account"
```

Safety levels: `read-only` → `write` → `destructive` → `financial` → `irreversible`.

**Why agents care:** Agents retry on timeouts. If a `DELETE` operation is retried, data is lost. Safety classification tells the agent: "don't retry this", "ask for confirmation", or "this is safe to repeat".

![Safety classification: 5 risk levels from read-only to irreversible](https://agentbadge.xyz/images/blog/what-ai-agent-needs-understand-api/4s.webp)

---

## Version A vs Version B

Consider two APIs with identical OpenAPI structure:

**Version A — OpenAPI only:**

- Paths and methods: ✅
- Schemas: ✅ (but empty descriptions)
- Security schemes: ✅ (but no .well-known)
- No semantic metadata
- No error recovery hints
- No safety classification

**Version B — OpenAPI + Agent Context:**

- Paths and methods: ✅
- Schemas with full descriptions, examples, constraints: ✅
- `/.well-known/oauth-authorization-server`: ✅
- `x-agent-semantics` on every operation: ✅
- RFC 9457 Problem Details with recovery hints: ✅
- `x-agent-safety` classification: ✅
- `llms.txt` with API summary: ✅

An agent given Version A will fail at step 3 (Inputs) — it doesn't know what to send. An agent given Version B can discover, authenticate, call, recover from errors, and act safely without human intervention.

![Version A vs Version B: sparse spec vs rich agent context](https://agentbadge.xyz/images/blog/what-ai-agent-needs-understand-api/2s.webp)

The difference is not the model. The difference is the context.

---

## This Is Agent Readiness

These 8 context layers are not a wish list. They are measurable properties. [Agent Readiness](https://agentbadge.xyz/blog/what-is-agent-readiness) is the framework that measures whether an API provides sufficient context for autonomous use.

Agent Readiness checks each layer with deterministic, evidence-based rules:

- **Discovery:** Does `llms.txt` exist? Does `/.well-known/openapi` resolve?
- **Capabilities:** Are operation descriptions non-empty and intent-level?
- **Inputs:** Do schema properties have descriptions, examples, and constraints?
- **Authentication:** Is `securitySchemes` populated? Does `.well-known/oauth-authorization-server` exist?
- **Semantics:** Are `x-agent-semantics` or equivalent extensions present?
- **Output:** Do responses include full schemas with `next_actions`?
- **Errors:** Are error responses structured (RFC 9457) with recovery hints?
- **Safety:** Is `x-agent-safety` or equivalent classification present?

72 checks in seconds. Free, no signup.

```bash
npx @agentbadge/cli scan https://api.example.com
```

![Agent context layers stack: 8 building blocks from Discovery to Safety](https://agentbadge.xyz/images/blog/what-ai-agent-needs-understand-api/6s.webp)

---

## What's Next

This article defined the 8 context layers. The next question is: **can we measure them?**

In the next article — "Can We Measure Agent Readiness?" — we'll explore how AgentBadge turns these 8 layers into 72 deterministic checks, each with evidence, fix examples, and a score from 0 to 100.

---

## Related Articles

- [What Is Agent Readiness?](https://agentbadge.xyz/blog/what-is-agent-readiness) — Article 1: the foundational concept
- [API Has SEO Agent Readiness](https://agentbadge.xyz/blog/api-has-seo-agent-readiness) — Article 2: SEO vs agent discovery
- [The Web Is Becoming Agentic](https://agentbadge.xyz/blog/web-becoming-agentic-api-discovery) — Article 3: agentic web and API discovery
- [From SEO to GEO to Agent Readiness](https://agentbadge.xyz/blog/from-seo-to-geo-to-agent-readiness) — Article 4: evolution of optimization
- [Why AI Agents Fail to Use APIs](https://agentbadge.xyz/blog/why-ai-agents-fail-to-use-apis) — Article 5: 7 failure modes these 8 layers solve

---

*Don't certify. Measure.*

> Originally published at [AgentBadge](https://agentbadge.xyz/blog/what-ai-agent-needs-to-understand-api)
