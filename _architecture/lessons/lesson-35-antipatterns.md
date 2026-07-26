---
title: "Lesson 35 — Architecture Anti-Patterns & Pitfalls"
nav_order: 3
parent: "Phase 8: The Architect in Practice"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 35: Architecture Anti-Patterns & Pitfalls

{: .note }
> **Words to know**
> - **anti-pattern** — a common "solution" that looks reasonable but reliably makes things worse; worth knowing by name so you catch it early.
> - **big ball of mud** — a system with no discernible structure; everything depends on everything. The default you get by not choosing.
> - **distributed monolith** — microservices that are so coupled they must deploy together — the costs of both styles, the benefits of neither.
> - **accidental complexity** — complexity you added (over-engineering, gold-plating), as opposed to complexity the problem demands.
> - **gold-plating / speculative generality** — building more (or more general) than anyone asked for, "just in case".
> - **YAGNI** — "You Aren't Gonna Need It" — the counter to speculative building (Lesson 31).
> - **premature optimization / scaling** — optimizing or scaling before you have evidence you need to; a debt, not a virtue.
> - **analysis paralysis** — deliberating so long (chasing certainty) that you never decide; the opposite failure to reckless decisions.
> - **leaky abstraction** — an abstraction that fails to hide its underlying details, so callers must know them anyway.

## Concept

Most architectural disasters aren't novel — they're the *same handful of failure modes*, repeated. The
value of learning them **by name** is that a named pattern is one you can *recognize early* — ideally in
your own design, on paper, before it's built — and correct while it's still cheap. This lesson is a field
guide to the classics: what each looks like, why it harms, and the corrective. And the most important
place to spot them is **in your own designs first** — every one of these felt reasonable to the person who
built it.

```
   THE RECURRING FAILURE MODES (recognize them EARLY, in your own design)

   STRUCTURE GONE          TOO MUCH               TOO LITTLE / TOO LATE
   ┌──────────────────┐    ┌──────────────────┐   ┌──────────────────────┐
   │ big ball of mud  │    │ over-engineering  │   │ premature optimization│
   │ distributed      │    │ gold-plating      │   │ premature scaling     │
   │  monolith        │    │ speculative       │   │ analysis paralysis    │
   │ leaky abstraction│    │  generality       │   │ ivory tower           │
   │ accidental       │    │ résumé/hype-driven│   │ (decide too late /    │
   │  vendor lock-in  │    │ 2nd-system effect │   │  never)               │
   └──────────────────┘    └──────────────────┘   └──────────────────────┘
        no structure          more than needed        wrong amount/timing
   corrective: choose         corrective: YAGNI,       corrective: evidence-
   & enforce boundaries       "just enough"            driven, decide & move
```

