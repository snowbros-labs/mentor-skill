# Frontend & Web Platform Curriculum

Frontend as a serious engineering discipline — not "the easy part," but the part that runs on hardware you don't control, networks you can't trust, and in front of the only judge that matters. Prerequisite: software-engineering.md basics. Pairs with design.md (the what and why of the interface; this file is the how) and connects to security.md §8 (frontend security). The unifying mental model: **the frontend is a distributed-systems client with a rendering engine attached** — state sync, latency masking, and failure handling are the real problems; pixels are the visible part.

## Table of contents

1. [The web platform](#1-the-web-platform)
2. [Rendering and the browser's pipeline](#2-rendering-and-the-browsers-pipeline)
3. [JavaScript/TypeScript at depth](#3-javascripttypescript-at-depth)
4. [Component architecture](#4-component-architecture)
5. [State management](#5-state-management)
6. [Data fetching and server state](#6-data-fetching-and-server-state)
7. [Rendering strategies: CSR, SSR, and the spectrum](#7-rendering-strategies-csr-ssr-and-the-spectrum)
8. [Web performance](#8-web-performance)
9. [CSS architecture](#9-css-architecture)
10. [Build systems and tooling](#10-build-systems-and-tooling)
11. [Frontend testing](#11-frontend-testing)
12. [Progressive enhancement, resilience, and offline](#12-progressive-enhancement-resilience-and-offline)

---

## 1. The web platform

- **First principle**: frameworks come and go; the platform compounds. HTML/CSS/DOM/HTTP knowledge from 2010 still pays; framework specifics from 2018 mostly don't. Teach platform-first, framework-second.
- Semantic HTML as the foundation stack: elements as meaning (the accessibility tree is *derived from* markup — semantics is how screen readers, SEO, and reader modes all work for free), forms as the platform's most underused machinery (validation, autofill, submission — half of most SPAs reimplements `<form>` badly), the URL as state (deep-linking, back-button correctness as a *correctness* requirement users feel instantly).
- Browser APIs worth fluency: DOM and events (bubbling/capturing/delegation — the model under every framework's synthetic events), History, Intersection/Resize/Mutation observers (the performance-friendly way to watch things), storage options mapped to their tradeoffs (cookie/localStorage/IndexedDB — cross-link security §8 for the XSS/CSRF exposure of each), Web Workers (the only real threads — cross-link §8).
- The event loop as required literacy: tasks vs microtasks, why long synchronous work freezes the page, `requestAnimationFrame`'s slot in the frame — this one model explains most "janky UI" mysteries (cross-link software-engineering §7).
- **Exercise themes**: rebuild a framework-heavy widget in platform HTML/JS and compare; find the div-soup accessibility failures and fix with semantics alone; explain a described jank from event-loop first principles.

## 2. Rendering and the browser's pipeline

- The pixel pipeline: parse → style → layout → paint → composite; which CSS properties trigger which stages (transform/opacity composite cheaply; width/top trigger layout — this is *why* the animation rule from design.md §9 exists).
- Layout thrashing: interleaved reads and writes of layout properties forcing synchronous reflow; batching reads/writes; `contain` and `content-visibility` as isolation tools.
- Compositing: layers, `will-change` (and its memory cost when abused), 60fps as a 16.6ms budget with the frame's fixed overhead already inside it.
- **Exercise themes**: given a janky animation, identify the pipeline stage at fault from DevTools and fix it; explain why animating `height` stutters while `transform: scaleY` doesn't, then implement the accessible version.

## 3. JavaScript/TypeScript at depth

- The parts that bite: closures and scoping (the loop-variable classic), `this` binding rules, prototypes (what classes desugar to), equality and coercion discipline, modules (ESM vs CJS as a real interop tax).
- Async JavaScript properly: promises as values, `async/await` sugar and its error-handling traps (the forgotten `await`, the unhandled rejection, `Promise.all` vs `allSettled` fan-out semantics), AbortController as the cancellation idiom (fetch-and-navigate-away is a leak and a state bug).
- TypeScript as design tool, not annotation chore: model the domain in types (unions over booleans-and-hope, discriminated unions as the state-machine idiom, `unknown` over `any` at boundaries), narrowing, generics where they earn keep, parse-don't-validate at the API boundary (runtime validation — zod-class — because the network is untyped no matter what your types say; cross-link security input validation).
- **Exercise themes**: type an API response pipeline end-to-end with runtime validation at the edge; find the three async bugs (race, unhandled rejection, missing cancellation) in a provided component.

## 4. Component architecture

- Components as the unit of thought: single responsibility at UI scale, props as API design (software-engineering §2 applies verbatim — components *are* deep-vs-shallow modules), composition over configuration (children/slots over prop explosions; the 23-prop component is the god-object smell in UI clothing).
- The container/presentational instinct (logic vs rendering separation) updated for hooks-era reality: custom hooks as the logic-extraction unit, headless components (logic without pixels — the design-system enabler, cross-link design.md §7).
- State ownership: lift state to the lowest common owner and no higher; derived state computed, not stored (the duplicated-state sync bug is the most common React bug); controlled vs uncontrolled inputs as a deliberate choice.
- Render discipline (React-flavored but transferable): what triggers re-render, referential identity as the currency of memoization, keys as identity (the index-as-key bug), effects honestly (effects are for synchronizing with external systems, not for deriving state — the misused-effect is the second most common React bug).
- Framework literacy without tribalism: React's model (immutability + reconciliation), signals-based reactivity (Solid/Vue/Svelte — fine-grained updates without VDOM), and what each paradigm makes easy/hard; the transferable skill is recognizing *reactive dataflow* as the shared idea.
- **Exercise themes**: refactor a 400-line component into a composed family; find the unnecessary re-renders with the profiler and fix the top two; convert a stored-derived-state bug into computed state.

## 5. State management

- **Taxonomy first, library second** — the biggest state-management error is one bucket for everything: server state (someone else owns it; it's a *cache* — §6), UI state (ephemeral, component-local: modals, inputs), app state (client-owned, cross-cutting: theme, session), URL state (shareable/bookmarkable — belongs in the URL, the most-forgotten bucket), form state (its own beast: dirty/touched/validation lifecycles).
- Local-first by default: most "we need a store" moments are mis-bucketed server or URL state; global stores are for genuinely global, client-owned state — which is less than people think.
- When stores earn their place: prop-drilling pain vs context performance costs vs external stores (Redux-class: predictability and devtools at boilerplate cost; Zustand-class: the lightweight middle); state machines (XState-class) where flows have real states and transitions (wizards, media players, connection lifecycles) — impossible-state elimination as the sell (make illegal states unrepresentable, again).
- Immutability and unidirectional data flow as the debugging enabler: state → view → action → state; when you can't answer "who changed this and when," the architecture failed.
- **Exercise themes**: bucket every piece of state in a described app into the five categories; replace a misused global store with URL + server-cache state; model a checkout flow as a state machine.

## 6. Data fetching and server state

- Server state is a **cache with a UI attached** — staleness, revalidation, and invalidation are the actual problems (architecture.md §8 economics apply at component scale). The query-library pattern (TanStack Query-class): cache keys as dependencies, stale-while-revalidate, focus/interval revalidation, invalidation on mutation.
- Mutations done honestly: optimistic updates (assume success, render immediately, reconcile on failure — with rollback UX designed, not improvised; cross-link design.md perceived performance), request deduplication, race handling (the stale-response-overwrites-fresh bug: sequence numbers or cancellation).
- Loading and error as designed states, not afterthoughts: skeletons vs spinners vs progressive content (design.md §6), error boundaries and retry affordances, empty states; the loading waterfall as the classic performance bug (parallelize or hoist fetches; render-as-you-fetch).
- Real-time: polling vs SSE vs WebSockets (escalate only as needed — polling is underrated), reconnection with backoff, out-of-order message handling, presence/optimistic-concurrency basics for collaborative UI.
- **Exercise themes**: fix the fetch waterfall in a provided page; implement optimistic toggle-with-rollback; design the cache-invalidation map for a CRUD feature (which mutations invalidate which queries, and why).

## 7. Rendering strategies: CSR, SSR, and the spectrum

- The decision is about *where HTML is produced and when*: CSR (blank-page cost, cheap hosting, app-like), SSR (fast first paint, server cost, hydration), SSG (build-time HTML — unbeatable for content that changes rarely), ISR/streaming/islands/server-components as the modern refinements (send less JS, hydrate less, stream sooner).
- Hydration honestly: why it's expensive (re-running the app to attach listeners), the uncanny valley (visible but dead UI), mismatch bugs (cross-link security §8 hydration risks), and the ecosystem's countermoves (partial hydration, islands, resumability).
- Decision framework by page class, not by app: marketing page ≠ dashboard ≠ editor — mixed strategies within one product are correct, not messy; SEO and social-preview realities; the meta-framework layer (Next/Remix/Astro-class) as productized versions of these tradeoffs.
- **Exercise themes**: assign rendering strategies across a described product's page classes and defend each; diagnose a hydration mismatch from its symptom.

## 8. Web performance

The user-facing performance discipline (general method in software-engineering §8 — measure first applies doubly here because lab ≠ field):

- **Core Web Vitals as the shared language**: LCP (loading — largest element painted), INP (interaction responsiveness — the event-loop lesson cashing out), CLS (visual stability — reserve space for async content); lab vs field data (RUM) and why field wins arguments.
- The loading budget: the critical path (HTML → CSS → render-blocking JS), resource hints (preload/preconnect/prefetch), code splitting by route as the default (ship the page's JS, not the app's), lazy-loading below the fold (images, components), font strategy (FOUT over invisible text, subsetting, `font-display`), image discipline (modern formats, responsive `srcset`, dimensions-for-CLS — images are the #1 payload in most pages).
- The runtime budget: long tasks (>50ms) blocking interaction, breaking work up (yielding, `startTransition`-class scheduling, workers for real computation), list virtualization for anything unbounded, memory leaks in SPAs (listeners and closures that outlive their view — heap-snapshot literacy).
- The network reality: test on throttled mid-range devices, not the dev machine (the median user's hardware is ~5 years behind yours — this is an equity issue as much as an engineering one); bundle analysis as routine (know what you ship; the accidental 300KB dependency).
- Performance as regression discipline: budgets in CI (size, vitals), because performance decays by a thousand small merges and recovers only by painful projects.
- **Exercise themes**: audit a provided page with Lighthouse/DevTools, fix the top three findings, show before/after; find the memory leak from a heap snapshot; set and justify a performance budget for a described product.

## 9. CSS architecture

- CSS at scale is an architecture problem: the cascade and specificity as the global-mutable-state of styling; every methodology (BEM, CSS Modules, utility-first, CSS-in-JS, scoped styles) is a strategy for *containing the cascade* — teach them as answers to the same question, with tradeoffs (utility-first: locality and constraint-by-design-tokens vs HTML noise; modules: real scoping vs naming ceremony; CSS-in-JS: co-location and dynamism vs runtime cost).
- Modern layout fluency: Grid for 2D structure, Flexbox for 1D flow, custom properties as the token delivery mechanism (design.md §7 tokens land here), container queries (component-responsive over page-responsive — the architectural upgrade), logical properties (i18n/RTL-ready by default, cross-link design.md §3).
- Design-system CSS: tokens → custom properties → components; theming (light/dark via token swap, not selector forests); the utility/component boundary in practice.
- **Exercise themes**: refactor specificity-war CSS into a contained architecture; implement a themed component set from design tokens; make a fixed component container-responsive.

## 10. Build systems and tooling

- What the toolchain actually does — demystified: transpile (syntax down-leveling), bundle (module graph → deliverable chunks), minify, tree-shake (dead-export elimination and why side-effects break it), source maps (the debugging lifeline), dev server + HMR; why native ESM changed the game (dev-time unbundling — Vite-class speed).
- Judgment calls: transpilation targets (browserslist as an *audience* decision, not a default you inherit), polyfill weight, `package.json` exports/module resolution as the interop swamp worth one deep lesson, monorepo tooling at awareness level (workspaces, task graphs, remote caching — Turborepo/Nx-class).
- Developer experience as an engineering product (the platform-engineering mindset applied to frontend): fast feedback loops (dev server, HMR, type-check speed) compound into team velocity; linting/formatting as codified agreement (argue once, in config); CI for frontend (typecheck, lint, test, build, size budget, preview deploys — cross-link devops.md).
- **Exercise themes**: read a bundle-analyzer report and cut 30%; explain why a library double-bundles (ESM/CJS) and fix the resolution; design the CI pipeline for a described frontend.

## 11. Frontend testing

- The trophy over the pyramid here: heavy integration-level component tests (render, interact, assert what the *user* sees), because frontend units are often trivial and wiring is the risk (software-engineering §3's "cheapest level that catches the failure" logic, applied).
- Testing-library discipline: query by role/label (accessibility-first selectors double as a11y checks — tests fail when semantics fail), user-event simulation over implementation poking; the async patterns (`findBy`, waiting for states, fake timers) that kill flakiness.
- Mock the network, not the modules (MSW-class): tests exercise real fetch/cache/error paths against a controlled API — catches the integration bugs module-mocks hide.
- E2E judiciously (Playwright-class): the critical journeys (signup, checkout), cross-browser reality, flake discipline (auto-waiting, test isolation, no sleep(3000)); visual regression testing for design-system components (screenshot diffs where pixels *are* the contract); accessibility assertions in CI (axe-class) as the floor, manual passes as the ceiling (design.md §11).
- **Exercise themes**: write component tests for a form (validation, submission, server error, success) with MSW; de-flake a provided E2E test; add visual regression to a button set.

## 12. Progressive enhancement, resilience, and offline

- The resilience ladder: works without JS where feasible (links navigate, forms submit — the platform baseline), degrades gracefully when JS fails (a script error shouldn't blank the page — error boundaries, defensive third-party loading), handles network failure as a designed state (offline banners, retry affordances, queued mutations).
- Offline and sync when the product warrants: service workers honestly (cache strategies — network-first/stale-while-revalidate per resource class; the update problem — stale-SW bugs; security §8 scope risks), background sync, conflict resolution at awareness level (last-write-wins vs merge vs CRDTs for collaborative products).
- Third-party script hygiene: every tag is a performance tax, a failure mode, and a supply-chain grant (security §13) — load async, sandbox where possible, measure their cost, own the kill switch.
- **Exercise themes**: make a described form survive JS failure and network failure; design the offline strategy for a field-work app (what caches, what queues, what conflicts).

---

## Review checklist (frontend changes)

1. Semantics/a11y: roles from markup, keyboard path, focus management, labels — design.md §11 quick pass.
2. States: loading, error, empty, success all designed and reachable; no layout shift from async content.
3. State placement: right bucket (server/UI/app/URL/form); no stored-derived-state; race-safe mutations.
4. Performance: bundle impact checked, no new waterfall, images sized, long-task risk on interaction paths.
5. Types: boundaries validated at runtime, no `any` leaks, illegal states unrepresentable where cheap.
6. Tests: user-visible behavior covered at component level, critical path E2E updated if touched.
7. Security quick pass: security.md §8 (sinks, injection surfaces, storage choices, postMessage origins).
8. Resilience: what happens when this component's fetch fails? When its script errors?

## Case study themes (L3+)

The cost-of-JavaScript literature (why the industry swung back toward servers); a hydration-era framework migration write-up (what SSR actually bought and cost); Figma's performance engineering (C++/WASM where the web platform ran out); GitHub's move off jQuery (platform-first bet); any public RUM-driven performance turnaround (before/after Vitals and business metrics). For each: forces → decision → outcome → what would you have done.
