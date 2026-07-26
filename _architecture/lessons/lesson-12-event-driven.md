---
title: "Lesson 12 — Event-Driven Architecture"
nav_order: 4
parent: "Phase 3: Architectural Styles"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 12: Event-Driven Architecture

{: .note }
> **Words to know**
> - **event** — a record that something *happened* (past tense: `OrderPlaced`); a fact, not a request.
> - **command** — a request for something *to happen* (`PlaceOrder`); directed at one handler, can be rejected.
> - **broker / message bus** — infrastructure that carries messages between producers and consumers (Kafka, RabbitMQ, SNS/SQS).
> - **pub/sub** — publish–subscribe: a producer publishes; any number of subscribers receive, without the producer knowing them.
> - **choreography vs orchestration** — components react to events on their own (choreography) vs a central coordinator directs the steps (orchestration).
> - **temporal decoupling** — producer and consumer don't need to be running at the same time; the broker buffers.
> - **dead-letter queue (DLQ)** — where messages go when they can't be processed, so they're not lost.

## Concept

Event-driven architecture (EDA) flips the direction of dependency. Instead of service A
*calling* service B ("do this"), A *announces a fact* ("this happened") and whoever cares
reacts. The producer doesn't know or care who's listening. This trades away the legible,
linear control flow of request/response in exchange for powerful **decoupling** and
**scale** — and it's one of the sharpest trade-offs in this whole track.

```
   REQUEST/RESPONSE (coupled)        EVENT-DRIVEN (decoupled)
   A ──"charge this"──▶ Payment      A ──emits──▶ [ OrderPlaced ]
   A ──"reserve"──────▶ Inventory                     │ (broker)
   A ──"email"────────▶ Notify         ┌──────────────┼──────────────┐
   A knows all 3, waits for all 3,     ▼              ▼              ▼
   fails if any is down NOW.        Payment       Inventory       Notify
                                    (reacts)      (reacts)        (reacts)
   Add a 4th step → change A.       A knows NONE of them. Add a 5th
                                    consumer → A doesn't change at all.
```

Two properties define the style. **Temporal decoupling:** because a broker buffers the
event, the producer and consumer need not be up at the same instant — Inventory can be
down for maintenance and still process the `OrderPlaced` events when it comes back.
**Extensibility:** adding a new reaction (a fraud-check service that also listens for
`OrderPlaced`) requires *zero* change to the producer — you just add a subscriber. These
are real, valuable properties. The bill: you can no longer *read* the system's behavior
as a linear flow, and you inherit eventual consistency and a swarm of new failure modes.

## Going Deeper

**Events vs commands vs messages.** Precision matters. A **command** is a request for
something to happen (`PlaceOrder`) — it's directed at one specific handler, expresses
intent, and can be rejected/validated. An **event** is a notification that something
*already happened* (`OrderPlaced`) — it's a fact in the past tense, directed at no one in
particular, and can't be "rejected" (it happened). This isn't pedantry: commands couple
you to a handler and a workflow; events decouple you from consumers entirely. Naming your
messages as past-tense facts vs imperative requests keeps you honest about which coupling
you're choosing.

**Three flavors of "event," with very different coupling:**
- **Event notification** — a thin "something happened, id=123" ping; the consumer must call
  back to get details. Lightest coupling, but chatty (consumers call back to the source).
- **Event-carried state transfer** — the event carries the data the consumer needs
  (`OrderPlaced` includes the order details), so consumers don't call back. Reduces
  coupling and load, at the cost of duplicated data and larger events.
- **Event sourcing** — the events *are* the source of truth; state is derived by replaying
  them (Lesson 20). The most powerful and the most complex.

Choosing among these is a real design decision, not an incidental one.