They cluster into three families. **Structure gone** — the design has lost (or never had) real
boundaries: the *big ball of mud* (no structure at all), the *distributed monolith* (services coupled so
tightly they're a monolith with network latency added), *leaky abstractions*, *accidental vendor
lock-in*. **Too much** — complexity you inflicted on yourself: *over-engineering*, *gold-plating*,
*speculative generality*, *résumé/hype-driven design*, the *second-system effect*. **Too little / too
late** — the wrong amount or timing: *premature optimization* and *premature scaling* (doing it before
the evidence), and *analysis paralysis* and the *ivory tower* (never deciding, or deciding disconnected
from reality). Learn them so you can name them — because you can't correct what you can't see, and the
first place to look is the mirror.

## Going Deeper

**Structure gone: the big ball of mud and the distributed monolith.** The **big ball of mud** is the
system with no discernible architecture — no boundaries, everything coupled to everything, changes
rippling unpredictably. It's the *default* you get by *not* deliberately choosing and enforcing structure
(Lesson 9 — monoliths rot into it without internal modules); the corrective is real, enforced boundaries
(Lessons 6, 30's fitness functions). The **distributed monolith** is arguably worse and more insidious:
you did the expensive work of splitting into microservices, but the services are so tightly coupled
(shared database, synchronous chains, deploy-together dependencies) that you got the *costs* of
distribution (network latency, partial failure, operational complexity — Phase 4) with *none* of the
benefits (independent deploy, scale, fault isolation). It usually comes from splitting by technical layer
or getting granularity wrong (Lesson 13) rather than along real bounded-context seams (Lesson 7). The
corrective is to fix the boundaries — align them with the domain, decouple the data, make services truly
independently deployable — or, honestly, to merge them back into a well-modularized monolith.

**Too much: over-engineering, gold-plating, speculative generality.** This family is **accidental
complexity** (Lesson 31) that you added: **over-engineering** (a solution more complex than the problem
warrants), **gold-plating** (adding polish and features nobody asked for), and **speculative generality**
(building generic, configurable, "flexible" machinery for imagined future needs that usually don't arrive
as guessed). All three are the **YAGNI** violation, and all three are *seductive* because they feel like
craftsmanship and foresight — but they're a permanent tax on everyone who touches the system, and (the
counter-intuitive part from Lesson 31) they often make the system *harder* to change, not easier, because
you've committed to the wrong abstraction. Closely related: **résumé-driven / hype-driven design**
(Lesson 33 — choosing tech for excitement, not fit) and the **second-system effect** (Lesson 32 — the
rewrite that crams in everything the first system lacked). The corrective throughout: *just enough, just
in time*; solve the problem you have, keep it simple and reversible, add complexity when a real need
actually arrives.

{: .warning }
> **Too little / too late: premature optimization, premature scaling, analysis paralysis**
> These are failures of <em>timing and evidence</em>:
> - <strong>Premature optimization</strong> (Knuth's "root of all evil"): optimizing before you've
>   measured where the bottleneck actually is (Lesson 23) — you add complexity and sacrifice clarity to
>   speed up code that wasn't the problem, and often miss the real hotspot. Corrective: measure first,
>   optimize the proven bottleneck.
> - <strong>Premature scaling</strong>: building for a scale you don't have and may never reach —
>   sharding, microservices, multi-region for a product with modest traffic (Lesson 11's team-of-6
>   wanting 20 services). It's <em>also a debt</em>: the complexity is paid now, for certain, against a
>   benefit that's speculative. Corrective: scale when the numbers (capacity thinking, Lesson 22)
>   actually demand it; a monolith on one big box goes remarkably far.
> - <strong>Analysis paralysis</strong>: deliberating endlessly in pursuit of certainty you'll never
>   have (recall Lesson 2 — "it depends," and Lesson 31 — the last responsible moment is a moment, not
>   "never"). The architect must <em>decide and move</em> with the information available, especially on
>   the reversible two-way doors. Corrective: match deliberation to the door-type (Lesson 29); decide the
>   reversible ones fast.
> - The <strong>ivory tower</strong> (Lesson 34): deciding disconnected from the reality of the code —
>   too <em>late</em> to be informed by it and too <em>high</em> to be credible.
> Note the symmetry: the "too much" family over-invests <em>ahead</em> of need; this family either
> over-invests in the wrong place (premature opt/scale) or never invests at all (paralysis). Both miss
> the <em>right amount at the right time</em>, which is the whole architectural skill.

**Leaky abstractions and accidental lock-in.** Two more worth naming. A **leaky abstraction** (Spolsky)
is one that fails to fully hide what it's abstracting, so callers are forced to understand the underlying
details anyway — the abstraction adds ceremony without delivering the encapsulation it promised (an ORM
that hides SQL until you must hand-tune the query it generates). The lesson isn't "never abstract" but to
be honest about what an abstraction really buys and where it leaks. **Accidental vendor lock-in** is the
Lesson 31 leak in a specific costume: a vendor's SDK, format, or assumptions spread through your code
until leaving is prohibitively expensive — lock-in you never *chose*, that shows up as a cost at the worst
possible moment. Corrective: keep volatile/vendor dependencies behind a boundary (ports & adapters,
anti-corruption layer) and *enforce* it with a fitness function (Lessons 30–31).

**Spotting them in your own design first.** The meta-skill: every one of these anti-patterns *felt
reasonable* to the person who created it — the big ball of mud was built one sensible shortcut at a time;
the speculative generality felt like good foresight; the premature scaling felt like being responsible.
So the productive stance is *self-suspicion*: run the "what would break this?" review (Lesson 30) against
your own designs and ask specifically, "which of these named failure modes am I drifting into?" Naming
them gives you the vocabulary to catch yourself — which is far cheaper than having a reviewer, or
production, catch you.

---

## Lab — Design Exercise

**The situation:** A team proposes this design for a **new internal tool** (a dashboard used by ~50
internal staff to manage support tickets). The proposal:
1. **20 microservices** ("tickets", "ticket-status", "ticket-comments", "ticket-tags", "users",
   "user-avatars"…), because "microservices are best practice and we want it to scale," each a separate
   deploy — but they all read and write the *same shared database*, and creating a ticket with its tags
   and initial comment requires synchronous calls across five of them in a chain.
2. A **generic, plugin-based rules engine** with a custom DSL, so that "any future workflow can be
   configured without code," even though the only current requirement is "assign a ticket to a team based
   on its category."
3. Hand-tuned **caching and query optimization** throughout, added up front "for performance," on a tool
   with ~50 users and no measured performance problem.

**Name each anti-pattern, explain the harm, and propose the corrective.**

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is naming each failure mode precisely, explaining <em>why</em> it harms, and giving a
proportionate corrective. A strong answer:
<br><br>
<strong>1. The 20 microservices sharing one database with synchronous chains → distributed monolith (+
premature scaling + wrong granularity).</strong> <em>Name:</em> this is a <strong>distributed
monolith</strong> — the services are split in name but coupled in reality (they share a database, so they
can't evolve their data independently, and a five-service synchronous chain to create one ticket means
they must be up together and deploy together). It's also <strong>premature scaling</strong> (20 services
for a 50-user internal tool with no scale need) and <strong>wrong granularity</strong> (Lesson 13 —
"ticket-status", "ticket-tags", "user-avatars" are nano-services split by data table, not by bounded
context). <em>Harm:</em> you pay the <em>full</em> distributed-systems tax — network latency, partial
failure, operational complexity, distributed data problems — and get <em>none</em> of the benefits
(nothing is independently deployable because of the shared DB and sync chains; there's no scale need to
serve). It's the worst of both worlds, for a tool that a modular monolith would serve trivially.
<em>Corrective:</em> build a <strong>(modular) monolith</strong> — one deployable with clean internal
module boundaries (Lesson 9), one database it owns, in-process calls instead of a synchronous network
chain. If a genuine need to split ever appears, the good internal boundaries make it cheap later
(Lesson 9). Scale when the numbers demand it, not preemptively.
<br><br>
<strong>2. The generic plugin rules engine with a custom DSL → speculative generality / over-engineering
(YAGNI).</strong> <em>Name:</em> <strong>speculative generality</strong> and <strong>gold-plating</strong>
— building a configurable, general-purpose engine for imagined future workflows when the only real
requirement is one simple category→team assignment. <em>Harm:</em> it's a large amount of accidental
complexity (a custom DSL is a language you now own, document, test, and debug forever) built for a future
that's a guess — and per Lesson 31 it likely makes future change <em>harder</em>, because real future
needs won't match the DSL you designed, so you'll fight your own abstraction. It's a permanent tax for a
speculative benefit. <em>Corrective:</em> <strong>YAGNI</strong> — implement the one actual requirement
directly (a simple rule: category → team, a few lines of code or a config map). Add generality only if and
when multiple real, concrete workflow requirements actually arrive and reveal the <em>right</em>
abstraction. "Just enough, just in time."
<br><br>
<strong>3. Up-front hand-tuned caching/query optimization on a 50-user tool with no measured problem →
premature optimization.</strong> <em>Name:</em> <strong>premature optimization</strong> — adding
complexity for performance before measuring that there's a performance problem or where it is.
<em>Harm:</em> you've sacrificed simplicity and clarity (caching adds invalidation/consistency problems,
Lesson 21; hand-tuned queries are harder to read and change) to speed up a system that, at 50 users,
almost certainly has no performance issue — and if it ever did, you've optimized guessed hotspots rather
than measured ones, so you may have missed the real one anyway. <em>Corrective:</em> remove the
speculative optimization; write the simple, clear version; <em>measure</em> if and when performance
actually becomes a concern, then optimize the <em>proven</em> bottleneck (Lesson 23, measure-first).
<br><br>
<strong>The through-line.</strong> All three share a root: <em>solving problems the tool doesn't have</em>
(imagined scale, imagined workflows, imagined load) at the cost of real, immediate, permanent complexity —
the "too much / too early" failure. The unifying corrective is <strong>proportion</strong>: match the
architecture to the <em>actual</em> drivers (Lesson 4) — a 50-user internal tool's drivers are simplicity
and maintainability, not scale — and add complexity only when a real need earns it. A modular monolith,
the one simple rule implemented directly, and no premature optimization would be a <em>better</em>
architecture here, by far. (And a strong answer notes these all "felt responsible" to the proposers —
which is exactly why naming the anti-patterns matters: it lets you catch reasonable-feeling over-engineering
early.)
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Big ball of mud | Foote & Yoder — <http://www.laputan.org/mud/> |
| Distributed monolith / microservices premium | <https://martinfowler.com/bliki/MicroservicePremium.html> |
| YAGNI & speculative generality | <https://martinfowler.com/bliki/Yagni.html> |
| Premature optimization | <https://en.wikipedia.org/wiki/Program_optimization#When_to_optimize> |
| The law of leaky abstractions | Joel Spolsky — <https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/> |
| AntiPatterns (the classic catalog) | <https://en.wikipedia.org/wiki/Anti-pattern> |

