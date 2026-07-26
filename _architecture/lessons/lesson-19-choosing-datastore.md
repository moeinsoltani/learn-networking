---
title: "Lesson 19 — Choosing a Data Store (Polyglot Persistence)"
nav_order: 1
parent: "Phase 5: Data & Scale"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 19: Choosing a Data Store (Polyglot Persistence)

{: .note }
> **Words to know**
> - **data model** — how a store organizes data: relational (tables), document (JSON), key-value, wide-column, graph, etc.
> - **access pattern** — how your app actually reads and writes data: the queries, the read/write ratio, the shape of lookups.
> - **relational / RDBMS** — tables with rows, joins, and ACID transactions (Postgres, MySQL).
> - **document store** — stores self-contained documents (JSON-like), flexible schema (MongoDB).
> - **key-value store** — a giant hash map: get/put by key, very fast (Redis, DynamoDB).
> - **polyglot persistence** — deliberately using different stores for different needs in one system.
> - **normalization / denormalization** — splitting data to avoid duplication vs duplicating it to speed reads.

## Concept

The reflex "we need a database, use the one we always use" is one of the most consequential
un-decisions in software. Different data stores are built for different **access patterns**,
and the right choice flows from *how you'll read and write the data*, not from what the data
"is." The architect's discipline: **match the store to the access pattern.**

```
   THE DATA-MODEL FAMILIES (each is FOR an access pattern)

   RELATIONAL   tables, joins, ACID     → complex queries, relationships,
   (Postgres)                             transactions, "I don't know all my
                                          queries yet" → the safe default
   DOCUMENT     self-contained JSON      → aggregate you read/write whole
   (Mongo)                                (a product, an order), flexible schema
   KEY-VALUE    get/put by key           → simple, ultra-fast lookups by id;
   (Redis,DDB)                            caching, sessions, high throughput
   WIDE-COLUMN  rows w/ huge, sparse     → massive write volume, time-series-ish,
   (Cassandra)  column sets               known query patterns, horizontal scale
   GRAPH        nodes + edges            → relationship-heavy traversals
   (Neo4j)                                (social graph, recommendations, fraud)
   SEARCH       inverted index           → full-text search, faceting, ranking
   (Elastic)                              (a product catalog search box)
   TIME-SERIES  timestamped points       → metrics, IoT, append-heavy by time
   (Influx)
```

The key inversion: don't ask "what does my data look like?" — ask "**how will I query and
write it?**" A social graph *is* relational data, but if your access pattern is deep
relationship traversal ("friends of friends of friends"), a graph store serves it far better
than SQL joins. A product catalog *is* structured, but if the access pattern is fuzzy
full-text search with facets, a search index beats a relational `LIKE`.

## Going Deeper