**Choreography vs orchestration.** Two ways to run a multi-step process across services:
- **Choreography** — no central coordinator; each service reacts to events and emits its
  own (Order emits `OrderPlaced` → Payment reacts and emits `PaymentTaken` → Shipping
  reacts…). Maximally decoupled and extensible, but *no single place shows the whole
  workflow* — the business process is emergent, smeared across services, and hard to
  follow or change.
- **Orchestration** — a central coordinator (an orchestrator/saga manager) explicitly
  directs the steps ("now charge, now reserve, now ship"). The workflow is legible and in
  one place, at the cost of reintroducing a coordinator that everything depends on.

Neither is "right." Choreography for loose, extensible flows; orchestration when the
workflow is complex enough that you need to *see* and control it (Lesson 17's sagas make
this concrete). The mistake is doing complex business processes as pure choreography and
then being unable to understand or debug the flow.

{: .warning }
> **The sharp edges you're signing up for**
> EDA's decoupling is paid for in ways juniors underestimate: (1) <strong>Eventual
> consistency</strong> — reactions happen "soon," not "now," so there's a window where the
> system is inconsistent (the order exists but the email hasn't sent, stock isn't yet
> decremented). The business must tolerate this (Lesson 15). (2) <strong>No linear flow to
> read</strong> — you can't follow the logic top-to-bottom; behavior is emergent across
> subscribers, which makes reasoning and onboarding harder. (3) <strong>Distributed
> debugging</strong> — a problem spans producers, the broker, and consumers with no single
> stack trace; you <em>must</em> have correlation IDs and tracing (Lesson 25). (4)
> <strong>Error handling is now your job</strong> — what happens when a consumer fails to
> process an event? You need retries, idempotency (Lesson 17), and dead-letter queues so
> messages aren't silently lost. (5) <strong>Ordering & duplicates</strong> — most brokers
> deliver at-least-once and don't globally order, so consumers must handle duplicate and
> out-of-order events. None of these exist in a synchronous monolith; all of them are the
> price of the decoupling.

**When EDA fits — and when it hides the system from you.** It fits when: you need loose
coupling and independent evolution, when reactions are genuinely asynchronous/optional
(notifications, analytics, downstream projections), when you need to fan out one event to
many consumers, or when temporal decoupling and buffering (load leveling) are valuable.
It's a poor fit when: the flow is fundamentally a synchronous request the user is waiting
on and needs an immediate answer, or when the process is a tightly-coupled transaction that
really wants ACID. And the meta-warning: EDA can make a system *harder to understand* — the
same decoupling that adds flexibility removes the ability to read the flow, so applied
everywhere by reflex it turns a comprehensible system into an inscrutable web of events.
Use it where the decoupling buys something; don't event-ify a simple synchronous flow just
because events feel modern.

---

## Lab — Design Exercise

**The situation:** Here's a synchronous checkout flow, all in one request the user waits on:

```
placeOrder(cart):
   order = createOrder(cart)          # write order
   charge = paymentGateway.charge(..) # call Stripe, wait
   inventory.reserve(order.items)     # call Inventory, wait
   emailService.sendConfirmation(..)  # call email, wait
   return order                       # user has waited for ALL of it
```

