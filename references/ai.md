# AI Systems Engineering Curriculum

Building production systems around large language models. Prerequisite: software-engineering.md L2+ (this is systems engineering first, ML second). The unifying mental model, stated once and returned to relentlessly: **the model is a fallible, non-deterministic, expensive subsystem** — the engineering discipline lives in the scaffolding around it (retrieval, tools, memory, evals, guardrails, serving), not in the model itself. Engineers who treat the model as magic build demos; engineers who treat it as an unreliable dependency with a cost meter build products.

This is the fastest-moving track — teach principles and failure modes that outlive specific models/frameworks, and always verify current model/API specifics against the claude-api skill or live docs rather than from memory (models and prices change monthly).

## Table of contents

1. [LLM fundamentals](#1-llm-fundamentals)
2. [Prompt engineering](#2-prompt-engineering)
3. [Structured output and function/tool calling](#3-structured-output-and-functiontool-calling)
4. [Embeddings and vector search](#4-embeddings-and-vector-search)
5. [RAG](#5-rag)
6. [Agents](#6-agents)
7. [Memory systems](#7-memory-systems)
8. [MCP and the tool ecosystem](#8-mcp-and-the-tool-ecosystem)
9. [Evaluation](#9-evaluation) — the discipline that separates production from demo
10. [AI safety, security, and guardrails](#10-ai-safety-security-and-guardrails)
11. [Serving, cost, and latency engineering](#11-serving-cost-and-latency-engineering)
12. [AI product and UX](#12-ai-product-and-ux)
13. [Fine-tuning and when not to](#13-fine-tuning-and-when-not-to)

---

## 1. LLM fundamentals

Working-knowledge depth — enough to reason about behavior and failure, not to train one:

- The transformer's load-bearing ideas: tokens (subword units — the reason for weird counting, typo sensitivity, and non-English cost multipliers), embeddings (tokens as vectors in meaning-space), attention (content-based routing of information — why context matters and why it's quadratic in length), autoregressive generation (one token at a time, each conditioned on all prior — the reason latency scales with output length and why streaming exists).
- The context window as the model's *entire* working memory: no persistence between calls (every stateless call re-sends everything — the architectural fact that generates memory systems §7 and the cost model §11); positional effects (lost-in-the-middle — information at the edges of long context gets more attention than the middle, a real retrieval-design constraint §5).
- Sampling and why output varies: temperature/top-p as the randomness knobs; hallucination reframed correctly — it's sampling from a distribution that has no "I don't know" mass by default, a *structural* property to engineer around (grounding, citations, abstention prompting, verification), not a bug awaiting a patch.
- The capability/limitation map that drives design: strong at language transformation, pattern completion, and synthesis; weak at exact arithmetic, up-to-date facts, and reliable multi-step precision — which is *why* tools (§3), retrieval (§5), and eval (§9) exist. Reasoning models (extended inference-time computation) shift but don't erase this map.
- **Exercise themes**: predict how a described task will fail and name the scaffolding that fixes it; estimate token count and cost for a described prompt/response before running it, then verify.

## 2. Prompt engineering

- The prompt is a program in a probabilistic language — teach it with software rigor, not incantation folklore:
- Structure: system vs user roles (durable instructions vs per-request input), few-shot examples as the highest-leverage technique (demonstration beats description; the examples *are* the spec), clear task decomposition, explicit output contracts.
- Reasoning techniques and their honest scope: chain-of-thought (think-before-answer for multi-step tasks — and when reasoning models make explicit CoT redundant), decomposition (break complex tasks into checkable steps), self-consistency and verification passes (at cost/latency multiples — price them).
- The security lens from day one: **prompt injection is the SQL injection of LLMs** — untrusted data sharing a channel with trusted instructions (§10 owns the defenses; introduce the parallel here because it reframes every "just tell the model to..." instinct as the doomed input-sanitization approach it is).
- Engineering discipline over prompt-tweaking: version prompts (they're code — diff, review, roll back), test them against an eval set (§9 — "it worked when I tried it" is not a methodology), keep them in the codebase not in someone's notebook, measure changes rather than vibe them.
- **Exercise themes**: turn a vague instruction into a contract-shaped prompt with few-shot examples and measure the improvement on a small eval set; break a provided prompt with an injection and then attempt to defend it (and discover why prompt-level defense alone fails — motivating §10).

## 3. Structured output and function/tool calling

- Structured output done right: schema-enforced generation (tool/JSON-schema constraints) beats "please respond in JSON" — the difference is a validity *guarantee* vs a hope; always validate at the boundary anyway (parse-don't-validate, software-engineering §10 / frontend §3 — the model's output is untrusted input to your system).
- Tool calling's core truth: **the model emits *intent*; your code executes.** The model never has capabilities — only requests them. This single framing drives the whole security posture: validate arguments before execution, apply least privilege to the tool surface (§10), and never wire a model output straight into a dangerous sink.
- Tool schema design *is* prompt engineering: names and descriptions are read by the model as instructions — a well-named, well-described, narrowly-scoped tool is invoked correctly; a vague one is misused. Return errors as informative feedback the model can recover from (the tool-use loop is a conversation).
- Reliability: tools fail and models retry — idempotency and the resilient-client bundle (backend.md §8) apply to model-invoked tools too; guardrails on the *number* of tool calls (runaway loops are a cost and safety event).
- **Exercise themes**: design the tool schema for a described capability and critique three bad versions of it; add validation + least-privilege + error-feedback to a naive tool implementation.

## 4. Embeddings and vector search

- Embeddings as coordinates in meaning-space: semantic similarity as geometric proximity (cosine similarity), why this enables search-by-meaning rather than search-by-keyword; embedding model choice and the dimension/quality/cost tradeoff.
- **Chunking as the most consequential and least glamorous decision** in the whole retrieval stack: chunk too big and you dilute relevance and waste context; too small and you sever meaning; strategies (fixed, semantic, structural/by-document-structure, overlapping) chosen by document type — and the honest lesson that chunking quality caps RAG quality more than any model choice.
- Vector storage: ANN indexes (HNSW at concept level — the speed/recall tradeoff of approximate search), metadata filtering (the underused half — most "vector search" is really filtered vector search: tenant, date, permissions), and the pragmatic default — **pgvector until you measure a reason to leave** (a dedicated vector DB is often premature; your data already lives in Postgres).
- **Exercise themes**: chunk three different document types (code, prose, tables) and defend each strategy; design the metadata filter schema for a permissioned multi-tenant knowledge base.

## 5. RAG

- Why RAG over fine-tuning for knowledge: freshness (update the index, not the weights), provenance (cite sources — the abstention/grounding answer to hallucination §1), access control (retrieve only what this user may see — a security boundary, not a nicety), and cost. Fine-tuning teaches *behavior/format*; RAG supplies *knowledge* — a distinction people conflate constantly (§13).
- The pipeline and where each stage fails: ingest/chunk (§4) → embed → retrieve → (rerank) → assemble context → generate → (cite/verify). Failure taxonomy to teach explicitly: retrieval miss (the answer wasn't fetched — no prompt can fix missing context), lost-in-the-middle (fetched but ignored — §1), context stuffing (too much retrieved, signal buried), and generation-retrieval mismatch (good context, model ignores it).
- Retrieval quality is the game: hybrid search (dense embeddings + sparse/BM25 keyword — each catches what the other misses), reranking (a second-stage model over the top-K — often the highest-ROI single addition), query transformation (rewriting/expanding the user's query before retrieval). **Evaluate retrieval separately from generation** (recall@k / precision on labeled queries — §9) because a RAG system has two failure surfaces and debugging the wrong one wastes weeks.
- Advanced patterns as pointers: agentic/iterative retrieval (the model decides what to fetch next — §6), graph RAG, multi-hop; introduce only after the basic pipeline and its eval are solid.
- **Exercise themes**: build a minimal RAG over a small corpus with a *retrieval* eval (hand-label 20 queries, measure recall@k), then improve retrieval and re-measure; diagnose a described RAG failure to the correct stage.

## 6. Agents

- The loop: perceive → decide → act → observe → repeat until goal-or-limit. An agent is a control loop with an LLM as the policy — most agent engineering is control-loop engineering (state, termination, error handling), not prompting.
- **The reliability math that kills naive agents**: per-step reliability compounds — 0.95^20 ≈ 0.36. This one calculation dictates the whole discipline: minimize steps, make each step verifiable, add checkpoints, prefer constrained workflows over open-ended autonomy where the task allows. "Give the agent tools and let it figure it out" is a demo strategy; production agents are mostly rails.
- Design axes: planning vs reactive (plan-then-execute enables review and recovery; react is simpler and cheaper for short tasks), workflow vs autonomous (the honest spectrum — a fixed pipeline with LLM steps is more reliable than a free agent and is *usually the right answer*; reach for autonomy only when the task space genuinely can't be pre-structured), single vs multi-agent (multi-agent multiplies coordination cost and failure surface — justify it, don't default to it).
- The non-negotiables: human-in-the-loop gates for irreversible/high-stakes actions (spend, delete, send — the same "confirm destructive actions" instinct as everywhere), sandboxing (agent-executed code/tools run confined — security §10), budgets (step count, token, cost, wall-clock — an unbounded agent is a runaway bill and a safety event), and observability (trace every step — debugging an agent without a trace of its decisions is hopeless; devops §7).
- **Exercise themes**: take a described "autonomous agent" idea and redesign it as a constrained workflow, justifying each place autonomy survives; add budgets, HITL gates, and step-tracing to a naive agent loop; compute the expected success rate of a described N-step agent and propose how to raise it.

## 7. Memory systems

- The problem restated from §1: context is finite and stateless per call; "memory" is the engineering of what to carry forward and how. Distinguish context-window management (this conversation) from durable memory (across sessions).
- Techniques by need: conversation summarization (rolling/hierarchical — compress old turns, keep recent verbatim; the summary-drift failure mode), fact/entity extraction into a store (structured memory), retrieval-based memory (RAG-over-history — §5 applied to past interactions), and working/scratchpad memory within a task.
- The underrated lesson: **forgetting is a feature.** Stale or wrong memory poisons behavior worse than no memory (the agent that "remembers" a fact the user corrected); memory needs invalidation, recency weighting, and confidence — the same derived-data discipline as caching (architecture §8) and the same freshness-contract thinking.
- **Exercise themes**: design the memory architecture for a described assistant (what's summarized, what's extracted to structure, what's retrieved, what expires); find the memory-poisoning failure in a described system.

## 8. MCP and the tool ecosystem

- Model Context Protocol as the standardization of model↔tool/resource/prompt connections: servers expose capabilities (tools, resources, prompts), clients (Claude Code, IDEs, apps) consume them — the "USB-C for AI tools" framing; why a standard beats N bespoke integrations (the M×N → M+N collapse).
- The build decision: MCP server vs plain API wrapper — build an MCP server when the capability should be reusable across many AI clients and benefits from the protocol's discovery/typing; a plain function/tool is fine for one app. Server-design judgment reuses API-design skill (backend.md §2) plus the tool-schema-as-prompt lesson (§3).
- Security posture for MCP (cross-link security §10): tools are capability grants to a probabilistic actor operating on possibly-injected input — least privilege, argument validation, human gates for dangerous tools, and treating tool *outputs* as untrusted (a compromised/poisoned tool result is injection §10).
- **Exercise themes**: design a small MCP server for a described capability (tools, resources, error contracts) and threat-model its abuse; decide MCP-server-vs-tool for three described integrations.

## 9. Evaluation

The discipline that separates production AI from demos — and the one beginners skip. Teach it as non-optional and teach it *early*, not as a final chapter.

- **Eval-driven development**: write the eval before (or with) the feature — the eval is the executable spec for a system whose behavior you can't read from the code (the parallel to TDD, software-engineering §3, is exact and worth drawing). "It looked good on five examples" is the signature of every AI project that later failed silently in production.
- The eval ladder by cost and fidelity: programmatic/assertion checks (exact match, schema validity, contains-citation, latency, cost — cheap, run on every change), LLM-as-judge (a model grades outputs against a rubric — scalable subjective grading, but the judge must itself be calibrated against human labels or you're measuring one model's bias; position bias, verbosity bias, and self-preference are real and must be controlled), human review (the ground truth, expensive — reserve for calibration and high-stakes).
- Building the harness: a reference dataset (curated inputs + expected properties — grown from real failures, not imagined ones), regression tracking across every prompt/model/parameter change (the model or prompt you "improved" often regresses something else — without the harness you ship it blind), and treating eval scores like a test suite in CI.
- Failure-mode-first design: enumerate how *this specific* feature fails (RAG → retrieval miss; agent → wrong tool; extraction → hallucinated field) and build a targeted eval per mode — generic "quality" scores hide the failures that matter. Watch for metric gaming (a prompt that games the LLM-judge without helping users — Goodhart again).
- Production evals: online monitoring (sampled outputs graded continuously — behavior drifts as inputs and models change), user-signal capture (thumbs, corrections, abandonment as implicit labels feeding the dataset), A/B testing model/prompt changes on real traffic (career.md experimentation applies).
- **Exercise themes**: write the eval set *first* for a described feature, then implement to pass it; calibrate an LLM-judge against 20 human labels and measure its agreement; design online eval for a shipped feature (what's sampled, graded how, alerting on what drift).

## 10. AI safety, security, and guardrails

- **Prompt injection as the defining security problem** (payoff of the §2 parallel): the model can't reliably distinguish trusted instructions from untrusted data in its context — so the SQLi-style "sanitize the input" instinct fundamentally *cannot* fully work. The real defenses are architectural: privilege separation (the model operates with least authority — §3/§6), treating all retrieved/tool-returned content as untrusted data not instructions, human confirmation for consequential actions, and dual-LLM / quarantine patterns for untrusted content. Direct vs indirect injection (the latter — poisoned documents/web pages/tool results — is the scary one for RAG and agents).
- The broader guardrail stack: input filtering (limited value alone — see above), output filtering (PII, safety, format validation before anything acts on it), the classic sandwich of pre/post checks, and the honest framing that guardrails are defense-in-depth, not a solved boundary.
- Data governance: PII handling in prompts and logs (models log too — §14 scrubbing applies), data residency and what leaves your boundary to a model provider, training-data-use terms, tenant isolation in retrieval (a RAG permission bug is a data breach — §5).
- Abuse and cost as safety: rate/cost limits as safety controls (an unbounded agent or an injection-driven loop is a financial DoS), jailbreak resilience as ongoing defense-in-depth (not a one-time fix), content moderation for user-facing generation.
- Responsible-AI awareness: bias and fairness in outputs, hallucination harm in high-stakes domains (medical/legal/financial — the abstention and human-review requirements get teeth here), transparency (telling users they're talking to AI), and the eval connection (safety properties are eval dimensions §9, not vibes).
- **Exercise themes**: threat-model a described RAG-agent for injection (direct and indirect) and design the architectural defenses; find the data-governance leak in a described logging/prompt setup; design cost/rate guardrails for an agent with tool access.

## 11. Serving, cost, and latency engineering

The production-operations chapter most curricula omit entirely — where AI systems actually meet the bill and the SLO:

- The cost model made explicit: priced per input + output token, output usually costlier, context re-sent every stateless call (so long histories and fat RAG contexts compound cost linearly — §1/§7); estimate before shipping and measure after (verify current pricing via the claude-api skill — never quote from memory).
- The cost/latency levers, ranked: prompt caching (huge win for repeated context — system prompts, RAG chunks, few-shot blocks; understand what's cacheable and the TTL), model right-sizing (route easy requests to small/cheap models, escalate to large only when needed — the single biggest lever most teams ignore; a cascade/router pattern), output-length control (you pay per output token and wait per output token — cap and structure it), context trimming (retrieve/carry less — §5/§7), batching where latency allows, and semantic caching (cache by meaning for repeated-ish queries, with the staleness-contract discipline of architecture §8).
- Latency engineering: streaming as the perceived-performance default (time-to-first-token is the UX metric, not total time — design.md perceived-performance and frontend §6 apply directly), the fan-out/tail-latency problem when a request hits multiple model calls (architecture §11 tail amplification — one slow call stalls the response), speculative and parallel patterns.
- Reliability of a flaky dependency: provider rate limits and outages (the resilient-client bundle — backend.md §8 — applied to model APIs: timeout, retry-with-backoff on 429/5xx, fallback model, circuit breaker), multi-provider abstraction as availability insurance (with the same portability-cost skepticism as multi-cloud, devops §6), graceful degradation (a cached/simpler/non-AI answer beats an error — a product decision, §12).
- Observability for AI systems (devops §7 specialized): trace token counts, cost, latency, and cache-hit-rate per request; log prompts/outputs (scrubbed) for debugging and eval-dataset growth; alert on cost and quality drift, not just errors.
- **Exercise themes**: take a described expensive AI feature and cut cost 60% without quality loss (routing, caching, context trimming, output limits) and defend each; design the resilience layer for a model-API dependency; instrument an AI request end-to-end for cost/latency/quality.

## 12. AI product and UX

- Designing for a probabilistic, sometimes-wrong system (design.md thinking meets AI's nature): set expectations honestly (the interface should signal uncertainty, not project false confidence), make verification cheap (citations, "show sources," editable outputs — the human stays in the loop by design), and design the *error* experience as first-class (regeneration, feedback, correction — because wrong outputs are certain, and peak-end rule means recovery quality dominates perception, design.md §2).
- The interaction patterns and their tradeoffs: streaming vs waiting, chat vs structured/inline AI (chat is often the *lazy* default — inline/embedded AI frequently serves the user better; challenge the reflex), suggestions-with-accept over autonomous action for reversible-but-consequential edits, confidence display without false precision.
- Trust and adoption: over-promising as the fastest way to lose a user (the demo that oversells sets the disappointment); progressive trust (let the AI earn autonomy as the user gains confidence); feedback loops that both improve UX and feed evals (§9) and memory (§7).
- The human-AI collaboration frame: augmentation vs automation as a *product* decision with ethical weight (whose job, whose accountability, what's the human's meaningful role); appropriate reliance (designing against both over-trust — automation complacency — and under-trust — ignoring a correct system).
- **Exercise themes**: redesign a described chat feature as inline/embedded AI and argue the tradeoff; design the uncertainty-and-correction UX for a described high-stakes AI feature.

## 13. Fine-tuning and when not to

- The decision framework taught as *default-to-not*: the ladder is prompt engineering → few-shot → RAG → fine-tuning, ascending in cost/effort/lock-in — exhaust the cheaper rungs first, because most "we need to fine-tune" instincts are really prompt or retrieval problems.
- What fine-tuning actually changes: behavior, format, tone, and consistency on a task shape — **not knowledge** (fine-tuning is a poor and stale way to inject facts; RAG owns knowledge — §5; conflating these is the field's most common expensive mistake). Legitimate uses: strict output format/style at scale, a narrow high-volume task where a small fine-tuned model beats a large prompted one on cost/latency, domain tone.
- The awareness map (don't teach as a how-to unless the user needs it): full fine-tune vs parameter-efficient (LoRA-class), the data problem (fine-tuning is a *data* project — quality and quantity of examples dominate; garbage in, confidently-wrong out), and the maintenance tax (a fine-tuned model is frozen against base-model improvements — you now own a model that ages).
- **Exercise themes**: given four described "we should fine-tune" requests, route each to the right rung of the ladder and justify; design the dataset and eval for a legitimate fine-tuning case.

---

## Review checklist (AI feature readiness)

1. Eval: an eval set exists and predates/accompanies the feature; failure-mode-specific dimensions; regression-tracked in CI; retrieval evaluated separately if RAG.
2. Grounding: hallucination controls appropriate to stakes (citations, abstention, human review); no unsourced high-stakes claims.
3. Security: injection threat-modeled (direct + indirect); tools least-privileged and argument-validated; retrieved/tool content treated as untrusted; human gates on consequential actions.
4. Data governance: PII in prompts/logs handled and scrubbed; tenant isolation in retrieval; what-leaves-the-boundary understood and permitted.
5. Cost: per-request cost estimated and measured; caching and model-routing considered; output length bounded; cost drift alerting.
6. Latency: streaming/TTFT for user-facing; tail-latency of multi-call paths considered; budgets set.
7. Reliability: model-API timeout/retry/fallback/breaker; graceful degradation designed.
8. Agent-specific: step/cost/time budgets; step tracing; termination conditions; sandboxed tools.
9. UX: uncertainty surfaced honestly; correction/regeneration path; not over-promising; feedback captured to eval/memory.
10. Observability: tokens/cost/latency/cache-hit/quality traced per request; outputs sampled for online eval.

## Case study themes (L3+)

The evolution from prompt-hacking to eval-driven development (why serious teams built eval harnesses); indirect prompt injection demonstrations against real assistants (the class of bug the industry is still absorbing); RAG-vs-fine-tuning postmortems (teams that fine-tuned for knowledge and regretted it); agent reliability walls in production (why "autonomous agent" demos didn't ship and what constrained versions did); the cost-blowup story (an unbounded/uncached feature that 10x'd the bill); prompt-caching and model-routing as the cost turnarounds that made features viable. For each: forces → decision → outcome → what would you have done. Verify all model/API specifics against the claude-api skill or current docs — this field's facts expire fast.
