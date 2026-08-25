---
title: "Why Your OpenAPI Spec Isn't Enough for AI Agents — OpenAPI仕様がAIエージェントにとって十分ではない理由"
emoji: 🔧
type: tech
topics: ai, api, openapi, agent
canonical_url: https://agentbadge.xyz/blog/why-openapi-isnt-enough
cover_image: https://agentbadge.xyz/images/blog/why-openapi-isnt-enough/1s.webp
published: false
---

<!-- ZENN PUBLISHING NOTES
- Canonical URL: https://agentbadge.xyz/blog/why-openapi-isnt-enough
- Topics: ai, api, openapi, agent (max 5)
- Body: English (unchanged from Hashnode version)
- Local intro: Japanese, 2-3 sentences before the English body
- Zenn: GitHub sync or web editor. No API yet.
- Image: hosted at agentbadge.xyz — Zenn supports external image URLs
- Publish via: WORKING-SMI/Zenn/Scripts/publish.sh why-openapi-isnt-enough "Article 7: Why Your OpenAPI Spec Isn't Enough for AI Agents"
-->

![cover](https://agentbadge.xyz/images/blog/why-openapi-isnt-enough/1s.webp)

# Why Your OpenAPI Spec Isn't Enough for AI Agents

## OpenAPI仕様がAIエージェントにとって十分ではない理由

> OpenAPIはAPIを記述します。Agent Readinessは、エージェントが実際にそのAPIを使用できるかどうかを記述します。完全なOpenAPI仕様があっても、AIエージェントは失敗することがあります。仕様が間違っているからではなく、仕様がインターフェースを記述しており、エージェントの体験を記述していないからです。この記事では、API記述とエージェント理解の間の構造的なギャップを、決済APIの実例を通じて探ります。日本の開発者にとって、OpenAPIの先にある「エージェント可読性」とは何かを考察します。

---

> OpenAPI describes an API. Agent Readiness describes whether an agent can actually use it.

Your API has a complete OpenAPI spec. Every endpoint, schema, and response code is documented. Yet when an AI agent tries to use it, the agent fails — not because the spec is wrong, but because the spec describes an interface, not an agent's experience.

This isn't about OpenAPI being bad. OpenAPI is a necessary foundation. But it's not a complete Agent Readiness layer.

---

## The Provocation

> "Our API has OpenAPI. Why does an AI agent still fail to use it?"

This is the question API teams ask after adding AI agent support. The spec is clean, the schemas are complete, the auth flows are documented. And yet — agents struggle.

The answer isn't that OpenAPI is insufficient as a specification. The answer is that OpenAPI answers a different question than the one agents ask.

OpenAPI answers: **"What endpoints exist?"**

Agents ask: **"Can I discover this API? Can I authenticate autonomously? Can I understand what an operation means? Can I recover from errors? Can I trust that a claim about this API is true?"**

These are different questions. And the gap between them is structural.

---

## One Real Example: A Payments API

Consider a payments API with three endpoints:

```text
POST /payments              — create a payment
GET  /payments/{id}         — retrieve payment status
POST /payments/{id}/refund  — refund a payment
```

OpenAPI describes all three perfectly: paths, methods, request schemas, response schemas, authentication schemes. A human developer reading this spec would understand how to use the API.

But an AI agent needs to answer questions that the spec doesn't address:

```text
Can I create a payment?
When should I call it?
What must happen first?
What does "pending" mean?
When can I refund?
What happens if payment fails?
Should I retry?
```

![Payments API example: three endpoint boxes with agent questions radiating outward as dashed lines](https://agentbadge.xyz/images/blog/why-openapi-isnt-enough/2s.webp)

Each of these questions maps to a layer beyond OpenAPI:

- **"Can I create a payment?"** — Discovery: Is there a `llms.txt` or `.well-known/openapi` so the agent can find the API?
- **"When should I call it?"** — Semantics: Is `POST /payments` idempotent? Does it charge money? Is it safe to retry?
- **"What must happen first?"** — Capabilities: What prerequisites exist? Does the agent need a customer ID first?
- **"What does 'pending' mean?"** — Semantics: What are the possible states and transitions?
- **"When can I refund?"** — Semantics + Safety: Is refund conditional on payment state? Is it reversible?
- **"What happens if payment fails?"** — Errors: Does the API return structured errors with recovery hints?
- **"Should I retry?"** — Safety: Is retry safe, or will it create duplicate payments?

OpenAPI describes the interface. These questions require context that goes beyond the interface.

Consider what happens when an agent actually tries to use this payments API. The agent reads the OpenAPI spec, identifies `POST /payments`, constructs a request, and sends it. So far, so good. But then:

- The response says `"status": "pending"`. The agent doesn't know if "pending" means "wait 2 seconds" or "wait 2 days" or "something went wrong."
- The agent tries to refund a payment. The API returns `400 Bad Request` with `{"error": "invalid_state"}`. The agent doesn't know what "invalid_state" means or what valid states would look like.
- The agent retries `POST /payments` after a timeout. A second payment is created. The agent didn't know the operation wasn't idempotent.

None of these failures are caused by a wrong OpenAPI spec. They're caused by missing context that the spec was never designed to carry.

---

## The Structural Gap

The gap is not about model intelligence. A more capable model still can't answer "Is this operation idempotent?" if the information isn't in the spec. The gap is structural: **API description ≠ agent understanding.**

This is not a call for a new magic file. Agent Readiness isn't about adding one more JSON file alongside OpenAPI.

It's about cumulative layers:

```text
OpenAPI
  + Discovery
  + Authentication
  + Semantics
  + Errors
  + Examples
  + Evidence
```

![Readiness stack: vertical building blocks from OpenAPI (base) to Evidence (top)](https://agentbadge.xyz/images/blog/why-openapi-isnt-enough/3s.webp)

Each layer builds on the previous. Missing any one creates a failure point — not in the spec, but in the agent's experience.

- **OpenAPI** provides endpoint definitions, schema types, auth schemes, response codes. Necessary. But not sufficient.
- **Discovery** makes the API findable by autonomous agents (`llms.txt`, `.well-known`, `ai-sitemap.xml`). Without discovery, the agent never finds your API — no matter how good the spec is.
- **Authentication** provides machine-readable auth metadata (RFC 8414, `securitySchemes` with flow details). Without it, the agent can't obtain credentials autonomously.
- **Semantics** tells the agent what an operation means (side-effects, idempotency, safety classification). Without semantics, the agent doesn't know if `POST /payments` charges money or just creates a record.
- **Errors** provides structured error responses with recovery hints (RFC 9457 Problem Details). Without structured errors, the agent can't recover — it just fails.
- **Examples** gives concrete request/response pairs for every operation. Without examples, the agent guesses at request shapes and gets 400s.
- **Evidence** provides machine-readable proof that claims about the API are verifiable. Without evidence, every claim is just marketing.

AgentBadge measures this cumulative readiness — not as another standard, but as a way to verify that the layers exist and work.

---

## Evidence: Don't Declare, Show

A claim without evidence is a marketing statement. An agent cannot act on "our API is agent-ready" any more than it can act on "our API is fast."

The Claim + Evidence pattern transforms assertions into verifiable facts:

| Claim | Evidence |
|-------|----------|
| "API is discoverable" | `GET /llms.txt` returns 200 with valid content |
| "Auth is machine-readable" | `GET /.well-known/oauth-authorization-server` returns RFC 8414 metadata |
| "Errors follow RFC 9457" | `GET /payments/invalid` returns `application/problem+json` |
| "Refunds are idempotent" | `x-agent-semantics: idempotent: true` in OpenAPI + test endpoint verifies |

![Claim vs Evidence: two-panel comparison showing text claim on left, code evidence on right](https://agentbadge.xyz/images/blog/why-openapi-isnt-enough/4s.webp)

This is the key concept that bridges to the measurement framework. Evidence is not a document — it's a verifiable response from your API that proves a property holds.

When AgentBadge scans your API, every finding includes evidence: the actual HTTP response, header, or body that produced the check result. Not "we think your API supports discovery" — but `GET /llms.txt → 200, content-type: text/plain, 847 bytes, valid format`.

This changes the conversation. Instead of debating whether an API is "agent-ready" in the abstract, you can point to specific, verifiable responses. Instead of a badge that says "ready," you get a report that says "72 checks run, 58 passed, 14 failed — here's the evidence for each."

Evidence also means reproducibility. Another agent, another scanner, another developer can run the same checks and get the same results. The claim isn't "trust us" — it's "verify yourself."

---

## The Measurement Problem

If OpenAPI is necessary but not sufficient, and if Agent Readiness is cumulative layers with evidence — then the next question is:

> **How do we objectively determine what an agent can actually discover, understand, and use?**

That's the measurement problem. And it's what [Article 8 — "Measuring Agent Readiness: A Practical Framework for AI-Ready APIs"](https://agentbadge.xyz/blog/measure-dont-certify) addresses.

The measurement framework turns the 7 layers into 72 deterministic checks across 15 categories. Each check produces evidence. Each evidence item is scored. Each score is verifiable.

![Article 7 to Article 8 bridge: flow from structural gap through measurement problem to framework](https://agentbadge.xyz/images/blog/why-openapi-isnt-enough/5s.webp)

---

## What You Can Do Now

1. **Check your discovery layer** — Does `GET /llms.txt` return 200? Does `/.well-known/openapi` exist?
2. **Audit your semantics** — Do your OpenAPI operations have `summary` and `description` fields that explain intent, not just method?
3. **Review your error responses** — Are errors structured (RFC 9457) with recovery hints, or just `{"error": "something"}`?
4. **Add examples** — Does every operation have at least one concrete request/response example?
5. **Run a scan** — `npx @agentbadge/cli scan https://your-api.com` — 72 checks in seconds, free, no signup.

```bash
npx @agentbadge/cli scan https://api.example.com

# JSON report with evidence
npx @agentbadge/cli scan https://api.example.com --format json > report.json
```

Every finding links to the HTTP response that produced it. Evidence, not assertions.

---

## Related Articles

- [What Is Agent Readiness?](https://agentbadge.xyz/blog/what-is-agent-readiness) — Article 1: the foundational concept
- [Why AI Agents Fail to Use APIs](https://agentbadge.xyz/blog/why-ai-agents-fail-to-use-apis) — Article 5: 7 failure modes
- [What Does an AI Agent Need to Understand an API?](https://agentbadge.xyz/blog/what-ai-agent-needs-to-understand-api) — Article 6: 8 context layers

---

*OpenAPI describes an API. Agent Readiness describes whether an agent can actually use it.*

> Originally published at [AgentBadge](https://agentbadge.xyz/blog/why-openapi-isnt-enough)
