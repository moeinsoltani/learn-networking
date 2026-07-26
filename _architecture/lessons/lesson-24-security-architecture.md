---
title: "Lesson 24 — Security Architecture & Threat Modeling"
nav_order: 2
parent: "Phase 6: Cross-Cutting Quality Attributes"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 24: Security Architecture & Threat Modeling

{: .note }
> **Words to know**
> - **defense in depth** — multiple independent layers of security, so one failure doesn't breach everything.
> - **least privilege** — every component/user gets the *minimum* access needed, nothing more.
> - **trust boundary** — a line where the level of trust changes (e.g., the internet ↔ your system); where you must authenticate/validate.
> - **threat modeling** — systematically asking "what can go wrong here?" during design.
> - **STRIDE** — a threat taxonomy: Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege.
> - **zero trust** — "never trust the network"; verify every request regardless of where it comes from.
> - **blast radius / segmentation** — how far an attacker gets after a breach, and how you limit it.

## Concept

Security is not a feature you add at the end — it's a **quality attribute of the structure** that
must be designed in, because the most important security decisions are *architectural* (where the
trust boundaries are, how components are segmented, where authentication and authorization happen)
and are painful or impossible to bolt on later. The architect's non-negotiable duty is to build
security *into* the design, not around it.

```
   SECURITY IS STRUCTURAL — it lives in the boundaries

   INTERNET (untrusted)
        │  ── TRUST BOUNDARY ── authenticate, validate, rate-limit HERE
        ▼
   [ API Gateway / edge ]  ← authN, TLS termination, WAF, rate limiting
        │  ── another boundary (don't trust "internal" blindly — zero trust) ──
        ▼
   [ Services ]  ← each authorizes; least privilege; mTLS between them
        │  ── boundary around sensitive data ──
        ▼
   [ Payment / PII store ]  ← tightest controls, smallest access, encrypted,
                              segmented so a breach elsewhere can't reach it

   DEFENSE IN DEPTH: many layers, so one failure ≠ total breach.
   LEAST PRIVILEGE: each box gets the minimum access it needs.
   BLAST RADIUS: design so a breach of one box can't reach everything.
```

Two principles anchor everything: **defense in depth** (never rely on a single control — layer
independent defenses so one failure isn't catastrophic) and **least privilege** (every component,
service, and credential gets the minimum access it needs — so a compromised component can do
limited damage). Both are fundamentally about *limiting the blast radius* of the breach you should
assume will eventually happen.

## Going Deeper

**Trust boundaries are an architectural concept.** A **trust boundary** is a line where trust
changes — the internet meeting your system, a user-facing service calling an internal one, your
code calling a third party. At every trust boundary you must *authenticate* (who is this?),
*authorize* (are they allowed?), and *validate* (is this input safe?), because you can't trust what
comes from the other side. Drawing the trust boundaries is a design activity: it tells you where the
security controls go. The classic mistake is assuming everything "inside" is trusted (the soft
chewy center) — which is exactly what zero trust rejects.

