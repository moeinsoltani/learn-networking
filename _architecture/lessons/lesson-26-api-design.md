---
title: "Lesson 26 — API Design & Management"
nav_order: 4
parent: "Phase 6: Cross-Cutting Quality Attributes"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 26: API Design & Management

{: .note }
> **Words to know**
> - **API (contract)** — the agreed interface between a provider and its consumers; a promise about how to call it and what comes back.
> - **backward compatible** — a change that doesn't break existing clients (old callers keep working).
> - **versioning** — supporting multiple API versions so consumers can migrate on their own schedule.
> - **expand–contract (parallel change)** — add the new, migrate consumers, then remove the old — never break in place.
> - **tolerant reader** — a consumer that ignores fields it doesn't understand, so additions don't break it.
> - **API gateway** — an edge component handling cross-cutting concerns (auth, rate limiting, routing) for many APIs.
> - **contract testing** — tests that verify provider and consumer agree on the contract, so services can evolve independently.

## Concept

An API is a **contract**, and once other people depend on it, it becomes one of the most expensive
things in your entire system to change. This is the defining property to internalize: a public API
is a **one-way door** (Lesson 2). You can refactor internal code freely, but the moment external
clients (other teams, customers, third parties) depend on your API's shape, you can no longer just
change it — a breaking change breaks *them*, and you often can't even make them upgrade. So APIs
demand a different discipline than internal code: design for the consumer, and evolve without
breaking.

```
   THE API IS A CONTRACT — and a PRODUCT for its consumers

   [ your service ]══ API (the contract) ══[ consumers you don't control ]
        internals:                            other teams, mobile apps,
        change freely                         customers, partners, old
        (reversible)                          app versions in the wild
                                              → you CANNOT force them to upgrade

   Once published, the API is a ONE-WAY DOOR:
   ✗ breaking change  → breaks consumers you can't reach
   ✓ COMPATIBLE change → add, never remove/rename in place
                         (expand → migrate → contract)
```

Two consequences follow. First, **design for the consumer**, not for your internal convenience — the
API's job is to serve the people calling it, so its shape should reflect their needs and their mental
model, and it should be documented as a product they can actually use. Second, **evolve without
breaking**: because you can't force upgrades, you add and deprecate rather than change and remove —
compatibility is a first-class discipline, not an afterthought.

## Going Deeper

**Style: REST, RPC/gRPC, GraphQL — pick for the consumer (recap + API lens).** From Lesson 16: REST
(resource-oriented, ubiquitous, cacheable — great for public/broad APIs), gRPC (binary, strongly
typed, fast — great for internal high-performance service-to-service), GraphQL (client picks the
fields — great for rich clients that would over-fetch or make many round-trips). The API-design lens
adds: for a *public* API, favor the style your consumers can most easily adopt and that you can
evolve compatibly (usually REST), and design the *resources/operations* around the consumer's tasks,
not your database tables. A good API models the consumer's domain, not your internals.

