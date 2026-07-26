---
title: "Lesson 05 — Coupling, Cohesion & Connascence"
nav_order: 1
parent: "Phase 2: Foundations of Structure"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 05: Coupling, Cohesion & Connascence

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

Almost every structural decision in architecture reduces to two forces you're
constantly balancing: **cohesion**[°](#w-cohesion) (keep things that belong together, together) and
**coupling**[°](#w-coupling) (keep things that shouldn't depend on each other, independent). Get them
right and the system is changeable — a modification touches one place. Get them wrong
and you have the two classic diseases: *low cohesion* (a module does five unrelated
things, so every change is a scavenger hunt) and *high coupling* (everything depends
on everything, so every change ripples unpredictably).

Two ideas, often confused, describe opposite sides of a boundary. **Cohesion**
is about what happens *within* a module: how well its parts belong together.
**Coupling** is about what happens *between* modules: how much they depend on
each other.

**High cohesion is good.** An `Order` module that creates, prices, and cancels
orders is cohesive — everything in it concerns the lifecycle of an order.
**Low cohesion is bad**, and it is easy to recognise once named: an `Order`
module that creates orders, sends emails, resizes images, and parses CSV files
has become a place where unrelated things happen to live.

**Loose coupling is good.** Module A depends on B only through a stable, narrow
interface — which means B's internals can be rewritten freely without A
noticing. **Tight coupling is bad**: A knows B's internals, so any change to B
breaks A, and the two modules are effectively one module with a misleading
folder structure.

The goal, stated once and applied for the rest of the track: **high cohesion,
loose coupling.** Almost every structural technique in this course is a way of
getting one or both.

The famous heuristic — **"high cohesion, low coupling"** — is the single most
durable design principle in software, older than any framework and true at every
scale: functions, classes, modules, services, teams. When you draw a boundary
(Lesson 6), you're maximizing cohesion inside it and minimizing coupling across it.

## Going Deeper

**You never eliminate coupling — you choose *which* coupling.** "Zero coupling"
means parts that never interact, i.e., not one system but many. Any system that does
something coherent has parts that depend on each other. So the goal isn't *no*
coupling; it's *loose* coupling (dependencies through stable, narrow interfaces you
can change behind) rather than *tight* coupling (dependencies on internals, so any
change ripples). The architect's job is deciding *where* the coupling goes and making
it loose — accepting that you're always trading one form of it for another.

**Afferent vs efferent — direction matters.** *Afferent* coupling is who depends on
you (incoming); *efferent* is who you depend on (outgoing). A module with high
afferent coupling (many things depend on it) is *stable* and must change carefully —
it's load-bearing. A module with high efferent coupling (it depends on many things) is
*unstable* — fragile, since any of its dependencies can break it. A healthy dependency
structure points from unstable/volatile modules *toward* stable/abstract ones (the
Stable Dependencies Principle) — the things everyone depends on should be the things
that change least. This directionality is exactly the "dependency rule" you'll meet in
layered and hexagonal architecture (Lesson 10).

{: .note }
> **Connascence — a precise vocabulary for coupling**
> "Coupling" is a blunt word. **Connascence**[°](#w-connascence) (Meilir Page-Jones) makes it precise:
> two components are connascent if a change in one requires a matching change in the
> other. It comes in *kinds*, roughly ordered weakest → strongest:
> - **Static (visible in the code):** *name* (both agree on a name), *type* (agree on
>   a type), *meaning* (agree on the meaning of a value, e.g. `0` = active),
>   *position* (agree on argument order), *algorithm* (both implement the same
>   algorithm, e.g. a checksum).
> - **Dynamic (only at runtime, harder):** *execution* (order of calls matters),
>   *timing* (timing matters, e.g. a race), *value* (several values must change
>   together to stay consistent), *identity* (both must reference the same instance).
>
> Three rules follow: prefer **weaker** connascence (name over position over
> meaning); prefer **static** over dynamic (compiler-catchable beats runtime
> surprise); and the closer two things are (same function) the stronger the
> connascence you can tolerate, but *across a boundary* (between services) keep it as
> weak and static as possible. It turns "this feels too coupled" into "this is
> connascence of *position across a service boundary* — refactor to connascence of
> *name*."

**Why this is the foundation of everything.** Coupling and cohesion aren't one
lesson's topic — they're the lens for the whole track. Microservices are an attempt
to get low coupling and independent deployability (at the cost of the distributed tax).
Bounded contexts (Lesson 7) are cohesion applied to the domain. A distributed monolith
(the anti-pattern of Lesson 11) is *tight coupling that also pays the network cost* —
the worst of both. Event-driven architecture (Lesson 12) buys loose *temporal*
coupling. When you can name the coupling precisely, you can reason about every style
in these terms instead of by vibes.

---

## Lab — Design Exercise

**The situation:** Two teams implement the same feature — "when an order is placed,
charge the customer and send a confirmation email." Here are their designs:

**Design A:** The `OrderService.placeOrder()` method directly instantiates
`StripeClient`, calls `stripe.charge(order.total, order.customer.creditCardNumber)`,
then instantiates `SmtpMailer` and calls
`mailer.send(order.customer.email, "Order confirmed", buildBody(order))`. All in one
method. `charge()` takes the amount and card number as the 1st and 2nd positional
arguments.

**Design B:** `OrderService.placeOrder()` publishes an `OrderPlaced` event to a
message bus containing the order id. A `PaymentService` and a `NotificationService`
each subscribe. `PaymentService` looks up the order and calls a `PaymentGateway`
interface (implemented by a Stripe adapter). `NotificationService` similarly sends
the email via a `Mailer` interface.

**Analyze the coupling and cohesion of each.** Name the *specific* couplings (use
connascence terms where you can), identify each design's strengths and weaknesses,
and say which is more *changeable* — and, crucially, in *what context* the simpler
Design A might actually be the better call.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The goal is precise analysis, not just "B is better." A strong answer:
<br><br>
<strong>Design A — coupling analysis.</strong> <code>OrderService</code> is
<em>tightly</em> coupled to concrete infrastructure: it depends directly on
<code>StripeClient</code> and <code>SmtpMailer</code> (concrete classes, not
interfaces), so swapping payment provider or mailer means editing
<code>OrderService</code> itself. There's <strong>connascence of position</strong> in
<code>charge(amount, cardNumber)</code> — if the argument order ever changes, every
caller silently breaks (and worse, might charge the wrong amount) — that's a strong,
easily-broken coupling that should be connascence of <em>name</em> (named parameters
or a request object). <strong>Cohesion is low:</strong> <code>placeOrder()</code> does
three unrelated jobs (order logic, payment, email), so it changes for three unrelated
reasons — a violation of single responsibility. There's also a <em>security</em> smell
(the raw card number flowing through order logic). <em>Strengths:</em> it's simple,
synchronous, easy to read top-to-bottom, and easy to debug — one call stack, one
transaction, no infrastructure. If payment or email fails, you know immediately, in
line.
<br><br>
<strong>Design B — coupling analysis.</strong> Coupling is <em>loose</em> and
directed at abstractions: <code>OrderService</code> depends on neither payment nor
email — it just announces a fact (<code>OrderPlaced</code>). <code>PaymentService</code>
depends on a <code>PaymentGateway</code> <em>interface</em> (connascence of
<em>name/type</em>, static and weak), so the Stripe adapter is swappable without
touching the service. <strong>Cohesion is high:</strong> each service does one thing
(orders / payments / notifications), changing for one reason. Coupling across
boundaries is weak and mostly static — the strongest new coupling is <em>temporal
decoupling</em> (the services no longer need to be up at the same instant), which is a
<em>benefit</em> for resilience. <em>Weaknesses:</em> it's a distributed, asynchronous,
event-driven design — you've bought eventual consistency (the email/charge happen
"soon," not "now"), new failure modes (what if <code>PaymentService</code> misses the
event? you now need idempotency, retries, dead-letter handling — Lessons 17/18), and
much harder debugging (no single stack trace; you trace an event across hops). It's a
lot of machinery.
<br><br>
<strong>Which is more changeable?</strong> Design B, clearly — low coupling to
abstractions and high cohesion mean each concern evolves independently (new payment
provider, new notification channel, scaling payments separately) without ripples. On
the pure coupling/cohesion axis, B wins.
<br><br>
<strong>But — the context trap (the real lesson).</strong> Design A is <em>not
wrong</em>; it's wrong <em>at scale and for independent evolution</em>, and right for
<em>simplicity</em>. For a small system, a single team, modest volume, and a hard
deadline, Design A ships faster, is far easier to operate and debug, keeps the
strong-consistency guarantee (charge and order in one transaction), and carries none
of the distributed tax. The <em>coupling problems in A are mostly cheap to fix without
going distributed</em>: introduce a <code>PaymentGateway</code> and <code>Mailer</code>
interface (fixing the concrete-class and position coupling) and extract the two
concerns into cohesive classes — all <em>within a monolith</em>, no message bus, no
eventual consistency. That "middle" design (A's simplicity + B's interfaces and
cohesion, still synchronous and in-process) is often the best answer, and naming it is
the mark of good judgment: <em>you can get high cohesion and loose coupling without
paying the distributed-systems tax</em>. The mistake juniors make is thinking B's
loose coupling <em>requires</em> events and services; it doesn't — loose coupling is
first about interfaces and boundaries, and only sometimes about the network.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Coupling & cohesion (the originals) | Constantine & Yourdon, *Structured Design* |
| Connascence — the full taxonomy | <https://connascence.io/> ; Page-Jones, *What Every Programmer Should Know About OO Design* |
| Afferent/efferent coupling, stability | Robert C. Martin, *Clean Architecture* (the "Component Coupling" principles) |
| Coupling in distributed systems | *Software Architecture: The Hard Parts*, Ford, Richards et al. |

---

## Checkpoint

**Q1.** Why is "eliminate all coupling" a wrong goal, and what's the correct
reformulation? Use afferent vs efferent coupling to describe what a *healthy*
dependency direction looks like.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
"Eliminate all coupling" is wrong because <strong>zero coupling means the parts don't
interact at all</strong> — that's not a coherent system, it's several unrelated
systems. Any system that does something meaningful has parts that depend on each
other; coupling is inherent, not a defect to be removed. The correct reformulation:
make coupling <strong>loose</strong> rather than <em>tight</em>, and put it <em>where
you choose</em> — depend on stable, narrow interfaces you can change behind, not on
another module's internals. You're not removing coupling; you're selecting its form
and location so that changes stay contained.
<br><br>
<strong>Afferent</strong> coupling = incoming dependencies (who depends on me);
<strong>efferent</strong> = outgoing (who I depend on). A <em>healthy dependency
direction</em> points from unstable, volatile, frequently-changing modules
<em>toward</em> stable, abstract, rarely-changing ones. The reasoning: a module that
many others depend on (high afferent coupling) is load-bearing — if it changes, it
breaks everyone, so it must be stable and change rarely. A module that depends on many
others (high efferent coupling) is fragile — any of its dependencies can break it — so
it should be the volatile, easily-replaced edge, not the core. So you want your
most-depended-upon components to be your most stable/abstract ones (interfaces, core
domain), and volatility concentrated at the leaves. Dependencies pointing "up the
stability gradient" (volatile → stable) mean change flows outward-in a contained way;
dependencies pointing the wrong way (a stable core depending on a volatile detail)
mean every little change at the edges shakes the foundation — which is exactly the
problem dependency inversion and hexagonal architecture fix (Lesson 10).
</details>

**Q2.** Two functions in the same class rely on being called in a specific order
(`init()` before `process()`). Name this connascence, say whether it's static or
dynamic, and explain why the *same* coupling would be far more dangerous across a
service boundary than within one class.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
This is <strong>connascence of execution (order)</strong> — the correctness depends on
the <em>order</em> in which the two are executed. It's <strong>dynamic</strong>: you
can't see the requirement by reading either function's signature; it only manifests at
runtime (call them in the wrong order and it breaks, with nothing at compile time to
warn you). Dynamic connascence is generally more dangerous than static because the
compiler/reader can't catch it — it hides until it fails.
<br><br>
Why it's far worse <em>across a service boundary</em> than within one class: two
principles of connascence combine. First, <strong>the strength you can tolerate scales
inversely with distance</strong> — within a single class the two methods are right next
to each other, one author owns both, the ordering is visible and testable locally, and
if it breaks it breaks in one place you control. Across a service boundary the two
sides are in different codebases, likely owned by different teams, deployed
independently, communicating over a network — so the implicit "call A before B" rule is
invisible to the team on the other side, undocumented in any shared contract, and can
be violated by a change they make with no idea it affects you. Second,
<strong>dynamic connascence across a network</strong> also picks up <em>timing</em> and
failure concerns the in-process version never had (what if the first call succeeds and
the second is lost? now the ordering requirement collides with at-least-once delivery
and partial failure — Lesson 17). So the same logical coupling (execution order) that's
a minor, contained, testable smell inside one class becomes a fragile, cross-team,
hard-to-detect, failure-prone contract across services. The rule: keep cross-boundary
connascence as <em>weak and static</em> as possible (ideally just connascence of name
on a stable message contract), and never let dynamic, ordering-dependent coupling span
a service boundary if you can avoid it.
</details>

---

## Homework

Pick one module, class, or service in your current system that's painful to change.
Diagnose it precisely: is the pain from *low cohesion* (it does too many unrelated
things, so changes are scattered and it changes for many reasons) or *tight coupling*
(too many things depend on its internals, so changes ripple), or both? Then use
connascence to name the two or three worst specific couplings, and for each, describe
the refactoring that would move it to a *weaker* and/or *more static* form. Note which
fixes are cheap (interfaces, named parameters) versus which would require a real
structural change.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The goal is to make "this is hard to change" precise and actionable. A strong
response:
<br><br>
<strong>Correctly separates the two diseases.</strong> Low cohesion and tight coupling
feel similar ("hard to change") but have different fixes. Low cohesion → the module
does several unrelated jobs and changes for several unrelated reasons; the fix is to
<em>split</em> it into cohesive pieces (extract the unrelated responsibilities). Tight
coupling → too many things reach into its internals, so changing it breaks callers; the
fix is to <em>introduce a stable interface/boundary</em> and hide the internals. Many
painful modules have both, and naming which is which tells you which refactoring to
reach for.
<br><br>
<strong>Names specific connascence, not just "it's coupled."</strong> Strong answers
find things like: connascence of <em>position</em> (long positional argument lists →
fix with a parameter object or named args: position → name, weaker and clearer);
connascence of <em>meaning</em> (magic values like status codes 0/1/2 scattered around
→ fix with an enum/constant: meaning → name); connascence of <em>algorithm</em> (two
places independently implement the same serialization/checksum and must match → fix by
extracting the single shared implementation); connascence of <em>execution</em> (hidden
ordering requirements → fix by making the API enforce order, e.g. a builder, or
combining the steps). The practice of <em>labeling</em> each coupling is the skill — it
turns a vague "yuck" into a specific, weaker target.
<br><br>
<strong>Distinguishes cheap fixes from structural ones.</strong> The valuable judgment:
some couplings weaken with a local, low-risk refactor (introduce an interface, replace
positional args with a request object, replace magic numbers with an enum — hours of
work, no architectural change) while others require a real structural change (splitting
a low-cohesion service into two, breaking a shared-database coupling, decoupling two
services' deploy cycles — a project). A good answer sequences them: bank the cheap,
weaker-connascence wins first (they immediately reduce the pain and risk), and treat the
structural ones as deliberate, planned changes with their own justification. This mirrors
the real architect's move — you don't rewrite; you incrementally reduce coupling toward
weaker, more static forms, cheapest-first, until the module is changeable again.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-coupling"></a>**coupling** — the degree to which one part of a system depends on another; how much a change *there* forces a change *here*.
- <a id="w-cohesion"></a>**cohesion** — the degree to which the things inside one module belong together; how single-purpose it is.
- <a id="w-connascence"></a>**connascence** (kuh-NAY-sunce) — a precise vocabulary for *kinds* of coupling: two things are connascent if changing one requires changing the other to keep the system correct.
- <a id="w-afferent-efferent-coupling"></a>**afferent / efferent coupling** — incoming dependencies (who depends on *me*) vs outgoing (who *I* depend on).
- <a id="w-static-vs-dynamic"></a>**static vs dynamic** — knowable by reading the code (static) vs only at runtime (dynamic).
- <a id="w-loose-vs-tight"></a>**loose vs tight** — how easily a dependency can be changed or broken without breaking the other side.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 6 — Modularity & Component Boundaries →](lesson-06-modularity-boundaries){: .btn .btn-primary }
