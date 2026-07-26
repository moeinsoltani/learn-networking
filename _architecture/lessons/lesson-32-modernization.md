---
title: "Lesson 32 — Modernizing Legacy: the Strangler Fig & Friends"
nav_order: 5
parent: "Phase 7: Documenting, Evaluating & Evolving Architecture"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 32: Modernizing Legacy — the Strangler Fig & Friends

{: .note }
> **Words to know**
> - **legacy system** — a running system that's valuable (it's in production, earning money) but hard to change; usually old, under-documented, and business-critical.
> - **big-bang rewrite** — replacing the whole system at once with a from-scratch rebuild, then cutting over. Usually fails.
> - **second-system effect** — the tendency of a from-scratch replacement to become over-engineered and bloated with everything the first one lacked.
> - **strangler fig** — grow the new system *around* the old, redirect functionality piece by piece, and delete the old once it's starved of traffic.
> - **branch by abstraction** — introduce an abstraction over the thing you're replacing, swap the implementation behind it, then remove the old.
> - **anti-corruption layer (ACL)** — a translation layer that keeps the old system's model from leaking into the new one (Lesson 7).
> - **seam** — a place where you can change behavior without editing in the surrounding code; the point where you can insert a redirect.
> - **parallel run** — run old and new side by side on the same inputs and compare outputs, before trusting the new one.

## Concept

You will rarely design a system on a blank page. Far more often you'll inherit a **legacy system**: old,
tangled, under-documented — and *running the business*, which is exactly why it's hard to change and why
you can't just stop the world to fix it. The instinct is the **big-bang rewrite**: freeze the old,
rebuild it from scratch, cut over on a big day. This almost always fails. The alternative — and the
central skill of modernization — is to change a running system **incrementally**, without ever stopping
it, using the **strangler fig** pattern.

```
   BIG-BANG REWRITE (usually fails)        STRANGLER FIG (grow new around old)
   ┌─────────────────────────┐             step 0: [ facade ]──▶[ LEGACY (all of it) ]
   │ freeze old · rebuild all │            step 1: [ facade ]─┬▶[ new: notifications ]
   │ from scratch · cut over  │                              └▶[ legacy: the rest    ]
   │ on the big day           │            step 2: [ facade ]─┬▶[ new: notifications ]
   │  ✗ target keeps moving   │                              ├▶[ new: billing        ]
   │  ✗ 2nd-system bloat      │                              └▶[ legacy: shrinking   ]
   │  ✗ lose fixed edge cases │            …    redirect route by route, verify each
   │  ✗ no value for months   │            step N: legacy starved → DELETE it
   └─────────────────────────┘             the system runs the WHOLE time
```

The strangler fig (named for the vine that grows around a tree until the tree is gone) works by placing
a **facade/router** in front of the old system and then, **one capability at a time**, building the new
implementation, redirecting that route to it, verifying it, and moving on — until the old system is
starved of traffic and can be deleted. The system keeps running and delivering value the entire time,
risk is taken in small increments, and each step is reversible. Supporting tools — **branch by
abstraction**, the **anti-corruption layer**, and **parallel run** — handle the hard parts (especially
the shared database, which is always the hardest part). The meta-skill is sequencing: extract by *risk
and value*, and know when *not* to modernize at all.

## Going Deeper

