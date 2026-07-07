# Orchestration Engine

The runtime for both modes. Read at the start of any BUILD task and any non-trivial TEACH routing decision. This file is the "how" behind SKILL.md's workflow: complexity classification, capability routing, token discipline, review-gate selection, checkpoint/resume, incremental documentation, self-improvement, and failure recovery. It is honest about what a markdown skill can and cannot enforce — see §9.

## Table of contents

1. [The task loop](#1-the-task-loop)
2. [Complexity classification](#2-complexity-classification)
3. [Capability routing](#3-capability-routing)
4. [Token discipline](#4-token-discipline)
5. [Review-gate selection](#5-review-gate-selection)
6. [Incremental documentation workflow](#6-incremental-documentation-workflow)
7. [Checkpoint and resume](#7-checkpoint-and-resume)
8. [Self-improvement](#8-self-improvement)
9. [Honest limits and the harness division of labor](#9-honest-limits-and-the-harness-division-of-labor)
10. [Extension system](#10-extension-system)

---

## 1. The task loop

Every BUILD task, and every non-trivial TEACH task, runs this loop. Steps are skippable *by classification*, not by laziness — a Tiny task legitimately skips most of them; a Massive one runs all.

```
understand → classify → route → compose → produce → review → document → checkpoint?
     │          │         │        │          │         │          │          │
  restate    tier      bundles   load only  solution  gates     if code   if low
  in 1 line  T/S/M/L/X          needed refs  or lesson (only     changed   context
                                                        implicated)         remains
```

The discipline that makes it cheap: **decide what to read before reading it.** Classification (§2) and routing (§3) happen from the task text and a minimal directory glance — not by loading files. Files load only after routing says they're needed.

## 2. Complexity classification

Classify before doing anything, from the request text plus at most a directory listing. The tier sets analysis depth, how many files you read, and which loop steps run.

| Tier | Shape | Files read | Loop depth | Docs update |
|---|---|---|---|---|
| **Tiny** | typo, one-line fix, rename, single obvious edit | 1 | edit + verify; skip routing/review | none |
| **Small** | one function/component, contained bug, small addition | 1–3 | route to 1 capability, 1 review gate | changelog line if notable |
| **Medium** | a feature within one module, a focused refactor | 3–8 | 1–2 bundles, relevant gates | changelog + touched maps |
| **Large** | cross-module feature, new subsystem, migration | targeted, not exhaustive | 2–3 bundles, multiple gates, likely a design note first | maps + decisions + debt |
| **Massive** | new service, architecture change, security-wide change | breadth via graphify/index, not full reads | full loop, design doc/RFC before code, all implicated gates | full incremental pass + ADR |

Two rules that prevent the classic waste:
- **Escalate on evidence, don't pre-inflate.** Start at your honest estimate; bump up only when the work reveals hidden scope. A Medium that turns out to touch auth becomes Large *when you learn that*, not preemptively.
- **Never reload the whole project.** If one module changed, read that module. Use `graphify-out/` or an existing `docs/project-index.md` for structural questions instead of re-reading source. Breadth comes from the index; depth comes from targeted reads.

## 3. Capability routing

Routing = mapping a task to the minimum capability set (the registry lives in SKILL.md). Method:

1. **Name the deliverable** in concrete terms ("a login form", "a faster dashboard query", "a threat model for the upload flow").
2. **Ask the manager question**: which specialists would a good engineering manager staff on *exactly this*? That set is your route.
3. **Take the smallest set that covers the deliverable.** When a capability is a "maybe", leave it out — a first pass will surface the real need, and you can load it then. Loading-on-suspicion is the token leak.
4. **Note cross-links, don't auto-load them.** A caching lesson *mentions* security and DevOps; it doesn't load those files unless the task centers there.

Anti-patterns to refuse:
- Loading every reference "to be thorough" — thoroughness is correct routing, not maximal reading.
- Running a design critique on a pure-backend change, or a DB-scaling analysis on a static form.
- Re-loading a capability file already in context this session.

Worked routes (mirror SKILL.md examples; extend as needed): auth review → Security + Backend + Architecture; login page → Design + Frontend + Security(authn); dashboard optimize → Frontend(perf) + Design(UX) + Backend(data); RAG feature → AI + Backend + Security; "why is this slow" → Software Core(perf) + the layer that owns the hot path.

## 4. Token discipline

Progressive disclosure is the primary token engine — unopened reference files cost nothing, so correct routing *is* the optimization. On top of that:

- **Read ranges, not whole files** when you know the section (each capability file has a TOC).
- **Reuse in-context knowledge.** If a file or a piece of the codebase is already loaded this session, don't re-read it; reason from what you have.
- **Summarize once.** Don't restate the plan, the context, or prior findings repeatedly — say it where it's load-bearing, then reference it.
- **Answer at the altitude asked.** A Tiny/Small task gets a direct answer, not a lesson or a full review write-up. Depth scales with tier, not with a fixed template.
- **Prefer the index over the source** for "where/what" questions (graphify, project-index).

The measure of success: the tokens spent map to the task's real complexity. A one-line fix that triggered three file reads and a five-section review report is a routing failure, not diligence.

## 5. Review-gate selection

Run only the gates the change implicates (SKILL.md lists the mapping). The selection *itself* is a judgment to state: "This touches auth input, so I ran the security framework and the backend checklist; it has no UI or infra surface, so design and DevOps gates don't apply." That sentence proves the gate set was chosen, not skipped.

Severity discipline in review output: lead with correctness/security issues, then design/maintainability, then nits — and label nits as nits. Don't bury the load-bearing finding under formatting comments (see career.md §3).

## 6. Incremental documentation workflow

The living-docs system. **Merge, never regenerate; touch only what changed.** Runs at task end for Medium+ tasks that materially changed the codebase (Tiny/Small skip it or add a changelog line). This is a documented workflow the orchestrator *follows when it runs a task* — it does not self-trigger without a `settings.json` hook (§9).

**Canonical doc set** (create lazily — only the ones a project actually needs; don't scaffold empty files):

| File | Holds | Update trigger |
|---|---|---|
| `docs/project-index.md` | top-level map: what/where, entry points, the other docs | structure changes |
| `docs/architecture.md` | components, boundaries, data flow, key decisions' shape | architecture changes |
| `docs/component-map.md` | components/modules and their relationships | new/removed components |
| `docs/api-map.md` | public endpoints/contracts | API surface changes |
| `docs/database-map.md` | tables, relationships, migrations | schema changes |
| `docs/security-map.md` | trust boundaries, authn/authz model, sensitive data flows | security-relevant changes |
| `docs/design-system.md` | tokens, components, patterns | design-system changes |
| `docs/decisions.md` | ADR log (context→decision→consequences, append-only) | any non-obvious decision |
| `docs/technical-debt.md` | known debt, owner, cost, paydown note | debt taken or paid |
| `docs/changelog.md` | human-readable change log | every notable change |
| `docs/context.json` | machine-readable snapshot: files, modules, relationships (see below) | structural changes |

**Machine-readable snapshot** (`docs/context.json`) — keep small and diff-friendly; this is the JSON knowledge layer the spec asked for, scoped to what's maintainable by hand-merge rather than a fake full-graph regenerator:

```json
{
  "updated": "2026-07-07",
  "modules": [
    {"path": "src/auth", "role": "authentication", "public": ["login", "verifyToken"],
     "depends_on": ["src/db", "src/config"], "trust_boundary": true}
  ],
  "routes": [{"method": "POST", "path": "/login", "handler": "src/auth/login", "auth": false}],
  "decisions": ["adr-001-session-tokens"],
  "open": ["rate-limit login endpoint (debt)"]
}
```

**Merge procedure** (the "incremental patch" the spec wanted, done honestly):
1. Read only the doc sections the change affects (not the whole doc set).
2. Compute the delta: new/modified/removed files, changed relationships, new decisions, new debt/TODOs, new public surface.
3. Patch those sections in place; append to append-only logs (decisions, changelog); leave untouched sections byte-for-byte alone.
4. Prefer the existing `graphify` skill for any deep structural graph — `context.json` is the lightweight always-current index, graphify is the heavy analysis.

If a `graphify-out/` already exists, treat the user's question as a graphify query first (per the graphify skill) rather than duplicating its work here.

## 7. Checkpoint and resume

The honest version of "budget manager / continuation". A markdown skill can't read its remaining token count, but it *can* recognize the signs of a long task and persist state so any continuation — same session or a fresh context — resumes cleanly instead of restarting or degrading.

**When to checkpoint**: a Large/Massive task with visible remaining work when the conversation is getting long, or before any risky/irreversible step, or when the user signals a pause. Do not wait to be told.

**Checkpoint format** — write to `docs/checkpoint.md` (or the scratchpad if the repo shouldn't carry it):

```markdown
# Checkpoint — <task name> — <date>
## Goal
<one line>
## Done
- <completed steps, with the files touched>
## In progress
- <the exact step underway and where it stands>
## Remaining
- <ordered next steps>
## Decisions made
- <choices already locked, so they aren't relitigated>
## Continuation plan
<the first action to take on resume>
```

**Resume**: read the checkpoint, confirm the "in progress" item's actual state against the code (state can drift from the note), then continue from the continuation plan. Never restart completed work. Delete or supersede the checkpoint when the task finishes.

The rule the spec asked for, kept honestly: **do not produce a degraded answer to beat a context limit.** Checkpoint and hand off instead. The harness's own context summarization carries the conversation across the window; the checkpoint file makes that carry lossless for the *task*.

## 8. Self-improvement

Cheap, honest version of "extract reusable rules":
- When the same decision, pattern, or correction recurs across tasks in a project, write it once to the right durable place — a project convention into `docs/decisions.md` or `docs/context.md`, a user preference into the harness `MEMORY.md`, a reusable code pattern into the codebase itself (a helper, a template).
- The test for extraction: has this come up ≥3 times, and will encoding it save real future work or prevent a repeat mistake? (Rule of three, from software-engineering.md §1 — applied to process, not just code.)
- Do not invent speculative rules from a single instance; that's the over-abstraction anti-pattern wearing a process costume.

## 9. Honest limits and the harness division of labor

Stated so nothing in this system misleads the user or a future maintainer:

| Spec ask | Truth | This system's honest answer |
|---|---|---|
| Token budget metering | The model cannot reliably read its own remaining context | Complexity tiers bound reading *proactively*; no fake live meter |
| "Wait for next context window, continue exactly" | The skill can't pause execution; the harness auto-summarizes context | Checkpoint file (§7) makes resumption lossless; harness carries the thread |
| Auto-run at end of every task | Skills can't self-trigger; only `settings.json` hooks run automatically | Documented workflow (§6) the orchestrator runs; a `Stop` hook can be added to settings.json if true automation is wanted |
| Full codebase knowledge graph, always current | Hand-maintained full graphs rot; regenerating every task is the waste the spec warns against | Lightweight `context.json` index + the existing `graphify` skill for deep graphs |
| Cross-session memory | Already provided by the harness | Reuse `MEMORY.md`; don't build a parallel store |

The design principle: **build the parts that are real, reuse the parts that already exist, and refuse to fake the parts that can't be done in markdown.** A system that honestly does 80% beats one that claims 100% and silently fails the other 20%.

## 10. Extension system

Adding a capability or a project-specific rule set, without touching the core:

- **New capability**: add one `references/<name>.md` following the house shape (TOC → why-it-matters → first principles → practice → anti-patterns → cross-cutting reviews → checklist → exercises → case studies), then add one row to the SKILL.md registry table and, if it introduces a review, one row to the gate mapping. That's the whole contract — the router picks it up by the registry.
- **Project overrides**: a repo can carry `docs/mentor.md` with project-specific conventions, preferred stack, and routing hints; the orchestrator reads it (if present) during Understand and lets it override defaults. This keeps the shared skill generic and the project specifics local.
- **Keep capabilities merged, not fragmented**: a new sub-topic of an existing capability goes *into* that capability's file as a section, not into a new file — the whole point of the registry is one file per capability.
