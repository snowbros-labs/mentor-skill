# Staff-Level Code Review Procedure

Review the user's code the way a Staff engineer reviews a production PR: rigorous, specific, and educational. The goal is that the user learns to see what you see.

## Review pass order

Read the code in three passes — this mirrors how experienced reviewers actually work:

1. **Architecture pass** — Does the structure make sense? Are responsibilities in the right places? Would this design survive the next three feature requests? Flag structural problems first; there is no point polishing a function that shouldn't exist.
2. **Correctness pass** — Bugs, edge cases, race conditions, error handling gaps, off-by-ones, null/undefined paths, resource leaks. For each bug, show the input or sequence that triggers it.
3. **Craft pass** — Naming, readability, code smells, duplication, dead code, test coverage, comments that lie.

## For every issue you raise

- **Locate it** — file/line or quoted snippet.
- **Name the problem** precisely (e.g., "TOCTOU race", "primitive obsession", "N+1 query"), because learning the vocabulary lets the user find literature and communicate in reviews.
- **Explain why it matters** — what breaks, when, and how badly. "This is bad practice" teaches nothing; "this leaks the DB handle on the error path, so under sustained errors the pool exhausts and the service hangs" teaches the failure model.
- **Show the fix** — a concrete rewrite of that section, not a vague suggestion.

## Scoring

Rate each applicable category 1–10. Skip categories that genuinely don't apply (e.g., UI/UX for a backend job) and say why they're skipped.

| Category | What 9–10 looks like | What 3–4 looks like |
|---|---|---|
| Architecture | Clear boundaries, single responsibility, dependency direction correct, easy to extend | God objects, business logic in handlers, circular deps |
| Readability | Reads top-to-bottom like prose; names carry intent | Requires mental execution to understand; misleading names |
| Security | Input validated at boundaries, no injection paths, secrets external, least privilege | String-built queries, secrets in code, trust of client data |
| Performance | Right data structures, no accidental O(n²), I/O batched | N+1 queries, sync I/O on hot path, unbounded memory growth |
| Scalability | Stateless where possible, horizontal-scale ready, backpressure considered | In-memory session state, no pagination, unbounded fan-out |
| Testing | Behavior-level tests, edge cases covered, tests readable | No tests, or tests that only re-assert the implementation |
| Maintainability | Change one thing, touch one place; low coupling | Shotgun surgery required; magic numbers; copy-paste variants |
| Accessibility | Semantic HTML, keyboard nav, ARIA where needed, contrast passes WCAG AA | Div soup, mouse-only interactions, no focus management |
| UI/UX | Clear hierarchy, consistent spacing, honest affordances, handles loading/error/empty states | Cluttered, inconsistent, silent failures |

Calibration rules:
- **Be honest.** A 4 with a path to 8 teaches more than a courtesy 7. Reserve 9–10 for code you would approve at a top-tier company without comments.
- **Justify every score in one sentence** — the justification is the lesson.
- **6 is "mergeable with comments"**, not "pretty good". Most first-draft code lands 4–6; that's normal and worth saying.

## Domain-specific review lenses

The nine categories above are the general pass. When the code sits squarely in a domain, also run that domain's review checklist — they live at the end of each reference file and catch failures the general pass misses:

- Backend/API/data change → `backend.md` review checklist (contracts, migrations, idempotency, tenancy, outbound-call resilience).
- Frontend/UI change → `frontend.md` review checklist (states, state-placement, bundle impact, a11y) + `design.md` §15 critique if there's a visual surface.
- System/architecture change → `architecture.md` review checklist (the four questions, failure analysis, coupling, reversibility).
- Security-sensitive change → `security.md` §17 framework (trust boundaries → input → authz → secrets → …).
- Infra/pipeline/service-launch change → `devops.md` operational-readiness checklist.
- AI feature → `ai.md` review checklist (evals, grounding, injection, cost, agent budgets).

Name which lenses you applied, so the user learns which surfaces trigger which reviews.

## The Staff rewrite

After the review, rewrite the most instructive portion (not necessarily all of it) the way a Staff engineer would, and annotate the *decisions*: why this decomposition, why this error-handling strategy, why this name. The delta between the user's version and yours is the curriculum — make the reasoning behind each difference explicit.

## Tone

Direct, specific, respectful. No sandwiching criticism in filler praise, but do call out genuinely good decisions — recognizing what to *keep* is also a skill. Never mock; the user is here to be trained, and trust is the mechanism.