**Threat modeling — "what can go wrong here?" as a design step.** Threat modeling is
systematically examining a design for how it could be attacked, *before* building it. The most
common lightweight framework is **STRIDE** — for each component and data flow, ask whether it's
vulnerable to: **S**poofing (pretending to be someone else), **T**ampering (altering data),
**R**epudiation (denying an action with no proof), **I**nformation disclosure (leaking data),
**D**enial of service (overwhelming it), **E**levation of privilege (gaining rights you shouldn't).
You don't need heavyweight ceremony — even a quick pass over a data-flow diagram asking "how would
I attack each boundary?" surfaces the risks worth designing against. Threat modeling is to security
what the pre-mortem is to project risk: find the failure before it finds you.

{: .note }
> **Where do authentication and authorization go? (An architectural decision)**
> A recurring design question: do you authenticate/authorize at the <em>gateway</em> (the edge),
> at each <em>service</em>, or both? The mature answer is usually <strong>both, at different
> levels</strong>: the gateway handles coarse authentication (is this a valid, logged-in
> request? — terminate TLS, validate the token, rate-limit) so unauthenticated traffic never
> reaches the services; and each service handles fine-grained <strong>authorization</strong> (is
> <em>this</em> user allowed to do <em>this specific</em> action on <em>this</em> resource?),
> because only the service knows its own rules and — critically — you should <em>not</em> assume a
> request is safe just because it came from "inside" (zero trust). Centralizing all authorization at
> the gateway is fragile (the gateway can't know every service's rules, and a bypass breaches
> everything); pushing all authentication into every service is redundant and error-prone. Split by
> level: coarse authN at the edge, fine-grained authZ at the service. (The mechanics — tokens, JWT,
> OAuth, mTLS — are the <a href="{{ '/security/learning-plan.html' | relative_url }}">Security &
> Identity</a> track; here it's <em>where</em> in the architecture they belong.)

**Zero trust — never trust the network.** The old model was "perimeter security": a hard outer wall
(firewall), and everything inside trusted. It fails because once an attacker is inside (a breached
service, a phished credential, a malicious insider), they have free rein — the soft center. **Zero
trust** replaces "trust based on network location" with "verify every request, every time,
regardless of origin": internal service-to-service calls are authenticated and authorized (often
via mTLS and per-request tokens) just like external ones; nothing is trusted merely for being "on
the internal network" (fallacy 4, Lesson 14). Architecturally this means every service verifies its
callers, sensitive data is segmented behind its own controls, and a breach of one component doesn't
grant access to others. It's defense in depth applied to the network trust assumption.

**Design the blast radius: segmentation and isolation.** Assume breach — then design so a breach is
*contained*. This is segmentation: put the most sensitive assets (payment data, PII, secrets) behind
their own tight boundaries with minimal access, so compromising a front-end service doesn't reach
the crown jewels. Isolate by sensitivity (a payment service with a tiny attack surface and its own
data store, Lesson 13), by tenant (one tenant's breach can't touch another's data), and by
environment. Secrets and key management are themselves architecture — how secrets are stored,
rotated, and accessed (a secrets manager, not config files or code) is a structural decision, not
an afterthought. The unifying question, per asset: "if an attacker gets *here*, how far can they go,
and what stops them?" — the same blast-radius thinking as resilience (Lesson 18), applied to
attackers instead of failures.

**Security is a quality attribute that trades against others.** Like every -ility (Lesson 3),
security conflicts with others — every auth check adds latency and friction (usability), encryption
adds CPU cost, segmentation adds complexity. So it's a trade-off to calibrate to the threat, not a
dial to max out blindly: a public banking API and an internal analytics dashboard warrant very
different levels. But note the asymmetry: under-investing in security has *catastrophic, sometimes
existential* downside (a breach can end a company), so the calibration errs toward caution for
anything touching sensitive data or the internet — and the *architectural* controls (trust
boundaries, segmentation, least privilege) are cheap to design in and ruinously expensive to
retrofit, so you build them from the start.

---

## Lab — Design Exercise

**The situation:** You're designing a system with: a **public web/mobile API** (used by customers
over the internet), a set of **internal microservices** (orders, catalog, user profiles), a
**payment integration** with a third-party provider (Stripe), and a **database holding customer PII
and order history**. The team's current plan: a firewall at the edge, and "everything behind the
firewall trusts everything else" (services call each other freely over plain HTTP, one shared
database, secrets in a config file).

