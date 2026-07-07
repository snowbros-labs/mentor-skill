# DevOps, SRE & Platform Engineering Curriculum

How software reaches production and survives there. Prerequisites: software-engineering.md core; pairs with backend.md and architecture.md. The unifying mental model: **production is the only environment that matters, and everything between a commit and a healthy production system is an engineering discipline of its own** — deploy pipelines, infrastructure, observability, and incident response are products with users (your own engineers), not chores.

## Table of contents

1. [The delivery mindset](#1-the-delivery-mindset)
2. [CI/CD](#2-cicd)
3. [Release engineering](#3-release-engineering)
4. [Containers and orchestration](#4-containers-and-orchestration)
5. [Infrastructure as code](#5-infrastructure-as-code)
6. [Cloud engineering](#6-cloud-engineering)
7. [Observability](#7-observability)
8. [SRE: SLOs, error budgets, toil](#8-sre-slos-error-budgets-toil)
9. [Incident response](#9-incident-response)
10. [Postmortems and learning from failure](#10-postmortems-and-learning-from-failure)
11. [Platform engineering and developer experience](#11-platform-engineering-and-developer-experience)
12. [Cost engineering](#12-cost-engineering)

---

## 1. The delivery mindset

- DevOps as the dissolution of a boundary, not a job title: "you build it, you run it" — the feedback loop from production back to design is the entire point; walls between dev and ops recreate the latency that causes bad software.
- The DORA research as the evidence base: four metrics (deployment frequency, lead time, change-failure rate, time-to-restore) — and the counterintuitive core finding worth internalizing: **speed and stability correlate**; teams that ship small and often break less, because small changes are diagnosable and revertable. "Slow down to be safe" is usually backwards; batch size is the enemy.
- Everything as code as the umbrella principle: app, infra, pipelines, dashboards, alerts, runbooks — reviewed, versioned, reproducible. Snowflakes (hand-configured anything) are outages waiting for their moment.
- **Exercise themes**: given a team's release ritual (monthly, manual QA gate, change advisory board), predict its DORA profile and propose the first three changes.

## 2. CI/CD

- Continuous integration properly: trunk-based development with short-lived branches (the DORA-backed default; long-lived branches are integration debt accruing interest), every commit builds and tests, the build stays green as a social contract (a red main is a stop-the-line event).
- Pipeline design as latency engineering: fast feedback first (lint/typecheck/unit in minutes), expensive stages later (integration, E2E), parallelize by default, cache aggressively (dependency and build caches), flake quarantine with owners (software-engineering §3 — a flaky gate trains people to ignore gates).
- Continuous *delivery* vs *deployment*: every commit deployable vs every commit deployed; the artifact discipline that enables both — build once, promote the same artifact through environments (rebuilding per environment reintroduces "works in staging" mystery).
- Pipeline security (cross-link security §13): least-privilege CI tokens, no secrets in logs, artifact signing/provenance, protected branches and required reviews as the control plane.
- **Exercise themes**: redesign a described 55-minute serial pipeline to under 15 (identify what parallelizes, what caches, what moves post-merge); write the branch-protection and promotion policy for a described team.

## 3. Release engineering

The discipline that makes deploys boring — separated from CI/CD because *releasing* is a product/risk activity, not just pipeline plumbing:

- **Deploy ≠ release**: deploying code and exposing behavior are different events, and separating them (feature flags) is the single most powerful release-risk tool. Flag lifecycles (backend.md §7 — retire or rot), kill switches for new risky paths as standard equipment.
- Progressive delivery ladder: rolling deploys (the baseline; needs graceful shutdown and readiness from backend.md §8) → blue/green (instant cutover and rollback at 2x cost) → canary (expose a slice, compare metrics, promote or abort — the grown-up default at scale; automated canary analysis as its mature form) → percentage/cohort rollouts via flags.
- Rollback as a first-class requirement: rollback must be *fast, tested, and data-compatible* — which is why expand/contract schema migration (backend.md §3) is a release-engineering concern; the deploy that can't roll back because the migration already dropped the column is a self-built trap. Roll-forward as the honest alternative when state has moved on — but as a decision, not a default forced by neglect.
- Versioning and compatibility: semver as communication, N-1 compatibility during rollouts (old and new run *simultaneously* during every rolling deploy — the mid-deploy mixed-fleet is a real state to design for), client/server version skew for mobile (you don't control the client's upgrade — an API shipped to an app store is a decade-long contract).
- Knight Capital as the canonical case: dead code + partial deploy + reused flag = $460M in 45 minutes; the lessons are deploy verification, config hygiene, and kill switches — teach it in full.
- **Exercise themes**: design the canary rollout (metrics, thresholds, abort criteria, bake time) for a described risky change; find the rollback trap in a described deploy plan.

## 4. Containers and orchestration

- Containers demystified: processes with isolated namespaces and resource limits, not VMs; images as layered immutable filesystems — the reproducibility win ("works on my machine" becomes "ships my machine"); image discipline (small bases, multi-stage builds, non-root, pinned digests — cross-link security §12).
- Kubernetes at working depth, taught as its core loop: **desired state + reconciliation** — you declare, controllers converge; everything else (Deployments, Services, HPA) is instances of that one idea. The practical set: pods/deployments/services/ingress, requests and limits (and what OOMKill and CPU throttling actually do), liveness vs readiness (backend.md §8 — the distinction that prevents self-inflicted outages), HPA basics, namespaces and RBAC (security §12).
- Kubernetes honestly: it solves fleet problems (bin-packing, self-healing, rolling deploys, service discovery) at real complexity cost — the "do you need it" decision framework (PaaS/serverless/managed containers cover most products for years; adopt K8s when fleet size, multi-team isolation, or portability requirements *arrive*, not because the industry did).
- Service mesh at awareness level: mTLS, retries, traffic splitting as infrastructure (security §11) — and the same "complexity budget" honesty.
- **Exercise themes**: right-size requests/limits from provided metrics and explain each failure mode of getting them wrong; decide PaaS vs K8s for three described companies and defend it.

## 5. Infrastructure as code

- Declarative + idempotent as the core property: describe end state, let the tool diff and converge (same mental model as Kubernetes and React — teach the rhyme; it's the industry's convergent answer to state management).
- The Terraform-class workflow: plan as the review artifact (the plan *is* the PR body), state as the sacred-and-dangerous part (locking, remote backends, never hand-edited, drift detection), modules for reuse with the same API-design judgment as code (software-engineering §2).
- Environments without copy-paste: same modules + different variables; ephemeral preview environments as the maturity marker; the staging-parity problem stated honestly (staging never equals prod — invest accordingly in progressive delivery §3 and observability §7 rather than staging perfectionism).
- GitOps: the repo as desired state, an operator converges the cluster, drift self-heals — audit trail and rollback-by-revert for free.
- Immutable infrastructure: replace, don't patch; servers as cattle; configuration drift as the disease immutability cures.
- **Exercise themes**: review a provided Terraform plan and catch the destructive change hiding in it; design the environment strategy (and its variables matrix) for a described product.

## 6. Cloud engineering

- The service-model ladder and its responsibility line: IaaS → managed services → serverless; shared-responsibility (security §12) at each rung; the build-vs-managed default — **managed unless you have a reason** (your engineers' time is the scarcest resource; undifferentiated heavy lifting is rented, not built).
- Core primitives literacy (provider-agnostic concepts): compute (VMs/containers/functions and their cold-start/duration/cost envelopes), networking (VPCs, subnets, security groups, load balancers, DNS — enough to draw and secure the network diagram, security §11–12), storage tiers (object/block/file + lifecycle policies), managed databases (and what "managed" does and doesn't cover — it patches; it doesn't design your indexes).
- Serverless honestly: what it buys (zero idle cost, scale-to-zero, no fleet ops) and its bill (cold starts, duration limits, vendor coupling, distributed-debugging difficulty, cost surprises at sustained load — the crossover math where always-on becomes cheaper).
- Multi-cloud skepticism as the senior position: portability abstractions cost you each cloud's best services and are usually insurance nobody claims; multi-*region* within one cloud is the availability play that actually gets exercised (architecture §12).
- **Exercise themes**: place a described workload on the service-model ladder with cost/ops reasoning; draw and secure the VPC layout for a three-tier app.

## 7. Observability

- The discipline of asking production questions — monitoring tells you *known* things broke; observability lets you debug *unknown* things (high-cardinality queryable telemetry). Debugging methodology (software-engineering §5) applied to systems you can't attach a debugger to.
- The three signals and what question each answers: **logs** (what happened, in detail — structured/JSON always, leveled, sampled at volume, secrets-and-PII scrubbed per security §14), **metrics** (how much/how fast, cheap at scale, pre-aggregated — but low-cardinality by nature: metrics find *that* something's wrong), **traces** (where time went across services — traces find *where*; the waterfall view that makes distributed latency debuggable). Correlation IDs propagated everywhere as the thread that stitches all three (backend.md checklist item made real).
- Instrumentation method: RED per service (rate, errors, duration) as the default dashboard, USE per resource (utilization, saturation, errors) for capacity, the four golden signals as the SRE phrasing; percentiles not averages (p99 — software-engineering §8); OpenTelemetry as the vendor-neutral instrumentation layer worth standardizing on.
- Alerting as a design discipline: **alert on symptoms (user pain / SLO burn), not causes** (a CPU alert nobody acts on is noise; a latency-SLO burn alert is a real event); every page has a runbook link and an action; alert fatigue as the real killer (each false page trains responders to ignore the next one — the alert review is part of the system); burn-rate alerting on error budgets (§8) as the mature pattern.
- Dashboards with a point of view: overview → drill-down hierarchy, deploy markers on everything (the first debugging question is "what changed"), dashboards-as-code.
- **Exercise themes**: instrument a described three-service flow — what's logged/measured/traced at each hop, and which signal answers which of five given production questions; triage a provided alert inventory into keep/rewrite/delete with reasons.

## 8. SRE: SLOs, error budgets, toil

- SLI → SLO → error budget as the reliability control system: SLI (a measured indicator: availability, latency-under-threshold), SLO (the target the *product* actually needs — architecture §12's "nines are a product decision," operationalized), error budget (1 − SLO = the failure you're *allowed*; spend it on velocity, freeze features when it burns dry). The genius is turning "reliability vs speed" from a values fight into arithmetic both sides can read.
- SLO craft: measure at the user (a healthy server behind a broken LB is an outage; synthetic + real-user measurement), latency SLOs as percentile-under-threshold, per-journey SLOs over per-service vanity (checkout matters; the admin panel less — different targets), the 100%-is-wrong lesson (each nine costs 10x; users can't distinguish 99.99% from 99.999% but your on-call rotation can).
- Toil: manual, repetitive, automatable, scaling-with-load work — cap it (Google's 50% rule as the reference point), budget its automation, treat a growing toil ratio as an architecture smell feeding back into design.
- On-call as a designed human system: sustainable rotations, escalation paths, handoffs, page-quality review; compensation and the cultural marker — how a company treats on-call predicts its production quality.
- **Exercise themes**: define SLIs/SLOs for three user journeys of a described product, with measurement points and burn-rate alert thresholds; identify the toil in a described team's week and rank the automation ROI.

## 9. Incident response

Auto-clarity note: this section trains a *practice* — during real incidents, communication is explicit and calm, never clever.

- Incident command as a role system (borrowed from emergency services because it works): Incident Commander (coordinates, decides, does *not* debug — the IC who dives into logs abandons the incident), operations/subject experts (hands on keyboards), communications lead (stakeholder updates on a cadence, so responders aren't interrupted by "any update?"), scribe (timeline as it happens — memory is reconstructed, logs are real).
- Severity taxonomy with pre-agreed definitions and response expectations per level — the classification argument happens once, in a doc, not at 3am.
- **Mitigate first, understand later**: the incident goal is restoring service, not finding root cause — rollback/failover/flag-off/shed-load are honorable first moves; the investigation happens after, on the preserved evidence (forensics instinct from security §14). "What changed?" as the opening question (deploys, configs, dependencies — hence deploy markers, §7).
- Runbooks: per-alert, action-oriented, tested (an untested runbook is a rumor — same rule as failover, architecture §12); game days to exercise both systems and humans.
- Communication under fire: status-page honesty (users forgive outages; they don't forgive silence or spin), internal cadence updates, the handoff protocol for long incidents.
- **Exercise themes**: run a tabletop incident (you narrate unfolding symptoms, the user plays IC — graded on process: mitigation-first, role discipline, comms cadence); write the runbook for a described alert.

## 10. Postmortems and learning from failure

- Blameless as an epistemological stance, not politeness: humans are the adaptive element in a system that failed them; "human error" is where investigation *starts*, not where it ends (why did the system make that error easy, invisible, and high-consequence?). Blame produces hiding; hiding produces repeat incidents.
- Postmortem craft: timeline from the scribe's record, contributing factors over singular "root cause" (complex failures are conjunctions — the Swiss-cheese model; "the deploy AND the missing alert AND the stale runbook"), the five whys used carefully (it linearizes; use it per-factor, not as the whole analysis), impact quantified honestly.
- Action items with teeth: owned, deadlined, tracked like features, and *reviewed for completion* — the postmortem whose actions rot teaches the org that postmortems are theater; prefer few structural fixes over many aspirational ones ("add an alert" beats "be more careful" every time; "remove the failure mode" beats both).
- The learning loop back to design: incident patterns feed architecture reviews (recurring timeout incidents → the resilience audit from backend.md §8), near-misses count (the incident that almost happened is free tuition), sharing postmortems widely as cultural infrastructure.
- **Exercise themes**: write the blameless postmortem from a provided incident timeline (graded on contributing-factor analysis and action-item quality); find the blame-shaped sentences in a provided postmortem draft and rewrite them.

## 11. Platform engineering and developer experience

- The platform-as-product stance: internal platforms (deploy tooling, golden paths, service scaffolding) have users (engineers), UX (design.md thinking applied to CLIs and dashboards!), adoption metrics, and roadmaps; mandated platforms nobody likes get worked around, and the workaround becomes the real (unsupported) platform.
- Golden paths over gates: make the right thing the easy thing (a `create-service` scaffold with observability, CI, security defaults baked in beats a 40-item compliance checklist); guardrails over approvals (policy-as-code that fails fast in CI beats a human review queue — same latency logic as everything else in this file).
- Developer experience as measurable engineering: build times, local-dev fidelity, time-to-first-PR for new hires, deployment lead time — the inner loop (edit-test-debug seconds) compounds into org velocity more than almost any architecture decision; frontend.md §10 is this discipline applied to one stack.
- Cognitive-load budgeting as the platform's real product: every technology a product team must operate is a tax; the platform's job is absorbing complexity so product teams spend their complexity budget on the product (Team Topologies' platform-team framing at awareness level; Conway's law from architecture §1 again).
- **Exercise themes**: design the golden path for "new microservice" at a described company — what's scaffolded, what's enforced, what's optional; pick the three DX metrics you'd instrument first for a described 50-engineer org and say what decision each would drive.

## 12. Cost engineering

- Cost as a first-class quality attribute (architecture §1's list includes it; here's the practice): cloud bills are usage-shaped, so architecture decisions *are* spending decisions — the engineer who can read a bill influences design with evidence.
- The waste taxonomy where money actually hides: idle capacity (dev environments running weekends, over-provisioned instances — rightsizing from real utilization), egress and cross-AZ traffic (the invisible line item that punishes chatty architectures), storage lifecycle neglect (logs and snapshots kept forever by default), over-eager redundancy (multi-region for the admin panel — architecture §12's product-decision logic applied to spend).
- Unit economics as the mature frame: cost per request/user/tenant (multi-tenancy from backend.md §7 meets the bill — the noisy neighbor is also the expensive neighbor); tagging/allocation so cost has owners; cost regression alerts like performance budgets (frontend.md §8's discipline, applied to the bill).
- The levers ranked: architectural (caching, batching, async — software-engineering §8's universal levers save money too, same physics), commitment discounts (reserved/spot with their availability tradeoffs), and the honest one — deleting things (the unused service still bills).
- **Exercise themes**: given a described bill breakdown, find the three biggest savings without availability loss; compute the unit cost per tenant and identify the subsidized whale.

---

## Review checklist (operational readiness — run before a service ships)

1. Deploy: pipeline green-gated, artifact promoted not rebuilt, progressive rollout plan, rollback tested and data-compatible (expand/contract verified).
2. Config: flags for risky paths with kill switches, secrets from a vault not env-committed, config change = audited change.
3. Observability: RED dashboard, deploy markers, correlation IDs propagate, logs structured and scrubbed, traces on the critical path.
4. Alerts: symptom-based, SLO-linked, each with a runbook and an owner; page-worthy vs ticket-worthy separated.
5. SLOs: defined per user journey, measured at the user, error budget policy agreed with product.
6. Capacity: load-tested at projected peak with headroom, autoscaling verified (including scale-*down*), limits/quotas set.
7. Failure: dependency timeout/retry/breaker audit (backend.md §8), degraded modes designed, one game-day scenario run.
8. Incident readiness: severity matrix applies, on-call rotation staffed, runbooks exist, escalation paths named.
9. Security: CI/CD least-privilege, images scanned and pinned, infra security review (security §12–13).
10. Cost: tagged for allocation, unit-economics estimate, top cost driver identified with its owner.

## Case study themes (L3+)

Knight Capital (the full release-engineering syllabus in one incident); AWS S3 2017 (the typo, the blast radius, the status page that depended on S3); Cloudflare's 2019 regex CPU exhaustion (global config blast radius + the rollback that worked); GitHub October 2018 (split-brain, the 24-hour recovery, publishing the postmortem); Google's SRE book chapters as standing references (SLOs, toil, incident management); the Rogers 2022 or Facebook 2021 BGP outages (the network layer nobody thinks about until DNS doesn't resolve). For each: forces → decision → outcome → what would you have done.
