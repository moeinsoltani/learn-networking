---
title: "Lesson 21 — Caching Strategies"
nav_order: 3
parent: "Phase 5: Data & Scale"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 21: Caching Strategies

{: .note }
> **Words to know**
> - **cache** — a fast, temporary copy of data kept close to where it's used, to avoid recomputing/refetching it.
> - **cache hit / miss** — the data was found in the cache (hit) or wasn't, requiring a fetch from the source (miss).
> - **staleness** — the cached copy no longer matches the source of truth.
> - **invalidation** — removing or updating a cache entry when the underlying data changes.
> - **TTL (time to live)** — an expiry time after which a cache entry is considered stale and dropped.
> - **cache-aside / read-through / write-through / write-behind** — patterns for who fills the cache and when.
> - **stampede / thundering herd** — many requests all miss the cache at once and hit the source simultaneously.

## Concept

Caching is the sharpest double-edged sword in performance work. It can turn a slow, expensive
system fast and cheap — and it can introduce the most maddening class of bugs you'll ever
debug (data that's *sometimes* wrong, for *some* users, *some* of the time). The two hardest
problems in caching are the two hardest problems in computer science, per the old joke:
**cache invalidation** and naming things (and off-by-one errors). The joke is real:
*keeping the cache correct* is where caching hurts.

```
   WHY CACHE: source is slow/expensive/far.  Keep a fast copy close.

   [ client ] → [ CDN ] → [ API gateway ] → [ app cache ] → [ DB (source of truth) ]
      cache        cache        cache          cache            ← truth lives here
      (browser)   (edge)      (per-request)   (Redis)
   ── each layer can cache; each adds SPEED and adds STALENESS ──

   THE TWO HARD PROBLEMS:
   1. INVALIDATION — when the DB changes, how do the copies learn they're stale?
   2. COHERENCE    — with copies at many layers, which is "right" right now?

   A cache is a DELIBERATE trade: latency/load ↓  in exchange for  staleness ↑
```

The mental model to hold: **every cache is a deliberate consistency trade-off** (Lesson 15).
You're choosing to serve possibly-stale data in exchange for speed and reduced load. That's
often a great trade — but it must be a *conscious* one, tied to "how stale can this data be
before the business cares?" A cache added "for performance" without deciding the acceptable
staleness is a latent correctness bug.

## Going Deeper

**The caching layers — you can cache at every level.** A request can be served from a cache at
many points, each closer to the user (faster) but further from the truth (more stale):
- **Client/browser** cache — fastest, but you can't invalidate it (it's on the user's machine);
  controlled by HTTP cache headers.
- **CDN / edge** cache — serves static and cacheable content from near the user; great for
  assets and cacheable API responses.
- **Gateway / reverse-proxy** cache — caches responses at the API edge.
- **Application** cache (in-process or distributed like Redis) — caches computed results,
  query results, session data; the most common and controllable.
- **Database** cache — the DB's own buffer/query caches.

More layers = faster but harder to keep coherent (which copy is current?). The architect
decides *what* to cache *where*, and — critically — the staleness each layer introduces.

**The patterns — who fills the cache, and when.**
- **Cache-aside (lazy loading)** — the app checks the cache; on a miss, it loads from the DB and
  populates the cache. Most common; the cache only holds what's been requested; the app owns the
  logic. Downside: the first request is always a miss (cold), and there's a window for staleness.
- **Read-through** — the cache itself loads from the DB on a miss (the app just asks the cache).
  Cleaner app code; the cache library owns loading.
- **Write-through** — writes go to the cache *and* the DB synchronously, so the cache is always
  fresh. Consistent reads, but writes are slower (two writes).
- **Write-behind (write-back)** — writes go to the cache and are flushed to the DB
  asynchronously later. Fast writes, but risk of data loss if the cache dies before flushing, and
  more complexity. Use with care.

The choice depends on your read/write ratio and consistency needs — read-heavy with tolerable
staleness suits cache-aside; write-heavy needing fresh reads may want write-through.

**Invalidation — TTL vs explicit, and why it's hard.** Two ways to keep entries from going
stale forever:
- **TTL (expiry)** — entries expire after N seconds; simple, self-healing, but data can be stale
  for up to N seconds and you refetch even when nothing changed. Good default when *bounded*
  staleness is acceptable.
- **Explicit invalidation** — when the underlying data changes, actively evict/update the cache
  entry. Fresher, but *hard*: you must reliably know every place a piece of data is cached and
  invalidate all of them on every change (across layers, across nodes) — miss one and you serve
  stale data indefinitely. This is the "invalidation is hard" problem.

Often you combine them (explicit invalidation *plus* a TTL as a safety net, so a missed
invalidation self-heals eventually).

{: .warning }
> **The cache stampede (thundering herd) — the failure mode that bites at scale**
> A popular cached item expires (or the cache restarts). Suddenly <em>every</em> concurrent
> request for it <em>misses</em> at the same instant and <em>all</em> hit the database
> simultaneously to recompute it — a <strong>stampede</strong> that can overwhelm the very
> database the cache was protecting, sometimes causing an outage right when traffic is highest.
> It's a classic second-order effect (Lesson 2): the cache made the system faster in the common
> case but created a new, worse failure mode on expiry. Mitigations: <em>request coalescing /
> single-flight</em> (only one request recomputes; the rest wait for its result),
> <em>stale-while-revalidate</em> (serve the slightly-stale value while one request refreshes in
> the background), <em>staggered/jittered TTLs</em> (so many keys don't all expire at once), and
> pre-warming hot keys. A cache without stampede protection is an outage waiting for a cold
> moment.

**When a cache is hiding a problem you should fix instead.** A crucial architect's judgment: a
cache should *accelerate a healthy system*, not *mask a sick one*. If you're caching to hide a
missing database index, an N+1 query, or a fundamentally inefficient data model, you're papering
over a defect — the cache adds staleness and a new failure mode while the underlying problem
festers (and bites the moment the cache misses). Before adding a cache, ask: "is the source slow
because of *load* (a cache is the right answer) or because of a *defect* (fix the defect first)?"
Caching a broken query is treating a symptom; the cache should be the optimization *after* the
system is fundamentally sound, not a bandage over a wound.

---

## Lab — Design Exercise

**The situation:** A product-detail page is hammering the database — every page view runs several
queries for the product info, its price, its review summary, and its inventory status, and at
peak the database is the bottleneck. The product's core info changes rarely (a few times a week);
its price changes occasionally (daily-ish, via a promotions system); its review summary changes
slowly (as reviews trickle in); its inventory status changes constantly (with every purchase).

**Design a caching strategy across the layers.** For each piece of data, decide *whether* and
*where* to cache it, the pattern (cache-aside/write-through/etc.), the invalidation approach
(TTL/explicit), and — the crux — **name exactly what staleness each cached item introduces and
whether the business can tolerate it.** Include stampede protection for the hot items, and note
anything you'd fix *instead of* caching.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is caching per-data-item by its change rate and tolerable staleness, not one blanket
cache. A strong answer:
<br><br>
<strong>First, check for a defect to fix instead.</strong> "Several queries per page view" is a
smell — before caching, confirm the queries are healthy (proper indexes, not N+1). If the review
<em>summary</em> is being recomputed by aggregating all reviews on every page load, that's a
defect: precompute/store the summary (a maintained counter/average), don't cache an expensive
aggregation you shouldn't be running live. Cache the healthy system, don't bandage the sick one.
<br><br>
<strong>Then, cache per item by change rate + tolerable staleness:</strong>
<ul>
<li><strong>Product core info (changes a few times/week) → cache aggressively.</strong>
Application cache (Redis), cache-aside, with a generous TTL (e.g., 1 hour) <em>plus</em> explicit
invalidation when a product is edited (the edit is rare, so invalidating on write is cheap and
keeps it fresh). Also cacheable at the CDN/edge for the static parts. <em>Staleness introduced:</em>
up to the TTL if invalidation is missed — but core info changing a few times a week means an
edit-to-visibility lag of seconds (with invalidation) or at most an hour (TTL safety net) is
completely tolerable. Business impact: none.</li>
<li><strong>Price (changes ~daily via promotions) → cache with shorter TTL + explicit
invalidation on price change.</strong> Cache-aside in Redis, TTL of a few minutes, and have the
promotions system explicitly invalidate the price key when it changes a price. <em>Staleness:</em>
a customer could briefly see an old price (up to the TTL / until invalidation propagates). This
needs a business conversation: showing a stale <em>higher</em> price briefly loses a sale; showing
a stale <em>lower</em> price then charging the real (higher) price at checkout is worse (angry
customer, or you honor the shown price and lose margin). Mitigation: keep the display-price
staleness small (short TTL + invalidation), and — critically — the <em>authoritative price at
checkout</em> must be read fresh (or re-validated), not from the display cache. So: cache the
display price with small bounded staleness; never let the checkout charge from a stale cache.</li>
<li><strong>Review summary (changes slowly) → cache with a modest TTL, no explicit invalidation
needed.</strong> Cache-aside, TTL of, say, 5–15 minutes. <em>Staleness:</em> the review count/
average is a few minutes behind — completely harmless (Lesson 15's eventual-consistency case),
nobody notices or is harmed by a rating being momentarily off. TTL alone is fine; explicit
invalidation isn't worth the effort for data this tolerant.</li>
<li><strong>Inventory status (changes constantly, every purchase) → cache barely or not at all
for the authoritative check; a short-TTL cache only for the <em>display</em>.</strong> This is the
hard one. The <em>displayed</em> "in stock / 3 left" can be cached with a very short TTL (seconds)
or shown as approximate ("in stock") — brief staleness there is a minor UX issue. But the
<em>authoritative</em> "can I actually sell this?" check at add-to-cart/checkout must <em>not</em>
be served from a stale cache — an oversell has real cost (Lesson 15). So: cache the display status
loosely (short TTL), but do the real stock decision against the source (or a strongly-consistent
reservation). Caching this like the others would cause overselling.</li>
</ul>
<strong>Stampede protection for the hot items.</strong> A popular product's cached entries will
have many concurrent readers, so on expiry you risk a thundering herd onto the DB. Add
<em>request coalescing/single-flight</em> (one request recomputes, others wait) and/or
<em>stale-while-revalidate</em> (serve the just-expired value while one request refreshes in the
background) for the core-info and price keys, and <em>jitter the TTLs</em> so many products don't
all expire simultaneously. Without this, a cold cache moment at peak traffic could stampede the
database — the cache's own failure mode.
<br><br>
<strong>The meta-lesson:</strong> there is no single "cache the product page" answer — each piece
of data gets a caching decision matched to its <em>change rate</em> and its <em>tolerable
staleness</em>: aggressive long-TTL caching for the rarely-changing core info, short-TTL +
invalidation for price (with the authoritative checkout price read fresh), modest TTL for the
tolerant review summary, and near-no caching for the authoritative inventory decision (with only
the display loosely cached). Every cached item's staleness is <em>named</em> and checked against
the business cost of being wrong — and the two items where staleness is dangerous (price at
checkout, inventory at sale) are deliberately kept out of the cache's authoritative path. Plus
stampede protection so the cache doesn't become an outage on a cold moment, and fixing the review-
summary aggregation rather than caching a query that shouldn't run live. That per-item,
staleness-conscious, stampede-aware design is what separates caching that speeds a system up from
caching that introduces a class of "sometimes wrong for some users" bugs.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Caching patterns (aside/through/behind) | <https://learn.microsoft.com/azure/architecture/patterns/cache-aside> |
| Cache invalidation & the hard problems | Kleppmann, *Designing Data-Intensive Applications* (caching/derived data) |
| Cache stampede / thundering herd | <https://en.wikipedia.org/wiki/Cache_stampede> |
| HTTP caching (client/CDN) | <https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching> |
| Stale-while-revalidate | <https://web.dev/articles/stale-while-revalidate> |

---

## Checkpoint

**Q1.** Why is every cache a "deliberate consistency trade-off," and what goes wrong when a cache
is added "for performance" without deciding the acceptable staleness? Illustrate with data that
can tolerate staleness and data that can't.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
Every cache is a consistency trade-off because a cache is, by definition, a <em>copy</em> of data
that lives somewhere else (the source of truth), and the instant the source changes, the copy is
<em>stale</em> until it's invalidated or expires. So caching means deliberately accepting that
reads may return data that's out of date, in exchange for speed and reduced load on the source.
You are choosing "possibly stale but fast" over "always current but slow/expensive" — that's the
trade, and it maps directly onto the consistency spectrum (Lesson 15): a cache moves that data
from strong consistency toward eventual consistency.
<br><br>
<strong>What goes wrong without deciding acceptable staleness:</strong> if you add a cache "for
performance" without asking "how stale can this data be before the business cares?", you've
silently made a consistency decision by accident — and if the data is something that <em>can't</em>
tolerate staleness, you've introduced a correctness bug that surfaces intermittently (wrong data
for some users, some of the time, depending on cache state) — the hardest kind to diagnose. The
cache "works" in testing (where you probably see fresh data) and fails subtly in production. The
fix is to make the staleness trade <em>conscious</em>: for each cached item, decide and document
the acceptable staleness window and the invalidation strategy that keeps it within that window.
<br><br>
<strong>Illustration:</strong> a <em>product review count / average rating</em> can tolerate
staleness — if it's a few minutes behind, nobody is harmed or even notices (Lesson 15), so
caching it with a modest TTL is a pure win. A <em>price at checkout</em> or an <em>account
balance</em> or an <em>inventory "can I sell this?" decision</em> <em>cannot</em> tolerate
staleness — a stale price means charging the wrong amount or a dispute, a stale balance means a
double-spend, a stale inventory read means overselling — each with real financial/legal/trust
cost. Caching those on the authoritative path (without recognizing they can't be stale) is exactly
the "added for performance, introduced a correctness bug" failure. The discipline: cache the
tolerant data freely, keep the intolerant data off the cache's authoritative path (or read it
fresh), and in <em>every</em> case name the staleness you're accepting rather than backing into it.
</details>

**Q2.** Explain the cache stampede (thundering herd), why it's a classic second-order effect, and
two mitigations. Separately, when is a cache "hiding a problem you should fix instead"?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>The cache stampede:</strong> a popular item is cached, serving many requests cheaply from
the cache. Then it expires (TTL elapses) or the cache restarts, so the item is suddenly absent —
and <em>all</em> the concurrent requests for that hot item <em>miss</em> at the same instant and
<em>all</em> go to the database simultaneously to recompute it. That synchronized flood
(the "thundering herd") can overwhelm the database — often the very database the cache existed to
protect — potentially causing an outage, and typically at peak traffic (when the item is hottest
and most requests are in flight).
<br><br>
<strong>Why it's a classic second-order effect (Lesson 2):</strong> the cache's first-order effect
is unambiguously good — it makes the common case (a cache hit) fast and offloads the database. But
that very optimization <em>creates a new, worse failure mode</em> at the second order: because the
cache normally shields the database from load, the database is no longer provisioned/expecting the
full load, so the moment the shield drops (expiry/restart) the sudden unshielded flood is more
damaging than the steady load would have been. The mechanism that helped (offloading the DB)
produced the harm (a DB that can't survive a cold moment). The benefit and the new risk are two
steps of the same decision.
<br><br>
<strong>Two mitigations:</strong> (1) <em>Request coalescing / single-flight</em> — ensure only
<em>one</em> request recomputes the missing value while all the others wait for and share that
single result, so a cold key triggers one DB fetch instead of thousands. (2)
<em>Stale-while-revalidate</em> — serve the just-expired (slightly stale) value immediately while a
single background request refreshes it, so readers never all pile onto the DB at once (you accept a
touch more staleness to avoid the herd). (Also valid: jittered/staggered TTLs so many keys don't
expire simultaneously; pre-warming hot keys.)
<br><br>
<strong>When a cache is hiding a problem you should fix instead:</strong> when the source is slow
not because of <em>load</em> but because of a <em>defect</em> — a missing database index, an N+1
query pattern, an inefficient data model, or an expensive aggregation being recomputed live that
should be precomputed. In those cases the cache <em>masks</em> the underlying inefficiency (reads
are fast while the cache is warm) but the defect is still there, now made <em>worse</em> because
the cache adds staleness and a stampede risk on top, and the real problem bites hard the moment the
cache misses (the uncached path is still broken, now hit by a herd). The judgment: a cache should
<em>accelerate a fundamentally healthy system</em>, not <em>bandage a sick one</em>. Before
caching, ask whether the slowness is load (cache is the right tool) or a defect (fix the index/
query/model first); caching a broken query treats the symptom while the disease grows.
</details>

---

## Homework

Audit the caches in your current system (or where you'd add one). For each cache, identify: the
data cached, its change rate, the staleness it introduces, and whether the business tolerates that
staleness — looking specifically for any cache on data that *can't* actually tolerate being stale
(a latent correctness bug). Check whether your hot caches have stampede protection or would herd
the database on a cold moment. Finally, ask of one cache: is it accelerating a healthy system, or
masking a defect (a missing index, an N+1, an expensive live aggregation) that you should fix
instead? Name the one caching change you'd make.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise turns the staleness-and-stampede discipline into a concrete audit. A strong response:
<br><br>
<strong>Names the staleness per cache and hunts the dangerous one.</strong> The highest-value
finding is a cache on data that <em>can't</em> tolerate staleness sitting on an authoritative path
— a price, balance, permission/entitlement, or inventory decision served from a cache that can be
out of date — which is a latent, intermittent correctness bug (wrong for some users when the cache
is stale). Even if it hasn't caused a visible incident yet, recognizing it (and moving that data
off the authoritative cache path, or reading it fresh) is exactly the "make the staleness trade
conscious" lesson. Equally, a good answer confirms the <em>tolerant</em> caches (counts, summaries,
rarely-changing display data) are fine and well-matched.
<br><br>
<strong>Checks stampede protection.</strong> The common finding is that hot caches have <em>no</em>
stampede protection — a simple TTL with no coalescing/stale-while-revalidate — meaning a cold
moment (deploy, cache restart, synchronized expiry) at peak traffic would herd the database. This
is a latent availability risk that hasn't fired only because the cache hasn't gone cold at a bad
moment. Naming it and the mitigation (single-flight, stale-while-revalidate, jittered TTLs) is a
concrete resilience improvement.
<br><br>
<strong>Distinguishes acceleration from masking.</strong> The most instructive reflection is
finding a cache that's <em>hiding a defect</em> — caching the result of a query that's slow because
of a missing index, an N+1, or a live aggregation that should be precomputed — and recognizing that
the right fix is the underlying query/index/model, with the cache as an optional optimization
<em>after</em>, not a bandage. The takeaway a good answer reaches: caching is powerful but each
cache carries a named staleness cost, a stampede risk, and a temptation to mask defects — so the
one change worth making is usually either (a) removing a dangerous cache from an authoritative path,
(b) adding stampede protection to a hot cache, or (c) fixing a defect a cache is hiding — a specific,
prioritized action rather than "add more caching."
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 22 — Scaling Data (Partitioning, Sharding, Replication) →](lesson-22-scaling-data){: .btn .btn-primary }
