---
title: "Lesson 29 — ADRs & Capturing Decisions"
nav_order: 2
parent: "Phase 7: Documenting, Evaluating & Evolving Architecture"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 29: ADRs & Capturing Decisions

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

Every significant architectural decision is made in a meeting, a thread, or someone's head — and then
the *reasoning evaporates*. Six months later, a new engineer sees the choice, doesn't understand why,
assumes it was arbitrary, and either reverses it (breaking something the original reason protected) or
reopens the whole debate from scratch. The decision survived in the code; the **why** did not. An
**Architecture Decision Record (ADR)** exists to fix exactly this: it's a short, dated document that
captures *one* significant decision — the context that forced it, the decision itself, and the
**consequences**[°](#w-consequences) — so the reasoning outlives the meeting.

Consider the same decision with and without a record.

**Without an ADR**, the decision gets made in a meeting or a Slack thread, and
the *reasoning* evaporates within weeks. Six months later somebody asks "why is
it like this?" — and there are only two available outcomes: relitigate the
decision from scratch, or reverse it and break something the original reasoning
would have protected.

**With an ADR**, there is a short dated document:

> **ADR-014: Use a saga for booking**
> **Context:** split data, no two-phase commit available…
> **Decision:** a choreographed saga
> **Consequences:** eventual consistency, compensating transactions to write
> **Alternatives:** 2PC — rejected because…
> **Status:** Accepted, 2026-03-02

What survives the meeting is the **why**, and it pays off in three ways: new
joiners can read the reasoning instead of asking, settled questions stop being
reopened, and you have the raw material for the evaluation work in Lesson 30.

An ADR is deliberately *lightweight* (a page, in the repo, in Markdown) and — crucially —
**immutable**: you don't edit an old decision, you write a *new* ADR that supersedes it. The single
most valuable part is the one most people skip: the **alternatives you considered and rejected**,
because that's what stops the next person from re-proposing an option you already ruled out. ADRs turn
architecture from a set of unexplained facts into a legible *history of reasoning* — which is what makes
onboarding fast, **relitigation**[°](#w-relitigation) rare, and later evaluation (Lesson 30) possible.

## Going Deeper

**The ADR structure.** The canonical format (Michael Nygard's) is short by design:
- **Title & status** — a number and a name ("ADR-014: Use a choreographed saga for booking"), plus
  status (proposed / accepted / superseded).
- **Context** — the forces at play *at the time*: the requirements, constraints, quality attributes,
  and situation that made a decision necessary. This is written in the present tense of the decision,
  and it's what makes the record understandable later ("ah, *that's* what was true then").
- **Decision** — what you decided, stated plainly and actively ("We will…").
- **Consequences** — what follows, **good and bad**. The honesty here is the point: name the downside
  you're accepting, not just the upside. A decision with only positive consequences listed is a sales
  pitch, not a record.
- **Alternatives considered**[°](#w-alternatives-considered) — the options you weighed and rejected, and *why*. (More below.)

That's it — a page. The lightness is a feature: a heavyweight process doesn't get used, and an ADR
nobody writes captures nothing.

**Record the rejected options — the most valuable part.** The instinct is to document only what you
chose. But the *rejected* alternatives are the highest-value content, because they pre-empt
**relitigation**: without them, every new team member who thinks of option B (which you already
considered and ruled out) will propose it, and you'll re-run the debate — costing time and eroding
confidence in the decision. With "Alternatives considered: option B — rejected because it couldn't meet
the p99 latency ASR; option C — rejected due to vendor lock-in" written down, the next person sees the
reasoning and moves on (or, if the *context has genuinely changed*, writes a new ADR superseding the old
one, which is exactly right). The rejected options are also the raw material for evaluating the
architecture later (Lesson 30) — they show which forces were weighed.

{: .warning }
> **Match the deliberation to the door-type (one-way vs two-way).**
> Not every decision deserves the same rigour. Bezos's frame (Lesson 2): a <strong>two-way door</strong>
> is easily reversible — decide fast, try it, undo it if wrong; over-deliberating it wastes time. A
> <strong>one-way door</strong> is hard/expensive to reverse (a public API contract, a data model, a
> core technology, a security boundary) — these deserve real deliberation, wider input, and a careful
> ADR, because the cost of being wrong is high and you can't cheaply back out. The <em>skill</em> is
> matching the deliberation budget to the door: don't ceremony a reversible choice to death, and don't
> speed-run an irreversible one. A common ADR failure is spending equal energy on everything; a better
> one is reserving heavyweight ADRs for the genuine one-way doors and keeping the rest light — <em>and
> recording which type you judged it to be</em>, so a reviewer can later ask "was that really a one-way
> door, and did the deliberation match?"

**Status lifecycle — ADRs are immutable, they supersede.** An ADR is a record of a decision *made at a
point in time*, so you never overwrite it — that would destroy the history. Instead it has a
lifecycle: **proposed** (under discussion), **accepted** (decided and in force), and later
**superseded** or **deprecated** (a newer ADR replaces it, with a link both ways: "superseded by
ADR-031"). This immutability is what makes ADRs a *history* rather than a snapshot — you can read the
chain and see how the architecture's reasoning evolved, and why. Overwriting an accepted ADR when you
change your mind loses exactly the thing ADRs exist to preserve.

**Keep them next to the code, and what ADRs are *for*.** ADRs live in the repository (commonly a
`doc/adr/` or `docs/decisions/` folder, in Markdown), so they're versioned with the code, reviewed in
PRs, and discoverable by anyone in the codebase — not in a wiki that rots (same logic as Lesson 28's
diagram-as-code). Their three payoffs: **onboarding** (a new engineer reads the ADR log and understands
*why* the system is the way it is, fast), **anti-relitigation** (decisions with recorded reasoning stop
being re-argued), and **raw material for evaluation and evolution** (Lessons 30–31: to judge or change
an architecture, you need to know what was decided and why). *(This deepens
[Leadership Lesson 7]({{ '/leadership/lessons/lesson-07-adrs.html' | relative_url }}) — there the lens
is team process and shared understanding; here it's the architect's discipline of making the technical
reasoning durable and legible.)*

---

## Lab — Design Exercise

**The situation:** Recall a real architectural decision from your past (or use this one if you'd rather:
"our monolith's orders and payments share a database transaction; we're splitting payments into its own
service, so we must decide how to keep a booking consistent across the two services now that we've lost
the shared transaction"). 

**Write a full ADR** for it — with all five parts, and *especially* the **alternatives considered**
(at least two rejected options with the reason each was rejected) and the **honest consequences** (name
a real downside you accepted). Then **critique your own ADR**: was this a one-way or two-way door, and
did the deliberation you actually did match the door-type?

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is a complete, honest record — including rejected options and a named downside — plus the
self-aware door-type critique. A strong answer (using the sample decision):
<br><br>
<strong>ADR-014: Use a choreographed saga for cross-service booking consistency</strong><br>
<strong>Status:</strong> Accepted — 2026-03-02<br>
<strong>Context:</strong> We are extracting Payments from the Orders monolith into its own service with
its own database (driven by independent scaling and team ownership — see ADR-011). This removes the
single ACID transaction that previously kept "order created" and "payment captured" atomically
consistent. We now need a way to keep a booking consistent across two services and two databases. Our
quality-attribute drivers: bookings must never end in a "charged but no order" or "order but not
charged" state that goes unresolved; the checkout path has a p95 latency budget of 800ms; the two teams
want to deploy independently.<br>
<strong>Decision:</strong> We will use a <strong>choreographed saga</strong> (Lesson 17): Orders and
Payments react to each other's events, each owning a compensating action (cancel order / refund
payment), with idempotency keys on every step and a transactional outbox to avoid dual-write loss.<br>
<strong>Consequences:</strong>
<ul>
<li><em>Good:</em> no distributed transaction / 2PC, so the two services stay loosely coupled and
independently deployable; each service scales and fails independently; the flow tolerates one service
being briefly down (events wait).</li>
<li><em>Bad (the downside we accept):</em> the booking is now <strong>eventually consistent</strong> —
there is a window where the order exists but payment isn't yet confirmed, which the UI and support
tooling must handle explicitly. Debugging a failed saga across two services is harder than reading one
transaction. And "compensating" a charge is a refund, not a true rollback (the customer briefly saw a
charge). We accept this because the alternative couplings are worse.</li>
</ul>
<strong>Alternatives considered:</strong>
<ul>
<li><em>Two-phase commit (2PC) across the two databases — rejected:</em> reintroduces tight temporal
coupling (both services must be up and locked together to commit), scales poorly, and undermines the
independent-deployability that motivated the split in the first place. It would defeat the purpose.</li>
<li><em>Keep orders and payments in one shared database with a shared transaction — rejected:</em> that
is <em>not</em> splitting the service; it recreates the coupling we're trying to remove and blocks
independent scaling/ownership (a distributed monolith if we split the code but not the data).</li>
<li><em>Orchestrated saga (a central coordinator) instead of choreographed — considered, deferred:</em>
would give a single place to see and control the flow (easier to debug), but adds a coordinator
component and more coupling to it; we chose choreography for now given only two participants, and noted
we'd revisit if the flow grows to many steps.</li>
</ul>
<br>
<strong>Self-critique — door-type:</strong> This is closer to a <strong>one-way door</strong> — the
consistency model and the split of data across services is expensive to reverse once each service owns
its data and other things depend on the events; so it deserved real deliberation, the rejected
alternatives written down, and wider input from both teams. If in reality we'd decided it in a hallway
in ten minutes and never recorded the alternatives, the critique names that mismatch: an irreversible
decision got a reversible-decision amount of deliberation, which is exactly the risk to flag. (Contrast:
choosing the retry backoff constant is a two-way door — tune it later — and would <em>not</em> warrant
a heavyweight ADR.) A good answer explicitly compares the effort spent to the door-type and says whether
they matched.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| ADRs — the original format | Michael Nygard — <https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions.html> |
| ADR tooling & templates | <https://adr.github.io/> |
| Lightweight ADRs | <https://martinfowler.com/articles/scaling-architecture-conversationally.html> |
| One-way vs two-way doors | <https://en.wikipedia.org/wiki/Two-way_door> ; Lesson 2 |
| ADRs as onboarding & process | [Leadership Lesson 7]({{ '/leadership/lessons/lesson-07-adrs.html' | relative_url }}) |

---

## Checkpoint

**Q1.** What are the five parts of an ADR, and why are the "alternatives considered" and honest
"consequences" the most valuable — and most often skipped — parts?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>The five parts:</strong> (1) <strong>Title & status</strong> (a numbered name + proposed/
accepted/superseded); (2) <strong>Context</strong> — the forces, requirements, constraints, and
situation <em>at the time</em> that made a decision necessary; (3) <strong>Decision</strong> — what you
decided, stated plainly ("We will…"); (4) <strong>Consequences</strong> — what follows, good and bad;
(5) <strong>Alternatives considered</strong> — the options weighed and rejected, and why.
<br><br>
<strong>Why alternatives-considered is most valuable (and skipped):</strong> people naturally document
only what they chose, so the rejected options — which take extra effort to write — get dropped. But
they're the highest-value content because they <em>prevent relitigation</em>: without them, every new
person who thinks of an option you already ruled out will re-propose it, forcing you to re-run the
debate; with them, the reader sees "option B — rejected because it couldn't meet the latency ASR" and
either accepts it or, if the context has genuinely changed, writes a superseding ADR (which is the
correct move). They also record which forces were weighed, which is exactly what you need to evaluate
the architecture later (Lesson 30).
<br><br>
<strong>Why honest consequences matter (and get softened):</strong> the temptation is to list only the
benefits — turning the ADR into a justification rather than a record. But every architectural decision
is a <em>trade-off</em> (Lesson 2), so a decision with no downsides listed is either dishonest or
un-thought-through. Naming the real cost you accepted (e.g., "this makes the booking eventually
consistent, which the UI must handle") is what makes the record trustworthy and useful: the next person
knows the downside was seen and accepted deliberately, not overlooked, and knows what to watch for. An
ADR that hides the trade-off fails at its one job — capturing the reasoning honestly.
</details>

**Q2.** Why are ADRs immutable (superseded rather than edited), why keep them next to the code, and how
does matching deliberation to door-type make the practice sustainable?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Immutability / supersession:</strong> an ADR records a decision made <em>at a point in time</em>
under the context that was true then. If you overwrite it when you later change your mind, you destroy
the very history the practice exists to preserve — you lose why the old decision was made and can no
longer see how the reasoning evolved. So instead an ADR has a status lifecycle: proposed → accepted →
<em>superseded/deprecated</em>, and when you change course you write a <em>new</em> ADR that supersedes
the old one (linked both ways). The chain of ADRs is a legible history of how the architecture's
reasoning changed, which is more valuable than a single always-current snapshot.
<br><br>
<strong>Next to the code:</strong> ADRs kept in the repo (a <code>docs/decisions/</code> folder in
Markdown) are versioned with the code, reviewed in PRs, and discoverable by anyone working in the
codebase — the same reason we prefer diagrams-as-code and infrastructure-as-code. In a separate wiki or
slide deck they rot, drift, and get lost; next to the code they stay findable and current, and the
decision to change one is itself part of the change's review.
<br><br>
<strong>Door-type matching makes it sustainable:</strong> if every decision got a heavyweight ADR and
lengthy deliberation, the practice would collapse under its own ceremony and people would stop doing it.
Matching effort to the <em>door-type</em> keeps it lean: <strong>two-way doors</strong> (easily
reversible — a retry constant, a library you can swap) get a light touch or no ADR — decide fast, change
later. <strong>One-way doors</strong> (hard to reverse — a public API contract, a data model, a core
technology, a consistency model) get the real deliberation, wider input, and a careful ADR with
alternatives, because being wrong is expensive and you can't cheaply back out. Reserving the rigour for
the decisions that warrant it means the ADRs that exist are the ones that matter, the process stays
usable, and — bonus — recording <em>which</em> door-type you judged it to be lets a later reviewer check
whether the deliberation actually matched the stakes.
</details>

---

## Homework

Look at how your team currently captures architectural decisions. Is there any durable record of *why*
the significant choices were made, or does the reasoning live only in people's heads and old Slack
threads? Find a recent significant decision and check: could a new engineer six months from now
understand why it was made, and what alternatives were rejected? If not, write the ADR for it now —
retroactively — with real alternatives and an honest downside. Then propose the lightest-weight ADR
practice your team would actually sustain (where the files live, which decisions warrant one, who
writes them). Bonus: find a decision currently being *relitigated* and notice whether a recorded ADR
would have prevented the re-argument.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
A strong response makes the abstract discipline concrete for a real team.
<br><br>
<strong>Diagnoses where the "why" currently lives.</strong> The usual finding is that significant
decisions have no durable record of reasoning — the <em>what</em> is in the code, but the <em>why</em>
and the rejected alternatives are gone, surviving only as tribal knowledge that walks out the door when
people leave. Recognizing that this is <em>why</em> onboarding is slow and old debates keep reopening is
the insight.
<br><br>
<strong>Writes a real retroactive ADR.</strong> Picking an actual recent decision and writing its ADR —
with genuine alternatives-considered (the options really weighed and why they lost) and an honest
consequence (a downside actually accepted) — is where it gets concrete. The test the answer should apply:
"could a new engineer in six months understand this from the record alone?" If writing it is hard because
no one remembers the alternatives, that itself proves the cost of not having captured them.
<br><br>
<strong>Proposes a <em>sustainable</em> practice.</strong> The architect's move is the lightest process
the team will actually keep doing: ADRs as Markdown in the repo, reviewed in PRs; a shared template; and
a clear bar for which decisions warrant one (the one-way doors — significant + hard to reverse — not
every choice). Over-engineering the process guarantees it dies; matching it to door-type keeps it alive.
<br><br>
<strong>The bonus lands the point.</strong> Finding a decision currently being relitigated — and seeing
that a recorded ADR with its rejected alternatives would have ended the re-argument in seconds — makes
the value visceral. The takeaway a good answer reaches: architecture is a history of reasoning, not a
set of unexplained facts; the reasoning (especially the rejected options and the honest trade-off) is
the part that evaporates and the part most worth preserving; and the practice only survives if it's
lightweight and reserved for the decisions that genuinely matter.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-adr-architecture-decision-record"></a>**ADR (Architecture Decision Record)** — a short, dated document capturing one significant decision: its context, the decision, why, and the consequences.
- <a id="w-context-in-an-adr"></a>**context (in an ADR)** — the forces and situation that made the decision necessary — what was true when you decided.
- <a id="w-consequences"></a>**consequences** — what results from the decision, good *and* bad; the trade-off you accepted, spelled out.
- <a id="w-alternatives-considered"></a>**alternatives considered** — the other options you weighed and rejected, and why — often the most valuable part.
- <a id="w-status-lifecycle"></a>**status lifecycle** — proposed → accepted → (later) superseded/deprecated; ADRs are immutable records, not living documents you overwrite.
- <a id="w-one-way-two-way-door"></a>**one-way / two-way door** — Bezos's terms: a hard-to-reverse decision (deliberate) vs an easy-to-reverse one (move fast).
- <a id="w-relitigation"></a>**relitigation** — re-arguing a decision that was already made, usually because no one recorded *why*.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 30 — Evaluating Architecture: ATAM, Trade-offs & Fitness Functions →](lesson-30-evaluating){: .btn .btn-primary }
