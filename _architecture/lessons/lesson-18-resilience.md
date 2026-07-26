---
title: "Lesson 18 — Reliability & Resilience Patterns"
nav_order: 5
parent: "Phase 4: Distributed Systems"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 18: Reliability & Resilience Patterns

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

In a distributed system, failure is not an exception — it's the *normal operating condition*.
Something is always slow, restarting, or briefly unreachable (Lesson 14's fallacies made this
concrete). So **resilience**[°](#w-resilience) isn't about *preventing* failure; it's about *designing so that the
failures that will certainly happen stay contained* and the system degrades gracefully instead
of collapsing. The mindset shift: stop asking "how do I make this never fail?" and start
asking "when this fails — and it will — what happens, and how do I limit the **blast radius**[°](#w-blast-radius)?"

Start with how a single slow dependency takes down an entire site.

The Payment service gets slow. Checkout's calls to it hang, because nobody set
a timeout. Checkout's threads all block, waiting. Checkout runs out of threads
and stops responding at all. Everything that calls Checkout now hangs too. One
slow dependency — not even a failed one — has taken down the whole system.

The resilience toolkit exists to contain exactly that chain:

| Pattern | What it does |
|---|---|
| **Timeout** | Don't wait forever — fail fast. The single highest-value line of the list |
| **Retry with backoff** | Survive transient blips — carefully, since naive retries amplify an overload |
| **Circuit breaker** | Stop calling a dependency that is down; fail fast now, probe for recovery later |
| **Bulkhead** | Isolate resources so one failure cannot drain them all |
| **Fallback** | Degrade gracefully — a cached, default, or partial response |
| **Load shedding** | Refuse excess work rather than collapse under it |

Notice that the failure above needed only the first row to be prevented. Most
cascading failures are a missing timeout wearing a more complicated costume.

The classic failure is a *cascade*: one slow dependency, plus a missing **timeout**[°](#w-timeout), exhausts the
caller's threads, which takes the caller down, which takes *its* callers down — one small
problem becomes a total outage. Nearly every resilience pattern exists to break some link in
that chain.

## Going Deeper

**Timeouts — the one everyone forgets, and the most important.** Every remote call must have a
timeout. Without one, a hung dependency makes *you* hang indefinitely, holding threads/
connections until you fall over too (the cascade above). A timeout converts "hang forever" into
"fail fast," which you can then handle (retry, fall back, error out cleanly). If you add only
one resilience mechanism, add timeouts on every network call — it's the single highest-leverage
one, and its absence is the most common cause of cascading outages.

**Retries — necessary but dangerous.** Retrying a failed call handles transient blips (a
dropped packet, a momentary hiccup). But naive retries are a foot-gun (Lesson 2's second-order
effect): if a dependency is struggling because it's *overloaded*, every client retrying
*multiplies* the load — a **retry storm** that turns a partial degradation into a full outage.
Safe retries need: **exponential backoff** (wait longer between attempts), **jitter** (randomize
the wait so all clients don't retry in synchronized waves), a **retry budget/cap** (bounded
attempts), and **idempotency** (Lesson 17 — you may retry something that actually succeeded).
Retry *transient* errors only; don't retry a 400 "bad request" (it'll fail identically).

**Circuit breaker — stop hammering a downed dependency.** Wrapping a dependency in a circuit
breaker: when failures exceed a threshold, the breaker *opens* — subsequent calls fail
*immediately* (without even trying), for a cooldown period. This does two things: it protects
*you* (you're not spending threads/time on calls that will fail — fail fast instead of hang)
and it protects *the dependency* (you stop hammering something that's already down, giving it
room to recover). After the cooldown the breaker goes *half-open*, lets a trial call through,
and closes again if it succeeds. It's the pattern that most directly stops a slow/down
dependency from cascading.

{: .note }
> **Bulkheads — isolate so one leak can't sink the ship**
> Named after a ship's watertight compartments: if one floods, the **bulkheads**[°](#w-bulkhead) keep it from
> sinking the whole vessel. Architecturally, you <em>isolate resources</em> so one
> misbehaving dependency can't consume all of them. Example: if your service calls Payments,
> Search, and Recommendations all from one shared thread pool, a slow Payments will consume
> every thread and starve the calls to Search and Recommendations too — one dependency's
> problem becomes everyone's. Give each dependency its <em>own</em> limited pool (a bulkhead):
> now a slow Payments exhausts only <em>its</em> pool, and Search/Recommendations keep working.
> Bulkheads contain the blast radius so a single failure is partial, not total. The same idea
> scales up: isolating tenants, isolating critical paths from non-critical ones, running the
> risky workload on separate infrastructure.

**Graceful degradation and fallbacks.** When a dependency is down, the best response is often
*partial function*, not total failure. If Recommendations is down, show the page *without* the
recommendations (a fallback), don't fail the whole page. If the live price service is down,
serve a slightly stale cached price. If personalization is unavailable, show a generic
experience. Designing fallbacks means deciding, per dependency, "what's the degraded-but-useful
behavior when this is unavailable?" — turning a hard failure into a soft one. Not everything can
degrade (you can't "gracefully degrade" taking payment), but a surprising amount can, and it's
the difference between "the site is down" and "one feature is temporarily missing."

**Backpressure and load shedding — protect yourself from overload.** When more work arrives than
you can handle, the naive response (accept it all, queue unboundedly, run out of memory, and
collapse) is the worst — you fail *everyone*. **Load shedding** deliberately rejects excess work
(return "try again later" / 503) so the requests you *do* accept succeed — better to serve 80%
well than 100% badly and then crash. **Backpressure** propagates "slow down" upstream so
producers stop overwhelming consumers. Both embody the principle: *a system under overload
should degrade predictably, not collapse catastrophically.*

**Fault vs error vs failure (Nygard), and designing the blast radius.** Nygard's distinction:
a **fault** is a defect/condition (a bug, a dependency down), an **error** is a fault manifesting
(an exception, a bad response), and a **failure** is the system not delivering its service. The
art of resilience is *stopping faults from becoming failures* — a fault (a downed dependency) is
inevitable, but with timeouts, breakers, bulkheads, and fallbacks it produces a contained error
(one feature degraded), not a system failure (the site down). Underlying all of it is
consciously designing the **blast radius**: for each possible failure, how far does the damage
spread, and what limits it? A resilient architecture is one where no single failure has a large
blast radius.

---

## Lab — Design Exercise

**The situation:** Your e-commerce checkout calls a **Payment** service synchronously.
Payment has started getting slow (its own database is struggling). The symptom: when Payment
slows down, the *entire checkout service* becomes unresponsive and customers can't check out at
all — even the parts that don't need Payment. Investigation shows: checkout calls Payment with
*no timeout*, from the *same thread pool* it uses for everything, and *keeps retrying
immediately* on failure.

**Prescribe the resilience patterns, in priority order,** to fix this. For each, explain
exactly what it contains and why. Then describe the graceful-degradation behavior you'd design,
and state which pattern most directly stops the cascade.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is diagnosing the cascade and applying the patterns in the order of leverage. A
strong answer:
<br><br>
<strong>1. Timeouts (first — highest leverage, directly stops the hang).</strong> The root of
the cascade is the no-timeout call: when Payment is slow, checkout threads <em>block
indefinitely</em> waiting, so they're never returned to the pool, the pool exhausts, and
checkout stops responding to <em>everything</em>. Add a timeout on the Payment call (say, a few
hundred ms to a couple seconds, tuned to Payment's normal latency): now a slow Payment call
<em>fails fast</em> instead of hanging forever, threads are released, and the pool doesn't
exhaust. This is the single most important fix — the cascade fundamentally depends on the
unbounded wait, and the timeout removes it.
<br><br>
<strong>2. Fix the retries (they're currently making it worse).</strong> Checkout "keeps
retrying immediately on failure" — a <strong>retry storm</strong>: when Payment is struggling
(overloaded), immediate retries <em>multiply</em> the load on it, deepening its problem and
guaranteeing the failures continue (second-order effect). Replace with <em>exponential backoff +
jitter</em>, a bounded retry budget, and retry only <em>transient</em> errors (not a definitive
decline). Combined with idempotency (Lesson 17 — a retried charge must not double-charge), this
lets checkout survive brief blips without hammering a downed Payment into the ground.
<br><br>
<strong>3. **Circuit breaker**[°](#w-circuit-breaker) on the Payment call.</strong> Even with timeouts, if Payment is down
for minutes, every checkout request still <em>tries</em> Payment, waits for the timeout, then
fails — wasting time/threads and continuing to pound a dead dependency. Wrap Payment in a
<strong>circuit breaker</strong>: after failures cross a threshold, the breaker <em>opens</em>
and calls fail <em>immediately</em> (no timeout wait) for a cooldown, then half-opens to test
recovery. This protects checkout (fail instantly instead of waiting out timeouts on every
request) <em>and</em> Payment (stops the hammering so it can recover).
<br><br>
<strong>4. Bulkhead — isolate Payment's thread pool.</strong> The reason a slow Payment took
down <em>even the parts of checkout that don't need Payment</em> is the <em>shared thread
pool</em>: Payment calls consumed every thread, starving everything else. Give Payment calls
their <em>own</em> limited pool (a <strong>bulkhead</strong>): now a slow/down Payment can only
exhaust <em>its</em> pool, and the rest of checkout (cart viewing, order history, anything not
needing Payment) keeps working. This contains the blast radius from "all of checkout" to "just
the payment step."
<br><br>
<strong>Graceful degradation:</strong> decide the degraded behavior when Payment is unavailable.
You can't "gracefully degrade" actually taking a payment — but you <em>can</em> turn a hard
failure into a soft one: show the customer a clear "payments are temporarily unavailable, your
cart is saved, please try again shortly" message (not a generic error or a hung page), keep the
rest of the site fully functional, and — a stronger option — queue the order for
<em>asynchronous</em> payment processing if the business allows ("we'll confirm your order
shortly"). At minimum, the failure is contained to the payment step with a clear message, while
browsing, cart, and everything else stay up.
<br><br>
<strong>Which pattern most directly stops the cascade:</strong> the <strong>timeout</strong> —
because the cascade's mechanism is unbounded waiting exhausting the thread pool, and the timeout
is what converts "hang forever" into "fail fast," releasing threads before the pool drains.
(The <strong>bulkhead</strong> is the close second and the reason the failure was <em>total</em>
rather than partial — with per-dependency pools, even a fully hung Payment would have left the
rest of checkout working from the start.) Priority order overall: timeouts first (stop the
hang), fix retries (stop making it worse), circuit breaker (stop hammering/waiting on a down
dependency), bulkhead (contain the blast radius), fallback (degrade gracefully). Together they
turn "one slow dependency = whole site down" (a fault becoming a system failure) into "one slow
dependency = the payment step is temporarily degraded with a clear message" (a fault contained
to a small blast radius) — which is the entire goal of resilience engineering.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Stability patterns (timeout, circuit breaker, bulkhead, etc.) | Michael Nygard, *Release It!* (2e) |
| Circuit breaker | Martin Fowler — <https://martinfowler.com/bliki/CircuitBreaker.html> |
| Bulkhead & isolation | <https://learn.microsoft.com/azure/architecture/patterns/bulkhead> |
| Retries, backoff & jitter | AWS Builders' Library — "Timeouts, retries, and backoff with jitter" |
| Load shedding & backpressure | Google SRE Book — "Handling Overload" |

---

## Checkpoint

**Q1.** Walk through a cascading failure caused by a single slow dependency, and explain which
resilience pattern breaks the cascade at which point. Why is the timeout the highest-leverage
single fix?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>The cascade:</strong> a downstream dependency (say Payment) gets slow. A caller
(Checkout) makes synchronous calls to it <em>with no timeout</em>, so each call
<em>blocks indefinitely</em> waiting for a response. Those blocked calls hold onto Checkout's
worker threads (or connections), and because they never return, the threads are never freed. As
more requests come in and also block on Payment, Checkout <em>exhausts its thread pool</em> —
now it has no threads left to serve <em>any</em> request, including ones that don't touch
Payment. Checkout stops responding entirely. Then everything that calls Checkout starts hanging
too (same mechanism), and the failure propagates outward until a single slow dependency has
taken down the whole system.
<br><br>
<strong>Where each pattern breaks it:</strong>
<ul>
<li><strong>Timeout</strong> breaks it at the first link: with a timeout, the call to Payment
<em>fails fast</em> instead of blocking forever, so the thread is released back to the pool and
the pool never exhausts. The cascade can't start.</li>
<li><strong>Bulkhead</strong> breaks it at the "took down everything" link: with Payment on its
own isolated thread pool, even a fully-hung Payment only exhausts <em>that</em> pool, so
Checkout's other functions keep their threads and stay up — the failure is partial, not total.</li>
<li><strong>Circuit breaker</strong> breaks the "keep trying a dead dependency" link: once
Payment is clearly down, the breaker opens and calls fail instantly (no waiting even for the
timeout), so Checkout stops spending resources on doomed calls and stops hammering Payment.</li>
<li><strong>Fixing retries</strong> (backoff + jitter) breaks the "make it worse" link:
immediate retries would multiply load on the struggling Payment; backoff prevents the retry
storm.</li>
</ul>
<strong>Why the timeout is the highest-leverage single fix:</strong> the cascade's fundamental
mechanism is <em>unbounded waiting exhausting a finite resource (threads/connections)</em>.
Everything downstream of that — pool exhaustion, unresponsiveness, propagation — follows from
the fact that calls hang forever. The timeout attacks that root directly: it makes waiting
<em>bounded</em>, so threads are always eventually released and the pool can't be drained by a
slow dependency. Without a timeout, the other patterns are less effective (a circuit breaker
still needs to <em>detect</em> failures, which requires calls to fail rather than hang forever;
a bulkhead limits the damage but that pool still exhausts). With a timeout, the worst case
becomes "calls fail fast and we handle it" instead of "calls hang and we die." It's the
cheapest change with the biggest effect, which is why "every remote call must have a timeout"
is the first rule of resilience — and why a missing timeout is the most common root cause of
cascading outages.
</details>

**Q2.** Why are retries both necessary and dangerous? Give the specific mechanism by which
naive retries make an outage worse, and the four things that make retries safe.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Necessary:</strong> in a distributed system many failures are <em>transient</em> — a
dropped packet, a brief network hiccup, a momentary GC pause, a quick restart. Retrying such a
call usually succeeds on the next attempt, so retries let the system ride through the constant
minor blips that are normal (Lesson 14) instead of surfacing every one as a user-visible error.
Without retries, transient failures become permanent failures for the user.
<br><br>
<strong>Dangerous — the mechanism:</strong> the danger appears precisely when the failure is
<em>not</em> a random blip but <em>overload</em>. If a dependency is failing/slow because it's
overloaded, then every client retrying <em>adds more load</em> to the already-struggling
service. Naive (immediate, unbounded) retries mean each failing request becomes 2, 3, or more
requests, all aimed at the service that's drowning — a <strong>retry storm</strong> that
multiplies the load exactly when the service can least handle it, deepening the overload,
causing more failures, causing more retries, in a vicious cycle. This is the classic
second-order effect (Lesson 2): the mechanism that helps the common case (transient blips) makes
the worst case (overload) dramatically worse, turning a partial degradation into a full,
self-sustaining outage.
<br><br>
<strong>The four things that make retries safe:</strong>
<ol>
<li><strong>Exponential backoff</strong> — wait progressively longer between attempts (e.g.,
100ms, 200ms, 400ms…) so retries don't pile on immediately, giving a struggling dependency room
to recover instead of being hammered.</li>
<li><strong>Jitter</strong> — add randomness to the backoff so that many clients that all failed
at the same instant don't all retry at the <em>same</em> later instant (synchronized waves that
re-create the storm); jitter spreads the retries out.</li>
<li><strong>A retry budget / cap</strong> — bound the number of attempts (and ideally the total
retry rate across the system), so a persistent failure results in a bounded, finite amount of
retry load rather than infinite retrying that never lets the dependency recover.</li>
<li><strong>Idempotency</strong> (Lesson 17) — because a retry might be re-sending an operation
that actually <em>succeeded</em> (the request worked but the response was lost), the operation
must be idempotent so retrying it doesn't double-apply (double-charge, double-ship). Retrying a
non-idempotent operation is a correctness bug.</li>
</ol>
(Plus: retry only <em>transient/retryable</em> errors — a 500/timeout, not a 400 "bad request"
that will fail identically every time — and ideally combine with a circuit breaker so that once
a dependency is clearly down, you stop retrying entirely and fail fast until it recovers.)
Together these convert retries from a foot-gun into a safe mechanism: they still absorb transient
failures, but they can no longer amplify an overload into a storm, and they can't corrupt state
by re-applying succeeded operations.
</details>

---

## Homework

Audit the resilience of one critical synchronous call path in your system. For each remote call
on the path, check: does it have a timeout? Are retries safe (backoff, jitter, cap, idempotent)
or naive? Is there a circuit breaker on the risky dependencies? Are resources bulkheaded, or
would one slow dependency starve the others? Is there a graceful-degradation fallback, or does
one failure fail the whole path? Identify the single biggest gap (most likely a missing
timeout or a shared resource pool) and the failure it would cause, then design the fix. Finally,
for one important dependency, define what "graceful degradation" would look like when it's
unavailable.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise turns the resilience toolkit into a concrete audit of a real path. A strong
response:
<br><br>
<strong>Runs the checklist per call and finds the gaps.</strong> The near-universal findings on
real systems: at least one remote call with <em>no timeout</em> (the highest-severity gap — a
cascading-failure waiting to happen), retries that are naive or absent (immediate retries that
would storm, or none so transient blips become user errors), no <em>circuit breaker</em> on
dependencies that can go down, and a <em>shared thread pool / connection pool</em> across
dependencies (so one slow dependency would starve the rest — no bulkheading). Naming the
specific failure each gap would cause (this missing timeout could hang the whole service; this
shared pool means a slow Search takes down checkout too) is what makes the audit actionable
rather than a checklist.
<br><br>
<strong>Prioritizes the single biggest gap.</strong> The architect's move is to rank by blast
radius, and the top gap is almost always either a <em>missing timeout on a critical call</em>
(can take the whole path/service down via thread exhaustion) or a <em>shared resource pool</em>
(makes any single dependency failure total rather than partial). A good answer picks one,
explains the cascade it enables, and specifies the fix (add the timeout tuned to the
dependency's normal latency; give the risky dependency its own bulkheaded pool) — the
cheapest, highest-leverage change, done first.
<br><br>
<strong>Designs graceful degradation for one dependency.</strong> The valuable per-dependency
question: "what's the degraded-but-useful behavior when this is down?" — show the page without
recommendations, serve a stale cached value, show a generic experience, or (for something that
can't degrade, like taking payment) a clear "temporarily unavailable, cart saved, try again"
message plus keeping the rest of the site up. Deciding this per dependency turns hard failures
into soft ones and is the difference between "the site is down" and "one feature is briefly
missing." The overall takeaway a good answer reaches: resilience is not one feature but a set of
deliberate choices (timeout, safe-retry, breaker, bulkhead, fallback, load-shedding) applied per
call path to contain the blast radius of the failures that <em>will</em> happen — and most
systems have latent cascading-failure risks (a missing timeout, a shared pool) that haven't
fired <em>yet</em> only because the dependency hasn't been slow at the wrong moment.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-resilience"></a>**resilience** — the ability to keep working (perhaps degraded) despite failures, and to recover.
- <a id="w-timeout"></a>**timeout** — a limit on how long you'll wait for a response before giving up.
- <a id="w-retry-with-backoff-jitter"></a>**retry with backoff + jitter** — retrying a failed call, waiting progressively longer, with randomness to avoid synchronized storms.
- <a id="w-circuit-breaker"></a>**circuit breaker** — a switch that stops calling a failing dependency for a while, so you don't hammer it or hang on it.
- <a id="w-bulkhead"></a>**bulkhead** — isolating resources (like thread pools) so one overloaded dependency can't sink the whole ship.
- <a id="w-backpressure-load-shedding"></a>**backpressure / load shedding** — refusing or slowing incoming work when overloaded, instead of collapsing.
- <a id="w-blast-radius"></a>**blast radius** — how far the damage spreads when one thing fails.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 19 — Choosing a Data Store (Polyglot Persistence) →](lesson-19-choosing-datastore){: .btn .btn-primary }