**"Start with Postgres" is an honest default — know when to deviate.** For most systems,
starting with a mature relational database (Postgres) is the right call: it handles a
*huge* range of access patterns competently (it even does JSON documents, full-text search,
and more natively now), gives you ACID transactions and joins, is battle-tested, and — crucially
— you often *don't yet know all your query patterns* early on, and relational's flexibility to
support ad-hoc queries is exactly what you want when the access pattern is still emerging. You
deviate to a specialized store when a *specific, well-understood* access pattern is poorly
served by relational and the specialization clearly pays off (extreme write volume, deep graph
traversal, full-text search at scale, sub-millisecond key lookups at massive throughput). The
mistake in *both* directions: reaching for a trendy NoSQL store by default (losing transactions
and joins you'll miss), and forcing a genuinely graph/search/time-series workload into
relational because "we always use Postgres."

**The questions that actually decide.** When choosing, answer these about the *access pattern*:
- **Read/write ratio and volume** — read-heavy? write-heavy? how much? (Wide-column for huge
  writes; caches/replicas for huge reads.)
- **Query shape** — do you look things up by a single key, or run complex multi-condition
  queries and aggregations, or traverse relationships, or do full-text search? (Key-value vs
  relational vs graph vs search.)
- **Relationships** — is the data highly interconnected and are the relationships what you
  query? (Graph.)
- **Consistency needs** — do you need ACID transactions across records (relational), or is
  eventual consistency fine (many NoSQL)?
- **Schema stability** — fixed and relational, or varying/evolving per record (document)?
- **Scale requirements** — will one node do, or do you need built-in horizontal scaling
  (many NoSQL stores trade features for this)?

The store falls out of the honest answers to these — it's a *derived* decision, not a taste.

{: .warning }
> **NoSQL's real trade: you buy scale/flexibility by giving up joins and transactions.**
> The NoSQL stores didn't invent free performance — they made a <em>trade</em>. Most gained
> horizontal scalability and schema flexibility by <em>dropping</em> the things relational
> databases work hard to provide: multi-record ACID transactions, joins, and rich ad-hoc
> queries. That's a great deal <em>when your access pattern doesn't need those</em> (you read
> whole aggregates by key, you don't join, eventual consistency is fine) — and a painful one
> when it does (you discover, three months in, that you need a transaction across two documents,
> or a join, and now you're doing it in application code, badly). So the question isn't "is
> NoSQL faster/more scalable?" (sometimes, for its access pattern) but "does my access pattern
> actually <em>need</em> what I'd be giving up?" Choosing NoSQL to avoid a scaling problem you
> don't have, and thereby losing transactions/joins you <em>do</em> need, is a common and
> expensive mistake.

**Polyglot persistence — powerful, but it has an operational cost.** Because different parts of
a system have different access patterns, a mature system often uses *several* stores
deliberately: Postgres for transactional order data, Elasticsearch for the product-search box,
Redis for sessions and caching, maybe a graph store for recommendations. This **polyglot
persistence** matches each store to its workload — genuinely better than forcing everything into
one. *But* each store you add is another thing to run, monitor, back up, secure, patch, and
build expertise in (an operational and cognitive tax). So the discipline is to add a specialized
store when the access-pattern benefit clearly exceeds that operational cost — not to collect
databases because each is individually optimal. Often the honest answer is "Postgres does this
80%-well and isn't worth a second store yet," and sometimes it's "this search workload genuinely
needs Elasticsearch." Weigh the specialization benefit against the operational burden, per store.

**Data ownership is the deeper architectural point.** In a distributed system, *which service
owns which data* (and therefore which store) is more architecturally significant than the store
technology itself — the database-per-service question (Lesson 24). Choosing a store is often
downstream of the boundary decision: once a bounded context owns its data, it can pick the store
that fits *its* access pattern, independently of other contexts. Polyglot persistence and
service boundaries reinforce each other — each service's autonomy includes choosing its own store.

---

## Lab — Design Exercise

**The situation:** You're designing the data layer for an e-commerce platform. It has these
distinct pieces of data, each with a different access pattern:

1. **Orders & payments** — transactional; an order has line items, must be atomic (all-or-
   nothing), queried by customer, by date range, with reporting.
2. **Product catalog search** — the customer's search box: fuzzy full-text matching, filtering
   by facets (brand, price range, category), ranked by relevance, high read volume.
3. **Shopping-cart & user sessions** — looked up by session/user key, read and written on nearly
   every request, must be very fast, short-lived, no complex querying.
4. **Product recommendations** — "customers who bought X also bought Y", "people in your network
   liked", traversing relationships between users, purchases, and products.

**Choose a data store for each, driven by the access pattern,** and justify it. Then step back:
is this polyglot persistence worth it here, or would starting with just Postgres be the wiser
first move? Argue the trade-off honestly.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is deriving each store from its access pattern, then weighing polyglot vs simplicity.
A strong answer:
<br><br>
<strong>1. Orders & payments → relational (Postgres/MySQL).</strong> The access pattern demands
exactly what relational is built for: <em>ACID transactions</em> (an order + its line items +
the payment must be atomic — the one place you can't afford eventual consistency, Lesson 15),
<em>relationships/joins</em> (order → line items → products → customer), and <em>rich ad-hoc
queries and reporting</em> (by customer, by date range, aggregations). NoSQL would force you to
give up the transactions and joins you critically need here. Relational is not a default here —
it's the <em>correct</em> choice for a transactional, relational, query-rich workload.
<br><br>
<strong>2. Product catalog search → a search index (Elasticsearch/OpenSearch).</strong> The
access pattern is <em>full-text fuzzy matching + faceted filtering + relevance ranking</em> at
high read volume — precisely what an inverted-index search engine is for, and precisely what
relational is <em>bad</em> at (a SQL <code>LIKE '%term%'</code> can't do fuzzy matching,
relevance ranking, or efficient faceting at scale). Note the pattern: the catalog data's
<em>source of truth</em> may still be Postgres, with the search index as a derived read model
kept in sync (a CQRS-ish read projection, Lesson 20) — you don't have to make Elasticsearch
authoritative.
<br><br>
<strong>3. Cart & sessions → key-value store (Redis, or DynamoDB).</strong> The access pattern
is <em>get/put by a single key, on nearly every request, must be very fast, short-lived, no
complex queries</em> — the textbook key-value case. Redis gives sub-millisecond lookups, TTL for
expiry, and takes the read/write load of session/cart access off your relational database. No
joins or transactions needed, so nothing is lost by using a key-value store, and a lot of speed
and DB-load-relief is gained.
<br><br>
<strong>4. Recommendations → graph store (Neo4j), or a specialized rec system.</strong> The
access pattern is <em>relationship traversal</em> — "customers who bought X also bought Y,"
"people in your network liked" — which is deep, multi-hop traversal across users/purchases/
products. This is exactly where graph databases shine and where relational joins become
painful/slow (multi-level self-joins). (Caveat: recommendations are often built with dedicated
ML/rec-engine pipelines rather than a live graph DB — a reasonable alternative — but the
<em>access pattern</em> is graph-shaped, which is the point.)
<br><br>
<strong>Is polyglot worth it here? The honest trade-off.</strong> Each specialized store is
individually well-matched, but each adds operational cost (run, monitor, back up, secure, patch,
learn). So the mature answer depends on <em>stage</em>:
<ul>
<li><strong>Early / MVP: start with just Postgres.</strong> Postgres can do <em>all four</em>
adequately at first — relational for orders (great), its built-in full-text search for the
catalog (good enough for a small catalog), a table (or Redis if you must) for sessions, and
recursive queries for shallow recommendations (mediocre but workable). Running <em>one</em>
store is a huge operational simplicity win, and early on you don't yet have the scale that
justifies the specialized stores. Adding Elasticsearch + Redis + Neo4j on day one is
over-engineering — four systems to operate for a product that hasn't proven demand.</li>
<li><strong>At scale: deviate to the specialized store <em>where the pattern demands it and the
benefit exceeds the operational cost</em>.</strong> The <em>first</em> to justify itself is
usually search (Postgres full-text search genuinely struggles with faceting + relevance at a
large catalog and high volume → Elasticsearch earns its keep), and Redis for sessions/caching
(when DB load from per-request cart/session access becomes a problem). The graph store is the
last and most optional (recommendations can start crude and improve). Orders <em>stay</em> on
Postgres regardless — you never want to give up those transactions.</li>
</ul>
<strong>The meta-lesson:</strong> the <em>right target state</em> is polyglot (each store
matched to its access pattern — that's genuinely better than one-size-fits-all), but the
<em>right path</em> is to start simple (Postgres for as much as it reasonably serves) and add a
specialized store when a specific access pattern's need clearly outgrows what the general-purpose
store provides and the specialization's benefit beats its operational tax. Both mistakes are
real: forcing everything into one store forever (the search box stays bad, the DB groans under
session load) and reaching for four stores on day one (drowning a young system in operational
complexity). Match store to access pattern — but sequence the additions by need, not by what's
individually optimal in isolation.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Polyglot persistence | Martin Fowler — <https://martinfowler.com/bliki/PolyglotPersistence.html> |
| Data models & storage engines | *Designing Data-Intensive Applications*, Kleppmann (Ch. 2–3) |
| "Just use Postgres" (the default argument) | <https://www.amazingcto.com/postgres-for-everything/> |
| NoSQL data modeling & trade-offs | <https://en.wikipedia.org/wiki/NoSQL> |

---

## Checkpoint

**Q1.** Why should the *access pattern*, not "what the data is," drive the choice of data store?
Give an example where the same data is best served by different stores depending on how it's
queried.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The access pattern should drive the choice because a data store's design is fundamentally about
<em>how it reads and writes</em> — its performance characteristics, the queries it makes cheap vs
expensive, the guarantees it offers — and those matter only in relation to <em>how you'll
actually use the data</em>. The same data can be trivial to serve with one store's access model
and painful with another's; what determines fit is the query shape, read/write ratio,
relationship-traversal needs, and consistency requirements — i.e., the access pattern — not the
data's intrinsic "type." Asking "what does my data look like?" leads to picking a store by
superficial resemblance; asking "how will I query and write it?" leads to picking the store
whose engine is built for that workload.
<br><br>
Example: a <strong>social network's connections</strong> (who follows whom). As <em>data</em>,
this is perfectly relational — a <code>follows(follower_id, followee_id)</code> table. If the
access pattern is simple ("list who I follow"), a relational store serves it fine. But if the
access pattern is <em>deep relationship traversal</em> — "friends of friends of friends,"
"shortest path between two users," "people you may know two hops away" — a relational store
serves it <em>badly</em>: each hop is another self-join, and multi-hop traversals become
enormous, slow join queries that degrade sharply with depth. The <em>same data</em> is far
better served by a <strong>graph database</strong>, whose engine makes traversing edges cheap
regardless of depth. Conversely, if that same follow data is mostly written in huge volume and
read by simple key lookups, a wide-column or key-value store might fit better still. The data
didn't change — the <em>access pattern</em> did, and with it the right store. (Same lesson with a
product catalog: as data it's structured/relational, but if the access pattern is fuzzy
full-text search with facets and ranking, a search index beats relational; if it's "fetch the
whole product by id," a document or key-value store fits.) The store is a <em>derived</em>
decision from the access pattern, which is why two systems holding identical data can correctly
choose different stores.
</details>

**Q2.** What is the real trade NoSQL stores make, and why is "we should use NoSQL because it's
more scalable" a dangerous way to choose? When is "start with Postgres" the right default, and
when should you deviate?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>The real NoSQL trade:</strong> NoSQL stores generally gained horizontal scalability and
schema flexibility by <em>giving up</em> features relational databases provide — most notably
multi-record ACID transactions, joins, and rich ad-hoc querying. Performance and scale weren't
free; they were <em>bought</em> by dropping guarantees and capabilities. That's an excellent
trade when your access pattern doesn't need those things (you read whole aggregates by key, never
join, and eventual consistency is acceptable) and a costly one when it does.
<br><br>
<strong>Why "NoSQL because it's more scalable" is dangerous:</strong> it chooses on a single axis
(scale) in the abstract, usually to solve a scaling problem the system <em>doesn't actually
have</em>, while ignoring what's being sacrificed. Teams adopt a NoSQL store for imagined future
scale, then discover months in that they need a transaction across two records, or a join, or a
complex query — capabilities they gave away — and end up reimplementing them in application code,
badly and unreliably (hand-rolled joins, application-level "transactions" that aren't atomic).
The scale benefit was hypothetical; the lost transactions/joins are a daily pain. The choice
should be driven by whether your access pattern <em>needs</em> what you'd be giving up, not by
"scalable" as a generic virtue.
<br><br>
<strong>When "start with Postgres" is right:</strong> for most systems, as the default —
especially early, when (a) you don't yet know all your query patterns, and relational's ability
to serve ad-hoc queries flexibly is exactly what you want while the access pattern is still
emerging; (b) you benefit from ACID transactions and joins (most business data is relational and
needs consistency); and (c) a mature relational DB competently serves a huge range of workloads
(including JSON documents and full-text search natively), so one well-understood store covers a
lot. It's also the operationally simplest choice (one system to run).
<br><br>
<strong>When to deviate:</strong> when a <em>specific, well-understood</em> access pattern is
genuinely poorly served by relational <em>and</em> the specialization clearly pays off beyond its
operational cost — e.g., full-text search with faceting/ranking at scale (→ search index),
sub-millisecond key lookups at massive throughput (→ key-value), deep relationship traversal (→
graph), or extreme write volume / time-series (→ wide-column / time-series). The deviation is a
<em>derived</em> decision from a real, identified access-pattern need — not a default reach for
the trendy option, and not stubbornly forcing a genuinely graph/search/time-series workload into
relational because "we always use Postgres." Start with the general-purpose store, add a
specialized one where a concrete access pattern's need outgrows it and the benefit exceeds the
operational tax.
</details>

---

## Homework

For your current system, list its major data stores and, for each, the access pattern it serves.
Then look for two mismatches: (1) data forced into a store that fits it poorly (a graph-shaped or
search-shaped or write-heavy workload jammed into a general-purpose relational DB and struggling,
or vice versa), and (2) a specialized store you're running whose operational cost may not be
justified by its benefit (a second/third database that a single well-used store could have
served). If you're early-stage and reaching for multiple stores, challenge whether "just
Postgres" would serve you well enough for now. Note the one store decision you'd revisit.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise applies access-pattern-driven thinking and the polyglot-cost trade to a real
system. A strong response:
<br><br>
<strong>Names each store's access pattern</strong> (not just its name) and judges the fit. The
first mismatch to hunt is a workload <em>fighting its store</em>: a full-text search
implemented with slow relational <code>LIKE</code> queries (should be a search index), a
relationship-heavy feature drowning in multi-level joins (graph-shaped), a write-heavy or
time-series workload straining a general-purpose DB, or high-frequency session/cache access
loading the primary database (should be a key-value cache). Recognizing that a performance pain
point is actually an access-pattern/store mismatch — rather than something to fix with more
hardware — is a valuable reframing.
<br><br>
<strong>Questions polyglot cost the other way.</strong> The second mismatch is an
<em>unjustified</em> specialized store — a second or third database adopted for a workload the
primary store could have handled adequately, now carrying ongoing operational cost (backup,
monitoring, patching, expertise) that exceeds its benefit. This is the "we added MongoDB/Neo4j/
Cassandra and it's more trouble than it's worth" finding, and the honest conclusion may be to
consolidate back. For early-stage systems, the challenge is sharper: are the multiple stores
justified <em>yet</em>, or would starting with just Postgres (using its JSON, full-text search,
etc.) have avoided a pile of premature operational complexity?
<br><br>
The takeaway a good answer reaches: the right store per workload is derived from the access
pattern, and a mature system is often deliberately polyglot — but each store must earn its
operational keep, so the two failures are symmetric (a workload jammed into the wrong store, and
too many stores for the value they add). Naming the one store decision to revisit — whether
that's introducing a specialized store for a workload that has outgrown the general one, or
retiring a specialized store whose cost isn't justified — turns the audit into a concrete,
prioritized action rather than an abstract "we should think about our data layer."
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 20 — Event Sourcing & CQRS →](lesson-20-event-sourcing-cqrs){: .btn .btn-primary }
