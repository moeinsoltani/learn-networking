---
title: "Lesson 28 — Documenting Architecture: the C4 Model & Views"
nav_order: 1
parent: "Phase 7: Documenting, Evaluating & Evolving Architecture"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 28: Documenting Architecture — the C4 Model & Views

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

An architecture that lives only in one person's head, or in a single sprawling diagram no one can read,
is barely an architecture at all — it can't be built to, reviewed, remembered, or evolved. Documentation
is the deliverable that **outlives you**: it's how others build what you designed, how a reviewer judges
it, and how the next architect understands *why* before they change it. But the classic failure is "one
giant diagram" that tries to show everything — every box, every arrow, every layer — and therefore
serves *no one*: too detailed for the exec, too vague for the developer, unreadable for everybody.

The fix is the single most important idea in architecture documentation: **different audiences need
different views.** You don't draw *the* diagram; you draw a small set of diagrams, each at one level of
abstraction, each answering the questions of one audience.

The usual failure is **one giant diagram** — every box and line in the estate on
a single canvas. It carries far too much detail for an executive and far too
little context for a developer, and the honest description is that it is
readable by nobody.

The **C4 model** fixes this by drawing several diagrams, each at one zoom level
and each for one audience:

| Level | What it shows | Who it is for |
|---|---|---|
| **L1 — Context** | Our system, its users, and the external systems it talks to | Everyone, including non-technical stakeholders |
| **L2 — Container** | Zoom in: the applications, services, and databases that make it up | Developers and operations — the big picture |
| **L3 — Component** | Zoom into *one* container's internal parts | Developers working on that container |
| **L4 — Code** | Classes and their relationships | Rarely drawn — your IDE already shows this |

The discipline that makes it work is **one abstraction level per diagram**, and
zooming outward-in. A diagram that mixes a load balancer, a Java class, and a
business capability has no audience, which is why nobody updates it.

