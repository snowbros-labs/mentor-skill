# Learning Roadmap & Progression System

The map that turns individual lessons into an education. Read this when starting a mentoring relationship, when the user asks "what next" or "where am I", and when placing a finished lesson in context.

## The five levels

Levels describe *judgment*, not years. A user can be Senior in backend and Foundations in design — track per-discipline positions, not one global rank.

| Level | Defining capability | Failure mode that caps people here |
|---|---|---|
| **L1 Foundations** | Can build working things by following patterns; understands what the code does | Copies without understanding why; can't debug off the happy path |
| **L2 Practitioner** | Ships features independently; writes tests; understands the why of local decisions | Local optimum thinking — good functions, poor systems; accepts requirements as given |
| **L3 Senior** | Owns systems end-to-end; designs for failure; reviews others' work well; predicts second-order effects | Depth without breadth; can't operate across discipline boundaries |
| **L4 Staff** | Designs across systems and teams; makes tradeoffs explicit; multiplies others; picks the right problem, not just the right solution | Technical excellence without communication/influence stalls here |
| **L5 Principal/Elite** | Sets technical direction; sees around corners; judgment trusted on problems with no precedent; creates leverage through standards, platforms, and people | — |

Level assessment: don't ask, *observe*. Give a probe task (small design question, code sample to critique) and place the user by the quality of their reasoning — what they notice, what they ask, what tradeoffs they name unprompted. Reassess implicitly every few lessons.

## The six tracks

1. **Software Engineering Core** (`software-engineering.md`) — the trunk; everything else branches from it
2. **Backend & Data** (`backend.md`) → deepens into **Architecture & Distributed Systems** (`architecture.md`)
3. **Frontend & Web** (`frontend.md`) → connects to **Design** (`design.md`)
4. **Security** (`security.md`) — starts as a lens on other tracks, becomes its own depth
5. **AI Engineering** (`ai.md`) — requires Core L2+ first; systems discipline around a probabilistic component
6. **Operations & Delivery** (`devops.md`) — how software actually reaches and survives production
7. **Leadership & Communication** (`career.md`) — the L3→L4 gate is mostly this track

Cross-links matter more than track purity: schedule "connective" lessons regularly (e.g., "how caching decisions touch security, UX, and cost simultaneously"). Elite judgment lives at the intersections.

## Suggested sequencing by level

**L1 → L2**: Core (code quality, testing basics, debugging methodology) + one platform track (frontend or backend) to depth + security as a lens (trust boundaries, injection family) + Git/CI fluency (devops basics). Portfolio: ship one complete, tested, deployed application.

**L2 → L3**: Core (refactoring, concurrency, performance engineering) + second platform track to competence + architecture fundamentals (coupling, SOLID, data modeling) + full security curriculum §1–10 + observability + design fundamentals (§1–5) + technical writing basics. Portfolio: a system that survives load, failure injection, and a security review; a written design doc for it.

**L3 → L4**: Architecture at depth (distributed systems, event-driven, scaling case studies) + security architecture + design critique competence + AI engineering track + incident command + RFC writing, decision records, influencing without authority + interviewing (both sides of the table). Portfolio: a multi-service system with documented tradeoffs, an eval-driven AI feature, a postmortem, and an RFC that others could execute.

**L4 → L5**: mostly synthesis and judgment — case studies of real failures and real systems, "what would you have done" analyses, teaching others (the user should be able to give the lessons of this curriculum), setting standards, platform thinking.

## Portfolio project ladder

Projects are the assessment instrument — knowledge that hasn't shipped is unverified. Each project targets one level gate and gets reviewed with full critique modes (code review + design critique + architecture review + security review as applicable).

1. **L1 gate**: CLI tool or small full-stack app — tested, documented, deployed. Reviews: code review.
2. **L2 gate**: production-shaped web app — auth, persistence, CI/CD, error handling, accessibility pass, basic observability. Reviews: code + security + design.
3. **L3 gate**: system under stress — realistic load, caching layer, background jobs, graceful degradation, dashboards/alerts, load-test report, threat model document. Reviews: all four.
4. **L3–L4 gate (AI)**: eval-driven AI feature — eval set written *before* implementation, injection-resistant tool use, cost/latency budget, regression tracking.
5. **L4 gate**: multi-component platform with a written RFC, ADRs, runbooks, an induced-failure postmortem, and a "what I'd do differently" retrospective — judged on the *documents* as much as the code.

## Assessment gates

Before declaring a level transition, require all of:
- **Build**: the gate project passes its reviews at "mergeable" scores (mostly 6+, nothing below 4 unaddressed).
- **Critique**: the user reviews a flawed artifact you construct (code with planted bugs across severity levels, or a design with planted violations) and finds the critical issues unprompted. The ability to *see* is the level.
- **Explain**: the user teaches one topic from the level back to you; probe with follow-ups. Feynman test — gaps hide in the explanations.
- **Decide**: a scenario with no clean answer ("your team wants microservices, you have four engineers, deadline in three months") — judged on tradeoff articulation, not on picking your preferred answer.

## Learning mechanics (how to teach, not just what)

Grounded in what actually works in skill acquisition:
- **Retrieval over re-reading** — quiz earlier material at spaced intervals (next lesson, next week, next month). Getting it wrong then corrected beats getting it told twice.
- **Interleaving** — mix domains within a study arc; blocked practice feels better and transfers worse.
- **Desirable difficulty** — pitch exercises slightly above comfort; struggle followed by resolution builds durable models. If the user cruises, escalate.
- **Worked examples → faded scaffolding** — full worked example first, then partial ("you do the error handling"), then independent. Fade faster for stronger users.
- **Error-driven lessons** — planted-flaw exercises ("find what's wrong") build the reviewing eye faster than construction exercises alone.
- **Case studies at L3+** — real-world failures (outages, breaches, redesigns) as analysis material: what forces, what decision, what alternative, what would you have done. Judgment is trained on cases, not principles alone.
- **Projects over toy reps** — one integrative project outweighs twenty isolated katas past L1.

## Session cadence

Each session: (1) retrieval check on prior material if any is due, (2) review pending exercise attempts before new content, (3) the lesson, (4) exercise assignment, (5) map placement — covered / connects-to / next. Keep a running sense of the user's per-track level and pending loops; state it when asked "where am I".
