---
title: "Lesson 11 — Microservices"
nav_order: 3
parent: "Phase 3: Architectural Styles"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 11: Microservices

{: .note }
> **Words to know**
> - **microservice** — a small, independently deployable service, owned by one team, sized around a business capability (bounded context).
> - **independently deployable** — you can ship one service without rebuilding or redeploying the others; the defining property.
> - **distributed monolith** — services that must be deployed together because they're tightly coupled; the worst-of-both anti-pattern.
> - **distributed-systems tax** — the added cost microservices impose: network failure, latency, eventual consistency, harder debugging/testing, ops burden.
> - **"you must be this tall"** — Fowler's phrase: prerequisites (automation, observability, org maturity) you need before microservices pay off.
> - **service ownership** — one team owns a service end to end (build, deploy, run, on-call).

## Concept

Microservices are the most hyped and most misapplied style in modern software. An
architect's duty is to understand them *honestly* — the real benefits, and the very
large bill that comes attached. A microservice is a **small, independently deployable
service, owned by one team, sized around a business capability** (usually a bounded
context, Lesson 7). The defining property is the middle one: **independent
deployability**. If you can't deploy your services independently, you don't have
microservices — you have a distributed monolith.

```
   WHAT MICROSERVICES BUY          WHAT THEY COST (the distributed tax)
   ─────────────────────           ─────────────────────────────────
   ✓ independent deploy            ✗ network: calls fail, are slow, retry
     (ship one, not all)           ✗ eventual consistency (no cross-service
   ✓ independent scale               ACID transaction — sagas, Lesson 17)
     (scale the hot one)           ✗ distributed debugging (no single stack
   ✓ independent tech                trace; you need tracing, Lesson 25)
     (right tool per service)      ✗ testing across services is hard
   ✓ fault isolation               ✗ operational burden explodes (many
     (one down ≠ all down)           deploys, versions, monitors, on-call)
   ✓ team autonomy                 ✗ data is fragmented (Lesson 24)
     (org scaling)                 ✗ more moving parts = more failure modes

           Microservices are an ORGANIZATIONAL solution
           as much as a technical one — they scale TEAMS.
```

The honest summary: microservices trade *simplicity* for *independence* — of
deployment, scaling, technology, failure, and teams. That independence is genuinely
valuable **when you need it**, and a pure liability **when you don't** — which is why
the whole lesson is about knowing which situation you're in.

## Going Deeper

**Microservices are as much an org solution as a technical one.** The most
under-appreciated point: the biggest problem microservices solve is *organizational
scaling*. With one monolith, many teams contend in one codebase and one deploy
pipeline — they block each other, coordinate constantly, and step on each other's
changes. Give each team its own independently deployable service and they regain
autonomy: they ship on their own schedule, own their own tech, and don't need a
company-wide release train. This is Conway's Law (Lesson 6) used deliberately. The
corollary is decisive: **if you don't have the org-scaling problem — one small team,
no deployment contention — you don't have the problem microservices primarily solve**,
and you're paying the tax for a benefit you can't use.

