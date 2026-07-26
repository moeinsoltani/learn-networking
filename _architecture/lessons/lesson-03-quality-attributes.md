---
title: "Lesson 03 — Architecture Characteristics (the \"-ilities\")"
nav_order: 3
parent: "Phase 1: The Architect's Role & Mindset"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 03: Architecture Characteristics (the "-ilities")

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

Here is a truth that reorganizes how you see systems: **an architecture is not
shaped by what the system does — it's shaped by the qualities it must have.** Two
systems that do the exact same thing (say, "store and retrieve orders") can need
completely different architectures — one because it must handle 10 orders a day, the
other 10 million; one because a wrong answer costs a cent, the other because it costs
a lawsuit. The *features* are nearly identical. The **quality attributes** —
scalability, availability, consistency, security — are what drive the design.

Requirements come in two kinds, and only one of them shapes the architecture.

**Functional requirements** describe *what the system does*: "users can place an
order," "admins can issue a refund," "send a receipt email." These are the
features, and — the counter-intuitive part — they rarely drive the architecture
at all.

**Quality attributes**, the "-ilities," describe *how well it does it*, and
these are what the architecture is actually for:

| Attribute | The question it asks |
|---|---|
| Scalability | How much load must it take? |
| Availability | How much downtime is acceptable? |
| Performance | How fast must it respond? |
| Security | Protected against what, and whom? |
| Maintainability | How easily can it be changed? |
| Testability, deployability, … | And so on |

The practical upshot: you can usually add a feature to any reasonable
architecture, but you cannot add availability, or security, or the ability to
change safely, to a system that was not built for it. That is why architects
spend their time on the second list.

