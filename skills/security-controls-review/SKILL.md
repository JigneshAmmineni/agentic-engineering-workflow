---
name: security-controls-review
description: Threat-model and propose security controls for a whole app, or for one API, feature, or design still on paper. Researched against current primary sources, mapped to frameworks, with a tiered roadmap or security requirements. Use for security reviews, threat models, launch-readiness checks, which controls to add, or SOC 2/ISO/OWASP/prompt-injection/PII coverage questions.
---

# Security controls review

Output: ONE cited Markdown file in the repo telling the owner what threats matter and
what to build or fix, in what order, for THIS system or component. Not a generic
checklist: every threat is tied to an element you found in the code or design, every
"in place" claim points at a file, and every external claim cites a URL.

## 0. Where this runs: in a subagent, reporting back

This review reads half the repo and does live web research — it must not eat the
main conversation's context.

- **Invoked in the main conversation:** do NOT do the review inline. Spawn ONE
  general-purpose subagent whose prompt carries: the user's request verbatim, the
  repo/working directory, any files or design docs the user pointed at, and the
  instruction to follow this skill (name it) end to end. When it completes, relay its
  final report to the user: the saved file's path plus its ≤6-line summary, unaltered.
- **Already a subagent** (the spawned worker, or any other subagent context): do all
  the work yourself, inline. Never spawn further agents from inside this skill —
  including step 5's research: do it area by area, sequentially. (Nested research
  agents duplicate each other and burn tokens unseen.) Your final message IS the
  report hand-back: end with the saved path and the ≤6-line summary.

## 1. Pick the mode and scope

Read before asking: README, architecture/ADR docs, requirements, deploy configs
(Dockerfiles, compose, Terraform, CI workflows, proxy config), dependency manifests and
lockfiles, auth code, DB schema/migrations, `.env.example`, and any design doc the user
pointed at.

**Mode** — infer from the request; ask only if truly ambiguous:
- **System review** — the whole running (or nearly running) app. Output: threat model +
  tiered control roadmap with ✅/🟡/❌ status against the code.
- **Component review** — one API, feature, service, agent tool, or a design doc not yet
  built. Output: threat model + numbered security requirements with acceptance
  criteria, ready to paste into the project's spec. Existing controls the component
  will rely on (auth seam, RLS, rate limiter) are still verified against the code.

Then settle — infer first, ask at most 3 questions in one batch for what you can't tell:
- **Users and geography** (B2C/B2B; EU/UK/California/other; minors possible?)
- **Stage** (design / prototype / pre-launch / live / selling to enterprises)
- **Sensitive data** (health, payments, biometrics/voice, children, documents, location)
- **Prior output.** Look for an existing `security-controls*` or `threat-model*` doc. If
  one covers the same scope, UPDATE it (re-verify, add a dated changelog entry) — never
  a second file. A component review may cite the system review instead of repeating it.

## 2. Model the system (what are we working on?)

Draw a data-flow diagram in Mermaid for the scope: external entities (users, attackers'
entry points, third-party APIs), processes, data stores, data flows, and **trust
boundaries** (dashed subgraphs: internet→edge, edge→app, app→DB, app→each provider,
model context→tools). Label every flow with the data it carries.

Below it, an element table: element → type → exposure → data touched → notes (exact
versions of internet-facing frameworks and parsers of untrusted input; provider names).

Component mode: model the component plus one hop on each side (its callers and what
it calls), and mark which neighbors are trusted and why.

## 3. Threat-model it (what can go wrong?)