**Redesign it as event-driven** where appropriate. Decide which steps should stay
synchronous (the user needs an immediate answer) and which should become event-driven
reactions, choose choreography or orchestration for the async part and justify it, and —
most importantly — **enumerate the new failure modes and consistency issues you just signed
up for**, with the mitigation each one needs. Note what you'd tell the product owner about
the user-visible behavior change.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is applying EDA selectively and being fully honest about the costs. A strong
answer:
<br><br>
<strong>What stays synchronous vs becomes an event.</strong> The user is waiting, and some
answers must be immediate: <em>creating the order</em> and <em>taking payment</em> should
stay synchronous — the user needs to know <em>now</em> whether their card was accepted and
the order was placed (you can't tell them "your payment may or may not have gone through,
check back later"). So keep: create order + charge, synchronously, and return success to the
user once those two succeed. Then <em>emit an <code>OrderPlaced</code> (and
<code>PaymentTaken</code>) event</em> and make the rest — <em>inventory reservation</em> and
<em>the confirmation email</em> — event-driven reactions that happen asynchronously after the
user has gotten their response. (An even better design may make inventory reservation part of
the synchronous path if overselling is unacceptable — see consistency below; this is a real
business decision, not a technical default.)
<br><br>
<strong>Choreography or orchestration?</strong> For this modest flow (two async reactions,
email and inventory, that don't depend on each other), <em>choreography</em> is fine and
simplest: Payment/Order emits <code>OrderPlaced</code>; Inventory and Notification each
subscribe and react independently. If the flow grew more complex — steps that depend on each
other, compensations on failure (refund if we can't fulfill) — you'd switch to
<em>orchestration</em> (a saga/coordinator, Lesson 17) so the multi-step process is legible
and controllable in one place rather than emergent and un-followable.
<br><br>
<strong>The new failure modes and consistency issues (the crux):</strong>
<ul>
<li><strong>Eventual consistency.</strong> After the user sees "order placed," the stock
isn't yet decremented and the email hasn't sent — there's a window where the system is
inconsistent. If two users check out the last item simultaneously, you may
<em>oversell</em>, because the reservation is now async. <em>Mitigation / decision:</em> is
overselling tolerable (backorder, apologize) or not (then reservation must be synchronous, or
you need a synchronous stock check with async confirmation)? This is a business call, and the
whole point is that going async <em>forced</em> it into the open.</li>
<li><strong>Lost or failed reactions.</strong> What if Inventory or the email consumer
crashes while processing the event? In the old synchronous code it threw an error inline; now
the event could be lost. <em>Mitigation:</em> a durable broker with at-least-once delivery,
consumer <strong>retries</strong>, and a <strong>dead-letter queue</strong> so a repeatedly-
failing event is parked for investigation instead of silently dropped.</li>
<li><strong>Duplicate delivery.</strong> At-least-once delivery means a consumer may get the
same <code>OrderPlaced</code> twice (e.g., a retry after a timeout) — naively, it charges twice
or sends two emails. <em>Mitigation:</em> consumers must be <strong>idempotent</strong> (Lesson
17) — dedupe on the order id / an idempotency key so processing an event twice has the same
effect as once.</li>
<li><strong>Debuggability.</strong> "Why didn't customer X get their email?" now spans the
producer, the broker, and the consumer with no single stack trace. <em>Mitigation:</em> a
<strong>correlation/trace id</strong> threaded from the original request through the event and
into every consumer, plus distributed tracing and centralized logging (Lesson 25) — non-
optional here.</li>
<li><strong>Ordering.</strong> If two events for the same order arrive out of order,
consumers must cope. <em>Mitigation:</em> partition/key by order id where ordering matters,
and design consumers to tolerate reordering.</li>
</ul>
<strong>What to tell the product owner.</strong> "We can make checkout faster and more
resilient by doing payment synchronously (so the customer still gets an instant yes/no on
their card) and moving the confirmation email — and possibly the stock reservation — to happen
just after. Two things to decide: (1) The confirmation email will arrive a moment later, not
in the exact instant of checkout — fine for most, but worth knowing. (2) Bigger: if we make
stock reservation asynchronous, there's a small window where we could accept an order for an
item that just sold out. Do we want to prevent that (keep reservation synchronous, slightly
slower checkout) or tolerate it (faster, but we occasionally backorder/apologize)? That's a
business trade-off, not a technical one — I need your call." That last paragraph is the mark of
the architect: the async redesign didn't just change tech, it surfaced a
<em>business</em> consistency decision (oversell vs latency) that was hidden inside the old
synchronous code, and the right move is to name it and hand it to the owner rather than
silently choosing.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Event-driven architecture (styles, flavors) | Martin Fowler — <https://martinfowler.com/articles/201701-event-driven.html> |
| Events, commands, event notification vs state transfer | Fowler — "What do you mean by 'Event-Driven'?" (as above) |
| Choreography vs orchestration | Sam Newman, *Building Microservices* (2e), Ch. 4 |
| EDA trade-offs & topologies | *Fundamentals of Software Architecture*, Richards & Ford (Ch. 14) |
| Dead-letter queues, at-least-once delivery | <https://en.wikipedia.org/wiki/Dead_letter_queue> |

---

## Checkpoint

**Q1.** Distinguish an *event* from a *command*, and explain how that difference maps onto
the coupling you're choosing. Why does emitting `OrderPlaced` decouple the producer in a way
that calling `chargePayment()` does not?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
A <strong>command</strong> is a request for something <em>to happen</em>, phrased as an
imperative and directed at a specific handler (<code>PlaceOrder</code>,
<code>ChargePayment</code>): it expresses intent, expects to be carried out, and can be
validated or rejected. An <strong>event</strong> is a notification that something
<em>already happened</em>, phrased as a past-tense fact and directed at no one in particular
(<code>OrderPlaced</code>, <code>PaymentTaken</code>): it's a statement about the past that
can't be "rejected" (it happened), and it doesn't say who should do anything about it.
<br><br>
This maps directly onto coupling. A command <strong>couples the sender to a specific
recipient and an intended outcome</strong> — <code>chargePayment()</code> means the producer
knows there's a payment step, knows (something about) who handles it, is directing that it
happen, and typically waits for and depends on the result. Add a step to the workflow and you
must change the sender to issue another command. An event <strong>decouples the producer from
consumers entirely</strong> — emitting <code>OrderPlaced</code> just announces a fact; the
producer neither knows nor cares who (if anyone) reacts. Payment, inventory, email, fraud-
check, analytics can all subscribe, and the producer is unaware of every one of them.
<br><br>
So <code>OrderPlaced</code> decouples in a way <code>chargePayment()</code> can't because of
the <em>direction of knowledge</em>: with the command, the producer must know about the
consumer and the intended action (outbound dependency on Payment); with the event, the
knowledge flows the other way — the <em>consumers</em> know about the event and choose to
listen, while the producer knows nothing about them. That's why you can add a fifth reaction
(a new subscriber to <code>OrderPlaced</code>) with <em>zero change to the producer</em>,
whereas adding a fifth command means editing the sender. Naming messages honestly (past-tense
facts vs imperative requests) keeps you clear about which coupling you're actually choosing —
and it's why "event-driven" flows are extensible in a way "call the next service" flows are
not.
</details>

**Q2.** Event-driven architecture buys decoupling and scale but imposes real costs. Name
three specific costs that don't exist in a synchronous monolith, and the mitigation each
requires. Why is "just use events everywhere" a mistake?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
Three costs that a synchronous monolith simply doesn't have:
<ul>
<li><strong>Eventual consistency.</strong> Reactions happen "soon," not "now," so there's a
window where the system is inconsistent (the order exists but stock isn't decremented, the
email isn't sent). In a synchronous flow those all completed before returning.
<em>Mitigation:</em> design the business to tolerate the window (and decide explicitly where
it can't — Lesson 15), and make the inconsistency converge reliably.</li>
<li><strong>Duplicate / out-of-order delivery.</strong> Brokers typically deliver
at-least-once and don't globally order, so a consumer may get the same event twice or events
out of sequence — which naively means double-charging or corrupt state. A synchronous call
happens exactly once, in order. <em>Mitigation:</em> make consumers <strong>idempotent</strong>
(dedupe on an id/idempotency key so processing twice equals processing once — Lesson 17), and
partition by key where ordering matters.</li>
<li><strong>Distributed, un-followable debugging.</strong> A problem spans producer, broker,
and consumers with no single stack trace and no linear flow to read — behavior is emergent.
The monolith gave you one call stack and top-to-bottom code. <em>Mitigation:</em>
<strong>correlation/trace ids</strong> threaded through every event and distributed tracing +
centralized logging (Lesson 25), plus, for complex flows, orchestration so the process is
visible in one place; and dead-letter queues so failed events are captured, not lost.</li>
</ul>
(Also valid: lost messages needing durable brokers + retries + DLQs; the loss of a readable
linear control flow harming comprehension and onboarding.)
<br><br>
Why "use events everywhere" is a mistake: EDA's decoupling is bought precisely by
<em>removing the readable, synchronous flow</em> — so applied indiscriminately it converts a
comprehensible system into an inscrutable web of events, and it imposes eventual consistency,
idempotency requirements, and distributed-debugging costs on flows that never needed them. A
simple synchronous request the user is waiting on for an immediate answer (place order, get
yes/no) is <em>harmed</em> by being event-ified: you add latency uncertainty, consistency
windows, and debugging difficulty to buy a decoupling the flow doesn't benefit from. The
discipline (Lesson 2) is to use events <em>where the decoupling, fan-out, temporal buffering,
or extensibility buys something real</em> — asynchronous, optional, or multi-consumer
reactions — and keep synchronous request/response where the user needs an immediate answer or
the operation genuinely wants a transaction. Events are a powerful tool with a high cost, not
a default; the architect applies them selectively, not religiously.
</details>

---

## Homework

Find one flow in your system (or a well-known system) that is currently synchronous and
consider whether part of it *should* be event-driven — and one that is currently event-driven
and ask whether the decoupling is actually buying anything or just making the system harder to
follow. For the synchronous-to-event candidate, list the specific new failure modes and
consistency issues you'd take on and the mitigations. For the event-driven-to-question
candidate, decide honestly whether it's decoupling that earns its cost or events-for-events'-
sake that has made a simple flow inscrutable. The goal is to feel the trade-off in both
directions.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise builds the selective judgment the lesson argues for by forcing you to look
<em>both</em> ways. A strong response:
<br><br>
<strong>For the synchronous → event candidate:</strong> a good pick is an
<em>asynchronous-by-nature</em> reaction currently done inline — sending emails/notifications,
updating analytics, syncing a downstream system, generating a report — where making it
event-driven would decouple it, allow retries/buffering, and let the user's request return
faster without waiting on it. The valuable part is the honest cost list: eventual consistency
(and where it bites), the need for idempotent consumers (at-least-once delivery), retries + a
dead-letter queue so failures aren't lost, and correlation ids/tracing for debugging. A
strong answer also notices any part that <em>must stay synchronous</em> (the user needs an
immediate answer, or it needs a transaction) — showing the selective, not wholesale,
application.
<br><br>
<strong>For the event-driven → question candidate:</strong> this is the more educational
direction, because it resists the reflex that events are always good. The honest finding is
sometimes "yes, this earns its keep" (genuine fan-out to multiple consumers, real temporal
decoupling, independent evolution) and sometimes "no" — a simple flow that was turned into
events for fashion, so now a straightforward sequence is smeared across producers, a broker,
and consumers, impossible to read top-to-bottom, harder to debug, and carrying eventual-
consistency and duplicate-handling costs for a decoupling nobody needed. Recognizing an
<em>over-eventified</em> flow — and being willing to say it might be simpler and better as a
synchronous call — is exactly the maturity the lesson is teaching: events are a high-cost tool
justified by specific benefits (decoupling, fan-out, buffering, extensibility), and applying
them where those benefits are absent trades comprehensibility for nothing. Feeling the
trade-off in both directions — where sync should become async, and where async should arguably
be sync — is the point, and it's what separates "use the right tool for the flow" from "events
are modern, use them everywhere."
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 13 — Service Granularity & Decomposition →](lesson-13-decomposition){: .btn .btn-primary }
