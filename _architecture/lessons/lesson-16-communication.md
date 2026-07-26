---
title: "Lesson 16 — Communication Styles (Sync, Async, REST, gRPC, Messaging)"
nav_order: 3
parent: "Phase 4: Distributed Systems"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 16: Communication Styles (Sync, Async, REST, gRPC, Messaging)

{: .note }
> **Words to know**
> - **synchronous** — the caller sends a request and *waits* for the response before continuing.
> - **asynchronous** — the caller sends a message and continues; the response (if any) comes later, or not at all.
> - **temporal coupling** — the callee must be up *at the same moment* as the caller; sync calls have it, async messaging removes it.
> - **REST** — resource-oriented HTTP APIs; ubiquitous, human-readable, loosely typed.
> - **gRPC** — a high-performance, strongly-typed RPC framework over HTTP/2 with schemas (protobuf).
> - **message queue vs pub/sub** — a queue delivers each message to one consumer; pub/sub broadcasts to many.
> - **broker** — the middleware (Kafka, RabbitMQ, SQS/SNS) that carries and buffers asynchronous messages.

## Concept

Once you have multiple services, *how* they talk is one of your biggest coupling decisions.
The first fork is **synchronous vs asynchronous**, and the difference isn't just style — it
determines **temporal coupling**: whether the callee must be alive at the exact moment the
caller needs it.

```
   SYNCHRONOUS (request/response)         ASYNCHRONOUS (messaging)
   A ──request──▶ B                       A ──message──▶ [ broker ] ──▶ B
   A  ⏳ waits...                          A continues immediately.
   A ◀─response── B                        B processes when it can (even later).
   ✓ simple, immediate answer             ✓ temporal decoupling (B can be down now)
   ✓ easy to reason about, one flow       ✓ buffering / load leveling, fan-out
   ✗ TEMPORAL COUPLING: B must be UP now  ✗ eventual consistency, no direct answer
   ✗ A's latency = A + B + network        ✗ harder to trace; new failure modes
   ✗ B down or slow → A blocked/fails      (Lesson 12's costs)
```

