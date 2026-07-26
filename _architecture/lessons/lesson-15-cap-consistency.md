---
title: "Lesson 15 — CAP, PACELC & Consistency Models"
nav_order: 2
parent: "Phase 4: Distributed Systems"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 15: CAP, PACELC & Consistency Models

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

Once data lives on more than one machine, you hit the deepest trade-off in distributed
systems: when the network splits, you cannot have *both* perfect consistency and full
availability. The **CAP theorem**[°](#w-cap-theorem) (Eric Brewer) states it precisely — and it's constantly
misquoted. It is *not* "pick 2 of 3 always." Partitions are not something you choose; the
network *will* split sometimes (fallacy 1, Lesson 14). CAP is about what you do *when it
does*: **during a partition, you must choose between Consistency and Availability.**

CAP is widely quoted and usually misapplied, so start with the constraint that
makes it precise: **CAP only matters during a partition.**

Picture two nodes, A and B, with the network between them cut. A write arrives
at A. B cannot hear about it. Now a read arrives at B, and the system must
choose.

**Choose consistency** and B refuses or blocks the read until it can confirm it
is current — or it returns an error. The answer is correct, and the system is
*unavailable*.

**Choose availability** and B answers from what it has, which is the old value.
The system stays up, and the answer is *stale*. This is what "eventual
consistency" means in practice.

You cannot have both while partitioned; that is the whole theorem. What it does
**not** say is that you must pick two of three properties for your system
forever. When the network is healthy you get consistency and availability both,
and the choice is made **per piece of data, based on what that data needs** — a
bank balance and a "likes" counter should not answer this question the same
way.

The choice is per-data, not per-system: a bank balance during a partition should refuse
rather than risk a wrong number (choose **C**), while a social "like" count should keep
working and reconcile later (choose **A**). Getting this right means matching the
consistency choice to *the business cost of being wrong*.

## Going Deeper

**PACELC — the part CAP leaves out.** CAP only describes the partition case, which is rare.
**PACELC**[°](#w-pacelc) (Daniel Abadi) completes it: *if Partition, choose Availability or Consistency;
**Else** (the normal, no-partition case), choose Latency or Consistency.* The "else" is the
insight — even when the network is healthy, you're *constantly* trading consistency for
latency: to guarantee a read sees the latest write, the system must coordinate across
replicas (wait for a quorum, or read from the leader), which adds latency; to be fast, it
reads from the nearest replica, which may be slightly stale. So consistency-vs-latency is a
choice you make on *every* request, not just during the rare partition. Most "eventually
consistent" databases chose A-during-partition *and* L-in-normal-operation (PA/EL); most
strongly-consistent ones chose C/C (PC/EC). PACELC is the more useful framing day to day.

{: .note }
> **The consistency spectrum — it's not binary**
> "Consistent vs eventually consistent" is a simplification. There's a spectrum, strongest
> to weakest:
> - <strong>Linearizable / strong</strong> — behaves as if one single, always-current copy;
>   every read sees the latest write. Strongest, most coordination, highest latency, lowest
>   availability under partition.
> - <strong>Sequential / causal</strong> — weaker but preserves useful orderings (e.g., you
>   always see a reply <em>after</em> the message it replies to). Causal consistency is a
>   sweet spot for many collaborative apps.
> - <strong>Read-your-writes / monotonic</strong> — session guarantees: you always see your
>   <em>own</em> latest write, and never see time go backwards, even if others' writes are
>   delayed. Cheap and often exactly enough for good UX.
> - <strong>Eventual</strong> — replicas may disagree for a while but converge if writes
>   stop. Weakest, cheapest, most available and lowest-latency.
>
> The architect picks the <em>weakest model that still meets the business need</em> — because
> weaker is cheaper, faster, and more available. Over-choosing strong consistency
> "to be safe" needlessly sacrifices latency and availability.

**What "eventual consistency" actually means for a user.** It's not a hand-wave — it has
concrete, visible effects. It means there's a window where different users (or the same user
on different requests) can see different values: you post a comment and a friend doesn't see
it for a second; you change a setting and a stale read shows the old one; two people edit and
one edit briefly "wins" before reconciliation. Whether that's fine or catastrophic depends
entirely on the data. The architect's job is to (a) know the window exists, (b) decide if the
business can tolerate it for *this* data, and (c) apply the cheapest guarantee that makes the
UX acceptable (often "read-your-writes" — you see your own change immediately even if others
see it slightly later — which is far cheaper than global strong consistency).

**Match the model to the cost of being wrong.** This is the decision procedure. For each
piece of data, ask: *what's the business cost if a read returns a stale or divergent value?*
- Catastrophic (money moved twice, oversold inventory, a wrong medical record) → strong
  consistency, accept the latency/availability cost.
- Annoying but harmless (a like count off by one, a slightly stale "last seen", a
  recommendation not yet updated) → **eventual consistency**[°](#w-eventual-consistency), keep the speed and availability.
- In between → a session guarantee (read-your-writes) or causal consistency.

The same system holds data at *different* points on the spectrum: an e-commerce app wants
strong consistency on payment and inventory-decrement, but is perfectly happy with eventual
consistency on product review counts and recommendations. Choosing one global consistency
level for everything is the mistake — you either over-pay (strong everywhere, slow and
fragile) or under-protect (eventual everywhere, wrong where it matters).

**This connects to everything downstream.** The consistency choice drives data architecture
(Lesson 24 — the moment you split data across services, you're choosing eventual consistency
between them), the communication style (async messaging implies eventual consistency, Lesson
16), the need for sagas (Lesson 17 — a cross-service transaction *is* eventual consistency
with compensations), and caching (Lesson 21 — a cache is a deliberate staleness/consistency
trade). CAP/PACELC isn't a theory lesson; it's the lens for every "where does the data live
and how fresh is it?" decision in the rest of the track.

---

## Lab — Design Exercise

**The situation:** You're designing the data for an e-commerce platform that runs across
multiple data centers (so partitions and replica lag are real). For each of the following
data items, choose a consistency model (strong / causal / read-your-writes / eventual) and
justify it by the **business cost of a stale or divergent read** — and, where relevant, say
what you'd do *during a partition* (favor C or A):

1. A customer's **account balance / store credit** (used to pay for orders).
2. **Product inventory count** (the "3 left in stock" number, and the decrement on purchase).
3. A **product's review count and average rating** shown on the listing page.
4. A user's **shopping cart contents** (as they add/remove items).
5. A product's **"last viewed by you" / recently-viewed** list.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is matching each item to the weakest model that meets the business need, driven by
the cost of being wrong. A strong answer:
<br><br>
<strong>1. Account balance / store credit → strong consistency; favor C during a
partition.</strong> Cost of being wrong is <em>catastrophic</em>: a stale/divergent balance
means a customer could spend the same credit twice, or be charged incorrectly — real money,
real disputes. This is exactly the case where you accept the latency and reduced availability
of strong consistency. During a partition, <em>refuse</em> or block the operation rather than
risk a double-spend (choose C over A) — better to tell the customer "try again in a moment"
than to permit an inconsistent financial state.
<br><br>
<strong>2. Inventory count + decrement on purchase → strong consistency on the decrement
(the "can I sell this?" check), even if the displayed number is looser.</strong> Two aspects.
The <em>authoritative decrement</em> at purchase time must be strongly consistent / serialized
(often via a conditional/atomic operation or a reservation) — otherwise two customers buy the
last unit and you <em>oversell</em>, which has a real cost (cancellations, angry customers,
sometimes legal/SLA issues). The <em>displayed</em> "3 left" number browsing customers see can
be slightly stale (eventual) — showing "3 left" when it's really 2 is a minor UX imperfection.
So: eventual for the display, strong for the actual sell decision. During a partition, the
sell path should favor C (don't sell what you can't confirm is in stock); the display can stay
available with stale data.
<br><br>
<strong>3. Review count / average rating → eventual consistency.</strong> Cost of being wrong
is <em>negligible</em>: if the review count is 1,204 vs 1,205 for a few seconds, or the average
is 4.31 vs 4.32, no one is harmed and no one notices. This is the textbook case for keeping
availability and low latency — never sacrifice speed or uptime to make a rating count
instantly precise. During a partition, stay available with the last-known value (favor A).
<br><br>
<strong>4. Shopping cart → read-your-writes (a session guarantee), leaning eventual across
devices.</strong> The key requirement is that <em>the user always sees their own latest
change</em> — if they add an item and it doesn't appear, that's a broken, trust-destroying
experience. But this is cheaper than global strong consistency: you only need <em>their</em>
writes to be visible to <em>them</em> immediately (read-your-writes), not for the cart to be
globally linearizable. Across devices some lag is tolerable. It doesn't need the expensive
strong consistency of the balance, but it needs more than raw eventual — the session guarantee
is the right, cheaper middle. During a partition, favor availability for viewing the cart, but
be careful at checkout (which hits inventory/payment, the strong-consistency items).
<br><br>
<strong>5. Recently-viewed list → eventual consistency (weakest).</strong> Cost of being wrong
is essentially zero — a slightly stale or momentarily-out-of-order recently-viewed list is
completely harmless. Cheapest, most available, lowest latency; favor A always.
<br><br>
<strong>The meta-lesson:</strong> the <em>same system</em> spans the whole spectrum — strong
consistency where money and overselling are at stake (balance, the sell decision), a session
guarantee where the user's own experience depends on seeing their change (cart), and eventual
consistency where divergence is harmless (ratings, recently-viewed). The two mistakes are
symmetric: making <em>everything</em> strongly consistent (needlessly slow, fragile, and
partition-intolerant — you'd take the whole store down to keep a rating count exact) or
making <em>everything</em> eventually consistent (fast and available but you double-spend
credit and oversell stock). The architect chooses <em>per data item</em>, by the cost of a
wrong read, and reaches for the <em>weakest model that still meets the need</em> — because
weaker is cheaper, faster, and more available, and strong consistency is a cost you pay only
where the business genuinely requires it.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| CAP theorem (stated correctly) | <https://en.wikipedia.org/wiki/CAP_theorem> ; Brewer, "CAP Twelve Years Later" |
| PACELC | <https://en.wikipedia.org/wiki/PACELC_theorem> ; Abadi's original post |
| Consistency models spectrum | *Designing Data-Intensive Applications*, Kleppmann (Ch. 5, 9) |
| Eventual consistency & session guarantees | Werner Vogels, "Eventually Consistent" — <https://www.allthingsdistributed.com/2008/12/eventually_consistent.html> |

---

## Checkpoint

**Q1.** State the CAP theorem correctly (not "pick 2 of 3"), and explain why the choice only
arises during a partition. Then give a piece of data that should choose C during a partition
and one that should choose A, with the reason.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
Correct statement: <strong>when a network partition occurs</strong> (some nodes can't
communicate with others, though each is still running), a distributed system must choose
between <strong>Consistency</strong> (every read returns the most recent write, or an error)
and <strong>Availability</strong> (every request gets a non-error response, possibly stale) —
you cannot provide both while partitioned. It is <em>not</em> "pick 2 of the 3 (C, A, P) at
all times." That misreading treats P (partition tolerance) as an optional choice you trade
against the others, but partitions are <em>not something you choose</em> — the network
<em>will</em> split sometimes (it's a fact of distributed systems, fallacy 1), so any real
distributed system must tolerate partitions. The genuine choice is only about what to do
<em>when</em> a partition happens: C or A.
<br><br>
Why the choice only arises during a partition: when the network is healthy and all nodes can
communicate, a system can be <em>both</em> consistent and available — it coordinates across
nodes to keep them in agreement and still answers every request. There's no forced trade
(though PACELC notes you still trade consistency vs <em>latency</em> even then). The
<em>impossibility</em> of having both only bites when nodes are split: now a node receiving a
read either answers from its possibly-stale local state (available but maybe inconsistent) or
refuses/blocks until it can confirm it's current — which it can't, because it's partitioned
(consistent but unavailable). No third option exists while the split persists.
<br><br>
<strong>Should choose C during a partition:</strong> a <em>bank account balance</em> (or store
credit used to pay). If a partitioned node can't confirm the balance is current, it must refuse
the operation rather than risk letting the customer spend money that isn't there or spend the
same credit twice — a wrong answer here moves real money and causes real harm, so
unavailability ("try again shortly") is far better than inconsistency. <strong>Should choose A
during a partition:</strong> a <em>product's review count / average rating</em> (or a "like"
count, or recently-viewed list). If the count is briefly stale or off by one, nobody is harmed
and nobody notices, so the system should stay available and answer with the last-known value,
reconciling once the partition heals — sacrificing uptime to keep a rating count exact would be
a terrible trade. The principle: choose C where a stale/divergent read is <em>costly or
dangerous</em>, choose A where it's <em>harmless</em> — per data item, by the cost of being
wrong.
</details>

**Q2.** What does PACELC add to CAP, and why is the "Else" clause more relevant to everyday
design than the partition clause? Illustrate with the consistency-vs-latency choice on a
normal (non-partitioned) read.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
PACELC extends CAP by describing the <em>normal</em> case CAP ignores. It reads: <strong>if
Partition, choose Availability or Consistency; Else (no partition), choose Latency or
Consistency.</strong> CAP only tells you what happens during the rare partition; PACELC adds
that <em>even when the network is perfectly healthy</em>, a distributed system is still making
a trade — between consistency and <em>latency</em> — on every request.
<br><br>
The "Else" clause is more relevant to everyday design because <strong>partitions are rare, but
normal operation is constant.</strong> Most of the time your network isn't partitioned, so the
CAP choice (A vs C during a partition) doesn't apply — but the latency-vs-consistency choice
applies to <em>every single read and write</em>, all day, forever. So the "Else" side describes
the trade you're actually living with continuously, while the partition side describes an
exceptional event. That's why PACELC is the more useful daily framing: it names the constant
cost of strong consistency (latency) rather than only the exceptional cost (availability).
<br><br>
Illustration on a normal read: suppose data is replicated across three nodes. To serve a read
with <strong>strong consistency</strong> — guaranteeing it reflects the latest write — the
system must coordinate: read from the leader, or contact a quorum of replicas and wait for them
to agree, so no in-flight write is missed. That coordination costs <em>latency</em> (extra
round-trips, waiting for the slowest replica in the quorum). To serve the read with <strong>low
latency</strong> instead, the system reads from the nearest single replica and returns
immediately — but that replica might be a few milliseconds behind on replication, so the read
could be slightly stale (weaker consistency). No partition is involved at all; the healthy
system is simply choosing, on this read, whether to pay coordination latency for freshness or
return fast with possible staleness. This is why "eventually consistent" databases (which chose
low latency in the Else case — PA/EL) feel fast, and strongly-consistent ones (PC/EC) feel
slower even when nothing is broken. The architect's takeaway: strong consistency isn't only a
partition-time availability cost — it's a <em>latency tax on every normal request</em>, which
is exactly why you should choose the weakest consistency model the data actually needs.
</details>

---

## Homework

For your current system, list the major categories of data and, for each, identify the
consistency model it *actually* has today versus what it *needs* by the cost-of-being-wrong
test. Look specifically for two mismatches: (1) data that's eventually consistent but shouldn't
be (where a stale/divergent read has a real cost you're currently exposed to — an oversell, a
double-spend, a wrong-permissions read), and (2) data that's strongly consistent but doesn't
need to be (where you're paying latency/availability for freshness nobody needs). Then, if your
system is or will be multi-region/replicated, note which data must favor C during a partition
and which can favor A.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise turns CAP/PACELC into an audit of real data. A strong response:
<br><br>
<strong>Classifies data by the cost-of-being-wrong test</strong> rather than by habit, and
looks for the two symmetric mismatches, which is where the value is:
<ul>
<li><strong>Eventually-consistent-but-shouldn't-be</strong> is the dangerous mismatch: data
where the system currently tolerates staleness/divergence but a wrong read actually costs
something — inventory that can be oversold because the decrement isn't serialized, credit/
balance that could be double-spent, a permissions/entitlement check reading a stale replica so
a just-revoked user still has access, a uniqueness constraint that isn't enforced consistently.
Finding one of these is finding a latent correctness bug (or a whole class of them), and the
fix is to strengthen the guarantee <em>for that specific operation</em> (serialize the sell,
read the balance/permission from an authoritative source).</li>
<li><strong>Strongly-consistent-but-needn't-be</strong> is the performance/availability
mismatch: data forced through a strongly-consistent path (a single leader, a distributed
transaction, synchronous replication) where the business would be perfectly happy with eventual
consistency or a session guarantee — often analytics, counts, feeds, recommendations, or
display values. Here you're paying latency and fragility for freshness nobody needs, and the fix
is to <em>relax</em> the guarantee to buy speed and availability.</li>
</ul>
<strong>Assigns partition behavior</strong> where replication/multi-region is real: the honest
finding is usually that only a small set of data genuinely needs C-during-partition (money,
inventory decrement, auth/permissions), while most (feeds, counts, profiles, history) can and
should favor A — and that the system may not currently make this distinction (treating
everything the same). The deeper takeaway is the same as the lab's: consistency is a
<em>per-data</em> choice matched to the cost of being wrong, the goal is the <em>weakest model
that meets the need</em>, and both over- and under-choosing are real, correctable mistakes —
one costs you speed and uptime, the other costs you correctness. A good answer names at least
one concrete instance of each mismatch to act on.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-consistency-c-in-cap"></a>**consistency (C in CAP)** — every read sees the most recent write; all nodes agree on the current value.
- <a id="w-availability-a-in-cap"></a>**availability (A in CAP)** — every request gets a (non-error) response, even if it might be stale.
- <a id="w-partition-p"></a>**partition (P)** — a network split where nodes can't all communicate, though each is still alive.
- <a id="w-cap-theorem"></a>**CAP theorem** — during a partition you must choose consistency *or* availability; you can't have both.
- <a id="w-pacelc"></a>**PACELC** — the fuller rule: on Partition, choose A or C; Else (normal operation), choose Latency or Consistency.
- <a id="w-eventual-consistency"></a>**eventual consistency** — replicas may disagree briefly but converge to the same value if writes stop.
- <a id="w-linearizable-strong-consistency"></a>**linearizable / strong consistency** — the system behaves as if there's a single, up-to-date copy.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 16 — Communication Styles →](lesson-16-communication){: .btn .btn-primary }
