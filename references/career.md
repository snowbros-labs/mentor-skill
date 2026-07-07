# Leadership, Communication & Career Curriculum

The track that gates L3→L4→L5 — and the one strong engineers most often neglect until it caps them. The blunt truth: past Senior, technical skill is table stakes; **impact is delivered through other people, in writing, and via judgment about which problems matter.** This is engineering, not "soft skills" — it has principles, failure modes, and deliberate practice like any other track.

## Table of contents

1. [Technical communication](#1-technical-communication)
2. [Technical writing: docs, RFCs, ADRs](#2-technical-writing-docs-rfcs-adrs)
3. [Code review as a craft](#3-code-review-as-a-craft)
4. [Decision-making under uncertainty](#4-decision-making-under-uncertainty)
5. [Influence without authority](#5-influence-without-authority)
6. [Estimation, planning, and scoping](#6-estimation-planning-and-scoping)
7. [Mentorship and multiplying others](#7-mentorship-and-multiplying-others)
8. [The engineering career ladder](#8-the-engineering-career-ladder)
9. [Interviewing: both sides of the table](#9-interviewing-both-sides-of-the-table)
10. [Working with the business](#10-working-with-the-business)

---

## 1. Technical communication

- The core skill: **adjust altitude to the audience.** The same system explained to a junior engineer, a Staff peer, a PM, and a VP is four different messages — same truth, different resolution and vocabulary. The most common senior-communication failure is one-altitude-fits-all (drowning executives in detail, starving engineers of it).
- Lead with the conclusion (BLUF — bottom line up front): executives and busy readers want the answer first, support second; the mystery-novel structure (build-up to reveal) wastes the reader's most-attentive moment. This mirrors the harness's own "lead with the outcome" rule — it's universal.
- Communicating uncertainty honestly: confidence levels, what you know vs infer vs guess, the range not the false-precise point — credibility compounds and one confident-wrong claim spends a lot of it.
- Written vs synchronous judgment: default to writing for anything consequential or multi-person (it scales, persists, and forces rigor); use synchronous for high-bandwidth ambiguity resolution and relationship work; the meeting that should have been a doc, and the doc that should have been a five-minute call, are both failures worth naming.
- Explaining the hard thing simply: analogy, the concrete example before the abstraction, progressive disclosure (design.md §2 applied to prose), and knowing your curse-of-knowledge blind spots.
- **Exercise themes**: write the same incident explanation for four audiences; rewrite a buried-lede status update BLUF-first; take a jargon-dense paragraph and make it land for a PM without dumbing it down.

## 2. Technical writing: docs, RFCs, ADRs

Writing is a Staff engineer's primary tool — it's how ideas scale beyond the room and how thinking gets rigorous (fuzzy writing is fuzzy thinking made visible):

- The design doc / RFC as the central artifact: problem statement and *why now*, goals and explicit non-goals (the non-goals prevent half the review churn), proposed approach, **alternatives seriously considered** (a doc with no honest alternatives reads as a decision seeking rubber-stamp — reviewers can't engage; the alternatives are where the thinking shows), tradeoffs, risks and failure modes, rollout and rollback, open questions. The doc's job is to *enable good disagreement* before code is written, when it's cheap.
- ADRs (architecture.md §13): context → decision → consequences, immutable, superseded not edited — the durable "why" for the engineer three years out. Small and consistent beats elaborate and abandoned.
- Documentation as a product with users and a maintenance cost: the Diátaxis frame worth teaching — tutorials (learning-oriented), how-tos (task-oriented), reference (information-oriented), explanation (understanding-oriented) are *different documents with different jobs*, and the classic doc failure is blending them into one that serves none. READMEs, runbooks (devops §9), and the "why" comments from software-engineering §1 all live here. Docs rot — treat staleness as a bug, and prefer docs that live next to code.
- Writing that gets read: skimmable structure (headers, lists, the one bolded sentence per section), diagrams at the right altitude (C4, architecture §13), examples over prose for anything concrete.
- **Exercise themes**: write a real RFC for a system from an earlier lesson (graded on the non-goals and the alternatives sections — where docs live or die); classify a set of doc snippets by Diátaxis type and fix the blended one; write the runbook and the ADR for the same decision.

## 3. Code review as a craft

The reviewer's side (the author's rubric is code-review-rubric.md) — code review is the highest-frequency way engineers shape each other's work and culture:

- The purposes, ranked and sometimes in tension: catch defects, yes — but equally, spread knowledge, maintain consistency, and teach. A review that only gates and never teaches wastes the moment.
- What to actually review (and what to let go): correctness, design, and clarity are worth comments; style is the formatter's job (argue it once in config, never in a PR — software-engineering §1). The senior discipline is **distinguishing "this is wrong" from "this isn't how I'd do it"** and mostly staying silent on the latter — nitpick-flooding buries the load-bearing comment and trains authors to dread review.
- How to comment: ask questions over issuing decrees ("what happens if this is null?" teaches; "add a null check" doesn't), explain the *why* (rubric principle — a reason travels to the next PR; a demand doesn't), praise genuinely good decisions (recognizing what to keep is a skill and it makes the criticism land), and distinguish blocking from non-blocking explicitly (the "nit:" / "optional:" convention).
- Review latency as a team-health metric: a PR sitting a day blocks a person and grows merge conflicts — fast, frequent, small reviews beat thorough-but-slow (the DORA batch-size lesson, devops §1, applied to review); large PRs get rubber-stamped because nobody can hold 800 lines in their head, so the reviewer's leverage starts by pushing for smaller PRs.
- Receiving review well (the author's craft): assume good faith, separate ego from code, disagree with reasons not defensiveness, and thank the reviewer who found the bug — the culture is co-created.
- **Exercise themes**: review a provided PR and mark each comment blocking/non-blocking/praise, then cut your own nitpicks by half; rewrite three decree-comments as teaching-questions; take a defensive PR-reply thread and rewrite the author's side.

## 4. Decision-making under uncertainty

- The reversibility frame (Bezos's one-way vs two-way doors): reversible decisions should be made fast and low-ceremony by whoever's closest; irreversible ones deserve rigor, review, and seniority. The two failure modes are symmetric and both common — bureaucratizing cheap reversible calls, and YOLO-ing expensive irreversible ones. Naming which kind a decision is *is* half the skill.
- Deciding with incomplete information (the normal case — perfect information means you decided too late): the cost of delay vs the cost of being wrong, the last responsible moment (architecture §13 — decide when information peaks, not before and not after), reversible experiments to buy information (spikes, prototypes, canaries — devops §3).
- Making tradeoffs explicit and legible: name the axes (speed/quality/cost/risk/scope — you can't max all), state what you're optimizing *for* and what you're sacrificing, use a decision matrix when the options are many and the stakes justify it. The Staff move is surfacing the tradeoff for the group to own, not hiding it inside a technical choice nobody else can see.
- Cognitive-bias hygiene in engineering decisions: sunk cost (the rewrite you're too deep to abandon), confirmation bias (software-engineering §5's debugging lesson generalizes to all decisions), anchoring, recency (the last outage over-weights the next design), bikeshedding (Parkinson's law of triviality — the group spends the meeting on the color of the button and rubber-stamps the database choice). Disagree-and-commit as the norm that keeps decisions from either stalling or fracturing the team.
- **Exercise themes**: classify a set of decisions as one-way/two-way doors and assign each the right process; make a real recommendation from a deliberately-incomplete scenario and state your confidence and what would change your mind; spot the bias driving a described bad decision.

## 5. Influence without authority

The defining L4 skill: Staff+ engineers rarely have direct reports but must move work across teams they don't control.

- Influence is trust plus a well-made case, compounded over time: a track record of being right *and honest about when you were wrong* is the currency; you spend credibility on the calls that matter and don't fritter it on every disagreement.
- Meeting people where they are: engineers move on technical merit, PMs on user/business outcome, leadership on strategic impact and risk — the *same proposal* must be translated into each frame (the §1 altitude skill, applied to persuasion). "It's more elegant" moves no one who isn't an engineer.
- Building alignment: socialize ideas before the big meeting (no surprises — pre-wire the stakeholders one-on-one so the meeting ratifies rather than debates), write it down so it can be engaged with async (§2), find and enlist the skeptics early (a converted skeptic is your strongest advocate; an ambushed one is a lasting blocker).
- Driving change across teams: start with a lighthouse/proof rather than a mandate (show it working somewhere small — adoption pulls harder than policy pushes; the golden-path lesson from devops §11), make the right thing the easy thing, and accept that durable change is slow and relationship-bound.
- Disagreeing with leadership constructively: bring data and alternatives not just objections, disagree in private and commit in public, and know the difference between a hill worth dying on and a preference.
- **Exercise themes**: translate one technical proposal into engineer/PM/exec pitches; plan the socialization path for a cross-team change (who you pre-wire, in what order, with what framing); write the constructive-dissent message to a leader for a decision you think is wrong.

## 6. Estimation, planning, and scoping

- Why estimates are hard and always optimistic: the planning fallacy is structural, not a personal failing — teach reference-class forecasting (what did *similar* past work actually take, not what does this feel like) as the corrective, plus estimating in ranges/confidence-intervals rather than false-precise dates.
- Decomposition as the estimation tool: unknowns hide in big chunks; breaking work down surfaces them (and the act of decomposing *is* design — vague estimates signal vague understanding, which is the real risk to flag). Spikes to buy information on the genuinely-unknown parts before committing dates.
- Scoping as the senior lever: the negotiable variables are scope, time, quality, people — and **quality is the one you must refuse to trade** past a floor (cut scope, move the date, add people early — but shipping broken to hit a date pays interest forever, software-engineering §4 tech-debt). MVP thinking (the smallest thing that delivers and validates value — vertical slices, architecture §4), the 80/20 on which 20% delivers the value.
- Communicating estimates honestly to non-engineers: the confidence and the assumptions travel *with* the number (an estimate stripped of its assumptions becomes a promise you didn't make), re-forecasting as you learn (silence until the deadline is the cardinal sin — surface slippage early, when options still exist).
- **Exercise themes**: decompose and range-estimate a described feature and list the top three unknowns to spike first; given a fixed deadline that won't fit, propose the scope cut and the honest message to stakeholders.

## 7. Mentorship and multiplying others

- The leverage shift that defines Staff+: your output stops being your commits and becomes the capability of everyone around you — the hardest identity transition for strong individual contributors ("but I could just do it faster myself" is true and beside the point; doing it yourself scales to one, teaching scales to many).
- Teaching that transfers (this whole curriculum's pedagogy, career.md-side): let people struggle productively before rescuing (the desirable-difficulty lesson from roadmap.md — the rescue that robs the struggle robs the learning), ask guiding questions over giving answers (§3's review lesson generalizes), model your reasoning out loud (juniors need to see *how* seniors think, not just the conclusion), give feedback that's specific, timely, and kind-but-honest (the same calibration as the mentor's own operating principles).
- Sponsorship vs mentorship: mentorship is advice; sponsorship is spending your credibility to put someone on the visible project or say their name in the promo room — sponsorship is what actually moves careers, and it's the underused senior tool.
- Growing a team's capability as engineering: identifying and paying down "bus factor" (the knowledge silo is a reliability risk — devops thinking applied to people), documentation and golden paths as force multipliers (devops §11), psychological safety as the precondition for the blameless culture that makes postmortems (devops §10) and honest review (§3) work at all.
- **Exercise themes**: turn a "just tell me the answer" request into a guiding-questions coaching dialogue; identify the bus-factor risks in a described team and design the paydown; distinguish three situations as needing mentorship vs sponsorship.

## 8. The engineering career ladder

- The IC track as real and senior (not a consolation for non-managers): Senior → Staff → Principal → Distinguished as increasing *scope of impact* (team → org → company → industry) and *ambiguity handled*, not increasing lines of code. The management track as a *different job*, not a promotion — teach the fork honestly so the user chooses rather than drifts.
- What actually changes going up: the L3→L4 jump is scope and influence (§5) and problem-selection (picking the right problem beats solving the assigned one well); the L4→L5 jump is judgment on unprecedented problems and technical direction-setting for the org. Titles lag impact — operate at the level before you hold it (the promotion recognizes what you're already doing).
- The archetypes (Staff Engineer literature — Larson): tech lead, architect, solver, right-hand — different flavors of Staff impact; knowing which fits the user's strengths and context.
- Career as a portfolio and a narrative: visible impact (the work nobody sees doesn't count for promotion — a real and slightly cynical truth to teach, alongside doing work worth seeing), the promo packet as evidence-gathering you do continuously not frantically, brag docs, choosing projects for growth-and-visibility not just interest.
- Avoiding the plateaus: the senior-who-won't-delegate (§7), the architect-who-can't-communicate (§1–2), the expert-in-a-dying-technology (deliberate breadth), burnout as a career risk not a badge.
- **Exercise themes**: self-assess against the per-track levels (roadmap.md) and name the specific gap to the next level and its evidence; draft a brag-doc entry for real recent work framed by impact.

## 9. Interviewing: both sides of the table

- **As a candidate** — the four rounds as skills to train, not hoops: coding (think out loud — they're buying your problem-solving process, not the memorized answer; clarify → approach → tradeoffs → code → test → complexity; the DS&A from software-engineering §6 as vocabulary, but communication and structure are what's actually scored), system design (the whole architecture.md file cashing out — requirements and scale clarification *first*, then the four questions §1, drive it yourself, name tradeoffs unprompted; the candidates who fail are usually the ones who start drawing boxes before asking what they're building), behavioral (STAR structure, real stories with your specific decisions and what you'd do differently — the §10 self-awareness signal), and the domain/values rounds.
- The meta-skill: interviews reward *structured thinking made visible* — the same BLUF/altitude/tradeoff-naming skills from §1 and §4, under time pressure. Practice the narration, not just the solving.
- **As an interviewer** (an L4+ responsibility with real stakes): design questions that reveal thinking not trivia (open problems with room for tradeoff discussion beat gotchas), reduce bias (structured rubrics, consistent questions, separate signal from "reminds me of me" — the affinity trap), create conditions for the candidate to do their best work (a nervous candidate under a hostile interviewer shows you nothing useful), and calibrate ("would I want to work with and learn from this person" over "did they get the optimal answer"). A bad interviewer is a hiring-quality bug and a brand cost.
- The ethics and the long game: the industry is small and interviewing is two-way (the candidate is evaluating you too); reputation compounds on both sides.
- **Exercise themes**: run a mock system-design interview (user as candidate, graded on requirement-clarification and tradeoff-narration, not the "right" architecture); take a STAR story and sharpen it to specific-decision-and-learning; critique a provided interview question for what signal it actually measures and redesign a gotcha into a thinking-revealer.

## 10. Working with the business

- The frame that unlocks L4+ influence: engineering exists to produce *outcomes*, and the engineer who can connect technical work to user and business value (§5) gets trusted with bigger problems and argues from evidence instead of preference. "This is better engineering" loses to "this cuts checkout latency 200ms, which past experiments say lifts conversion ~2%" every time — learn to speak in outcomes.
- Product and metrics literacy (enough to be a real partner, not a ticket-taker): the metric taxonomy (leading vs lagging, North Star vs guardrail, vanity vs actionable), funnels and where they leak (design.md §13), retention/engagement/activation basics, and unit economics/cost-as-a-feature (devops §12) — the engineer who understands the business model designs better systems.
- Experimentation and decision science literacy: A/B testing done honestly (hypothesis, sample size and power, significance vs practical importance, the peeking problem, why most results are noise), the limits of experimentation (it optimizes locally and can't tell you strategy — some bets must be made on judgment), and the qualitative-vs-quantitative complement (what users *say* vs *do*, UX research from design.md §8). Guardrail metrics so a win on one number isn't a silent loss on another (Goodhart, again — the through-line of this whole curriculum).
- Prioritization frameworks as communication tools (RICE/ICE and kin — imperfect but they make the tradeoff *legible and shared* rather than hidden in someone's head), the build/buy/partner decision at product scale (dependency judgment, software-engineering §12, at the org level), and saying no / managing scope with stakeholders (§6).
- Engineering ethics as part of the job, not an afterthought: dark patterns (design.md §13) and the LTV cost of deception, privacy and data responsibility (security §17–18), accessibility as inclusion not compliance (design.md §11), the societal impact of what gets shipped (AI's weight here especially, ai.md §10) — and the engineer's standing and responsibility to raise concerns.
- **Exercise themes**: reframe three "better engineering" proposals as business-outcome arguments with the metric each moves; critique a described A/B test for power, peeking, and guardrails; take a feature request and prioritize a described backlog with RICE, then defend where the framework's number felt wrong.

---

## Self-review checklist (before shipping any significant communication)

1. Conclusion first? Could a busy reader get the answer from the first paragraph?
2. Right altitude for *this* audience — and translated for each audience if multiple?
3. Alternatives and tradeoffs shown honestly (for a proposal/decision), or is this a decision seeking rubber-stamp?
4. Uncertainty stated — confidence levels, assumptions traveling with estimates, what would change my mind?
5. Skimmable — structure, one clear point per section, diagrams at the right altitude?
6. Does it enable good disagreement, or foreclose it? Have I pre-wired the people who'll be surprised?
7. Should this be a doc instead of a meeting (or a call instead of this doc)?

## Case study / discussion themes (L3+)

The Staff Engineer archetypes (Larson) applied to the user's own context; famous engineering-culture documents (Netflix, Stripe, Amazon's narrative/six-pager and working-backwards memo) as writing-culture studies; a real promo-packet structure walk-through; postmortem-culture as an org-design achievement (devops §10, the leadership side); the "brilliant jerk" cost calculation (why top orgs fire high-performers who erode teams). For each: what forces, what the practice optimizes for, and how the user would apply or adapt it.
