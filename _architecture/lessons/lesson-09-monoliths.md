---
title: "Lesson 09 — Monoliths & the Modular Monolith"
nav_order: 1
parent: "Phase 3: Architectural Styles"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 09: Monoliths & the Modular Monolith

{: .note }
> **Words to know**
> - **monolith** — a system deployed as a single unit; all its code runs in one process.
> - **modular monolith** — a monolith with *enforced* internal module boundaries; one deployable, many well-separated modules.
> - **rot / decay** — the gradual loss of internal structure until a monolith becomes a big ball of mud.
> - **in-process call** — a normal function call within one program (fast, reliable) vs a network call between services (slow, can fail).
> - **"monolith first"** — Fowler's advice to start with a monolith and extract services only when a driver forces it.
> - **deployable unit** — the thing you ship and run as one; a monolith has one, microservices have many.

## Concept

The internet says monoliths are legacy and microservices are the future. The truth an
architect must hold: **the monolith is the right default far more often than the
discourse admits.** A monolith isn't a failure state — it's a deliberate style that
optimizes for simplicity, and simplicity is a feature, not an embarrassment.

```
   MONOLITH                         The problem isn't "monolith."
   ┌─────────────────────────┐      The problem is "monolith with no
   │  Orders  Payments        │      internal boundaries" → rots into:
   │  Inventory  Shipping      │
   │  (all in ONE process,     │      BIG BALL OF MUD
   │   ONE deployable,         │      ┌─────────────────────────┐
   │   ONE database)           │      │ everything ↔ everything  │
   └─────────────────────────┘      │  no boundaries, tangled  │
   ✓ simple  ✓ fast calls           └─────────────────────────┘
   ✓ transactions  ✓ easy refactor
                                     MODULAR MONOLITH (the fix)
   ✗ one deploy for everyone         ┌───────┬───────┬─────────┐
   ✗ scales as one blob              │Orders │Paymnts│Inventory│  ← enforced
   ✗ tempts internal rot             │ [api] │ [api] │  [api]  │    boundaries,
                                     └───────┴───────┴─────────┘    one deployable
```

A monolith's real strengths are substantial: **one deployment** (no orchestration,
versioning, or distributed rollout), **normal function calls** (fast, reliable — no
network), **ACID transactions** across the whole system (the single hardest thing to
give up), **trivial refactoring** across module lines (your IDE does it; across
services it's a coordinated multi-repo migration), and **simple debugging** (one
process, one stack trace, one log). You lose all of these the moment you distribute.

## Going Deeper

**Why monoliths get a bad name: they *rot*.** The legitimate complaint isn't about the
monolithic *deployment* — it's about what monoliths become without discipline. Because
nothing physically stops one part from reaching into another (it's all one process, one
codebase, often one shared database), under deadline pressure the boundaries erode
(Lesson 6) until you have a big ball of mud where every change is dangerous. People then
blame "the monolith" and reach for microservices — but they're treating the *symptom*
(no boundaries) with an expensive cure (physical boundaries via the network) when the
disease (undisciplined coupling) can and will follow them into the distributed world as
a distributed monolith.

**The modular monolith is the antidote.** Keep the single deployable — but enforce
internal module boundaries as if they were service boundaries: each module owns its own
data (its own tables, never read directly by others), exposes a narrow interface, and is
prevented (by package structure + a fitness function, Lesson 30) from reaching into
others' internals. You get high cohesion and controlled coupling — *most* of the
structural benefit people chase microservices for — while keeping the monolith's
simplicity, transactions, and fast calls. This is the underrated best default for the
majority of systems, and it's what "get the boundaries right, defer the deployment
topology" (Lesson 6) produces.

{: .note }
> **"Monolith first" — and why a good modular monolith makes a later split cheap**
> Martin Fowler's advice: for a new system, start with a monolith, because at the
> start you *don't yet know the right boundaries* — the domain is still being
> discovered, and drawing service boundaries early (when you understand least) tends to
> put them in the wrong places, which is brutally expensive to fix once they're
> physical. Build a modular monolith, let the boundaries prove themselves as the domain
> clarifies, and *if* a driver later demands it (a module needs independent scaling,
> a team needs autonomy), extract that module into a service — cheaply, because it's
> already a clean module with its own data and a narrow interface. The modular monolith
> is thus both a great destination *and* the ideal launchpad for selective
> distribution. The failure mode is the reverse: starting distributed, guessing the
> boundaries wrong, and paying forever.