{: .warning }
> **Versioning & compatibility: never break v1 — use expand–contract**
> Because you can't force consumers to upgrade, the cardinal rule is <strong>never break an existing
> version in place</strong>. Techniques:
> - <strong>Backward-compatible changes are safe:</strong> <em>adding</em> an optional field, a new
>   endpoint, or a new optional parameter doesn't break existing clients (if they're tolerant
>   readers). Prefer additive change.
> - <strong>Breaking changes need a new version or expand–contract:</strong> removing/renaming a
>   field, changing a type, or making an optional field required <em>will</em> break clients. Don't
>   do it in place. Either introduce a new version (v2) and run v1 and v2 in parallel while consumers
>   migrate, or use the <strong>expand–contract</strong> (parallel-change) pattern: <em>expand</em>
>   (add the new field/endpoint alongside the old), <em>migrate</em> (move consumers over, on their
>   schedule), <em>contract</em> (remove the old only once no one uses it).
> - <strong>Versioning strategies:</strong> URI (<code>/v1/orders</code> — visible, simple, common
>   for public APIs) vs header/content-negotiation (cleaner URLs, less visible). Either works; the
>   discipline (parallel versions, deprecation windows) matters more than the mechanism.
> - <strong>Deprecation is a process, not an event:</strong> announce, provide a migration path and a
>   generous window, monitor who's still on the old version, and only then remove. Surprise removals
>   destroy consumer trust.
> The <strong>tolerant reader</strong> principle on the consumer side helps: consumers should ignore
> unknown fields (so the provider can add fields safely) and not over-depend on incidental details —
> "be conservative in what you send, liberal in what you accept" (Postel's law).

**The API gateway — cross-cutting concerns at the edge.** As the number of services and consumers
grows, an **API gateway** centralizes the concerns every API needs: authentication, rate limiting,
routing, TLS termination, request logging/tracing, and sometimes response aggregation. Benefits:
consumers hit one entry point, cross-cutting policy is enforced consistently in one place (not
reimplemented per service), and services are shielded from the raw internet (Lesson 24). Cautions:
the gateway is a chokepoint (a scaling and availability concern — Lesson 18) and shouldn't accrete
business logic (that belongs in services; a gateway bloated with domain logic becomes a new monolith
and coupling point). Keep it for genuinely cross-cutting, generic concerns.

**Contract testing — evolve services independently, safely.** In a distributed system, the risk is
that a provider changes its API and unknowingly breaks a consumer, discovered only in production.
**Contract testing** (e.g., consumer-driven contracts, Pact) captures the agreement between a
provider and its consumers as executable tests: the consumer declares what it needs, the provider's
build verifies it still satisfies that contract, so a breaking change is caught in CI, not
production. This is what *lets services evolve independently* (the whole point of microservices,
Lesson 11) without integration fear — a lighter, faster alternative to full end-to-end integration
tests, and an architectural enabler of team autonomy.

**Documentation is part of the deliverable.** An API nobody can figure out how to use is a failed
API, regardless of how well it's built. Machine-readable specs (OpenAPI/Swagger for REST, protobuf
for gRPC, the schema for GraphQL) that generate docs, clients, and mocks are the standard — they make
the contract explicit, discoverable, and testable. Treating the API as a *product* (Lesson: design
for the consumer) means its documentation, examples, and developer experience are first-class, not an
afterthought — because the consumers' ability to adopt it is what makes it valuable.

---

## Lab — Design Exercise

**The situation:** You maintain a widely-used **public REST API**: `GET /v1/orders/{id}` returns an
order as JSON, consumed by your own mobile apps (old versions still in the wild), several partner
integrations, and internal services. A new business requirement means every order must now include a
**`taxRegion`** field, and — for compliance — the API must *require* callers to send a `taxRegion` on
the order-creation endpoint `POST /v1/orders`. You cannot break the existing consumers (you can't
force the old mobile apps or partners to upgrade).