**Synchronous** (REST, gRPC): the caller waits for an answer. Simple, gives an immediate
result, easy to follow — but it *couples the caller to the callee's availability and
latency*: if B is down or slow, A is blocked or fails, and A's response time is the sum of
its own plus B's plus the network. **Asynchronous** (messaging via a broker): the caller
fires a message and moves on. It removes temporal coupling (B can be down and catch up
later) and enables buffering and fan-out — at the cost of eventual consistency and the
absence of a direct answer (Lesson 12's trade-offs). The heuristic: **prefer async where
the interaction can tolerate it; keep sync where the caller genuinely needs an immediate
answer.**

## Going Deeper

**Synchronous protocol choices — REST vs gRPC vs GraphQL.** When you do go synchronous, the
protocol is a secondary trade-off:
- **REST/HTTP+JSON** — the ubiquitous default. Human-readable, works everywhere, huge
  tooling, loosely typed, cacheable via HTTP. *For:* public APIs, broad compatibility,
  simplicity. *Against:* verbose, weaker contracts, over/under-fetching, slower to
  serialize than binary.
- **gRPC** — binary (protobuf), strongly-typed via a schema, HTTP/2 multiplexing, streaming,
  code-gen'd clients. *For:* high-performance internal service-to-service calls, strict
  contracts, polyglot with generated stubs. *Against:* not human-readable, harder through
  browsers/proxies, more setup — usually internal, not public.
- **GraphQL** — the client specifies exactly the fields it wants in one query. *For:* rich
  clients (mobile/web) that would otherwise over-fetch or make many round-trips; a flexible
  aggregation layer. *Against:* server complexity, caching is harder, easy to write
  expensive queries; it's a query layer, not usually service-to-service plumbing.

The common pattern: REST or GraphQL at the *edge* (public/client-facing), gRPC *between*
internal services where performance and strict contracts matter.

**Asynchronous topologies — queue vs pub/sub vs stream.** Async isn't one thing:
- **Point-to-point queue** — each message is delivered to *one* consumer (competing
  consumers pull work off the queue). *For:* distributing work / commands (a job to be done
  once). E.g., SQS, RabbitMQ queues.
- **Publish/subscribe** — each message is broadcast to *every* subscriber. *For:* events
  where many independent parties react (Lesson 12's fan-out). E.g., SNS, RabbitMQ topics.
- **Event stream / log** — an ordered, retained log consumers read at their own position and
  can replay. *For:* event sourcing, analytics, high-throughput pipelines, reprocessing.
  E.g., Kafka. The retention + replay is the distinguishing feature.

Choosing among these is a real design decision: "a job done once" (queue) is not "a fact many
react to" (pub/sub) is not "a replayable ordered history" (stream).

{: .note }
> **Orchestration vs choreography — at the wire level**
> The same choice from Lesson 12 reappears concretely. A multi-step cross-service process
> can be <strong>orchestrated</strong> (a central coordinator makes synchronous or
> asynchronous calls in a defined sequence — the workflow is explicit and in one place) or
> <strong>choreographed</strong> (each service reacts to events and emits its own — maximally
> decoupled, but the workflow is emergent and spread across services). Synchronous
> orchestration is the easiest to <em>read</em> but has the most temporal coupling (every
> step must be up); asynchronous choreography is the most decoupled and resilient but the
> hardest to follow and debug. Match it to the complexity: simple/legible flows can be
> synchronous or lightly choreographed; complex flows with compensations usually want
> explicit orchestration (a saga orchestrator, Lesson 17) so the process is visible.

**"Prefer async" — the heuristic and its limits.** Async communication generally makes
systems more resilient and scalable (temporal decoupling means a downstream outage doesn't
immediately fail the caller; buffering absorbs spikes; fan-out is free). So a good default
bias is: *use async messaging where the interaction can tolerate a delayed/eventual result.*
But the limit is real: when the caller genuinely needs an immediate answer to proceed — a
user waiting to know if their payment succeeded, a request that must return data *now* —
async is the wrong tool (you'd be faking a request/response over a message bus, adding
complexity for nothing). The skill is recognizing which interactions are *fundamentally
synchronous* (someone is waiting on the answer) versus *fundamentally asynchronous* (a
reaction that can happen soon), and not forcing either into the other's shape.

**The availability math nobody mentions.** A synchronous chain multiplies unavailability.
If A calls B calls C synchronously and each is 99.9% available, the chain is 0.999³ ≈ 99.7%
— *less* available than any single service, because any link failing fails the whole request.
This is a hidden cost of synchronous inter-service calls (and an argument for async, or for
fewer hops, or for resilience patterns — Lesson 18). Chaining synchronous calls deeply is one
of the quiet ways distributed systems become *less* reliable than the monolith they replaced.

---

## Lab — Design Exercise

**The situation:** You're designing the inter-service communication for an online-ordering
system. For each of these four interactions, choose **synchronous REST**, **synchronous
gRPC**, or **asynchronous messaging** (and if async, queue vs pub/sub vs stream), and justify
it by latency needs, the immediacy of the answer required, temporal coupling, and fan-out:

1. The **mobile app** fetches a customer's order history to display on screen.
2. At checkout, the **Order service** must verify the customer's **payment** will be accepted
   before telling the customer "order confirmed."
3. When an order is placed, **Inventory**, **Shipping**, **Notifications**, and **Analytics**
   all need to know and react.
4. Two internal services on a hot path — **Order** calls **Pricing** thousands of times per
   second to compute totals, needs the answer in single-digit milliseconds, strict contract.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is matching each interaction's needs (immediacy, latency, fan-out, coupling) to the
right style. A strong answer:
<br><br>
<strong>1. Mobile app → order history: synchronous REST (or GraphQL).</strong> The user is
looking at the screen and needs the data <em>now</em> — fundamentally synchronous
(request/response). REST over HTTP is the right default for a client-facing, public-ish edge
API: ubiquitous, works through mobile networks/proxies, human-debuggable, cacheable. GraphQL
is a reasonable alternative if the mobile client would otherwise over-fetch or make several
round-trips (let it request exactly the fields it needs in one query). Not gRPC (awkward from
mobile/browser, no human-readability benefit here); not async (there's no delayed-reaction
semantics — someone is waiting on the answer).
<br><br>
<strong>2. Checkout → verify payment before confirming: synchronous (REST or gRPC).</strong>
This is <em>fundamentally synchronous</em> — the customer is waiting to be told "confirmed,"
and you cannot confirm until you know the payment is accepted, so you must have the answer
<em>before</em> responding. Temporal coupling is unavoidable here (the payment path must be up
to take payment), and that's acceptable because there's no meaningful "confirm the order but
we'll check payment later" — the answer gates the user's next step. Wrap it in a timeout +
circuit breaker (Lesson 18) so a slow payment provider degrades gracefully, but the call
itself is synchronous. (Contrast: the <em>confirmation email</em> after this point is async —
see #3.)
<br><br>
<strong>3. Order placed → Inventory, Shipping, Notifications, Analytics react: asynchronous
messaging, pub/sub (publish an <code>OrderPlaced</code> event).</strong> Classic
<em>fan-out</em>: four independent parties need to know, none of them needs to answer the
caller, and the Order service shouldn't be coupled to all four (or blocked if one is down).
Publish one <code>OrderPlaced</code> event; each subscribes and reacts on its own. This gives
temporal decoupling (Notifications can be briefly down and catch up), extensibility (add a
fifth reactor — fraud check — with zero change to Order, Lesson 12), and resilience. Use
<strong>pub/sub</strong> because it's one fact many parties react to (not a single job for one
worker). Analytics specifically may prefer a retained <strong>event stream</strong> (Kafka)
if it wants to replay/reprocess history — a reasonable refinement (stream for analytics,
pub/sub for the operational reactors, or a stream serving both). Accept eventual consistency
here (the email/stock-decrement/shipping-label happen "soon" — fine for these reactions).
<br><br>
<strong>4. Order → Pricing, thousands/sec, single-digit ms, strict contract: synchronous
gRPC.</strong> The answer is needed <em>immediately and in-line</em> (you can't compute an
order total asynchronously — Order is waiting on the price), so it's synchronous. At this
volume and latency budget, the protocol matters: <strong>gRPC</strong> — binary protobuf
(fast serialization), HTTP/2 multiplexing (efficient at high call rates), a strict schema
(the strong contract you want between internal services), and generated clients. REST+JSON
would be slower to serialize and looser-typed at a rate where that overhead adds up. This is
the canonical "internal, high-performance, strict-contract service-to-service" case gRPC is
built for. (Also: watch the <em>chattiness</em> — thousands of calls/sec is a lot of temporal
coupling and availability multiplication; consider batching where possible, and definitely
put a timeout/circuit-breaker on it.)
<br><br>
<strong>The meta-lesson:</strong> the four interactions land on four different answers because
their needs differ — a client-facing "give me data now" (sync REST/GraphQL), a gating "I need
the answer to proceed" (sync, timeout-wrapped), a "many parties react to a fact" (async
pub/sub, eventual consistency accepted), and a "high-volume, low-latency, strict internal call"
(sync gRPC). The two axes that decided each: <em>does the caller need an immediate answer to
proceed?</em> (yes → sync; no → async) and, secondarily for sync, <em>edge/public vs
high-performance internal?</em> (REST/GraphQL vs gRPC), and for async, <em>one worker, many
reactors, or replayable history?</em> (queue vs pub/sub vs stream). Forcing one style
everywhere — all-sync (fragile, temporally coupled, availability-multiplying) or all-async
(faking request/response over a bus for interactions someone is waiting on) — is the mistake;
the architect matches the style to each interaction's real needs.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Sync vs async, temporal coupling | Sam Newman, *Building Microservices* (2e), Ch. 4 |
| REST vs gRPC vs GraphQL | <https://grpc.io/docs/what-is-grpc/introduction/> ; <https://graphql.org/learn/> |
| Messaging patterns (queue, pub/sub, streams) | *Enterprise Integration Patterns*, Hohpe & Woolf |
| Availability of synchronous chains | *Release It!*, Nygard (integration points) |

---

## Checkpoint

**Q1.** Explain "temporal coupling" and how synchronous and asynchronous communication differ
on it. Why does a deep chain of synchronous calls end up *less* available than a single
service?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Temporal coupling</strong> is the requirement that the callee be alive and responsive
<em>at the same moment</em> the caller needs it. It's a coupling in <em>time</em>: the two
parties must be up simultaneously for the interaction to work.
<br><br>
<strong>Synchronous</strong> communication (REST, gRPC) has strong temporal coupling: the
caller sends a request and blocks waiting for the response, so if the callee is down or slow
<em>right now</em>, the caller is blocked or fails <em>right now</em>. The interaction only
succeeds if both are up at the same instant. <strong>Asynchronous</strong> messaging removes
temporal coupling: the caller hands a message to a broker and continues immediately; the broker
buffers it, and the consumer processes it whenever it's able — even if it was down when the
message was sent and comes back later. The two never need to be up simultaneously — the broker
decouples them in time. (That's exactly why async is more resilient to downstream outages, and
why it enables buffering/load-leveling.)
<br><br>
Why a deep synchronous chain is <em>less</em> available than one service: availability
multiplies along a synchronous call chain, because <em>any</em> link failing fails the whole
request. If A calls B calls C synchronously and each service is independently 99.9% available,
the end-to-end availability is roughly 0.999 × 0.999 × 0.999 ≈ 0.997 = 99.7% — <em>worse</em>
than any single service's 99.9%, because the request now depends on all three being up at once
(temporal coupling across the whole chain). Add more synchronous hops and it degrades further.
So paradoxically, decomposing a monolith into a synchronous chain of services can make the
system <em>less</em> reliable than the monolith it replaced: the monolith's internal calls
couldn't "fail to arrive," but each synchronous network hop can, and their failure
probabilities compound. This is a major hidden cost of synchronous inter-service communication,
and it's why architects (a) minimize the depth of synchronous chains, (b) prefer async where
the interaction tolerates it (breaking the temporal coupling so a downstream outage doesn't
immediately fail the caller), and (c) apply resilience patterns — timeouts, circuit breakers,
fallbacks (Lesson 18) — to the synchronous calls that must remain, so one slow/down dependency
degrades gracefully instead of failing the whole chain.
</details>

**Q2.** Give the trade-offs that would lead you to choose REST vs gRPC for a synchronous call,
and queue vs pub/sub for an asynchronous one. Why is "one style everywhere" a mistake?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>REST vs gRPC (synchronous):</strong> choose <strong>REST</strong> (HTTP+JSON) when you
want ubiquity and simplicity — a public or client-facing API, broad compatibility (works from
browsers, mobile, through proxies), human-readability for debugging, and HTTP-level caching;
the cost is verbosity, weaker typing, and slower serialization. Choose <strong>gRPC</strong>
(binary protobuf over HTTP/2) when you want performance and strict contracts — high-volume,
low-latency <em>internal</em> service-to-service calls, where the binary serialization,
HTTP/2 multiplexing, strongly-typed schema, and generated polyglot clients pay off; the cost is
that it's not human-readable, awkward through browsers/some proxies, and needs more setup. The
usual pattern: REST (or GraphQL) at the edge, gRPC between internal services on hot paths.
<br><br>
<strong>Queue vs pub/sub (asynchronous):</strong> choose a <strong>point-to-point queue</strong>
when the message is a <em>job to be done once</em> by exactly one worker (competing consumers
pull work off it) — distributing tasks/commands. Choose <strong>pub/sub</strong> when the
message is a <em>fact that many independent parties react to</em> — each subscriber gets its own
copy and reacts independently (event fan-out). The distinction is "one consumer does this work"
vs "many consumers each react to this fact." (And a third option, an event <em>stream/log</em>
like Kafka, when you also need an ordered, retained history that consumers can replay.)
<br><br>
Why "one style everywhere" is a mistake: each interaction has different needs — immediacy,
latency, fan-out, tolerance for delay, contract strictness — and one style can't serve them
all well. <em>All-synchronous</em> imposes temporal coupling and availability-multiplication on
interactions that don't need an immediate answer (a reaction that could be async is now a
fragile blocking call that fails when a downstream is down), and forces high-fan-out "many
react to this" cases into awkward multiple direct calls. <em>All-asynchronous</em> forces
interactions where someone is genuinely waiting for an answer (fetch data for a screen, verify
payment before confirming) into a fake request/response over a message bus — adding eventual
consistency, complexity, and latency for a decoupling the interaction can't use. Similarly,
using gRPC for a public browser API, or REST+JSON for a thousands-per-second hot path, or a
queue where you needed fan-out, each mismatches the tool to the need. The architect's job is to
read each interaction's real requirements — <em>does the caller need an immediate answer?</em>
(sync vs async), <em>edge or high-performance internal?</em> (REST/GraphQL vs gRPC), <em>one
worker or many reactors or replayable log?</em> (queue vs pub/sub vs stream) — and choose per
interaction, accepting that a healthy distributed system deliberately uses several
communication styles, each where it fits.
</details>

