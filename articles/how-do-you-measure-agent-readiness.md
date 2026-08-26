---
title: "How Do You Measure Agent Readiness? — エージェントの準備度をどう測定するか"
emoji: 📏
type: tech
topics: ai, api, agent, testing, measurement
canonical_url: https://agentbadge.xyz/blog/how-do-you-measure-agent-readiness
cover_image: https://agentbadge.xyz/images/blog/how-do-you-measure-agent-readiness/1s.webp
published: false
---

<!-- ZENN PUBLISHING NOTES
- Canonical URL: https://agentbadge.xyz/blog/how-do-you-measure-agent-readiness
- Topics: ai, api, agent, testing, measurement (max 5)
- Body: English (unchanged from Hashnode version)
- Local intro: Japanese, 2-3 sentences before the English body
- Zenn: GitHub sync or web editor. No API yet.
- Image: hosted at agentbadge.xyz — Zenn supports external image URLs
- Publish via: WORKING-SMI/Zenn/Scripts/publish.sh how-do-you-measure-agent-readiness "Article 8: How Do You Measure Agent Readiness?"
-->

![cover](https://agentbadge.xyz/images/blog/how-do-you-measure-agent-readiness/1s.webp)

# How Do You Measure Agent Readiness?

## エージェントの準備度をどう測定するか

> Agent Readinessが本物の概念なら、それは測定可能であるべきです。そして、その測定は再現可能でなければなりません。この記事では、LLMの意見ではなく、決定論的チェックと証拠に基づく測定フレームワークを紹介します。URL、ルールセット、タイムスタンプが同じであれば、結果も常に同じになる。それが「意見」ではなく「測定」である条件です。日本の開発者にとって、APIの「AI対応」を主観的ラベルではなく客観的指標で評価するアプローチを考察します。

---

> If Agent Readiness is real, it should be measurable. And the measurement should be reproducible.

You've read about [what Agent Readiness is](https://agentbadge.xyz/blog/what-is-agent-readiness). You've seen [why AI agents fail to use APIs](https://agentbadge.xyz/blog/why-ai-agents-fail-to-use-apis) and [what an agent needs to understand](https://agentbadge.xyz/blog/what-ai-agent-needs-to-understand-api). You know [why OpenAPI alone isn't enough](https://agentbadge.xyz/blog/why-openapi-isnt-enough).

Now the question shifts from "what" to "how":

> **How do you objectively determine whether an API is ready for AI agents?**

This article introduces a measurement framework for Agent Readiness — one built on deterministic checks, evidence, and reproducibility. Not opinions. Not LLM scores. Measurable properties that any scanner can verify.

![Hero — Subjective labels on the left ("AI-friendly", "Agent-ready", "Optimized for AI") with question marks, deterministic formula on the right (same URL + same ruleset + same time = same result) with a green checkmark](https://agentbadge.xyz/images/blog/how-do-you-measure-agent-readiness/1s.webp)

---

## The Measurement Problem

Labels like "AI-friendly API", "Agent-ready", and "Optimized for AI" are everywhere. They sound useful. They aren't.

Two auditors can look at the same API and disagree on whether it's "agent-friendly." An LLM can score the same API differently on different runs. A marketing page can claim "AI-optimized" without any way to verify what that means.

The problem isn't that these labels are wrong. The problem is that they're **not reproducible**. If two people can look at the same API and reach different conclusions, the measurement isn't real — it's an opinion.

If Agent Readiness is a real property of an API, it should be measurable. And the measurement should satisfy a simple requirement:

```text
same URL + same ruleset + same point in time = same result
```

This is the reproducibility requirement. It's what separates measurement from opinion.

---

## What Should We Measure?

Agent Readiness isn't a single number. It's a set of properties across four categories:

- **Discovery** — Can an agent find the API?
- **Documentation** — Can an agent understand the API?
- **Authentication** — Can an agent authenticate autonomously?
- **Machine Readability** — Can an agent interact machine-to-machine?

But these aren't just checkboxes. Each category contains specific, testable assertions — properties that can be verified with HTTP requests:

```text
Discovery
  ✓ OpenAPI is discoverable
  ✓ llms.txt exists
  ✓ Documented API entry point exists

Authentication
  ✓ Authentication mechanism is declared
  ✓ Required credentials are documented
  ✓ Protected endpoint behavior is understandable
```

The question isn't "does the API have OpenAPI?" The question is "can we verify that OpenAPI is discoverable?" — and that's a testable property.

![Deterministic pipeline: URL → Scanner → Evidence → Rules → Score, with AI copilot as optional dashed step at the end](https://agentbadge.xyz/images/blog/how-do-you-measure-agent-readiness/2s.webp)

---

## Deterministic Before Intelligent

This is the central principle of the measurement framework.

First:

```text
HTTP response → Rule → Evidence → Result
```

Then, AI can help interpret complex cases. But the AI is a copilot, not the primary engine.

The wrong approach:

```text
URL → LLM → "Looks agent-ready: 76/100"
```

The right approach:

```text
URL → Deterministic scanner → Evidence → Rules → Score → AI copilot (optional)
```

This is what distinguishes AgentBadge from an AI auditor. Deterministic checks are reproducible — same input, same output, every time. LLM assessments are not. An LLM might score the same API as 76 today and 82 tomorrow. A deterministic scanner will give you the same result as long as the API hasn't changed.

This doesn't mean AI is useless. AI is excellent at interpreting ambiguous evidence, suggesting fixes, and explaining results. But the measurement itself — the check, the evidence, the score — should be deterministic.

---

## Evidence, Not Opinions

Every assertion in the measurement framework comes with evidence. Not "we think this is true" — but the actual HTTP response that proves it.

Here's what an evidence card looks like:

```text
OPENAPI_DISCOVERABLE
Status: VERIFIED

Evidence:
  GET /openapi.json
  HTTP 200
  Content-Type: application/json
  Valid OpenAPI document
```

![Evidence card: OPENAPI_DISCOVERABLE with Status: VERIFIED in green, evidence block showing GET /openapi.json, HTTP 200, Content-Type: application/json, Valid OpenAPI document](https://agentbadge.xyz/images/blog/how-do-you-measure-agent-readiness/3s.webp)

This is the key difference between measuring and certifying. A certification says "this API is agent-ready." An evidence card says "here is the HTTP response that proves OpenAPI is discoverable."

> **Don't tell developers what to believe. Show them what we measured.**

When every assertion includes evidence, the conversation changes. Instead of debating whether an API is "ready," you can point to specific findings: 72 checks run, 58 passed, 14 failed — here's the evidence for each.

---

## Assertions

A scan result is not a magic score. It's a set of assertions — each one testable, each one with a status and evidence:

| Assertion | Status | Evidence |
|-----------|--------|----------|
| OpenAPI discoverable | VERIFIED | `/openapi.json → 200` |
| Authentication documented | VERIFIED | `securitySchemes` present in spec |
| Machine-readable errors | MISSING | HTML error response, not structured |
| Agent guide | MISSING | `404 /agent-guide.json` |

![Assertions table: four rows showing Assertion, Status, and Evidence columns — two VERIFIED in green, two MISSING in red](https://agentbadge.xyz/images/blog/how-do-you-measure-agent-readiness/4s.webp)

This table is the heart of the measurement. Before you look at the score, you look at the assertions. Each assertion tells you something specific about the API — and each one is independently verifiable.

---

## VERIFIED / INFERRED / CONFLICT / MISSING

Every assertion has one of four statuses:

- **VERIFIED** — Direct proof exists. The scanner found the evidence.
- **MISSING** — Not found. The scanner looked and didn't find it.
- **INFERRED** — There are reasonable grounds to believe this is true, but the evidence is insufficient for verification.
- **CONFLICT** — Two sources contradict each other.

Here's a real example of CONFLICT:

```text
OpenAPI spec says:    POST /refund
Agent Guide says:     POST /refund-request
```

Two sources, same API, different paths. The assertion status is CONFLICT — not VERIFIED, not MISSING. The scanner can't verify which is correct without making a live request, so it flags the contradiction.

![Status model: four cards in a 2x2 grid — VERIFIED (green checkmark), MISSING (red x), INFERRED (yellow question mark), CONFLICT (orange warning) with one-line definitions](https://agentbadge.xyz/images/blog/how-do-you-measure-agent-readiness/5s.webp)

The distinction between INFERRED and VERIFIED matters. INFERRED means "this looks right, but we can't prove it." VERIFIED means "here's the proof."

> **Confidence is not the same thing as verification.**

---

## Scoring

Only after assertions are established do we compute a score. The score is derived from the assertions — not the other way around.

```text
Discovery           18/20
Documentation       19/25
Authentication      17/20
Machine Readability 15/20
Verification        10/15
─────────────────────────
Total               79/100
```

![Scoring breakdown: five category bars in cyan with scores, total 79/100 in green, and a category floor example showing Discovery = 0 blocking a 91/100 total](https://agentbadge.xyz/images/blog/how-do-you-measure-agent-readiness/6s.webp)

There's a critical rule in the scoring model: **category floor**. A high total score should not hide a critical zero in a fundamental category.

If Discovery = 0, the API is effectively invisible to agents. No amount of excellent documentation or perfect authentication can compensate for the fact that agents can't find the API. A score of 91/100 with Discovery = 0 is misleading — it suggests the API is nearly ready when it's actually missing the most fundamental layer.

The category floor prevents this. If any critical category is zero, the total score is capped. A high score should reflect actual readiness, not average out a fatal gap.

> **A high score should not hide a critical zero.**

---

## Score ≠ Certification

AgentBadge doesn't say "this API is safe" or "this API is approved for agents."

It says: **"Here is what we measured, under this ruleset, at this point in time."**

This distinction matters for three reasons:

1. **Trust** — Developers can verify the evidence themselves. They don't need to trust a badge; they can check the proof.
2. **Legal risk** — Certification implies endorsement. Measurement implies observation. AgentBadge observes and reports; it doesn't endorse.
3. **Reproducibility** — Anyone can run the same checks and get the same results. The measurement is transparent, not opaque.

> **Don't certify. Measure.**

---

## Reproducibility

A measurement is only useful if it can be independently verified. The reproducibility formula is:

```text
URL + timestamp + ruleset version + scan artifact + report hash
```

Example:

```text
Agent Readiness v1.0
Scan: 2026-08-26T14:03:22Z
Ruleset: agentbadge-ruleset@1.0.0
Report hash: a3f7b2c1...
Score: 79/100
```

Every scan records the URL, the timestamp, the ruleset version, and produces a report hash. The scan artifact is preserved. Another scanner — or another developer — can run the same checks against the same URL with the same ruleset and verify the results.

This is what makes the measurement real. It's not a subjective assessment that changes with the auditor. It's a deterministic process that produces the same output for the same input.

---

## Static Measurement vs Real Agent Behavior

An honest caveat: **static readiness does not prove that every AI agent will successfully use an API.**

AgentBadge measures whether an API *can be* discovered, understood, and potentially used by an agent — based on observable evidence. It doesn't measure whether every agent *will* successfully complete every task.

These are different questions:

- **Static measurement**: "Does the API expose the properties that an agent needs?" (Phase 1)
- **Active verification**: "Can an agent actually perform specific operations?" (Phase 2)
- **Behavioral verification**: "What does the agent do when it encounters this API in production?" (Future)

The measurement framework starts with Phase 1 — static measurement. It's the foundation. But it's not the end of the road.

```text
Phase 1: Static measurement (current)
    ↓
Phase 2: Active verification (next)
    ↓
Future: Behavioral / runtime verification
```

Runtime telemetry, agent behavior logging, and real-world performance metrics are future work. They belong in later articles, not this one. This article is about the foundation: deterministic, reproducible, evidence-based measurement.

---

## What to Do with Results

Measurement without action is just data. The cycle is:

```text
Measure → Evidence → Fix
```

Here's what a typical result looks like:

```text
Score: 76/100

Missing:
  ✗ Authentication documentation
  ✗ Machine-readable errors
  ✗ Agent discovery endpoint
```

The question every API team asks: **"What can I do about it?"**

The answer is in the evidence. Each MISSING assertion points to a specific, fixable gap:

- **Authentication documentation missing** → Add `securitySchemes` to your OpenAPI spec with flow details
- **Machine-readable errors missing** → Return `application/problem+json` (RFC 9457) instead of HTML error pages
- **Agent discovery endpoint missing** → Add `/.well-known/openapi` or `/agent-guide.json`

Fix the gaps. Re-scan. Verify the improvements with measurable evidence.

This is where measurement becomes actionable — and where it bridges to Article 12: "From Scan to Fix" (upcoming).

---

## For AI Agents

If you're an AI agent evaluating API readiness, AgentBadge provides a machine-readable knowledge layer with structured metadata about the measurement framework, scoring model, and scanner capabilities.

```text
/agent-guide/
/agent-guide/articles/how-do-you-measure-agent-readiness
/agent-guide/concepts/agent-readiness
/agent-guide/concepts/scoring
/agent-guide/capabilities/scanner
/agent-guide/knowledge-map.json
```

---

## Related Articles

- [What Is Agent Readiness?](https://agentbadge.xyz/blog/what-is-agent-readiness) — Article 1: the foundational concept
- [Why AI Agents Fail to Use APIs](https://agentbadge.xyz/blog/why-ai-agents-fail-to-use-apis) — Article 5: 7 failure modes that measurement addresses
- [What Does an AI Agent Need to Understand an API?](https://agentbadge.xyz/blog/what-ai-agent-needs-to-understand-api) — Article 6: 8 context layers that measurement checks
- [Why Your OpenAPI Spec Isn't Enough for AI Agents](https://agentbadge.xyz/blog/why-openapi-isnt-enough) — Article 7: the structural gap that measurement fills
- *Inside an Agent Readiness Scanner* — Article 9 (upcoming): the engineering architecture behind the measurement engine

---

*Don't certify. Measure.*

> Originally published at [AgentBadge](https://agentbadge.xyz/blog/how-do-you-measure-agent-readiness).