**Design the compatible evolution.** Handle both the additive change (returning a new field) and the
breaking change (a newly-required input field) without breaking existing clients — using
backward-compatible changes, expand–contract/versioning, tolerant readers, and a deprecation path.
State what you'd do differently for the two changes and why one is harder.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is separating the additive (easy) change from the breaking (hard) one and applying
compatible-evolution technique to each. A strong answer:
<br><br>
<strong>Change 1 — returning a new <code>taxRegion</code> field on <code>GET</code>: additive,
safe, do it in place.</strong> <em>Adding</em> a field to a response is backward-compatible: existing
consumers that don't know about <code>taxRegion</code> simply ignore it (assuming they're
<strong>tolerant readers</strong> — they don't choke on unknown fields, which well-behaved JSON
clients don't). So you can add <code>taxRegion</code> to the <code>GET /v1/orders/{id}</code> response
<em>right now</em>, no new version needed — old mobile apps and partners keep working (they just
don't use the new field), new consumers can start reading it. This is why additive change is the
preferred way to evolve APIs: it extends without breaking. (One caveat: make sure no consumer does
strict schema validation that rejects unknown fields — if a partner foolishly does, coordinate — but
the default assumption of tolerant readers holds, and part of good API governance is telling
consumers to <em>be</em> tolerant readers precisely so you can add fields safely.)
<br><br>
<strong>Change 2 — making <code>taxRegion</code> a <em>required input</em> on <code>POST
/v1/orders</code>: breaking, must NOT be done in place.</strong> Making a previously-absent field
<em>required</em> breaks every existing caller that doesn't send it — old mobile apps and partners
would suddenly get errors on order creation. This is the hard one because it's a genuinely breaking
change to the request contract, and you can't force consumers to upgrade. Options, best-first:
<ul>
<li><strong>Preferred: expand–contract with a compatibility bridge, keeping v1 working.</strong>
<em>Expand:</em> accept <code>taxRegion</code> as an <em>optional</em> field on
<code>POST /v1/orders</code> now, so new/updated clients can start sending it. For old clients that
<em>don't</em> send it, <em>derive a sensible default</em> where possible (e.g., infer
<code>taxRegion</code> from the shipping address or account country already in the request) so their
orders still succeed and are compliant — the server fills the gap rather than rejecting them.
<em>Migrate:</em> announce the deprecation, publish the migration guide, and actively move consumers
to sending <code>taxRegion</code> explicitly, on their schedule, with a generous window; monitor who
still omits it. <em>Contract:</em> only once (nearly) everyone sends it — or the old clients have
aged out — do you tighten the rule. If you can always derive a correct default, you may never need to
hard-break v1 at all.</li>
<li><strong>If you truly can't derive a default and must make it required:</strong> introduce a new
version — <code>POST /v2/orders</code> — where <code>taxRegion</code> is required, and run
<strong>v1 and v2 in parallel</strong>. New consumers use v2; existing consumers stay on v1 (where
you handle the missing field as above — default it, or accept the compliance gap for legacy orders
per legal guidance) and migrate to v2 over a deprecation window. You never break v1 in place; you
give consumers a v2 to move to on their own timeline, and retire v1 only after the window and
migration.</li>
</ul>
<strong>What's different between the two, and why one is harder:</strong> Change 1 (adding a
<em>response</em> field) is <em>additive</em> — it gives consumers <em>more</em>, and tolerant readers
ignore what they don't use, so nothing breaks and you do it in place. Change 2 (making a
<em>request</em> field required) is <em>subtractive/restrictive</em> — it <em>demands more</em> from
consumers than the old contract did, so every consumer not already meeting the new demand breaks. The
asymmetry is the key lesson: <strong>you can safely add to what you return and add optional things you
accept, but you cannot safely add new requirements on what consumers must send, nor remove/rename what
they depend on</strong> — those need expand–contract or a new parallel version and a deprecation
process. The mature move is to avoid the hard break entirely where possible (accept the field
optionally + derive a default, so v1 keeps working), and only version + deprecate when a real break is
unavoidable — always keeping the old version alive until consumers have migrated, because with a
public API you don't control the upgrade schedule and a surprise break destroys the trust the API's
value depends on.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| API as contract; evolving APIs | *Building Microservices* (2e), Sam Newman (Ch. 4–5) |
| Expand–contract / parallel change | Martin Fowler — <https://martinfowler.com/bliki/ParallelChange.html> |
| Tolerant Reader / Postel's law | <https://martinfowler.com/bliki/TolerantReader.html> |
| API gateway pattern | <https://microservices.io/patterns/apigateway.html> |
| Consumer-driven contract testing (Pact) | <https://docs.pact.io/> ; <https://martinfowler.com/articles/consumerDrivenContracts.html> |
| REST API design | <https://learn.microsoft.com/azure/architecture/best-practices/api-design> |

---

## Checkpoint

