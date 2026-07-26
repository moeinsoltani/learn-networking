---
title: "Lesson 20 — Event Sourcing & CQRS"
nav_order: 2
parent: "Phase 5: Data & Scale"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 20: Event Sourcing & CQRS

{: .note }
> **Words to know**
> - **CQRS** — Command Query Responsibility Segregation: separate the *write* model from the *read* model.
> - **command** — an operation that changes state (a write); **query** — an operation that reads state.
> - **event sourcing** — store the sequence of state-changing *events* as the source of truth, instead of just the current state.
> - **projection / read model** — a view built by processing events, shaped for a specific query.
> - **fold / replay** — deriving current state by applying all past events in order.
> - **snapshot** — a saved current-state checkpoint so you don't replay from the beginning every time.
> - **audit trail** — a complete, immutable history of what happened and when.

## Concept

Two powerful patterns, frequently confused with each other and frequently over-applied.
They're *independent* (you can use either alone), and each trades significant complexity for
significant capability — so the architect's job is knowing when that trade is worth it.

**CQRS** separates the model you use to *change* data (commands/writes) from the model you use
to *read* it (queries). Instead of one model serving both, you have a write model optimized for
consistency and validation, and one or more read models optimized for specific queries — often
in different shapes, even different stores, scaled independently.

**Event sourcing** stores the *sequence of events* that happened, as the source of truth,
rather than just the current state. Current state is *derived* by replaying the events.

```
   TRADITIONAL (state-oriented)        EVENT SOURCING (event-oriented)
   account: { balance: 80 }            events: [ Opened,
   (you see 80; the history            +100 Deposited,
    of how it got there is lost)        -30 Withdrawn,
                                         +10 Deposited ]
   UPDATE overwrites the past.          → current balance = fold = 80
                                         → the full history IS the data;
   CQRS: one WRITE model (validate,       nothing is overwritten.
   commit events/state) + separate     
   READ models (shaped per query),     Often paired with CQRS: events are the
   kept in sync.                        write side; projections build read models.
```

They pair naturally (event sourcing is a write model; CQRS projections turn events into read
models), which is why they're taught together — but you can do CQRS with a normal database and
no events, and you can event-source without full CQRS. Keep them mentally separate.

## Going Deeper

**CQRS — what it buys and costs.** The core insight: reads and writes often have opposite needs.
Writes need a normalized, consistent, validated model; reads need denormalized, query-shaped,
fast-to-serve data — and the read/write ratio is often wildly lopsided (many more reads than
writes). CQRS lets each side be optimized independently: the write model enforces invariants; one
or more read models are shaped exactly for their queries (even in a different store — a search
index, a denormalized view, a cache), and scaled separately (add read replicas/projections
without touching writes). *Cost:* two models to maintain, and the read models are usually
**eventually consistent** with the writes (there's a lag between a write and the read model
reflecting it — Lesson 15). *When it's worth it:* complex domains where read and write models
genuinely diverge, or heavily lopsided read/write loads. *When it's overkill:* simple CRUD where
one model serves both fine — then CQRS is pure ceremony.

**Event sourcing — the genuine superpowers.** Storing events as truth gives capabilities that
are hard or impossible otherwise:
- **Perfect audit trail** — you have the complete, immutable history of everything that
  happened, for free. In regulated/financial domains this is enormous (you can prove exactly
  what occurred and when).
- **Temporal queries** — "what was the state on March 3rd?" is just "replay events up to March
  3rd." You can reconstruct any past state.
- **Rebuild any read model** — because events are the truth, you can build a *new* projection
  (a new read view) by replaying history, even one you didn't anticipate originally. Bug in a
  projection? Fix the code and replay.
- **Natural fit with event-driven architecture** (Lesson 12) — the events you store are the
  events you publish.

{: .warning }
> **Event sourcing's real costs — respect them**
> The superpowers come with heavy costs that sink teams who adopt it casually: (1)
> <strong>Eventual consistency everywhere</strong> — current state is derived from projections,
> which lag; "just read the row" is gone. (2) <strong>Event versioning forever</strong> — your
> events are permanent, so when your event schema changes (and it will over years), you must
> handle <em>every historical version</em> of every event forever (upcasting, versioning
> strategies) — a real, permanent tax. (3) <strong>You can't easily "just query the table"</strong>
> — answering an ad-hoc question means building a projection, not writing a SQL query; the
> flexibility of relational querying is gone from the write side. (4) <strong>Replay & snapshot
> complexity</strong> — replaying millions of events is slow, so you need snapshots, which add
> machinery. (5) <strong>A steep learning curve</strong> — most engineers haven't built this way,
> and mistakes (e.g., putting behavior/validation in the wrong place, or events that are really
> disguised CRUD) are common. The honest guidance from the community, including its advocates:
> <strong>you probably don't need event sourcing</strong> for most systems — use it where the
> audit/history/temporal capabilities are genuinely valuable (finance, ledgers, compliance-heavy
> domains, systems where "how did we get here?" is a first-class question), not because it sounds
> elegant.

