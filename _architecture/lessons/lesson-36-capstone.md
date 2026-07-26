---
title: "Lesson 36 — Capstone: Design a System End to End"
nav_order: 4
parent: "Phase 8: The Architect in Practice"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 36: Capstone — Design a System End to End

{: .note }
> **Words to know**
> - **capstone** — the culminating exercise that puts the whole track together; synthesis, not new material.
> - **the arc** — the end-to-end path an architect walks: drivers → quality attributes → style → boundaries → data → communication & resilience → cross-cutting → deployment → documentation → evaluation & evolution.
> - **ASR (architecturally significant requirement)** — the small set of requirements that actually shape the architecture (Lesson 4).
> - **defended design** — a design presented with its reasoning, trade-offs, and rejected alternatives made explicit — not just boxes and arrows.
> - **conscious trade-off** — a downside you named and accepted on purpose, rather than one you stumbled into.

## Concept

This is the synthesis. Everything in the track has been a *part* of the job — reasoning about quality
attributes, choosing structure and style, taming distributed data, designing for the cross-cutting
qualities, documenting, evaluating, evolving. The capstone puts them in **sequence**: taking a real
product brief from raw requirements to a *defended* architecture, walking the whole arc the way an
architect actually does. There is no new material here — the skill being tested is *integration*: doing
the parts in the right order, letting each stage feed the next, and — the thread through the entire track
— making every trade-off **explicit and conscious** instead of implicit and accidental.

```
   THE ARC — the architect's end-to-end path (each stage feeds the next)

   ① drivers & ASRs ────────▶ ② quality attributes (measurable scenarios)
      (Lesson 4)                  (Lesson 3)
        │                            │
        ▼                            ▼
   ③ style choice ◀──────────── from the drivers (Lesson 8)
        │
        ▼
   ④ bounded contexts & boundaries (Lessons 5–7)
        │
        ▼
   ⑤ data & consistency (Lessons 15,17,19,22) ─▶ ⑥ communication & resilience (16,18)
        │                                              │
        ▼                                              ▼
   ⑦ cross-cutting: security · observability · scale (23–25)
        │
        ▼
   ⑧ deployment (27) ─▶ ⑨ documentation: C4 + key ADRs (28–29)
        │
        ▼
   ⑩ evaluation & evolution plan (30–32)  ─── and throughout: NAME THE TRADE-OFFS
```

The arc is not a rigid waterfall — you'll loop back (a data-consistency reality forces a style rethink) —
but the *ordering* matters: the **drivers and ASRs come first** (Lesson 4 — find the few things that
actually shape the architecture), because everything downstream is *justified by them*. A style chosen
without drivers is fashion; a boundary drawn without the domain is guesswork; a consistency model chosen
without the business cost of being wrong is a gamble. The capstone's real lesson, and the track's: an
architect's output is not a diagram — it's a **chain of justified decisions**, each traceable to a driver,
each with its trade-off named. Produce that, and you've learned to reason like an architect.

## Going Deeper

**The arc, stage by stage — how each feeds the next.** The value of the sequence is that each stage
*constrains and informs* the next, so decisions compound into a coherent whole rather than a pile of
independent choices:
1. **Drivers & ASRs** (Lesson 4) — extract the handful of architecturally significant requirements
   (quality attributes + key functionals + constraints) from the brief, and *ignore the rest for now*.
   This is the foundation everything else is justified against.
2. **Quality attributes** (Lesson 3) — turn the vague "-ilities" into *measurable scenarios* (stimulus →
   response → measure). "Reliable" becomes "survives a region outage with < 30s recovery, no committed-data
   loss." Now you can design *for* them and later *evaluate against* them.
3. **Style choice** (Lesson 8) — pick a starting architectural style *from the drivers* (monolith?
   modular monolith? services? event-driven?). Default to the simplest that serves the ASRs; you can
   evolve later (Lesson 31).
4. **Bounded contexts & boundaries** (Lessons 5–7) — find the domain seams; draw component/service
   boundaries by change axis and team, aligned to the domain, with dependencies pointing toward
   stability. This is the highest-leverage act.
5. **Data & consistency** (Lessons 15, 17, 19, 22) — choose stores by access pattern; pick a consistency
   model per data item against the *business cost of being wrong*; face the distributed-data
   consequences (sagas, idempotency) that any split imposes.