---

## Homework

Map the communication styles in your system: for each significant inter-service (or
service-to-external) interaction, note whether it's sync or async and the protocol/topology,
then judge whether that matches the interaction's real needs. Look for two things: (1) a
synchronous call (or a deep synchronous chain) that's causing availability or latency problems
and could be async, or needs resilience patterns; and (2) an async interaction that's really a
disguised request/response (someone is waiting on the answer) that would be simpler as a sync
call. If you have deep synchronous chains, estimate the compounded availability and decide
whether it's acceptable.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise makes the sync/async and protocol/topology choices visible on a real system. A
strong response:
<br><br>
<strong>Finds a fragile synchronous chain.</strong> The common, high-value finding is a deep
synchronous call chain on a critical path where the compounded availability is worse than any
single service (0.999ⁿ) and/or the latency is the sum of all hops — often with missing
timeouts/circuit breakers, so one slow dependency can hang the whole chain. Good answers either
(a) identify hops that could become <em>async</em> to break the temporal coupling (a reaction
that doesn't need to block the response), reducing the chain's fragility, or (b) where the chain
must stay synchronous, prescribe the resilience patterns (timeouts, circuit breakers,
fallbacks — Lesson 18) and consider reducing the number of hops. Estimating the compounded
availability and asking "is 99.x% acceptable for this path?" is exactly the reliability
reasoning the lesson wants.
<br><br>
<strong>Finds a mismatched style.</strong> The two mismatches to look for are symmetric: an
async interaction that's really a disguised request/response (built on a message bus but with
someone waiting synchronously for the answer — added complexity and eventual consistency for a
decoupling that isn't used; simpler as a direct sync call), and a sync interaction that should
be async (a fan-out done as several blocking direct calls, or a "fire and don't really need the
answer now" reaction done synchronously — coupling the caller to downstreams it needn't wait
for). Also worth flagging: protocol/topology mismatches — REST+JSON on a hot internal path that
wants gRPC, or a queue used where fan-out (pub/sub) was actually needed.
<br><br>
The takeaway a good answer reaches: a healthy system uses <em>several</em> communication styles
deliberately (sync where an answer is awaited, async where a reaction can be deferred; the right
protocol/topology per interaction), and the failures come from applying one style by reflex —
which shows up as either fragile over-synchronous chains or over-complicated async flows. Naming
one concrete instance of each to fix, and (for the sync chains that must stay) the resilience
patterns to add, turns the audit into an action list.
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 17 — Distributed Data (Transactions, Sagas, Outbox, Idempotency) →](lesson-17-distributed-data){: .btn .btn-primary }