**Redesign the security architecture.** Draw the **trust boundaries**, apply defense-in-depth,
least-privilege, and zero-trust, decide where authN and authZ go, segment to limit the blast
radius, and run a quick **STRIDE** pass on the riskiest boundary. Name the biggest weaknesses in
the current plan.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is thinking in trust boundaries and blast radius, not a single wall. A strong answer:
<br><br>
<strong>Biggest weaknesses in the current plan:</strong> it's the classic "hard shell, soft
center" (perimeter-only) design that violates every principle here. (1) <em>"Everything behind the
firewall trusts everything else"</em> — once <em>any</em> component is breached (a vulnerable public
API, a phished credential), the attacker has free rein over all services and all data (no
segmentation, no zero trust, huge blast radius). (2) <em>Plain HTTP between services</em> — fallacy
4 (the internal network isn't secure); traffic including PII is sniffable/tamperable by anyone who
gets inside. (3) <em>One shared database</em> — PII, orders, and everything else in one store with
one set of credentials means any service's compromise exposes <em>all</em> data. (4) <em>Secrets in
a config file</em> — a leaked repo or a breached host hands over every credential. (5) No mention of
authZ at services (only a perimeter firewall), so a request that gets past the edge is trusted for
anything.
<br><br>
<strong>The redesign — trust boundaries + defense in depth + zero trust:</strong>
<ul>
<li><strong>Edge / trust boundary 1 (internet ↔ system):</strong> an API gateway that terminates
TLS, authenticates every request (validates the session/token), rate-limits (DoS defense), and runs
a WAF. Unauthenticated/abusive traffic never reaches the services. This is the coarse authN layer.</li>
<li><strong>Service layer / zero trust:</strong> services do <em>not</em> trust each other by
network location. Service-to-service calls use <strong>mTLS</strong> (mutual auth + encryption in
transit — fixing the plain-HTTP flaw) and carry per-request identity/tokens, and each service does
its own fine-grained <strong>authorization</strong> ("is this user allowed to do this action on this
resource?"). So a breached catalog service can't freely command the orders or payment service — it
must still authenticate and be authorized.</li>
<li><strong>Data segmentation (blast radius):</strong> split the one shared database by sensitivity/
ownership (Lesson 13/24 data ownership). PII lives in a store behind its own tight boundary with
<em>least-privilege</em> access (only the user-profile service can touch it, not every service),
encrypted at rest. Order data separately. Now a compromise of the catalog service reaches
<em>catalog</em> data, not the PII crown jewels.</li>
<li><strong>Payment isolation:</strong> the payment path is the most sensitive — isolate it (its own
service, minimal attack surface, its own tightly-controlled boundary) and, crucially,
<em>tokenize</em> — never let raw card data flow through your general services or storage; hand
payment to Stripe and store only tokens, shrinking your PCI scope and blast radius dramatically.</li>
<li><strong>Secrets:</strong> move secrets out of config files into a <strong>secrets manager</strong>
(with rotation and least-privilege access), so a leaked repo/host doesn't hand over credentials, and
each service gets only the secrets it needs.</li>
<li><strong>Least privilege throughout:</strong> each service's DB credentials, cloud IAM roles, and
network access are scoped to the minimum it needs — so a compromised service's reach is bounded.</li>
</ul>
<strong>STRIDE pass on the riskiest boundary — the public API edge (internet ↔ system):</strong>
<ul>
<li><em>Spoofing:</em> attacker impersonates a user → mitigate with strong authentication (tokens/
sessions, MFA for sensitive actions), validate tokens at the gateway.</li>
<li><em>Tampering:</em> attacker alters requests/data in transit → TLS everywhere (in transit),
integrity checks, server-side validation (never trust client-supplied authorization fields like
"isAdmin").</li>
<li><em>Repudiation:</em> user denies an action → audit logging of security-relevant actions with
correlation IDs (Lesson 25), so there's a trail.</li>
<li><em>Information disclosure:</em> leaking PII → encrypt in transit and at rest, least-privilege
data access, don't over-return data in API responses, segment PII.</li>
<li><em>Denial of service:</em> flooding the API → rate limiting, throttling, and load shedding at
the gateway (Lesson 18); autoscaling with caps.</li>
<li><em>Elevation of privilege:</em> a normal user gaining admin rights → fine-grained authZ checked
<em>server-side</em> at each service (never trust client claims), least privilege, careful handling
of the token→permissions mapping.</li>
</ul>
<strong>The meta-lesson:</strong> the fix isn't "a bigger firewall" — it's replacing perimeter trust
with <em>layered, structural</em> security: trust boundaries where trust changes (with authN/authZ/
validation at each), zero trust between services (mTLS + per-request authZ, no trusting the internal
network), segmentation so a breach is <em>contained</em> (PII and payment isolated behind their own
tight boundaries with least privilege), secrets managed properly, and a threat-model pass (STRIDE)
to find the specific attacks per boundary. All of this is <em>architectural</em> — designed into the
structure — and cheap to build in now versus ruinous to retrofit after a breach, which is exactly
why security is a first-class quality attribute the architect owns from the start, not a feature
bolted on at the end.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Threat modeling & STRIDE | <https://en.wikipedia.org/wiki/STRIDE_model> ; *Threat Modeling*, Adam Shostack |
| Zero trust architecture | NIST SP 800-207 — <https://csrc.nist.gov/pubs/sp/800/207/final> |
| Defense in depth & least privilege | OWASP — <https://owasp.org/www-community/> |
| The mechanics (authN/authZ, tokens, mTLS) | [Security & Identity track]({{ '/security/learning-plan.html' | relative_url }}) |
| OWASP Top 10 (common app vulnerabilities) | <https://owasp.org/www-project-top-ten/> |

---

## Checkpoint

**Q1.** Why is "a firewall at the edge, and everything inside trusts everything else" a dangerous
architecture, and how do zero trust and segmentation fix it? Frame the answer in terms of blast
radius.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The "hard shell, soft center" (perimeter-only) model is dangerous because it puts <em>all</em> its
security in a single boundary and assumes everything past it is safe — so the moment <em>anything</em>
gets inside, there are no further defenses. And things <em>will</em> get inside: a vulnerability in a
public-facing service, a phished or stolen credential, a compromised dependency, a malicious
insider, or a misconfiguration. Once an attacker is past the firewall, the "everything trusts
everything" design gives them free rein — they can call any service, reach any data, move laterally
across the whole system unchallenged. In blast-radius terms, the blast radius of <em>any</em> single
breach is the <em>entire system</em>: one compromised component equals total compromise. The single
wall is also a single point of failure — bypass it once and everything falls.
<br><br>
<strong>Zero trust fixes the "trust everything inside" assumption:</strong> instead of trusting a
request because of where it comes from (inside the network), <em>every</em> request is
authenticated and authorized regardless of origin — internal service-to-service calls use mutual
TLS and per-request tokens and are checked just like external ones. So a breached component can't
simply command other services by virtue of being "inside"; it still has to authenticate and pass
each service's authorization. This removes the free lateral movement — the attacker who breaches one
service hits a verification wall at every next hop.
<br><br>
<strong>Segmentation fixes the "reach everything" problem</strong> by containing the blast radius
structurally: sensitive assets (PII, payment data, secrets) live behind their own tight boundaries
with least-privilege access, and data is split by ownership rather than pooled in one shared store
with one credential. So a breach of a front-end or catalog service reaches only <em>that</em>
component's limited data and permissions — not the crown jewels, which sit behind separate controls
the compromised service was never granted access to. Together, zero trust (verify every hop) and
segmentation (isolate sensitive assets, least privilege) transform the blast radius from "the whole
system" to "the one compromised component and its minimal reach" — which is the entire goal:
<em>assume breach, and design so a breach is contained</em> rather than catastrophic. Defense in
depth is the umbrella principle — many independent layers, so no single failure (including the
perimeter's) breaches everything.
</details>

**Q2.** What is threat modeling with STRIDE, why should it happen at *design* time, and why is
security best understood as a quality attribute that trades against others (with an important
asymmetry)?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Threat modeling with STRIDE</strong> is systematically examining a design for how it could
be attacked, by walking each component and data flow (especially each trust boundary) and asking
whether it's vulnerable to each STRIDE category: <strong>S</strong>poofing (impersonation),
<strong>T</strong>ampering (altering data), <strong>R</strong>epudiation (denying an action with no
proof/audit), <strong>I</strong>nformation disclosure (leaking data), <strong>D</strong>enial of
service (overwhelming it), and <strong>E</strong>levation of privilege (gaining unauthorized
rights). For each identified threat you then decide a mitigation. It gives structure to "what can go
wrong here?" so you don't just rely on intuition or miss a category.
<br><br>
<strong>Why at design time:</strong> because the most effective security controls are
<em>architectural</em> — trust boundaries, where authN/authZ live, segmentation, isolation of
sensitive data — and these are cheap to design in but painful or impossible to retrofit after the
structure is built (you can't easily add segmentation or zero trust to a system built on "everything
trusts everything"). Finding a threat during design lets you address it structurally, before it's
baked in; finding it after launch means expensive rework or, worse, a breach. Threat modeling is the
security analog of a pre-mortem — surface the attack while it's still cheap to prevent, rather than
discovering it in an incident.
<br><br>
<strong>Why it's a quality attribute that trades against others — with an asymmetry:</strong> like
every -ility (Lesson 3), security conflicts with other qualities — each authentication/authorization
check adds latency and user friction (usability, performance), encryption costs CPU, segmentation
and least-privilege add complexity and operational overhead. So security isn't a dial to max out
blindly; it's calibrated to the threat and the asset (a public banking API and an internal analytics
tool warrant very different levels — over-securing the latter just adds cost and friction for no
benefit). <em>But</em> there's a crucial asymmetry with most other trade-offs: the downside of
<em>under</em>-investing in security is potentially <strong>catastrophic and non-recoverable</strong>
— a breach can leak millions of users' data, incur massive fines, destroy trust, or end the company —
whereas the cost of the security control is usually bounded (some latency, some complexity). Because
the tail risk is existential, the calibration errs toward caution for anything touching sensitive
data or exposed to the internet, and the cheap-to-design-in <em>architectural</em> controls (trust
boundaries, segmentation, least privilege, zero trust) are built from the start rather than deferred.
So security is a genuine trade-off (you don't blindly maximize it and cripple usability), but one you
resolve with a heavy thumb on the scale toward protection where the blast radius of getting it wrong
is severe — which is most places that matter.
</details>

---

## Homework

Draw the trust boundaries of your current system: where does the internet meet your system, where do
user-facing components call internal ones, where do you call third parties, and where does the most
sensitive data live? For each boundary, check whether authentication, authorization, and input
validation actually happen there. Then look for the "soft center" problem: do your internal services
trust each other by network location (plain HTTP, no per-request authZ), and would a breach of one
component reach sensitive data it shouldn't? Run a quick STRIDE pass on your riskiest boundary
(usually the public edge or the most sensitive data store), and name the single highest-priority
architectural security gap to fix.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise turns the principles into an audit of a real system. A strong response:
<br><br>
<strong>Maps trust boundaries and checks controls at each.</strong> The valuable act is drawing
where trust actually changes (internet↔edge, edge↔services, service↔third-party, service↔sensitive
data) and honestly checking whether authN, authZ, and input validation happen at each — a common
finding being that the edge is guarded but internal boundaries are not (authorization assumed rather
than enforced at services, or done only at the gateway).
<br><br>
<strong>Hunts the soft center.</strong> The highest-value finding on most real systems is some form
of "everything inside trusts everything" — internal services calling each other over plain HTTP with
no mutual auth and no per-request authorization, and/or a shared database whose compromise via any
one service would expose sensitive data broadly. Recognizing this as a large-blast-radius risk
(one breached component → wide reach) and knowing the fixes (mTLS + per-request authZ between
services; segmenting sensitive data behind its own least-privilege boundary; isolating/tokenizing
payment data; moving secrets to a manager) is exactly the zero-trust/segmentation lesson applied.
Secrets-in-config and over-broad service credentials are common concrete gaps.
<br><br>
<strong>Runs STRIDE on the riskiest boundary and prioritizes one fix.</strong> Applying STRIDE to
the public edge or the most sensitive data store surfaces specific, categorized threats and
mitigations rather than vague "we should be more secure." The architect's move is then to pick the
<em>single highest-priority architectural gap</em> by blast radius — usually either the lack of
segmentation around sensitive data (so a breach is contained) or the lack of zero-trust between
services (so lateral movement is blocked), because those are structural, high-impact, and expensive
to retrofit later. The takeaway a good answer reaches: security lives in the structure (trust
boundaries, segmentation, least privilege, zero trust), most systems have a "soft center" that turns
any breach into a large one, and the fixes are architectural and best built in now — so the concrete
output is one prioritized structural change (contain the blast radius of a breach), not a checklist
of app-level patches.
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 25 — Observability Architecture →](lesson-25-observability){: .btn .btn-primary }
