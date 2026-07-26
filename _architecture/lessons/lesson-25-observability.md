---
title: "Lesson 25 — Observability Architecture"
nav_order: 3
parent: "Phase 6: Cross-Cutting Quality Attributes"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 25: Observability Architecture

{: .note }
> **Words to know**
> - **observability** — how well you can understand a system's internal state from its outputs, including for problems you didn't anticipate.
> - **monitoring** — watching predefined metrics/alerts for known failure modes; a subset of observability.
> - **the three pillars** — logs, metrics, and traces.
> - **distributed tracing** — following one request as it flows across many services, stitched by a shared trace ID.
> - **correlation / trace ID** — a unique id attached to a request and propagated to every service it touches.
> - **structured logging** — logs as machine-parseable key-value data (JSON), not free-form text.
> - **SLI / SLO / error budget** — a measured indicator, a target for it, and the allowed amount of failure.

## Concept

A distributed system you can't *see into* is a system you can't *operate*. When something breaks
across ten services, "check the logs" is useless if you can't tell which service, which request, or
what the request was doing. **Observability** — the ability to understand what your system is doing
and why, *including for problems you never anticipated* — is not a tool you buy after an outage; it's
an architectural property you must design in, because the hooks (trace IDs threaded through every
service, structured logs, instrumented code) have to be built into the system itself.

```
   MONITORING vs OBSERVABILITY

   MONITORING (known-unknowns)      OBSERVABILITY (unknown-unknowns)
   "CPU > 80%? alert."              "checkout is slow for some users —
   Predefined dashboards/alerts      why? which service? which requests?"
   for failures you FORESAW.         Explore/ask new questions of the
                                     system's outputs after the fact.
   ── monitoring is a SUBSET of observability ──

   THE THREE PILLARS
   LOGS    — discrete events: "what happened here, in detail"
   METRICS — aggregated numbers over time: "how much / how fast / how many"
   TRACES  — one request across services: "where did the time/error go"
              stitched by a shared TRACE ID threaded from the edge ───┐
                                                                       ▼
   [edge]──trace:abc──▶[svc A]──abc──▶[svc B]──abc──▶[svc C]  ← follow ONE request
```

**Monitoring** answers the questions you knew to ask (CPU, error rate, latency dashboards for known
failure modes — *known-unknowns*). **Observability** is broader: the ability to ask *new* questions
you didn't predefine ("why is checkout slow *for users in this region on this device*?" — the
*unknown-unknowns* that cause the hardest incidents). In a distributed system, you need both, and the
observability part must be architected in.

## Going Deeper

**The three pillars, and what each answers.**
- **Logs** — discrete, timestamped records of events ("order 123 failed validation: missing
  address"). They tell you *what happened* in detail at a point. The architectural requirement:
  **structured logging** (JSON key-value, not free text) so logs are queryable/aggregatable, and
  **centralized** (shipped to one searchable place, not sitting on individual hosts), because in a
  distributed system the relevant log lines are scattered across many services.
- **Metrics** — numeric measurements aggregated over time (request rate, error rate, latency
  percentiles, queue depth, CPU). They tell you *how much / how fast / how many*, cheaply and
  continuously, and are what you alert and dashboard on. Two useful method-frameworks: **RED** (for
  services: Rate, Errors, Duration) and **USE** (for resources: Utilization, Saturation, Errors —
  Lesson 23).
- **Traces** — the path of a *single request* across all the services it touches, showing where the
  time went and where it failed. This is the pillar that distributed systems *cannot live without*
  and monoliths didn't need: in a monolith one stack trace shows the whole request; across ten
  services, only distributed tracing reconstructs the end-to-end journey.

{: .warning }
> **The correlation/trace ID is an architectural requirement — thread it from the edge**
> The single most important observability decision in a distributed system: assign every incoming
> request a unique <strong>trace ID</strong> at the edge (the gateway), and <strong>propagate it to
> every service, log line, and event</strong> the request touches. Without it, when "checkout
> failed," you have ten services' worth of unrelated logs and no way to know which lines belong to
> <em>this</em> request — you're debugging blind. With it, you filter every log and span by the one
> trace ID and see the whole request's journey across all services in order. This is not something
> you can bolt on after an incident — it must be built into how services receive, log, and forward
> requests (typically via a shared context/header, e.g., W3C Trace Context / OpenTelemetry). An
> architect who splits a system into services <em>without</em> mandating trace-ID propagation has
> designed a system that's nearly impossible to debug — the correlation ID is as fundamental to a
> distributed architecture as the service boundaries themselves.

**Observability must be designed in, not retrofitted.** The recurring theme: you cannot add real
observability the day an outage hits, because it depends on instrumentation woven through the code
and the request path — trace IDs propagated, structured logs emitted, key operations timed, business
and system metrics exported. If those hooks aren't there, the data doesn't exist to answer the
question, and you're guessing during the incident. So observability is an architectural
<em>requirement</em> established up front (part of the platform every service is built on — a
standard logging library, a tracing SDK like OpenTelemetry, a metrics client), exactly like the
"you must be this tall" prerequisites for microservices (Lesson 11). Splitting into services without
building observability first is one of the classic ways distributed systems become operationally
miserable.

**SLIs, SLOs, and error budgets — the language of reliability with the business.** Observability
data becomes decision-making when you frame it as:
- **SLI (Service Level Indicator)** — a measured signal of health, from the user's perspective (e.g.,
  "the proportion of requests served successfully in under 300 ms").
