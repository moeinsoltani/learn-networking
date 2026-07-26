---
title: "Lesson 23 — Scalability & Performance Architecture"
nav_order: 1
parent: "Phase 6: Cross-Cutting Quality Attributes"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 23: Scalability & Performance Architecture

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

"Fast" and "scalable" are not the same thing, and conflating them is a classic mistake. **Latency**[°](#w-latency)
is how long one operation takes; **throughput**[°](#w-throughput) is how many you can do per second; **scalability**[°](#w-scalability)
is whether you can *increase* throughput (usually by adding hardware) as load grows. A system can
be fast but not scalable (blazing on one machine, falls over at 2× load) or scalable but not
especially fast (each request is a bit slow, but you can handle millions by adding nodes). The
architect designs for the *specific* need — and often they trade against each other.

Three words get used interchangeably and mean quite different things.

**Latency** asks *how long does one request take?* — 50 ms. **Throughput** asks
*how many can we handle per second?* — 10,000 requests per second.
**Scalability** asks a different kind of question entirely: *when we add
hardware, does throughput actually rise* — ideally close to linearly?

That third one is the architectural property. A system where adding a node
barely helps **does not scale**, and the reason is always the same: something
shared is the bottleneck — a database, a lock, a queue, a single-writer
service. Adding machines cannot fix contention on a shared resource.

Which points at the main enabler of horizontal scale: **statelessness.** A
stateless service lets any instance serve any request, so scaling out is a
matter of adding instances. A stateful service pins requests to particular
instances, and scaling becomes a data-migration problem wearing a capacity
costume. Push state into a datastore or a cache and the services themselves
become the easy part.

The single most important architectural enabler of scale is **statelessness**: if a service keeps
no per-client state between requests (state lives in a database, cache, or the request itself),
then any instance can handle any request, and you scale simply by running more instances behind a
load balancer. State pinned inside a service instance ("sticky sessions," in-memory user data) is
the enemy of horizontal scale — so a core move is *pushing state out* of the compute layer into
shared stores.

## Going Deeper

**Find the bottleneck before optimizing anything.** A system's throughput is limited by its single
worst **bottleneck**[°](#w-bottleneck) — the slowest, most-contended component — and optimizing anything *else* is wasted
effort (Amdahl's Law: speeding up a part that's 10% of the time can improve things by at most 10%).
So performance work starts with *measurement*, not guessing: profile the system, use the **USE
method** (for each resource — CPU, memory, disk, network — check Utilization, Saturation, Errors)
to find what's actually saturated, and target *that*. The cardinal sin is optimizing based on
intuition ("this loop looks slow") instead of data; the bottleneck is very often not where you'd
guess (it's usually I/O — a database query, a network call — not CPU). "Premature optimization is
the root of all evil" (Knuth) is really "optimize the measured bottleneck, ignore the rest."

**Async and queue-based load leveling.** A powerful scalability pattern: put a **queue** between a
spiky producer and a downstream that can only process at a steady rate. The queue absorbs bursts
(requests pile up in the queue instead of overwhelming or being rejected by the downstream), and
the downstream processes at its sustainable pace. This **load leveling**[°](#w-load-leveling) turns a spiky, peak-driven
capacity problem into a steady-average one — you provision for the *average*, not the *peak*,
because the queue buffers the difference. It's why async processing (Lesson 16) is a scalability
tool, not just a decoupling one: it lets you decouple the *arrival* rate from the *processing* rate.
(The trade: eventual consistency and latency for the queued work — Lesson 15.)

{: .note }
> **Latency is a budget you spend — allocate it before you build**
> A user-facing request often fans out to several services/queries, and its total latency is the
> sum (plus the network hops). Treat that total as a <strong>budget</strong>: if the page must
> respond in 300 ms, and it calls auth (20 ms) + product service (?) + pricing (?) + rendering
> (50 ms), then the product and pricing calls <em>together</em> have ~230 ms to spend, minus
> network. Allocating the budget across the hops <em>before</em> coding reveals where it will blow
> (a call that needs 400 ms in a 230 ms budget is a design problem to solve now — parallelize the
> calls, cache, precompute, or cut a hop — not a surprise to discover in production). This is the
> performance analog of capacity planning, and it's how you design for a latency target instead of
> measuring the miss afterward. Also: measure and design to the <strong>p99</strong>, not the
> average — the average hides the slow tail that real users experience, and at scale the tail
> <em>is</em> the experience (a request fanning out to 10 services hits <em>someone's</em> p99
> nearly every time).

**Premature scaling is a debt too.** The flip side of all this: building for scale you don't have
is over-engineering (Lesson 35). A startup designing for a billion users it may never get pays —
in complexity, cost, and slower delivery — for a benefit that's hypothetical, while a competitor
who kept it simple ships and wins. Statelessness and clean boundaries are cheap and keep the
*option* to scale open; actually building the sharded, multi-region, queue-everywhere machinery
before the load exists is premature. Design so you *can* scale (**stateless**[°](#w-stateless), measurable, evolvable),
but scale *when the numbers demand it* — which is why capacity math and measurement come first.

---

## Lab — Design Exercise

**The situation:** A user-facing "order summary" page must respond in **300 ms at p99**. To render
it, the request fans out to four backend calls: **Auth** (validate the session), **Orders** (fetch
the order), **Pricing** (recompute the current total), and **Recommendations** ("you might also
like"). Today the page calls them **sequentially**, and measured p99 latencies are: Auth 30 ms,
Orders 120 ms, Pricing 250 ms, Recommendations 200 ms.

**Do the latency-budget analysis.** Add up the current path, show where the 300 ms budget blows,
and redesign to fit — using parallelization, caching/precomputation, cutting or degrading a hop,
and the async/graceful-degradation tools from earlier lessons. Say which single change matters most
and what you'd measure to confirm.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is budgeting latency across hops and finding the design fixes before shipping. A strong
answer:
<br><br>
<strong>The current (sequential) path blows the budget badly.</strong> Sequential total ≈ 30 + 120
+ 250 + 200 = <strong>600 ms</strong> at p99 — double the 300 ms budget. (And note these are each
p99s; summed sequentially the tail is even uglier, since the request hits several services' tails.)
So the sequential design is a non-starter; the budget analysis reveals this <em>before</em> a single
line is written.
<br><br>
<strong>Fix 1 — parallelize the independent calls (the biggest single win).</strong> Auth, Orders,
Pricing, and Recommendations are largely independent (you don't need Orders' result to call
Pricing if Pricing takes the order id, etc. — and even if there's a light dependency, most can run
concurrently). Fired in <em>parallel</em>, the total is no longer the <em>sum</em> but the
<em>max</em> — ≈ max(30, 120, 250, 200) = <strong>250 ms</strong> (plus a little coordination
overhead). That alone gets us under 300 ms. Parallelizing fan-out is the highest-leverage change,
because sequential fan-out is the classic self-inflicted latency wound — you were paying the sum
when you could pay the max.
<br><br>
<strong>Fix 2 — attack Pricing (250 ms), now the tall pole.</strong> With parallelization, total ≈
the slowest call, so Pricing (250 ms) sets the floor and leaves almost no margin. Options: cache
the computed price with a short TTL (Lesson 21 — price changes daily-ish, so a few-minutes-stale
display price is acceptable, and the authoritative price is re-checked only at checkout, not here);
or precompute/denormalize the total; or optimize the pricing query. Getting Pricing to, say, ~100
ms gives comfortable headroom.
<br><br>
<strong>Fix 3 — degrade or defer Recommendations (200 ms).</strong> Recommendations is the
<em>least critical</em> part of an order-summary page — it's a "nice to have," not the order. So it
should <em>never</em> be allowed to blow the budget or fail the page (Lesson 18's graceful
degradation): give it a tight timeout (e.g., 100 ms) and, if it doesn't respond in time, render the
page <em>without</em> recommendations (or load them asynchronously/lazily after the page paints).
This both protects the latency budget (a slow rec engine can't drag the page over 300 ms) and the
availability (a down rec engine doesn't fail the order summary). Deferring it to a separate async/
lazy load is even better — it's off the critical path entirely.
<br><br>
<strong>The redesigned path:</strong> fire Auth + Orders + Pricing in parallel (with Pricing cached
down to ~100 ms), giving a critical path of ≈ max(30, 120, 100) ≈ 120 ms; render the page; load
Recommendations lazily/async with a tight timeout and graceful fallback so it's not on the 300 ms
critical path at all. Comfortable p99 well under budget.
<br><br>
<strong>Single most important change:</strong> <em>parallelizing the fan-out</em> — it converts the
sum (600 ms) into the max (250 ms) in one stroke, turning an impossible budget into a nearly-met
one, and it's the change with by far the biggest ratio of impact to effort. (Caching Pricing and
degrading Recommendations then buy the headroom, but parallelization is the structural fix.)
<br><br>
<strong>What to measure to confirm:</strong> the <em>p99</em> end-to-end latency of the page under
realistic peak load (not the average, and not in a quiet test — the tail under load is what the
budget is about), plus the per-hop p99s to verify each stays within its allocation, and the
graceful-degradation behavior (does the page still render on time when Recommendations is slow/
down?). The through-line: a latency target is a <em>budget allocated across hops</em>, the design
must fit it <em>by construction</em> (parallelize, cache, degrade, cut hops) rather than being
measured-and-missed after launch, and the least-critical work belongs off the critical path — all
decided from the numbers, up front.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Latency vs throughput, tail latencies | "The Tail at Scale," Dean & Barroso — <https://research.google/pubs/pub40801/> |
| Statelessness & horizontal scaling | *The Twelve-Factor App* (Processes) — <https://12factor.net/processes> |
| USE method for bottlenecks | Brendan Gregg — <https://www.brendangregg.com/usemethod.html> |
| Queue-based load leveling | <https://learn.microsoft.com/azure/architecture/patterns/queue-based-load-leveling> |
| Amdahl's Law | <https://en.wikipedia.org/wiki/Amdahl%27s_law> |

---

## Checkpoint

**Q1.** Distinguish latency, throughput, and scalability, and explain why statelessness is the key
architectural enabler of horizontal scaling. Give an example of a system that's fast but not
scalable.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Latency</strong> is how long a <em>single</em> operation takes (delay per request, e.g.,
"this call returns in 50 ms"). <strong>Throughput</strong> is how <em>many</em> operations the
system handles per unit time (capacity, e.g., "10,000 requests/second"). <strong>Scalability</strong>
is whether you can <em>increase</em> throughput as load grows — ideally by adding resources
(hardware) and having capacity rise roughly in proportion. They're independent: a system can have
low latency (fast) but poor scalability (can't handle more load), or high scalability but mediocre
latency (each request is a bit slow, but you can serve millions by adding nodes).
<br><br>
<strong>Why statelessness enables horizontal scaling:</strong> horizontal scaling means handling
more load by running more instances of a service. That only works if <em>any instance can handle
any request</em> — which requires that instances hold no per-client state between requests. If a
service is <strong>stateless</strong> (state lives in a shared database/cache or travels in the
request itself), then a load balancer can send any request to any instance, and adding instances
linearly adds capacity — scaling is as easy as starting more copies. If a service is
<strong>stateful</strong> (it keeps a user's session or data in its own memory), then that user's
requests must return to the <em>same</em> instance ("sticky sessions"), which breaks even load
distribution, complicates adding/removing instances (where does the state go?), and caps scaling —
you can't just add copies because the state is trapped in specific ones. So pushing state <em>out</em>
of the compute layer (into shared stores) is what makes the compute layer freely scalable; state in
the instance is the enemy of horizontal scale.
<br><br>
<strong>Fast-but-not-scalable example:</strong> a service that keeps all its working data in memory
on a single machine and serves requests blazingly fast from that in-memory state (low latency) — but
because the data and state live on that one node, you <em>can't</em> add more instances to handle
more load (each new instance wouldn't have the data, and requests can't be freely distributed), and
when load exceeds what the one machine can do, it falls over. It's fast at low load and completely
unscalable — the opposite of a stateless service that might have slightly higher per-request
latency but can absorb 100× the load by adding nodes. (Another: a monolith that's snappy on one big
server but, being stateful/single-instance, has no path to handle a traffic surge except a bigger
box — vertical only, with a ceiling.)
</details>

**Q2.** Why must you find the bottleneck (by measurement) before optimizing, and what is "latency
as a budget"? Why design and measure to p99 rather than the average?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Why measure the bottleneck first:</strong> a system's throughput is limited by its single
worst bottleneck, and (Amdahl's Law) optimizing anything <em>other</em> than the bottleneck yields
almost no improvement — if a component is 10% of the total time, making it infinitely fast improves
the whole by at most 10%. So effort spent optimizing a non-bottleneck is largely wasted. And the
bottleneck is frequently <em>not</em> where intuition suggests — developers guess "this CPU-heavy
loop," but the real limit is usually I/O (a slow database query, a network call). Therefore you must
<em>measure</em> (profile, apply the USE method — check each resource's Utilization, Saturation,
Errors) to find what's actually saturated, and target that. Optimizing by intuition instead of data
is the "premature optimization is the root of all evil" trap — you polish code that doesn't matter
while the real bottleneck (often an unindexed query) sits untouched. Measurement tells you the one
thing worth fixing.
<br><br>
<strong>Latency as a budget:</strong> a user-facing request's total latency is the sum of the work
along its path — each service call, query, and network hop — so a latency <em>target</em> (e.g.,
"300 ms at p99") is a <strong>budget</strong> to be <em>allocated across those hops</em>. You divide
the budget among the components (auth gets X, the data fetch gets Y, rendering gets Z), and check
<em>during design</em> whether each can meet its share and whether the sum fits. This surfaces
problems before coding: if one hop needs 400 ms in a 230 ms remaining budget, that's a design
problem to solve now (parallelize the calls so you pay the max not the sum, cache/precompute, or cut
the hop) — rather than a miss discovered in production. It converts "hope it's fast enough" into
"design it to fit a number."
<br><br>
<strong>Why p99 not average:</strong> the average hides the tail, and the tail is what users
actually feel. A 100 ms <em>average</em> can conceal that 1% of requests take 3 seconds — and those
slow requests are real users having a bad experience (often your most active users, who make the
most requests and thus hit the tail most). Worse, at scale <em>tail latency amplifies</em>: a
request that fans out to 10 services needs <em>all</em> of them to be fast, so it hits <em>someone's</em>
p99 nearly every time — meaning the <em>system's</em> typical latency is governed by the
<em>components'</em> tail latency, not their average (Dean & Barroso, "The Tail at Scale"). So
designing and measuring to p99 (or p99.9) targets the experience that actually matters and accounts
for how tails compound across a distributed request, whereas optimizing the average can make the
headline number look good while the real user experience — the slow tail — stays bad.
</details>

---

## Homework

Take one important user-facing request path in your system and do the latency-budget analysis: what
is (or should be) the target (at p99), and how is that budget spent across the hops/queries on the
path? Find where it blows or is at risk, and identify the fixes (parallelize sequential fan-out,
cache/precompute a slow hop, degrade or defer non-critical work off the critical path). Separately,
assess the system's scalability: which services are stateless (freely horizontally scalable) and
which hold state that would block scaling, and where the first throughput bottleneck would appear
under 10× load. Be honest about anywhere you're scaling (or designing to scale) prematurely.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise applies budget-thinking and scalability analysis to a real path. A strong response:
<br><br>
<strong>Budgets a real path and finds the self-inflicted latency.</strong> The most common,
high-value finding is <em>sequential fan-out</em> that should be parallel — a path making several
independent backend calls one after another, paying the sum when it could pay the max. Naming the
target (at p99), adding up the hops, seeing where it exceeds the budget, and prescribing the fixes
(parallelize the independent calls, cache/precompute the slow hop, give non-critical work like
recommendations a tight timeout + graceful fallback so it's off the critical path) is exactly the
design-to-a-number discipline. A good answer also notes whether the path is measured at p99 at all —
often it's only monitored on averages, hiding the tail users feel.
<br><br>
<strong>Assesses scalability via statelessness and the next bottleneck.</strong> The valuable
analysis: which services are genuinely stateless (state pushed to shared stores → freely add
instances) versus which hold in-memory/sticky state that would block horizontal scaling (a
scaling-blocker to fix by externalizing state). And, under 10× load, where does the <em>first</em>
throughput bottleneck appear? — very often it's the shared database (Lesson 22), or a specific hot
query, or a stateful service, rather than the stateless compute tier. Identifying the real
first-limit (rather than assuming) tells you where scaling effort should actually go.
<br><br>
<strong>Checks for premature scaling honestly.</strong> The balancing finding: is anything built
(or being designed) for scale the system doesn't have — queue-everywhere, sharding, multi-region —
paying complexity now for hypothetical future load? The mature conclusion is usually "keep the cheap
scalability enablers (statelessness, clean boundaries, measurement) that preserve the <em>option</em>
to scale, but don't build the heavy machinery until the numbers demand it." The overall takeaway a
good answer reaches: performance is designed to a measured budget across the request path (parallelize,
cache, degrade), scalability is enabled by statelessness and limited by the first shared bottleneck
(usually data), and both fast and scalable are specific, measured targets — not vague virtues — with
premature scaling being as real a mistake as ignoring scale.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-latency"></a>**latency** — how long *one* operation takes (delay); measured per request.
- <a id="w-throughput"></a>**throughput** — how *many* operations per unit time the system handles (capacity).
- <a id="w-scalability"></a>**scalability** — how well the system handles *more* load, ideally by adding resources.
- <a id="w-stateless"></a>**stateless** — a service that keeps no per-client state between requests, so any instance can serve any request.
- <a id="w-horizontal-scaling"></a>**horizontal scaling** — adding more instances; enabled by statelessness.
- <a id="w-bottleneck"></a>**bottleneck** — the one component that limits the whole system's throughput.
- <a id="w-load-leveling"></a>**load leveling** — using a queue to smooth spiky load so downstream isn't overwhelmed.
- <a id="w-percentile-p99"></a>**percentile (p99)** — the value below which 99% of measurements fall; the "tail" users actually feel.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 24 — Security Architecture & Threat Modeling →](lesson-24-security-architecture){: .btn .btn-primary }