Follow the **Threat modeling method** section below. In short: STRIDE per element,
LINDDUN where personal data flows, the AI/agentic pass for any model component, and
abuse cases for each attacker persona. Rate each threat High/Medium/Low on likelihood ×
impact. Keep threats concrete ("attacker pre-registers victim's email, inherits account
on Google sign-in"), never category names alone.

## 4. Pick the research agenda

Select every profile in **App profiles** below that the element table matches —
usually several. Union their "look at" and "research" lists, add any threat from step 3
whose mitigation you can't state confidently, then pick frameworks from **Framework
landscape** marked relevant for the profiles and stage. Write the agenda as a bulleted
list at the top of your notes; it is your stopping criterion.

## 5. Research (primary sources, time-sensitive facts re-checked)

Work the agenda area by area (web/API, infra+supply chain, AI/agentic,
privacy/legal), inline and sequentially — per §0 you are running as the one subagent
and spawn nothing further. For each area carry the diagram, the threat table, that
slice of the agenda, and the source rules; collect findings with a URL per claim."

Source rules:
- Prefer the body that owns the fact: OWASP/NIST/CISA/MITRE pages, the regulator or
  statute text, the vendor's own docs/terms/pricing, NVD/GHSA/vendor advisories, the
  paper itself (not a blog about it).
- A secondary source is acceptable only when no primary exists; label it as such.
- **Always re-check, never recall:** CVEs affecting the exact versions you recorded;
  each data processor's current retention/training/ZDR terms; framework versions; law
  effective dates and thresholds. These go stale in months.
- When sources conflict, say so in the caveats instead of picking one silently.

## 6. Decide the responses (what are we going to do about it?)

For each threat: mitigate, eliminate (design it out), transfer (to a provider, with the
contract term cited), or accept (with the reason and an owner). Then:
- **System mode:** mark each control ✅ in place / 🟡 partial / ❌ gap / N/A by reading
  code or config, citing `path:line`. "The docs say so" is 🟡 at best — the code is the
  truth. Note controls better than typical; owners need to know what not to rip out.
- **Component mode:** write each response as a requirement `SEC-n: <the system shall…>`
  with an acceptance criterion that a test or review can check, plus the threat IDs it
  addresses. Flag design decisions the owner must make (with the options and trade-offs)
  separately from requirements. For any existing control the component reuses, verify
  and cite it as in system mode.

AI components: treat prompt injection as contained by design (least agency, output
controls, confirmation, data/instruction separation), with detection as one layer.

## 7. Write it up

Use the matching template in **Report templates** below. Rules:
- Lead with the 3–5 things that most reduce real risk, ordered by likelihood × impact,
  not framework order. Ordinary gaps (backups, patching, secrets, account takeover)
  usually outrank exotic ones; say so when true.
- System mode tiers: **T0 before launch / next**, **T1 ~90 days**, **T2 at scale or on
  customer demand**. Each row: control, status, effort (S/M/L), framework mapping.
- Map to frameworks for vocabulary, not as the organizing principle.
- Legal sections are statute readings, not legal advice — say so once.
- Save where the repo keeps docs (match convention; else `docs/security/`), tell the
  user the path, and give a ≤6-line summary in chat.

## Done when (did we do a good job?)

- Every element and every trust-boundary crossing in the diagram has at least one
  threat considered, or an explicit "nothing applies, because…".
- Every threat has a response: control/requirement, or accepted risk with a reason.
- Every agenda item is covered or listed under "not researched, why".
- Every ✅/🟡 cites a file; every external claim cites a URL; following two random
  citations lands on a primary source.
- System mode: the owner could pick next week's work from the T0 table alone.
  Component mode: the SEC requirements could be pasted into the spec and built against.

---

# Threat modeling method

Structure follows Shostack's four questions (steps 2, 3, 6, and "Done when" above).

## STRIDE per element

| Element type | Check for |
|---|---|
| External entity (user, provider, attacker-controlled input) | **S**poofing, **R**epudiation |
| Process (API route, worker, agent loop, tool) | all six: **S**poofing, **T**ampering, **R**epudiation, **I**nformation disclosure, **D**enial of service, **E**levation of privilege |
| Data store (DB table, bucket, cache, log, trace store, vector index) | **T**, **I**, **D** (+ **R** if it is an audit log) |
| Data flow (each arrow, especially across a trust boundary) | **T**, **I**, **D** |

Prompts that catch real bugs: Who can call this, and how is it proven? Can a caller
reach another user's object (BOLA)? What happens on 10,000 calls a minute, or one huge
input? What does this element log, and who reads the logs? What is the most privileged
thing it can do, and can untrusted input steer it there?

## LINDDUN (wherever personal data flows)

**L**inking, **I**dentifying, **N**on-repudiation (users unable to deny, when they
should be able to), **D**etecting, **D**ata disclosure, **U**nawareness and
unintervenability (users can't see, correct, export, or delete their data), **N**on-
compliance. Walk each flow carrying personal data, and each store holding it — including
logs, traces, backups, and every third-party processor.

## AI and agentic pass (any model component)

- **Lethal trifecta check:** does one agent context combine (1) private data, (2)
  untrusted content, and (3) a way to send data out (links, tools, network, rendered
  output)? If all three, exfiltration via injection is possible — break one leg.
- Map every source of text entering model context (user turns, uploads, retrieved
  documents, tool results, memory, web pages, transcripts) and the role it lands in.
- For each tool: read vs. write vs. irreversible; whose identity it runs as; what
  confirmation it needs; whether it can be undone.
- Walk OWASP Top 10 for LLM Applications and OWASP Top 10 for Agentic Applications
  against the diagram; for multi-agent or heavily autonomous systems also use OWASP's
  Agentic AI Threats and Mitigations guide or CSA MAESTRO (layered agentic threat model).
- Voice/audio: anything the STT transcribes is untrusted input.

## Abuse cases (attacker personas)

Write at least one concrete abuse case per persona that applies: anonymous internet
user; authenticated malicious user; compromised user account; author of malicious
content the system ingests (indirect injection, malicious file); cost/quota abuser
(denial of wallet); malicious or compromised dependency; compromised CI/deploy
credential or operator account.

## Rating

High/Medium/Low on likelihood × impact, one sentence of reasoning each. Don't use
numeric scoring schemes; they imply precision you don't have.

---

# API checklist

Walk this for every route in scope (system mode: sample the riskiest routes and any
pattern shared by all; component mode: every route of the component). Each line is a
question with the standard answer; a "no" is a threat for step 3.

**OWASP API Top 10 (2023)**
- **API1 BOLA:** every query taking an id is scoped by the caller's identity/tenant from
  the verified token (never from the request); a DB backstop (RLS) exists; a cross-user
  CI test calls every route as user A with user B's ids and expects 404. Writes too:
  a credential bound to one tenant/store cannot write for another.
- **API2 authentication:** tokens fully validated (signature, issuer, audience, expiry,
  revocation where needed); endpoints reachable before auth (login, signup, enrollment,
  reset, availability checks) are rate limited; no secrets in URLs or query strings.
- **API3 property-level authz (BOPLA):** separate input and output schemas per
  resource; unknown fields rejected (`extra="forbid"` or equivalent); persistence models
  never on the wire in either direction; server-owned fields (owner, role, quota,
  status) never bindable.
- **API4 resource consumption:** see Rate limits and bounds below.
- **API5 function-level authz:** privileged routes behind a role check at the router or
  method level so new routes inherit it; default deny.
- **API6 sensitive business flows:** "what if this legitimate flow is automated a
  million times?" asked and answered (per-account flow limits, quotas, enumeration
  monitoring, or a product decision to reveal less).
- **API7 SSRF:** any user-influenced URL is fetched via destination checks (resolve,
  reject private/loopback/link-local incl. 169.254.169.254, connect to the checked IP),
  HTTPS only, no redirects, ideally through an egress proxy.
- **API8 misconfiguration:** API docs/OpenAPI, debug routes, stack traces, permissive
  CORS off in production; generic error bodies.
- **API9 inventory:** the committed OpenAPI file is the inventory; a CI test fails on
  shadow routes (reachable, undocumented — e.g. hidden from schema) and on any route
  diff not in the PR; zombie versions have retirement dates and per-route traffic data;
  every hostname/gateway fronting the API is listed.
- **API10 unsafe consumption:** responses from third-party APIs (partners, AI/STT/TTS
  providers) are untrusted input: schema-validated, size-bounded, timeouts on every
  call, TLS verified, redirects not followed, circuit breaker so their outage isn't yours.

**Rate limits and bounds**
- **Key:** named explicitly. Per IP only before auth (NAT shares IPs; attackers rent
  many). After auth, per user/device/API key (can't rotate cheaply). Per tenant as an
  outer ceiling (noisy neighbor). Stack them; never key on a value the client picks.
- **Algorithm:** named explicitly. Fixed window lets ~2× the limit through at the
  boundary; sliding window or token bucket (bursts up to capacity, sustained rate) is
  the default for API traffic.
- **Where:** edge/gateway for volume, app for meaning (per-function limits). With more
  than one replica, counters are shared (e.g. Redis, atomic update); an in-memory limiter
  is correct only with one worker. The limiter's own per-key state is bounded, or it is
  an API4 hole itself. A gateway that reads token claims for keying validates the token
  first. Distributed limits are approximate — size with slack.
- **Response:** 429 with `Retry-After` (optionally the IETF `RateLimit`/
  `RateLimit-Policy` headers); clients back off exponentially with jitter.
- **Quotas:** rate limits miss the slow drip just under the limit; daily/monthly quotas
  cap total spend. Expensive routes (model inference, SMS, paid third-party calls) get
  their own tighter limits and per-account quotas; heavy work is queued with a per-tenant
  concurrency cap; provider spend caps and billing alerts back it up.
- **Bounds:** server-enforced max page size (client's value is a request); body size at
  the edge and in the app; array and string length caps in schemas; DB query timeouts;
  cursor pagination for large sets. GraphQL: depth limit, cost analysis, batch limits
  (a per-request limit counts a 500-operation batch as one).
- **Idempotency:** retried writes (batches, payments, jobs) carry an idempotency key
  and are processed once.

**API keys** (if issued)
- Recognizable prefix (enables secret-scanning custom patterns and log redaction),
  ≥256 random bits, stored only as a hash (fast SHA-256 is fine for random keys), shown
  once; scoped to one tenant and a permission set; expiring; two active keys for
  zero-downtime rotation; last-used tracking and cleanup of unused keys; sent in a
  header, never a URL; per-key limits and logs so a leak's blast radius is knowable.

**Webhooks — receiving**
- In order: read the raw body before parsing; reject timestamps outside a tolerance
  (~5 min); HMAC over id/timestamp + raw body, constant-time compare against each
  expected-version signature (supports rotation, ignores unknown schemes); atomically
  record the event id and skip repeats; ack fast, process asynchronously. Events may
  arrive out of order.

**Webhooks — sending**
- Per-endpoint signing secret (prefixed); unique event id + timestamp; retries with
  exponential backoff and jitter over a bounded period; disable endpoints that keep
  failing and notify the owner; HTTPS only; redirects count as failure; minimal payloads
  (id + type, receiver fetches through the authenticated API); fan out only to the
  event owner's endpoints (API1); destination checked at **send** time, not just
  registration (DNS can change); delivery workers behind an egress proxy in a network
  segment that can't reach internal services.

**Compromised-credential containment**
- Assume an attacker holds one valid token or key. Is it bound to the narrowest scope
  (one user/tenant/device)? Does per-credential volume get compared to its baseline?
  Can that one credential be revoked without affecting others?

**Monitoring**
- Alerts on 429 rate per key, per-credential volume vs. baseline, auth-failure spikes,
  and spend on expensive routes and providers — abuse is detected, not only blocked.

**Acceptance-test patterns** (component mode — make SEC criteria checkable)
- N+1 requests from one caller at once → the last gets 429 with `Retry-After`; a second
  caller is unaffected.
- Cross-user/tenant call with another owner's id → 404.
- Unknown or server-owned field in a body → 422; output never contains internal columns.
- Webhook with a timestamp just past tolerance → rejected; same event id twice →
  processed once.
- Route hidden from the schema, or added without updating the committed OpenAPI → CI
  fails.
- Issued key appears raw only in the issue response; storage and logs hold the hash only.

---

# App profiles — where to look

Match every profile the element table triggers. Each lists what to inspect and what to
research.

**Every app (baseline — always include).** Look at: dependency manifests + lockfiles
(exact versions of internet-facing frameworks), CI workflows (secret handling, action
pinning, who can deploy), secrets storage, backup/restore, logging config (what content
reaches logs), admin/operator accounts (MFA?), domain/DNS registrar account. Research:
CVEs/advisories for the recorded versions (NVD, GHSA, vendor); OWASP Top 10; CIS
Controls IG1; breach-notification duties for the user geography.

**B2C / consumer accounts** (open signup, individual users). Look at: signup/login,
email verification, password policy, account linking across providers
(pre-hijacking), enumeration responses, rate limits, account deletion + export, age
gate, privacy policy/terms. Research: NIST SP 800-63B-4; the identity provider's own
abuse/bot protection docs; GDPR/CCPA/state privacy rights; COPPA if minors plausible;
credential-stuffing defenses.

**B2B / multi-tenant SaaS** (orgs, roles, invites, SSO). Look at: tenant isolation
(RLS, tenant_id filters, separate DBs), role model + function-level checks, invites,
SSO/SCIM, customer-facing audit logs, data residency. Research: OWASP API1/API5; SOC 2
/ ISO 27001 / CSA STAR expectations; SAML/OIDC pitfalls; customer DPA expectations.

**Any HTTP API** (every app with routes — first-party frontend calls count; this is
not just for public APIs). Look at: every route × method (compare the live OpenAPI /
route table with what's documented), how each id in a path or body is scoped to the
caller, input vs. output schemas, every client-supplied number (page size, limits,
array/string lengths), rate limiter implementation and key, timeouts, outbound calls to
third-party APIs (including AI providers), docs/debug endpoints and CORS in prod. Walk
the **API checklist** section below for each route in scope. Research: OWASP API
Security Top 10 (2023) category pages; NIST SP 800-228; OWASP REST Security and SSRF
cheat sheets; the framework's own auth/validation docs.

**Public API / webhooks / API keys** (third parties call you, keys, webhooks in or out).
Look at: key format, storage, scoping, expiry, rotation, last-used tracking; webhook
receive path (signature, timestamp, dedupe) and send path (delivery worker, egress,
retries); per-key and per-tenant limits and quotas; API version retirement. Research:
Standard Webhooks spec; the reference provider's webhook docs (e.g. Stripe); GitHub's
token-format and secret-scanning docs (custom patterns); IETF RateLimit header draft;
the gateway's rate-limit docs if one is used.

**LLM features** (any model call with user input). Look at: system prompt (secrets or
authorization logic in it?), every source of untrusted text entering context and its
role, how output is rendered (markdown → HTML? images? links?), per-user token/cost
limits, provider data settings in code, where prompts/outputs are logged or traced.
Research: OWASP Top 10 for LLM Applications; the provider's CURRENT retention,
training-use, ZDR, and spend-cap terms; spotlighting/datamarking; markdown
image/link exfiltration; guardrail options (provider classifiers, Google Model Armor,
Lakera, Prompt Guard; Presidio for PII); red-team tools (promptfoo, garak, PyRIT).

**Agentic** (tools that read/write data, send messages, browse, execute code; memory).
Look at: tool registry (read/write class and blast radius per tool), identity tools run
as, confirmation gates for destructive or irreversible actions, undo/versioning,
per-turn tool budgets, memory write path and provenance, code-execution sandboxing.
Research: OWASP Top 10 for Agentic Applications ("least agency"); CaMeL / dual-LLM
patterns; AgentDojo-style evaluation; memory poisoning.

**Voice / audio / realtime media** (mic, STT/TTS, WebRTC, telephony). Look at:
signaling auth, media port exposure, TURN credentials, concurrent-session caps, whether
raw audio is stored, speaker ID / voice embeddings / cloning, mic-state UX, STT/TTS
providers' data terms. Research: WebRTC security (DTLS-SRTP, ICE exposure, TURN
credential lifetime); audio prompt-injection research; biometric laws (does the app
create a voiceprint?); TTS providers' voice-consent rules.

**File uploads / document processing.** Look at: size/type validation (magic bytes),
parser libraries + versions (PDF, Office, image, archive), parsing isolation
(subprocess, memory/CPU limits, timeout), storage location, how files are served back
(Content-Disposition, separate origin), malware scanning need. Research: parser CVEs;
OWASP File Upload cheat sheet; decompression-bomb limits.

**Rich client rendering** (markdown/HTML, editors, user-generated content). Look at:
renderer config (raw HTML? sanitizer?), link-scheme allowlist, image loading, CSP and
security headers, cookie flags. Research: OWASP XSS and CSP cheat sheets; Trusted Types;
sanitizer library advisories.

**Self-hosted infra** (you manage OS, container host, firewall). Look at: firewall rules,
SSH exposure, container user/capabilities/network mode, OS patching, disk encryption,
snapshot/backup schedule, monitoring. Research: CIS Benchmarks for the OS, container
runtime, and cloud account (e.g. Debian/Ubuntu, Docker, Kubernetes, GCP/AWS/Azure
Foundations); cloud VM hardening (Shielded VM, OS Login, metadata-service protection);
managed secret stores.

**Serverless / PaaS / managed** (Vercel, Cloud Run, Lambda, Supabase, Firebase). Look
at: function IAM roles, env-var secrets, preview-deployment exposure, database security
rules (Firestore rules, Supabase RLS), public buckets. Research: the platform's
shared-responsibility docs; platform-specific advisories; CIS Benchmark for the cloud.

**Mobile apps.** Look at: local token storage, certificate pinning, deep links,
embedded keys, app attestation. Research: OWASP MASVS/MASTG; Play Integrity / App
Attest; store privacy labels.

**Payments / money movement.** Look at: whether card data touches your servers, webhook
verification, idempotency, refund/payout authorization. Research: PCI DSS 4.0.1 (which
SAQ); the processor's integration security guide; fraud controls.

**Sensitive data classes.** Health: HIPAA (BAAs with every processor), Washington My
Health My Data. Children: COPPA, age-appropriate design codes. Biometrics: Illinois
BIPA, Texas CUBI, Colorado, Washington — consent before any template creation.
Precise location: state "sensitive data" opt-in rules. Financial: GLBA Safeguards. Look
at where that data is stored, which processors receive it, and retention.

**Geography.** EU/UK users: GDPR (lawful basis, DPIA, DPAs, transfers, 72h breach
notice), EU AI Act (Art. 50 disclosure if AI), NIS2/DORA if the sector applies.
California / other US states: CCPA regulations + a current state-law tracker.
Elsewhere: the local data-protection authority's site.

---

# Framework landscape

Snapshot as of 2026-10 — versions and dates move; re-check each one you cite (step 5).

**Engineering checklists (almost always relevant)**
- OWASP Top 10 (2025) — web risk awareness. owasp.org/Top10
- OWASP API Security Top 10 (2023) — any app exposing an API. owasp.org/API-Security
- OWASP ASVS 5.0 — testable requirements; L1 for launch, L2 for auth/data; the best
  source for component-mode SEC requirements. owasp.org
- OWASP MASVS/MASTG — mobile. mas.owasp.org
- OWASP Cheat Sheet Series — implementation guidance per topic. cheatsheetseries.owasp.org
- CWE Top 25 — prioritizing code-level weakness classes. cwe.mitre.org
- NIST SP 800-228 — API protection for cloud-native systems (pre-deployment and
  runtime controls); the reference when someone asks for an API protection standard.
- Standard Webhooks spec — signing and header conventions for webhooks. standardwebhooks.com
- IETF RateLimit header fields (Internet-Draft, httpapi WG) — syntax may still change.

**AI-specific (any LLM/ML component)**
- OWASP Top 10 for LLM Applications (2025) and Top 10 for Agentic Applications (2026);
  OWASP Agentic AI Threats and Mitigations guide. genai.owasp.org
- MITRE ATLAS — attacks on AI systems; threat naming and red-team scenarios. atlas.mitre.org
- NIST AI RMF 1.0 + AI 600-1 (GenAI Profile) — lightweight AI governance vocabulary. nist.gov
- Google SAIF — secure AI design, natural fit on Google cloud. saif.google
- CSA MAESTRO — layered threat modeling for agentic/multi-agent systems. cloudsecurityalliance.org
- ISO/IEC 42001 — certifiable AI management system; only if enterprise buyers ask.
- CSA AI Controls Matrix — AI control catalog for vendor questionnaires.
- EU AI Act — Art. 50 transparency (tell users they talk to an AI) applies since
  2026-08-02; high-risk obligations moved to 2027-12. eur-lex.europa.eu

**Threat modeling**
- Shostack's four-question frame and STRIDE; Threat Modeling Manifesto.
  threatmodelingmanifesto.org
- LINDDUN (privacy threats). linddun.org
- OWASP Threat Modeling cheat sheet / Threat Dragon (diagramming tool).

**Operational / organizational baselines**
- CIS Controls v8.1 (IG1→IG3) — what to do operationally; IG1 for small teams. cisecurity.org/controls
- CIS Benchmarks — how: hardening settings per OS, container runtime, Kubernetes, cloud
  account. cisecurity.org/cis-benchmarks
- NIST CSF 2.0 — Govern/Identify/Protect/Detect/Respond/Recover structure. nist.gov/cyberframework
- NIST SP 800-63B-4 — authentication rules (password length, no forced rotation,
  breached-password screening, phishing-resistant option at AAL2).
- NIST SP 800-53 r5 — exhaustive catalog; reference only.
- CISA Secure by Design — MFA by default, eliminate vuln classes, VDP. cisa.gov/securebydesign
- MITRE ATT&CK — adversary techniques for detection and IR. attack.mitre.org
- MITRE D3FEND — defensive techniques for mapping countermeasures. d3fend.mitre.org

**Supply chain**
- SLSA (build provenance) · OpenSSF Scorecard (repo hygiene) · NIST SSDF SP 800-218 ·
  SBOM (CycloneDX/SPDX) when customers or regulators ask.

**Attestations / certifications (only on customer demand)**
- SOC 2 (AICPA Trust Services Criteria) — US B2B. SOC 3 is its public summary; SOC 1
  covers controls over customers' financial reporting (rarely relevant to app startups).
- ISO/IEC 27001:2022 (Annex A) — international/enterprise B2B.
- CSA STAR — Level 1 self-assessment (CAIQ questionnaire, public registry), Level 2
  third-party audit; built on the CSA Cloud Controls Matrix.
- FedRAMP / StateRAMP — US government.

**Sector / data-type regimes (only if triggered)**
- PCI DSS 4.0.1 (card data) · HIPAA (US PHI) · COPPA (under-13) · FERPA (student
  records) · GLBA Safeguards (US financial) · DORA / NIS2 (EU sectors) · biometric laws
  (BIPA, CUBI, Washington, Colorado) · EU Cyber Resilience Act (products with digital
  elements sold in the EU).

**Privacy (any personal data)**
- GDPR / UK GDPR · CCPA/CPRA + current regulations · other US state comprehensive
  privacy laws (use a current tracker) · breach-notification laws (always — goes in the
  IR plan).

**Default selection by stage**
- Design / prototype / pre-launch, small team: OWASP Top 10 + API Top 10 + ASVS L1
  (+ LLM/Agentic Top 10 if AI), CIS Controls IG1 + the relevant CIS Benchmarks if
  self-hosting, 800-63B for auth, privacy law for user geography, AI Act Art. 50 if EU +
  AI. Skip certifications.
- Live, growing: add ASVS L2, NIST CSF structure, SLSA L1–2, Scorecard, DPIA/risk
  assessments, vulnerability disclosure policy.
- Selling to enterprises: add SOC 2 / ISO 27001 / CSA STAR mapping, vendor
  questionnaires (CAIQ/AICM), SBOM, ISO 42001 if AI-heavy buyers.

---

# Report templates

## System mode

```
# Security controls review — <app>
_Run: YYYY-MM-DD · Mode: system · Stage: … · Users: …_
_Legal sections are statute readings, not legal advice._

## Summary            — 3–5 highest-value actions + one line on what's already strong
## System model       — Mermaid DFD with trust boundaries; element table
## Threat model       — table: ID | Element | STRIDE/LINDDUN/AI | Threat | Rating
                        + abuse cases by persona
## Profiles and frameworks applied — one line each on why; exclusions and why
## Threat catalog and controls — per area (web/API · infra & supply chain · identity ·
                        AI/LLM · agentic · voice/media · privacy):
                        Threat IDs | Mitigation | Status ✅🟡❌ N/A | Evidence (path:line / URL)
## Privacy and legal  — regimes that bind, obligations, triggers to avoid
## Roadmap            — T0 / T1 / T2 tables: # | Control | Threat IDs | Status | Effort | Maps to
## Suggested order of work — week-by-week for T0
## Accepted risks     — risk | reason | owner | revisit date
## Not researched     — item | why
## Caveats            — conflicting sources, preprint-only evidence, moving versions
## Changelog          — dated entries
## Sources            — numbered URLs; mark secondary sources
```

## Component mode

```
# Security review — <component / API / feature>
_Run: YYYY-MM-DD · Mode: component · Design doc: <link> · System review: <link if any>_

## Summary            — top threats and the 3–5 requirements that matter most
## Component model    — Mermaid DFD: component + one hop each side, trust boundaries;
                        element table; trusted neighbors and why
## Threat model       — table: ID | Element | STRIDE/LINDDUN/AI | Threat | Rating
                        + abuse cases by persona
## Security requirements — SEC-n | Requirement ("the component shall…") |
                        Acceptance criterion | Threat IDs | Maps to
## Design decisions for the owner — decision | options | trade-offs | recommendation
## Reused controls    — control | where it lives (path:line) | verified? | gaps it leaves
## Accepted risks     — risk | reason | owner
## Not researched / Caveats / Changelog / Sources — as in system mode
```

Effort: S < 1 day · M 1–3 days · L 1–2 weeks.
