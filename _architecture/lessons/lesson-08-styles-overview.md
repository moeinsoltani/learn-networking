---
title: "Lesson 08 — A Map of Architectural Styles"
nav_order: 4
parent: "Phase 2: Foundations of Structure"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 08: A Map of Architectural Styles

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

Before Phase 3 dives into individual styles, you need the *map* — the menu you're
choosing from, and the one thing each option optimizes for. An **architectural style**[°](#w-architectural-style)
is a proven high-level shape for a whole system. Crucially, each style is *for*
something (Lesson 3's quality attributes): it makes some qualities easy and pays for
them by making others hard. There is no "best style," only "best style for these
drivers."

Architectural styles are easier to hold in your head if you take the **first
fork** before looking at the menu: **is this one deployable unit, or many?**

Answering "one" puts you among the **monolithic** styles:

| Style | What it is for |
|---|---|
| **Layered (n-tier)** | Simplicity, and clean separation into layers |
| **Modular monolith** | Enforced module boundaries — modularity *without* the distribution tax |
| **Microkernel (plug-in)** | Extensibility: a small core plus plug-ins |

Answering "many" puts you among the **distributed** styles:

| Style | What it is for |
|---|---|
| **Service-based** | Coarse-grained services — a few, not dozens |
| **Event-driven** | Decoupling and scale through asynchronous messages |
| **Microservices** | Independent deployment of many small services |
| **Space-based** | Extreme scale, by removing the shared database bottleneck |

Two things are worth noticing. The first fork is the expensive one — everything
below "distributed" comes with the network, and Lesson 14's fallacies apply to
all of them. And **choosing no style at all is itself a choice**: what it
produces, reliably, is the big ball of mud.

Two truths frame everything in Phase 3. First, **"monolithic vs distributed" is the
biggest fork** — it determines whether you pay the distributed-systems tax (Phase 4) at
all, and it's the decision with the largest consequences. Second, **most real systems
are hybrids** — a modular monolith with one service extracted, or microservices with an
event-driven backbone. The styles are a vocabulary and a set of trade-off profiles, not
boxes you must fit into purely.

## Going Deeper

**The styles and what each is *for* (the one-line version):**

- **Layered / n-tier** — organize by technical layer (presentation → business → data).
  *For:* simplicity, familiarity, a clear default. *Against:* the domain can end up
  coupled to the database; changes cut across layers (Lesson 6). The most common
  starting monolith. (Lesson 10.)
- **Modular monolith** — one deployable, but with enforced internal module boundaries.
  *For:* cohesion and low coupling *without* the distributed tax; the underrated best
  default. (Lesson 9.)
- **Pipeline / pipes-and-filters** — data flows through a sequence of processing steps.
  *For:* data-transformation workflows, ETL, compilers. Simple, composable.
- **Microkernel / plug-in**[°](#w-microkernel-plug-in) — a minimal core plus plug-ins that add features. *For:*
  extensibility and product customization (IDEs, browsers, tools with an ecosystem).
- **Service-based** — a handful of coarse-grained services (not dozens of tiny ones),
  often sharing a database. *For:* some independent deployability with far less
  complexity than microservices; a pragmatic middle ground.
- **Event-driven** — components communicate by emitting and reacting to events. *For:*
  decoupling, extensibility, and scale; *against:* legibility (no linear flow to read)
  and eventual consistency. (Lesson 12.)
- **Microservices** — many small, independently deployable, bounded-context-sized
  services. *For:* independent deploy/scale/tech and team autonomy; *against:* the full
  distributed tax. (Lesson 11.)
- **Space-based**[°](#w-space-based) — replicate processing and keep data in an in-memory grid to remove
  the database as the bottleneck. *For:* extreme, spiky scale (high-volume sites);
  niche and complex.

**The default you get by not choosing is the worst one.** If no one makes a deliberate
style decision, the system doesn't stay style-less — it becomes a **big ball of mud**[°](#w-big-ball-of-mud):
no discernible structure, every part coupled to every other, changes unpredictable.
This is the natural entropy of software under deadline pressure. Choosing *any*
coherent style and enforcing it (Lesson 6's boundaries) is the defense; the ball of mud
is what wins by default.

{: .note }
> **How to actually choose a starting style (from the drivers)**
> Work from Lesson 4's drivers, in this rough order:
> 1. **Start with the constraints** — team size/skill, timeline, budget, ops maturity.
>    A small team with a tight deadline and no distributed-systems experience should
>    almost never start with microservices, whatever the tech fashion.
> 2. **Then the top quality attributes.** Need extreme independent scaling and team
>    autonomy across many teams? That pushes toward distributed. Need simplicity,
>    strong consistency, and speed of a small team? That pushes toward a (modular)
>    monolith.
> 3. **Default to the simplest style that meets the drivers**, and prefer one you can
>    *evolve* (a modular monolith → extract services later) over one you can't easily
>    walk back. "Monolith first" (Fowler) is the honest default for most new systems;
>    you distribute when a driver *forces* it, not preemptively.

**Hybrids are normal — don't be a purist.** Real systems mix styles because different
parts have different drivers. A modular monolith with the one real-time, high-scale
component extracted as a service. Microservices with an event-driven backbone for
integration and a couple of synchronous request/response paths where latency demands
it. A layered structure *inside* each microservice. The map isn't a set of mutually
exclusive religions; it's a palette. The skill is matching the style (or blend) to each
part's drivers, and being able to *say why*.

---

## Lab — Design Exercise

**The situation:** Pick a starting architectural style for each of three products, and
justify it *from the drivers* (Lesson 4) — not from fashion.

1. **A startup's MVP:** a 3-person team, 3-month runway to prove the idea, unknown but
   probably-modest initial traffic, requirements changing weekly.
2. **A bank's core ledger:** correctness is paramount (money must never be wrong),
   strong regulatory/audit requirements, moderate and predictable load, a large
   experienced team, expected to run for 20 years.
3. **A media-streaming backend:** tens of millions of users, massive and spiky traffic,
   many teams working in parallel, needs independent scaling of very different workloads
   (recommendations vs streaming vs billing), mature ops/DevOps org.

**For each, name the style (or hybrid) you'd start with and the drivers that decided
it.** Note where you'd deliberately choose simplicity over the "impressive" option.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is deriving the style from constraints and quality attributes, and resisting
one-size-fits-all. A strong answer:
<br><br>
<strong>1. Startup MVP → modular monolith (or even a simple layered monolith).</strong>
Drivers: tiny team, tight deadline, requirements changing weekly, unproven idea, modest
traffic. Every one of these argues for <em>simplicity and speed of change</em>: one
deployable, no network hops, strong consistency, easy refactoring as the (still-unknown)
domain shifts weekly, no distributed tax a 3-person team can't afford to operate. A
<em>modular</em> monolith (enforced internal boundaries) over a plain one, so that
<em>if</em> the idea succeeds and one part needs to scale, extraction is cheap — you buy
future optionality for almost no present cost. Choosing microservices here would be the
classic mistake: spending the scarce 3-month runway building distributed infrastructure
instead of finding product-market fit, for scale that may never come. This is the
clearest "choose simplicity over impressive" case.
<br><br>
<strong>2. Bank core ledger → a monolith or a small set of coarse services, layered
internally, strong consistency throughout.</strong> Drivers: correctness paramount,
heavy audit/regulatory needs, moderate <em>predictable</em> load, experienced team,
20-year lifespan. The dominant quality attributes are <em>correctness/consistency,
auditability, and longevity</em> — <em>not</em> extreme scale (load is moderate). That
profile argues <em>against</em> distributing the ledger: you want ACID transactions and
strong consistency (a distributed ledger reintroduces the hardest consistency problems
in exactly the place you can least afford to get money wrong), a legible, auditable flow,
and boring, durable technology (Lesson 33). So: keep the core ledger transactional and
consistent (monolithic or a very small number of coarse services with a shared,
strongly-consistent store); event-source or heavily audit-log for the regulatory trail
(Lesson 20's event sourcing is a genuinely good fit here — immutable history). Don't
chase microservices for a system whose driver is correctness-at-moderate-scale;
distribution would <em>add</em> risk to the one quality that matters most.
<br><br>
<strong>3. Media-streaming backend → microservices with an event-driven backbone (a
**hybrid**[°](#w-hybrid)).</strong> Drivers: tens of millions of users, massive spiky traffic, many
parallel teams, wildly different workloads needing independent scaling, mature ops. Now
the drivers finally justify distribution: <em>independent scalability</em> (streaming,
recommendations, and billing have totally different load profiles and must scale
separately), <em>team autonomy</em> (many teams need to deploy independently —
microservices are as much an org solution as a technical one), and <em>fault
isolation</em> (recommendations failing shouldn't stop playback). The mature ops org
means they can actually <em>pay</em> the distributed tax (deployment automation,
observability — the "you must be this tall" prerequisites of Lesson 11). Event-driven
backbone for the integration and analytics paths (decoupling, scale), with synchronous
paths where latency demands it (serving a stream). This is the one case where "the
impressive option" is actually correct — because the drivers demand it, not because it's
fashionable.
<br><br>
<strong>The meta-lesson:</strong> the <em>same</em> question ("what style?") gets three
different right answers because the drivers differ — and the most common industry
mistake is applying product 3's answer (microservices) to product 1's situation
(startup MVP), copying the shape without the drivers that justify it. Start from the
constraints and top quality attributes; default to the simplest style that meets them;
distribute only when a driver forces it.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Architectural styles catalog & trade-off profiles | *Fundamentals of Software Architecture*, Richards & Ford (Part II) |
| "Monolith First" | Martin Fowler — <https://martinfowler.com/bliki/MonolithFirst.html> |
| Big Ball of Mud | Foote & Yoder — <http://www.laputan.org/mud/> |
| Choosing an architecture from quality attributes | *Software Architecture in Practice*, Bass, Clements & Kazman |

---

## Checkpoint

**Q1.** What is the "big ball of mud," why is it the *default* outcome, and what makes
it the thing every architectural style is fundamentally trying to prevent?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The <strong>big ball of mud</strong> is a system with no discernible architecture: no
clear structure or boundaries, every part coupled to every other, data and control flow
tangled, so changes are unpredictable (touch one thing, break something unrelated) and
the system becomes progressively harder and riskier to modify. It's the absence of
structure, not a structure.
<br><br>
It's the <strong>default outcome</strong> because software under real-world pressure
tends toward entropy: without deliberate, <em>enforced</em> boundaries, each deadline
produces one more shortcut — a module reaching into another's internals "just this once,"
a bit of logic dropped wherever it was convenient — and these accumulate. Nobody decides
to build a ball of mud; it's what you get by <em>not</em> deciding and not enforcing.
Structure decays unless something actively maintains it, so "no chosen style" doesn't
stay neutral — it degrades into mud.
<br><br>
Every architectural style is fundamentally a defense against this: each imposes a
<em>discipline</em> — layers with rules about what can call what, modules with enforced
boundaries, services separated by process and network — that resists the entropy. The
specific quality a style optimizes for (scalability, deployability, etc.) varies, but
the <em>baseline</em> job they all share is providing a structure coherent enough to
keep coupling under control as the system grows and changes. That's why "choose a style
and enforce it" matters even when no single style is objectively best: any coherent,
enforced structure beats the mud, and the mud is what wins by default. (And note the
word <em>enforced</em> — from Lesson 6: an unenforced style is just a suggestion the mud
eventually overrides.)
</details>

**Q2.** Someone says "we should use microservices because that's how modern systems are
built." Using the idea that each style optimizes for specific drivers, give the reasoning
for when that's right and when it's a mistake.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The framing to correct: microservices aren't "modern" vs "old-fashioned" — they're a
style that <strong>optimizes for a specific set of drivers</strong> and pays a heavy
price for them. What they optimize for: <em>independent deployability</em> (teams ship
without coordinating), <em>independent scalability</em> (scale each workload
separately), <em>independent technology choices</em>, <em>fault isolation</em>, and —
crucially — <em>team autonomy at organizational scale</em> (microservices are as much an
org solution as a technical one). What they cost: the entire distributed-systems tax —
network failure modes, eventual consistency, distributed transactions/sagas, much harder
debugging and testing, and heavy operational prerequisites (deployment automation,
observability, on-call maturity — the "you must be this tall to ride").
<br><br>
<strong>When it's right:</strong> when the drivers actually demand those benefits and the
org can pay the tax — many teams that block each other deploying, real and
<em>differential</em> scale pressure (workloads that must scale independently), and a
mature DevOps/ops capability. The media-streaming backend (many teams, tens of millions
of users, wildly different workloads, mature ops) is a genuine fit.
<br><br>
<strong>When it's a mistake:</strong> when those drivers are <em>absent</em> — a single
small team (no coordination problem to solve, so no autonomy benefit), modest or uniform
load (no differential-scaling benefit), immature ops (can't pay the tax), or a tight
deadline / unproven product (the distributed infrastructure burns the runway you need for
the actual product). Adopting microservices here imports all the cost and none of the
benefit, and typically produces a <em>distributed monolith</em> (services still coupled,
so you get the network tax <em>and</em> the coupling — the worst outcome, Lesson 11). The
correct response is: "microservices solve org-and-scale problems; which of those do we
actually have? If the answer is 'none yet,' the modern, disciplined choice is a modular
monolith we can evolve — and we distribute the specific parts that a driver later
forces." The word "modern" is doing the work of an argument that should be made from the
drivers.
</details>

---

## Homework

Identify the architectural style(s) your current system actually uses (as opposed to
what anyone intended) — and whether it's a coherent style or has drifted toward a big
ball of mud in places. Then, for the *dominant* quality attributes your system truly
needs (from Lesson 3's homework), assess: does the style you have match the drivers you
have? Name one place where the style is a good fit for the drivers and one where there's
a mismatch (a distributed design serving drivers that didn't need distribution, or a
tangled monolith straining against a driver that now demands separation).

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
This connects the whole map to a real system. A strong response:
<br><br>
<strong>Names the <em>de facto</em> style honestly.</strong> The useful finding is
often that the system is a <em>hybrid</em> (or a drifted version of an intended style):
"nominally a layered monolith, but with a couple of services bolted on and a big ball of
mud in the older core." Seeing the real style — including the mud where structure has
decayed — rather than the aspirational diagram is the skill, and it usually reveals that
different <em>parts</em> of the system are in different states.
<br><br>
<strong>Checks style against drivers, both directions.</strong> The two mismatches to
look for are the two classic failures: (1) <em>over-distribution</em> — a part that was
split into services (or microservices adopted wholesale) for drivers that didn't require
it (no independent-scaling need, single team), now paying the distributed tax for no
benefit, possibly a distributed monolith; and (2) <em>under-structuring</em> — a part
that's a tangled monolith straining against a driver that has <em>grown</em> into
existence (a component that now genuinely needs independent scaling or team autonomy, or
that's become a mud ball blocking all change). Finding one of each — where the style fits
and where it fights the drivers — is the goal, and it directly sets up the rest of the
track: the over-distributed part is a candidate to consolidate; the under-structured part
is a candidate for the boundary work (Lesson 6) and, if a driver truly demands it,
extraction (Phase 3 / Lesson 32's strangler fig). The deeper takeaway many people reach:
the right style isn't fixed — it should track the drivers, which <em>change</em> as the
system and org grow, so "the style that fit at year one" may be the mismatch at year
three (the bridge to evolutionary architecture, Lesson 31).
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-architectural-style"></a>**architectural style** — a recurring high-level shape for a whole system (layered, microservices, event-driven…), each optimizing for a different quality.
- <a id="w-monolithic-vs-distributed"></a>**monolithic vs distributed** — one deployable unit vs many communicating over a network; the first big fork.
- <a id="w-big-ball-of-mud"></a>**big ball of mud** — the "style" you get by not choosing one: no discernible structure, everything coupled to everything.
- <a id="w-microkernel-plug-in"></a>**microkernel / plug-in** — a small core plus plug-in modules that extend it.
- <a id="w-space-based"></a>**space-based** — a style that removes the database bottleneck using in-memory data grids for extreme scale.
- <a id="w-hybrid"></a>**hybrid** — a mix of styles; the norm in real systems.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 9 — Monoliths & the Modular Monolith →](lesson-09-monoliths){: .btn .btn-primary }