**When a monolith genuinely stops fitting.** It's not never — the honest signals that a
monolith (even a modular one) is straining: (1) *deployment coupling hurts* — teams are
blocked because every change ships together and the deploy is risky/slow; (2)
*differential scaling* — one part needs 50 machines and the rest needs 2, but you must
scale the whole blob; (3) *team autonomy at scale* — many teams contending in one
codebase; (4) *fault isolation* — one component's failure takes everything down and that's
unacceptable; (5) *technology diversity* — one part genuinely needs a different stack.
Note these are the *same drivers* that justify microservices (Lesson 11) — which is the
point: you distribute when a driver forces it, and you extract the *specific* straining
module, not the whole thing at once.

**"But we'll need to scale."** Usually said long before it's true. A well-built monolith
scales further than people think — horizontally (run many identical instances behind a
load balancer, if it's stateless — Lesson 23) and vertically (bigger machines), with the
database typically the real ceiling (which sharding/replication addresses, Lesson 22,
*regardless* of monolith-vs-services). Premature distribution to "prepare for scale" you
may never reach is a debt too (Lesson 2) — you pay the full distributed tax now for a
benefit that's hypothetical.

---

## Lab — Design Exercise

**The situation:** You inherit a 4-year-old e-commerce monolith. It works and serves the
business, but it's become a big ball of mud: any change risks breaking something
unrelated, the `orders` code reads and writes the `inventory` and `pricing` tables
directly, business logic is smeared across controllers and helpers, and the team is
scared to touch it. Leadership, having read about microservices, wants to "break it into
microservices to fix the mess."

**Advise them.** Explain why "microservices to fix a mud ball" is likely the wrong first
move, and design the alternative: how you'd introduce module boundaries *within* the
monolith (a modular monolith) without splitting into services — what you'd separate,
what "each module owns its data" means here given the shared tables, and what would
*enforce* the boundaries. Say what would have to become true before extracting any
actual service.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The core judgment: the problem is <em>missing boundaries</em>, not <em>monolithic
deployment</em> — so fix the boundaries first, and only distribute if a driver later
demands it. A strong answer:
<br><br>
<strong>Why "microservices to fix the mess" is the wrong first move.</strong> The mess
is undisciplined coupling and smeared logic. Splitting a tangled monolith into services
<em>as the way to untangle it</em> is the hardest and riskiest possible path: you'd be
drawing service boundaries through code whose boundaries you don't yet understand (so
you'll draw them wrong), and the existing coupling doesn't vanish — it becomes
<em>distributed</em> coupling (services making chatty synchronous calls into each other,
sharing data, deployed separately but changed together: a distributed monolith — the
worst of both worlds, Lesson 11). You'd also add the entire distributed tax (network
failures, eventual consistency, distributed transactions, harder debugging) <em>on top
of</em> a team that's already scared of the code. You can't extract a clean service from
a module that isn't clean yet.
<br><br>
<strong>The right first move: refactor toward a modular monolith — in place.</strong>
<ul>
<li><strong>Identify the modules</strong> by domain/bounded context (Lesson 7): Orders,
Inventory, Pricing, Catalog, Shipping, etc. — the cohesive capabilities.</li>
<li><strong>Establish module ownership of data.</strong> The shared tables are the
crux. Assign each table to exactly one owning module (Inventory owns the stock tables,
Pricing owns the price tables). The rule: <em>a module may only read/write its own
tables</em>; when Orders needs stock, it calls Inventory's <em>interface</em>, not
Inventory's tables. You don't have to physically split the database yet — you enforce
the <em>logical</em> ownership in code first (no cross-module table access), which is
the hard, valuable part and can be done incrementally.</li>
<li><strong>Extract the smeared logic</strong> out of controllers/helpers into the
owning module's boundary (introduce interfaces — Lesson 5's cheap coupling fixes), so
each module has a clear public API and hidden internals.</li>
<li><strong>Enforce the boundaries mechanically.</strong> Convention won't hold in a team
that's already crossed every line. Use package/module structure with visibility rules,
plus an <strong>architecture fitness function</strong> (Lesson 30) in CI that fails the
build if, say, the Orders package imports Inventory internals or accesses Inventory's
tables. The boundary is only real if the build enforces it (Lesson 6).</li>
</ul>
<strong>Do it incrementally, not as a big-bang rewrite</strong> (Lesson 32): pick the
most painful or most-changing module, carve it out cleanly behind an interface, prove
the pattern, repeat. The system stays running and shippable throughout.
<br><br>
<strong>What would have to become true before extracting a real service:</strong> a
<em>driver</em> must force it — e.g., Inventory needs to scale independently (Black
Friday load hits it differently), or a separate team needs to own and deploy Pricing
without coordinating, or one component's failure must be isolated. <em>And</em> the
module must already be clean (its own data, narrow interface) so extraction is cheap and
low-risk. Once both hold, you extract <em>that one module</em> into a service — a small,
well-understood, reversible step — not the whole monolith at once. The framing for
leadership: "The mess is fixable, and cheaper to fix, <em>inside</em> the monolith. Doing
that first also makes any future service split safe. Jumping straight to microservices
would import all the cost, keep the coupling, and hand a scared team a distributed
debugging problem. Let's make it a clean modular monolith, then distribute only the
specific parts that a real business need forces us to." That advice — treat the boundary
problem, defer the deployment problem — is the whole lesson.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Modular monoliths | Simon Brown — <https://www.youtube.com/watch?v=5OjqD-ow8GE> ; "Majestic Modular Monolith" |
| "Monolith First" / "Microservice Premium" | Martin Fowler — <https://martinfowler.com/bliki/MonolithFirst.html> |
| "Don't start with microservices" | <https://martinfowler.com/articles/dont-start-monolith.html> |
| The monolith's strengths & when to split | *Building Microservices* (2e), Sam Newman (Ch. 3) |

