---
title: "Lesson 17 — Distributed Data (Transactions, Sagas, Outbox, Idempotency)"
nav_order: 4
parent: "Phase 4: Distributed Systems"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 17: Distributed Data (Transactions, Sagas, Outbox, Idempotency)

{: .note }
> **Words to know**
> - **ACID transaction** — all-or-nothing, consistent, isolated, durable; the guarantee a single database gives you.
> - **two-phase commit (2PC)** — a protocol to make a transaction span multiple databases; correct but slow and fragile at scale.
> - **saga** — a sequence of local transactions across services, with **compensating** actions to undo earlier steps if a later one fails.
> - **compensating transaction** — an action that semantically undoes a completed step (refund a charge, release a reservation).
> - **dual-write problem** — updating a database *and* publishing a message as two separate steps that can partially fail.
> - **transactional outbox** — write the event to the same database, in the same transaction, then publish it separately — solving the dual-write.
> - **idempotency** — processing the same message twice has the same effect as processing it once.

## Concept

Here's the hardest truth of distributed data: **the moment you split data across services,
you lose the database transaction.** In a monolith, "charge the customer AND reserve stock"
is one ACID transaction — both happen or neither does, guaranteed by the database. Split
Payment and Inventory into separate services with separate databases, and no single
transaction can span them. You must now maintain consistency *yourself*, across the network,
in the face of partial failures. This is the tax that Lesson 15's eventual consistency and
Lesson 14's fallacies cash out into concrete engineering.

```
   MONOLITH (one database)              DISTRIBUTED (separate databases)
   BEGIN TRANSACTION                    Payment.charge()   ← succeeds
     charge customer                    Inventory.reserve() ← FAILS
     reserve stock                      ...now what? Customer is charged,
   COMMIT  (both, or neither)           stock isn't reserved. No rollback.
   ────────────────────────             ────────────────────────────────
   The DB guarantees atomicity.         YOU must guarantee it, with:
                                        • SAGA (compensating transactions)
                                        • OUTBOX (reliable event publish)
                                        • IDEMPOTENCY (safe retries)
```

There is no magic that gives you back cross-service ACID at scale. Instead you accept
*eventual* consistency and engineer three things to make it correct: **sagas** (undo on
failure), the **outbox** (publish events reliably), and **idempotency** (survive the retries
that are guaranteed to happen). Together they're how real distributed systems stay correct
without a distributed transaction.

## Going Deeper

**Why two-phase commit doesn't save you.** 2PC *does* let a transaction span multiple
databases (a coordinator asks all participants "can you commit?", then "commit"), and it's
correct — so why not use it? Because it scales terribly and couples everything: it holds locks
across services for the duration (killing throughput and concurrency), and it's fragile to
partitions (if the coordinator dies at the wrong moment, participants are left blocked,
holding locks, unsure whether to commit). It reintroduces exactly the tight temporal coupling
and availability-multiplication microservices were meant to escape. So at scale, 2PC is
generally avoided; you accept eventual consistency and use sagas instead.

**The saga pattern.** A saga is a sequence of *local* transactions (each service commits in
its own database), coordinated so that if a later step fails, **compensating transactions**
undo the earlier ones. "Charge, then reserve, then ship" — if reserve fails after charge, run
the compensating action *refund*. Compensations are *semantic* undo, not a rollback (you can't
un-charge; you issue a refund). Two flavors (Lesson 12): **orchestrated** (a central saga
coordinator explicitly drives the steps and compensations — legible, controllable, a single
place to see the workflow) and **choreographed** (each service reacts to events and emits the
next — decoupled but the flow is emergent and harder to follow). For anything with real
compensation logic, orchestration is usually worth it because you can *see* the process.

