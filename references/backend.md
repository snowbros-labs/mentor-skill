# Backend & Data Engineering Curriculum

The server side as its own discipline: APIs, data modeling, databases, async work, and the reliability habits that keep backends honest. Prerequisite: software-engineering.md L2 topics. Deepens into architecture.md for system-scale concerns. The unifying mental model: **a backend is a state machine guarding invariants across concurrent, unreliable clients** — every topic here is about protecting correctness while staying fast and available.

## Table of contents

1. [HTTP and the web's contract](#1-http-and-the-webs-contract)
2. [API design](#2-api-design)
3. [Data modeling](#3-data-modeling)
4. [Relational databases at depth](#4-relational-databases-at-depth)
5. [Beyond relational](#5-beyond-relational)
6. [Async work and background jobs](#6-async-work-and-background-jobs)
7. [State, sessions, and multi-tenancy](#7-state-sessions-and-multi-tenancy)
8. [Backend reliability patterns](#8-backend-reliability-patterns)
9. [Data pipelines and analytics awareness](#9-data-pipelines-and-analytics-awareness)

---

## 1. HTTP and the web's contract

- The protocol as designed, because most "framework knowledge" is HTTP knowledge in disguise: methods and their semantics (safe, idempotent — and why PUT is idempotent while POST isn't, which becomes retry policy), status code families used *honestly* (200-with-error-body as the anti-pattern that breaks every client and monitor), headers as the metadata channel (content negotiation, caching, auth), cookies and their scoping.
- Caching as protocol feature: `Cache-Control`, ETags/conditional requests, `Vary`, what's cacheable where (browser/CDN/proxy) — HTTP caching is the cheapest performance win in most systems and the most commonly ignored.
- Connection reality: TLS handshakes, keep-alive, HTTP/2 multiplexing (head-of-line blocking it solved) and HTTP/3/QUIC (the one it remained to solve), why connection pools exist and how they exhaust.
- **Exercise themes**: assign correct status codes and cache headers for a described API; explain what breaks when POST is retried blindly.

## 2. API design

- **First principle**: an API is a public promise — cheap to make, expensive to keep, nearly impossible to retract (Hyrum's law). Design for the consumer's mental model and for evolution.
- REST pragmatically: resources and representations, URI design (nouns, hierarchy, no verbs), when to deviate from purity (actions that aren't CRUD — `/orders/123/cancel` beats tortured PATCH semantics), pagination (cursor over offset for anything that grows — offset breaks under concurrent writes and invites scraping), filtering/sorting conventions, partial responses.
- The protocol menu with decision rules: REST (default for public/resource-shaped), GraphQL (many clients with divergent data needs; costs: complexity, caching difficulty, the authz-per-field and query-cost problems — cross-link security §7), gRPC (internal service-to-service: contracts + performance; costs: browser hostility, binary debugging), webhooks (push to consumers; delivery semantics and verification — security §7), WebSockets/SSE (server push; SSE as the underused simple option).
- Contract discipline: schema-first (OpenAPI/protobuf) so contracts are reviewable artifacts and clients/servers generate rather than drift; versioning strategies (URL vs header vs additive-only evolution — prefer additive; expand/contract for breaking changes); deprecation as a designed process (announce, measure usage, sunset) not an event.
- Error responses as API surface: consistent problem shape (RFC 7807-style), machine-readable codes for programmatic handling + human messages for debugging, never leaking internals (security), correlation IDs for support.
- Idempotency as a design requirement, not an afterthought: idempotency keys for anything money-shaped or side-effecting (Stripe's API as the canonical study), because clients retry and networks duplicate.
- **Exercise themes**: design the API for a described product through schema-first review; find the five design flaws in a provided OpenAPI spec; design the versioned migration of a breaking change.

## 3. Data modeling

The highest-leverage backend decision — schemas outlive the code that reads them, and a wrong model taxes every feature after it:

- Model the *domain*, not the UI: entities, relationships, cardinality; invariants as the organizing question ("what must always be true?") — invariants determine transaction boundaries and aggregate shapes (cross-link DDD in architecture.md).
- Normalization taught as *why*: redundancy = update anomalies; 1NF→3NF as the discipline, then denormalization as a *measured* performance decision with an owner for each accepted anomaly (not a default).
- The hard modeling problems worth teaching directly: time (validity vs transaction time, bitemporal awareness, "what did we believe on March 3?" — audit and finance care), money (integers-in-minor-units or decimals, never floats; currency as part of the value), identity (natural vs surrogate keys; why emails make terrible primary keys), hierarchies (adjacency list vs path materialization vs closure tables), soft deletion (its query-tax and leak risks vs audit needs), history/audit trails (event-shaped append-only tables vs update-in-place + audit log).
- Schema migration as routine surgery: expand/contract (add nullable → backfill → enforce → drop old), never breaking readers mid-flight, migration testing against production-shaped data volume (the migration that locks a 100M-row table teaches this the hard way).
- **Exercise themes**: model a described domain and defend every table; find the update anomaly in a denormalized schema; write the zero-downtime migration plan for a column split.

## 4. Relational databases at depth

The default persistence choice and the deepest well — most "database problems" are unlearned fundamentals:

- **Indexes as the core mental model**: B-tree mechanics at working depth (sorted structure → point and range queries in log time), composite index column-order logic (equality columns first, then range; the leftmost-prefix rule), covering indexes, why every index taxes writes, reading a query plan as a fluency requirement (`EXPLAIN` — seq scan vs index scan vs nested loop vs hash join and what each implies).
- Transactions and isolation, honestly: ACID unpacked, the anomaly ladder (dirty read → non-repeatable read → phantom → write skew) mapped to isolation levels (read committed → repeatable read/snapshot → serializable), why most databases default weaker than people assume and what bugs that ships (the double-booking/write-skew class — also a security race, cross-link), row locking (`SELECT ... FOR UPDATE`), optimistic vs pessimistic concurrency (version columns), unique constraints as the last line that survives all application bugs.
- Connection reality: pools (sizing against the utilization curve — more is not better), long transactions as the silent killer (bloat, lock queues, replication lag), prepared statements (performance + injection immunity, cross-link security).
- Operational literacy every backend engineer needs even with a DBA: VACUUM/bloat (Postgres), replication (sync vs async, what read replicas actually guarantee — stale reads as a *product* decision), backups tested by restoring (an untested backup is a hope, not a backup), slow-query logging as continuous practice.
- Query-shape discipline: N+1 detection and eradication (ORM literacy — know what SQL your ORM emits; the abstraction leaks and the leak is your job), batch operations, keyset pagination, `LIKE '%x'` and function-wrapped columns as index-defeaters.
- **Exercise themes**: given schema + slow query + plan, fix it; produce the write-skew bug at repeatable-read and fix it three ways (serializable, lock, constraint); size a connection pool and defend it.

## 5. Beyond relational

Polyglot persistence judged by access pattern, not fashion — the decision framework: what are the queries, what's the consistency need, what's the scale reality (most systems: Postgres does everything until it measurably doesn't):

- Key-value and caches (Redis): data structures beyond GET/SET (sorted sets for leaderboards/rate limiters, streams), eviction policies, persistence tradeoffs, cache patterns and their staleness contracts (cross-link architecture.md §8 for invalidation).
- Document stores: schema-on-read flexibility and its bill (the schema still exists — it's just unenforced and scattered across readers), when document shape genuinely fits (content trees, event payloads).
- Wide-column (Cassandra-class): write-optimized LSM storage, query-first modeling (design tables from queries, denormalize by default), tombstone and compaction awareness.
- Search engines (Elasticsearch-class): inverted indexes, analyzers, relevance scoring basics, the sync problem (search index as derived data — cross-link pipelines §9).
- Time-series, columnar/OLAP, vector stores (pointer to ai.md): recognize the workload shapes.
- The umbrella mental model: **derived data** — caches, search indexes, read models, analytics tables are all *derivations* of a source of truth; the design questions are always "how does it stay in sync, how stale can it be, how do we rebuild it?"
- **Exercise themes**: pick stores for four described workloads with access-pattern arguments; design the sync between Postgres and a search index including failure and rebuild.

## 6. Async work and background jobs

- **Why**: the request path is for what the user must wait for; everything else belongs behind a queue. Getting this boundary right is the difference between a backend that degrades gracefully and one that falls over together.
- Queue semantics as contract: at-least-once as the practical default → **consumers must be idempotent** (the single most important sentence in this file; idempotency keys, natural idempotency, dedupe tables), at-most-once and exactly-once honestly (exactly-once = at-least-once + idempotency in practice), ordering guarantees and what they cost (partition-level ordering, the head-of-line tradeoff).
- Job system design: retry policies with backoff + jitter, poison messages and dead-letter queues (with an owner and an alarm — a DLQ nobody reads is a data-loss device with extra steps), visibility timeouts, job priorities and fairness (one tenant's burst starving others — cross-link multi-tenancy §7), graceful shutdown (finish or release, never orphan).
- The transactional outbox: the canonical answer to "write to DB *and* publish an event, atomically" — because dual writes without it produce the ghost-event and lost-event bugs that take weeks to diagnose. Plus its sibling: change data capture.
- Scheduled work: cron-shaped jobs, the missed-run and double-run questions (locking, leader election at awareness level), drift and timezone traps.
- Long-running work: progress reporting, cancellation, checkpointing/resumability; sagas as pointers to architecture.md (multi-step business processes with compensation).
- **Exercise themes**: design the email-sending pipeline (dedupe, retry, DLQ, rate limits); implement outbox-shaped pseudocode and explain each failure window it closes; find the non-idempotent consumer in provided code.

## 7. State, sessions, and multi-tenancy

- Statelessness as the scaling enabler: why session-in-memory breaks horizontal scaling and deploys; externalizing state (signed tokens vs server-side session stores — the tradeoff table, cross-link security §4/§6); sticky sessions as the crutch to know and avoid.
- Multi-tenancy as an architecture decision made early and paid forever: pooled (shared tables + tenant_id — cheapest, and one missing WHERE clause from disaster; RLS as the safety net, cross-link security §10) vs siloed (schema- or database-per-tenant — isolation at operational cost) vs hybrid; tenant context propagation through every layer (the request-scoped tenant ID that must never be ambient-global); noisy-neighbor controls (per-tenant rate limits, quotas, fair queuing).
- Configuration and feature flags as backend state: flag lifecycles (a flag is a temporary fork — retire it or it becomes permanent complexity), config vs code deployment separation, secrets handling pointers (security §2, devops).
- **Exercise themes**: design tenancy for a described B2B SaaS at three growth stages; find the tenant-leak bug in provided code.

## 8. Backend reliability patterns

The request-path disciplines (deeper system patterns live in architecture.md; operational practice in devops.md):

- The resilient-client bundle as reflex, for every outbound call: timeout (always, tuned to the callee's p99, not a folk-number), retry (only idempotent operations, backoff + jitter, budgeted — retry storms turn a blip into an outage), circuit breaker (fail fast when the callee is down; half-open probing), fallback (cached/default/degraded answer where the product allows — a *product* conversation).
- Rate limiting and admission control: token bucket vs sliding window, per-what (user/tenant/IP/endpoint), 429 + Retry-After as honest signaling, load shedding (rejecting cheaply at the front beats collapsing expensively in the middle) and priority shedding (drop analytics before checkout).
- Health and lifecycle: liveness vs readiness (the distinction that prevents self-inflicted rolling-restart outages), graceful shutdown sequencing (stop accepting → drain → finish jobs → close pools), startup dependencies and crash-loop hygiene.
- The request-path audit as a teachable review: for each hop — what's the timeout, what happens on failure, is it retried, is it idempotent, what does the user see, what does the operator see (log/metric/trace — devops.md)?
- **Exercise themes**: run the request-path audit on a described three-service flow and find the five gaps; tune timeout/retry/breaker numbers for given SLOs and defend them.

## 9. Data pipelines and analytics awareness

Backend engineers feed the data platform whether they know it or not — awareness level:

- OLTP vs OLAP as workload shapes (row vs columnar storage, why analytics queries murder the production DB and how replicas/warehouses answer it); the warehouse/lake vocabulary (ELT vs ETL, star-schema basics).
- Event instrumentation as API design: analytics events are contracts too — naming, schemas, versioning (the analytics team is a consumer; breaking their events silently is a production incident with a delay).
- Batch vs streaming basics: windowing, late data, watermarks at concept level; when a nightly batch honestly suffices (usually).
- Privacy plumbing: PII classification at write time, retention and deletion as designed capabilities (GDPR deletion across derived data is brutal if unplanned — cross-link security §17-18).
- **Exercise themes**: design the event schema for a checkout funnel; plan user-deletion across a system with caches, search indexes, and a warehouse.

---

## Review checklist (backend changes)

1. API: contract documented, errors shaped consistently, breaking-change check, pagination on anything unbounded.
2. Data: migration reversible/expand-contract, indexes for new query shapes, no N+1, transaction boundaries match invariants.
3. Concurrency: race windows (check-then-act), idempotency for anything retried, unique constraints as backstop.
4. Outbound calls: timeout, retry policy, breaker, fallback — all four consciously decided.
5. Async: consumers idempotent, DLQ + alarm, graceful shutdown.
6. Tenancy/authz: tenant scoping on every query (security §5), no ambient tenant state.
7. Observability: new paths logged/measured/traced, correlation IDs propagated (devops.md).
8. Security quick pass: security.md §17 items 1–9.

## Case study themes (L3+)

Stripe's idempotency and API versioning discipline; the classic retry-storm outage shape (AWS DynamoDB 2015 as one public example); GitHub's MySQL partitioning journey; "one missing WHERE tenant_id" incident write-ups; Slack's job-queue redesign. For each: forces → decision → outcome → what would you have done.