---

## Checkpoint

**Q1.** A team blames "the monolith" for their unmaintainable big ball of mud. Separate
the two things they've conflated, and explain why microservices often fail to fix the
actual problem.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
They've conflated <strong>monolithic deployment</strong> (shipping as one unit) with
<strong>lack of internal boundaries</strong> (undisciplined coupling). These are
independent: you can have a monolith with excellent internal structure (a modular
monolith) and a distributed system with terrible coupling (a distributed monolith). The
big ball of mud is caused by the <em>second</em> — no enforced boundaries, so everything
coupled to everything — not by the fact that it deploys as one unit. The single deployable
is not what makes it unmaintainable; the missing boundaries are.
<br><br>
Why microservices often fail to fix it: the underlying disease is a <em>discipline/
boundary</em> problem, and distributing doesn't cure that — it <em>relocates</em> it and
adds cost. If the team couldn't maintain clean boundaries in one codebase (where a good
IDE, the compiler, and refactoring tools make it <em>easiest</em>), they won't
automatically maintain them across services (where boundaries must be honored over a
network, across repos and teams). The existing tight coupling doesn't disappear on the
way out — it becomes distributed coupling: services that make chatty synchronous calls
into each other and share data, so they must be changed and deployed together anyway (a
distributed monolith) — now with network failures, eventual consistency, distributed
transactions, and much harder debugging piled on top. So you get the mud <em>plus</em> the
distributed tax: strictly worse. The real fix is to <em>introduce and enforce boundaries</em>
— which you can do far more cheaply <em>inside</em> the monolith (turning it modular), and
which is a prerequisite for any successful later split anyway. Reaching for microservices
to fix a mud ball treats an expensive symptom-relocation as a cure.
</details>