{: .warning }
> **The dual-write problem and the transactional outbox**
> A subtle, common bug: a service needs to <em>update its database</em> AND <em>publish an
> event</em> ("I saved the order; now tell everyone <code>OrderPlaced</code>"). If these are
> two separate operations, they can partially fail — the DB write commits but the publish
> fails (event lost, downstream never reacts), or the publish succeeds but the DB write rolls
> back (a phantom event for something that didn't happen). You <em>cannot</em> wrap a database
> and a message broker in one transaction. The fix is the <strong>transactional outbox</strong>:
> write the event into an <code>outbox</code> table <em>in the same database transaction</em>
> as the business change (so they commit atomically — either both or neither), then a separate
> process reads the outbox and publishes the events to the broker, marking them sent. The
> event is now guaranteed to be published if and only if the business change committed. (A
> related technique, <em>change data capture</em>, tails the DB log to publish changes.) The
> outbox is the standard, correct answer to "save and publish reliably," and forgetting it is
> a leading cause of lost or phantom events.

**Idempotency is survival, not a nicety.** In a distributed system, retries are *guaranteed*
— networks time out, so callers retry, and brokers deliver at-least-once, so consumers get
duplicates. If processing a message twice does the wrong thing (charges twice, ships twice,
double-decrements stock), you have a correctness bug that <em>will</em> fire in production.
So **every consumer and every operation that can be retried must be idempotent**: processing
the same message twice has the same effect as once. The usual mechanism is an
**idempotency key** — a unique id on the request/message; the consumer records which keys it
has processed and ignores (or returns the cached result of) a duplicate. "Exactly-once
delivery" is mostly a myth (the network can't guarantee it); what you actually build is
"at-least-once delivery + idempotent processing = effectively-once *processing*." Idempotency
is the load-bearing technique that makes at-least-once safe.

**Eventual consistency is a business conversation, not just a technical one.** When you give
up the cross-service transaction, you're accepting a window where the system is inconsistent
(charged but not yet reserved; order placed but email not yet sent). Whether that's acceptable
— and how it should resolve when a step fails (refund? backorder? alert a human?) — is a
*business* decision, not a technical default. The architect's job is to surface it: "we can't
make charge-and-reserve atomic across services; if reserve fails after charge, do we refund
automatically, hold and retry, or backorder?" That question, handed to the product owner, is
the honest expression of the distributed-data trade-off (echoing Lesson 12's oversell-vs-latency
call).

---

## Lab — Design Exercise

**The situation:** You're building a travel-booking feature: a trip books a **flight**, a
**hotel**, and a **rental car**, each owned by a separate service with its own database. The
booking should be all-or-nothing *from the customer's perspective*: if any of the three can't
be booked, the others must be undone. There is no shared database and no distributed
transaction.

**Design this as a saga.** Specify: the sequence of local transactions and their compensating
actions; whether you'd orchestrate or choreograph and why; where the transactional outbox
applies; and how idempotency keys make the guaranteed retries safe. Then name the
consistency window the customer might observe and how you'd handle a compensation that itself
fails.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is assembling saga + compensation + outbox + idempotency into a correct design. A
strong answer:
<br><br>
<strong>The saga steps and compensations:</strong>
<ol>
<li><em>Book flight</em> (local transaction in Flight service) → compensation: <em>cancel
flight booking</em> (with any cancellation policy applied).</li>
<li><em>Book hotel</em> (local transaction in Hotel service) → compensation: <em>cancel hotel
booking</em>.</li>
<li><em>Book car</em> (local transaction in Car service) → compensation: <em>cancel car
booking</em>.</li>
</ol>
If step 3 (car) fails, run the compensations for steps 2 and 1 (cancel hotel, cancel flight),
leaving the customer as if nothing was booked. If step 2 fails, compensate step 1. Note the
compensations are <em>semantic undo</em>, not rollbacks — you issue a cancellation, which may
have its own rules (fees, a refund) — that's inherent to sagas and a point to surface to the
business (see below).
<br><br>
<strong>Orchestrate, not choreograph.</strong> This flow has real compensation logic and a
strict all-or-nothing intent, so use an <strong>orchestrated saga</strong>: a
<code>TripBookingOrchestrator</code> that explicitly drives book-flight → book-hotel →
book-car, and on any failure drives the compensations in reverse. Reasoning: the workflow and
its failure/undo paths are complex enough that you want them <em>visible and controllable in
one place</em> (Lesson 12/16) — with choreography the "undo everything on failure" logic would
be smeared across three services reacting to each other's failure events, which is far harder
to reason about, test, and debug. Orchestration gives you a single, legible state machine for
the booking's lifecycle.
<br><br>
<strong>Where the outbox applies:</strong> each step is "update <em>my</em> database AND emit
an event/command the orchestrator (or next step) reacts to" — the classic dual-write. So each
service uses a <strong>transactional outbox</strong>: e.g., the Flight service writes the
booking row and a <code>FlightBooked</code> outbox event <em>in one local transaction</em>, and
a separate publisher relays it to the orchestrator/broker. This guarantees the event fires if
and only if the booking actually committed — no phantom "booked" events for bookings that rolled
back, and no lost events for bookings that succeeded. The orchestrator itself persists its saga
state transactionally (and can use an outbox for the commands it issues), so a crash mid-saga
can be resumed from durable state.
<br><br>
<strong>Idempotency for the guaranteed retries:</strong> the orchestrator will retry steps on
timeout (it can't tell "the booking failed" from "the booking succeeded but the ack was lost"),
and the broker delivers at-least-once. So <em>every</em> step and compensation carries an
<strong>idempotency key</strong> (e.g., a per-saga, per-step unique id). Each service records the
keys it has processed: a duplicate "book flight" with a key it has already seen returns the
existing booking rather than booking a second flight; a duplicate "cancel" that's already been
applied is a no-op. This turns at-least-once delivery into effectively-once <em>processing</em> —
without it, a retried "book flight" double-books and a retried compensation could over-refund.
Idempotency is what makes the whole retry-based saga safe.
<br><br>
<strong>The consistency window and failed compensations:</strong> the customer may briefly
observe a <em>partial</em> state — the flight is booked but the hotel isn't yet, or a failed
booking is mid-unwind — because there's no atomic cross-service commit; the "all-or-nothing"
is eventual, achieved by the saga converging, not instantaneous. You handle this by not
showing the trip as "confirmed" until the saga completes (show "booking…"), so the customer's
view reflects the true state. For a <strong>compensation that itself fails</strong> (you
charged/booked the flight but the cancel call keeps failing): retry the compensation with
backoff (idempotently), and if it still won't succeed after a bounded number of attempts, route
it to a <strong>dead-letter queue / human intervention</strong> and alert — because a stuck
compensation means real money/inventory is in a wrong state that must be reconciled, and
silently dropping it is the worst outcome. This is why sagas need durable state and monitoring:
the failure modes don't disappear, they become things you must detect and resolve.
<br><br>
<strong>Surface the business decision:</strong> tell the product owner that "all-or-nothing"
across three providers means eventual (saga-based) consistency with <em>real</em> compensations
— e.g., cancelling a booked flight may incur a fee or a delayed refund, and there's a brief
window of partial state. The questions for them: on a failed booking, do we auto-cancel-and-
refund (accepting any fees) or hold-and-retry? How do we present "booking in progress" to the
customer? Those are business calls the distributed design forces into the open — and surfacing
them, rather than silently choosing, is the architect's job (Lesson 15/12's recurring theme).
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| The saga pattern | Chris Richardson — <https://microservices.io/patterns/data/saga.html> |
| Transactional outbox | <https://microservices.io/patterns/data/transactional-outbox.html> |
| Idempotency & idempotency keys | Stripe API docs — <https://docs.stripe.com/api/idempotent_requests> |
| Why 2PC doesn't scale; distributed transactions | *Designing Data-Intensive Applications*, Kleppmann (Ch. 9) |
| Database-per-service | <https://microservices.io/patterns/data/database-per-service.html> |

---

## Checkpoint

**Q1.** Why do you lose ACID transactions when you split data across services, why isn't
two-phase commit the answer at scale, and what does a saga give you instead (including the
nature of "compensation")?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Why you lose ACID:</strong> an ACID transaction is a guarantee provided by a
<em>single database</em> — it can make "charge AND reserve" atomic (all-or-nothing) because
both operations are in one database it fully controls. When you split Payment and Inventory
into separate services with separate databases, no single database controls both, so there is
<em>no</em> transaction that can span them: you make two independent local commits over a
network, and either can succeed while the other fails (charged but not reserved), with no
built-in rollback. The atomicity the database used to guarantee is simply gone; you now have
partial-failure states to handle yourself.
<br><br>
<strong>Why not 2PC:</strong> two-phase commit <em>can</em> technically span multiple databases
(a coordinator runs "prepare" then "commit" across participants) and is correct, but it doesn't
scale: it holds locks across all participants for the duration of the transaction (destroying
throughput and concurrency), and it's fragile under partitions — if the coordinator fails at
the wrong moment, participants are left <em>blocked</em>, holding locks, not knowing whether to
commit or abort. It reintroduces exactly the tight temporal coupling and availability-
multiplication that distribution was meant to avoid (every participant must be up and locked
together). So at scale it's generally rejected in favor of accepting eventual consistency.
<br><br>
<strong>What a saga gives you instead:</strong> a saga replaces one distributed ACID transaction
with a <em>sequence of local transactions</em> (each service commits in its own database
normally) plus <strong>compensating transactions</strong> that undo earlier steps if a later
step fails. Instead of atomic all-or-nothing enforced by a database, you get
<em>eventual</em> all-or-nothing enforced by the saga: do step 1, step 2, step 3; if step 3
fails, run the compensations for 2 and 1. Crucially, a compensation is a <strong>semantic
undo, not a rollback</strong> — you can't erase a completed local commit, so you perform a new
action that <em>counteracts</em> it (you can't un-charge a card, so you issue a refund; you
can't un-book, so you cancel). This means compensations have real-world semantics and sometimes
costs (a cancellation fee, a delayed refund), and there's a window where the system is partially
done before the saga converges — the price of trading a strong, instant, single-database
transaction for one that works across independently-owned services without their tight coupling.
The saga doesn't restore ACID; it gives you a disciplined way to reach a consistent end state
eventually, in exchange for accepting temporary inconsistency and semantic (rather than
automatic) undo.
</details>

**Q2.** Explain the dual-write problem and how the transactional outbox solves it, and
separately explain why idempotency is mandatory (not optional) in a distributed system. How do
they combine to give "effectively-once" processing?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>The dual-write problem:</strong> a service often needs to do two things — update its
own database <em>and</em> publish a message/event ("I saved the order; now emit
<code>OrderPlaced</code>"). A database and a message broker are separate systems that
<em>cannot</em> be enrolled in one transaction, so if you do them as two separate steps, they
can partially fail: the DB commit succeeds but the publish fails (the event is <em>lost</em> —
downstreams never learn the order happened), or the publish succeeds but the DB write rolls
back (a <em>phantom</em> event for something that didn't actually happen). Either way the system
is left inconsistent, and it's a common, subtle production bug.
<br><br>
<strong>The transactional outbox solves it</strong> by making the "publish" part of the
<em>same local transaction</em> as the business write. Instead of publishing directly, the
service inserts the event into an <code>outbox</code> table in the same database, within the
same transaction as the order write — so they commit atomically (both or neither). A separate
publisher process then reads unsent rows from the outbox and publishes them to the broker,
marking them sent (retrying until it succeeds). Now the event is guaranteed to be published
<em>if and only if</em> the business change committed: no lost events (the outbox row is
durably committed with the data), no phantom events (if the transaction rolls back, the outbox
row rolls back too). It converts an impossible atomic DB-plus-broker write into an atomic
DB-plus-DB write followed by a reliable, retryable relay.
<br><br>
<strong>Why idempotency is mandatory:</strong> in a distributed system, <em>retries and
duplicates are guaranteed, not exceptional</em>. Callers can't distinguish "the operation
failed" from "it succeeded but the acknowledgement was lost," so they retry; brokers deliver
<em>at-least-once</em>, so consumers receive duplicates; the outbox publisher itself may publish
a message more than once after a crash. So any operation that can be retried <em>will</em> be
invoked more than once with the same intent — and if processing it twice does the wrong thing
(charges twice, ships twice, double-decrements stock), that's a correctness bug that
<em>will</em> fire in production. Therefore every retryable operation/consumer must be
<strong>idempotent</strong>: processing the same message twice has the same effect as once,
typically via an <strong>idempotency key</strong> the consumer records and dedupes on.
<br><br>
<strong>How they combine into "effectively-once":</strong> true exactly-once <em>delivery</em>
is essentially impossible over an unreliable network — you cannot guarantee a message is
delivered precisely once. So the achievable design is <strong>at-least-once delivery +
idempotent processing</strong>: the broker (and outbox) guarantee the message is delivered
<em>at least</em> once (never lost — durability), and idempotency guarantees that processing it
<em>more</em> than once is harmless (duplicates are absorbed). The net observable behavior is
that each logical operation takes effect <em>exactly once</em> — "effectively-once processing"
— even though delivery was at-least-once. The outbox ensures the event reliably exists and gets
published; idempotency ensures the guaranteed duplicates don't cause double-effects. Together
they're how distributed systems stay correct without a distributed transaction: reliable
publication (outbox) + safe reprocessing (idempotency) = consistency achieved eventually and
exactly, on top of an unreliable, at-least-once substrate.
</details>

---

## Homework

Find one place in your system (or a system you know) where a business operation spans two or
more separate data stores or services and needs to stay consistent — a checkout, a signup that
provisions across systems, an update that must propagate. Analyze it: is it currently relying
(incorrectly) on hope, doing an unsafe dual-write, or lacking idempotency so retries could
double-apply? Then redesign it properly: the saga steps and compensations, where an outbox is
needed for reliable event publishing, and the idempotency keys that make retries safe. Finally,
name the business decision the eventual consistency forces (what happens on a failed step) that
should be surfaced to a product owner.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise applies the full distributed-data toolkit to real code. A strong response:
<br><br>
<strong>Diagnoses the current risk honestly.</strong> The common findings on a real
multi-store operation are one or more of: (a) an <em>unsafe dual-write</em> — the code updates
a database and then publishes an event (or calls another service) as separate steps, with no
outbox, so a crash between them loses the event or leaves the systems diverged; (b) <em>missing
idempotency</em> — a retry (from a timeout, a broker redelivery, a user double-click) would
double-apply the operation (double charge, double provision, double decrement), a latent
correctness bug; (c) <em>no compensation path</em> — if a later step fails, earlier steps are
left committed with no undo, so the system is silently left inconsistent and "fixed" by manual
database surgery when someone notices. Naming which of these the real operation suffers from is
the diagnostic value.
<br><br>
<strong>Redesigns with the right pieces.</strong> A strong redesign names: the <em>saga</em>
steps as local transactions with explicit <em>compensations</em> (and whether to orchestrate —
usually yes if there's real undo logic — or choreograph); the <em>transactional outbox</em> at
each step that must both update its DB and emit an event, so publication is atomic with the
data and can't be lost or phantomed; and the <em>idempotency keys</em> on every step and
compensation so the guaranteed retries and duplicates are absorbed (at-least-once + idempotent =
effectively-once). It also handles the failure-of-a-compensation case (retry with backoff, then
dead-letter + human alert) rather than assuming compensations always succeed.
<br><br>
<strong>Surfaces the business decision.</strong> The final, distinctly-architectural step:
identify the choice the eventual consistency forces and that a product owner — not an engineer —
should make. On a failed step: auto-compensate-and-refund (accepting any fees), hold-and-retry,
or backorder/apologize? What does the user see during the window of partial state
("processing…")? How long before a stuck saga escalates to a human? Recognizing that the
distributed design converts a hidden technical detail into an explicit business trade-off — and
that the right move is to name it and hand it up, not silently choose — is the mark of the
lesson landing. The overall takeaway: cross-store consistency in a distributed system isn't free
or automatic; it's engineered from sagas, outboxes, and idempotency, and it always carries a
business decision about how failures resolve.
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 18 — Reliability & Resilience Patterns →](lesson-18-resilience){: .btn .btn-primary }