The **C4 model**[°](#w-c4-model) operationalizes this as four zoom levels — Context, Container, Component, Code —
where each level *zooms in* on the previous, adding detail one layer at a time. It's not "more boxes";
it's a **map metaphor**: a country map, then a city map, then a street map, then a building plan — same
territory, chosen level of detail for the reader. Do this, keep each diagram to one abstraction level,
give it a **legend**[°](#w-legend), and document the *why* (not just the what), and your architecture becomes something
people can actually build, review, and remember.

## Going Deeper

**Why one diagram fails: the 4+1 insight.** The classic
[4+1](https://en.wikipedia.org/wiki/4%2B1_architectural_view_model) **view**[°](#w-view) model (Kruchten) made the
point decades ago: a single diagram can't serve the *logical* structure (what a developer wants), the
*process/runtime* behavior (what an operator wants), the *development* structure (modules/teams), and
the *physical* deployment (infrastructure) all at once — so you describe the architecture through
several complementary views, tied together by a few key **scenarios** (the "+1"). You don't have to use
**4+1**[°](#w-4-1) by name, but internalize its lesson: **pick views by audience and by the question each answers.**
A deployment diagram answers "where does it run?"; a sequence diagram answers "what happens when a user
checks out?"; a component diagram answers "how is this service built?" Trying to answer all of them in
one picture answers none.

**The C4 model — zoom levels, not more boxes.** [C4](https://c4model.com/) gives a simple, disciplined
hierarchy:
- **Level 1 — System Context:** your system as a single box, surrounded by its **users** and the
  **external systems** it talks to. No internal detail. Audience: *everyone*, including non-technical
  stakeholders. It answers "what is this thing, who uses it, and what does it depend on?"
- **Level 2 — Container:** zoom inside the system box to show the **containers** — the separately
  deployable/runnable units (the web app, the mobile app, the API service, the database, the message
  broker) and how they communicate. Audience: developers and ops — this is the "big technical picture"
  and usually the single most useful diagram.
- **Level 3 — Component:** zoom inside *one* container to show its major **components** (the internal
  building blocks / groupings of code) and their relationships. Audience: developers working on that
  container.
- **Level 4 — Code:** the class/code level. C4 says you *rarely need to draw this* — the IDE and code
  already show it, and it's the fastest to go stale.

The discipline that makes C4 work: **one abstraction level per diagram** (don't put a class inside the
context diagram), consistent notation, and you only draw the levels the audience needs — most teams
get enormous value from just **Context + Container** and stop there.

{: .warning }
> **"Container" in C4 does NOT mean Docker**
> This trips people up constantly. A C4 <strong>container</strong> is a <em>separately deployable or
> runnable thing</em> — a server-side application, a single-page app, a mobile app, a database, a
> serverless function, a message broker. It's about the <em>unit of deployment/execution</em>, not
> Linux containers. A Postgres database is a C4 "container"; so is your React front-end. Don't conflate
> the two, and if your audience might, say "application/data store" out loud when you present it.

**Diagrams that carry meaning.** A diagram is a communication tool, and most diagrams fail as
communication: unlabeled arrows (does the line mean "calls", "depends on", "sends events to", "reads
from"?), inconsistent shapes, no indication of direction or protocol. Make diagrams *mean* something:
give every diagram a **legend** (what each shape, line style, and colour denotes), **label the
relationships** (not just "→" but "→ *reads orders from, via REST/JSON*"), keep notation consistent
across diagrams, and title each diagram with its scope and level. A diagram a stranger can understand
without you standing next to it explaining it is the goal.

**Diagram-as-code and the documentation package.** Diagrams drawn in a GUI tool rot: they live outside
version control, no one updates them, and they drift from reality. **Diagrams-as-code** (text-based:
Mermaid, PlantUML, Structurizr, Graphviz) keep the diagram in the repo next to the code, so it's
versioned, diffed in review, and can be regenerated — the same reason we prefer IaC (Lesson 27). Beyond
diagrams, a real architecture *document* is more than pictures: templates like
[arc42](https://arc42.org/) and the "Documenting Software Architectures" (views-and-beyond) approach
give a package — context and goals, the significant decisions (ADRs, Lesson 29), the quality-attribute
requirements (Lesson 3), constraints, and risks — because **the most important thing to document is
the *why***, and a diagram alone never captures the reasoning, the trade-offs, or the rejected
alternatives. Document decisions and rationale, not just structure.

---

## Lab — Design Exercise

**The situation:** Consider a familiar shape of system — an **online food-ordering platform**: customers
use a web app and a mobile app to browse restaurants and place orders; an API backend handles orders and
talks to a database; payments go to a third-party provider (Stripe-like); order-status notifications are
sent by email/SMS through an external messaging provider; restaurants use a separate dashboard to accept
orders.

**Sketch the C4 Context and Container diagrams** (in words / ASCII), and for each diagram **identify
which stakeholder it serves** and what question it answers. Then name one thing you would *not* put in
the Context diagram, and why.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is putting the right level of detail at each zoom level and knowing who each serves. A strong
answer:
<br><br>
<strong>Level 1 — System Context</strong> (the whole system as one box + who/what it touches):
<pre>
     [Customer] ──places orders──▶ ┌─────────────────────────┐ ◀──accepts orders── [Restaurant staff]
                                    │  Food-Ordering Platform │
                                    │      (our system)       │
                                    └───────────┬─────────────┘
                                    │           │            │
                          charges via│    sends │    sends   │
                                    ▼      email/SMS via      ▼
                            [Payment provider] [Messaging provider]
</pre>
<em>Serves:</em> <strong>everyone — including non-technical stakeholders</strong> (product, execs, a new
joiner). <em>Answers:</em> "What is this system, who uses it, and what external systems does it depend
on?" No internal detail at all — that's the point.
<br><br>
<strong>Level 2 — Container</strong> (zoom inside the box to the deployable units):
<pre>
   [Customer]                                   [Restaurant staff]
      │ HTTPS                                        │ HTTPS
      ▼                                              ▼
   ┌─ Web App ─┐   ┌─ Mobile App ─┐            ┌─ Restaurant Dashboard ─┐
   └─────┬─────┘   └──────┬───────┘            └───────────┬───────────┘
         └────────┬───────┘   REST/JSON                    │
                  ▼                                         │
             ┌──────────── API Backend ───────────┐◀───────┘
             │ (order service)                     │
             └──┬───────────────┬──────────────┬───┘
       reads/writes         calls           calls
                  ▼               ▼              ▼
           [Orders DB]   [Payment provider] [Messaging provider]
</pre>
<em>Serves:</em> <strong>developers and ops</strong> — the big technical picture. <em>Answers:</em>
"What are the deployable pieces (web app, mobile app, dashboard, API, database), and how do they
communicate (protocols, direction)?" This is usually the single most useful diagram. Note the labels on
the relationships (REST/JSON, reads/writes) and that <em>[Orders DB]</em> is a C4 "container" (a data
store), not a Docker container.
<br><br>
<strong>What to keep OUT of the Context diagram, and why:</strong> the <em>internal</em> pieces — the
web app, mobile app, API service, and database — do <em>not</em> belong in Level 1. The Context diagram
shows the system as a <em>single box</em>; showing its internals there mixes two abstraction levels in
one diagram (the exact "one giant diagram" failure) and overwhelms the non-technical audience it's meant
for. Those internals appear when you <em>zoom in</em> to the Container level. Equally, you would not put
class/code detail in the Container diagram — that's Level 3/4. The whole discipline is <strong>one
abstraction level per diagram</strong>: each diagram shows exactly one zoom level, so each audience gets
a picture pitched at their question.
<br><br>
A mature answer also notes: both diagrams need a <strong>legend</strong> (what boxes vs external systems
vs data stores mean, what the line labels mean), and the accompanying document should record the
<em>why</em> — e.g., why payments are delegated to a provider (Lesson 24/33), which the diagram alone
can't convey.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| The C4 model | <https://c4model.com/> |
| 4+1 architectural view model | <https://en.wikipedia.org/wiki/4%2B1_architectural_view_model> |
| Documenting Software Architectures (views & beyond) | Clements et al. — <https://www.sei.cmu.edu/documentation/> |
| arc42 documentation template | <https://arc42.org/> |
| Diagrams as code (Mermaid) | <https://mermaid.js.org/> |
| The value of software-architecture docs | Martin Fowler — <https://martinfowler.com/architecture/> |

---

## Checkpoint

**Q1.** Why does "one giant diagram" fail, and how does the C4 model's zoom-level approach fix it? What
are the four levels and who does each serve?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Why one diagram fails:</strong> a single diagram trying to show everything — external context,
internal structure, runtime behavior, deployment — ends up serving no audience. It's too detailed and
cluttered for a non-technical stakeholder (who just needs "what is this and who uses it?"), yet too
high-level or too tangled for a developer (who needs the actual components and relationships). Different
audiences have different questions, and a picture pitched at all of them at once is pitched at none —
it's unreadable and, worse, it goes stale because no one owns it.
<br><br>
<strong>How C4 fixes it — zoom levels, not more boxes:</strong> instead of one picture, C4 gives a
hierarchy of diagrams where each <em>zooms in</em> on the previous, adding one level of detail — like a
country map → city map → street map. Each diagram sits at <em>one</em> abstraction level and serves one
audience's question, so nothing is over- or under-detailed for its reader.
<br><br>
<strong>The four levels:</strong>
<ul>
<li><strong>Context (L1):</strong> the whole system as one box, with its users and external systems.
Serves <em>everyone, including non-technical stakeholders</em>; answers "what is it, who uses it, what
does it depend on?"</li>
<li><strong>Container (L2):</strong> zoom inside the system to the separately deployable/runnable units
(apps, services, databases) and how they talk. Serves <em>developers and ops</em>; the big technical
picture and usually the most useful diagram.</li>
<li><strong>Component (L3):</strong> zoom inside one container to its internal components. Serves
<em>developers on that container</em>.</li>
<li><strong>Code (L4):</strong> classes/code. Rarely drawn — the IDE shows it and it goes stale fast.</li>
</ul>
The discipline that makes it work: one abstraction level per diagram, consistent notation with a legend,
and drawing only the levels your audience needs (Context + Container is often enough).
</details>

**Q2.** What makes a diagram actually communicate (vs. just decorate), why do we prefer diagrams-as-code,
and what's the "most important thing to document"?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>What makes a diagram communicate:</strong> it must <em>mean</em> something a stranger can read
without you narrating it. That requires: a <strong>legend</strong> (what each shape, line style, and
colour denotes — box vs external system vs data store, sync call vs event); <strong>labeled
relationships</strong> (not a bare arrow but "reads orders from, via REST/JSON" — the arrow's meaning
and often protocol/direction); <strong>consistent notation</strong> across diagrams; a title stating the
diagram's scope and level; and <strong>one abstraction level per diagram</strong> so it's not a mix of
context and classes. A diagram with unlabeled arrows and undefined shapes is decoration — it looks like
communication but conveys almost nothing precisely.
<br><br>
<strong>Why diagrams-as-code:</strong> GUI-drawn diagrams live outside version control, so no one updates
them and they drift from reality until they're actively misleading. Writing diagrams as <em>text</em>
(Mermaid, PlantUML, Structurizr) puts them in the repo next to the code — versioned, diffed in code
review, and regenerable — so they stay current and are subject to the same discipline as code (the same
reason we favor infrastructure-as-code). A diagram that's wrong is worse than no diagram; keeping it in
code is how you keep it right.
<br><br>
<strong>The most important thing to document: the <em>why</em>.</strong> Structure (the what) is
visible in the code and the diagrams; what's <em>invisible and irreplaceable</em> is the reasoning — the
decisions made, the trade-offs weighed, the alternatives rejected and why (Lesson 29, ADRs), the
quality attributes the design serves, and the constraints it lives under. A diagram shows the boxes but
never says <em>why</em> those boxes and not others, which is exactly what the next engineer needs to
avoid relitigating or unknowingly breaking the design. So a real documentation package (arc42 /
views-and-beyond) captures decisions and rationale alongside the views — because the reasoning is the
part that outlives you and can't be reconstructed from the code.
</details>

---

## Homework

Pick a system you know well and produce (in words, ASCII, or diagram-as-code) its **C4 Context and
Container diagrams**. Be honest about whether you actually know the containers and their relationships,
or whether drawing it exposed gaps in your own understanding. For each diagram, name the stakeholder it
serves. Then look at your team's *existing* architecture documentation: is there any? Is it one giant
stale diagram, or views by audience? Does it explain the *why* or only the *what*, and is it in version
control or rotting in a wiki/slide deck no one opens? Identify the single most valuable documentation
improvement — often just "draw and maintain the Container diagram" or "start recording the why."

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
A strong response is concrete about the system and honest about the gaps.
<br><br>
<strong>Producing the diagrams surfaces understanding gaps.</strong> A genuinely useful outcome is
discovering that drawing the Container diagram was <em>harder than expected</em> — you weren't sure
which pieces are separately deployable, or how two services actually communicate, or what that external
dependency really is. That's the diagram doing its job: exposing fuzzy understanding. A good answer keeps
each diagram at one level (Context = system + users + externals only; Container = the deployable units
and labeled relationships) and doesn't leak internals into the Context view.
<br><br>
<strong>Assesses the existing docs realistically.</strong> The common findings: no documentation at all
(it's tribal knowledge); or one sprawling, out-of-date diagram that serves no one and no longer matches
reality; or docs that show structure but never the reasoning. Recognizing <em>which</em> failure mode —
and that a stale diagram can be worse than none because it misleads — is the insight.
<br><br>
<strong>Names the highest-value improvement.</strong> The architect's move is one specific, maintainable
step: usually (a) create and <em>keep current</em> a Container diagram (the single most useful view) as
diagram-as-code in the repo, or (b) start capturing the <em>why</em> (ADRs, Lesson 29) so decisions stop
being relitigated, or (c) split the one giant diagram into audience-specific views. The takeaway a good
answer reaches: documentation is the deliverable that outlives you, its whole art is choosing views by
audience and keeping each at one abstraction level, and the highest-leverage, most-neglected content is
the reasoning — the <em>why</em> a diagram can never show.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-view"></a>**view** — a diagram or document that shows the system from *one* audience's angle (a developer's, an operator's, an exec's) rather than everything at once.
- <a id="w-4-1"></a>**4+1** — a classic idea: describe an architecture through several complementary views (logical, process, development, physical) tied together by scenarios.
- <a id="w-c4-model"></a>**C4 model** — Context → Container → Component → Code: four zoom levels for describing software structure, most-zoomed-out first.
- <a id="w-container-in-c4"></a>**container (in C4)** — *not* a Docker container; a separately deployable/runnable thing (an app, a service, a database, a single-page app). A unit of deployment.
- <a id="w-diagram-as-code"></a>**diagram-as-code** — diagrams written as text (so they can be versioned and diffed) rather than drawn in a tool.
- <a id="w-arc42-views-and-beyond"></a>**arc42 / views-and-beyond** — templates for a full architecture documentation package (not just diagrams).
- <a id="w-legend"></a>**legend** — the key that says what each shape, line, and colour means; a diagram without one is guesswork.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 29 — ADRs & Capturing Decisions →](lesson-29-adrs){: .btn .btn-primary }