**Q2.** Give three genuine signals that a (modular) monolith has stopped fitting and it's
time to extract a service — and explain why the right move is to extract *one straining
module*, not to "split into microservices."

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
Three genuine signals (any of Lesson 11's drivers appearing for real):
<ul>
<li><strong>Differential scaling.</strong> One part of the system needs far more (or
differently-shaped) resources than the rest — e.g., a search or media component needs 50
machines while everything else is happy on 3 — but the monolith forces you to scale the
whole blob together, wasting resources or capping the hot component.</li>
<li><strong>Deployment coupling is hurting.</strong> Every change ships as one unit, so
teams block each other, releases are large and risky, and one component's change can't go
out without re-testing/re-deploying everything — velocity is suffering because of the
shared deploy.</li>
<li><strong>Team autonomy / fault isolation.</strong> Many teams are contending in one
codebase and stepping on each other, or one component's failure takes the whole system
down and that blast radius is unacceptable — a driver that a physical boundary would
actually solve.</li>
</ul>
(Also valid: a genuine need for a different technology stack for one workload.)
<br><br>
Why extract <em>one straining module</em> rather than "split into microservices": (1)
<strong>Only some parts have the driver.</strong> The signals apply to specific
components (the one that needs to scale, the one a team wants to own) — the rest of the
system is served perfectly well by the monolith and would only <em>lose</em> (gain the
distributed tax, lose transactions and simplicity) if split. You distribute where a
driver forces it, and nowhere else. (2) <strong>Risk and reversibility.</strong>
Extracting one clean, well-understood module (which, in a modular monolith, already has
its own data and a narrow interface) is a small, low-risk, reversible step you can
validate before doing the next; a big-bang split into many services at once is a massive,
risky, hard-to-reverse project (Lesson 32's warning against big-bang rewrites). (3)
<strong>You keep the benefits where they still apply.</strong> The result — a monolith
with the one or two genuinely-different workloads carved out as services — is a
deliberate hybrid (Lesson 8) that pays the distributed tax only where it buys something,
keeping the monolith's simplicity everywhere else. "Split into microservices" throws away
the simplicity wholesale to solve a problem that only exists in a couple of places.
</details>

---

## Homework

For your current system: if it's a monolith, honestly rate its internal modularity — is
it a modular monolith or has it rotted toward a mud ball, and where? Identify the two
worst-coupled areas and what enforcing a boundary there would take. If it's already
distributed (services/microservices), ask the harder question: were the boundaries drawn
where real drivers demanded, or did you split things that would have been simpler and
cheaper as one deployable — and are any of your "services" actually a distributed monolith
(deployed separately but always changed and released together)? Write what you'd
consolidate or separate if you could.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise forces an honest read of the deployment-vs-boundary situation. A strong
response:
<br><br>
<strong>For a monolith:</strong> rates modularity by the real test — "when I change
module A, what unrelated things break?" — and locates the worst coupling (very often a
shared database table that several modules read and write directly, welding them together
regardless of code structure). A good answer names what enforcing the boundary would take:
usually assigning each table to one owning module, replacing cross-module table access
with interface calls, and adding a fitness function so the build enforces it — and is
realistic that untangling shared <em>data</em> is harder than untangling shared
<em>code</em>. The takeaway is that most monoliths are fixable in place, and cheaper to
fix that way, which is empowering rather than the "we must rewrite as microservices"
despair the team may have absorbed.
<br><br>
<strong>For a distributed system:</strong> the sharper and more valuable question, and
the uncomfortable common finding is a <strong>distributed monolith</strong> — services
that are deployed separately but must be changed and released <em>together</em> because
they're tightly coupled (chatty synchronous calls, shared database, a change to one
forcing lockstep changes to others). That's the worst case: the full distributed tax with
none of the independent-deployability benefit. Recognizing it (the tell is "we can't
deploy service X without also deploying Y and Z") is the point, and the honest conclusion
is often that some services should be <em>consolidated</em> back into one deployable
(you're paying the network cost for a boundary that isn't buying independence). Equally,
a good answer might find a genuine case for <em>more</em> separation — a module inside a
monolith that a real driver (differential scaling, team autonomy) now justifies extracting.
The meta-lesson: the right number of deployable units is set by drivers, not fashion, and
both over-distribution (distributed monolith) and under-structuring (mud-ball monolith)
are failures to match the deployment topology to the actual boundaries and drivers — and
naming which one your system suffers from is the first step to fixing it.
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 10 — Layered, Hexagonal & Clean Architecture →](lesson-10-layered-hexagonal){: .btn .btn-primary }