**They're independent — don't cargo-cult the pair.** A frequent mistake is adopting "CQRS +
event sourcing" as one inseparable, impressive-sounding package for a system that needed neither.
You can: do CQRS with two normal database models and no events (common and useful — e.g., a
denormalized read replica for a reporting view). Event-source one aggregate while the rest of the
system is plain CRUD. Use event sourcing without elaborate CQRS. Treating them as a mandatory
duo, applied system-wide, is how teams bury a simple domain under enormous accidental complexity.
Apply each, independently, where its specific trade pays off.

**Selective application is the mark of judgment.** The best use is usually *targeted*: event-
source the *one* part of the system where audit/history is genuinely valuable (the ledger, the
order lifecycle, the workflow with compliance requirements), and keep the rest as normal CRUD or
light CQRS. A payments ledger event-sourced (perfect audit, replayable, immutable history) inside
a system whose user-profile and catalog modules are plain CRUD is a mature design. "Event-source
everything" is not.

---

## Lab — Design Exercise

**The situation:** For each of these three systems, decide whether to use **event sourcing** (and
whether CQRS), and defend the yes/no strictly from the trade-offs — the audit/history/temporal
value on one side, the eventual-consistency / versioning / learning-curve costs on the other:

1. A **bank account ledger** — records deposits, withdrawals, transfers; must have a complete,
   provable, immutable audit trail for regulators; "what was the balance on date X" is a real
   question; correctness of history is paramount.
2. A **CMS (content management system)** — editors create and edit articles; the current
   published version is what matters; occasional "who changed this and can we revert" is nice but
   not central; mostly straightforward CRUD.
3. An **IoT telemetry pipeline** — millions of sensor readings per minute, append-only, queried
   for analytics and time-ranges; no complex per-entity business rules; extreme write volume.