**Why big-bang rewrites usually fail.** The rewrite is seductive ("the old code is a mess; a clean start
will be faster") and usually a disaster, for compounding reasons:
- **The moving target.** The business doesn't freeze while you rewrite — new features keep landing in the
  old system, so you're rebuilding a system that's *still changing*, and you have to catch up to a line
  that keeps moving.
- **The second-system effect** (Brooks): the replacement tends to become *over-engineered* — the team
  crams in every capability and generalization the first system lacked, producing a bloated system that's
  late and complex.
- **Lost knowledge.** The old system's ugliness includes years of accumulated bug fixes and edge-case
  handling — the weird tax rule, the one customer's special case, the race condition someone fixed at 3am.
  A rewrite silently *discards* all of that hard-won correctness and rediscovers the same bugs in
  production.
- **No value for a long, risky stretch.** For months (or years) the business gets *nothing* new while the
  team rebuilds what already existed — and if the project is cancelled midway (many are), it was pure
  loss.

This is why "sacrificial architecture" and rewrites are last resorts, and incremental modernization is
the default.

**The strangler fig pattern.** [Named by Fowler](https://martinfowler.com/bliki/StranglerFigApplication.html),
the pattern: put a **facade** (a router/proxy) in front of the legacy system so all traffic flows through
a point you control. Then repeatedly: pick one capability, build its new implementation, **redirect that
route** from the facade to the new implementation, verify it works, and leave the rest on the legacy
system. Over time more and more routes point to new code, the legacy system handles less and less, and
eventually it's "strangled" — starved of traffic — and can be deleted. The virtues: the system *never
stops running*, value ships continuously (each migrated capability is a delivered improvement), risk is
small and incremental (one route at a time), and each step is **reversible** (if the new route
misbehaves, point the facade back at the legacy one). It's the incremental-change philosophy of Lesson 31
applied to replacing a whole system.

**Branch by abstraction and the anti-corruption layer.** Two enabling tools:
- **[Branch by abstraction](https://martinfowler.com/bliki/BranchByAbstraction.html):** when you can't
  put a facade *around* the system but need to replace something *inside* it, introduce an **abstraction**
  (an interface) over the thing being replaced, route existing callers through it, build the new
  implementation behind the same abstraction, switch over, and delete the old — all on the mainline,
  without a long-lived branch. It's the strangler idea applied *internally*.
- **Anti-corruption layer** (Lesson 7): as you build new services alongside the legacy system, the
  legacy's often-messy data model will try to leak into the clean new one. An **ACL** is a translation
  layer that maps between the legacy model and the new model, so the new system speaks its *own*
  language and isn't corrupted by the old one's quirks. Essential when the new and old must coexist and
  communicate for a long migration.

{: .warning }
> **The database is the hard part — carving a service out of a monolith**
> Extracting a capability from a monolith, the code is the easy part; the <strong>shared database</strong>
> is where it gets genuinely hard, because the data for "your" capability is tangled with everything else
> (foreign keys, joins, shared tables, transactions that span concerns). You can't just move the service
> and leave its data behind. Approaches, in rough order:
> - <strong>Find the seam in the data</strong> first: which tables/columns truly belong to this
>   capability? Often the boundaries you drew (Lesson 6) don't match the database's coupling, and that
>   mismatch <em>is</em> the difficulty.
> - <strong>Split the schema before the service</strong> where possible — separate the capability's
>   tables, break the cross-capability foreign keys (replace joins with API calls or data duplication),
>   so the data is decoupled even while the code still lives together.
> - <strong>Give up the cross-capability transaction</strong> — once the data is split you've lost the
>   shared ACID transaction (Lesson 17) and need a saga / eventual consistency, exactly as when splitting
>   any service. This is usually the real cost of the extraction, and it's a business conversation, not
>   just a technical one.
> - <strong>Parallel run and reconcile</strong> (below) to be sure the new data path matches the old
>   before you cut over.
> The lesson: <em>"just pull the service out"</em> underestimates the database every time — the data
> coupling, not the code, is what makes monolith decomposition hard.

**Parallel run, and sequencing by risk and value.** For a risky migration (especially anything touching
money or critical correctness), a **parallel run** de-risks the cutover: run the old and new
implementations *side by side* on the same real inputs, compare their outputs, and only trust the new one
once it matches the old for long enough. It catches the discrepancies (the lost edge cases!) *before*
they hit customers. And the overall **sequencing** is an architectural judgment: migrate capabilities in
an order that balances **risk** (do a low-risk, well-understood capability first to build the muscle and
the facade infrastructure; save the terrifying core for when you're practiced) and **value** (migrate the
thing that unblocks the most benefit — the bottleneck, the part changing most often — earlier). Finally,
the wisest modernization skill is knowing **when *not* to modernize**: a stable, rarely-changed, working
legacy component that no one needs to touch may be best *left alone* — modernizing it is cost with no
return. Spend the modernization budget where change is actually needed.

---

## Lab — Design Exercise

**The situation:** You've inherited a legacy monolith that, among many other things, handles
**notifications** — it sends order confirmations, shipping updates, and marketing emails, with the
notification logic scattered through the codebase and reading from (and writing to) the monolith's single
shared database (a `notifications` table, plus reads of `users`, `orders`, and `preferences` tables via
joins). The business wants notifications extracted into its own service (to add SMS/push, and because the
notification code changes often and destabilizes unrelated deploys).

**Plan the strangler-fig extraction** of the notifications capability: the **sequence** of steps, the
**seam** where you insert the redirect, and — the hard part — how you'd handle the **shared database**.
Note where you'd use a **parallel run**.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is a genuinely incremental sequence, a real seam, and (above all) a credible plan for the
shared data. A strong answer:
<br><br>
<strong>1. Find the seam.</strong> The natural seam for notifications is the <em>point where the rest of
the system asks for a notification to be sent</em>. Introduce an abstraction — an internal
<code>Notifier.send(event)</code> interface (branch by abstraction) — and route <em>all</em> the
scattered "send an email" call sites through it. Right now its implementation is still the old in-monolith
code, but now there's a single, controllable seam: one place that decides how notifications are sent.
(This alone is valuable — it consolidates the scattered logic.)
<br><br>
<strong>2. Stand up the new service behind the seam (strangler facade).</strong> Build a new Notifications
service. Change the <code>Notifier</code> implementation to publish an event / call the new service
instead of running the old code — the seam is where you redirect. Start with the <em>lowest-risk,
highest-churn</em> notification type first (e.g. shipping updates), not order confirmations (which are
money-adjacent and higher-risk) — sequence by risk and value. Migrate one notification type at a time,
leaving the rest on the old path, so it's incremental and reversible (flip the seam back if the new
service misbehaves).
<br><br>
<strong>3. Handle the shared database — the hard part.</strong> The notification code currently reads
<code>users</code>, <code>orders</code>, <code>preferences</code> via joins and writes a
<code>notifications</code> table. The new service can't reach into the monolith's DB (that would just
recreate the coupling — a distributed monolith). So:
<ul>
<li><em>Own its own data:</em> the <code>notifications</code> table (the send log, templates, delivery
status) clearly belongs to the new service — move it to the service's own database.</li>
<li><em>Break the joins:</em> the reads of <code>users</code>/<code>orders</code>/<code>preferences</code>
must become <em>data passed in</em> or <em>API calls</em>, not joins. Best: when the monolith emits the
triggering event ("order shipped"), <em>include</em> the data the notification needs (recipient, contact
info, order summary) in the event (event-carried state transfer, Lesson 12), so the service doesn't need
to reach back at all. Where it must look something up (e.g. current notification preferences), call an
API — and put an <strong>anti-corruption layer</strong> in the new service so the monolith's messy
<code>preferences</code> model doesn't leak into the clean new one.</li>
<li><em>Give up the shared transaction:</em> once the data is split, "record the order <em>and</em> the
notification" is no longer one ACID transaction — use the outbox/event + idempotency approach (Lesson 17)
so a notification isn't lost or double-sent.</li>
</ul>
<strong>4. Parallel run before trusting it.</strong> For each migrated notification type — especially the
money-adjacent ones (order confirmations, payment receipts) — run old and new <em>side by side</em> on the
same real triggers, comparing what each <em>would</em> send (recipient, content, timing), and only cut the
old path off once the new service matches for long enough. This catches the lost edge cases (the special
template for one region, the suppression rule for opted-out users) <em>before</em> customers get wrong or
duplicate emails.
<br><br>
<strong>5. Strangle and delete.</strong> Once every notification type routes to the new service and the
parallel run is clean, remove the old in-monolith notification code and its now-unused DB coupling. The
capability is fully extracted, the monolith is smaller, and notifications can now deploy independently and
add SMS/push.
<br><br>
The mature answer emphasizes: the <em>code</em> extraction (the seam, the facade) is the easy, mechanical
part; the <em>data</em> — breaking the joins, deciding what the events carry, giving up the shared
transaction, and parallel-running to preserve the accumulated edge cases — is the real work and the real
risk. And it sequenced by risk/value (low-risk type first) and kept every step reversible — never a
big-bang cutover.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Strangler fig application | Martin Fowler — <https://martinfowler.com/bliki/StranglerFigApplication.html> |
| Branch by abstraction | <https://martinfowler.com/bliki/BranchByAbstraction.html> |
| Monolith to Microservices | Sam Newman — <https://samnewman.io/books/monolith-to-microservices/> |
| Working Effectively with Legacy Code (seams) | Michael Feathers |
| Anti-corruption layer | <https://learn.microsoft.com/azure/architecture/patterns/anti-corruption-layer> |
| Second-system effect | <https://en.wikipedia.org/wiki/Second-system_effect> |

---

## Checkpoint

**Q1.** Why do big-bang rewrites usually fail, and how does the strangler fig pattern avoid those
failure modes?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Why big-bang rewrites usually fail:</strong>
<ul>
<li><em>The moving target</em> — the business keeps adding features to the old system while you rewrite,
so you're chasing a line that keeps moving and never catch up.</li>
<li><em>The second-system effect</em> — the from-scratch replacement tends to become over-engineered,
bloated with every generalization and feature the first system lacked, making it late and complex.</li>
<li><em>Lost knowledge</em> — the old system's messiness includes years of accumulated bug fixes and
edge-case handling; a rewrite silently discards all that hard-won correctness and rediscovers the same
bugs in production.</li>
<li><em>No value for a long, risky stretch</em> — the business gets nothing new for months/years while
the team rebuilds what already existed, and if the project is cancelled midway (common), it's pure
loss.</li>
</ul>
<strong>How the strangler fig avoids them:</strong> instead of replacing everything at once, you put a
facade/router in front of the legacy system and migrate <em>one capability at a time</em> — build the
new implementation, redirect that route, verify, move on — until the old system is starved and deleted.
This defuses each failure mode: the <em>system keeps running and shipping value the whole time</em> (no
long dead stretch); risk is taken in <em>small, reversible increments</em> (one route, revertible by
pointing the facade back); there's <em>no second-system bloat</em> because you migrate capability by
capability rather than reimagining the whole thing; and because each capability is migrated and
<em>verified</em> (parallel run) individually, the accumulated edge cases are preserved rather than
discarded wholesale. It's incremental, guided, reversible change (Lesson 31) applied to replacing a whole
system — which is why it's the default and the rewrite is a last resort.
</details>

**Q2.** When carving a service out of a monolith, why is the shared database the hard part, and what does
a parallel run buy you?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Why the database is the hard part:</strong> extracting the <em>code</em> for a capability is
mostly mechanical, but the capability's <em>data</em> is tangled with the rest of the monolith — its
tables are joined to other tables, share foreign keys, and participate in transactions that span multiple
concerns. You can't just move the service and leave its data reachable by everything else, and you can't
let the new service reach back into the monolith's database (that recreates the coupling — a distributed
monolith). So you have to actually <em>split the data</em>: identify which tables/columns truly belong to
the capability (often the boundary you want doesn't match the database's coupling, and that mismatch is
the difficulty), break the cross-capability joins (replace them with data passed in events or with API
calls + an anti-corruption layer), and — the big one — <em>give up the shared ACID transaction</em> that
used to keep the capability consistent with the rest, replacing it with a saga / eventual consistency and
idempotency (Lesson 17). That last part is usually the real cost of the extraction and is a business
conversation about acceptable consistency, not just a technical refactor. "Just pull the service out"
underestimates the data coupling every time.
<br><br>
<strong>What a parallel run buys:</strong> for a risky migration (especially money/correctness-critical
paths), a parallel run means running the old and new implementations <em>side by side on the same real
inputs</em> and comparing their outputs, trusting the new one only once it matches the old for long
enough. Its value is catching <em>discrepancies before customers do</em> — precisely the lost edge cases
(the special-case rule, the regional template, the suppression logic) that a rewrite/extraction tends to
drop. Instead of discovering in production that the new path sends wrong or duplicate results, you see the
mismatch against the proven old behavior and fix it first. It de-risks the cutover by validating the new
implementation against the accumulated correctness of the old one, on real traffic, before switching over.
</details>

---

## Homework

Identify a legacy system or a monolith module in your world that people talk about rewriting. First, apply
the honest test: would a rewrite hit the four failure modes (moving target, second-system bloat, lost
edge cases, long value drought)? Then sketch a strangler-fig alternative: what facade/seam would you
insert, which capability would you extract *first* (and why — risk and value), and — the hard part — how
would you untangle its data from the shared database (what belongs to it, which joins must break, which
shared transaction you'd have to give up)? Where would you parallel-run? Finally, be honest about whether
part of that system should be modernized *at all*, or left alone as stable, working, rarely-touched code
where change would be cost with no return.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
A strong response resists the rewrite instinct and thinks incrementally.
<br><br>
<strong>Applies the rewrite honesty test.</strong> The valuable move is naming which of the four failure
modes a proposed rewrite would actually hit — usually all four, and especially the <em>lost edge cases</em>
(the accumulated correctness no one has fully catalogued) and the <em>value drought</em> (months with
nothing shipped). Recognizing that "the code is a mess" is not sufficient justification for a rewrite —
that the mess often <em>encodes</em> hard-won correctness — is the maturity.
<br><br>
<strong>Sketches a credible incremental extraction.</strong> A good answer picks a real seam (the point
where the rest of the system invokes the capability), chooses a <em>first</em> capability to extract by
risk and value (low-risk/high-churn first to build the muscle and the facade; save the terrifying core
for later), and — most importantly — confronts the <strong>data</strong>: what data truly belongs to the
capability, which cross-cutting joins must be broken (via events carrying state or API calls + an ACL),
and which shared transaction is lost (requiring a saga / eventual consistency, a business conversation).
Getting specific about the database is what separates a real plan from wishful thinking.
<br><br>
<strong>Places the parallel run and knows when to leave things alone.</strong> Identifying where a
parallel run de-risks the cutover (the money/correctness-critical paths) shows the safety discipline. And
the wisest point: honestly deciding that some stable, rarely-changed component should be <em>left
alone</em> — modernizing it would be cost with no return — because the modernization budget belongs where
change is actually needed. The takeaway a good answer reaches: you'll inherit far more systems than you
design from scratch; the core skill is changing a running system incrementally (strangler fig + branch by
abstraction + ACL + parallel run) rather than betting the business on a big-bang rewrite; the data
coupling — not the code — is what makes it hard; and knowing when <em>not</em> to modernize is as much a
part of the judgment as knowing how.
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 33 — Build vs Buy & Technology Selection →](lesson-33-build-vs-buy){: .btn .btn-primary }
