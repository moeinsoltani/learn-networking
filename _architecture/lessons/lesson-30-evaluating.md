---
title: "Lesson 30 — Evaluating Architecture: ATAM, Trade-offs & Fitness Functions"
nav_order: 3
parent: "Phase 7: Documenting, Evaluating & Evolving Architecture"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 30: Evaluating Architecture — ATAM, Trade-offs & Fitness Functions

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

The most expensive place to discover an architectural flaw is **production**; the cheapest is **on
paper, before you build**. A design is a set of bets about quality attributes — "this will be fast
enough, available enough, secure enough, changeable enough" — and evaluation is the discipline of
*testing those bets before you've spent months building the wrong thing*, and then continuing to test
them automatically as the system evolves. You don't need heavyweight ceremony; you need a stance: **"what
would break this?"** applied to a design against the specific quality attributes it's *for* (Lesson 3).

Architecture can be evaluated **before** you build it, and the mechanism is the
quality-attribute scenario from Lesson 03 — stimulus, response, measure.

Concrete examples:

> "10× traffic on Black Friday → checkout stays under 1 s at p99."
> "The primary database region fails → reads are still served, and recovery
> takes under 30 s."

You then probe the design against each scenario, with a review stance of **"what
would break this?"** rather than "does this look reasonable?" The findings sort
into three kinds:

- **Sensitivity points** — places where one quality attribute swings hard on a
  single decision.
- **Trade-off points** — places where two attributes conflict, so improving one
  degrades the other. These are the decisions worth an ADR.
- **Risks** — including the scenario you may not have thought of, which is why
  the exercise is done with several people.

Finally, **fitness functions** make the important scenarios **continuous**: an
automated test that fails the build when p99 latency regresses turns a one-time
review into an ongoing guarantee.