**For each: event source or not, CQRS or not, and why — from the trade-offs, not the fashion.**

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is weighing the specific capability against the specific cost, per system. A strong
answer:
<br><br>
<strong>1. Bank ledger → YES, event source (and CQRS for the read side). The textbook fit.</strong>
Every superpower is exactly what this domain needs: a <em>complete, immutable, provable audit
trail</em> (regulatory requirement — you must prove precisely what happened; event sourcing gives
it for free and by construction, whereas a state-oriented design that overwrites balances
<em>destroys</em> exactly the history regulators demand); <em>temporal queries</em> ("balance on
date X" = replay to date X — a real, required question here); immutability matching the domain's
nature (a ledger <em>is</em> an append-only history of transactions — event sourcing models it
directly rather than fighting it). The costs are acceptable/worth it: eventual consistency on read
projections is manageable and the correctness lives in the immutable event log; event versioning
is a real tax but justified; the learning curve is worth paying for a domain where "how did we get
here?" is a first-class, legally-required question. CQRS pairs well: the event log is the write
model; build read projections (current balances, statements) shaped per query. This is the case
event sourcing was made for.
<br><br>
<strong>2. CMS → NO, don't event source; plain CRUD (maybe light versioning). </strong> The access
pattern is state-oriented: <em>the current published version is what matters</em>, editors
overwrite content, and the queries are "get the current article." The audit/history value is
<em>nice-to-have, not central</em> — and the modest "who changed this / revert" need is served far
more cheaply by simple versioning (keep prior versions of a document, or a change-log table) than
by full event sourcing. Weighing the trade: the superpowers (perfect audit, temporal queries,
replayable projections) buy little here (nobody needs "the article's state on March 3rd" for
compliance), while the costs (eventual consistency, permanent event versioning, learning curve,
"can't just query the table") are pure overhead burying a simple CRUD domain. Event sourcing here
is over-engineering — the classic "you probably don't need event sourcing" case. Plain CRUD with a
version history if wanted; no CQRS ceremony needed.
<br><br>
<strong>3. IoT telemetry → NO (not event sourcing in the DDD sense); use an append-only /
time-series store, and CQRS-<em>ish</em> read/write separation is natural.</strong> This is a
subtle one and a good discriminator. The data is append-only events, which <em>looks</em> like
event sourcing — but event sourcing is about deriving <em>domain state</em> for <em>aggregates
with business rules</em> by folding events, and here there are <em>no per-entity business
invariants to enforce</em> and no "current state of an aggregate" to rebuild; it's just a firehose
of readings you store and query by time. So the right tool is a <strong>time-series / append-only
store</strong> (Lesson 19) built for extreme write volume and time-range queries — not an
event-sourcing framework with aggregates, folds, snapshots, and compensations, which would add
irrelevant machinery. You <em>do</em> naturally separate the high-volume write path from the
analytics read path (a CQRS-flavored split — write to the ingest store, project into analytics
views), but that's read/write separation for scale, not domain event sourcing. The lesson:
"append-only events" is not automatically "event sourcing" — event sourcing is a
<em>domain-modeling</em> choice for stateful aggregates, and applying its framework to a
stateless telemetry firehose is a category error.
<br><br>
<strong>The meta-lesson:</strong> the same question gets three different answers because the
<em>value of the capability</em> differs — event sourcing shines where an immutable, provable,
temporally-queryable history is genuinely valuable (the ledger), is overkill where the domain is
state-oriented CRUD (the CMS), and is the wrong <em>category</em> where append-only data has no
domain aggregates to source (the telemetry firehose, which wants a time-series store). And CQRS
tracks a separate question (do reads and writes diverge enough, or is the load lopsided enough, to
justify separate models?) — yes for the ledger's read projections and the telemetry's write/read
split, no for the simple CMS. The discipline: apply each pattern, independently, where its
specific trade pays off — never as a mandatory, impressive-sounding duo applied everywhere.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| CQRS | Martin Fowler — <https://martinfowler.com/bliki/CQRS.html> |
| Event sourcing | Martin Fowler — <https://martinfowler.com/eaaDev/EventSourcing.html> |
| "You probably don't need event sourcing" (cautions) | community consensus; see Greg Young's talks & Fowler's caveats |
| Event sourcing & CQRS in practice | Vaughn Vernon, *Implementing Domain-Driven Design* (Ch. on ES/CQRS) |
| Event sourcing pattern (with trade-offs) | <https://microservices.io/patterns/data/event-sourcing.html> |

---

## Checkpoint

**Q1.** Explain the difference between CQRS and event sourcing, why they're independent, and why
treating them as one mandatory package is a mistake.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>CQRS</strong> (Command Query Responsibility Segregation) is about <em>separating the read
model from the write model</em>: instead of one model serving both, you have a write model
optimized for validation/consistency and one or more read models shaped for specific queries
(possibly in different stores, scaled independently). It says nothing about <em>how</em> you store
data — you can do CQRS with two ordinary database models.
<br><br>
<strong>Event sourcing</strong> is about <em>what you store as the source of truth</em>: instead
of storing current state (and overwriting it on each change), you store the ordered sequence of
<em>events</em> that happened, and derive current state by replaying them. It says nothing about
separating reads from writes — you can event-source with a single model.
<br><br>
<strong>Why they're independent:</strong> they answer different questions. CQRS answers "should
reads and writes use the same model?" (a structural separation). Event sourcing answers "should I
store state or the history of changes?" (a storage-of-truth choice). You can have either without
the other: CQRS with a normal state-based database and no events (common — e.g., a denormalized
read replica for reporting alongside a normalized write DB); event sourcing with a single model
and no elaborate read/write split. Neither implies the other.
<br><br>
<strong>Why they pair</strong> (which causes the confusion): event sourcing gives you a stream of
events as the write side, and CQRS projections are a natural way to turn those events into
query-shaped read models — so they fit together well, and are often taught and used together.
<br><br>
<strong>Why treating them as one mandatory package is a mistake:</strong> because each carries its
<em>own</em> significant complexity and its <em>own</em> justification, and bundling them means
adopting <em>both</em> costs when your system may need <em>neither</em> — or need only one. A team
that hears "CQRS + event sourcing" as a single impressive pattern and applies it wholesale to a
simple CRUD domain buries that domain under two layers of accidental complexity (separate read/
write models <em>and</em> event storage with versioning, replay, eventual consistency) for
benefits it doesn't use. The disciplined approach is to evaluate each independently: use CQRS
where reads and writes genuinely diverge or the load is lopsided; use event sourcing where an
immutable, auditable, temporally-queryable history is genuinely valuable — and apply each
<em>selectively</em> (often to just one part of the system), not as an inseparable, system-wide
duo. Keeping them mentally separate is what lets you take the one you need without paying for the
one you don't.
</details>

**Q2.** Event sourcing has real superpowers and real costs. Name two superpowers and two costs,
and describe the kind of domain where the trade clearly pays off versus where "you probably don't
need it" applies.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Two superpowers:</strong> (1) <em>A complete, immutable audit trail</em> — because you
store every event and never overwrite, you have the full, provable history of everything that
happened and when, for free; nothing is lost. (2) <em>Temporal queries and rebuildable read
models</em> — you can reconstruct the state at <em>any</em> past point ("what was it on date X?" =
replay to date X), and you can build a <em>new</em> read projection you didn't originally
anticipate by replaying history (and fix a buggy projection by replaying), because the events are
the truth and the views are derived. (Also valid: natural fit with event-driven architecture — the
events you store are the events you publish.)
<br><br>
<strong>Two costs:</strong> (1) <em>Eventual consistency and loss of "just query the table"</em> —
current state is derived from lagging projections, so you can't simply read a row for the truth,
and ad-hoc questions require building a projection rather than writing a SQL query. (2)
<em>Permanent event versioning</em> — events are stored forever, so when the event schema evolves
over years you must handle every historical version of every event indefinitely (upcasting/
versioning), a real and permanent tax; plus replay/snapshot machinery and a steep learning curve.
<br><br>
<strong>Where the trade pays off:</strong> domains where the audit/history/temporal capabilities
are <em>genuinely and often centrally valuable</em> — financial ledgers and accounting (a provable,
immutable transaction history is a regulatory requirement and matches the domain's append-only
nature), compliance-heavy or regulated systems, and workflows where "how did we get to this state,
exactly, and when did each thing happen?" is a first-class question (order/claim lifecycles,
systems needing full reconstructability). In these, the immutable history isn't a nice extra —
it's a core requirement, so the costs are worth paying and event sourcing often models the domain
<em>better</em> than a state-overwriting design (which would destroy the very history you need).
<br><br>
<strong>Where "you probably don't need it" applies:</strong> ordinary state-oriented domains where
the <em>current</em> state is what matters and history is at most a nice-to-have — typical CRUD
applications (a CMS, a user profile, a product catalog), where nobody needs provable temporal
reconstruction and a simple version-history table would cover the modest "who changed this" need.
Here the superpowers buy little while the costs (eventual consistency, permanent versioning,
learning curve, no easy querying) are pure overhead that buries a simple domain — the classic
over-engineering trap. The honest default (echoed even by event sourcing's advocates) is that
<em>most</em> systems are in this second category, so event sourcing should be a deliberate,
targeted choice for the parts of a system where its specific capabilities are genuinely needed —
not a default reached for because it sounds sophisticated.
</details>

---

## Homework

Look at your current system for a place where event sourcing's capabilities would genuinely add
value — a part where audit trail, "how did we get here?", temporal reconstruction, or replayable
history is (or should be) a real requirement (a ledger, an order/workflow lifecycle, a
compliance-sensitive record). Would event sourcing that *one* part pay off, weighed against the
costs? Separately, consider CQRS: is there a place where your reads and writes have genuinely
divergent needs or a lopsided read/write load, such that separating the models (even with normal
databases, no events) would help? Be honest about where *neither* is justified and plain CRUD is
correct — the goal is calibrated, selective judgment, not adopting the patterns.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise trains selective, cost-weighed application rather than pattern enthusiasm. A strong
response:
<br><br>
<strong>Finds the targeted event-sourcing candidate (if any).</strong> The valuable move is
locating the <em>one</em> part of the system where the audit/history/temporal capabilities are a
genuine requirement, not a nice-to-have — commonly a financial/ledger component, an order or
claim lifecycle where "prove exactly what happened and when" matters, or a compliance-sensitive
record. A good answer then weighs honestly: does the immutable-history value here clearly exceed
the eventual-consistency, permanent-versioning, and learning-curve costs? Often the conclusion is
"yes for this <em>one</em> aggregate, no for the rest of the system" — which is exactly the mature,
selective application the lesson argues for. Equally valid and honest is "actually, even our most
audit-sensitive part is served well enough by an append-only audit-log table alongside normal
state, without full event sourcing" — recognizing the cheaper alternative is a sign of good
judgment, not a failure to find a use.
<br><br>
<strong>Evaluates CQRS separately.</strong> The independent question: is there a place where reads
and writes genuinely diverge (a complex write model but a very different, denormalized read shape)
or where the read/write load is heavily lopsided (vastly more reads than writes, or reads needing a
different store like a search index)? If so, separating the models — even with ordinary databases
and no events (a read replica, a denormalized reporting view, a search projection) — may help, and
naming that keeps CQRS decoupled from event sourcing in your mind. If reads and writes are served
fine by one model, the honest answer is "no CQRS needed here."
<br><br>
<strong>Names where neither applies.</strong> Crucially, a strong response identifies the parts
(usually the majority) where plain CRUD is <em>correct</em> and both patterns would be
over-engineering — resisting the pull to adopt an impressive pattern where a simple table serves
perfectly. The overall takeaway: these are targeted tools, applied to the specific parts of a
system whose requirements justify their cost, and evaluated independently of each other — the
calibrated conclusion is often "event-source this one ledger, add a read projection there, and
leave everything else as simple CRUD," which is a far more sophisticated answer than either "adopt
CQRS+ES everywhere" or "never use them."
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 21 — Caching Strategies →](lesson-21-caching){: .btn .btn-primary }
