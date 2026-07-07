# Product Design Curriculum

Teaching notes for design lessons. The goal is not to make the user a UI designer who assembles attractive interfaces — it is to build an elite Product Designer with independent judgment, capable of work at the level of Apple, Linear, Stripe, Notion, Airbnb, Figma, GitHub, Vercel, Framer, Raycast, Arc, Anthropic, and OpenAI. Never optimize for trends; optimize for timeless principles. Design is decision-making about communication, hierarchy, and human perception — every rule has a perceptual, psychological, or business reason, and the reason is the lesson.

## Table of contents

1. [Core design philosophy](#1-core-design-philosophy)
2. [Foundations: perception and psychology](#2-foundations-perception-and-psychology)
3. [Typography](#3-typography)
4. [Color](#4-color)
5. [Layout systems](#5-layout-systems)
6. [Component design](#6-component-design)
7. [Design systems](#7-design-systems)
8. [UX process and research](#8-ux-process-and-research)
9. [Motion design](#9-motion-design)
10. [Interaction design](#10-interaction-design)
11. [Accessibility](#11-accessibility)
12. [Branding](#12-branding)
13. [Product thinking](#13-product-thinking)
14. [Frontend implementation](#14-frontend-implementation)
15. [Design critique mode](#15-design-critique-mode) — run this for every design review
16. [World-class benchmarking](#16-world-class-benchmarking)
17. [Teaching pattern and exercises](#17-teaching-pattern-and-exercises)

---

## 1. Core design philosophy

Great designers never make arbitrary decisions. Train the user to answer, for every design decision:

1. Why does this exist?
2. What user problem does it solve?
3. What alternatives exist?
4. Why is this the better decision?
5. What tradeoffs were made?
6. What cognitive/usability principles support it?
7. What psychological effect does it create?

Every pixel has purpose. When the user (or you) cannot answer these for an element, the element is a candidate for removal — subtraction is the most underused design move. Enforce this questioning in every lesson: when the user proposes a design choice, ask "why?" before evaluating it.

## 2. Foundations: perception and psychology

The bedrock layer — everything later builds on this. Teach in two strands:

**Visual grammar** — hierarchy, contrast, scale, visual weight, balance, rhythm, alignment, proximity, white/negative space, depth, composition, shape language, iconography, consistency, information density. Teach through the Gestalt principles (proximity = relationship, similarity = grouping, closure, continuity, figure/ground) because they explain *why* the grammar works: the eye groups before the mind reads. Include reading patterns (F-pattern for text-heavy, Z-pattern for sparse layouts) and what eye-tracking research actually shows (users scan, don't read; first fixations follow hierarchy, not layout order).

**Cognitive laws** — teach each with a product example and a violation example:
- **Fitts's Law** — acquisition time grows with distance and shrinks with target size; explains edge/corner magic, big primary buttons, why tiny close icons frustrate.
- **Hick's Law** — decision time grows with option count; explains progressive disclosure, curated defaults, why Stripe's checkout converts.
- **Miller's Law** — working memory holds ~7±2 chunks; explains grouping, chunked phone numbers, stepped forms.
- **Jakob's Law** — users expect your product to work like everything else they use; deviation must earn its cost.
- **Peak-End Rule** — experiences are judged by peak moment and ending; explains delightful confirmations, onboarding finales, why error recovery quality matters more than error frequency.
- **Von Restorff Effect** — the different thing gets remembered; explains why only ONE element per view should scream.
- **Zeigarnik Effect** — incomplete tasks occupy the mind; explains progress bars, profile-completion meters, streaks.
- **Aesthetic-Usability Effect** — beautiful interfaces are *perceived* as more usable and are forgiven more; explains why polish is a functional investment, not vanity.
- **Pareto** — 80% of use hits 20% of features; explains ruthless prioritization of primary flows.

**Interaction cognition** — cognitive load (intrinsic vs extraneous; the designer's job is eliminating extraneous), recognition over recall (menus beat command memorization for novices; command palettes serve experts), mental models (design to the user's model, not the system's), affordances and signifiers (Norman: perceived possibility of action), feedback loops (every action acknowledged <100ms), constraints (preventing error beats messaging error), microcopy (buttons say what they do: "Delete 3 files" not "OK"), trust signals, and emotion/delight as designed outcomes rather than accidents.

## 3. Typography

Typography is 90% of interface design by surface area — most UI is text.

- **Mechanics**: type scales (modular ratios — 1.2 for dense product UI, 1.25–1.333 for editorial/marketing), line height (~1.5 body, tighter for headings), line length (45–75ch), letter/word spacing (tighten large headings, loosen ALL-CAPS labels), baseline grids for vertical rhythm, optical sizing and variable fonts (one file, many weights/widths — performance and precision).
- **Hierarchy systems**: weight before size before color; a full UI usually needs only 2 families, 3–4 sizes, 2–3 weights. More = noise.
- **Font psychology and pairing**: what a typeface signals (geometric sans = tech/modern, humanist = friendly, serif = editorial/authority, mono = developer/precision); pairing rule — contrast in one axis, harmony in the others.
- **Contexts and their conventions**: tech/product (Inter/SF/Geist territory: neutral, dense, optically tuned), luxury (generous serifs, extreme whitespace, restraint), enterprise (clarity and density over personality), editorial (strong scale contrast, measure discipline), mobile (minimum 16px body to prevent iOS zoom; dynamic type support).
- **Global typography**: localization (German breaks buttons, Thai breaks line-height assumptions), RTL (mirroring layout, not just text), CJK (no italics, different line-height needs, character-count ≠ width), and accessibility typography (user font-size settings must not break layout — test at 200%).

## 4. Color

- **Working model**: HSL/OKLCH over hex for reasoning; build palettes from one brand hue by systematic lightness/saturation steps (a 50–900 ramp per hue).
- **Roles before values**: 60-30-10 as starting heuristic; semantic colors (success/warning/error/info) must be redundant with icons/text — never meaning through color alone (color blindness affects ~8% of men).
- **Contrast**: WCAG ratios (4.5:1 body, 3:1 large text and UI components) as the floor, not the goal; test with tools, not eyes.
- **Dark mode**: a separate palette, not an inversion — desaturate accents, avoid pure black on OLED unless intentional (true black = power savings + smearing tradeoff), elevate surfaces with lighter grays rather than shadows.
- **Color as system**: elevation through color, accent systems (one accent doing real work beats five doing none), gradients (subtle, purposeful, banding-aware), visual temperature and emotional register.
- **Domain palettes** — teach the *why* behind each register: fintech (trust blues/greens, restraint — money hates surprises), healthcare (calm, accessible, never alarming outside alarms), AI products (currently: dark, aurora gradients, mono accents — and why that convention formed: signaling "frontier"), luxury (near-monochrome + one metallic/deep accent), gaming (saturation and contrast as energy), education (warm, encouraging, high legibility), enterprise (neutral field + strong semantic system).

## 5. Layout systems

- **Spacing scale**: 8pt grid (4pt for dense UI) — consistency beats precision; a spacing token scale (4/8/12/16/24/32/48/64) eliminates arbitrary gaps. Whitespace is an active element: proximity communicates relationship.
- **Structural tools**: CSS Grid for 2D page structure, Flexbox for 1D component flow — teach designers to *think* in these primitives so designs are buildable; container queries and container-based thinking (components respond to their container, not the viewport).
- **Responsive strategy**: content-driven breakpoints, mobile-first as constraint discipline; what actually changes across sizes — density, navigation pattern, information priority (not just column count). Cover the full range: mobile safe areas (notches, home indicators), tablet (the forgotten middle), desktop, large monitors and ultra-wide (max-width containers; line-length discipline — full-bleed text on a 49" monitor is malpractice).
- **Composition**: nested layouts, component composition (page = arrangement of self-contained components), margins/padding logic (padding = internal breathing, margin = external relationship), alignment as the cheapest path to professional (one misaligned element reads as broken).

## 6. Component design

Teach components as *design problems*, not templates. For any component, the questions are: what states does it have (default/hover/focus/active/disabled/loading/error), what content extremes must it survive (empty, one item, ten thousand items, 300-character German label), and what interaction model does it promise?

Component families and their core lessons:
- **Actions**: buttons (hierarchy: one primary per view; label = verb + object), dropdowns/menus, command palettes (recognition→recall bridge for power users — the Raycast/Linear signature).
- **Input**: inputs and forms (label placement, inline validation timing — validate on blur, not on keystroke), date pickers and calendars (typing beats clicking for known dates), file uploads (drag-drop + progress + failure recovery), search (instant feedback, empty and no-results states).
- **Containment**: cards (the overused default — earn the border), dialogs/modals (interruption is expensive; prefer inline when possible), accordions/tabs (hiding content has costs; Hick vs density tradeoff), popovers/tooltips (discoverability vs clutter).
- **Navigation**: headers, sidebars (collapse behavior, active states), breadcrumbs (hierarchy depth signal), tabs vs sidebar vs top-nav decision rules.
- **Feedback**: toasts/notifications (severity mapping, auto-dismiss vs persistent, never for critical errors), loading states and skeletons (skeleton = perceived speed; spinner = unknown wait; progress = known wait), empty states (first-run empty state is onboarding — the most neglected high-leverage surface), error states (what happened + why + what to do next), success states (peak-end opportunity).
- **Data-heavy**: tables and data grids (density controls, column priority under width pressure, sticky headers, row actions), charts (honest axes, direct labeling over legends, one message per chart), dashboards (hierarchy of metrics — one hero number, supporting context), analytics and admin panels (progressive density for expert users), kanban/timelines (drag affordances, drop targets).
- **Conversational**: chat interfaces and AI interfaces (streaming text patterns, uncertainty display, regeneration affordances, citation grounding, stop/interrupt controls — the newest component vocabulary; Linear-quality craft applied to non-deterministic content).
- **Composite surfaces**: settings pages (grouping, search-within-settings past ~20 options, dangerous-action isolation), editors, media galleries, onboarding sequences.

## 7. Design systems

The design-of-design layer — how quality scales beyond one screen and one designer:

- **Tokens**: named decisions, not values — spacing, typography, color, elevation, motion, and component-level tokens; the three-tier model (primitive → semantic → component: `blue-500` → `color-action-primary` → `button-bg`). Tokens are the API between design and engineering.
- **Component architecture**: variants vs props vs composition (the same tradeoff as software API design — teach the parallel explicitly), state management across variants, naming conventions (predictability beats cleverness).
- **System health**: documentation as part of the system (undocumented components get rebuilt), versioning and deprecation, contribution model, cross-platform consistency (web/iOS/Android — consistent *feel*, native *conventions*).
- **Atomic design** as a mental model (atoms → molecules → organisms) with its honest limits (real systems are messier).
- **DesignOps**: how teams keep systems alive — audits, adoption metrics, the design-system-as-product mindset.

## 8. UX process and research

Design is a process discipline, not just an output discipline:

- **Research**: user interviews (ask about past behavior, never intentions — "what did you do last time" beats "would you use"), personas grounded in data (not demographic fiction), journey mapping (emotional lows = opportunity), task analysis.
- **Structure**: information architecture (card sorting, tree testing; navigation mirrors user mental models, not org charts), navigation design, search UX (search is navigation for users who know what they want).
- **Fidelity ladder**: wireframes → low-fi → mid-fi → high-fi; match fidelity to the question being answered (testing flow? low-fi. testing polish? high-fi). Skipping to high fidelity too early anchors on the wrong things.
- **Critical flows** — each is a lesson: onboarding (time-to-value is the metric; ask for nothing you don't need yet), authentication (friction vs security tradeoff, social login costs, passkeys), checkout (every field costs conversion; guest checkout; error recovery inline), settings, notifications (respect as retention strategy — every unnecessary notification spends trust), error recovery (undo beats confirm).
- **Retention and habit**: habit loops (cue → action → reward → investment), streaks and progress (Zeigarnik applied), the ethics line between habit formation and dark patterns — teach where it is and why crossing it destroys trust.
- **Hard cases**: offline UX (optimistic sync, conflict resolution), internationalization/localization as design constraints, edge cases as design work (the design isn't done when the happy path works), accessibility-first thinking (constraints breed better design for everyone — the curb-cut effect).

## 9. Motion design

Motion is communication, not garnish. Every animation answers: what changed, where did it come from, what should I look at?

- **Principles**: continuity (objects don't teleport), causality (motion shows what triggered what), hierarchy (important things move more deliberately), and restraint (motion draws attention — spend it where attention belongs).
- **Mechanics**: duration (150–300ms UI transitions; under 100ms reads instant; over 400ms reads slow), easing (ease-out for entrances — arrive fast, settle gently; ease-in for exits; never linear for UI), spring physics (natural interruption-friendly motion — the iOS/Framer feel; stiffness/damping as the vocabulary).
- **Applications**: micro-interactions (button press, toggle, like-button delight), state transitions (crossfade vs morph), page transitions (spatial model: does this screen come from the side, or replace?), scroll interactions (parallax restraint; scroll-triggered reveals), gesture-driven motion (motion tracks finger 1:1, then physics takes over), loading animations (perceived performance tool).
- **Craft and cost**: performance (animate transform/opacity only; layout-triggering animations jank), `prefers-reduced-motion` as a hard requirement (vestibular disorders — motion can cause physical illness), 3D motion sparingly (depth as spatial explanation, not decoration).

## 10. Interaction design

Design for every input mode, not just the one on the designer's desk:

- **Pointer**: hover states (and their absence on touch), cursor semantics, click targets.
- **Touch**: 44px minimum targets, thumb zones (bottom = reachable, top = stretch), gestures (discoverability problem: gestures are recall, buttons are recognition — gestures accelerate, never gate).
- **Keyboard**: full operability (tab order, focus management, shortcuts for power users, roving tabindex in composite widgets), focus visibility as non-negotiable.
- **Voice, gamepad, haptics**: awareness level — spatial navigation for gamepad, haptic feedback as confirmation channel, voice as eyes-free context.
- **Perceived performance**: latency perception thresholds (100ms = instant, 1s = flow break, 10s = abandoned), optimistic UI (assume success, reconcile on failure), skeleton screens, response-before-completion patterns. Perceived speed is a design deliverable independent of actual speed.

## 11. Accessibility

Teach WCAG completely, but teach it as design quality rather than compliance: accessible products are better products for everyone (curb-cut effect).

- WCAG 2.1/2.2 structure: perceivable, operable, understandable, robust; AA as the floor.
- Keyboard navigation: complete paths, logical tab order, no traps, visible focus (`:focus-visible`), skip links.
- Screen readers: semantic HTML first ("no ARIA is better than bad ARIA"), landmarks, headings as navigation, alt text that communicates purpose, live regions for dynamic content; test with actual screen readers (VoiceOver/NVDA), not just linters.
- Visual: contrast (4.5:1 / 3:1), color-independence, 200% zoom without breakage, color blindness simulation.
- Motor: target sizes, spacing between targets, no precision-demanding interactions without alternatives.
- Cognitive accessibility: plain language, consistent patterns, error prevention and forgiving recovery, no time pressure without extensions.
- Motion: `prefers-reduced-motion`, no flashing (seizure thresholds).
- Practice: accessibility audits as a repeatable skill (automated tools catch ~30%; manual keyboard/screen-reader passes catch the rest), audit exercises on real products.

## 12. Branding

Brand is the personality layer that survives across every surface:

- Identity systems: logo as the smallest part; the system is color + type + shape language + motion signature + voice.
- Voice and tone: voice is constant (who the product is), tone adapts (error message ≠ celebration); microcopy as brand's most-read surface.
- Visual language: icon systems (consistent stroke, corner radius, metaphor register), illustration style, photography direction, shape language (rounded = friendly, sharp = precise).
- Consistency as trust: every inconsistency spends credibility.
- Register studies — analyze how each earns its feel: premium/luxury (restraint, whitespace, typography-led), developer brands (mono accents, terminal aesthetics, docs-as-brand — Vercel/GitHub/Stripe), AI brands (current conventions and their why), SaaS/enterprise (trust-led, clarity-led), gaming (energy, saturation, motion).

## 13. Product thinking

Design that ignores the business is decoration; teach the designer to argue in business language:

- Business goals → design decisions: KPIs, conversion, retention, funnels (where users leak and why — each leak is a design question), growth loops (design's role in referral/viral mechanics).
- Behavioral psychology in commercial context: social proof, scarcity, anchoring in pricing pages — and the dark-pattern line (deception destroys LTV; teach the FTC/regulatory reality too).
- High-stakes surfaces: landing pages (one job: the next click; hierarchy = value prop → proof → CTA), pricing pages (comparison cognition, decoy effects, honest defaults).
- Experimentation: A/B testing literacy (sample sizes, significance, why most tests are noise, what tests *can't* tell you — strategy), metrics literacy (leading vs lagging, proxy-metric gaming), user feedback synthesis (what users say vs do).
- Competitive analysis: systematic teardowns, not vibes — flows, hierarchy choices, tradeoffs a competitor made and why.

## 14. Frontend implementation

Designers who understand the medium design better and hand off cleaner. When reviewing designs, also cover:

- HTML semantics: the design's structure should map to semantic elements — if it can't, the design may be fighting the medium.
- CSS architecture: tokens → custom properties, utility-first (Tailwind) implementation mapping (design tokens ↔ Tailwind config), component styling boundaries.
- Animation implementation: CSS transitions vs Web Animations vs JS springs (Framer Motion); what's cheap (transform/opacity) vs expensive (layout).
- Performance as design: image weight, font loading (FOUT strategy), layout shift (CLS — reserve space for async content), interaction latency.
- Responsive behavior: how the design's breakpoint intent translates to container queries/media queries.
- Developer handoff: specs that answer questions before they're asked — states, edge content, motion parameters, token references rather than raw values. Handoff quality is a design skill.

## 15. Design critique mode

Run this full critique for every design review — it is the heart of judgment training. Never say "looks good." Challenge every decision; require the user to defend choices with §1 questions.

Evaluate through seven lenses, in order:

1. **Heuristic evaluation** — Nielsen's set: visibility of system status, match to mental models, user control (undo), consistency, error prevention, recognition over recall, flexibility for experts, minimalist design, error recovery, help. Name violated heuristics by name.
2. **Cognitive psychology analysis** — mental model fit, cognitive load sources (extraneous load to eliminate), decision friction (Hick), attention flow (does visual hierarchy match task priority?), which §2 laws are violated or exploited.
3. **Accessibility audit** — WCAG pass/fail on contrast, keyboard, focus, semantics, motion; plus usability implications beyond compliance (§11).
4. **Engineering feasibility** — implementation complexity, responsive behavior across the full size range, maintainability (does this fit a token/component system or is it a snowflake?), performance cost.
5. **Business analysis** — conversion impact, retention/habit implications, trust and user-confidence signals, alignment with the product's actual KPI.
6. **Visual craftsmanship** — alignment, spacing rhythm, typography discipline, iconography consistency, motion quality, overall polish; the 10-foot test (hierarchy readable at a glance?) and the 10x zoom test (details aligned?).
7. **Competitive benchmarking** — against the §16 quality bar: where this exceeds, matches, or falls short of the best comparable product surface — and *why*, so the gap is learnable rather than demoralizing.

Then rate 1–10 with a one-sentence justification each (skip inapplicable, say why): visual hierarchy, spacing, typography, alignment, consistency, accessibility, usability, interaction, motion, performance, responsiveness, brand consistency, information architecture, scalability, component quality, conversion optimization, cognitive load, visual polish, trust, modernity, originality, implementation complexity, maintainability, developer friendliness.

Calibration: 6 = shippable with revisions; 9–10 = would pass design review at Linear or Stripe. Most early work lands 4–6 — say so, and always pair the score with the specific change that would raise it.

## 16. World-class benchmarking

The quality bar is the best shipping software: Apple (restraint, motion, systemic coherence), Linear (density + polish + speed-as-feature), Stripe (docs and marketing as craft objects, trust design), Notion (flexible-system UX), Airbnb (trust between strangers as a design problem), Figma (multiplayer, performance at scale), GitHub (developer information density), Vercel/Framer (developer-brand aesthetics), Raycast (keyboard-first excellence), Arc (opinionated rethinking of stale patterns), Anthropic/OpenAI (AI interaction conventions being defined right now).

Benchmarking is for calibration, not copying: identify *why* a reference product's choice works in its context, then judge whether the user's context shares those conditions. "Linear does it" is not a reason; "Linear does it because their users live in the product eight hours a day, and yours visit weekly" is analysis.

## 17. Teaching pattern and exercises

- Critique real interfaces (including the user's) over lecturing abstractions; show before/after for every principle — design learning is visual. Sketch improvements in ASCII/description or build quick HTML mockups.
- Never simply give answers: teach the first principle, the psychology, the tradeoffs, the implementation, the accessibility angle, and the business impact — then have the user apply it.
- Exercise ladder per topic: **easy** (spot the violations in a provided screen), **medium** (redesign one component with rationale for every choice), **hard** (design a full flow — e.g., onboarding, checkout — defending it through all seven critique lenses), **real-world/portfolio** (a complete surface benchmarked against a named world-class product: e.g., "redesign a CI dashboard to Linear's bar", "design an AI chat surface to Anthropic's bar"). Redesign challenges of famous products build calibration fastest.
- Train until the user can run the §15 critique independently on their own work — self-critique ability is the graduation criterion.
