---
title: "Lesson 07 — Domain-Driven Design Essentials"
nav_order: 3
parent: "Phase 2: Foundations of Structure"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 07: Domain-Driven Design Essentials

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

Where do good boundaries (Lesson 6) actually come from? Domain-Driven Design's answer:
**from the business domain itself, expressed in a shared language.** Not from the
database schema, not from the framework's layers — from how the business actually
thinks and talks about its work. DDD is the discipline of letting the **domain**[°](#w-domain) drive the
model, the boundaries, and even the code's vocabulary.

The load-bearing idea is the **bounded context**[°](#w-bounded-context): a boundary within which a model and
its terms mean exactly one thing. The revelation that makes DDD click: **the same word
means different things in different parts of the business, and that's not a problem to
resolve — it's a boundary to respect.**

Start with the observation that unlocks domain-driven design: **"Customer" is
not one thing.**

| In the **Sales** context | In the **Support** context | In the **Billing** context |
|---|---|---|
| A lead, a pipeline stage, a deal size, a contact | A ticket history, entitlements, an SLA | An account, invoices, a payment method, tax status |

Same word, three genuinely different models. Nothing is wrong here — each team
means something real and specific by "customer," and each is right within its
own context.

The mistake is trying to build **one** `Customer` object that serves all three.
What you get is a bloated, low-cohesion model that couples three unrelated
concerns, so that a change to tax handling can break the sales pipeline.

DDD's answer is to stop fighting it: **three contexts, three `Customer` models,
translated at the seams.** The translation between them is real work, and it is
much less work than the alternative.

Bounded contexts are the *natural seams* of a system — and, not coincidentally, the
right size for a module or a service. When people ask "how do I know where to split
services?", DDD's answer is usually "along your bounded contexts." That's why this
lesson sits between boundaries (Lesson 6) and styles (Lesson 8).

## Going Deeper

**Ubiquitous language: one vocabulary, no translation.** In most projects there's the
business's language and the developers' language, and someone mistranslates between
them constantly ("what you call a 'policy' we store as an 'agreement record'"). DDD
insists on *one* language — the same terms used by domain experts, in conversation,
in the docs, and *in the code* (class names, methods). The payoff is fewer
mistranslation bugs and a codebase that a domain expert could almost read. The
language is *scoped to a context* (that's why "Customer" can differ across contexts) —
ubiquitous *within* a bounded context, not globally.

**Strategic vs tactical DDD — start strategic.** DDD has two halves. **Strategic**
DDD is about the big picture: identifying **subdomains**[°](#w-subdomain), drawing bounded contexts,
mapping how they relate — this is the architecturally significant part and where you
should focus. **Tactical** DDD is the in-context building blocks: entities,
value objects, *aggregates*[°](#w-aggregate) (a cluster of objects with one consistency boundary and
one "root"), repositories, domain events. Tactical patterns are useful but optional
and often over-applied; the strategic part is what earns DDD its place in an
architecture track. If you take one thing from DDD, take *bounded contexts*.

{: .note }
> **Core vs supporting vs generic — where to spend your best people**
> Not all of the domain deserves equal investment. **Core** subdomains are your
> competitive advantage — the thing your business does that others don't (the
> matching algorithm for a ride-hailing app, the pricing engine for an insurer). Put
> your strongest engineers and your best design here; this is where custom-built,
> carefully-modeled code pays off. **Supporting** subdomains are necessary but not
> special (an admin tool, an internal reporting module) — build them simply, don't
> gold-plate. **Generic** subdomains are solved commodities (authentication, email
> sending, payments) — *buy or use off-the-shelf*, never build. A frequent, expensive
> mistake is lavishing custom engineering on a generic subdomain (building your own
> auth) while under-investing in the core. Classifying subdomains tells you where to
> build vs buy (Lesson 33) and where the architecture must be excellent vs merely
> adequate.

**Context mapping: how contexts relate.** Once you have contexts, you map their
relationships, because integration is where coupling sneaks back in. The key patterns:
*partnership* (two contexts succeed or fail together, coordinate closely);
*customer-supplier* (upstream provides, downstream consumes, with the downstream's
needs given weight); *conformist* (downstream just accepts the upstream's model as-is
— cheap but couples you to it); and the crucial **anti-corruption layer (ACL)**[°](#w-anti-corruption-layer-acl) — a
translation layer that converts an external/upstream model into *your* context's
terms, so their model (and their changes) can't leak in and corrupt yours. The ACL is
the DDD tool you reach for whenever you integrate with a legacy system or a third party
you don't control — it's the boundary that keeps their mess out of your model.

**Why bounded contexts are the seams for services.** A bounded context is
high-cohesion (one consistent model, one language, one team's understanding) and it
integrates with others through explicit, translated contracts (loose coupling) — which
is *exactly* the property you want in a service boundary. Splitting services along
bounded contexts gives you cohesive, independently-changeable services; splitting them
some other way (by technical layer, or arbitrarily) gives you the distributed monolith.
This is the bridge from Phase 2 into the styles of Phase 3.

---

## Lab — Design Exercise

**The situation:** You're modeling an e-commerce platform. The team's instinct is to
build one big shared `Customer` class, one `Product` class, and one `Order` class used
everywhere, all backed by shared tables. But you notice the word "Product" means
different things to different parts of the business:

- To **Merchandising**, a Product has descriptions, images, categories, SEO keywords.
- To **Inventory**, a Product is a SKU with stock levels, warehouse locations, reorder
  thresholds.
- To **Pricing**, a Product is a price, a cost, tax rules, and discount eligibility.

**Design the bounded contexts.** Identify at least three, show how "Product" (and/or
"Customer") differs across them, classify each as core/supporting/generic, and place
one anti-corruption layer where it's needed (e.g., integrating a third-party shipping
provider). Explain why one universal `Product` model would be worse than several.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is recognizing that one word maps to several models, and turning that into
clean contexts. A strong answer:
<br><br>
<strong>The bounded contexts</strong> (at least):
<ul>
<li><strong>Catalog / Merchandising</strong> — "Product" = the customer-facing listing
(title, description, images, categories, SEO). Model optimized for browsing and search.
<em>Supporting-to-core</em> (the shopping experience can be a differentiator).</li>
<li><strong>Inventory</strong> — "Product" = a SKU with stock, warehouse location,
reorder thresholds. Model optimized for logistics; changes on warehouse events, not
marketing events. <em>Supporting</em> (necessary, not a differentiator — unless
logistics <em>is</em> your edge).</li>
<li><strong>Pricing</strong> — "Product" = price, cost, tax rules, discount
eligibility. Changes constantly for commercial reasons. Often <em>core</em> for a
retailer (pricing/promotion strategy is competitive).</li>
<li><strong>Ordering / Checkout</strong> — "Product" appears here only as a line item
(id, name snapshot, price snapshot at time of purchase). Note it deliberately
<em>copies</em> a snapshot rather than referencing the live Catalog/Pricing models —
an order must remember what was bought and charged even if the product/price later
changes.</li>
</ul>
And "Customer" similarly differs: a marketing profile in one context, a shipping
address + order history in Ordering, an account + payment method in Billing.
<br><br>
<strong>Classification drives investment.</strong> Pricing and (say) the recommendation
/ search experience are <em>core</em> — invest, model carefully, build custom.
Inventory and Merchandising are <em>supporting</em> — build adequately. Payment
processing, authentication, tax calculation, and shipping-rate lookup are
<em>generic</em> — buy/integrate (Stripe, an auth provider, a tax API, the shipping
carrier), don't build.
<br><br>
<strong>Anti-corruption layer.</strong> Integrating a third-party shipping provider
(FedEx/UPS) is the textbook ACL: their API speaks <em>their</em> model (their address
format, their service codes, their tracking status vocabulary). Rather than let that
model spread through your Ordering/Delivery context, you build an ACL — a translation
layer that converts your domain's concepts ("shipment," "delivery estimate") to and
from the carrier's API, isolated in one place. Benefits: (1) your domain stays clean and
in your own language; (2) if you switch carriers, or the carrier changes their API, the
blast radius is the ACL, not your whole context; (3) you can support multiple carriers
behind one internal interface. The same pattern applies to integrating a legacy system
or any upstream you don't control.
<br><br>
<strong>Why one universal <code>Product</code> is worse.</strong> A single shared model
serving all contexts would have to contain <em>everything</em> — descriptions, images,
stock levels, warehouse locations, prices, tax rules, discount logic — a bloated,
low-cohesion "God object" that changes for every reason (a marketing copy edit, a stock
adjustment, a price change all touch the same class/table). Every context would be
coupled to every other through it: Inventory's schema changes would ripple into
Merchandising; a Pricing change could break Checkout. It would also force impossible
compromises (Inventory wants strong consistency on stock; Merchandising is fine with
eventual consistency on descriptions — one model can't optimize both). Splitting into
contexts gives each a <em>small, cohesive, purpose-fit</em> model that changes for one
reason, owned by one team, integrated through explicit contracts — the same "high
cohesion, loose coupling" from Lesson 5, now derived from the domain. And because the
contexts are clean seams, they're the natural units if/when you later split into
services (Phase 3) — whereas the universal <code>Product</code> model could never be
cleanly split at all.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Domain-Driven Design (the original) | Eric Evans, *Domain-Driven Design* (the "blue book") |
| Bounded context | Martin Fowler — <https://martinfowler.com/bliki/BoundedContext.html> |
| Strategic DDD, context mapping, subdomains | Vaughn Vernon, *Implementing Domain-Driven Design* |
| Ubiquitous language | <https://martinfowler.com/bliki/UbiquitousLanguage.html> |
| Anti-corruption layer | <https://learn.microsoft.com/azure/architecture/patterns/anti-corruption-layer> |

---

## Checkpoint

**Q1.** Explain "the same word means different things in different bounded contexts"
with an example, and why DDD treats that as a *feature* (a boundary to respect) rather
than an inconsistency to eliminate.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
Example: "Customer" (or "Product," "Policy," "Account") means genuinely different
things in different parts of a business. To a <em>Sales</em> context a Customer is a
lead with a pipeline stage, deal size, and contacts; to a <em>Support</em> context the
same Customer is a ticket history, entitlements, and an SLA; to a <em>Billing</em>
context they're an account with invoices, a payment method, and tax status. The word is
the same; the <em>model</em> — the data and behavior that matter — is different in each.
<br><br>
DDD treats this as a <strong>feature, not a bug</strong>, because the differences are
<em>real</em>: they reflect that each part of the business genuinely cares about
different aspects of the same real-world thing. Trying to "eliminate the
inconsistency" by building one unified Customer model that serves all contexts produces
exactly the pathology of Lesson 6 and 7 — a bloated, low-cohesion God object that
contains every context's concerns, changes for every context's reasons, and couples all
the contexts together through itself (a Sales schema change rippling into Billing).
Instead, DDD says: draw a <strong>bounded context</strong> around each meaning, let
"Customer" mean one consistent thing <em>within</em> each context, and translate at the
seams where they integrate. The differing meanings <em>are</em> the natural boundaries
— they tell you where one cohesive model ends and another begins. So rather than forcing
false uniformity, you respect the differences and get several small, cohesive,
independently-evolvable models, integrated through explicit contracts. The disagreement
about what a word means is <em>information</em> about where the boundary should go.
</details>

**Q2.** What are core, supporting, and generic subdomains, and how does classifying a
subdomain change your build-vs-buy and engineering-investment decisions? Give the
classic mistake.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The three kinds of subdomain, by strategic importance:
<ul>
<li><strong>Core</strong> — your competitive advantage, the thing your business does
that others don't and that customers pay for (a ride-hailing app's matching, an
insurer's pricing engine, a search company's ranking). This is where the differentiation
lives.</li>
<li><strong>Supporting</strong> — necessary for the business to function but not a
differentiator and not special (an internal admin tool, a reporting module, an
onboarding workflow). Needed, but nobody chooses you because of it.</li>
<li><strong>Generic</strong> — a solved, commodity problem that every business has and
that's identical across businesses (authentication, sending email, payment processing,
tax calculation).</li>
</ul>
<strong>How it changes decisions:</strong> the classification tells you where to
<em>build vs buy</em> and where to <em>spend your best engineering</em>. Core → build it
yourself, put your strongest people on it, model it carefully, make the architecture
excellent; this is the one place custom work clearly pays off, because being better here
<em>is</em> the business. Supporting → build it, but simply and adequately; don't
gold-plate, don't over-engineer, spend as little as you can while meeting the need.
Generic → <em>buy or use off-the-shelf/open-source</em>; never build, because a vendor
has solved it better than you will and building it yourself is pure opportunity cost
with no competitive upside (Lesson 33).
<br><br>
<strong>The classic mistake:</strong> lavishing custom engineering on a
<em>generic</em> subdomain while <em>under-investing in the core</em> — the team spends
six months lovingly hand-building their own authentication system or their own message
queue (generic, solved problems where "custom" buys nothing), and meanwhile the core
differentiator gets rushed and mediocre. It usually happens because the generic problem
is technically fun and well-understood (so it's tempting), while the core problem is
messy and domain-specific (so it's avoided). The discipline of classifying subdomains
forces the question "is this actually our competitive edge, or is it plumbing everyone
has?" — and redirects the scarce best-engineering-effort to where it creates advantage
(core) instead of where it's just reinventing a commodity (generic).
</details>

---

## Homework

Map the bounded contexts of your current system (or a system you know well). List the
contexts, and for at least one term that appears in several of them (a "Customer,"
"Order," "Account," "Item"), show how its model genuinely differs across contexts —
and check whether your real system respects that (separate models) or violates it (one
shared God object coupling everything). Then classify your subdomains as
core/supporting/generic, and honestly assess whether your engineering investment
matches: are you building anything generic that you should buy, or under-investing in
your actual core?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
This applies the two strategic-DDD tools — context mapping and subdomain classification
— to a real system, which is where they pay off. A strong response:
<br><br>
<strong>Finds the shared-term tension.</strong> The most instructive finding is usually
a term (very often "User"/"Customer" or "Product"/"Item") that the code models as
<em>one</em> shared class/table but that the business actually means differently in
different areas — the God-object coupling in the flesh. Recognizing that your real
system has forced false uniformity where the domain wanted separate models is the whole
point; it explains why changes in one area mysteriously break another (they're coupled
through the shared model). The fix direction is to split it into per-context models with
translation at the seams — even if that's aspirational for now, naming it is the value.
Occasionally the finding is the opposite (the system <em>does</em> respect the contexts
well), which is worth noting as a strength and understanding <em>what</em> kept it
clean.
<br><br>
<strong>Classifies subdomains and checks investment alignment.</strong> The valuable,
often-uncomfortable finding: many teams discover they've <em>built something generic</em>
they should have bought (a home-grown auth system, a custom job scheduler, a bespoke
notification service) — sunk cost in commodity plumbing that carries ongoing maintenance
burden and zero competitive upside — and/or that their actual <em>core</em>
differentiator is under-invested (rushed, under-staffed, architecturally weakest)
because the generic-but-fun work absorbed the attention. Surfacing this misallocation is
exactly what the core/supporting/generic lens is for: it reframes "where should our best
engineers and our custom-build effort go?" as a strategic question with a clear answer
(the core), and turns "should we keep maintaining our custom X?" into a build-vs-buy
reconsideration (Lesson 33). A good answer names at least one concrete reallocation it
implies — something to stop building and buy, or something core to invest more in.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-domain"></a>**domain** — the business problem space the software serves (e.g., "e-commerce," "insurance claims").
- <a id="w-ubiquitous-language"></a>**ubiquitous language** — one shared vocabulary, used identically by developers and domain experts and in the code.
- <a id="w-bounded-context"></a>**bounded context** — a boundary within which a model and its language are consistent; the same word can mean different things in different contexts.
- <a id="w-subdomain"></a>**subdomain** — a part of the domain; classified as **core** (your competitive edge), **supporting** (needed but not special), or **generic** (a solved commodity).
- <a id="w-context-map"></a>**context map** — a diagram of how bounded contexts relate and integrate.
- <a id="w-anti-corruption-layer-acl"></a>**anti-corruption layer (ACL)** — a translation layer that stops another context's model from leaking into and polluting yours.
- <a id="w-aggregate"></a>**aggregate** — a cluster of objects treated as one unit for consistency (tactical DDD).

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 8 — A Map of Architectural Styles →](lesson-08-styles-overview){: .btn .btn-primary }