Two ideas do the work. First, **scenario-based evaluation** (**ATAM**[°](#w-atam) *in spirit*): take the top quality
attributes, write concrete scenarios for each, walk them through the design, and surface three things —
**sensitivity points**[°](#w-sensitivity-point) (decisions one attribute is very sensitive to), **trade-off points**[°](#w-trade-off-point) (decisions
where attributes conflict — the architect's real subject matter), and **risks**[°](#w-risk) (places the design may
not meet a scenario). Second, **fitness functions**[°](#w-fitness-function): for the attributes that matter continuously, turn
the evaluation into an *automated test* that runs forever ("no module may import the database directly",
"p99 latency < 200ms", "no service calls another service's database") — so the architecture doesn't
silently erode after the review is over. Evaluation is how you find flaws while they're still cheap to
fix, and fitness functions are how you keep them fixed.

## Going Deeper

**Scenario-based evaluation — ATAM in spirit.** The
[ATAM](https://en.wikipedia.org/wiki/Architecture_tradeoff_analysis_method) is a formal method, but its
*core* is simple and portable: you evaluate an architecture against **quality-attribute scenarios**[°](#w-quality-attribute-scenario), not
against opinions. The lightweight version: (1) name the top 3–5 quality attributes that actually drive
this system (the ASRs, Lesson 4); (2) write 1–2 concrete **scenarios** per attribute — each a testable
*stimulus → response → measure* (Lesson 3), e.g. "traffic spikes 10× during a flash sale (stimulus) →
checkout continues to complete (response) → at p99 under 1 second (measure)"; (3) walk each scenario
through the proposed design and ask *how* the design meets it — and where it might not. This replaces
"I think it'll be fine" with "here's the scenario, here's how the design handles it, here's where I'm
not sure." Concrete scenarios are what make an evaluation honest.

**The three findings: sensitivity points, trade-off points, risks.** Walking the scenarios surfaces
three kinds of finding, and telling them apart is the skill:
- A **sensitivity point** is a decision that *one* quality attribute is strongly sensitive to — turn
  this dial and that attribute moves a lot (e.g., the replication strategy heavily determines
  availability). Worth knowing because it's where to focus attention.
- A **trade-off point** is a decision that affects *two or more* attributes in *opposite* directions —
  improving one worsens another (e.g., synchronous replication improves consistency but hurts latency
  and availability; a cache improves performance but hurts consistency). **Trade-off points are the
  architect's true subject matter** (Lesson 2 — everything is a trade-off); an evaluation's job is to
  make them *explicit* so they're chosen, not stumbled into.
- A **risk** is a decision or gap that might fail to meet a scenario — the thing the whole exercise is
  hunting for, so you can mitigate it before building rather than discovering it in an incident.

**The "what would break this?" stance.** The productive mindset for evaluating a design — your own or
someone else's — is adversarial-but-kind: actively try to *break* the design on paper. Push the
scenarios to their extremes (what at 100× load? what when this dependency is down? what when two of
these fail at once? what's the blast radius?). This is the same instinct as threat modeling (Lesson 24,
"what can go wrong here?") applied to all quality attributes. It's far cheaper to break a design in a
review than to have production break it for you, and the flaws you find on paper are the ones you can
still fix for free.

{: .warning }
> **Fitness functions — make the architecture continuously testable**
> A one-time review protects the design at a single moment; then entropy takes over and the
> architecture <em>erodes</em> as people make locally-reasonable changes that globally violate it (a
> module reaches into the database it shouldn't, a cyclic dependency creeps in, latency drifts up). An
> <strong>architecture fitness function</strong> is an <em>automated, continuous</em> test of a quality
> attribute — it runs in CI or in production forever, and fails the build or alerts when the attribute
> is violated. Examples:
> - <strong>Structural:</strong> "no cyclic dependencies between modules"; "the domain layer must not
>   import the persistence layer" (enforce the hexagonal boundary of Lesson 10 — a boundary you can't
>   test is only a suggestion, Lesson 6); "no service may connect to another service's database".
> - <strong>Operational:</strong> "p99 latency &lt; 200ms" (a performance test in CI or an SLO alert);
>   "the service starts in &lt; 5s"; "no endpoint without authentication".
> Fitness functions turn <em>implicit</em> architectural rules into <em>explicit, enforced</em> ones —
> the difference between an architecture you <em>hope</em> holds and one you <em>know</em> holds. They
> are the mechanism that makes evolutionary architecture (Lesson 31) safe: you can change freely as long
> as the fitness functions still pass, because they're the guardrails.

**Evaluating others' architectures — kindly and usefully.** Much evaluation is reviewing designs you
didn't create, which is as much a *social* act as a technical one (this is where architecture meets
leadership — cross-link to design reviews). The useful stance: evaluate against the *stated* quality
attributes and scenarios (not your personal taste), separate genuine risks from preferences, ask
questions rather than issue verdicts ("what happens to this scenario when X fails?" rather than "this is
wrong"), and remember the goal is a better *design*, not a won argument. A review that makes the author
defensive surfaces fewer real risks than one that makes them curious. *(This pairs with the
[Leadership]({{ '/leadership/learning-plan.html' | relative_url }}) track's design-review and
feedback material — the technical evaluation only helps if it's delivered so people can hear it.)*

---

## Lab — Design Exercise

**The situation:** A team proposes this design for a **ticket-sales system** for concerts: a stateless
API tier behind a load balancer, a single primary Postgres (with a read replica) holding the seat
inventory and orders, a Redis cache in front of reads, and payment via a third-party provider. Their
stated **top three quality attributes** are: (1) **correctness under contention** — the same seat must
never be sold twice, even during a stampede when a popular show goes on sale; (2) **availability** —
the site should stay up during that on-sale spike; (3) **performance** — browsing seat maps should feel
instant.

**Run a lightweight evaluation.** Write **two scenarios per attribute** (stimulus → response → measure),
then identify **the biggest risk** and **one trade-off point** in the design.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is writing testable scenarios and then reading the design for the risk and the conflict. A
strong answer:
<br><br>
<strong>Scenarios (2 per attribute, each stimulus → response → measure):</strong>
<ul>
<li><em>Correctness under contention #1:</em> 10,000 users click "buy seat A1" within the same second
(stimulus) → exactly one succeeds, all others are told it's taken (response) → zero double-sells,
measured over the whole on-sale (measure).</li>
<li><em>Correctness under contention #2:</em> a user's payment fails after the seat is held (stimulus)
→ the held seat is released back to inventory (response) → within N seconds, and never left stranded
(measure).</li>
<li><em>Availability #1:</em> traffic jumps 50× at the on-sale minute (stimulus) → the site keeps
serving browse and buy (response) → error rate stays under 1% (measure).</li>
<li><em>Availability #2:</em> the primary Postgres fails over (stimulus) → reads keep being served,
writes recover (response) → within 30s, no data loss on committed orders (measure).</li>
<li><em>Performance #1:</em> a user opens a seat map during the spike (stimulus) → it renders
(response) → p95 under 500ms (measure).</li>
<li><em>Performance #2:</em> browsing under normal load (stimulus) → seat map loads (response) → p99
under 200ms (measure).</li>
</ul>
<strong>Biggest risk:</strong> <em>correctness under contention on the single primary</em> is where this
design most plausibly breaks. Selling the same seat twice must be prevented by a real concurrency control
at the write path — a transactional "reserve seat" that atomically checks-and-holds under contention
(row lock / conditional update / a proper seat-hold with a unique constraint). The design leans on a
single primary Postgres, which <em>can</em> do this correctly, but the risk is (a) whether the
application actually implements the atomic reserve-or-fail (or naively reads-then-writes, which double-
sells under a stampede), and (b) whether that hot write path (everyone contending on the popular show's
seats) becomes the availability bottleneck — the very moment correctness is hardest is the moment load
is highest, and the single primary is the contention point. Naming this — that correctness and the
write-path bottleneck collide precisely during the on-sale — is the key finding.
<br><br>
<strong>One trade-off point:</strong> the <strong>Redis cache in front of reads</strong> is a classic
trade-off point (Lesson 21): it improves <em>performance</em> (fast seat-map browsing, offloads the
primary — helping availability too) but hurts <em>correctness/consistency</em> — a cached seat map can
show a seat as available that was <em>just sold</em>, so users click "buy" on a seat that's gone. The
design must resolve this deliberately: browsing can be slightly stale (accept it — the cache serves the
seat <em>map</em>), but the <em>purchase</em> must go to the authoritative store and re-check
atomically (never trust the cache at the moment of sale). That's the trade-off made explicit: stale
reads for browse speed, authoritative check at buy — performance vs correctness, resolved by
<em>where</em> each is allowed.
<br><br>
A strong answer also notes a <strong>sensitivity point</strong> (availability is highly sensitive to how
the single primary handles the write spike — that one decision dominates whether the site survives
on-sale) and applies the <em>"what would break this?"</em> stance: what if 100× not 50×? what if payment
provider is slow and holds pile up? what if failover happens <em>during</em> the on-sale? — surfacing
risks on paper while they're free to fix. Finally: several of these — "no double-sell", "purchase always
re-checks the authoritative store" — should become <strong>fitness functions</strong> (a load test that
asserts zero double-sells; a check that the buy path never reads seat availability from cache) so
correctness stays enforced as the code evolves.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| ATAM | <https://en.wikipedia.org/wiki/Architecture_tradeoff_analysis_method> |
| Quality-attribute scenarios | *Software Architecture in Practice*, Bass/Clements/Kazman |
| Architecture fitness functions | *Building Evolutionary Architectures*, Ford/Parsons/Kua |
| Fitness functions in practice | <https://www.thoughtworks.com/insights/articles/fitness-function-driven-development> |
| Testing architecture (structural) | <https://www.archunit.org/> |
| Evaluating & reviewing designs (people side) | [Leadership Lesson 9]({{ '/leadership/lessons/lesson-09-design-reviews.html' | relative_url }}) |

---

## Checkpoint

**Q1.** Why evaluate an architecture *before* building it, and what are sensitivity points, trade-off
points, and risks? Which is "the architect's true subject matter" and why?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Why evaluate before building:</strong> an architecture is a set of bets about quality attributes,
and the cost of discovering a bad bet grows enormously the later you find it — a flaw caught on paper in
a review costs a conversation; the same flaw caught in production costs an outage, a rewrite, and lost
trust. Evaluating a design against concrete quality-attribute scenarios <em>before</em> you build tests
those bets while they're still cheap to change (and doing it continuously, via fitness functions, keeps
them tested as the system evolves). It replaces "I think it'll be fine" with a structured "here's the
scenario, here's how the design meets it, here's where it might not."
<br><br>
<strong>The three findings:</strong>
<ul>
<li><strong>Sensitivity point</strong> — a decision that <em>one</em> quality attribute is strongly
sensitive to: change this one dial and that attribute moves a lot (e.g. availability is very sensitive to
the replication/failover strategy). Tells you where to focus.</li>
<li><strong>Trade-off point</strong> — a decision that affects <em>two or more</em> attributes in
<em>opposite</em> directions: improving one worsens another (synchronous replication → better
consistency but worse latency/availability; a cache → faster but staler). </li>
<li><strong>Risk</strong> — a decision or gap that might fail to meet a scenario; the thing the
evaluation is hunting for, so you can mitigate it before it becomes an incident.</li>
</ul>
<strong>Trade-off points are the architect's true subject matter</strong> because architecture <em>is</em>
the discipline of trade-offs (Lesson 2 — everything is a trade-off, and the job is making them explicit).
A trade-off point is exactly where a decision pits quality attributes against each other, and the
architect's value is in <em>surfacing</em> that conflict so it's chosen deliberately (with the business
cost weighed) rather than stumbled into. Sensitivity points and risks matter, but the trade-off points
are where the essential architectural judgment happens.
</details>

**Q2.** What is a fitness function, and why is it necessary in addition to a one-time evaluation? Give
two examples (one structural, one operational).

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>What it is:</strong> an architecture fitness function is an <em>automated, continuous</em> test
that a quality attribute still holds — it runs in CI or in production indefinitely and fails/alerts when
the attribute is violated. It turns an <em>implicit</em> architectural rule ("the domain shouldn't
depend on the database"; "we need p99 under 200ms") into an <em>explicit, enforced, executable</em> one.
<br><br>
<strong>Why it's necessary beyond a one-time review:</strong> a review protects the architecture at a
single moment, but architectures <em>erode</em>. Over time, many locally-reasonable changes accumulate
into global violations — someone adds a shortcut import that breaks a boundary, a cyclic dependency
creeps in, latency drifts upward — and no single change looks wrong in review, so the erosion is
invisible until the architecture no longer holds. A fitness function catches each violation the moment
it's introduced (the build goes red), so the architecture stays enforced continuously rather than
decaying after the review. It's the difference between an architecture you <em>hope</em> still holds and
one you <em>know</em> holds — and it's what makes safe evolution possible (Lesson 31): you can change
freely as long as the fitness functions pass, because they are the guardrails. A rule you can't test is
only a suggestion (Lesson 6).
<br><br>
<strong>Two examples:</strong>
<ul>
<li><em>Structural:</em> "no module in the domain layer may import the persistence/ORM layer" (enforces
the hexagonal dependency rule of Lesson 10), or "no cyclic dependencies between modules", checked in CI
by a tool like ArchUnit — a violating import fails the build.</li>
<li><em>Operational:</em> "p99 checkout latency &lt; 200ms", asserted by a performance test in CI or an
SLO alert in production (Lesson 25) — a regression trips it; or "every HTTP endpoint requires
authentication", checked automatically so no unauthenticated route can ship.</li>
</ul>
</details>

---

## Homework

Take a design your team is currently building or recently shipped and run a lightweight evaluation on it.
Name its top 3 quality attributes (be honest about which actually drive it), write two testable scenarios
each, and walk them through the design — hunting for the biggest risk and at least one trade-off point.
Apply the "what would break this?" stance: push each scenario to an extreme. Then pick the two most
important architectural rules the design relies on (a boundary that mustn't leak, a latency ceiling, a
"no service touches another's DB" rule) and design a **fitness function** for each — how would you test
it automatically and continuously? Identify whether any of those rules is currently *un*-enforced (only
a hope) and would silently erode.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
A strong response turns evaluation from an opinion into a structured probe of a real design.
<br><br>
<strong>Writes real scenarios, not vibes.</strong> The discipline is forcing each quality attribute into
a testable stimulus → response → measure ("10× traffic → checkout completes → p99 &lt; 1s"), which
immediately exposes where the design is hand-wavy. A good answer is honest that some "requirements" were
too vague to be scenarios until made measurable — that act of sharpening is half the value.
<br><br>
<strong>Finds the risk and the trade-off with the adversarial stance.</strong> Pushing scenarios to
extremes ("what at 100×?", "what when two dependencies fail at once?", "what's the blast radius?") is
what surfaces the biggest risk while it's still cheap to fix on paper. Naming an explicit trade-off point
— a decision where two attributes pull opposite ways (cache: performance vs consistency; sync replication:
consistency vs availability) — shows the architect's real subject matter: making the conflict visible so
it's chosen, not stumbled into.
<br><br>
<strong>Designs fitness functions and spots the unenforced rules.</strong> For the two load-bearing
rules, a good answer specifies a concrete automated check (an ArchUnit test for a boundary; a CI perf
test or SLO alert for a latency ceiling; a check that no unauthenticated endpoint ships) and — crucially
— identifies which rules are <em>currently only hoped for</em>, enforced by nothing, and therefore
quietly eroding. The takeaway a good answer reaches: evaluation is cheap insurance against expensive
production surprises; its core is scenarios + the "what would break this?" stance; the trade-off points
are the essential findings; and fitness functions are how you stop the evaluated architecture from
silently decaying the day after the review — turning rules you hope hold into rules you know hold.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-atam"></a>**ATAM** — Architecture Tradeoff Analysis Method: a structured way to evaluate a design against its quality-attribute scenarios and find the risks and trade-offs. Here we use it *in spirit*, not the full ceremony.
- <a id="w-quality-attribute-scenario"></a>**quality-attribute scenario** — a testable statement of a requirement: a *stimulus* → the system's *response* → a *measure* (Lesson 3).
- <a id="w-sensitivity-point"></a>**sensitivity point** — a decision that strongly affects *one* quality attribute (turn this dial and availability moves a lot).
- <a id="w-trade-off-point"></a>**trade-off point** — a decision that affects *two or more* attributes in opposite directions (better performance here costs consistency there).
- <a id="w-risk"></a>**risk** — a decision that might not meet a quality-attribute requirement; the thing an evaluation is hunting for.
- <a id="w-fitness-function"></a>**fitness function** — an automated, continuous check that a quality attribute still holds (e.g., "no cyclic dependencies", "p99 < 200ms").
- <a id="w-evaluate-before-you-build"></a>**evaluate before you build** — judging a design on paper is orders of magnitude cheaper than discovering its flaws in production.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 31 — Evolutionary Architecture & Managing Change →](lesson-31-evolutionary){: .btn .btn-primary }