- **SLO (Service Level Objective)** — the target for an SLI (e.g., "99.9% of requests succeed under
  300 ms, measured monthly").
- **Error budget** — the allowed shortfall (99.9% target ⇒ 0.1% ≈ ~43 min/month of failure is
  "budgeted"). This reframes reliability from "never fail" (impossible) to "stay within budget," and
  becomes a shared, quantitative language with product/leadership: when the error budget is healthy,
  ship features faster; when it's spent, slow down and invest in reliability. It turns observability
  from raw data into a governance tool, and connects to the business (Lesson 3's measurable quality
  attributes) — an SLO is a quality-attribute scenario you continuously measure.

**The cost dimension.** Observability isn't free — logs, metrics, and traces at high volume cost real
money (storage, ingestion, vendor bills) and can even affect performance. So there's a trade-off
(Lesson 2): sample traces (you rarely need 100%), set log levels sensibly, aggregate metrics, and
retain data by usefulness. The architect designs observability that answers the important questions
without an unbounded bill — enough visibility to operate, calibrated to cost.

---

## Lab — Design Exercise

**The situation:** Your e-commerce system spans five services on a request path: **Gateway →
Orders → Pricing → Inventory → Payment**. A customer complaint arrives: "checkout is slow and
sometimes fails." Right now, each service logs to its own local files in free-text format, there are
no request IDs, latency is only visible as per-service CPU graphs, and there's no way to follow a
single checkout across the services. The on-call engineer spends hours guessing which service is at
fault.

**Design the observability architecture** so that "checkout is slow/failing" can be diagnosed in
minutes. Specify what you'd instrument across the three pillars — the trace-ID propagation, what to
log (and how), which metrics (RED), and how tracing ties it together — and define one SLO with an
error budget for checkout. Explain what changes about the on-call engineer's experience.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is designing the three pillars around answering real diagnostic questions, anchored by
trace propagation. A strong answer:
<br><br>
<strong>The foundational fix — trace-ID propagation (do this first).</strong> Assign every checkout
request a unique <strong>trace ID</strong> at the Gateway and propagate it through Orders → Pricing
→ Inventory → Payment (via a standard header/context — W3C Trace Context / OpenTelemetry). Every log
line and every span carries it. This single change is what turns "ten services' worth of unrelated
logs" into "filter by this one ID and see the entire checkout's journey." Without it, none of the
rest is much use; with it, diagnosis becomes possible.
<br><br>
<strong>Distributed tracing (the pillar this problem most needs).</strong> Instrument each service
to emit a <em>span</em> for its work (and its outgoing calls), all sharing the trace ID, sent to a
tracing backend (Jaeger/Tempo/vendor). Now a slow/failed checkout produces a <strong>waterfall</strong>
showing exactly where the time went and where it failed: "Gateway 20ms → Orders 30ms → Pricing
<strong>2,400ms</strong> → Inventory 40ms → Payment failed." The engineer <em>sees</em> that Pricing
is the slow hop (or that Payment threw the error), instead of guessing. This is the highest-value
pillar for a "which service in the chain is at fault?" problem — exactly what a distributed system
needs and a monolith's single stack trace used to give for free.
<br><br>
<strong>Structured, centralized logging.</strong> Replace free-text local-file logs with
<strong>structured</strong> logs (JSON key-value, including the trace ID, service name, user/order
id, and event) shipped to a <strong>central</strong> searchable store (ELK/Loki/vendor). Now the
engineer filters all five services' logs by the failing checkout's trace ID and reads the whole
story in order — "Pricing: timeout calling promotions DB" — instead of SSHing into five hosts to
grep free text. Logs answer "what exactly happened," traces answer "where in the flow."
<br><br>
<strong>Metrics — RED per service, on the checkout path.</strong> Export <strong>Rate</strong>
(requests/sec), <strong>Errors</strong> (error rate), and <strong>Duration</strong> (latency
<em>percentiles</em>, esp. p99 — Lesson 23) for each service, plus business metrics (checkout success
rate, checkouts/min). Dashboards and alerts on these catch the problem <em>proactively</em> — "checkout
error rate spiked, Pricing p99 jumped to 2s" — often before customers complain, and tell you the
<em>scope</em> (all users? one region?) that traces then let you drill into. (Per-service CPU graphs
alone were the wrong signal — they measure a resource, not the user-facing symptom.)
<br><br>
<strong>One SLO with an error budget for checkout.</strong> Define an SLI — "the proportion of
checkout requests that succeed in under, say, 2 seconds" — and an SLO — "99.9% of checkouts succeed
under 2s, measured monthly." The <strong>error budget</strong> is the allowed 0.1% (~43 min/month of
failing/slow checkouts). This makes reliability measurable and shared with the business: while the
budget is healthy, ship features; when incidents burn it, prioritize reliability work. The "checkout
is slow and sometimes fails" complaint now has a <em>number</em> attached and a threshold that
triggers action.
<br><br>
<strong>What changes for the on-call engineer:</strong> the incident goes from <em>hours of
guessing</em> to <em>minutes of diagnosis</em>. They (1) see from metrics/alerts that checkout error
rate and Pricing latency spiked (the <em>what</em> and <em>scope</em>), (2) pull a trace of a failing
checkout and see the waterfall pinpoint Pricing as the slow/failing hop (the <em>where</em>), and (3)
filter the central structured logs by that trace ID to read Pricing's exact error (the <em>why</em> —
"timeout to promotions DB"). Three pillars, one trace ID tying them together, answering what/where/
why in sequence. The meta-lesson: this capability is <em>architected in</em> (trace propagation,
structured/central logging, RED metrics, an SLO) — it can't be conjured during the incident, which
is exactly why observability is a design-time requirement for any distributed system, as fundamental
as the service boundaries, and the correlation ID threaded from the edge is its keystone.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Observability vs monitoring; the three pillars | *Observability Engineering*, Majors, Fong-Jones & Miranda |
| Distributed tracing & context propagation | OpenTelemetry — <https://opentelemetry.io/docs/concepts/> ; W3C Trace Context |
| RED & USE methods | Tom Wilkie (RED); Brendan Gregg (USE) — <https://www.brendangregg.com/usemethod.html> |
| SLIs, SLOs, error budgets | Google SRE Book — <https://sre.google/sre-book/service-level-objectives/> |
| Structured logging | <https://www.honeycomb.io/blog/structured-logging-and-your-team> |

