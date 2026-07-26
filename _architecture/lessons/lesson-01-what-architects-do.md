---
title: "Lesson 01 — What a Software Architect Actually Does"
nav_order: 1
parent: "Phase 1: The Architect's Role & Mindset"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 01: What a Software Architect Actually Does

{: .note }
> **Words to know** *(simple definitions for this lesson's jargon)*
> - **architecture** — the set of decisions about a system that are both *significant* and *hard to change later*.
> - **quality attribute / "-ility"** — a property of the whole system (scalability, security, maintainability) rather than a feature it performs.
> - **irreversible / one-way door** — a decision that is expensive or impossible to undo once made.
> - **ivory tower** (idiom) — a place cut off from real work; the "ivory-tower architect" draws diagrams but never faces the code's reality.
> - **guardrail / constraint** — a boundary you set so teams can move fast *within* it without breaking the system.
> - **stakeholder** — anyone affected by the system: engineers, product, security, operations, the business, customers.

## Concept

The hardest thing to accept in this transition is that your job is no longer to
produce the best code — it is to make the **decisions that shape the system**, most
of which you will never write a line for. Ralph Johnson's famous definition:
*architecture is the set of decisions you wish you could get right early, because
they are expensive to change later.* That "expensive to change" is the whole
distinction. Which programming style to use in a function is cheap to change — not
architecture. Whether the system is one deployable or fifty, whether services share
a database, what your consistency model is, where the trust boundary sits — those
are expensive to change, so they're architecture.

```
   The altitude shift:

   SENIOR DEVELOPER                 ARCHITECT
   ────────────────                 ─────────
   "How do I build this well?"      "What should we build, and how
                                     should the whole thing be shaped?"
   Optimizes a component            Optimizes the system & its qualities
   Depth in one area                Breadth across many, deep enough in each
   Output = working code            Output = decisions, constraints,
                                     shared understanding
   Reversible choices, fast         Irreversible choices, made carefully
```

Your real deliverables are not diagrams. They are: **decisions** (with the reasoning
preserved), **constraints and guardrails** (so teams move fast safely), and
**shared understanding** (everyone building the system holds the same mental model).
Diagrams and documents are just how those get communicated.

## Going Deeper

**The role is a spectrum, not one job.** "Architect" spans *application* architects
(one system, deep), *solution* architects (a product spanning several systems), and
*enterprise* architects (standards and strategy across a whole company). More
useful than the titles: architecture is an *activity* that exists on every team,
whether or not anyone holds the title. On a small team the senior engineers do it
together; the title just concentrates the responsibility. Don't chase the badge;
chase the ability to make good structural decisions and get them adopted.

**The ivory-tower trap.** The failure mode of the role is the architect who
produces beautiful diagrams disconnected from reality — decisions that don't survive
contact with the code, made by someone who hasn't felt the pain they're prescribing.
The best architects stay close to the ground: they read the code, they prototype the
risky parts, they pair with teams, they are *embarrassed* to hand down a design they
couldn't build themselves. Richards & Ford call one of the "laws of architecture":
*an architect who doesn't code loses touch with the implications of their
decisions.* You don't have to be the best coder on the team — but you must stay
technically credible and grounded.

{: .note }
> **Architecture vs design — where's the line?**
> There's no crisp boundary, only a gradient of *significance × reversibility*.
> A rule of thumb: if getting it wrong would be expensive to fix and affects many
> teams or the whole system's qualities, it's architecture; if it's local and cheap
> to change, it's design, and you should let the team own it. Part of the job is
> *not* over-reaching — architecting the things that matter and delegating the rest.

**You serve stakeholders, not elegance.** Every architectural decision trades one
group's interests against another's: operations wants simplicity, product wants
speed, security wants control, finance wants a smaller cloud bill. The architect's
job is to surface those tensions and make a *defensible* call, not to optimize for
technical beauty in a vacuum. This is why the role is as much communication and
influence as it is technology — a theme that runs through the whole track and
connects directly to the [Engineering Leadership]({{ '/leadership/learning-plan.html' | relative_url }})
track.

---

## Lab — Design Exercise

**The situation:** A team lists the decisions they made on a project last quarter:

1. Chose PostgreSQL as the primary database.
2. Named a service `billing-svc` instead of `payments-svc`.
3. Decided the system would be a modular monolith, not microservices.
4. Adopted a specific JSON logging library.
5. Made the public API versioned via URL path (`/v1/...`).
6. Used tabs instead of spaces in the codebase.
7. Chose eventual consistency between the "orders" and "inventory" modules.
8. Picked React for the admin dashboard's front end.

**Sort these into "architectural" and "not architectural,"** and for each, state the
one word that decided it: is it *significant and hard to reverse*? Then defend the
line you drew.

**Your answer:**

<details>
<summary>Show Model Answer</summary>
<br>
The test is <em>significance × reversibility</em>, not "is it technical" or "did it
feel important." Applying it:
<br><br>
<strong>Architectural</strong> (significant + hard to reverse):
<ul>
<li><strong>#3 modular monolith vs microservices</strong> — the single most
architectural decision here; it shapes deployment, team structure, data, and every
other choice, and reversing it is a massive project.</li>
<li><strong>#7 eventual consistency between orders and inventory</strong> — a
consistency model is deeply architectural: it's baked into the data design and the
business logic, hard to reverse, and affects correctness the user can see.</li>
<li><strong>#5 URL-path API versioning</strong> — a public API is a one-way door
(external clients depend on it); the versioning strategy is expensive to change once
consumers exist.</li>
<li><strong>#1 PostgreSQL</strong> — borderline but architectural: the primary data
store is heavy to move once data and code depend on it, and it constrains many later
choices (transactions, scaling path). Note it's <em>less</em> irreversible than #3
or #7 — you <em>can</em> migrate a database, painfully.</li>
</ul>
<strong>Not architectural</strong> (local and/or cheap to reverse):
<ul>
<li><strong>#2 the service name</strong> — a rename is a find-and-replace; trivial
to reverse. (Naming <em>matters</em> for clarity, but it's not architecture.)</li>
<li><strong>#4 logging library</strong> — swappable behind an interface; local,
reversible. (Whether you have <em>observability</em> at all is architectural — but
the specific library isn't.)</li>
<li><strong>#6 tabs vs spaces</strong> — pure style; a linter reformats it in
seconds. The clearest "not architecture" on the list.</li>
<li><strong>#8 React for the admin dashboard</strong> — reversible and contained to
one non-critical surface; a framework choice for a small internal UI is a two-way
door. (Note: React for the <em>whole customer-facing product</em> would be closer to
architectural — context changes the answer, which is itself the lesson.)</li>
</ul>
<strong>The line you drew:</strong> the good defense names the two axes explicitly —
<em>how expensive is this to change, and how far do its consequences reach?</em> —
and shows that the same <em>kind</em> of decision (a framework, a database) can land
on either side depending on scope and reversibility (#8 vs #1). The deeper point:
architects spend their scarce attention on the top group and deliberately
<em>don't</em> litigate the bottom group — an architect who mandates tabs-vs-spaces
is doing the job wrong in both directions (over-reaching on the trivial, and burning
the credibility they need for the decisions that matter).
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| "Architecture is the decisions that are hard to change" | Martin Fowler — <https://martinfowler.com/architecture/> |
| The many meanings of "architect" | *Fundamentals of Software Architecture*, Richards & Ford (Ch. 1–2) |
| The architect who still codes | Gregor Hohpe, *The Software Architect Elevator* |
| Ralph Johnson's definition (via Fowler's "Who Needs an Architect?") | <https://martinfowler.com/ieeeSoftware/whoNeedsArchitect.pdf> |

---

## Checkpoint

**Q1.** Give the two-part test that decides whether a decision is "architectural,"
and use it to explain why "which logging library to use" usually isn't, while "does
the system share one database or many" usually is.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The test is <strong>significance × reversibility</strong>: a decision is
architectural when it is both <em>significant</em> (its consequences reach across
the system or many teams, affecting the system's qualities) <em>and hard to
reverse</em> (expensive, risky, or slow to undo once it's in place). Both must hold.
<br><br>
<strong>Logging library</strong> — usually <em>not</em> architectural: it's normally
hidden behind a thin interface, its blast radius is contained, and you can swap it in
an afternoon. Low reversibility cost, limited significance. (Note the nuance:
<em>whether the system has real observability at all</em> — structured logs, traces,
correlation IDs threaded through every service — <em>is</em> architectural, because
it must be designed into the whole system and is painful to retrofit. The specific
library is the reversible detail; the observability strategy is the architecture.)
<br><br>
<strong>One database vs many</strong> — architectural: it determines whether you can
use ACID transactions, how services couple through data, your consistency model,
your scaling path, and your team boundaries. Reversing it (splitting a shared
database, or re-merging split ones) is one of the most expensive migrations in
software. High significance, very high reversibility cost — squarely architecture.
</details>

**Q2.** What is the "ivory-tower architect" anti-pattern, and what is the concrete
habit that prevents an architect from becoming one?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The <strong>ivory-tower architect</strong> makes decisions and draws diagrams from a
position disconnected from the real code and the people building it — prescribing
designs they've never had to implement, unaware of the practical implications, and
handing them down as decrees. The result is architecture that doesn't survive
contact with reality: designs that are impractical, that the teams resent and
route around, and whose flaws only surface late (because the architect wasn't close
enough to feel them early).
<br><br>
The habit that prevents it: <strong>staying close to the ground</strong> — reading
the actual code, prototyping/spiking the risky parts yourself, pairing with the
teams who implement, and treating your own technical credibility as something you
maintain rather than something the title grants. You don't need to be the top coder,
but you must stay grounded enough that (a) your decisions are informed by real
implementation constraints, and (b) the engineers trust that you understand what
you're asking of them. As the "laws of architecture" put it: an architect who stops
engaging with the code loses touch with the consequences of their own decisions —
and influence is earned through credibility, not conferred by a job title.
</details>

---

## Homework

For your *current* system at work, write down the five decisions you consider most
architectural — the ones that would be most expensive to change today. For each, note
(a) roughly *when* it was made and by whom, (b) whether the reasoning behind it is
written down anywhere, and (c) whether, with hindsight, it was categorized correctly
at the time (was an irreversible decision made casually, or a reversible one
over-agonized?). Then reflect: how many of your five have their *why* preserved, and
what would it cost the team if the people who made them left tomorrow?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
There's no single right list — the value is in the exercise and what it reveals.
A strong response will surface some uncomfortable but common findings, and the point
is to notice them:
<br><br>
<strong>(a) The reasoning is usually <em>not</em> written down.</strong> Most teams
discover that their most consequential decisions live only in the memory of whoever
made them — the "why" was never captured (this is exactly the problem ADRs solve,
Lesson 29). If several of your top five have no recorded reasoning, that's a real
bus-factor and re-litigation risk: a future engineer will either reconstruct it by
archaeology or "fix" a deliberate choice not knowing why it was made.
<br><br>
<strong>(b) The categorization was often wrong at the time.</strong> A good
reflection finds at least one decision that was <em>irreversible but made
casually</em> (e.g., a data model or public API shape decided in an afternoon that's
now baked into everything — the dangerous mistake) and possibly one that was
<em>reversible but over-deliberated</em> (weeks of debate over something you could
have changed cheaply — wasted energy). This builds the instinct the whole role
depends on: matching how carefully you decide to how expensive the decision is to
reverse.
<br><br>
<strong>(c) The blast radius of losing the deciders.</strong> The honest answer for
most teams is "significant" — which reframes documentation and knowledge-sharing not
as bureaucracy but as de-risking. The meta-lesson: doing this audit once usually
converts people into believers in writing decisions down, because they viscerally
feel how much critical reasoning is currently unpreserved. If your five are all
well-documented and were all correctly categorized, you're in rare and enviable
shape — and you should note <em>what practice</em> made that true, so you keep it.
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 2 — Architectural Thinking & the Nature of Trade-offs →](lesson-02-architectural-thinking){: .btn .btn-primary }
