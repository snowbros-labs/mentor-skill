# Core Software Engineering Curriculum

The trunk of the whole curriculum — every other track assumes this one. Covers the craft of writing, testing, changing, debugging, and speeding up code. The unifying mental model: **software engineering is programming integrated over time** (Google's definition) — every practice here exists because code is read, changed, and operated far more than it is written.

## Table of contents

1. [Code quality and readability](#1-code-quality-and-readability)
2. [Naming, abstraction, and API design](#2-naming-abstraction-and-api-design)
3. [Testing](#3-testing)
4. [Refactoring and working with legacy code](#4-refactoring-and-working-with-legacy-code)
5. [Debugging methodology](#5-debugging-methodology)
6. [Data structures and algorithms, pragmatically](#6-data-structures-and-algorithms-pragmatically)
7. [Concurrency and parallelism](#7-concurrency-and-parallelism)
8. [Performance engineering](#8-performance-engineering)
9. [Error handling and robustness](#9-error-handling-and-robustness)
10. [Language and paradigm literacy](#10-language-and-paradigm-literacy)
11. [Version control fluency](#11-version-control-fluency)
12. [Dependencies and reuse judgment](#12-dependencies-and-reuse-judgment)

---

## 1. Code quality and readability

- **First principle**: code is communication to humans that happens to execute. The reader's working memory is the constrained resource; every quality practice reduces reader load.
- Readability mechanics: functions that do one thing at one altitude (no mixing policy with mechanism), early returns over nesting, guard clauses, small scopes, locality (things used together live together).
- Comments: explain *why* and *constraints*, never *what* — a comment restating the code is noise that rots; a comment explaining the invariant or the weird-looking workaround's reason is gold.
- Code smells as vocabulary: long parameter lists, feature envy, primitive obsession, shotgun surgery, divergent change, data clumps, speculative generality. Teach students to *name* smells — naming enables discussion and literature search.
- Simplicity discipline: YAGNI (the cost of speculative flexibility is real and usually unpaid-off), duplication vs wrong abstraction ("duplication is far cheaper than the wrong abstraction" — Sandi Metz; rule of three before extracting).
- Consistency beats personal preference: match the codebase's idiom; style arguments are solved by formatters and linters, not review comments.
- **Anti-patterns**: cleverness (code golf in production), premature abstraction, config for things that never vary, "manager/util/helper" grab-bag modules.
- **Exercise themes**: refactor a readable-but-smelly function; review a PR with planted smells; defend or kill an abstraction given three call sites.

## 2. Naming, abstraction, and API design

Split from "readability" because interface design is its own skill and the highest-leverage one:

- Names carry the design: a name you can't choose cleanly signals a concept you haven't isolated. Names state intent (`remainingRetries` not `count`), match domain language (DDD's ubiquitous language at code level), and get longer as scope widens.
- Abstraction = hiding a decision behind a boundary so it can change independently (Parnas). The question is never "is this abstract enough" but "which decisions does this hide, and are they the ones likely to change?"
- Deep vs shallow modules (Ousterhout): the best interfaces are small relative to the functionality they hide; a shallow wrapper that renames another API is negative value.
- API design as contract design: preconditions, postconditions, invariants; make illegal states unrepresentable (type-driven design); design for the caller's mental model, not the implementation's convenience.
- Evolution: additive change beats breaking change; versioning and deprecation as first-class design activities; Hyrum's law (every observable behavior will be depended on — someone parses your error strings).
- **Interview thread**: "design the interface for X" questions are this section; practice narrating decision-hiding rationale.

## 3. Testing

- **Why**: tests are the enabling constraint for everything else — refactoring, performance work, and dependency upgrades are all gated on the ability to detect breakage cheaply. Untested code is unchangeable code.
- **Mental model**: a test is a specification of behavior that happens to be executable. Test *behavior through public interfaces*, not implementation — implementation-coupled tests punish refactoring, which inverts their purpose.
- The pyramid and its critics: many fast unit tests, fewer integration, few E2E — but the shape matters less than the principle: push each assertion to the cheapest level that can catch its failure. The "testing trophy" (integration-heavy) fits systems where units are trivial and wiring is the risk.
- Test quality: one behavior per test, arrange-act-assert, descriptive names that read as specs (`rejects_expired_tokens` not `test_auth_3`), test data builders over 40-line setups, determinism (flaky tests destroy trust and then all tests get ignored — treat flakes as P1 bugs).
- Doubles taxonomy and judgment: stub/fake/mock/spy; over-mocking as the classic failure (a test that mocks everything verifies nothing but your mocks; prefer fakes and real collaborators where cheap).
- TDD honestly: red-green-refactor as a design pressure and focus tool; works brilliantly for algorithmic/contract-shaped problems, awkward for exploratory UI work — teach it as a tool, not a religion.
- Beyond example-based: property-based testing (state invariants, let the framework hunt inputs), mutation testing (tests your tests), snapshot tests (cheap but assert-everything-verify-nothing risk), contract tests (service boundaries), load and soak tests (§8), accessibility tests.
- Coverage: a useful *negative* signal (uncovered code is definitely untested) and a dangerous target (Goodhart — 100% coverage with assertion-free tests is worse than honest 70%).
- **Production considerations**: CI as the enforcement point, test speed as developer-experience budget (a 40-minute suite means people batch changes and skip runs), quarantine workflows for flakes.
- **Exercise themes**: write tests for given code, then mutate the code and see what escapes; receive code with a bug and write the failing test *first*; refactor an over-mocked suite.

## 4. Refactoring and working with legacy code

- **Why**: most professional work is changing existing code; greenfield is the rare case. The skill ceiling here separates L2 from L3.
- Refactoring = behavior-preserving transformation in small, reversible steps, each keeping tests green. Big-bang rewrites are the anti-pattern: they lose embedded knowledge, freeze features, and usually die (teach the case-study literature — Netscape).
- Core moves by name: extract function/variable, inline, rename, move, replace conditional with polymorphism, introduce parameter object, replace magic literal. Vocabulary again — named moves compose into plans.
- Legacy code (Feathers): legacy = code without tests. Seams — places to break dependencies so tests can attach; characterization tests — pin current behavior (bugs included) before changing anything; sprout and wrap techniques for adding to untestable code without expanding it.
- Strangler fig for system-scale migration: new implementation grows around the old at the boundary, traffic shifts incrementally, old dies when starved. Connects to expand/contract (parallel change) for API and schema migrations — never break in one step, always expand → migrate → contract.
- Tech debt as vocabulary and portfolio: deliberate vs inadvertent debt (Fowler quadrant), debt as leverage (sometimes correct!) vs debt as neglect; making debt visible (debt register, "rule of maintenance days") so paydown is a decision, not a plea.
- **When NOT to refactor**: code that never changes and works; right before a deadline without tests; refactoring as procrastination from the actual task.
- **Exercise themes**: characterization-test an ugly function then improve it; plan (in writing) a strangler migration for a described monolith; find the seams in dependency-tangled code.

## 5. Debugging methodology

- **Why**: debugging is 30–50% of engineering time and almost never taught as a discipline. The difference between L1 and L3 debugging is method, not intuition.
- **The scientific loop**: observe → hypothesize → predict → test → conclude, one variable at a time. The cardinal sin is shotgun debugging — changing things until it works teaches nothing and often "fixes" by coincidence.
- Core tactics, each with when-to-use: reproduce first (a bug you can't reproduce, you can't verify fixed; shrink to minimal repro), read the error *actually* (the answer is in the message a shocking fraction of the time), bisect (git bisect over history, binary-search over inputs/config/code path — halving beats guessing), differential debugging (works there, fails here — diff the environments), rubber-duck/write it out (explaining forces serialization of assumptions — the assumption you can't defend is usually the bug), instrument (logging/debugger/tracer — and know when printf beats a debugger: distributed and timing-sensitive bugs).
- Heisenbugs, race conditions, and environment-dependent bugs: why observation changes behavior, why "can't reproduce locally" points at environment or concurrency, strategies (record/replay, stress + sanitizers §7, production observability §devops).
- The bug is never where you're sure it is: confirmation bias in debugging; the discipline of listing assumptions and testing the "obviously fine" ones after the third failed hypothesis.
- Post-fix discipline: understand *why* it broke and why the fix works (a fix you can't explain is a coincidence you shipped); add the regression test; ask "where else does this pattern exist?"
- **Exercise themes**: seeded-bug hunts with a hypothesis log (grade the process, not just the find); a "no-touch" debugging round (diagnose from logs/stack trace alone); root-cause a race condition.

## 6. Data structures and algorithms, pragmatically

- **Why**: not for leetcode (that's `career.md` interview prep) — because data-structure choice is the most common silent performance decision in real code, and Big-O intuition is what makes you *notice* the accidental O(n²).
- Working set: arrays/lists (locality wins — the vector beats the linked list in practice almost always, cache lines are the reason), hash maps (the workhorse; know collision behavior and iteration-order guarantees of your language), sets, stacks/queues/deques, heaps (top-K, schedulers), trees (sorted iteration, range queries — usually via your DB's B-tree), tries (prefix problems), graphs (model, then BFS/DFS/Dijkstra/topo-sort as the four you actually use).
- Big-O with honesty: asymptotics for growth-thinking, constants for reality (n=50 means the O(n²) with tiny constants wins); amortized analysis (why vector push is O(1)-ish); space/time trades as the default lever.
- The real-world patterns: hashing for dedupe/join/memoize, sorting as preprocessing that unlocks binary search and merging, sliding windows and two pointers for stream/sequence work, memoization vs tabulation when overlapping subproblems appear.
- Beyond memorization: recognize *problem shapes* — "this is top-K", "this is a graph reachability question", "this needs a monotonic stack" — shape recognition is the transferable skill.
- **Exercise themes**: find the accidental O(n²) in a provided PR; pick structures for described workloads and defend with access patterns; estimate before measuring, then measure (§8) and reconcile.

## 7. Concurrency and parallelism

- **Why**: all modern systems are concurrent (async servers, UI event loops, multi-service backends); concurrency bugs are the most expensive class to find late.
- **First principles**: concurrency = structure (dealing with many things), parallelism = execution (doing many things) — Pike's distinction. The root of all evil: *shared mutable state*; every model below is a strategy for taming it.
- The models, with when-each-wins: shared memory + locks (mutexes, RW locks; deadlock conditions and the lock-ordering discipline that prevents them), message passing (channels/CSP, actors — share memory by communicating), async/event-loop (single-threaded concurrency: no data races within the loop, but starvation and blocking-the-loop hazards; what async actually buys — I/O concurrency, not CPU parallelism), data parallelism (map-reduce shape, worker pools), immutability + functional composition (no shared mutation, no problem — the quiet winner where throughput allows).
- The bug taxonomy: race conditions (TOCTOU — also a security bug, cross-link security.md §10), deadlock/livelock/starvation, atomicity violations, memory-visibility and reordering (why "it works on my machine" is a memory-model statement; happens-before as the concept worth teaching even without the formalism).
- Practical discipline: keep critical sections tiny, prefer higher-level primitives (queues, executors, `async`/`await`) over raw threads and locks, make data flow one-directional where possible, idempotency for anything retried (cross-link backend.md), sanitizers and race detectors (TSan, `go race`) in CI.
- Backpressure as the system-level concurrency concept: unbounded queues turn overload into memory death; bounded queues + shed/block/drop decisions propagate slowdown honestly (cross-link architecture.md).
- **Exercise themes**: find the race in provided code and fix it three ways (lock, channel, redesign to immutable); explain why an async handler stalls a whole service; design a worker pool with backpressure and graceful shutdown.

## 8. Performance engineering

- **Why**: performance is a feature (conversion, cost, UX) and a system property that decays silently. The discipline is measurement-driven — the guessing tax is enormous.
- **The iron law: measure before optimizing.** Profile, find the actual hotspot (it is not where you think — same lesson as debugging §5), fix, re-measure. Amdahl's law bounds the win: optimizing 10% of runtime caps at 10%.
- Mental models: latency vs throughput vs utilization (and how they trade — the utilization/latency hockey stick from queueing theory: past ~70–80% utilization, latency explodes; this one graph explains half of production behavior), percentiles over averages (p50 lies, p99 is where users suffer, tail amplification in fan-out systems — one slow shard makes every request slow), the memory hierarchy (L1 ~1ns → RAM ~100ns → SSD ~100µs → network ~1ms+ — orders of magnitude that make locality and batching the two universal optimizations).
- The universal levers, in typical payoff order: do less work (algorithmic — §6), do it fewer times (caching, memoization — with invalidation cost priced in, cross-link architecture.md), batch it (amortize per-call overhead — the N+1 query is this lesson), do it later (async, queues — move work off the latency path), do it in parallel (§7 — after the others; parallelizing waste is still waste), do it closer (locality: CPU cache, CDN, data co-location).
- Tooling literacy: profilers (sampling vs instrumenting), flame graphs (read one fluently), benchmarking pitfalls (warmup, variance, optimizing the benchmark instead of the workload, microbenchmarks that lie about real systems), load testing (closed vs open workload models — open models reveal the death spiral closed ones hide), continuous performance regression tracking in CI.
- Resource-specific chapters (pointers): memory (allocation pressure, GC behavior, leaks in managed languages — retained references; leak-hunting with heap snapshots), I/O (sync vs async, buffering, zero-copy awareness), serialization overhead (JSON vs binary at high QPS).
- **When NOT to optimize**: before measuring, below user-perceptible thresholds, at the cost of clarity for non-hot paths — premature optimization is real, but so is premature pessimization (choosing the obviously-worse structure out of laziness); the fix for both is knowing which paths are hot.
- **Exercise themes**: profile a provided slow program, produce a flame graph, fix the top hotspot only; find the N+1; explain a p99 regression that left p50 untouched; capacity-estimate a described system (back-of-envelope is an interview staple and a real skill).

## 9. Error handling and robustness

- **First principle**: errors are not exceptional — they are half the state space, and the half that ships unreviewed kills systems. Design the failure path with the same care as the success path.
- The strategy question per error: can the *caller* act on this? Recoverable (retry, fallback, degrade) vs unrecoverable (crash fast and loudly beats limping into corruption). Fail fast at boundaries, be tolerant in cores.
- Mechanics by paradigm: exceptions (checked/unchecked tradeoffs, exception safety), result/either types (errors in the type system force handling — the modern trend and why), error codes (C legacy, still the ABI reality); in all of them: never swallow, wrap with *context* (the chain should read as a story: "failed to render invoice → failed to load customer → connection refused"), and separate the operator-facing detail (logs) from the user-facing message (no internals — cross-link security).
- Robustness patterns: input validation at trust boundaries (cross-link security §3), timeouts on *everything* that crosses a process boundary (the most underrated reliability tool), retries with exponential backoff + jitter and an idempotency requirement, circuit breakers, graceful degradation (cached/stale/partial answers beat errors for many products — a product decision, so involve design).
- Resource safety: RAII/defer/finally/with — the guarantee that cleanup runs on *every* path; leaks on the error path as the classic review find.
- Crash-only thinking: if the system must survive kill -9 (it must), then clean shutdown is an optimization, and recovery paths are the real code path — test them.
- **Exercise themes**: review code where all bugs live on error paths; add robustness to a naive service client (timeout, retry, breaker, idempotency) and defend each parameter choice.

## 10. Language and paradigm literacy

- **Why**: languages are tools with design philosophies; elite engineers are polyglot enough to pick the right one and steal ideas across them. Deep in one or two, conversant in the paradigms.
- The paradigm map, taught through what each makes *easy* and *hard*: imperative/procedural (the machine's truth), OOP (encapsulated state + polymorphism; taught honestly — its wins are interface polymorphism and modeling, its failure mode is inheritance-lasagna and state-hiding-as-state-scattering; composition over inheritance), functional (immutability, pure functions, data transformation pipelines; its wins are reasoning and testability, its costs are learning curve and sometimes performance), declarative (SQL, config, UI frameworks — describe *what*, let the engine derive *how*; connects React and Terraform, same idea).
- Type systems as the axis that matters most in practice: static vs dynamic (tooling, refactorability, and team-scale as the real arguments — not personal comfort), strong typing patterns (make illegal states unrepresentable, parse-don't-validate, newtypes over primitives), gradual typing (TypeScript/Python typing as industry reality).
- Memory models as literacy: GC (throughput vs pause tradeoffs), reference counting (+ cycles), ownership/borrowing (Rust's contribution — compile-time memory safety and what it costs in flexibility); why this axis explains most "which language" decisions for systems work.
- Runtime literacy: compiled vs JIT vs interpreted, what the event loop is (JS single-thread model), what the GIL means (Python parallelism), why "slow language, fast system" happens (I/O-bound reality) and when it stops happening.
- **Exercise themes**: solve the same problem in two paradigms and compare; find the bug that a type system would have caught, then encode the invariant in types; justify a language choice for three described projects.

## 11. Version control fluency

- Git's model, not git's commands: content-addressed snapshots, refs as pointers, the three trees (working/index/HEAD); once the model lands, the commands become obvious and un-scary.
- History as communication: atomic commits (one logical change — reviewable, revertable, bisectable), commit messages that explain *why* (the diff shows what), when to squash vs preserve.
- Collaboration mechanics: branching strategies honestly compared (trunk-based with feature flags = the high-performing default per DORA research; GitFlow's niche: versioned/released software), rebase vs merge (rebase local/unshared for clean history, merge for shared — the golden rule of not rewriting published history), conflict resolution as comprehension (a conflict is two intents colliding — understand both before picking lines).
- Power tools worth fluency: interactive rebase (history editing before publish), bisect (§5), reflog (the undo of last resort — teach it and remove the fear), worktrees, blame-then-log-the-commit for archaeology (the *why* of old code lives in its commit and PR).
- **Exercise themes**: repair a described botched rebase using reflog; split a tangled diff into three atomic commits; find when and why a behavior changed using bisect + archaeology.

## 12. Dependencies and reuse judgment

- The build/buy/borrow decision: every dependency is a trust grant (cross-link security §13 supply chain) plus a maintenance contract (upgrades, breakage, abandonment risk). Left-pad as the canonical case study.
- Evaluation checklist: maintenance signal (release cadence, issue triage), bus factor, transitive weight (`npm ls` depth), license, API quality, the write-it-yourself cost *including maintenance* (the underestimated term).
- Vendoring, lockfiles, and reproducible builds; upgrade discipline (small frequent upgrades beat big scary ones — same expand/contract logic as §4); wrapping third-party APIs at the boundary so replacement stays possible (a thin adapter, not speculative abstraction).
- **Exercise themes**: evaluate two competing libraries for a described need and write the one-paragraph decision; design the boundary that would let a payment provider be swapped.

---

## Review checklists

**Reviewing others' code** — see `code-review-rubric.md` (the three-pass procedure lives there).

**Self-review before publishing a change** (train this as a reflex — the diff read-through catches 80% of review comments before they're made):
1. Read the full diff as if reviewing a stranger's PR — every line, including tests.
2. Does the change do one thing? Split if not.
3. Error paths: every new failure mode handled, resources cleaned up on all paths, timeouts present on new I/O.
4. Tests: does a test fail if the key behavior regresses? Did you run them?
5. Naming and comments: will the next reader (you, in a year) understand *why*?
6. Security pass: new inputs validated? secrets? injection surfaces? (security.md §17 quick list)
7. Performance pass: new queries (N+1?), new allocations in hot paths, new synchronous I/O on latency paths.
8. Commit message tells the why; PR description gives the reviewer context and testing evidence.

## Case study themes (L3+)

Teach judgment through real histories: the Knight Capital deployment failure (release engineering + feature-flag hygiene), the left-pad incident (dependency judgment), Therac-25 (race conditions + error-handling culture — the canonical "software kills" case), the Netscape rewrite (refactor vs rewrite), Google's monorepo/trunk-based development at scale, SQLite's testing discipline (aviation-grade testing as a choice). For each: forces → decision → outcome → what would you have done.