6. **Communication & resilience** (Lessons 16, 18) — choose sync vs async per interaction; design for the
   failures you *know* are coming (timeouts, retries+backoff, circuit breakers, bulkheads, graceful
   degradation).
7. **Cross-cutting** (Lessons 23–25) — scalability (design the load path, statelessness), security (trust
   boundaries, least privilege, threat-model the riskiest boundary), observability (traces/metrics/logs,
   thread the correlation ID) — designed *in*, not bolted on.
8. **Deployment** (Lesson 27) — how it's built, released (zero-downtime strategy), and run;
   deployability as a first-class attribute; cost as a design output.
9. **Documentation** (Lessons 28–29) — a C4 Context + Container sketch for the audiences, and the key
   ADRs capturing the significant decisions with their alternatives and consequences.
10. **Evaluation & evolution** (Lessons 30–32) — run scenarios against the design to find risks and
    trade-off points; write the fitness functions; name what you'd defer to the last responsible moment
    and how you'd evolve or modernize.

**The through-line: everything is a trade-off, made explicit.** The single sentence that defines the job
(Lesson 2) is also the capstone's grading criterion. A design that lists only benefits is a sales pitch;
a *defended* design names, at each stage, **what it gave up** — the consistency traded for availability,
the simplicity traded for scale, the flexibility traded for boring-tech reliability — and *why the trade
is right for these drivers*. The mark of the architect is not a design with no downsides (impossible) but
a design whose downsides were *chosen consciously*, weighed against the drivers, and recorded (ADRs) so
the next person understands the intent.

{: .warning }
> **The capstone tests integration, not recall — and judgment over "the right answer"**
> There is no single correct architecture for a real brief (Lesson 2 — "it depends"), so the capstone
> isn't graded on matching a key. It's graded on <strong>reasoning</strong>:
> - Did you <strong>start from the drivers/ASRs</strong>, or jump to a solution (a style, a technology)
>   before understanding what shapes the architecture?
> - Is each decision <strong>traceable to a driver</strong> — can you justify <em>why</em> this style,
>   these boundaries, this consistency model, from the requirements?
> - Did you make the <strong>trade-offs explicit</strong> — naming what each choice costs, not just what
>   it buys?
> - Did you resist the <strong>anti-patterns</strong> (Lesson 35) — no premature microservices/scaling,
>   no speculative generality, no distributed monolith, appropriate <em>proportion</em> to the actual
>   drivers?
> - Did you <strong>document and plan to evaluate/evolve</strong> — C4, key ADRs, fitness functions, and
>   an honest list of risks?
> A simpler design with well-reasoned, well-defended trade-offs beats a more sophisticated design
> presented as flawless. The whole track has been teaching one thing: <em>reason like an architect —
> make the trade-offs explicit</em>. The capstone is where you show it.

**How to present a defended design.** When you produce the capstone, structure it as the arc, and for the
deliverables: the **C4 Context + Container** sketch (Lesson 28) shows the shape for its audiences; the
**three key ADRs** (Lesson 29) capture your most significant decisions *with their rejected alternatives
and honest consequences* — pick the decisions that were genuine one-way doors; the **top quality
attributes with scenarios** (Lesson 3) show what the architecture is *for* and give the yardstick to
evaluate it; and the **honest list of trade-offs and risks** (Lessons 30, 35) shows you know where your
own design is weak. That honesty — knowing and stating your design's soft spots — is not a weakness in the
submission; it's the strongest signal that you're thinking like an architect rather than selling a
solution.

---

## Lab — Design Exercise (Capstone)

**The brief.** Design the architecture, end to end, for **"CityEats"** — a **food-delivery platform** for
a mid-sized market. The essentials from the product brief:
- Customers browse restaurants, place orders, pay, and track their delivery live on a map.
- Restaurants receive orders, accept/reject them, and mark them ready.
- Couriers are offered nearby deliveries, accept them, and their location is tracked during delivery.
- Payments go through a third-party provider; the platform takes a commission.
- **Peak load is extreme and spiky** (Friday/Saturday dinner; a promo can 10× traffic in minutes).
- **Money correctness is non-negotiable** (no double charges, no lost orders, accurate payouts).
- The **live-tracking** feature means high-frequency location updates from thousands of couriers.
- The company is a startup: a **small team**, needs to ship an MVP fast, but with a credible path to
  scale.