There are dozens of "-ilities," and they fall into families: **operational** (things
visible at runtime — availability, performance, scalability, reliability,
recoverability), **structural** (things about the code — maintainability,
testability, deployability, modularity, portability), and **cross-cutting** (things
that don't fit neatly — security, usability, accessibility, compliance). You will
never satisfy all of them; they conflict. The architect's first job is to find the
*few* that matter most for this system and design *for* them.

## Going Deeper

**They conflict — that's the whole game.** You cannot maximize everything. High
security fights usability and performance (every check adds friction and latency).
High performance fights maintainability (hand-tuned code is harder to change). High
availability fights consistency (Lesson 15 is entirely about this trade). High
scalability fights simplicity. Because Lesson 2 told you everything is a trade-off,
here's the concrete form: *choosing your top quality attributes means choosing which
others you'll sacrifice.* A system "designed for everything" is designed for nothing.

**Pick a small number, and rank them.** A useful discipline: force the stakeholders
to name the top 3–5 quality attributes, in priority order. Not fifteen — the point of
prioritizing is that lower ones lose when they conflict with higher ones. "This is a
system optimized for *availability and security*, accepting some cost to performance
and development speed" is a real architectural stance. "It should be scalable,
secure, fast, cheap, and easy to change" is a wish, not a stance.

{: .warning }
> **Vague quality attributes are useless — make them measurable.**
> "It should be fast" and "it should be reliable" cannot be designed for or tested;
> they're feelings. You must turn each into a **quality-attribute scenario**[°](#w-quality-attribute-scenario) with
> three parts: a **stimulus** (what triggers it), the **response** (what the system
> does), and a **measure** (the number that makes it pass or fail). "It should be
> fast" becomes: *"When a user requests the product page (stimulus), the system
> returns it (response) in under 200 ms at the 99th percentile under 1,000 concurrent
> users (measure)."* Now you can design for it, test it, and know if you met it.

**Beware the implicit ones.** The requirements that hurt most are the quality
attributes nobody stated because everyone "assumed" them. Security is assumed until a
breach. Recoverability is assumed until the first data loss with no backup.
Auditability is assumed until the regulator asks. Part of the architect's job is to
*surface the implicit* — to ask "what happens when this fails / gets attacked / gets
audited / grows 10×?" so the unstated quality attributes get stated while they're
still cheap to design for.

**"Architecture serves the characteristics" — the definition, restated.** Everything
in this track ultimately connects here. When you choose a style (Phase 3), you're
choosing which characteristics it optimizes. When you split data (Phase 5), you're
trading consistency for scalability. When you add resilience patterns (Phase 4),
you're buying availability. The characteristics are the *why* behind every structural
decision — which is why identifying and prioritizing them (this lesson and the next)
comes before any style or pattern.

---

## Lab — Design Exercise

**The situation:** A product manager hands you three requirements for a new
patient-appointment system:

1. *"The system should be fast."*
2. *"It should be reliable — we can't have it going down."*
3. *"It needs to be secure since it handles health data."*

These are unusable as written. **Turn each into a measurable quality-attribute
scenario** (stimulus → response → measure), and for each, name one *other* quality
attribute it will trade against — so the PM understands the cost of the number they
pick.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is converting feelings into testable scenarios and exposing the trade each
one buys. Strong answers (exact numbers are illustrative — the <em>structure</em> is
what matters, and in reality you'd negotiate the numbers with the PM):
<br><br>
<strong>1. "Fast" → performance scenario:</strong> "When a clinician searches for a
patient's appointments (stimulus), the results are returned (response) in under 300 ms
at the 95th percentile, under the normal peak of 200 concurrent clinicians (measure)."
Note we specified <em>which</em> operation (search), a percentile (p95, not average —
averages hide the slow tail), a load condition, and a number. <em>Trades against:</em>
cost and consistency — hitting a tight latency target often means caching or read
replicas, which introduce staleness, or more hardware, which costs money. The PM
should know that "under 50 ms" is a very different (and pricier) system than "under
500 ms."
<br><br>
<strong>2. "Reliable / can't go down" → availability scenario:</strong> "The
appointment-booking function (stimulus: a booking request) is available and succeeds
(response) 99.9% of the time measured monthly (measure) — i.e., no more than ~43
minutes of downtime per month." Forcing a specific number is where the real
conversation happens: 99.9% and 99.999% are wildly different architectures and costs
(the latter needs multi-region, redundancy, automated failover — possibly 10× the
cost). <em>Trades against:</em> consistency and complexity/cost — higher availability
usually means redundancy and, in a distributed system, accepting weaker consistency
during failures (CAP, Lesson 15), plus a much larger operational and financial bill.
"Can't go down" is not a spec; "99.9%, and here's what 99.99% would cost" is.
<br><br>
<strong>3. "Secure" → security scenario:</strong> "When an unauthorized party
attempts to access patient records (stimulus), the system denies access, logs the
attempt, and the data — at rest and in transit — is encrypted such that a stolen
database is unreadable (response); the system meets the relevant health-data
regulation's controls and passes an audit (measure)." Security is best expressed as
several scenarios (confidentiality, access control, auditability, breach detection).
<em>Trades against:</em> usability and performance — every auth check, encryption
step, and audit log adds friction for users and latency to requests; strong security
makes the system harder and slower to use, which is a real cost the PM is implicitly
choosing.
<br><br>
<strong>The meta-point to deliver to the PM:</strong> "Each of these is a dial, not a
switch. Tell me the <em>number</em> and the <em>priority</em>, because pushing one up
pushes another down — a system that's 99.999% available, sub-50 ms, and maximally
secure is possible but very expensive and slow to build. Where on each dial do we
actually need to be, given the budget and the users?" That reframing — from vague
adjectives to prioritized, measurable, trade-off-laden dials — is the entire value the
architect adds at requirements time.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Architecture characteristics — the full taxonomy | *Fundamentals of Software Architecture*, Richards & Ford (Ch. 4–6) |
| Quality-attribute scenarios (stimulus/response/measure) | *Software Architecture in Practice*, Bass, Clements & Kazman |
| ISO/IEC 25010 quality model | <https://en.wikipedia.org/wiki/ISO/IEC_9126> (successor 25010) |
| Non-functional requirements | <https://en.wikipedia.org/wiki/Non-functional_requirement> |

---

## Checkpoint

**Q1.** Two systems both "let users upload and share files." Give a plausible reason
their architectures could be completely different, and use it to explain the claim
"features don't drive the architecture — quality attributes do."

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The features are identical ("upload and share files"), but the <strong>quality
attributes</strong> can differ enormously, and those are what shape the design.
Example: System A is a small internal document store for a 50-person company —
low scale, occasional downtime is fine, files are modest. System B is a
consumer photo-sharing service with 50 million users — it must handle massive scale,
near-100% availability, global low-latency delivery, and petabytes of storage.
<br><br>
Same feature list, radically different architectures: System A is happily a single
server with a database and local/attached storage (a monolith); System B needs
object storage (S3-style), a global CDN for delivery, horizontal scaling and load
balancing, sharded metadata, replication across regions, and aggressive caching —
none of which the feature "share files" implies. What forced the difference was
<em>scalability, availability, and performance-at-scale</em> — the quality attributes —
not the feature.
<br><br>
This is why "features don't drive the architecture": you can list every feature and
still not know how to build the system, because the same features at 50 users vs 50
million users, or at "downtime is annoying" vs "downtime is a lawsuit," demand
different structures. The architect's design flows from the <em>-ilities</em> and
their required <em>levels</em>, with the features riding on top. (This is also why the
first question an architect asks about a new system isn't "what does it do?" but "how
much, how fast, how available, how secure — and what's the cost of getting each
wrong?")
</details>

**Q2.** Why is "the system should be fast" a useless requirement to an architect, and
what are the three parts you'd add to make it usable? Why do the three parts matter?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
"Fast" is useless because it's <strong>not measurable, not testable, and not
designable</strong> — it's a feeling, not a target. You can't design toward it (fast
compared to what?), you can't verify you met it (there's no pass/fail), and different
people will assume different numbers, so you'll ship something and argue about whether
it's "fast" forever. The same problem afflicts "reliable," "secure," "scalable" — all
vague until quantified.
<br><br>
The three parts of a <strong>quality-attribute scenario</strong> that fix it:
<ul>
<li><strong>Stimulus</strong> — what triggers the behavior and under what condition:
<em>which</em> operation, at <em>what load</em>. ("When a user requests the search
results page, under 1,000 concurrent users…") This matters because "fast" depends
entirely on the operation and the load — a search under peak traffic is a different
target than a settings-page load at 3 a.m.</li>
<li><strong>Response</strong> — what the system does in reply. ("…the system returns
the results…") This pins down exactly what's being measured, removing ambiguity about
where the clock starts and stops.</li>
<li><strong>Measure</strong> — the number that decides pass/fail, ideally a
percentile not an average. ("…in under 200 ms at the 99th percentile.") This matters
most: it makes the requirement <em>objective</em> (you can test it and know), and the
percentile matters because averages hide the slow tail — a 100 ms average can still
mean 5% of users wait 3 seconds, which is the experience they'll remember.</li>
</ul>
Together they turn a feeling into a contract: something the architect can design
toward, the team can test against, and everyone can agree was or wasn't met. And the
act of forcing the number surfaces the real conversation — "do we need p99 &lt; 100
ms or is &lt; 500 ms fine?" is where the cost/benefit trade-off actually gets decided,
rather than discovered after launch.
</details>

---

## Homework

For your current system, write down the (implicit or explicit) top five quality
attributes it's *actually* built for — and rank them. Then write down the five it
would need if usage grew 100×. Compare the two lists: where they differ is where your
architecture will have to change, and it tells you which of today's decisions are
quietly betting that growth won't come. Finally, pick the single quality attribute
your system most *implicitly* assumes (never stated, just expected) and write the
scenario that would test it — you may find you can't actually meet it today.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise builds three habits at once. A strong response:
<br><br>
<strong>Ranks honestly what the system is <em>actually</em> built for</strong> — not
the aspirational list, but what the design reveals. If everything runs on one server
with no redundancy, the system is built for <em>simplicity and development speed</em>,
<em>not</em> availability, no matter what anyone claims. Reading the real priorities
off the architecture (rather than the wish list) is the skill; often the stated and
revealed priorities differ, which is itself a finding.
<br><br>
<strong>Contrasts current vs 100×-growth priorities</strong> to locate the decisions
that are bets against growth. Typically scalability and availability rocket up the
list at 100×, and the honest observation is that several current decisions (single
database, shared state, synchronous everything) were correct-and-cheap for today's
scale but would be the first things to break — which is <em>fine</em> (premature
scaling is a debt too, Lesson 23) as long as it's a <em>conscious</em> bet ("we're
optimizing for shipping now, accepting we'll re-architect data if we hit 100×") and
not an accident. The value is making the bet explicit so it can be watched.
<br><br>
<strong>Surfaces the most-implicit attribute and tests it.</strong> The best finding
is discovering an assumed-but-unmet quality: writing the recoverability scenario
("when the primary database is lost, the system restores from backup with at most N
minutes of data loss within M hours") and realizing there's no tested backup, or no
measured RPO/RTO at all — i.e., the system <em>assumes</em> recoverability it can't
actually demonstrate. Or writing the security scenario and finding data isn't
encrypted at rest. Surfacing one such gap, while it's still cheap to fix, is exactly
the architect's job of "make the implicit explicit" — and doing it on your real system
tends to be more sobering (and more motivating) than any abstract lesson.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-quality-attribute-architecture-characteristic"></a>**quality attribute / architecture characteristic** — a property of *how* the system operates (fast, secure, changeable), as opposed to *what* it does (its features).
- <a id="w-functional-requirement"></a>**functional requirement** — what the system must *do* (a feature); a <a id="w-non-functional-requirement-nfr"></a>**non-functional requirement (NFR)** — how well it must do it.
- <a id="w-the-ilities"></a>**the "-ilities"** — the nickname for quality attributes, since so many end in -ility (scalability, availability, maintainability, testability…).
- <a id="w-quality-attribute-scenario"></a>**quality-attribute scenario** — a testable statement of a quality: a *stimulus* → the system's *response* → a *measure*.
- <a id="w-operational-structural-cross-cutting"></a>**operational / structural / cross-cutting** — three families of characteristics: at runtime, in the codebase, and spanning everything.
- <a id="w-implicit"></a>**implicit** — assumed but never stated; the dangerous kind of requirement.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 4 — Architectural Drivers & the "Architecturally Significant" →](lesson-04-architectural-drivers){: .btn .btn-primary }