---

## Checkpoint

**Q1.** What is a distributed monolith, why is it "the worst of both worlds," and how does it differ from
a big ball of mud?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Distributed monolith:</strong> a system split into multiple services (microservices in name) that
are nonetheless so <em>tightly coupled</em> — a shared database, synchronous call chains, dependencies
that force them to deploy together — that they behave like one monolith with a network in the middle. You
did the expensive work of distributing, but the pieces can't actually move independently.
<br><br>
<strong>Why it's the worst of both worlds:</strong> you pay the <em>full cost</em> of distribution —
network latency on every hop, partial failure and the fallacies of distributed computing (Lesson 14),
distributed-data problems (Lesson 17), and the operational complexity of running many services — while
getting <em>none</em> of the benefits microservices are <em>for</em>: because of the shared database and
synchronous coupling, nothing is independently deployable, independently scalable, or fault-isolated. A
plain monolith at least has simplicity and local transactions; real microservices at least have
independence; the distributed monolith has the drawbacks of both and the advantages of neither. It usually
comes from splitting along the wrong seams — by technical layer, or at the wrong granularity (Lesson 13)
— instead of along real bounded contexts (Lesson 7). Corrective: fix the boundaries (align with the
domain, decouple the data, make services genuinely independently deployable) or merge back into a
well-modularized monolith.
<br><br>
<strong>How it differs from a big ball of mud:</strong> a <strong>big ball of mud</strong> is a system
with <em>no discernible structure at all</em> — no boundaries, everything coupled to everything, typically
within one codebase; it's what you get by <em>never deliberately choosing</em> structure, and it rots in
over time. A distributed monolith <em>does</em> have boundaries drawn (the services are separated), but
they're the <em>wrong</em> boundaries or aren't enforced at the data/deploy level, so the coupling
persists <em>across</em> the network. Roughly: the big ball of mud never drew the lines; the distributed
monolith drew them in the wrong places (and added a network). Both fail on coupling, but one from absence
of structure and the other from mis-placed structure — and the distributed monolith is often worse
because the mistake also cost you the distribution tax.
</details>

