# Security Engineering Curriculum

Sequencing and teaching notes for security lessons. Teach threat *thinking* first — individual vulnerabilities are instances of a small number of root causes. A user who sees the root cause can recognize novel variants; a user who memorizes OWASP entries cannot.

## Table of contents

1. [The threat modeling mental model](#1-the-threat-modeling-mental-model) — teach this before anything else
2. [Secure design principles](#2-secure-design-principles) — the vocabulary of "why"
3. [Vulnerability families by root cause](#3-vulnerability-families-by-root-cause)
4. [Authentication, sessions, and identity](#4-authentication-sessions-and-identity)
5. [Authorization](#5-authorization)
6. [Tokens and federation](#6-tokens-and-federation)
7. [API security](#7-api-security)
8. [Frontend security](#8-frontend-security)
9. [Application-specific attack surfaces](#9-application-specific-attack-surfaces)
10. [Database security](#10-database-security)
11. [Architecture-level security](#11-architecture-level-security)
12. [Infrastructure security](#12-infrastructure-security)
13. [Secure DevOps and supply chain](#13-secure-devops-and-supply-chain)
14. [Detection, monitoring, and response](#14-detection-monitoring-and-response)
15. [OWASP cross-reference](#15-owasp-cross-reference)
16. [Teaching pattern for any vulnerability](#16-teaching-pattern-for-any-vulnerability)
17. [Security code review framework](#17-security-code-review-framework)

---

## 1. The threat modeling mental model

Teach this chain before STRIDE, before any vulnerability. Someone who internalizes trust boundaries will recognize whole classes of vulnerabilities without memorizing them.

```
Assets          what is worth protecting (data, money, reputation, availability)
  ↓
Actors          who touches the system (users, admins, services, attackers)
  ↓
Entry Points    where input enters (endpoints, uploads, webhooks, queues, headers)
  ↓
Trust Boundaries  where data crosses from less-trusted to more-trusted context
  ↓
Data Flow       how data moves between components across those boundaries
  ↓
Threats         what can go wrong at each boundary (STRIDE as the checklist here)
  ↓
Controls        what stops or limits each threat
```

STRIDE (Spoofing, Tampering, Repudiation, Information disclosure, DoS, Elevation of privilege) slots in at the "Threats" step — it is a checklist for enumerating, not a substitute for the model. Attack trees add depth for high-value assets.

First exercise for any student: draw this chain for an app they know well. Every later lesson refers back to their diagram.

## 2. Secure design principles

Establish these before diving into vulnerabilities — they are the reason secure implementations look the way they do. Reference them by name in every code review so the vocabulary sticks.

- **Least Privilege** — every actor/component gets the minimum access needed. Explains: scoped tokens, DB roles, IAM policies.
- **Defense in Depth** — no single control is trusted to hold. Explains: CSP on top of output encoding, WAF on top of parameterized queries.
- **Fail Secure** — errors deny by default. Explains: why a crashed authz check must reject, not allow.
- **Secure by Default** — safe unless deliberately opened. Explains: private buckets, deny-all network policies.
- **Zero Trust** — never trust based on network location; verify every request. Explains: service-to-service auth, mTLS.
- **Minimize Attack Surface** — every endpoint, dependency, and feature is exposure. Explains: removing debug routes, trimming dependencies.
- **Complete Mediation** — check authority on *every* access, not once at the front door. Explains: why cached authz decisions rot, why IDOR exists.
- **Separation of Duties** — no single actor can complete a sensitive action alone. Explains: deploy approvals, dual-control for payments.
- **Economy of Mechanism** — simple designs are auditable; complexity hides flaws.
- **Least Common Mechanism** — shared components are shared blast radius; isolate where sharing isn't required.

## 3. Vulnerability families by root cause

Organize vulnerabilities by root cause, not by OWASP list position. Teach the family's root cause first, then instances — a new variant (LDAP injection, SSTI) becomes recognizable rather than another thing to memorize.

| Root cause | Vulnerabilities | Core defense |
|---|---|---|
| **Data interpreted as code** | SQLi, XSS (reflected/stored/DOM), command injection, SSTI, LDAP injection, path traversal | Keep data and instructions in separate channels: parameterized queries, context-aware output encoding, no string-built shell commands |
| **Trusting user-supplied identity/references** | IDOR/BOLA, privilege escalation, mass assignment | Complete mediation: authorize every object access server-side; allowlist bindable fields |
| **Broken trust assumptions between components** | CSRF (browser's ambient cookie authority), SSRF (server's network position), open redirects | Prove intent (CSRF tokens, SameSite); allowlist outbound destinations; never trust "internal" |
| **Unsafe object/data handling** | Insecure deserialization, prototype pollution, XXE, zip-slip | Never deserialize untrusted data into live objects; parse with hardened, feature-minimal parsers |
| **Unsafe configuration** | Public buckets, debug endpoints in prod, permissive CORS, default credentials, verbose errors | Secure by default; config as reviewed code; environment parity checks |
| **Sensitive data exposure** | Secrets in code/logs, weak/homemade crypto, missing TLS, PII overexposure | Secrets external and rotated; boring standard crypto (argon2, AES-GCM, TLS); log scrubbing |

Per-instance teaching notes worth keeping:
- SQLi → why escaping is insufficient, parameterization as channel separation made concrete.
- XSS → encoding is *per context* (HTML body vs attribute vs JS vs URL); CSP as depth, not primary.
- SSRF → cloud metadata endpoints as the crown-jewel target; allowlists beat blocklists.
- RCE → deserialization, template injection, eval-family sinks.

## 4. Authentication, sessions, and identity

- Password storage: argon2id/bcrypt, why fast hashes (SHA-256) fail against offline cracking.
- Credential stuffing, MFA (and phishing-resistance ranking: SMS < TOTP < WebAuthn).
- Session management: cookie flags (HttpOnly, Secure, SameSite), rotation on privilege change, server-side revocation.
- **Identity lifecycle flows** — where real systems break, and generic curricula go silent:
  - Email verification: token entropy, expiry, single-use; verify before granting the email any authority.
  - Password reset: token handling, no user enumeration in responses, invalidate sessions on reset.
  - Account recovery: the backdoor that bypasses MFA if designed carelessly — recovery strength must match login strength.

## 5. Authorization

- The authn/authz distinction; authz checked at the *resource*, server-side, every time (complete mediation).
- IDOR/BOLA as the most common real-world API bug; BFLA (broken function-level authorization) as its verb-shaped sibling — can this *role* call this *function*, not just see this object.
- RBAC vs ABAC vs ReBAC (relationship-based, the Zanzibar model) — when role explosion signals you've outgrown RBAC.
- Deny by default; privilege escalation paths (horizontal and vertical).

## 6. Tokens and federation

- JWT: structure, alg confusion, why `none` existed, expiry/refresh strategy, storage tradeoffs (cookie vs localStorage — ties to XSS and CSRF families).
- OAuth 2.0 + PKCE: roles, flows, common misimplementations (redirect_uri validation, state parameter).
- OIDC on top of OAuth; ID token vs access token confusion as a recurring bug.

## 7. API security

- BOLA and BFLA (see §5) — the top two API killers in practice.
- Rate limiting (per-user, per-IP, per-endpoint) and pagination abuse (unbounded `limit`, offset scraping).
- Idempotency keys and replay attacks — why "retry safely" and "replay maliciously" are the same mechanism with different intent.
- Mass assignment; excessive data exposure (respond with a shaped DTO, never the raw model).
- Webhook security: verify HMAC signatures, timestamp windows against replay, don't trust source IP alone.
- Request signing (HMAC/AWS SigV4 style) for service consumers; API keys vs user tokens.
- API versioning and deprecating insecure endpoints (the forgotten v1 that still works).
- GraphQL: introspection exposure in prod, query depth/complexity limits, batching attacks (one request, thousand resolutions — rate limiting by request count fails), authz per-field not per-endpoint.
- gRPC: TLS by default, per-RPC auth interceptors, reflection service exposure.

## 8. Frontend security

Modern frontends carry their own attack surface — teach it as its own lesson, not a footnote to XSS.

- React XSS pitfalls: `dangerouslySetInnerHTML`, `href={userInput}` (`javascript:` URLs), props flowing into DOM sinks; why JSX auto-escaping protects text content only.
- DOM-based XSS: source→sink flows entirely in the browser (`location.hash` → `innerHTML`); invisible to server-side scanning.
- Hydration issues: server-rendered HTML mismatches and injection during hydration.
- CSP: as depth layer; nonce/hash-based over allowlists; report-only rollout.
- Trusted Types: making DOM sinks refuse raw strings — the browser-level fix for DOM XSS.
- iframe sandboxing and `postMessage` security: always verify `origin` on receive, always specify `targetOrigin` on send.
- Browser storage tradeoffs: localStorage (XSS-readable) vs cookies (CSRF-exposed but HttpOnly-protectable) vs in-memory.
- Service Workers: scope hijacking, cache poisoning, persistence after compromise.
- Browser extension risks: extensions read the DOM — never assume page content is confidential from the user's own environment.

## 9. Application-specific attack surfaces

Feature-shaped security most curricula skip; each is a lesson-sized topic when the user builds one:

- **File upload**: validate type by content not extension/Content-Type, size limits, store outside webroot with generated names, malware scanning; image processing risks (decompression bombs, ImageTragick-class parser exploits) — process in sandboxed workers.
- **PDF handling**: embedded JS, SSRF via remote resources during render, parser exploits — treat PDF parsing as untrusted-code territory.
- **Payment workflows**: never trust client-side amounts, server-side price computation, idempotency on charge creation, webhook verification for payment events, separation of duties for refunds.
- **Background jobs & scheduled tasks**: jobs run with high privilege and stale context — re-check authorization at execution time, not just enqueue time; poison-job handling.
- **WebSockets**: authenticate at handshake *and* re-authorize per message; origin checking; no ambient-cookie trust for cross-origin WS (CSWSH).

## 10. Database security

Many application vulnerabilities originate in the data layer:

- Parameterized queries (the non-negotiable baseline; ties to §3 injection family).
- Least-privilege DB roles: app user can't DROP, read-only replicas for reporting, separate migration credentials.
- Row-Level Security (RLS): pushing tenant isolation into the database so a forgotten WHERE clause can't leak across tenants.
- Transactions and isolation levels; race conditions as security bugs (double-spend via concurrent requests, TOCTOU on balance checks); locking strategies (`SELECT ... FOR UPDATE`) and unique constraints as the last line.
- Encryption at rest and in transit; what at-rest encryption actually protects against (stolen disks, not SQL injection).
- Backup security: backups are a full copy of your crown jewels with usually weaker controls — encrypt, access-limit, test restores.

## 11. Architecture-level security

Real-world incidents trace to architecture more often than to a single code snippet:

- Monolith vs microservices security: monolith has one perimeter and in-process trust; microservices multiply network surfaces and force explicit service identity.
- Service-to-service authentication: mTLS (mutual identity at connection level), signed service tokens (SPIFFE-style), why "it's on the internal network" is not authentication (Zero Trust).
- API Gateway patterns: centralized authn, rate limiting, and TLS termination — and the trap of gateways that check authn while services skip authz (complete mediation still applies per service).
- Reverse proxies: header spoofing (`X-Forwarded-For` trust), request smuggling at proxy/backend parsing mismatches.
- WAFs: useful depth layer, never the primary control; teach what they miss (logic flaws, BOLA — a WAF can't know your object ownership).
- CDN security: cache poisoning, cache-key design, what belongs in a shared cache (never per-user responses without proper keys).
- Network segmentation: blast-radius design — DB reachable only from app tier, egress restrictions (kills many SSRF escalations).
- Zero Trust architecture as the umbrella: identity-based access everywhere, continuous verification.

## 12. Infrastructure security

- Docker hardening: non-root user, read-only filesystem, minimal base images (distroless), no secrets in layers, `.dockerignore`.
- Kubernetes: RBAC (least privilege for service accounts), Secrets (and why etcd encryption + external secret stores matter), NetworkPolicies (default-deny), pod security standards.
- Cloud IAM: least privilege, no wildcard policies, role assumption over long-lived keys; security groups as instance-level firewalls.
- The metadata service (169.254.169.254) — ties back to SSRF; IMDSv2 as the mitigation.
- TLS everywhere: cert management, HSTS; DNS security (dangling CNAMEs → subdomain takeover, DNSSEC at awareness level).
- Shared-responsibility model: what the cloud provider secures vs what remains yours.

## 13. Secure DevOps and supply chain

The pipeline is itself a production system with production credentials — attackers know this:

- GitHub Actions security: untrusted `pull_request_target` input, script injection via branch/PR names, pinning actions to SHAs, `GITHUB_TOKEN` least privilege.
- Secret scanning (pre-commit and CI); what to do when a secret leaks (rotate first, then clean history — the secret is burned regardless).
- Dependency auditing and lockfiles; supply chain attacks (typosquatting, dependency confusion, hijacked maintainers).
- SBOMs: knowing what you ship so you can answer "are we affected?" in hours not weeks.
- SAST and DAST: what each catches, what each misses (SAST: taint-flow bugs, noisy; DAST: runtime issues, no logic flaws); neither replaces review.
- Artifact signing and provenance (Sigstore/SLSA at concept level): proving the artifact came from your pipeline.
- Branch protection, required reviews, and release pipelines as separation-of-duties controls.

## 14. Detection, monitoring, and response

Security is not just prevention — assume some control eventually fails:

- Logging for security: authn events, authz denials, sensitive-action audit trails; never log secrets/PII (scrubbing); structured logs so detection can query them.
- Audit trails: immutable, attributable ("who did what when"), covering admin actions above all.
- SIEM concepts: centralized log aggregation, correlation rules, why detection needs logs designed for it.
- Alerting: symptoms over noise — impossible-travel logins, authz-denial spikes, egress anomalies; alert fatigue as a real failure mode.
- Detection engineering: writing detections as code, testing them by simulating the attack behavior.
- Incident response basics: contain → eradicate → recover → post-mortem; why the plan must exist before the incident.
- Forensics basics: preserve evidence before rebuilding; logs/disk/memory as artifact sources.
- Threat intelligence at awareness level: known-bad indicators, feeds, and their limits.

## 15. OWASP cross-reference

Reference identifiers during reviews so the user builds industry vocabulary — but map each back to its §3 root-cause family:

Top 10 (2021): A01 Broken Access Control, A02 Cryptographic Failures, A03 Injection, A04 Insecure Design, A05 Security Misconfiguration, A06 Vulnerable Components, A07 Identification & Authentication Failures, A08 Software/Data Integrity Failures, A09 Logging & Monitoring Failures, A10 SSRF.

Also point to **OWASP API Security Top 10** when teaching §7 (BOLA is its #1 for a reason) and **OWASP ASVS** as the checklist for verifying a real application.

## 16. Teaching pattern for any vulnerability

1. Show the vulnerable code and have the user try to spot the flaw before revealing it.
2. Demonstrate the exploit conceptually (payload + what the attacker gains). Keep exploit detail educational, not operational.
3. Name the root-cause family (§3) — this is the step that generalizes.
4. Show the secure implementation and *why* it closes the hole; name the design principle (§2) it embodies.
5. Name the defense-in-depth layers that would have limited the damage anyway.
6. Exercise: a different code sample containing the same root cause in disguise.

## 17. Security code review framework

A repeatable pass order for reviewing any system. Walk it top to bottom — earlier items shape how you read later ones. For a quick review, do 1–8; for a full assessment, all 20.

1. **Trust boundaries** — draw them first; everything below is evaluated per boundary
2. **Input validation** — every entry point, validated against a positive model
3. **Output encoding** — per destination context (HTML, JS, SQL, shell, logs)
4. **Authentication** — strength, lifecycle flows (reset/recovery/verification), MFA
5. **Authorization** — per-resource, per-function, server-side, deny by default
6. **Secrets** — none in code/logs/history; storage, rotation, blast radius
7. **Cryptography** — standard algorithms, correct modes, key management
8. **Session handling** — cookie flags, rotation, revocation
9. **Database access** — parameterization, least-privilege roles, RLS, race conditions
10. **API exposure** — surface inventory, rate limits, pagination, versioned deprecations
11. **Logging** — security events captured, secrets/PII excluded, audit trail integrity
12. **Error handling** — fail secure, no internal leakage in responses
13. **Configuration** — secure defaults, prod/dev parity, no debug surfaces
14. **Dependencies** — audit status, lockfiles, update policy
15. **CI/CD** — pipeline permissions, secret handling, artifact integrity
16. **Infrastructure** — container/IAM/network posture (§12)
17. **Third-party integrations** — webhook verification, scoped credentials, data shared
18. **Privacy** — data minimization, retention, PII flows, regulatory exposure
19. **Performance implications of controls** — rate limits, crypto cost, logging volume (controls that fall over get disabled)
20. **Defense in depth** — for each critical asset, what holds when the primary control fails?
