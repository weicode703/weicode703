<p align="center">
  <img src="./assets/profile-hero.svg" alt="WeiLiang — AI Application Developer & Builder" width="100%" />
</p>

<p align="center">
  <a href="./README.md">简体中文</a>
  ·
  <strong>English</strong>
</p>

<h1 align="center">Hi, I'm WeiLiang 👋</h1>

<h3 align="center">I don't build demos that run once. I build AI applications with human checkpoints, rule-based guardrails, and verifiable results.</h3>

<p align="center">
  Agent Orchestration · Multimodal Retrieval · Data Engineering · AI-assisted Development
</p>

<p align="center">
  <a href="https://github.com/weicode703?tab=repositories">Repositories</a>
  ·
  <a href="https://weicode703.github.io/interview-notes/">Knowledge Base</a>
</p>

---

## What I care about: whether AI actually runs in real business

In enterprise scenarios, the bottleneck of AI is never the model — it's **data reliability and the controllability of tool boundaries**. So I design AI applications as real software systems: the model is just the understanding component, while what actually decides delivery is the data pipeline, validation rules, human checkpoints, failure recovery, and post-launch verification.

My projects cover exactly this chain:

```text
house-agent  →  multi-subgraph Agent, human checkpoints, anti-hallucination guardrails
AIWear       →  multimodal retrieval, RAG degradation, streaming engineering
sync engine  →  idempotent enterprise sync with daily full reconciliation (closed-source)
skill        →  packaging repetitive workflows into reusable automation
```

---

## 01 / Zhiju Agent

### [A renting-conversation Agent that asks for confirmation and falls back safely](https://github.com/weicode703/house-agent)

> A spoken request comes in, gets structured into intent and parameters, then routes into four subgraphs — recommendation, reservation, preferences, small talk. Before any action with real consequences, it always stops and waits for human confirmation.