**Q1.** Why is a public API "one of the most expensive things to change," and what makes additive
changes safe while removing/renaming/newly-requiring fields is breaking? Explain expand–contract.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
A public API is expensive to change because it's a <strong>contract that external consumers you don't
control depend on</strong> — other teams, partner integrations, mobile apps already installed on
users' phones, third parties. Unlike internal code (which you can refactor freely because you control
all the callers), you generally <em>cannot force API consumers to upgrade</em>: the old mobile app is
on the user's device, the partner integrates on their own schedule. So it's a <strong>one-way
door</strong> (Lesson 2) — once published and depended upon, a breaking change breaks consumers you
can't reach or coordinate, causing outages for them and destroying trust in your API. That's why APIs
demand compatibility discipline that internal code doesn't.
<br><br>
<strong>Why additive is safe:</strong> <em>adding</em> to what you return (a new field), or adding a
new endpoint or a new <em>optional</em> input, doesn't break existing consumers — they simply don't
use the new thing, and <strong>tolerant readers</strong> ignore response fields they don't recognize.
The old contract is still fully honored; you've only extended it. So additive changes can be made in
place, and are the preferred way to evolve.
<br><br>
<strong>Why removing/renaming/newly-requiring is breaking:</strong> these <em>violate the existing
contract</em> that consumers rely on. Removing or renaming a field breaks any consumer reading it
(their code expects a field that's gone or now spelled differently). Changing a type breaks parsing.
Making a previously-optional input <em>required</em> breaks every consumer that wasn't already sending
it (they now get errors). In each case, code written against the old contract stops working — and you
can't force those consumers to fix their code. The asymmetry: you can give consumers <em>more</em>
(additive) safely, but you can't take away what they depend on or <em>demand more</em> from them
without breaking the ones who don't yet comply.
<br><br>
<strong>Expand–contract (parallel change):</strong> the technique for making a breaking change safely,
in three phases. <em>Expand</em> — add the new alongside the old (e.g., add the new field/endpoint,
or accept the new input optionally), so both old and new work simultaneously and new consumers can
adopt the new form. <em>Migrate</em> — move consumers from the old to the new, on <em>their</em>
schedule, over a generous deprecation window, with announcements and a migration path, monitoring who's
still on the old. <em>Contract</em> — only once no one depends on the old form do you remove it. The
key property is that at no point is a working consumer broken: there's always an overlap window where
both the old and new contract are honored, so consumers migrate voluntarily rather than being broken
by a change they didn't ask for. It's how you evolve an API through even breaking changes without
violating the "never break existing consumers in place" rule.
</details>

**Q2.** What cross-cutting concerns does an API gateway centralize, what are its risks, and how does
contract testing let services evolve independently?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>What an API gateway centralizes:</strong> the generic, cross-cutting concerns that every API/
service needs but shouldn't each reimplement — <em>authentication</em> (validate the caller/token at
the edge), <em>rate limiting / throttling</em> (protect against abuse and overload), <em>routing</em>
(direct requests to the right service), <em>TLS termination</em>, <em>request logging and tracing</em>
(attach the correlation ID, Lesson 25), and sometimes <em>response aggregation</em>. The benefits:
consumers have one entry point; cross-cutting policy is enforced <em>consistently in one place</em>
rather than divergently across many services; and internal services are shielded from the raw internet
(a security boundary, Lesson 24).
<br><br>
<strong>Its risks:</strong> (1) it's a <em>chokepoint</em> — every request flows through it, so it's a
scaling bottleneck and an availability single-point-of-concern (if the gateway is down, everything is
unreachable), demanding its own resilience and horizontal scaling (Lesson 18/23). (2) It tends to
<em>accrete business logic</em> — teams are tempted to put domain logic, transformations, and
orchestration into the gateway, which turns it into a bloated, shared coupling point (effectively a new
monolith at the edge) that every team must coordinate changes through, undermining the service autonomy
it was meant to support. The discipline: keep the gateway to genuinely <em>generic, cross-cutting</em>
concerns and keep business logic in the services.
<br><br>
<strong>How contract testing enables independent evolution:</strong> the danger in a distributed system
is that a provider changes its API and unknowingly breaks a consumer — discovered only in production,
which makes teams afraid to change anything (killing the independent-deployability that's the whole
point of services, Lesson 11). <strong>Contract testing</strong> (e.g., consumer-driven contracts /
Pact) captures the agreement between a provider and each of its consumers as <em>executable tests</em>:
each consumer specifies what it actually needs from the provider's API (which fields, which endpoints),
and the provider's build runs those contracts to verify it still satisfies every consumer. So if the
provider makes a change that would break a consumer, the provider's <em>CI fails</em> — the break is
caught at build time, by the team making the change, before it ships, rather than in production by the
consumer. This lets each service evolve and deploy <em>independently</em> with confidence: a provider
can refactor freely as long as the contract tests pass, and knows immediately if a change would break a
consumer. It's a lighter, faster, more reliable alternative to full end-to-end integration tests (which
are slow, brittle, and require standing up everything together), and it's the mechanism that makes
"independent deployability" safe in practice — turning the abstract promise of service autonomy into
something teams can trust, because the contracts, not luck, guarantee compatibility.
</details>