---

## Checkpoint

**Q1.** Distinguish monitoring from observability, and explain why the three pillars (logs, metrics,
traces) each answer a different question. Why is distributed tracing the pillar a distributed system
can't live without?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Monitoring vs observability:</strong> <em>monitoring</em> is watching predefined metrics and
alerts for failure modes you <em>anticipated</em> — the known-unknowns ("alert if CPU &gt; 80% or
error rate &gt; 1%"). It answers questions you knew to ask in advance. <em>Observability</em> is the
broader ability to understand the system's internal state from its outputs well enough to ask
<em>new</em> questions you <em>didn't</em> predefine — the unknown-unknowns ("why is checkout slow
specifically for users in this region on this device version?"). Monitoring is a subset: you can
monitor what you foresaw, but the hardest incidents are the ones you didn't foresee, and those
require observability — the ability to explore and interrogate the system after the fact.
<br><br>
<strong>The three pillars answer different questions:</strong> <em>Logs</em> are discrete event
records — they answer "<em>what exactly happened</em> at this point?" in detail (the specific error,
the specific input). <em>Metrics</em> are aggregated numbers over time — they answer "<em>how much /
how fast / how many</em>?" (request rate, error rate, latency percentiles) cheaply and continuously,
and are what you dashboard and alert on to catch problems and see their scope. <em>Traces</em> follow
a single request across services — they answer "<em>where did the time or the error go</em>?" in the
end-to-end flow. They're complementary: metrics tell you <em>something</em> is wrong and how
widespread; traces tell you <em>where</em> in the request path; logs tell you <em>why</em> at that
spot. You need all three because each answers a question the others can't.
<br><br>
<strong>Why distributed tracing is indispensable for distributed systems:</strong> in a monolith, a
single request runs in one process, so one stack trace (and one log stream) shows the entire request
end to end — you can see where it spent time and where it failed. When the same request is split
across ten services, that unified view is <em>gone</em>: each service sees only its own slice, and
the request's journey is scattered across ten processes with no inherent connection. Distributed
tracing reconstructs the end-to-end journey by stitching together the per-service spans using a
shared trace ID propagated across every hop — giving you back the "follow this one request through
the whole system" view that the monolith had for free. Without it, when a cross-service request is
slow or fails, you literally cannot tell which of the ten services is responsible or what the request
was doing — you're reduced to guessing (the "on-call spends hours" scenario). Metrics and logs alone
can't do this: metrics are aggregated (they lose the individual request), and logs are per-service
(they don't connect across services without the trace ID). So tracing is the pillar that specifically
solves the problem distribution <em>created</em> — the loss of the single, followable request path —
which is why it's the one a distributed architecture cannot operate without, and why threading a
trace ID from the edge is a first-class architectural requirement, not an afterthought.
</details>

**Q2.** Why must observability be designed in rather than retrofitted, and what are SLIs/SLOs/error
budgets — why do they turn observability data into a shared language with the business?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Why designed in, not retrofitted:</strong> observability depends on <em>instrumentation
woven through the system</em> — trace IDs generated at the edge and propagated across every service,
structured logs emitted with that context, key operations timed, and business/system metrics
exported. If those hooks aren't already in the code and the request path when an incident hits, the
<em>data simply doesn't exist</em> to answer the question — you can't retroactively produce a trace
for a request that wasn't traced, or filter logs by an ID that was never attached. During the outage
you'd have to add the instrumentation, redeploy, and wait for the problem to recur — useless in the
moment. So observability is an architectural capability established up front (a standard tracing SDK
like OpenTelemetry, a structured-logging library, a metrics client that every service builds on),
exactly like the operational prerequisites for microservices (Lesson 11). Splitting a system into
services without first building trace propagation, centralized structured logging, and metrics is
one of the classic ways teams create a distributed system they can't debug — the visibility has to
be part of the platform, not something bolted on after the first painful incident.
<br><br>
<strong>SLIs / SLOs / error budgets:</strong> an <strong>SLI</strong> (Service Level Indicator) is a
measured signal of health from the user's perspective — e.g., "the fraction of requests that succeed
in under 300 ms." An <strong>SLO</strong> (Service Level Objective) is the target for that indicator
— e.g., "99.9% of requests succeed under 300 ms, measured monthly." The <strong>error budget</strong>
is the allowed shortfall implied by the SLO — 99.9% means 0.1% (~43 minutes/month) of failure is
acceptable, "budgeted." Together they reframe reliability from the impossible "never fail" to the
measurable "stay within budget."
<br><br>
<strong>Why they're a shared language with the business:</strong> raw observability data (graphs,
logs, traces) is engineering detail that product and leadership can't act on directly. SLOs and error
budgets translate it into a <em>quantified, business-legible</em> agreement about how reliable the
system should be, with an explicit, shared threshold for action. This enables concrete, non-emotional
conversations: when the error budget is <em>healthy</em>, the team can prioritize shipping features
(reliability is fine, spend the budget on velocity); when the budget is <em>spent</em> (too many
incidents), everyone — including product — agrees to slow feature work and invest in reliability. It
turns "is the system reliable enough?" from a vague argument into a number both sides can see and
govern by, and it connects observability to the business by making a quality attribute (reliability)
measurable and continuously tracked — an SLO is essentially a quality-attribute scenario (Lesson 3)
you monitor forever. So SLIs/SLOs/error budgets are what elevate observability from "data we look at
during incidents" to "a governance tool that aligns engineering and the business on reliability."
</details>

---

## Homework

Assess your current system's observability by imagining a specific cross-service incident ("feature X
is slow/failing for some users") and asking: could you diagnose it in minutes? Check each pillar — do
you have trace IDs propagated across services (can you follow one request end to end?), structured and
centralized logs (or free text on scattered hosts?), and RED-style metrics at percentiles (or just
resource graphs?). Identify the single biggest observability gap (very often: no trace-ID propagation,
so requests can't be followed across services) and what it costs you during incidents. Then define one
SLO with an error budget for a critical user journey, and note how it would change your
reliability-vs-features conversations.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise stress-tests observability against a realistic incident. A strong response:
<br><br>
<strong>Runs the "could I diagnose this in minutes?" test per pillar</strong> and honestly finds the
gaps. The most common and most damaging finding is <em>no trace-ID propagation</em> — requests can't
be followed across services, so a cross-service problem means correlating scattered logs by hand
(timestamps and guesswork), which is exactly why incidents take hours. Other frequent gaps:
free-text/local logs instead of structured+centralized (can't query across services), and metrics
that are only resource graphs (CPU/memory) rather than user-facing RED signals at percentiles (so you
see the machine is busy but not that checkout's p99 spiked). Naming what each gap <em>costs</em> during
an incident (hours of guessing, no way to isolate the failing service, no early warning) makes the
case concrete.
<br><br>
<strong>Identifies the single biggest gap and its remedy.</strong> For most distributed systems the
top gap is the missing correlation/trace ID (the keystone — without it, the other pillars are far less
useful for cross-service problems), so the highest-leverage fix is threading a trace ID from the edge
through every service, log, and event (via OpenTelemetry / W3C Trace Context) — a foundational,
platform-level change. A good answer prioritizes this over, say, prettier dashboards, because it's
what unlocks minutes-not-hours diagnosis.
<br><br>
<strong>Defines an SLO and reflects on the business conversation.</strong> Writing one real SLI/SLO/
error budget for a critical journey (e.g., "99.9% of checkouts succeed under 2s monthly") turns
reliability into a shared number and a threshold for action. The reflection a strong answer reaches:
this reframes reliability-vs-features from a recurring emotional tug-of-war ("engineering wants to fix
tech debt, product wants features") into a governed trade-off (healthy budget → ship features; spent
budget → invest in reliability, and product agrees because the number is shared). The overall takeaway:
observability is an architected capability (trace propagation + structured/central logs + RED metrics
+ SLOs) that must exist <em>before</em> the incident, its keystone is the correlation ID, and most
systems have a concrete, high-value gap (usually tracing) whose fix converts painful multi-hour
incidents into fast diagnoses — a specific, prioritized investment rather than a vague "improve
monitoring."
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 26 — API Design & Management →](lesson-26-api-design){: .btn .btn-primary }