**Q2.** Contrast the "too much / too early" anti-patterns (over-engineering, speculative generality,
premature optimization/scaling) with analysis paralysis. What's the unifying skill that avoids both
extremes?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>The "too much / too early" family</strong> over-invests <em>ahead of need</em>:
<em>over-engineering</em> and <em>gold-plating</em> (more complexity/features than the problem warrants),
<em>speculative generality</em> (generic, configurable machinery for imagined futures — the YAGNI
violation), <em>premature optimization</em> (adding complexity for performance before measuring the
bottleneck), and <em>premature scaling</em> (building sharding/microservices/multi-region for scale you
don't have). They all feel like craftsmanship or foresight, but they impose <em>certain, permanent</em>
complexity now against a <em>speculative</em> future benefit — and often make the system <em>harder</em>
to change by committing to the wrong abstraction.
<br><br>
<strong>Analysis paralysis</strong> is the opposite failure: deliberating endlessly in pursuit of
certainty you'll never have, so you <em>never decide</em> and never invest — the ivory-tower variant
decides too late and too disconnected to be useful. Where the first family acts too much too early, this
one fails to act at all.
<br><br>
<strong>The unifying skill: proportion — the right amount at the right time.</strong> Architecture is
about matching the investment to the <em>actual</em> drivers (Lesson 4) and the <em>current</em> evidence,
not to imagined futures or to a quest for certainty. Concretely: solve the problem you have, keep it
simple and reversible ("just enough, just in time"), and add complexity only when a <em>real</em> need
arrives — but <em>do decide</em>, matching deliberation to the door-type (Lesson 29): reversible two-way
doors get decided fast (no paralysis), irreversible one-way doors get deliberated (but at the last
<em>responsible</em> moment, not forever). Measure before optimizing; scale when the numbers demand it;
generalize when multiple real needs reveal the right abstraction; and decide-and-move on the reversible
majority. Both extremes miss this — one invests too much too early, the other never invests — and the
architectural judgment is precisely calibrating the amount and the timing to reality.
</details>

---

## Homework

Audit a system you know (yours, ideally) for these anti-patterns — and be honest, because every one of
them felt reasonable to whoever built it. Which of the three families is your system most prone to?
Structure-gone (is there a big ball of mud, a distributed monolith, a vendor lock-in you never chose)?
Too-much (speculative generality, gold-plating, a hyped technology chosen for excitement)? Too-little/late
(premature scaling for traffic you don't have, premature optimization of unmeasured code, or the reverse —
paralysis)? Name the single most costly anti-pattern you're carrying, explain concretely how it harms, and
propose a proportionate corrective. Then turn the mirror on your *own* recent designs: which named failure
mode were you drifting toward, and did anyone (a reviewer, a fitness function, or production) catch it — or
did it ship?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
A strong response uses the vocabulary to see clearly and turns the mirror inward.
<br><br>
<strong>Categorizes honestly.</strong> The value is diagnosing <em>which family</em> the system leans
toward and naming the specific instance — a distributed monolith from a split done along the wrong seams; a
speculative rules engine or over-abstracted framework nobody needed; premature microservices/sharding for
absent scale; or an accidental vendor lock-in that leaked past a boundary. Precise naming (not just "it's
messy") is what makes the problem actionable, and admitting these all <em>felt responsible</em> at the time
is the maturity.
<br><br>
<strong>Explains harm and proposes a proportionate fix.</strong> A good answer ties the anti-pattern to a
concrete cost (the distributed monolith's latency + can't-deploy-independently; the speculative generality's
permanent tax and wrong abstraction; the premature optimization's sacrificed clarity for no measured gain)
and prescribes a <em>proportionate</em> corrective — usually simplifying toward "just enough": merge to a
modular monolith, delete the speculative machinery and implement the real requirement directly, remove
unmeasured optimization and measure-first, or put a leaked dependency back behind an enforced boundary.
<br><br>
<strong>Turns the mirror inward.</strong> The most valuable part: honestly identifying which named failure
mode <em>your own</em> recent design drifted toward, and whether anything caught it. The takeaway a good
answer reaches: architectural disasters are mostly the same recurring failure modes, every one felt
reasonable to its author, and the defense is naming them so you can recognize them early — especially in
your own designs, on paper, where they're still cheap to fix. The unifying cure across almost all of them is
<em>proportion</em>: match the architecture to the real drivers and current evidence, keep it simple and
reversible, and add complexity only when a genuine need earns it — while still <em>deciding</em> rather than
deliberating forever.
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 36 — Capstone: Design a System End to End →](lesson-36-capstone){: .btn .btn-primary }
