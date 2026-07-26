---
title: "Lesson 06 — Modularity & Component Boundaries"
nav_order: 2
parent: "Phase 2: Foundations of Structure"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 06: Modularity & Component Boundaries

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

If coupling and cohesion (Lesson 5) are the *forces*, drawing **boundaries**[°](#w-boundary) is the
*act*. Deciding where the lines go — what's a module, where one component ends and the
next begins — is the single highest-leverage thing an architect does, because a
boundary in the wrong place **leaks**[°](#w-leak) everywhere and every future change fights it, while
a boundary in the right place makes change local and cheap.

The essential idea: **a boundary is a promise.**

Inside the boundary sits everything about one thing — for an `Orders` module,
everything concerning the lifecycle of an order. That is the cohesion half of
Lesson 05. The internals are **hidden**: the data structures, the tables, the
helper classes, the sequence of steps.

Crossing the boundary is a **narrow, stable public interface**, and this is the
promise: everyone outside — `Inventory`, `Notifications`, anyone else — depends
**only** on that contract, and never on the internals.

The promise buys you exactly one thing, and it is the thing that makes large
systems survivable: **you can change the inside without asking anyone.** The
moment another module reaches past the interface — reading your tables,
importing your internal classes — the promise is void, and you have a
distributed monolith regardless of what the architecture diagram says.

A good boundary means: **high cohesion inside** (everything within changes together,
for the same reasons, owned by the same people) and **a narrow, stable interface
across** (outsiders depend only on the contract, never the guts). Get this right and
you can rework a module's internals freely; get it wrong and the internals leak, so
"internal" changes break half the system.

The question that decides boundaries: **what changes together?** Things that change
together for the same reason belong in the same module; things that change for
different reasons, at different rates, driven by different people, belong apart.

## Going Deeper

**Align boundaries with the domain, not the technical layer.** The most common
boundary mistake is slicing by *technical role* — "all the controllers here, all the
services there, all the repositories over there" (horizontal layers). The problem:
almost every real change is a *feature* ("add a discount to orders"), and a feature
cuts *across* all those layers, so you touch every module for every change — the
layers are low-cohesion with respect to how the system actually changes. Slicing by
*domain capability* instead (an "Orders" module that contains its own controller,
logic, and data access; a "Payments" module likewise — vertical slices) means a
feature change lives inside one module. Align the boundaries with the axis of change,
and the axis of change is almost always the domain, not the framework layer.

**Boundaries should follow team boundaries too (Conway's Law).** A system's structure
tends to mirror the communication structure of the org that builds it. If three teams
own one tangled module, they'll constantly step on each other; if each team owns a
cohesive module with a clear interface, they can work independently. So draw
boundaries that a *single team* can own end to end — this is as much an organizational
decision as a technical one, and it's why bounded contexts (Lesson 7) and team
topologies (a Leadership-track theme) keep showing up together.

{: .warning }
> **A boundary you can't enforce at runtime is only a suggestion.**
> You can declare "the Orders module must not reach into the Inventory module's
> tables" — but if nothing *stops* a developer from doing it, someone will, under
> deadline pressure, and your clean boundary quietly rots into a big ball of mud. The
> strength of a boundary is how hard it is to violate: a *convention* (a wiki page) is
> weakest; a *code structure* (separate packages/modules with visibility rules,
> enforced by the compiler or a linter/fitness function — Lesson 30) is stronger; a
> *physical* boundary (a separate service that can only be reached over an API, with
> no shared database) is strongest — but that last one costs you the whole distributed
> tax. Choose the weakest boundary that will actually hold for your team's discipline.

**The dependency direction, again.** Boundaries have arrows. Dependencies should point
from the volatile toward the stable, and from details toward abstractions (Lesson 5's
stability principle, Lesson 10's dependency rule). Concretely: your core domain
(stable, valuable) should not depend on your database or your web framework (volatile
details); the details depend on the domain, not the reverse. When you draw a boundary,
also decide which way the dependency crosses it — a boundary with dependencies pointing
the wrong way is barely a boundary at all.

**The cost of a wrong boundary is enormous and compounding.** A wrong internal
boundary is at least fixable (refactor within a monolith). A wrong *service* boundary
is brutal: you've now got two things that should be one, chatting over the network,
sharing a distributed transaction, deployed separately but changed together — a
distributed monolith (Lesson 11). Because splitting is expensive to reverse, the
prudent move is to get boundaries right *while they're still cheap to move* — which is
a strong argument for starting with a well-modularized monolith and only extracting
services once the boundaries have proven stable (Lesson 9).

---

## Lab — Design Exercise

**The situation:** A team is building the backend for a food-delivery app. The
features tangle together: a customer places an order; the system checks restaurant
menu availability, calculates price with promotions, charges payment, assigns a
courier, tracks delivery location, and sends notifications at each step (order
confirmed, courier assigned, delivered). The current code has an `OrderController`, an
`OrderService` (2,000 lines), and one big database the whole thing shares.

**Propose the component boundaries.** Name the modules, say what's inside each (the
cohesion), define the key dependencies between them (which way the arrows point), and
identify which boundaries you'd enforce strongly vs loosely. Explicitly reject the
"slice by technical layer" approach and explain why. Do *not* jump straight to
microservices — decide the boundaries first, deployment second.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is finding domain-aligned, cohesive boundaries and reasoning about
dependencies — <em>before</em> deciding anything about services. A strong answer:
<br><br>
<strong>Reject the layer slice first.</strong> The instinct might be "split
OrderController / OrderService / OrderRepository." That's wrong: every feature ("add a
loyalty discount," "change courier assignment") cuts across all three layers, so you'd
touch all of them for every change — the layers don't align with the axis of change.
The 2,000-line <code>OrderService</code> is a <em>low-cohesion</em> symptom: it holds
ordering, pricing, payment, courier, tracking, and notification logic, which change for
six different reasons.
<br><br>
<strong>Slice by domain capability</strong> (vertical, cohesive modules — each owning
its logic <em>and</em> its data):
<ul>
<li><strong>Ordering</strong> — the order lifecycle (create, confirm, cancel), the
order as the aggregate. Owns order state.</li>
<li><strong>Catalog / Menu</strong> — restaurants, menus, item availability. Changes on
a restaurant's schedule, not an order's.</li>
<li><strong>Pricing & Promotions</strong> — price calculation, discounts, promo rules.
Changes constantly (marketing), for reasons unrelated to order mechanics — strong case
for its own boundary.</li>
<li><strong>Payments</strong> — charging, refunds, the payment-gateway integration.
Different change rate, different security/compliance concerns, integrates an external
system (an anti-corruption layer belongs here) — clearly its own module.</li>
<li><strong>Delivery / Dispatch</strong> — courier assignment and live location
tracking. This one has a genuinely different technical profile (real-time,
high-frequency location updates) — a candidate to isolate not just for cohesion but
for its runtime characteristics.</li>
<li><strong>Notifications</strong> — sending confirmations/updates across channels. A
classic <em>supporting</em> capability; cohesive and cross-cutting; a good candidate to
decouple via events (it just reacts to things that happened).</li>
</ul>
<strong>Dependencies (arrow direction).</strong> Ordering orchestrates but should
depend on the <em>others' interfaces</em>, not their internals: Ordering asks Catalog
"is this available?", asks Pricing "what's the total?", tells Payments "charge this,"
tells Delivery "assign a courier." Notifications should <em>not</em> be called
directly by everyone (that couples every module to it); instead the others emit domain
events ("OrderConfirmed," "CourierAssigned," "Delivered") and Notifications subscribes
— so the arrow points <em>from</em> Notifications <em>to</em> the events, and nobody
depends on Notifications. Pricing and Catalog are leaf-ish (things depend on them, they
depend on little) — stable, high-afferent; keep them clean.
<br><br>
<strong>Enforcement strength.</strong> Strongest boundaries around Payments (security
and compliance — never let other modules touch payment data or the gateway directly;
enforce via a hard interface and ideally separate data) and around Delivery (different
runtime profile). Notifications enforced via the event boundary (it can't reach back
into others). Between Ordering/Catalog/Pricing, at minimum enforce at the
package/module level with a fitness function ("no module accesses another's tables/
internals") even inside one deployable.
<br><br>
<strong>Deployment second — and probably "modular monolith" for now.</strong> The key
discipline: <em>these are logical boundaries first.</em> The right initial move is a
<strong>modular monolith</strong> — these six as enforced internal modules in one
deployable, sharing a database but with each module owning its own tables and never
reading another's directly. That gives high cohesion and controlled coupling
<em>without</em> the distributed tax, and — because the boundaries are now clean — makes
later extraction of the ones that actually need it (likely Delivery for its real-time
profile, and Payments for isolation) cheap and low-risk. Jumping straight to six
microservices would invent distributed transactions, network failure modes, and
eventual consistency the team hasn't earned yet. <em>Get the boundaries right; defer
the deployment topology.</em> That sequencing is the whole point of the lesson.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Vertical slices vs technical layers | Simon Brown, "Modular Monoliths" — <https://www.youtube.com/watch?v=5OjqD-ow8GE> ; martinfowler.com/bliki/PresentationDomainDataLayering.html |
| Conway's Law | <https://en.wikipedia.org/wiki/Conway%27s_law> |
| Component cohesion & coupling principles | Robert C. Martin, *Clean Architecture* |
| Package/module boundary enforcement (fitness functions) | *Building Evolutionary Architectures*, Ford, Parsons & Kua |

---

## Checkpoint

**Q1.** Why is slicing a system into "controllers / services / repositories"
(technical layers) usually worse than slicing it into "Orders / Payments / Delivery"
(domain capabilities)? Frame the answer in terms of cohesion and the axis of change.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The core reason is <strong>the axis of change</strong>: real changes to a system are
almost always <em>features</em> ("add a discount to orders," "support a new payment
method"), and a feature cuts <em>vertically</em> through the controller, the business
logic, and the data access. So if you've sliced <em>horizontally</em> by technical
layer, every feature change forces you to touch every layer/module — the layers have
<em>low cohesion with respect to how the system actually changes</em> (a "controllers"
module changes for every feature in the whole system, so it changes constantly and for
unrelated reasons). The layers look organized but fight every change.
<br><br>
Slicing by <strong>domain capability</strong> aligns the boundaries with the axis of
change: an "Orders" module contains its own controller, logic, and data access, so an
order-related change lives <em>inside one module</em> — high cohesion (everything about
orders, changing together, for order reasons) and the change is local. A payment change
touches only Payments. The boundaries now match the shape of the work.
<br><br>
Put simply: technical layering optimizes for "all code of type X together," which is
almost never how you change a system; domain slicing optimizes for "all code that
changes together, together," which is exactly how you change a system. (Layering still
has a place <em>within</em> a module — an Orders module can be internally layered — but
it's the wrong <em>primary</em> boundary, because it's orthogonal to the direction the
requirements move.)
</details>

**Q2.** What does "a boundary you can't enforce at runtime is only a suggestion" mean,
and what's the spectrum of enforcement strength from weakest to strongest? Why not
always pick the strongest?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
It means: declaring a boundary ("module A must not touch module B's internals") does
nothing on its own — if the system permits the violation, then under deadline pressure
someone <em>will</em> reach across it, and the boundary silently erodes until the clean
design has rotted into a big ball of mud. The <em>strength</em> of a boundary is how
hard it is to violate, not how clearly it's documented.
<br><br>
The spectrum, weakest → strongest:
<ul>
<li><strong>Convention</strong> — a wiki page or team agreement ("please don't cross
this line"). Weakest: nothing stops a violation; relies entirely on discipline and
memory, and erodes as the team grows or rushes.</li>
<li><strong>Code structure</strong> — separate packages/modules with visibility rules,
enforced by the compiler, the build, a linter, or an architecture fitness function that
fails the build on a forbidden dependency. Stronger: violations are caught
mechanically, not by hope.</li>
<li><strong>Physical / process boundary</strong> — a separate service reachable only
over an API, with no shared database. Strongest: crossing the boundary illicitly is
essentially impossible because the internals aren't even <em>reachable</em> (different
process, different data store).</li>
</ul>
<strong>Why not always pick the strongest:</strong> because the strongest boundary
(separate service) costs you the entire distributed-systems tax — network calls,
partial failure, eventual consistency, separate deployment and operations, harder
debugging (Phase 4). That price is worth paying only when you actually need
independent deployment/scaling/isolation. For most internal boundaries, a
code-structure boundary (enforced modules within one deployable — the modular
monolith) gives you real, mechanically-enforced separation at a fraction of the cost.
The rule: choose the <em>weakest boundary that will actually hold</em> for your team's
discipline and your real isolation needs — strong enough that it won't rot, but not so
strong that you're paying the network tax for a line that a compiler check could have
enforced.
</details>

---

## Homework

Draw the *actual* component boundaries of your current system as they exist today
(not as the docs claim) — where are the real seams, and where has everything leaked
together? Identify one boundary that has *rotted* (internals leaking across it, so
"internal" changes break other parts) and diagnose why it rotted: was it never
enforced beyond convention? Was it in the wrong place (against the axis of change) to
begin with? Then propose either the enforcement that would have held it, or the
re-drawn boundary that would align with how the system actually changes.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
This exercise builds the habit of reading a system's <em>real</em> structure, which
usually differs from its intended one. A strong response:
<br><br>
<strong>Maps the boundaries as they actually are.</strong> The valuable and common
finding is a gap between the intended architecture (the tidy diagram) and the real one
(modules that supposedly have interfaces but that everyone reaches around; a "shared"
database table that six modules read and write, quietly coupling them all). Learning to
see the <em>de facto</em> boundaries — often by asking "what actually breaks when I
change this?" — is the core skill.
<br><br>
<strong>Diagnoses <em>why</em> a boundary rotted, and it's usually one of two
causes.</strong> Either (a) it was <em>only a convention</em> — nothing enforced it, so
under pressure people crossed it (fix: add real enforcement — a module/package
structure with a fitness function, or, if the isolation genuinely warrants it, a
physical boundary), or (b) it was in the <em>wrong place</em> — drawn against the axis
of change (e.g., a technical-layer boundary, or a domain split that turned out to have
two capabilities that always change together), so people crossed it constantly because
the real work demanded it (fix: re-draw the boundary to match how the system actually
changes — merge things that always change together, split things that don't). The
distinction matters because the fixes are opposite: a right-place-but-unenforced
boundary needs a guardrail; a wrong-place boundary needs to be moved, and adding
enforcement to a wrong-place boundary just makes the friction worse.
<br><br>
<strong>The most instructive finding</strong> is usually a shared database as the
hidden coupling — modules that look separate in the code but are welded together
through common tables, so no code-level boundary can hold because the data boundary is
missing. Recognizing that <em>data ownership</em> is where boundaries most often leak
(each module should own its own data, Lesson 24 on data ownership) is a step toward the
deeper structural work of Phases 3–5. A good answer names whether the fix is a cheap
enforcement addition, a boundary re-draw, or the harder job of untangling shared data —
and is honest that the last is a real project, not a refactor.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-module-component"></a>**module / component** — a named unit of the system with a clear responsibility and a boundary; the building block you draw lines around.
- <a id="w-boundary"></a>**boundary** — the line separating what's inside a module from what's outside; where the interface (contract) lives.
- <a id="w-change-axis"></a>**change axis** — the direction along which requirements evolve; good boundaries align with the things that change together.
- <a id="w-seam"></a>**seam** — a place where you can alter behavior without editing in place; a natural point to split.
- <a id="w-leak"></a>**leak** — when a module's internals escape across its boundary, so outsiders depend on them.
- <a id="w-runtime-enforced-vs-convention"></a>**runtime-enforced vs convention** — a boundary the system actually prevents crossing vs one that's just a team agreement (easy to erode).

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 7 — Domain-Driven Design Essentials →](lesson-07-domain-driven-design){: .btn .btn-primary }
