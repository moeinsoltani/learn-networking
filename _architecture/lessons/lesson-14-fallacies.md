---
title: "Lesson 14 — The Fallacies of Distributed Computing"
nav_order: 1
parent: "Phase 4: Distributed Systems"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 14: The Fallacies of Distributed Computing

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

The moment your system spans more than one machine over a network, a different physics
applies — and the classic way developers get burned is by carrying single-machine
intuitions into it. In 1994–97, engineers at Sun (L. Peter Deutsch and James Gosling)
catalogued the **eight fallacies of distributed computing**: assumptions that are false,
that everyone makes anyway, and that each map to a real production outage.

The eight fallacies of distributed computing are each stated as an assumption
people make, and **every one of them is false**.

| The fallacy | The reality |
|---|---|
| 1. The network is reliable | Calls fail. Plan for it |
| 2. Latency is zero | A remote call is on the order of a thousand times slower than a local one |
| 3. Bandwidth is infinite | Large payloads and chatty calls saturate it |
| 4. The network is secure | Assume it is hostile: encrypt and authenticate |
| 5. Topology doesn't change | Nodes and routes move; never hardcode them |
| 6. There is one administrator | Many owners, many policies, no single view |
| 7. Transport cost is zero | Serialisation, CPU, and actual money are all real |
| 8. The network is homogeneous | Mixed protocols, versions, and hardware |

Underneath all eight is a single mental shift, and it is the one worth
installing permanently: **a remote call is nothing like a local call.** It looks
identical in your code — that is precisely the danger — and it can be slow,
fail, partially succeed, or arrive twice.

Here is code that assumes every one of the eight:

