---
title: "Lesson 22 — Scaling Data (Partitioning, Sharding, Replication)"
nav_order: 4
parent: "Phase 5: Data & Scale"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 22: Scaling Data (Partitioning, Sharding, Replication)

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

Most systems eventually hit the point where a single database can't keep up — and this is where
a great many systems actually break, because scaling *data* is far harder than scaling
stateless compute (you can't just run more copies; the data has to live somewhere). There's a
ladder of techniques, each solving a different bottleneck and charging a different tax.

Data scaling is a ladder, and the rule is to climb only as far as you must —
each rung solves a problem and charges a tax.

**1. Vertical — scale up.** A bigger box. It is simple, requires no
architectural change, and should genuinely be tried first. Its limits are a
hard ceiling and a price curve that turns unpleasant near the top.

*You climb when:* you hit the ceiling, or read load alone is too high.

**2. Read replicas.** Copies of the database that serve reads, which scales
reads and improves availability. *The tax:* **replication lag** — replicas are
behind the primary, so reads may be stale, and Lesson 15's consistency question
returns in a new place.

*You climb when:* writes are now the bottleneck, since replicas do nothing for
writes.

**3. Partitioning, or sharding.** Split the data across nodes, which finally
scales **writes** and storage. *The tax is the largest of the three:* the
**shard key decides everything**. A bad key gives you hot spots and queries that
must hit every shard, and cross-shard transactions are essentially gone —
straight back to Lesson 17.

The ladder is worth respecting in order. A great deal of premature sharding
exists in the world because step one was skipped.

The crucial distinctions: **replication**[°](#w-replication) (copies of the *same* data) scales **reads** and
buys availability, but every replica still holds *all* the data and every write must propagate
to all of them — so it does *nothing* for write throughput or storage limits.
**Partitioning/sharding** (each node holds a *different subset*) scales **writes** and storage,
but it's much harder and the **shard key**[°](#w-shard-key) choice makes or breaks it.

## Going Deeper

**Climb the ladder in order — don't skip to sharding.** Sharding is the most powerful and by far
the most painful step (it complicates queries, transactions, and operations permanently), so you
reach for it *last*, only when the earlier rungs are exhausted:
1. **Vertical scaling first** — a bigger machine is the simplest fix and buys a surprising amount;
   modern hardware is large. Ceiling and cost eventually, but try it before distributing.
2. **Read replicas** when *reads* are the bottleneck — route reads to replicas, writes to the
   primary. Also improves availability (a replica can be promoted if the primary fails).
3. **Sharding** only when *writes* or *storage* exceed one node — because that's the one thing
   replicas can't fix (every write still hits the primary, every node still stores everything).

Skipping straight to sharding a system that a bigger box or read replicas would have served is
premature complexity (Lesson 23's "premature scaling is a debt too").

**Replication lag reintroduces consistency questions.** The moment you add read replicas, you've
created eventual consistency (Lesson 15): a write commits on the primary, but the replicas are a
few milliseconds-to-seconds behind, so a read routed to a replica may not reflect a just-made
write. The classic bug: a user updates their profile (write → primary), the page reloads (read →
replica), and they see the *old* value — because the replica hasn't caught up. Mitigations:
**read-your-writes** (route a user's reads to the primary, or to a replica known to be caught up,
for a short window after they write) and being deliberate about which reads *can* tolerate lag
(most) vs which can't. Replication isn't free consistency — it's a consistency trade you must
manage.

{: .warning }
> **The shard key is the most important — and most irreversible — decision**
> When you shard, you split rows across nodes by a <strong>shard key</strong> (e.g., by
> <code>user_id</code>, <code>tenant_id</code>, <code>region</code>). This one choice determines
> whether sharding <em>helps</em> or <em>ruins</em> you, and it's brutally expensive to change
> later (re-sharding means moving huge amounts of data). Two ways a bad key destroys you: (1)
> <strong>**Hot spots**[°](#w-hot-spot)</strong> — if the key distributes load unevenly, one shard gets hammered
> while others idle (sharding by <code>country</code> when 80% of users are in one country; or a
> low-cardinality/sequential key that funnels writes to one shard), so you've added complexity
> <em>without</em> gaining balanced capacity. (2) <strong>**Cross-shard queries**[°](#w-cross-shard-query)</strong> — if your
> common queries need data from many shards (you sharded by <code>user_id</code> but constantly
> query "all orders in this date range across all users"), every such query must fan out to all
> shards and combine results — slow, complex, and it defeats the point. A <em>good</em> shard key
> distributes load evenly <em>and</em> keeps the data that's queried together on the same shard.
> Choose it by studying your access pattern (Lesson 19) — and know that you're choosing a one-way
> door.
> <br><br>
> And note: <strong>cross-shard transactions are essentially gone</strong> — you can't ACID across
> shards any more than across services, so sharding pushes you toward the same sagas/eventual-
> consistency tools as microservices (Lesson 17). Sharding by an aggregate boundary (all of one
> user's data on one shard) keeps most transactions single-shard, which is a big reason to align
> the shard key with the aggregate.

**Denormalize for the read path — the scaling trade-off.** Normalization (no duplicated data, join
to assemble) is a write-time and correctness virtue, but at scale joins get expensive and
cross-shard joins get impossible. So scaling often pushes you to **denormalize**: duplicate data so
the read path doesn't need joins (store a copy of the product name on the order line rather than
joining to products; precompute an aggregate rather than computing it live). The trade: faster,
shard-friendly reads in exchange for duplicated data you must keep consistent (usually via events —
Lesson 12/17) and more storage. This is the same read-optimized-view idea as CQRS (Lesson 20). At
scale, "normalize for correctness, denormalize for the read path you can't afford to join" is a
constant, conscious trade-off.

**Know your numbers before you scale (capacity thinking).** Before climbing the ladder, do the
back-of-the-envelope math (Lesson 23): what's your actual read and write throughput, data size,
and growth rate? What can one modern node handle? A great deal of "we need to shard" turns out to
be "we're at 5% of what one properly-indexed Postgres on a decent box can do, and the real problem
is a missing index or an N+1." Measure the real bottleneck (reads? writes? storage? a specific
query?) and target *that*, rather than reflexively sharding. Scaling the wrong dimension is wasted
effort; scaling before you need to is premature complexity.

---

## Lab — Design Exercise

**The situation:** A multi-tenant B2B SaaS runs on a single PostgreSQL instance. It's a growing
success: thousands of tenant companies, each with many users, generating orders and activity. The
database is now struggling — and investigation shows it's hitting the *write* ceiling (the volume
of inserts/updates across all tenants exceeds what one primary can handle), not just read load.

**Walk the scaling ladder for this system.** Say what you'd try at each rung (vertical → replicas →
shard) and why replicas alone won't solve *this* (write-bound) problem. Then design the sharding:
choose a **shard key**, and defend it against both the hot-spot risk and the cross-shard-query
risk. Explain what happens to transactions after sharding, and name what you'd verify *before*
sharding at all.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is climbing the ladder deliberately and choosing a shard key by the access pattern. A
strong answer:
<br><br>
<strong>Before anything: verify it's really a scaling problem, not a defect.</strong> Confirm the
write ceiling is genuine — proper indexes (over-indexing also slows writes), no pathological write
amplification, batching where possible, and that the numbers actually exceed a well-tuned single
node (capacity math). A surprising amount of "we've hit the write ceiling" is a fixable
inefficiency. Assume here it's real.
<br><br>
<strong>Rung 1 — Vertical scaling.</strong> First, a bigger box (more CPU/IO/RAM, faster disk).
It's the simplest step and buys real headroom; modern hardware handles very high write rates. If a
scale-up gets you enough runway for now, take it — it defers the painful sharding. But vertical has
a ceiling, and the problem is described as genuinely exceeding one node's write capacity, so assume
we're near it.
<br><br>
<strong>Rung 2 — Read replicas: won't solve <em>this</em> problem.</strong> This is the key
reasoning. Read replicas scale <em>reads</em> and improve availability, but <em>every write still
goes to the single primary</em>, and each write must additionally propagate to all replicas — so
replicas do <em>nothing</em> for write throughput (they arguably add a little write overhead). Our
bottleneck is <em>writes</em>, so replicas are the wrong tool here (they'd help if reads were the
problem). We can still add them for read scaling/availability, but they don't address the write
ceiling — which is exactly why we must go to sharding.
<br><br>
<strong>Rung 3 — Shard, because only sharding scales writes.</strong> Splitting the data across
multiple primaries, each taking a subset of the writes, is the one technique that raises write
throughput past a single node.
<br><br>
<strong>The shard key: <code>tenant_id</code>.</strong> This is a strong choice for a multi-tenant
SaaS, and defensible against both risks:
<ul>
<li><em>Hot-spot risk:</em> tenant_id has high cardinality (thousands of tenants) and, with a good
hash/range distribution, spreads writes across shards fairly evenly — no single value dominates
the way <code>country</code> or a sequential id would. <em>Caveat and mitigation:</em> a few
<strong>whale tenants</strong> (a huge customer generating outsized load) could hot-spot a shard;
mitigate by placing very large tenants on their own shard(s) or using a distribution that accounts
for tenant size — but for the many normal tenants, tenant_id distributes well.</li>
<li><em>Cross-shard-query risk:</em> in a B2B multi-tenant system, the <em>overwhelming majority of
queries are scoped to a single tenant</em> ("show <em>this company's</em> orders/users/activity") —
because tenants are isolated from each other by nature. Sharding by tenant_id therefore keeps
almost every query <em>single-shard</em> (all of a tenant's data lives on one shard), avoiding
fan-out. The rare cross-tenant queries (platform-wide analytics/reporting) are handled separately —
via a data warehouse / analytics pipeline fed by the shards — rather than as live cross-shard
queries. This is the big win: the access pattern (tenant-scoped) aligns perfectly with the shard
key.</li>
</ul>
<strong>Transactions after sharding:</strong> transactions <em>within</em> a tenant stay on one
shard, so they remain normal ACID transactions — which is most of what the app does (a tenant's
order + line items + activity are all on that tenant's shard). Only operations spanning
<em>multiple tenants</em> lose cross-node ACID, and those are rare and mostly analytical. So
sharding by tenant_id has the property that it aligns with the <em>aggregate/consistency boundary</em>
(the tenant), keeping the vast majority of transactions single-shard and dodging the sagas/eventual-
consistency tax (Lesson 17) for normal operations. (Had we sharded by, say, <code>order_date</code>,
a single tenant's data would scatter across shards and every tenant query would go cross-shard — a
disaster; the contrast shows why tenant_id is right.)
<br><br>
<strong>What to verify before sharding at all:</strong> (1) that it's genuinely write-bound and not
a fixable inefficiency (indexes, N+1, write amplification) or something a bigger box solves; (2)
the capacity math (are we really past one node, and how much growth does sharding need to absorb?);
(3) that the tenant-scoped access pattern truly holds (so single-shard queries dominate) — study the
actual query workload; and (4) the operational readiness for sharding (it permanently complicates
schema changes, backups, and queries — it's a one-way door you should be sure about). The
through-line: climb the ladder in order (vertical, then replicas <em>only if reads were the
problem</em>, then shard because only sharding scales writes), choose the shard key by the access
pattern so it distributes evenly and keeps together-queried data together (tenant_id, aligning with
the tenant aggregate/consistency boundary), and treat sharding as the last, most irreversible
rung — reached deliberately, after verifying the cheaper rungs won't do.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Partitioning & sharding | *Designing Data-Intensive Applications*, Kleppmann (Ch. 6) |
| Replication & replication lag | *Designing Data-Intensive Applications*, Kleppmann (Ch. 5) |
| Choosing a shard/partition key | <https://learn.microsoft.com/azure/architecture/best-practices/data-partitioning> |
| Read-your-writes & session consistency | Werner Vogels, "Eventually Consistent" |
| Vertical vs horizontal scaling | <https://en.wikipedia.org/wiki/Scalability#Horizontal_(scale_out)_and_vertical_scaling_(scale_up)> |

---

## Checkpoint

**Q1.** Explain the difference between replication and sharding in terms of *which bottleneck each
solves*. Why do read replicas do nothing for a write-bound system, and what's the consistency tax
replicas introduce?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Replication</strong> keeps <em>copies of the same complete data</em> on multiple nodes.
It solves the <strong>read</strong> bottleneck (spread read queries across many replicas) and
improves <strong>availability</strong> (a replica can be promoted if the primary fails). But every
replica holds <em>all</em> the data, and every write must go to the primary and then propagate to
<em>every</em> replica. <strong>Sharding (partitioning)</strong> splits the data so each node holds
a <em>different subset</em>. It solves the <strong>write</strong> and <strong>storage</strong>
bottlenecks: writes for different subsets go to different nodes (so total write throughput scales
with the number of shards), and each node stores only its slice (so total storage scales too).
<br><br>
<strong>Why read replicas do nothing for a write-bound system:</strong> the write bottleneck is
"one primary can't absorb the volume of writes." Adding read replicas doesn't move any writes off
the primary — <em>every</em> write still goes to the single primary first (that's what keeps the
replicas' copies correct), and then the write is <em>additionally</em> propagated to each replica.
So replicas not only fail to relieve write load, they add a little (the primary now also ships each
write out to N replicas). They help only if the bottleneck is <em>reads</em>. For a write-bound
system, the only technique that helps is sharding — distributing the writes across multiple
primaries — because that's the one thing that actually reduces the write volume any single node
must handle.
<br><br>
<strong>The consistency tax replicas introduce:</strong> <em>replication lag</em>. When a write
commits on the primary, the replicas are momentarily behind (milliseconds to seconds) until the
write propagates — so a read served from a replica may not reflect a just-made write. This
reintroduces eventual consistency (Lesson 15): the classic bug is a user updating a value (write →
primary) and immediately re-reading it (read → replica) and seeing the <em>old</em> value because
the replica hasn't caught up. So replication isn't "free" read scaling — it's read scaling in
exchange for stale reads, which you manage with techniques like <em>read-your-writes</em> (route a
user's reads to the primary or a caught-up replica for a window after they write) and by ensuring
only reads that <em>can</em> tolerate lag go to replicas. The tax is that you now have to reason
about which reads can be stale, a consistency concern the single-node system didn't have.
</details>

**Q2.** Why is the shard key the most important and most irreversible decision in sharding? Explain
how a bad key causes hot spots *and* cross-shard queries, and what makes a good key.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The shard key is the field that decides which shard each row lives on, so it determines
<em>everything</em> about how load and data distribute — and it's <strong>irreversible</strong> in
practice because changing it means re-sharding: physically moving enormous amounts of data across
nodes while the system runs, one of the most expensive and risky operations in data engineering.
You largely live with your first choice, so getting it right up front matters enormously.
<br><br>
<strong>How a bad key causes hot spots:</strong> if the key distributes data/load
<em>unevenly</em>, some shards get far more than their share while others idle. Examples: sharding
by <code>country</code> when most users are in one country funnels most load to one shard; a
low-cardinality key (few distinct values) can't spread across many shards; a sequential/
monotonic key (like an auto-increment id or timestamp) sends all <em>new</em> writes to whichever
shard holds the current range — a write hot spot. In all these, you've paid the full complexity of
sharding but one shard is the bottleneck, so you didn't actually gain balanced capacity.
<br><br>
<strong>How a bad key causes cross-shard queries:</strong> if your common queries need data that
the key has <em>scattered</em> across many shards, every such query must fan out to all shards and
merge the results — slow, complex, and defeating the purpose. Example: sharding by
<code>user_id</code> but constantly running "all orders in this date range across all users" — the
data for that query is spread over every shard, so it's a cross-shard fan-out every time. Worse,
<em>transactions</em> across shards are essentially gone (no cross-shard ACID, like across services
— Lesson 17), so a key that splits data that's changed together forces sagas/eventual consistency
everywhere.
<br><br>
<strong>What makes a good key:</strong> two properties together — (1) it <em>distributes load
evenly</em> (high cardinality, no dominant value, not monotonic — so writes and data spread across
shards fairly), and (2) it keeps <em>data that's queried and changed together on the same
shard</em> (so common queries are single-shard, not fan-outs, and common transactions stay
single-shard ACID). The way to find it is to study the <em>access pattern</em> (Lesson 19): shard
by the dimension your queries and transactions are naturally scoped to. In a multi-tenant SaaS,
<code>tenant_id</code> is often ideal — high cardinality (even distribution), and queries/
transactions are almost always tenant-scoped (so they stay single-shard), aligning the shard key
with the aggregate/consistency boundary. The general principle: a good shard key mirrors how the
data is accessed and kept consistent, so both load and query locality fall out correctly — and
because it's a one-way door, it's worth the analysis to get right before you commit.
</details>

---

## Homework

For your current system (or a system heading for scale), identify the likely *first* data
bottleneck as it grows — reads, writes, storage, or a specific hot query — and which rung of the
scaling ladder addresses it (be honest about whether you're anywhere near needing to shard, or
whether vertical scaling / a missing index / read replicas would serve for a long time). If you can
foresee sharding eventually, do the hard part now: propose a shard key, and stress-test it against
the hot-spot and cross-shard-query risks using your real access pattern — does your query workload
stay single-shard under this key, or would it fan out? Note whether the key aligns with your
transaction/aggregate boundary.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise builds capacity thinking and the shard-key discipline before a crisis forces them. A
strong response:
<br><br>
<strong>Identifies the real first bottleneck and the right rung — honestly.</strong> The most
common and valuable finding is that the system is <em>nowhere near</em> needing to shard: the first
bottleneck will be reads (→ replicas), storage growth (→ archiving/bigger disk), or, very often, a
<em>specific hot query or missing index</em> that has nothing to do with node capacity — and that
vertical scaling plus query/index fixes would provide years of runway. Recognizing that "we'll need
to shard" is usually premature, and that the cheaper rungs (bigger box, replicas, indexing) solve
most growth, is exactly the "don't skip to sharding" and "know your numbers" discipline. A good
answer does at least rough capacity math (current read/write rates and data size vs what one tuned
node handles) rather than assuming.
<br><br>
<strong>If sharding is genuinely foreseeable, does the shard-key analysis now.</strong> The
valuable part is stress-testing a candidate key against the real access pattern <em>before</em>
it's a one-way door under crisis pressure: would this key distribute load evenly (high cardinality,
no whale/dominant value, not monotonic), and — the crux — does the actual query workload stay
<em>single-shard</em> under it, or would common queries fan out across shards? For many systems a
natural aggregate boundary (tenant, user, account, region) is the right key <em>because</em> queries
and transactions are scoped to it; for others, the honest finding is that the query pattern is
multi-dimensional and <em>no</em> single key keeps everything single-shard — which is itself crucial
to learn early (it may push toward denormalization, a separate analytics store, or rethinking the
data model). A strong answer also checks whether the key aligns with the transaction/aggregate
boundary (so most transactions stay single-shard ACID and you avoid saga-everywhere).
<br><br>
The takeaway a good answer reaches: scaling data is a ladder climbed deliberately, most systems
need far less of it than feared (the first bottleneck is usually a query/index or reads, not a need
to shard), and if sharding does loom, the shard key is a high-stakes, near-irreversible choice that
should be reasoned out from the real access pattern — ideally long before the emergency that would
otherwise force a rushed, wrong choice.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-vertical-scaling-scale-up"></a>**vertical scaling (scale up)** — a bigger machine (more CPU/RAM/disk); simple, but has a ceiling and gets costly.
- <a id="w-horizontal-scaling-scale-out"></a>**horizontal scaling (scale out)** — more machines working together; harder, but scales far.
- <a id="w-replication"></a>**replication** — keeping copies of the same data on multiple nodes (for read scaling and availability).
- <a id="w-replication-lag"></a>**replication lag** — the delay before a write on the primary appears on the replicas.
- <a id="w-partitioning-sharding"></a>**partitioning / sharding** — splitting data across nodes so each holds a subset (for write scaling).
- <a id="w-shard-key"></a>**shard key** — the field used to decide which shard a row goes to; the single most important sharding choice.
- <a id="w-hot-spot"></a>**hot spot** — a shard that gets disproportionate load because of a bad shard key.
- <a id="w-cross-shard-query"></a>**cross-shard query** — a query that must touch many shards; slow and complex, to be avoided.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 23 — Scalability & Performance Architecture →](lesson-23-scalability-performance){: .btn .btn-primary }
