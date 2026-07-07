# Mentor — Engineering Intelligence System

A [Claude Code](https://claude.com/claude-code) **skill**: a single orchestration layer over ten specialist engineering capabilities. It routes any task to only the expertise it needs — like a senior engineering manager assigning specialists — and runs in two modes:

- **TEACH** — trains you toward elite, Principal-grade judgment from first principles (full curriculum, exercises, review gates, a leveled roadmap).
- **BUILD** — acts as an autonomous Staff engineer: designs, implements, and reviews real work, composing only the required capabilities and running the relevant review gates.

It is deliberately **honest about its limits**: it does what a markdown skill can actually do (route by progressive disclosure, bound reading by complexity, run review gates, checkpoint state) and reuses the harness for the rest (memory, context continuation) instead of faking a token meter or a self-triggering engine.

## Install

Clone into (or copy the repo contents into) a `mentor` folder in your Claude Code skills directory:

```bash
# global (all projects)
git clone https://github.com/VED2107/mentor-skill.git ~/.claude/skills/mentor
# or project-local
git clone https://github.com/VED2107/mentor-skill.git .claude/skills/mentor
```

Then invoke it explicitly — it does not auto-trigger on ordinary questions:

```
/mentor teach me database indexing
/mentor review my authentication
use mentor to build a login page
```

## How it works

```
task ─► understand ─► classify complexity ─► route to capabilities ─► compose (load only needed refs)
                          (Tiny→Massive)         (minimum set)          progressive disclosure = token engine
     ─► produce ─► review (only implicated gates) ─► update docs (incremental) ─► checkpoint if needed
```

The orchestration engine — complexity tiers, routing method, token discipline, review-gate selection, the incremental-documentation workflow, and checkpoint/resume — lives in [`references/orchestration.md`](references/orchestration.md).

## Capabilities

Ten merged capabilities (all of design is one file, all of security is one, etc.), each a self-contained reference with first principles → practice → anti-patterns → cross-cutting reviews → checklist → exercises → case studies:

| Capability | Reference |
|---|---|
| Software Core (quality, testing, refactoring, debugging, concurrency, performance) | `references/software-engineering.md` |
| Backend & Data (APIs, data modeling, databases, async, reliability) | `references/backend.md` |
| Frontend & Web (platform, rendering, state, web perf, tooling) | `references/frontend.md` |
| Architecture (coupling, DDD, distributed systems, scaling, HA) | `references/architecture.md` |
| Security (threat modeling, vuln families, appsec→infra→detection) | `references/security.md` |
| Design (perception, craft, systems, 7-lens critique mode) | `references/design.md` |
| AI Engineering (RAG, agents, evals, serving, safety) | `references/ai.md` |
| DevOps & SRE (CI/CD, observability, SLOs, incidents, platform) | `references/devops.md` |
| Leadership & Career (writing, decisions, influence, interviewing) | `references/career.md` |
| Learning System (levels, portfolio ladder, assessment) | `references/roadmap.md` |
| Code-review rubric | `references/code-review-rubric.md` |
| Orchestration engine | `references/orchestration.md` |

## Routing examples

The router pulls only what the task needs:

- **"Build a login page"** → Design + Frontend + Security(authn). *Not* K8s, DB scaling, AI.
- **"Review my authentication"** → Security + Backend + Architecture.
- **"Optimize my dashboard"** → Frontend(perf) + Design(UX) + Backend(data).
- **"Add a RAG feature"** → AI + Backend + Security(injection/data-governance).

## Design principles

- **Merged, not fragmented** — one capability = one file. The registry is the router.
- **Route to the minimum** — thoroughness is correct routing, not maximal reading.
- **Progressive disclosure is the token engine** — unopened references cost nothing.
- **Reuse the harness** — memory (`MEMORY.md`) and codebase graphs (the `graphify` skill) are not reinvented.
- **Honest over impressive** — no fake token meter, no fake self-triggering; see `orchestration.md` §9.

## License

MIT — see [LICENSE](LICENSE).