The single most expensive lie in our field is *"it's just a function call over the
network."* A local function call is fast (nanoseconds), reliable (it doesn't "fail to
arrive"), free (no serialization or **bandwidth**[°](#w-bandwidth)), and secure (no wire to sniff). A remote
call is none of those. Every **fallacy**[°](#w-fallacy) is a place where that lie bites.

## Going Deeper

Each fallacy, with the failure it causes and the mitigation it demands:

- **1. The network is reliable.** Calls *will* fail — packets drop, connections reset,
  services are momentarily unreachable. Code that assumes success (no error handling on a
  **remote call**[°](#w-remote-call)) corrupts state or hangs on the first blip. *Mitigation:* explicit failure
  handling, timeouts, retries with backoff, circuit breakers (Lesson 18).
- **2. Latency is zero.** A local call is ~nanoseconds; a same-datacenter remote call is
  ~milliseconds (thousands of times slower); cross-region is tens to hundreds of ms. The
  killer is *chattiness* — a loop that makes 100 remote calls where a monolith made 100
  in-process calls is now unusably slow. *Mitigation:* batch, coarse-grained interfaces,
  avoid N+1 remote calls, put a **latency**[°](#w-latency) budget on the path (Lesson 23).
- **3. Bandwidth is infinite.** Big payloads and high call volumes saturate links and
  cost money. Serializing a huge object graph on every call, or fanning out to thousands
  of consumers, hits real ceilings. *Mitigation:* send only what's needed, paginate,
  compress, watch payload sizes.
- **4. The network is secure.** Anything on the wire can be intercepted or tampered with;
  "internal" networks are breached routinely. *Mitigation:* encrypt in transit (TLS),
  authenticate every hop, assume zero trust (Lesson 24) — the [Security track]({{ '/security/learning-plan.html' | relative_url }})
  is this fallacy in depth.
- **5. Topology doesn't change.** Nodes are added/removed, IPs change, routes shift,
  autoscaling churns instances. Hardcoding hosts/IPs or assuming a fixed layout breaks on
  the first deploy or scale event. *Mitigation:* service discovery, DNS, load balancers,
  config not constants.
- **6. There is one administrator.** In reality many teams/orgs own different pieces, with
  different policies, versions, and change schedules, and no one has a global view or the
  authority to coordinate a change. *Mitigation:* explicit contracts and versioning
  (Lesson 26), assume you can't force upstreams to change in lockstep.
- **7. Transport cost is zero.** Every remote call has real costs: CPU to
  serialize/deserialize, bandwidth charges, cross-AZ/region data-transfer fees (a genuine
  line on the cloud bill). *Mitigation:* factor serialization and data-transfer cost into
  design (it's why chatty designs are expensive in money, not just latency).
- **8. The network is homogeneous.** Systems mix protocols, message formats, library
  versions, and hardware. Assuming everyone speaks exactly your format/version breaks at
  integration boundaries. *Mitigation:* standard, well-versioned interchange formats;
  tolerant readers (Lesson 26).

{: .warning }
> **The meta-lesson: a remote call is not a local call in disguise.**
> Many frameworks (RPC, some ORMs, "transparent" remoting) try to make a network call
> <em>look</em> like a local method call — same syntax, hidden wire. This is a trap: it
> encourages you to treat something slow, unreliable, insecure, and expensive as if it
> were fast, reliable, free, and safe. The result is code that's chatty (fallacy 2/3),
> assumes success (fallacy 1), and ignores security (fallacy 4) — precisely because the
> abstraction <em>hid</em> that a network was involved. The architect's stance:
> <em>make remote calls visible and treat them with respect</em> — coarse-grained,
> failure-handled, timed, secured — never pretend they're local.

**Why this is the foundation of Phase 4.** Every later distributed-systems topic is a
response to one or more fallacies. CAP and consistency models (Lesson 15) exist because
the network isn't reliable (**partitions**[°](#w-partition) happen). Sagas and idempotency (Lesson 17) exist
because calls fail and retry. Resilience patterns (Lesson 18) are the systematic answer to
fallacy 1. Communication choices (Lesson 16) trade against latency and reliability. If you
internalize the fallacies, the rest of the phase is "here's specifically how we cope with
each false assumption."

---

## Lab — Design Exercise

**The situation:** Here's a naive design for an order-checkout flow in a newly-distributed
system. Someone extracted services from a monolith and kept the code shape identical:

```
placeOrder(cart):
   for item in cart.items:                 # 40 items
       price = pricingService.getPrice(item)   # remote call, in a loop
   customer = customerService.get(cart.userId) # remote call
   payment = paymentService.charge(customer.card, total)  # remote call, no timeout
   inventoryService.reserve(cart.items)    # remote call, no error handling
   # (host addresses for each service are hardcoded constants)
   # (card + customer data sent in plaintext over the internal network)
   return "ok"
```

**Annotate every place this design falls for a fallacy.** For each, name the specific
fallacy, the production failure it will cause, and the mitigation. Then describe what the
*corrected* design looks like at a high level.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is seeing the network the code is pretending isn't there. A strong annotation:
<br><br>
<ul>
<li><strong>The pricing loop (40 remote calls)</strong> → <em>Fallacy 2 (latency is zero)
and 3 (bandwidth infinite), and 7 (**transport cost**[°](#w-transport-cost))</em>. In the monolith these were 40
nanosecond in-process calls; now they're 40 network round-trips per checkout — at even 5 ms
each that's 200 ms of pure latency for one step, and it scales with cart size and traffic
(and every call costs CPU/serialization and possibly cross-AZ data-transfer money).
<em>Failure:</em> checkout is slow and gets slower under load; a "chatty" hot path. <em>
Mitigation:</em> a coarse-grained batch call — <code>pricingService.getPrices(allItems)</code>
— one round-trip instead of 40; avoid N+1 remote calls.</li>
<li><strong><code>paymentService.charge(...)</code> with no timeout</strong> → <em>Fallacy 1
(network is reliable) and 2 (latency zero)</em>. <em>Failure:</em> if payment is slow or
hung, this call blocks indefinitely, exhausting threads/connections and cascading a
checkout-wide outage (the classic slow-dependency-takes-everything-down, Lesson 18).
<em>Mitigation:</em> a timeout on every remote call, plus a circuit breaker so a struggling
payment service is stopped from being hammered.</li>
<li><strong><code>inventoryService.reserve(...)</code> with no error handling</strong> →
<em>Fallacy 1 (network is reliable)</em>. <em>Failure:</em> the call can fail or the service
be momentarily unreachable; with no handling, the order is left in a broken half-completed
state (charged but not reserved) or the exception propagates raw. <em>Mitigation:</em>
explicit failure handling — and because payment already succeeded, a
<strong>compensating action / saga</strong> (Lesson 17) to refund or retry, since you can no
longer wrap charge+reserve in one transaction.</li>
<li><strong>Hardcoded host addresses</strong> → <em>Fallacy 5 (**topology**[°](#w-topology) doesn't change)</em>.
<em>Failure:</em> the first time a service is redeployed, scaled, or moved (new IPs,
autoscaling), the hardcoded hosts break. <em>Mitigation:</em> service discovery / DNS / a
load balancer, addresses from config not constants.</li>
<li><strong>Card + customer data in plaintext over the "internal" network</strong> →
<em>Fallacy 4 (network is secure)</em>. <em>Failure:</em> anyone who breaches the internal
network (routine) can sniff card data — a breach and a compliance violation. <em>
Mitigation:</em> TLS on every hop, authenticate services, treat the internal network as
hostile (zero trust, Lesson 24), and don't pass raw card data around at all (tokenize).</li>
<li><strong>The whole "keep the code shape identical" approach</strong> → the
<em>meta-fallacy</em>: treating remote calls as if they were the local calls they replaced.
<em>Failure:</em> all of the above at once. <em>Mitigation:</em> redesign the interaction for
the network, don't transliterate the monolith.</li>
</ul>
<strong>The corrected design at a high level:</strong> (1) Replace chatty loops with
<em>coarse-grained batch</em> calls (one <code>getPrices</code>, minimize round-trips). (2)
Put a <em>timeout</em> on every remote call and a <em>circuit breaker</em> on the risky
dependencies. (3) Add <em>explicit failure handling</em>, and since charge+reserve can no
longer be one ACID transaction across services, use a <em>saga with compensating actions</em>
(if reserve fails after charge, refund) and make consumers <em>idempotent</em> (Lesson 17).
(4) Resolve service addresses via <em>discovery/config</em>, never hardcode. (5)
<em>Encrypt and authenticate</em> every hop and stop passing raw card data (tokenize;
isolate payment — Lesson 13/24). The through-line: the corrected design <em>respects the
network</em> — it assumes calls are slow, failing, insecure, and costly, and is built
accordingly — instead of pretending the services are still one process. That respect,
applied everywhere a wire is crossed, is what the eight fallacies are teaching.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| The Fallacies of Distributed Computing | <https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing> |
| Deutsch's original list & explanations | Arnon Rotem-Gal-Oz, "Fallacies of Distributed Computing Explained" |
| Latency numbers every programmer should know | Jeff Dean's numbers — <https://gist.github.com/jboner/2841832> |
| Distributed systems fundamentals | *Designing Data-Intensive Applications*, Kleppmann (Ch. 8) |

---

## Checkpoint

**Q1.** State the meta-lesson behind all eight fallacies, and explain why "transparent"
remoting frameworks (that make a network call look like a local method call) are dangerous
precisely because of it.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The meta-lesson: <strong>a remote call is fundamentally not like a local call, and treating
it as one is the root error the eight fallacies all stem from.</strong> A local, in-process
call is fast (nanoseconds), reliable (it doesn't fail to arrive), free (no serialization,
bandwidth, or money), and secure (no wire to intercept). A remote call is the opposite on
every axis: slow (milliseconds — thousands of times slower, and it can be arbitrarily slow),
unreliable (it can fail, time out, or be lost), costly (CPU to serialize, bandwidth,
data-transfer charges), and insecure (traverses a network others can access). Every fallacy
is one facet of failing to respect that difference.
<br><br>
"Transparent" remoting frameworks — RPC systems, some ORMs' lazy-loading, anything that makes
<code>remoteService.doThing()</code> look syntactically identical to a local method call — are
dangerous <em>because</em> they hide exactly the thing you most need to see. By making the
network invisible, they invite you to write code as if the call were local: to put remote
calls in loops (chattiness — fallacies 2, 3, 7), to assume they succeed (no timeout or error
handling — fallacy 1), to ignore that data is crossing a wire (fallacy 4), and to forget it
costs money and time (fallacy 7). The abstraction is a <em>leaky and seductive</em> one: it
gives you the ergonomics of local calls while silently keeping the semantics of remote ones,
so developers reason with local intuitions about something that behaves nothing like local —
and the gap surfaces as chatty-and-slow hot paths, cascading timeouts, plaintext data on the
wire, and surprise cloud bills. The architect's stance is the reverse of transparency:
<em>make remote calls conspicuous and treat them with respect</em> — coarse-grained interfaces
to minimize round-trips, a timeout and failure-handling on every one, encryption and
authentication on every hop, and an awareness of the latency and money each costs. You want the
network to be <em>visible</em> in the design, not abstracted away, because the fallacies are all
consequences of forgetting it's there.
</details>

**Q2.** "Latency is zero" and "the network is reliable" are the two fallacies that bite
newly-distributed systems hardest. For each, give a concrete way that carrying monolith code
shape into a distributed system triggers the fallacy, and the mitigation.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>"Latency is zero" — triggered by chattiness.</strong> The classic way monolith code
triggers it: a loop or an N+1 pattern that made many <em>in-process</em> calls (nanoseconds
each, effectively free) is left unchanged when the callee becomes a separate service — so now
each iteration is a network round-trip (milliseconds). A loop pricing 40 cart items, or an
ORM lazily loading a child per row across a service boundary, turns "free" into hundreds of
milliseconds that grow with data size and load. The mitigation: design <em>coarse-grained</em>
interactions for the network — batch calls (fetch all 40 prices in one request), avoid N+1
remote calls, and put a <em>latency budget</em> on each request path so you notice when the
sum of hops blows it (Lesson 23). The rule of thumb: minimize the <em>number</em> of round
trips, because each one costs real, non-zero time.
<br><br>
<strong>"The network is reliable" — triggered by assuming success.</strong> Monolith code
rarely handles "the function didn't run at all," because an in-process call doesn't fail to
arrive. Transliterated to a distributed system, a remote call written with no timeout and no
error handling assumes the callee always responds — but the network drops packets, resets
connections, and makes services momentarily unreachable, and a hung dependency with no timeout
blocks the caller's threads until <em>it</em> falls over too (a cascading failure). The
mitigation: treat every remote call as able to fail or hang — put a <strong>timeout</strong>
on all of them (never wait forever), add <strong>retries with backoff and jitter</strong> for
transient failures (carefully — retries can storm, Lesson 2/18), wrap risky dependencies in a
<strong>circuit breaker</strong> so a downed service is stopped from being hammered, and,
because you've lost the cross-service transaction, use <strong>compensating actions/sagas</strong>
and <strong>idempotency</strong> (Lesson 17) so a failed or retried step leaves consistent
state. Both fallacies share the same cause — copying single-machine code shape onto a network —
and the same cure: redesign the interaction to respect that the network is slow and unreliable,
rather than pretending the services are still one process.
</details>

---

## Homework

Take a distributed interaction in your system (a service-to-service call path, or an
integration with an external API) and audit it against all eight fallacies. For each fallacy,
note whether the interaction currently assumes it's true (a latent bug) or handles it — and for
every "assumes true," describe the failure it would cause and the fix. Pay special attention to
chatty call patterns (fallacy 2), missing timeouts/retries (fallacy 1), hardcoded endpoints
(fallacy 5), and plaintext/unauthenticated hops (fallacy 4). Rank the gaps by how badly they'd
bite in production.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise turns the eight fallacies into a concrete audit checklist for real code. A strong
response:
<br><br>
<strong>Goes through all eight honestly</strong> and typically finds several latent
"assumes-true" gaps — the most common being: a chatty pattern (fallacy 2/3/7) on a hot path
(an N+1 or a loop of remote calls) that will slow down and cost money under load; at least one
remote call with <em>no timeout</em> (fallacy 1) that's a cascading-failure waiting to happen;
and often an "internal" hop that's unencrypted or unauthenticated (fallacy 4) because the
network was assumed safe. Naming the specific production failure each would cause — slow
checkout, thread-pool exhaustion taking down the service, a sniffable data path — is what makes
the audit real rather than theoretical.
<br><br>
<strong>Ranks by production impact</strong>, which is the architect's judgment. Usually the
missing timeout / no-circuit-breaker on a critical synchronous dependency ranks worst (it can
take the whole system down, not just degrade it), followed by security gaps (fallacy 4 —
breach/compliance risk), then chattiness (fallacy 2 — degrades under load), then topology/
hardcoding (fallacy 5 — breaks on the next deploy/scale). A good answer doesn't just list gaps
but prioritizes the fixes by blast radius, because you can't fix everything at once and the
cascading-failure and security gaps are the ones that turn into incidents.
<br><br>
The deeper payoff: doing this on a real interaction usually reveals that code which "works fine
in testing" is quietly assuming a reliable, fast, secure, unchanging network — and that the
fallacies aren't abstract history but a precise list of the ways that assumption will fail in
production. That realization is exactly what makes the rest of Phase 4 (consistency, sagas,
resilience) feel <em>necessary</em> rather than academic: each is the systematic answer to one
or more of the false assumptions this audit surfaced in your own system.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-fallacy"></a>**fallacy** — a false assumption people make without realizing it; here, eight false beliefs about networks.
- <a id="w-latency"></a>**latency** — the time for a message to travel there and back (delay), separate from bandwidth.
- <a id="w-bandwidth"></a>**bandwidth** — how much data you can push through per second (capacity).
- <a id="w-partition"></a>**partition** — a network split where some nodes can't reach others, though each is still running.
- <a id="w-remote-call"></a>**remote call** — invoking code on another machine over the network (vs a local, in-process call).
- <a id="w-topology"></a>**topology** — the arrangement of the network: which nodes, links, and routes exist (and it changes).
- <a id="w-transport-cost"></a>**transport cost** — the real money and CPU/serialization overhead of moving data over a network.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 15 — CAP, PACELC & Consistency Models →](lesson-15-cap-consistency){: .btn .btn-primary }