{: .warning }
> **The distributed monolith — the worst possible outcome**
> If you split a system into services but they remain tightly coupled — chatty
> synchronous calls into each other, shared database, a change to one forcing lockstep
> changes and deploys of others — you have a **distributed monolith**: you pay the
> <em>full distributed-systems tax</em> (network failures, latency, eventual
> consistency, hard debugging, ops burden) <em>and</em> keep the monolith's coupling
> (can't deploy independently), getting the downsides of both and the benefits of
> neither. It is strictly worse than either a clean monolith or clean microservices.
> The tell: "we can't deploy service X without also deploying Y and Z." It's caused by
> splitting along the wrong boundaries (Lesson 13) — usually splitting a system whose
> parts genuinely belong together, or splitting before the boundaries were understood.

**"You must be this tall to ride" — the prerequisites.** Fowler's "Microservice
Premium": microservices impose a productivity *tax* that only pays off above a certain
complexity, and only if you have the capabilities to operate them. The prerequisites —
without which microservices actively hurt: **deployment automation** (you can't
manually deploy 30 services — you need CI/CD and infrastructure-as-code), **thorough
observability** (you can't debug across services without distributed tracing,
centralized logging, and good metrics — Lesson 25), **operational maturity** (on-call,
incident response, service ownership culture), and often **containers/orchestration**
to manage the fleet. A team without these that adopts microservices spends all its time
fighting operational fires instead of shipping.

**Fault isolation cuts both ways.** A benefit: one service crashing doesn't take down
the others (if you've designed for it — Lesson 18). But the flip side: you've turned
in-process function calls (which don't "fail" independently) into network calls (which
fail, time out, and slow down independently), so you've *added* failure modes. Fault
isolation is real, but it's a benefit you must *engineer* (with timeouts, circuit
breakers, bulkheads, fallbacks) — it isn't free just because the services are separate.
Naively split, you get *more* fragility, not less: a synchronous chain of five services
is *less* available than one monolith, because now any of five can fail the request.

**The pragmatic ladder.** Microservices aren't binary with monoliths. The honest
progression (Lessons 8–9): start with a **modular monolith**; if drivers demand it, move
to **service-based** (a handful of coarse services, maybe still sharing a database) —
much of the benefit, far less tax; and go to **fine-grained microservices** only when the
scale, team count, and operational maturity genuinely justify it. Most systems should
stop at one of the earlier rungs. The mistake is treating "microservices" as the goal
rather than as a specific answer to specific drivers.

---

## Lab — Design Exercise

**The situation:** A team of 6 engineers runs a working modular monolith for a B2B SaaS
product. Traffic is modest and steady (a few hundred requests per second), all from
business customers during working hours. Deploys happen a few times a week and are
generally smooth. The team has read a lot about microservices and has drawn up a plan to
split the monolith into **20 microservices** "to be scalable, modern, and cloud-native."
They deploy manually, have basic logging but no distributed tracing, and no one has run
microservices before. They ask for your blessing.

**Advise them honestly.** Assess whether their drivers justify microservices, name what
the 20-service plan would cost this specific team, identify what (if anything) *would*
justify extracting a service, and give them a better path. Write the pushback in a way
that's honest but doesn't just crush them.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The judgment is matching the style to the drivers and being honest about this team's
readiness. A strong answer:
<br><br>
<strong>Do the drivers justify microservices? Almost none of them are present.</strong>
Go through the real benefits: <em>independent deployability / team autonomy</em> — they're
<em>one team of six</em>, so there's no deployment-contention or org-scaling problem to
solve (the primary thing microservices are for); <em>independent scaling</em> — modest,
steady, business-hours traffic with no described hot spot, so no differential-scaling need;
<em>independent technology</em> — no stated need for polyglot stacks; <em>fault
isolation</em> — not raised as a pain, and a naive synchronous split would <em>reduce</em>
availability, not improve it. The stated goals ("scalable, modern, cloud-native") are
<em>aspirations, not drivers</em> — they're not tied to a concrete problem the current
architecture is failing to solve. And crucially, the deploys are already smooth a few times
a week — the monolith is <em>working</em>.
<br><br>
<strong>What the 20-service plan would cost <em>this</em> team.</strong> They fail every
"you must be this tall" prerequisite: they deploy <em>manually</em> (20 services deployed by
hand is unworkable — they'd need CI/CD and IaC first), they have <em>no distributed
tracing</em> (so the first cross-service bug becomes an un-debuggable nightmare — Lesson 25),
and <em>no one has run microservices</em> (no operational experience with the failure modes).
On top of that, 20 services for 6 people is ~3 services each — an absurd ratio that
guarantees glue work, and splitting a working monolith along boundaries they'll be
<em>guessing</em> at (Lesson 13) is the recipe for a <strong>distributed monolith</strong>:
they'll keep the coupling, add the network tax, and end up worse than today. Realistically
they'd spend the next 6–12 months building distributed-systems plumbing and fighting
operational fires instead of shipping product — an enormous opportunity cost for a product
whose current architecture isn't the bottleneck.
<br><br>
<strong>What <em>would</em> justify extracting a service.</strong> A real driver, not a
vibe: one specific component develops a genuinely different scaling profile (e.g., a
reporting/export job that's CPU-heavy and should scale separately), or the team grows into
2–3 teams that start blocking each other in the codebase, or one component needs fault
isolation because its failure currently takes down critical paths. <em>Then</em> you extract
<em>that one component</em> — after building the prerequisites — not the whole thing.
<br><br>
<strong>A better path (the honest, encouraging version):</strong> "The instinct to invest in
architecture is good, and I don't want to just say no — let me redirect the energy. Right
now microservices would cost us months and buy us almost nothing, because we don't have the
problems they solve (we're one team, traffic is modest, deploys are smooth) and we're missing
the foundations they require (automated deploys, tracing, ops experience). Here's what I'd do
instead: (1) <em>Strengthen the modular monolith</em> — make sure the internal module
boundaries and data ownership are clean (Lesson 9). That's where most of the 'good
architecture' value is, and it's low-risk. (2) <em>Build the prerequisites anyway</em>,
because they're valuable regardless: automate deployments (CI/CD, IaC) and add proper
observability including tracing. These pay off immediately <em>and</em> are exactly what
you'd need later. (3) <em>Watch for a real driver</em> — when one specific part genuinely
needs independent scaling or a second team needs autonomy, we extract that <em>one</em>
service, cheaply, because the boundaries are already clean and the platform is ready. That
way we get scalability and modernity <em>when a real need appears</em>, without betting the
next year on distributed infrastructure for problems we don't have yet." That reframes "no"
as "not yet, and here's the better investment," honors their motivation, and teaches the
driver-first discipline instead of just overruling them (a Lesson 34 / leadership move).
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Microservices — definition & trade-offs | Sam Newman, *Building Microservices* (2e) |
| "Microservice Premium" / "you must be this tall" | Martin Fowler — <https://martinfowler.com/bliki/MicroservicePremium.html> |
| Microservices prerequisites | <https://martinfowler.com/bliki/MicroservicePrerequisites.html> |
| Microservices (overview) | Fowler & Lewis — <https://martinfowler.com/articles/microservices.html> |
| The distributed monolith | *Software Architecture: The Hard Parts*, Ford, Richards et al. |

---

## Checkpoint

**Q1.** What is the single defining property of microservices, what is a "distributed
monolith," and why is a distributed monolith worse than *either* a clean monolith or clean
microservices?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The defining property of microservices is <strong>independent deployability</strong> — you
can build, test, and ship one service without rebuilding or redeploying the others.
(Everything else — small, team-owned, bounded-context-sized — supports this.) If you can't
deploy them independently, you don't have microservices, whatever the diagram says.
<br><br>
A <strong>distributed monolith</strong> is a system split into separate services that are
nonetheless <em>tightly coupled</em> — they make chatty synchronous calls into each other,
share a database, and a change to one forces coordinated changes and lockstep deploys of the
others. The tell: "we can't deploy X without also deploying Y and Z." So they're physically
distributed but logically still one unit — you lost independent deployability, the whole point.
<br><br>
It's worse than <em>either</em> alternative because it combines the costs of both and the
benefits of neither. Versus a <strong>clean monolith</strong>: the monolith at least gives you
simplicity — one deploy, fast in-process calls, ACID transactions, one stack trace to debug.
The distributed monolith throws all that away (network calls that fail and slow down, eventual
consistency, distributed debugging, multi-service ops burden) while <em>still</em> forcing you
to deploy everything together — so you paid the entire distributed-systems tax and got none of
the simplicity. Versus <strong>clean microservices</strong>: real microservices at least buy
independent deployment, scaling, and team autonomy in exchange for the tax. The distributed
monolith pays the same tax but, because the services are coupled, delivers <em>none</em> of that
independence. So on every axis it's dominated: more cost than a monolith with the same coupling,
or the same cost as microservices with none of the benefit. It's the specific failure of
"microservices done wrong," and it's usually caused by splitting along the wrong boundaries
(Lesson 13) or before the boundaries were understood.
</details>

**Q2.** Explain "you must be this tall to ride" for microservices. Name the key
prerequisites and why a team lacking them is made *worse off*, not just "less optimal," by
adopting microservices.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
"You must be this tall to ride" (Fowler's Microservice Premium) means microservices impose a
<strong>productivity tax</strong> that only pays off above a certain scale/complexity <em>and</em>
only if you have the operational capabilities to run a distributed system. Below that bar, or
without those capabilities, you pay the tax and don't get the return — so the style makes you
slower, not faster.
<br><br>
Key prerequisites: (1) <strong>Deployment automation</strong> — CI/CD and infrastructure-as-code;
you cannot manually deploy and manage dozens of services. (2) <strong>Observability</strong> —
distributed tracing, centralized/structured logging, and good metrics; without them you can't
diagnose problems that span services (Lesson 25). (3) <strong>Operational maturity</strong> —
service ownership, on-call, incident response; each service is a thing to run, monitor, and wake
up for. (4) Often <strong>containers/orchestration</strong> to manage the fleet, and an
<strong>org structure</strong> of autonomous teams to actually use the independence.
<br><br>
Why a team lacking these is made <em>worse off</em>, not merely "less optimal": microservices
don't just fail to deliver benefits — they <em>actively add</em> costs the team can't absorb.
Without deployment automation, releasing many services by hand becomes a constant, error-prone
grind that consumes the team. Without tracing/observability, the <em>first</em> cross-service bug
(and there will be many, because you've replaced reliable in-process calls with failure-prone
network calls) becomes nearly un-debuggable — engineers burn days chasing problems that a single
stack trace would have shown instantly in a monolith. Without operational maturity, the explosion
of moving parts and new failure modes produces incidents the team isn't equipped to handle. So
the team ends up spending its time firefighting distributed-systems and deployment problems
<em>instead of</em> building product — strictly worse than the monolith they left, where those
problems didn't exist. That's the difference between "suboptimal" (you'd do slightly better with
another choice) and "worse off" (the choice imposes new, ongoing harm): microservices without the
prerequisites don't just under-deliver, they impose a distributed-systems operational burden that
a team without the tooling and experience will drown in. The prescription is therefore either
<em>don't adopt them yet</em> or <em>build the prerequisites first</em> — never adopt them
<em>and</em> lack the foundations.
</details>

---

## Homework

If your system is microservices (or moving that way), audit it against this lesson: (1)
Can you genuinely deploy each service independently, or is any subset a distributed monolith
(always deployed together)? (2) Which of the real benefits — independent deploy, scale, tech,
fault isolation, team autonomy — is each service split actually *buying*, and is that benefit a
driver you have or one you imagined? (3) Do you have the "you must be this tall" prerequisites,
or are you paying the tax without the foundations? If your system is a monolith, instead write
the honest list of what would have to become true (which specific driver, plus which
prerequisites) before you'd extract your first service — so "microservices" becomes a
conditional decision, not an aspiration.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise turns the lesson into an honest audit of a real system. A strong response:
<br><br>
<strong>For an existing distributed system:</strong> the most valuable (and common) finding
is at least one cluster of services that's really a <strong>distributed monolith</strong> —
identified by the independent-deploy test ("can we ship service X alone?" → "no, it always
goes out with Y and Z"). Naming that honestly is the point: it means those services should
probably be <em>consolidated</em>, because you're paying the network tax for a boundary that
delivers no independence. The benefit-audit (question 2) frequently reveals services that were
split for <em>imagined</em> drivers — "it seemed cleaner," "microservices are best practice" —
rather than a real need, buying nothing while adding cost; those are also consolidation
candidates. And the prerequisites check often exposes a team running microservices
<em>without</em> adequate tracing or deployment automation, which explains a lot of their
operational pain. The overall takeaway is usually "we have some genuine microservices earning
their keep, and some over-splitting we'd undo if we could" — a nuanced, driver-based read
rather than "microservices good/bad."
<br><br>
<strong>For a monolith:</strong> the value is converting "we should do microservices someday"
into a concrete, conditional trigger. A strong answer names a <em>specific</em> driver ("when
the reporting workload needs to scale independently of the API," "when we grow to a third team
that's blocked in the shared codebase") <em>and</em> the prerequisites to build first (automated
deploys, tracing, on-call maturity) — so the decision becomes "we extract service X when driver
D appears and platform P is ready," not a vague aspiration. That framing is exactly the
architect's discipline: distribution is a response to drivers, gated on readiness, applied to
the specific straining part — never a goal pursued for its own sake. The best answers also note
that most of the "good architecture" they want is available <em>now</em>, cheaply, by keeping the
modular monolith's boundaries clean — so the honest near-term investment is boundary hygiene plus
building the prerequisites, both of which pay off regardless of whether the split ever happens.
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 12 — Event-Driven Architecture →](lesson-12-event-driven){: .btn .btn-primary }