---

## Homework

Pick one important API in your system (public, partner-facing, or a heavily-depended-on internal
service API) and assess its evolvability. Has it been evolved with additive/compatible changes, or have
there been breaking changes that caused consumer pain? Is there a versioning strategy and a deprecation
process, or do changes ship and break people? Are consumers tolerant readers, and is the contract
captured anywhere (an OpenAPI/schema spec, contract tests) or only implicit? Identify the single biggest
API-evolution risk (often: no way to make a breaking change safely, or no contract tests so you can't
change the provider without fear) and the practice you'd add. Bonus: find a change you *need* to make and
design its compatible evolution.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise applies compatibility discipline to a real API. A strong response:
<br><br>
<strong>Assesses how the API has actually been evolved.</strong> The instructive finding is often a
history of <em>breaking changes shipped in place</em> (a field renamed, a response reshaped, an input
made required) that caused consumer pain — or the opposite, a team so afraid of breaking consumers that
the API has ossified and can't evolve at all. Both indicate a missing compatible-evolution discipline
(additive change, expand–contract, parallel versions), and naming which failure mode the API suffers
from points to the fix.
<br><br>
<strong>Checks for the supporting practices.</strong> The common gaps: no explicit versioning strategy
or deprecation process (so changes are ad hoc and break people, or fear-frozen); the contract captured
only implicitly (in code and tribal knowledge) rather than in a machine-readable spec (OpenAPI/protobuf/
GraphQL schema) that could generate docs/clients and make the contract explicit; and — especially for
internal service APIs — <em>no contract tests</em>, so the provider team can't change anything without
manual coordination or production surprises, which quietly undermines the independent deployability the
architecture promised. Recognizing that the absence of contract tests is <em>why</em> changing a
provider is scary is a valuable connection.
<br><br>
<strong>Names the biggest risk and the practice to add.</strong> The architect's move is to pick the
single highest-leverage improvement — usually either (a) establishing a compatible-evolution +
deprecation discipline (additive-first, expand–contract, parallel versions, generous windows) so
breaking changes stop hurting consumers, or (b) introducing contract testing so services can evolve
independently without fear, or (c) capturing the contract in an explicit spec so it's testable and
discoverable. The bonus (designing the compatible evolution for a real needed change) is where it gets
concrete: separating the additive part (safe, in place) from any breaking part (expand–contract or new
version + deprecation), exactly as in the lab. The takeaway a good answer reaches: an API is a contract
and a product, its evolvability is a first-class architectural concern (because you can't force
consumers to upgrade), and most APIs lack one of the three enablers — compatible-change discipline,
explicit contracts, or contract tests — whose addition is a specific, high-value investment rather than
a vague "improve our API."
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 27 — Cloud & Deployment Architecture →](lesson-27-cloud-deployment){: .btn .btn-primary }
