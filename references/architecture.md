# Architecture & Distributed Systems Curriculum

System-scale design: the discipline of shaping software so it survives growth, failure, and years of change. Prerequisites: software-engineering.md core, backend.md fundamentals. The Staff-level skill is not knowing patterns — it is matching pattern to context and pricing every choice. For every pattern: the problem it solves, what it costs, and where it is the *wrong* answer. An architect who can't argue against their own preferred pattern hasn't learned it.

## Table of contents

1. [What architecture is](#1-what-architecture-is)
2. [Coupling, cohesion, and modularity](#2-coupling-cohesion-and-modularity)
3. [Design principles: SOLID and beyond](#3-design-principles-solid-and-beyond)
4. [Architectural styles in the small](#4-architectural-styles-in-the-small)
5. [Domain-driven design](#5-domain-driven-design)
6. [Monoliths, microservices, and the middle](#6-monoliths-microservices-and-the-middle)
7. [Distributed systems fundamentals](#7-distributed-systems-fundamentals)
8. [Caching](#8-caching)
9. [Event-driven architecture](#9-event-driven-architecture)
10. [CQRS and event sourcing](#10-cqrs-and-event-sourcing)
11. [Scaling playbook](#11-scaling-playbook)
12. [High availability and resilience](#12-high-availability-and-resilience)
13. [Architecture as a practice](#13-architecture-as-a-practice) — decisions, documents, reviews, evolution

---

## 1. What architecture is

- **Working definition**: the set of decisions that are expensive to change — everything else is just code. Architecture's job is to keep the cost of *future* change low while meeting today's quality attributes.
- Quality attributes ("-ilities") as the real requirements: performance, availability, scalability, security, maintainability, operability, cost — features say what the system does; attributes say how well, and architecture is mostly attribute engineering. Attributes conflict (consistency vs availability, flexibility vs simplicity); architecture is choosing which to favor *where*.
- Forces → decision → consequences as the reasoning template: team size and skill, domain complexity, scale reality (not fantasy), latency/consistency needs, budget, deadline. The same requirements with a different team yield a different correct architecture.
- Conway's law as a load-bearing fact, not trivia: system shape mirrors communication structure; the inverse maneuver (shape teams to get the architecture you want) is a real Staff-level tool. Cross-link career.md.
- **The four review questions** — answer explicitly for any system under discussion: (1) why this architecture — which forces pushed here; (2) alternatives — 2–3 real options at the same altitude; (3) tradeoffs — what each buys and costs, with the losing options' appeal stated honestly; (4) when NOT to use this — the contexts where it actively hurts. Question 4 separates pattern-appliers from architects.

## 2. Coupling, cohesion, and modularity

The root concepts under every pattern — teach first, everything else is application:

- Coupling = the degree to which changing one thing forces changing another; **change amplification is the real cost metric**. Types worth naming: data coupling (fine) → control coupling → temporal coupling (hidden ordering requirements — insidious) → content coupling (reaching into internals — worst).
- Cohesion = whether a module's contents belong together; "modules that change together live together" as the packaging rule (this is also the argument for vertical/feature slicing over horizontal/layer slicing in codebases).
- Information hiding (Parnas, again — same principle as software-engineering §2, now at module scale): a module boundary hides a *decision*; good decomposition = decisions likely to change independently are hidden behind different boundaries.
- Dependency direction as the architectural lever: depend on stable things; stable things don't depend on volatile things. Every layered/hexagonal/clean diagram is this one rule drawn different ways.
- Afferent/efferent coupling, instability metrics at awareness level — useful as smells (the module everything imports *and* that imports everything is the problem child).
- **Exercise themes**: given a change request, trace its blast radius through a described codebase and propose the boundary that would have contained it; find the temporal coupling in a provided API.

## 3. Design principles: SOLID and beyond

- SOLID as heuristics, not laws — each with its violation's cost and its over-application's cost: SRP (one reason to change; over-applied → confetti of two-line classes), OCP (extension points where variation is *expected*; over-applied → speculative plugin systems nobody extends), LSP (behavioral subtyping — the subtle one; violations surface as instanceof-checks and surprised callers), ISP (client-shaped interfaces), DIP (depend on abstractions *at volatility boundaries* — not every class needs an interface; the test seam and the swap-ability must be worth the indirection).
- The principles that deserve equal billing: composition over inheritance (inheritance couples to implementation; the fragile-base-class problem), least astonishment, separation of policy from mechanism, make illegal states unrepresentable (type-level architecture), the rule of three / YAGNI as the counterweight to all of the above.
- **Anti-pattern gallery**: the god object, anemic domain model (data bags + procedure managers — OOP's shape without its point), distributed monolith (microservices' cost with monolith's coupling — the worst quadrant), lasagna (so many layers each does nothing), golden hammer.
- **Exercise themes**: given code, name which principle each smell violates and fix the two worth fixing (and defend leaving the rest); find the LSP violation by writing the test that exposes it.

## 4. Architectural styles in the small

Application-internal organization — the shape of one deployable:

- Layered (presentation/domain/data): the default; its failure mode is the pass-through layer and business logic leaking into controllers.
- Hexagonal / ports-and-adapters / clean architecture (one idea, three brandings): domain at the center, I/O at the edges, dependency rule pointing inward; testability falls out free. **When it's overkill**: a CRUD app with three endpoints doesn't need four layers — ceremony has a carrying cost; start simpler and introduce the boundary when the domain earns it.
- Vertical slice / feature-based organization: optimize for the axis of change (features change together; layers don't); pairs with "modules that change together live together" from §2 — often the honest choice for product codebases.
- Pipes and filters, plugin architectures, micro-kernel: recognize the shapes and their homes (data processing, extensible platforms, IDEs).
- **Exercise themes**: reorganize a described layered codebase into vertical slices and state what got easier and harder; decide (and defend) how much architecture a described new product deserves on day one.

## 5. Domain-driven design

- Strategic DDD first — it matters most and travels furthest: ubiquitous language (the same words in conversation, code, and DB — divergence is where bugs and misrequirements breed), bounded contexts (the same word means different things in different subdomains — "customer" in billing ≠ support; boundaries where language changes), context mapping (upstream/downstream, anti-corruption layers to keep another context's model from colonizing yours), core vs supporting vs generic subdomains (spend design effort on the core; buy the generic).
- Tactical DDD second, judged case by case: entities vs value objects (identity vs value equality — value objects are underused everywhere), aggregates as invariant-protection boundaries (the aggregate root guards the transaction — this decides your transaction and API shapes, cross-link backend.md §3), domain events (something happened, stated in domain language — the bridge to §9), repositories honestly (an abstraction worth it at domain boundaries, ceremony when wrapping an ORM 1:1).
- **When DDD earns its cost**: complex, differentiating domain logic with domain experts to mine. When it doesn't: CRUD, integrations, generic subdomains — modeling ceremony there is procrastination.
- **Exercise themes**: run a mini event-storming on a described business and propose bounded contexts; find the aggregate boundary that makes a described invariant enforceable without cross-aggregate transactions.

## 6. Monoliths, microservices, and the middle

- The honest tradeoff, stated once and repeated: **microservices trade code-level complexity for operational and distributed-systems complexity.** You exchange function calls for network calls, transactions for sagas, refactors for migrations, one deploy for a deployment matrix — and buy independent deployment, independent scaling, team autonomy, and fault isolation.
- The modular monolith as the default recommendation: enforce module boundaries in-process (compile-time visibility, separate schemas per module) — you get most of the coupling benefits, keep transactions and refactorability, and preserve the *option* to extract services along the boundaries you've proven. Extraction when a *specific* force demands it: a team stepping on others' deploys, a component with wildly different scaling or availability needs, a compliance isolation requirement.
- Service boundary design = bounded contexts (§5) + data ownership: each service owns its data, no shared databases (the shared DB is the distributed monolith's signature); API contracts between contexts; the distributed transaction question answered by sagas (§9) or by redrawing the boundary so the invariant lives inside one service (usually better).
- Sizing honestly: "micro" was always a misnomer — service-per-team or service-per-context, not service-per-entity; the 4-engineer startup with 12 services is the canonical self-inflicted wound.
- Migration paths: strangler fig at system scale (cross-link software-engineering §4), branch-by-abstraction, parallel-run verification.
- **Exercise themes**: the scenario debate — "team of 4, deadline 3 months, investors say microservices" (judged on tradeoff articulation); draw service boundaries for a described domain and identify which boundary the data model makes a lie.

## 7. Distributed systems fundamentals

The physics that every pattern above sits on — teach as *constraints*, not trivia:

- The fallacies of distributed computing as the syllabus header: the network is not reliable, latency is not zero, bandwidth is not infinite, the network is not secure (cross-link security §11), topology changes, there is no single admin, transport is not free.
- Failure is partial and ambiguous: a timeout does not tell you whether the work happened — this single fact generates idempotency requirements (backend.md §6), retry discipline, and reconciliation jobs. Design conversations that skip it are fantasy.
- Time and order: no global clock; clock skew is real (never use wall-clock time for ordering or locks); logical clocks and happens-before at concept level; "what happened first" is a design decision, not an observation.
- Consistency vocabulary, precisely: linearizability (reads see the latest write, globally) → sequential → causal → eventual; read-your-own-writes and monotonic reads as the *product-visible* guarantees users actually notice (the profile-photo-flicker bug). CAP honestly (partition tolerance isn't optional, so it's C-vs-A *during partitions*) and PACELC (the latency-vs-consistency trade you pay *all the time*, not just during partitions).
- Replication and partition basics: leader/follower (and the failover data-loss window), quorums (R+W>N intuition), multi-leader conflicts, partitioning/sharding (hash vs range, hot keys, resharding pain, cross-shard queries as the tax) — enough to *converse with* databases' documentation and choose honestly.
- Consensus at working-awareness level: why it's needed (leader election, distributed locks, config), why it's expensive, and the practical rule — never build your own; use the battle-tested primitive (etcd/ZooKeeper-class) or better, design to not need it.
- Exactly-once as theology: at-least-once delivery + idempotent processing is the real-world implementation of "exactly-once" — repeated from backend.md because it's the most valuable sentence in distributed systems.
- **Exercise themes**: given a sequence diagram with a timeout, enumerate every possible system state; design read-your-own-writes on top of async replication; explain to a PM why "just make it consistent everywhere" costs 10x latency.

## 8. Caching

- The economics: a cache converts consistency into latency and cost savings — every cache adds a *staleness contract* someone must own. State the contract explicitly ("this data may be up to 60s stale, and here's who signed off") or the cache is a bug generator.
- Placement ladder and what each layer buys: client (HTTP cache, frontend.md §6) → CDN (static + increasingly dynamic edge) → app-level (in-process vs shared/Redis — the coherence-vs-cheapness trade) → database-internal (buffer pools you get free).
- Patterns with their failure modes: cache-aside (the default; its subtle race — stale-set-after-invalidation), read-through/write-through (consistency at latency cost), write-behind (throughput at durability risk).
- Invalidation as the famous hard problem, made concrete: TTL (simple, staleness-bounded, thundering-herd-prone), explicit invalidation (precise, and the delivery of the invalidation message is itself a distributed-systems problem), versioned/keyed (immutable entries + key changes — sidesteps invalidation; the content-hash asset pattern from frontend builds).
- The operational traps taught by war story: stampede/thundering herd (locks, single-flight, jittered TTLs, early refresh), hot keys (the celebrity problem — local caching in front of the shared cache), cache penetration (negative caching for "doesn't exist"), the cold-start after a flush (a full cache is load-bearing infrastructure; losing it is an outage class of its own).
- **When not to cache**: before measuring, when staleness is unsellable (balances, inventory at checkout), when the query is cheap — a cache you didn't need is pure added failure surface.
- **Exercise themes**: add caching to a described read path — pick placement, pattern, TTL, invalidation, and write the staleness contract; diagnose a described stampede from its symptoms and fix it two ways.

## 9. Event-driven architecture

- Events vs commands vs queries — the semantic triad: an event states a fact ("OrderPlaced"), owned by the producer, zero-or-many consumers; a command requests an action, one handler, can be rejected. Naming them honestly matters: a "UserShouldBeEmailed event" is a command in a trench coat, and the mislabel obscures who owns the outcome.
- What events buy and cost: temporal decoupling and independent evolution (add a consumer without touching the producer) — paid for with eventual consistency (product-visible! cross-link §7), harder debugging (the flow exists in no single codebase — correlation IDs and tracing become mandatory, devops.md), and choreography sprawl.
- Orchestration vs choreography: a saga orchestrator (explicit state machine, one place to read the flow, one place to fail) vs event chains (maximal decoupling, emergent behavior nobody can trace past five hops); default to orchestration for business-critical multi-step flows, choreography for genuinely independent reactions.
- Sagas properly: compensation as the substitute for rollback (and compensations can fail too — the pattern is a state machine with human-escalation terminal states, not magic), the pending/confirmed modeling trick (design states so mid-saga is a legal, visible state).
- Delivery mechanics (backend.md §6 owns the queue details): the outbox as the producer-side integrity pattern, consumer idempotency as the consumer-side one; schema evolution for events (additive-only, versioned envelopes — an event published is a contract shipped, and you don't control the consumers' upgrade schedule).
- Event streaming vs message queues (Kafka-class vs RabbitMQ-class): log with replay + multiple independent readers vs work distribution; replayability as the underrated superpower (rebuild a projection, backfill a new consumer) and retention/compaction as its price.
- **Exercise themes**: convert a described synchronous checkout into a saga — draw the state machine including every failure and compensation; find the command-in-event-clothing in a described system and fix the ownership.

## 10. CQRS and event sourcing

Separated from §9 because conflating them (and reaching for them together by default) is the classic over-engineering move:

- CQRS in its cheap form: separate read models from write models — often just "denormalized views/replicas for queries, normalized model for commands"; justified whenever read and write shapes genuinely diverge (search, dashboards, feeds). The expensive form (separate stores, async projection) inherits the derived-data sync problem (backend.md §5) — pay it only when the cheap form measurably fails.
- Event sourcing honestly: the event log as source of truth, state as a fold over events; what it buys (perfect audit, temporal queries, replay/rebuild, natural fit for event-native domains — ledgers, trading, version control) and what it costs (schema evolution over *immutable history* — upcasters forever; snapshotting for replay cost; eventual-consistent projections; a paradigm your whole team must internalize; nearly impossible to retrofit *off*).
- The decision rule: most systems need neither; some need cheap CQRS; few need event sourcing — and those few usually know because auditability or temporality *is the product requirement*, not because the pattern is admired.
- **Exercise themes**: identify which of four described systems justifies which tier, with the requirement that forces it; design the event schema evolution for a two-year-old event-sourced aggregate gaining a field.

## 11. Scaling playbook

The applied sequence — because real scaling is a staircase of bottleneck removals, not a leap to "web scale":

- **Step 0: measure** — the bottleneck is a fact to find (utilization, saturation, queue depths — devops.md observability), not a vibe. Capacity estimation as a craft: back-of-envelope QPS/storage/bandwidth (interview staple, real skill; software-engineering §8).
- The canonical staircase, each step with its trigger: vertical scaling (embarrassingly effective, do it first — hardware is cheaper than engineering), stateless app tier + load balancer (requires the state externalization from backend.md §7), read replicas (for read-heavy; brings replication-lag product decisions), caching (§8 — after measuring, with contracts), async-ing the write path (queues absorb bursts — backend.md §6), search/analytics offload (stop torturing the OLTP database), CDN/edge (move latency to the speed of light's schedule), sharding (last, because it taxes every query, transaction, and migration forever after — exhaust the other steps first).
- Load balancing literacy: L4 vs L7, algorithms (round-robin vs least-connections; consistent hashing for cache-affinity), health-check integration (the LB is your first availability mechanism), connection draining for deploys.
- The bottleneck whack-a-mole truth: fixing one bottleneck reveals the next (the DB you saved now exposes the single-threaded worker); scaling work is iterative by nature, so architecture's job is keeping *options* open rather than pre-building all steps.
- **Exercise themes**: take a described single-server app through three order-of-magnitude growth steps, choosing the minimal intervention each time and naming what breaks next; capacity-estimate a described product from user counts down to QPS and storage.

## 12. High availability and resilience

- Availability math that shapes everything: nines and their yearly minutes; serial composition multiplies failure (five 99.9% dependencies ≈ 99.5% ceiling — your availability is capped by what you depend on synchronously, which is the deepest argument for async decoupling); redundancy math only works when failures are *independent* — correlated failure (same AZ, same deploy, same certificate expiry) is the silent assumption-breaker.
- **Nines are a product decision**: each nine multiplies cost and constrains architecture; "how much downtime can the business actually absorb, per feature?" — checkout and the admin panel do not deserve the same SLO. Cross-link devops.md SLOs/error budgets, where this gets operationalized.
- Redundancy mechanics: active-passive vs active-active, failover honestly (detection time + promotion time + the split-brain risk; fencing), multi-AZ as the sane default, multi-region as an order-of-magnitude cost/complexity jump (data replication lag, traffic steering, and the question nobody answers: have you ever *tested* the regional failover?).
- Resilience patterns at system scale (request-path versions in backend.md §8): bulkheads (isolate pools so one dependency's failure can't drown the rest), load shedding with priorities (drop analytics before checkout), graceful degradation as designed product behavior (the degraded-mode matrix: which features shed in which order — a product-design collaboration, cross-link design.md), backpressure end-to-end (unbounded queues convert overload into total collapse; bounded queues convert it into honest slowness).
- The failure-feedback loops that cause the big outages: retry storms (the blip that self-DDoSes — retry budgets, cross-link backend.md), cascading failure (slow is worse than down: a slow dependency consumes caller threads and spreads — timeouts and breakers are containment, not decoration), metastable states (systems that can't recover without intervention — the cold-cache restart death spiral from §8).
- Testing what you claim: chaos engineering's actual point (verify resilience *hypotheses* in controlled experiments — "we believe we survive one AZ loss; let's find out on Tuesday at 2pm, not during the incident"), game days, and the rule that an untested failover is a rumor.
- **Exercise themes**: compute the availability ceiling of a described synchronous chain and redesign to raise it; write the degraded-mode matrix for a described product; storyboard the cascade from one slow dependency through an untimeouted call graph.

## 13. Architecture as a practice

How architecture actually happens in organizations — the part that makes it a Staff skill rather than a diagramming hobby:

- Architecture Decision Records (ADRs): context → decision → consequences, immutable once accepted, superseded rather than edited; the value is the *why* preserved for the engineer (or you) three years out. Lightweight and consistent beats elaborate and abandoned.
- Design documents / RFCs as the real design tool: writing forces rigor that diagrams and meetings launder away (career.md owns the writing craft; here: the architecture sections — requirements and *non*-requirements, quality attributes with numbers, alternatives seriously costed, failure analysis, rollout and rollback).
- Architecture review as practiced well: review the *problem framing* first (most bad architectures answer the wrong question correctly), then apply the four questions from §1; red-team the failure modes ("what's the worst Tuesday this design can have?"); the reviewer's job is surfacing unpriced risk, not imposing taste.
- Evolutionary architecture: fitness functions (executable checks on architectural properties — dependency-direction tests, latency budgets in CI — so the architecture is enforced, not aspirational), the last responsible moment for decisions (keep options open; decide when information peaks), sacrificial architecture (building the version you'll throw away, *knowingly*, as a legitimate strategy — the sin is accidental permanence).
- Diagramming that communicates: C4's altitude discipline (context → container → component — most diagram confusion is mixed altitudes), data-flow diagrams for security review (security.md §1), sequence diagrams for the failure cases (the happy path rarely needs the diagram).
- **Exercise themes**: write the ADR for a decision made in a previous lesson's exercise; run a red-team review on a provided design doc (find the three unpriced risks); define two fitness functions for a described architecture and sketch their CI implementation.

---

## Review checklist (architecture / design review)

1. Problem framing: is this the right problem? requirements *and* explicit non-requirements? quality attributes with numbers, not adjectives?
2. The four questions (§1): forces named, alternatives real, tradeoffs priced, when-not-to-use stated.
3. Failure analysis: every network hop's timeout/retry/fallback; partial-failure states enumerated; the worst-Tuesday storyboard.
4. Data: ownership boundaries clean, consistency needs named per flow (not globally), derived-data sync and rebuild story, migration path.
5. Scale honesty: designed for measured/projected load, not fantasy load; the *next* bottleneck identified; options kept open.
6. Coupling: blast radius of the most likely changes traced; shared-database and distributed-monolith smells checked.
7. Operability: deploy/rollback story, observability plan, degraded modes designed (devops.md).
8. Security: trust boundaries drawn, service-to-service auth, secrets, blast-radius segmentation (security.md §11).
9. Organization: does the design fit the team that must build and run it? (Conway check — a 4-person team with a 12-service design fails this line.)
10. Reversibility: which decisions are one-way doors, and are they flagged for the extra scrutiny they deserve?

## Case study themes (L3+)

Amazon's move to services (the API mandate and what forced it); the Segment microservices→monolith return (honest cost accounting); Shopify's modular monolith at extreme scale (the road not taken to microservices); the AWS S3 2017 outage (blast radius and dependency chains); Cloudflare's regex outage (global blast radius of config); Knight Capital (deployment + feature-flag hygiene as architecture); GitHub's October 2018 split-brain (the failover data-loss window made real). For each: forces → decision → outcome → what would you have done.