**Produce a defended architecture end to end.** Walk the arc. Deliver: (a) the top **quality attributes
as scenarios**; (b) the **style choice** and why; (c) the **bounded contexts / boundaries**; (d) the
**data & consistency** decisions; (e) **communication & resilience**; (f) **cross-cutting** (security,
observability, scale); (g) **deployment**; (h) a **C4 Context + Container** sketch (words/ASCII); (i)
**three key ADRs** (with alternatives + consequences); (j) an honest list of **trade-offs and risks**, and
an **evolution plan**. There is no single right answer — you're graded on reasoning and explicit
trade-offs.

**Your response:**

<details>
<summary>Show Model Answer (a full walk of the reasoning)</summary>
<br>
This is one coherent, defensible answer — not <em>the</em> answer. What matters is that every decision is
traced to a driver and its trade-off is named. (A different architect could justify a different design;
they'd be graded the same way.)
<br><br>
<strong>① Drivers & ASRs (start here, always).</strong> From the brief, the architecturally significant
requirements are: <em>(1) correctness under contention and money-safety</em> (no double charge, no lost
order, accurate payouts) — the highest ASR; <em>(2) elasticity for extreme spiky load</em> (10× in
minutes on peak nights/promos); <em>(3) high-frequency real-time location ingestion</em> (thousands of
couriers, live tracking); <em>(4) small team / fast MVP with a path to scale</em> (a constraint that
pushes <em>against</em> premature complexity — Lesson 35). Non-ASRs to defer: the exact restaurant-search
ranking, the marketing site, admin tooling — real, but they don't shape the architecture. Naming these
four (and consciously setting the rest aside) is the foundation.
<br><br>
<strong>② Quality attributes as scenarios (measurable).</strong>
<ul>
<li><em>Correctness:</em> "10,000 customers order in the same minute during a promo → each order is
charged exactly once and never lost → zero double-charges/lost orders, verifiable in reconciliation."</li>
<li><em>Availability under spike:</em> "traffic 10×s in 5 minutes → ordering stays up → error rate < 1%,
p95 order-placement < 2s."</li>
<li><em>Real-time tracking:</em> "5,000 couriers each emit location every few seconds → customers see
live position → update latency < 5s, and this load does <em>not</em> degrade the ordering path."</li>
<li><em>Deliverability/velocity (team constraint):</em> "a small team ships changes → multiple times a
day, zero-downtime → no coordinated big-bang releases."</li>
</ul>
<br>
<strong>③ Style choice — start as a modular monolith, carve out the two things that don't fit it.</strong>
The team constraint (small, fast MVP) argues loudly <em>against</em> starting with many microservices
(Lesson 11 — the "you must be this tall" premium; Lesson 35 — premature scaling / distributed monolith is
the classic startup killer). So: a <strong>modular monolith</strong> for the core transactional domain
(ordering, restaurants, payments) — one deployable, clean internal bounded-context modules (Lesson 9), so
we get simplicity and transactions <em>now</em> and cheap splits <em>later</em>. But two workloads
genuinely don't fit the monolith and are split from day one because their <em>drivers differ</em>: (a)
<strong>live location ingestion/tracking</strong> — a fundamentally different workload (high-frequency
writes, different scaling shape, must not endanger the ordering path — a granularity <em>disintegrator</em>,
Lesson 13) → its own service + store; (b) <strong>notifications</strong> (order/courier updates) → async,
naturally separable. This is a hybrid (Lesson 8 — hybrids are the norm), chosen from the drivers: monolith
where simplicity wins, separate services only where a real driver forces it. <em>Trade-off named:</em>
we accept a little distribution (the tracking + notification split) to protect the ordering path and match
each workload's scaling shape, while keeping the transactional core simple.
<br><br>
<strong>④ Bounded contexts & boundaries (Lessons 5–7).</strong> Contexts: <em>Ordering</em> (cart, order
lifecycle), <em>Restaurant</em> (menu, accept/reject, ready), <em>Delivery/Dispatch</em> (offer to
couriers, assignment), <em>Payments</em> (charge, commission, payout), <em>Location/Tracking</em>
(ingest, live position), <em>Notifications</em>. Inside the monolith these are enforced modules (fitness
function: no module reaches into another's internals — Lesson 30). Location and Notifications are separate
services. "Customer" means different things in Ordering vs Payments — anti-corruption where they meet
(Lesson 7).
<br><br>
<strong>⑤ Data & consistency (Lessons 15, 17, 19, 22).</strong> <em>Store:</em> start with
<strong>Postgres</strong> for the transactional core (Lesson 19's honest default; ACID gives us
money-correctness for free while it's one database — this directly serves ASR #1 and the team constraint).
<em>Location</em> data is a different access pattern (high write volume, geo queries, ephemeral) → a store
suited to it (a Redis/geo or time-series store), <em>separate</em> from the transactional DB so its load
can't hurt ordering (ASR #3). <em>Consistency per item</em> (Lesson 15): order/payment state =
<strong>strong</strong> (money — the cost of being wrong is unacceptable); courier live location =
<strong>eventual</strong>/best-effort (a slightly stale dot on a map is fine); restaurant menu = can be
cached/eventually consistent for browse. <em>Payments across a boundary:</em> since Payments talks to a
third-party provider and (eventually) becomes its own service, use idempotency keys on every charge
(retries are guaranteed — Lesson 17) and an outbox so an order and its charge intent aren't lost in a
dual-write; a charge is idempotent so a retry never double-charges (ASR #1). While Payments is still a
module in the monolith, a local transaction covers order+payment-intent; when it's split later, this
becomes a saga with compensation (refund) — an evolution we <em>plan</em> for but don't pay for yet.
<br><br>
<strong>⑥ Communication & resilience (Lessons 16, 18).</strong> Inside the monolith: in-process calls
(no network tax). To the tracking service and notifications: <strong>async</strong> where tolerable
(location updates and notifications are fire-and-async — temporal decoupling protects ordering if those
lag). To the payment provider: sync request/response but wrapped in resilience — <em>timeout</em>
(everyone forgets it), <em>retry with backoff+jitter</em> on transient failure (idempotency makes retries
safe), a <em>circuit breaker</em> so a slow provider can't exhaust threads and take down checkout
(Lesson 18's exact scenario), and a <em>graceful path</em> (queue the order as "payment pending" rather
than hard-fail if the provider wobbles). <em>Bulkhead:</em> isolate the payment-call thread pool so its
saturation can't sink the whole app.
<br><br>
<strong>⑦ Cross-cutting (Lessons 23–25).</strong> <em>Scale:</em> keep the app <strong>stateless</strong>
(Lesson 27) behind a load balancer so we scale the ordering path horizontally for the spike (ASR #2); the
spiky writes (orders) hit Postgres — use read replicas for browse, and a queue to <em>load-level</em>
bursts (accept the order fast, process asynchronously) so a 10× spike doesn't hammer the write path
synchronously. Location ingestion scales independently (its own service/store). <em>Security</em>
(Lesson 24): trust boundary at the API gateway (authN there, Lesson 26); payments delegated to the PCI-
compliant provider so we never store card data (a deliberate build-vs-buy + security choice, Lesson 33 —
card handling is not our core and is a huge liability); least privilege between services; threat-model the
riskiest boundary (payment + payout — money movement). <em>Observability</em> (Lesson 25): correlation/
trace ID threaded from the edge through every hop, RED metrics on ordering, an SLO on order-placement
latency and success — so "checkout is slow on Friday" is diagnosable in minutes.
<br><br>
<strong>⑧ Deployment (Lesson 27).</strong> Twelve-factor (config in env, stateless, logs as streams);
containerized; <strong>rolling or blue-green zero-downtime</strong> releases (the team ships daily —
ASR #4) which forces <em>backward-compatible</em> changes (expand–contract, Lesson 26). Managed Postgres
and managed queue (rent the undifferentiated infra — Lesson 33/27); cost watched as a design output
(the location firehose and its store are the cost risk to watch).
<br><br>
<strong>⑨ Documentation — C4 (Lesson 28).</strong>
<pre>
CONTEXT:  [Customer] [Restaurant] [Courier] ──▶ ( CityEats ) ──▶ [Payment provider]
                                                     └──▶ [Maps/geocoding] [SMS/push provider]
CONTAINER:
  [Customer app] [Restaurant app] [Courier app]
        │  HTTPS/REST                │  WebSocket (location + live track)
        ▼                            ▼
   ┌── API gateway (authN, rate limit) ──┐
   │                                     │
   ▼                                     ▼
  [ Core Monolith ]  ── async events ──▶ [ Location/Tracking service ] ──▶ [Geo store]
   modules: Ordering, Restaurant,        [ Notifications service ] ──▶ [SMS/push]
   Dispatch, Payments                          ▲
        │ reads/writes                          │ async
        ▼                                        │
   [ Postgres (+read replica) ]  ── outbox ──────┘   [Payment provider (external)]
</pre>
<em>Context</em> serves everyone (what is CityEats, who uses it, what it depends on); <em>Container</em>
serves devs/ops (the deployable units and how they talk). Legend + labeled relationships assumed.
<br><br>
<strong>⑩ Three key ADRs (one-way doors, with alternatives + honest consequences — Lesson 29).</strong>
<ul>
<li><em>ADR-1: Start as a modular monolith, not microservices.</em> <strong>Context:</strong> small team,
fast MVP, money-correctness needs transactions. <strong>Decision:</strong> one deployable, enforced
internal modules; split only Location + Notifications. <strong>Alternatives:</strong> full microservices
(rejected — premature scaling / distributed-monolith risk for a small team, Lesson 35; the "you must be
this tall" premium we can't pay yet); big ball of mud single-module (rejected — would rot, and blocks the
later split). <strong>Consequences (good/bad):</strong> simplicity + transactions + fast delivery now, and
cheap splits later <em>if</em> we kept the module boundaries honest (a fitness function must enforce them —
if we let them rot, the future split gets expensive). Downside accepted: one deploy for the core (a very
big spike could eventually need us to split ordering out — planned, not now).</li>
<li><em>ADR-2: Delegate all card handling to a third-party PCI provider; idempotent charges + outbox.</em>
<strong>Context:</strong> money-correctness is the top ASR; card data is a massive liability and not our
core. <strong>Decision:</strong> never store cards; charge via provider with idempotency keys; outbox to
avoid dual-write loss. <strong>Alternatives:</strong> build our own payment handling (rejected — not core,
enormous compliance/security cost, Lesson 33); best-effort charge without idempotency (rejected — retries
are guaranteed, would double-charge, Lesson 17). <strong>Consequences:</strong> money-safe and off the PCI
hook; downside accepted — dependence on the provider (mitigated with circuit breaker + graceful "payment
pending"), and eventual-consistency window between order and confirmed payment the UI must handle.</li>
<li><em>ADR-3: Split location/tracking into its own service + store from day one.</em>
<strong>Context:</strong> thousands of couriers emitting high-frequency location; must not degrade
ordering (ASR #3). <strong>Decision:</strong> separate service + geo/time-series store, fed async.
<strong>Alternatives:</strong> location writes into the main Postgres (rejected — the write firehose would
contend with money-critical order writes — a sensitivity/trade-off point, Lesson 30); keep it in the
monolith process (rejected — different scaling shape, couples the two). <strong>Consequences:</strong>
ordering is protected and location scales independently; downside accepted — this is real distribution
(an extra service, async plumbing, an eventual-consistency store) taken on <em>deliberately</em> because a
genuine driver forces it (contrast the premature splits we refused in ADR-1).</li>
</ul>
<strong>Honest trade-offs & risks (Lessons 30, 35).</strong> Biggest risk: the ordering write path under a
10× promo spike on a single Postgres primary — mitigated by queue-based load-leveling and read replicas,
but the write ceiling is the thing to watch and the first place we'd scale (replica → partition by
region/tenant → split ordering service — the ladder, Lesson 22), deferred to the last responsible moment.
Trade-off points named: strong consistency for orders/payments <em>vs</em> the availability/latency it
costs under load (we chose correctness — it's the top ASR); eventual consistency for location <em>vs</em>
freshness (we chose scale — a stale dot is fine). Conscious simplicity: we <em>refused</em> premature
microservices, a generic workflow engine, and speculative multi-region — proportion to the actual drivers
(Lesson 35). <strong>Evolution plan (Lesson 31–32):</strong> keep module boundaries enforced by fitness
functions so we can strangler-fig ordering or payments out when scale or team growth demands (Lesson 32);
defer sharding and the payments-saga until the numbers require them; keep the payment provider behind an
anti-corruption boundary so it stays swappable.
<br><br>
<strong>Why this is a good architect's answer (the grading, Lesson 2).</strong> Every decision traces to
one of the four ASRs; every choice names what it <em>costs</em> as well as what it buys; it resists the
startup anti-patterns (no premature microservices/scaling/generality — <em>proportion</em>); it splits
<em>only</em> where a real driver forces it and can defend both the splits and the refusals; and it plans
to document (C4/ADRs), evaluate (scenarios, fitness functions), and evolve (strangler, the scaling ladder,
deferred decisions). It is not flawless — and it <em>says</em> where it's weak (the Postgres write ceiling).
That combination — justified by drivers, explicit about trade-offs, proportionate, documented, and honest
about its own soft spots — is what it means to reason like an architect. A simpler or a more elaborate
design could earn the same marks if defended the same way; a "perfect-looking" design presented without
its trade-offs would not.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| The whole arc | *Fundamentals of Software Architecture*, Richards & Ford |
| The hard parts (data, services, trade-offs) | *Software Architecture: The Hard Parts*, Ford, Richards, Sadalage & Dehghani |
| Data-intensive design | *Designing Data-Intensive Applications*, Kleppmann |
| Real-world system design walk-throughs | <https://github.com/donnemartin/system-design-primer> |
| C4 for the deliverable | <https://c4model.com/> |
| The whole track | [Software Architecture learning plan]({{ '/architecture/learning-plan.html' | relative_url }}) |

---

## Checkpoint

**Q1.** What is "the arc," why must the drivers/ASRs come *first*, and why is an architect's real output a
"chain of justified decisions" rather than a diagram?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>The arc</strong> is the end-to-end path an architect walks to take a brief to a defended design:
drivers & ASRs → quality attributes (as measurable scenarios) → style choice → bounded contexts &
boundaries → data & consistency → communication & resilience → cross-cutting (security, observability,
scale) → deployment → documentation (C4 + ADRs) → evaluation & evolution. It's not a rigid waterfall
(you loop back), but the ordering matters because each stage <em>constrains and informs</em> the next.
<br><br>
<strong>Why drivers/ASRs come first:</strong> the architecturally significant requirements — the handful
of quality attributes, key functionals, and constraints that actually shape the system (Lesson 4) — are
the <em>foundation everything downstream is justified against</em>. Choose a style without drivers and
it's just fashion; draw boundaries without the domain and it's guesswork; pick a consistency model without
the business cost of being wrong and it's a gamble. The drivers are what let you <em>justify</em> every
later decision and what you'll later <em>evaluate</em> the design against. Jumping to a solution (a style,
a technology) before understanding the drivers is the most common and most damaging capstone mistake.
<br><br>
<strong>Why the output is a chain of justified decisions, not a diagram:</strong> a diagram shows the
<em>what</em> (the boxes) but not the <em>why</em> — and the why (which driver each decision serves, which
alternatives were rejected, which trade-off was consciously accepted) is the actual architectural work and
the part that's irreplaceable (Lesson 28–29). Anyone can draw boxes; the architect's value is that each
box is <em>there for a reason traceable to a driver</em>, and each carries a named trade-off. A design
presented as a diagram alone can't be evaluated, defended, or evolved, because the reasoning is missing.
So the real deliverable is the reasoning — a chain of decisions each tied to a driver and each honest about
its cost — with the diagram as one view onto it.
</details>

**Q2.** Why is "make every trade-off explicit" the grading criterion for the capstone (and the track), and
why does a simpler, well-defended design beat a sophisticated one presented as flawless?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Why explicit trade-offs are the criterion:</strong> the one sentence that defines the whole
discipline (Lesson 2) is <em>everything in architecture is a trade-off</em> — there is no free lunch and no
context-free "best" design, so the architect's job isn't to find a design with no downsides (impossible)
but to make the downsides <em>conscious</em>: to name, at each decision, what it costs as well as what it
buys, weigh that against the drivers, and record it (ADRs) so the intent survives. A design that lists only
benefits is a sales pitch that has hidden its trade-offs (which means they'll be discovered painfully
later); a <em>defended</em> design surfaces them, which is both more honest and more useful — it can be
evaluated, questioned, and evolved. Since the track has been teaching one thing — reason like an architect,
make the trade-offs explicit — the capstone grades exactly that: is each decision traced to a driver, and
is its trade-off named?
<br><br>
<strong>Why simpler-but-defended beats sophisticated-but-flawless-looking:</strong> because the goal is
judgment, not sophistication. A more elaborate design is often <em>worse</em> (premature complexity,
speculative generality, distributed monolith — the anti-patterns of Lesson 35 that come from over-
building relative to the drivers), and presenting <em>any</em> design as flawless is itself the tell of an
immature architect — it means the trade-offs were hidden or not seen, which is more dangerous than a
simpler design whose costs are known. A simpler design that's <em>proportionate</em> to the actual drivers
and whose trade-offs and risks are stated <em>demonstrates the core skill</em>: matching the architecture
to the real requirements, resisting over-engineering, and being honest about where it's weak. Knowing and
stating your design's soft spots isn't a weakness in the submission — it's the strongest evidence that
you're thinking like an architect (weighing trade-offs against drivers) rather than selling a solution. So
the mark goes to reasoning, proportion, and honesty — not to apparent sophistication.
</details>

---

## Homework

Take a real product you know well — the one you work on, or one you'd love to build — and run the full arc
on it yourself, end to end, exactly as the capstone lab does: extract the ASRs, write the top quality
attributes as scenarios, choose a style and justify it from the drivers, draw the bounded contexts, make
the data/consistency and communication/resilience decisions, address the cross-cutting concerns and
deployment, sketch a C4 Context + Container, write three key ADRs (with alternatives and honest
consequences), and finish with an unflinching list of trade-offs, risks, and an evolution plan. Then do the
hardest and most valuable part: **defend it to yourself** — for each major decision, can you name the
driver it serves, the alternatives you rejected, and the trade-off you consciously accepted? Wherever you
can't, you found a decision you were making by reflex rather than reasoning. That gap is the whole point of
the track — and closing it, deliberately, one decision at a time, is what it means to have become an
architect.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
A strong response runs the whole arc on a real system and — the crux — <em>defends every major decision to
itself</em>.
<br><br>
<strong>Walks the arc in order, driver-first.</strong> The discipline that separates architect-thinking
from feature-thinking is starting from the ASRs and justifying everything downstream against them, rather
than jumping to a style or a favourite technology. A good answer shows the chain: these four drivers →
therefore this style → therefore these boundaries → therefore this consistency model → and so on, with
each link explicit.
<br><br>
<strong>Produces the deliverables honestly.</strong> Real quality-attribute scenarios (measurable, not
vague), a C4 Context + Container that stays at one abstraction level each, three ADRs that include the
<em>rejected</em> alternatives and a named downside apiece, and — the maturity signal — an unflinching list
of the design's own weak spots and risks (not a flawless pitch). Resisting the anti-patterns (proportion
to the drivers; no premature microservices/scaling/generality) throughout is part of the demonstration.
<br><br>
<strong>Defends every decision — and finds the reflexive ones.</strong> The most valuable outcome is
discovering the decisions you <em>couldn't</em> defend — where you chose by habit, fashion, or "best
practice" rather than by driver-and-trade-off. Each of those is a gap between doing architecture and
reasoning like an architect. The takeaway the whole track has built to: an architect's output is a chain
of justified decisions, each traceable to a driver and each with its trade-off made explicit and conscious;
the skill is integration and judgment, not recall; there's no single right answer, only well-defended ones;
and you've become an architect precisely to the degree that you can, for every significant decision, name
the driver it serves, the alternatives you rejected, and the trade-off you chose on purpose. Closing the
reflexive-decision gaps, deliberately, is the practice — for the rest of your career.
</details>

---

{: .note }
> **You've reached the end of the Software Architecture track.**
> All 36 lessons — from *what an architect actually does* to this end-to-end capstone — are behind you.
> The through-line was one sentence: *everything in architecture is a trade-off, and the job is making the
> trade-offs explicit.* Everything else — quality attributes, styles, distributed systems, data and scale,
> the cross-cutting qualities, documenting/evaluating/evolving, and the architect-in-practice skills — was
> in service of reasoning like an architect. Keep running the arc on real systems; the judgment compounds.
> Pair it with the [Engineering Leadership]({{ '/leadership/learning-plan.html' | relative_url }}) track
> for the people-half of the role.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Software Architecture learning plan]({{ '/architecture/learning-plan.html' | relative_url }}){: .btn .btn-primary }