| My role | Positioning | Status | Links |
|---|---|---|---|
| Independently designed & implemented | Multi-subgraph conversational Agent | Live | [Repository](https://github.com/weicode703/house-agent) |

### Key designs in this project

- **Single entry, multiple subgraphs**: the main graph routes intent via structured output into recommendation / reservation / preference / chat subgraphs; long-session state is persisted, cleaned up, and resumable from checkpoints.
- **Human-in-the-loop checkpoint**: before consequential actions like reservations, the subgraph raises an `interrupt`, surfaced to the user through a gateway, and resumes from the breakpoint after confirmation — **technically-possible-full-auto and should-it-be-full-auto are two different questions**.
- **Anti-hallucination double insurance**: after the model extracts 8 structured fields, regex hard validation kicks in (city must end with a district/county suffix, budget must match number + unit + direction word). Any failed check goes back to the user — rather ask twice than write once wrongly.
- **SQL review node**: AI-generated queries pass through an independent `check_query` review node that catches NOT IN with NULL, JOIN type mismatches and similar traps before execution.

**Core Stack**

`Python` `LangGraph` `LangChain` `MySQL` `Redis` `Docker`

---

## 02 / AIWear

### [Image-based outfit search + RAG styling chat](https://github.com/weicode703/AIWear)

> CLIP encodes garments into vectors for retrieval, Qwen-VL understands the image semantics, keyword rules act as fallback — three complementary paths guarantee there is always a result.

| My role | Positioning | Status | Links |
|---|---|---|---|
| Independently developed & deployed | Multimodal retrieval + RAG application | Live | [Repository](https://github.com/weicode703/AIWear) |

### Key designs in this project

- **Vector retrieval**: CLIP-ViT-Base-Patch16 encodes garment images into 512-dim vectors stored in Redis, powering Top-5 similar-item search.
- **Three complementary paths**: Qwen-VL handles image semantics, zero-shot classification and keyword rules act as fallbacks — every path has a backup.
- **Three-level RAG degradation**: on primary model failure it switches to an OpenAI-compatible endpoint, then to local capabilities — **the chat service never returns 502**; answers carry source citations to suppress hallucination.
- **Streaming & auth**: a custom five-event SSE protocol (start/progress/delta/done/error); JWT + Redis blacklist makes logout take effect immediately.
- **Delivery**: Docker Compose orchestrates the app, database, and cache in one command.

**Core Stack**

`Python` `FastAPI` `CLIP` `Qwen-VL` `Redis` `SSE` `Docker Compose`

---

## 03 / Enterprise delivery: data sync engine & AI data foundation

### Moving 98 recurring reporting tasks into the system — not a single change lost

> During my internship in an enterprise digitalization project, I worked with 16 department heads, structured 98 recurring reporting tasks into configuration, and automated the flow — task changes are written into DingTalk within 1 minute.

| Form | My role | Status | Links |
|---|---|---|---|
| Enterprise-grade data pipeline | Development member: data access & sync service | Running in production | Enterprise project · closed-source |

### Key designs in this project

- **Rules separated from execution**: all 98 tasks became structured configuration ("cycle, owner, data source, exception handling"); changes touch configuration only, never code — the speed comes from architecture, not performance.
- **Sync engine**: pure Python standard library (http.server + sqlite3 + hmac), with self-implemented HS256 JWT and webhook HMAC verification; SQLite transactional dedup for idempotency, exponential-backoff retries (60s → 1h, max 8 attempts), dead-letter queue beyond that, and a daily 03:00 full reconciliation as the final safety net.
- **Platform limits are requirements**: DingTalk sync sheets forbid third-party row creation, so I designed a "pre-seeded waiting rows + fill-and-reuse" scheme that runs entirely within the platform's rules — understand the boundary first, then design.
- **AI data foundation**: provides accurate, real-time data for Agent tool calls — the ceiling of an Agent's output is the quality of the data its tools return.

**Core Stack**

`Python stdlib` `SQLite` `Webhook` `HMAC / HS256` `DingTalk Open Platform`

---

## 04 / Tools & knowledge base

| Project | One-liner | Links |
|---|---|---|
| **Interview notes knowledge base** | A 40+ article question bank built along my own tech stack; frontmatter-driven question pool, audit script for format and broken links, CI auto-publish | [Live ↗](https://weicode703.github.io/interview-notes/) |
| **Campus-apply email skill** | Repetitive email application flows packaged as a Skill: job screenshot + mailbox screenshot → AI drafts the body → paste & send | [Repository ↗](https://github.com/weicode703/campus-apply-email-skill) |

---

## The method behind these projects

| Principle | How I engineer |
|---|---|
| **Model understands, rules guard** | Spoken language goes to the model; every AI step is followed by a deterministic validation layer. |
| **Rather ask twice than write once wrongly** | When AI output is uncertain, bounce it back for confirmation instead of guessing. |
| **AI does the work, verification protects quality** | Provide context, split tasks, constrain output, machine-check results, feed errors back — all five, no shortcuts. |
| **Recovery before root cause** | Stop the bleeding first, investigate after, then land the conclusion in monitoring and config. |
| **Deployment is not the finish line** | Config exists ≠ checks pass ≠ browser works ≠ real users actually use it — it counts only when the flow truly runs. |

## Tech landscape

| Layer | Technologies | What I build |
|---|---|---|
| **Agent orchestration** | LangGraph, LangChain, Structured Output, Interrupt | Intent routing, multi-subgraphs, human checkpoints, resume from breakpoints |
| **Retrieval & multimodal** | CLIP, Qwen-VL, Redis Vector, RAG | Image search, three-level degradation, source citations |
| **Backend** | Python, FastAPI, JWT, SSE | APIs, auth, streaming output, error protocols |
| **Data** | MySQL, SQLite, Redis, full reconciliation | Idempotent sync, backoff retries, dead letters, reconciliation |
| **Delivery** | Docker Compose, Nginx, Linux, GitHub Actions | Orchestration, deployment, CI builds |
| **AI tooling** | Claude Code, Codex, Skill | Context engineering, task splitting, machine acceptance |

## Currently working on

- Refining human-checkpoint and anti-hallucination patterns for Agents
- Going deeper into RAG reliability and evaluation methods
- Turning more repetitive flows into Skills and automation scripts
- Validating the "data reliability first" approach in real delivery

---

<h3 align="center">Make AI actually run in business.</h3>

<p align="center">
  Human checkpoints · Rule-based guardrails · Verifiable results
</p>

<p align="center">
  <a href="https://github.com/weicode703?tab=repositories">Explore my repositories</a>
  ·
  <a href="https://weicode703.github.io/interview-notes/">Read my notes</a>
</p>
