---
title: "Lesson 10 — Layered, Hexagonal & Clean Architecture"
nav_order: 2
parent: "Phase 3: Architectural Styles"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 10: Layered, Hexagonal & Clean Architecture

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

This is a *family* of styles with one shared goal: **keep the business logic
independent of the plumbing**, so the plumbing can change (or be tested) without
touching the domain. The naive layered architecture almost achieves this — and then
trips on one detail that hexagonal/clean architecture fixes.

**Naive layering** is the trap. Presentation sits on top of Business, which sits
on top of Data — and each layer *depends downward*, which means the domain
depends on the database. The consequence is that database details leak upward
into the core: your business logic imports ORM types, and the thing you most
want to keep stable is coupled to the thing most likely to change.

**Hexagonal architecture** — ports and adapters — inverts that. The **domain**
sits in the middle and depends on **nothing external**. It defines **ports**,
which are interfaces expressing what it needs: "I require a repository that can
save an order." **Adapters** on the outside implement those ports — a Postgres
adapter, a Stripe adapter, an SMTP adapter — and a driving adapter on the other
side translates incoming HTTP into calls on the domain.

The whole idea compresses into one rule: **dependencies point inward, toward
the domain.** The database, the web framework, and the payment provider all
become details plugged into the core, replaceable without touching the business
logic — which is also what makes the domain testable without any of them
running.

Here is what the coupled version looks like in practice:

In naive layering, the business layer calls the data layer, which means the domain
*depends on* the database/ORM — so a database change ripples up into your core business
rules, and you can't test the domain without a database. **Dependency inversion**[°](#w-dependency-inversion) flips
this: the domain defines an *interface* (a "**port**[°](#w-port)") for what it needs ("I need something
that can save an Order"), and the database **adapter**[°](#w-adapter) *implements* that interface. Now the
dependency points *from* the database *toward* the domain — the domain depends on
nothing, and the database is a swappable, mockable detail.

## Going Deeper

**Hexagonal = ports & adapters (Alistair Cockburn).** Picture the domain in the center
of a hexagon. Its edges are **ports** — interfaces the domain owns. **Adapters** on the
outside plug real technology into those ports. Two kinds: *driving* (primary) adapters
call *into* the domain (a REST controller, a CLI, a message consumer — they drive the
app), and *driven* (secondary) adapters are called *by* the domain through ports it
defines (a database repository, an email sender, a payment gateway). The hexagon shape
just means "many ways in, many ways out, all through interfaces" — the number six isn't
special.

**Clean / Onion architecture = the same idea, concentric.** Robert Martin's Clean
Architecture and Jeffrey Palermo's Onion Architecture are the same principle drawn as
concentric circles: the domain entities at the center, use cases around them,
interface adapters outside that, frameworks and drivers at the very edge — with **The
Dependency Rule**: *source-code dependencies point only inward*. Inner circles know
nothing of outer ones. Different books, different diagrams, one idea: the domain is the
stable center, and everything volatile (DB, web, UI, external services) is a detail on
the outside that depends inward.

{: .note }
> **What this buys, and what it costs**
> <strong>Buys:</strong> (1) <em>Testability</em> — you can test the domain with no
> database, no web server, no network, by plugging in fake adapters; tests are fast and
> focused on business rules. (2) <em>Swappable **infrastructure**[°](#w-infrastructure)</em> — change Postgres for
> DynamoDB, REST for gRPC, or a real payment provider for a test double, by writing a
> new adapter, without touching the domain. (3) <em>Domain clarity</em> — the business
> rules live in one place, uncontaminated by framework code, so they're easy to read and
> reason about. <strong>Costs:</strong> (1) <em>**Indirection**[°](#w-indirection) and ceremony</em> — more
> interfaces, more mapping between domain objects and DB/DTO models, more files;
> simple CRUD can feel buried under abstraction. (2) <em>A learning curve</em> — the
> inverted dependencies confuse people used to "the service calls the repository." (3)
> <em>Over-application risk</em> — for a small CRUD app with no real domain logic, the
> full ceremony is pure overhead. It's a trade-off (Lesson 2), not a universal good.

**When it's worth it vs when it's overkill.** The value scales with the *richness and
longevity of the domain logic* and the *volatility of the infrastructure*. Worth it: a
system with substantial, valuable business rules that must outlive its current database/
framework, or must be thoroughly tested, or integrates swappable external systems. Overkill:
a thin CRUD app that's basically "put form data in a table" — there's no domain to
protect, so the layers of indirection guard nothing and just add friction. The mature
judgment is applying the *principle* (keep the domain independent of volatile details) at
a *dose* matched to how much domain there is — you can have "the domain doesn't import the
ORM" without ten layers of ceremony.

**This nests inside other styles.** Hexagonal/clean isn't an alternative to monolith or
microservices — it's how you structure the *inside* of a deployable. A modular monolith's
modules can each be hexagonal; a microservice is often internally hexagonal. It's the
"in the small" companion to the "in the large" styles of this phase, and it's the
structural expression of Lesson 5's rule that dependencies should point from volatile
details toward the stable core.

---

## Lab — Design Exercise

**The situation:** Here's a common layered design for an order feature:

```
OrderController (HTTP)
   → OrderService.placeOrder(dto)
        → new OrderRepository()        // concrete class, wraps the ORM
        → orderRepo.save(order)        // OrderService imports the ORM's types
        → new StripeClient().charge()  // concrete payment client
        → new SmtpMailer().send()      // concrete mailer
```

`OrderService` (the business logic) directly imports and depends on the ORM, the Stripe
SDK, and the SMTP library. To unit-test `placeOrder`, you currently need a real database
and network.

**Redraw this as ports & adapters.** Identify the domain, the ports the domain should
define (and which are *driving* vs *driven*), and the adapters. Show which way the
dependencies now point. Then honestly name what got *easier* and what got *heavier* — and
say in what situation the original simpler design would have been the better call.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is inverting the dependencies and being honest about the trade. A strong
answer:
<br><br>
<strong>The domain</strong> is the order-placing business logic — the rules about what
placing an order <em>means</em> (validate, price, charge, confirm), independent of
<em>how</em> it's stored, charged, or notified. This is the center; it should import
nothing from the ORM, Stripe, or SMTP.
<br><br>
<strong>Ports the domain defines (interfaces it owns):</strong>
<ul>
<li><code>OrderRepository</code> (driven port) — "save/load an Order." The domain
declares it; it knows nothing of Postgres or the ORM.</li>
<li><code>PaymentGateway</code> (driven port) — "charge this amount to this customer."
No mention of Stripe.</li>
<li><code>Notifier</code> (driven port) — "send an order confirmation." No mention of
SMTP.</li>
<li>A <code>PlaceOrder</code> use-case interface (driving port) — the entry point the
outside world calls to drive the domain.</li>
</ul>
<strong>Adapters (concrete, on the outside):</strong>
<ul>
<li><em>Driving:</em> <code>OrderController</code> (HTTP) — a primary adapter that
translates an HTTP request into a call on the <code>PlaceOrder</code> port. (Could be
swapped/added: a CLI adapter, a message-consumer adapter — same domain, new way in.)</li>
<li><em>Driven:</em> <code>PostgresOrderRepository</code> implements
<code>OrderRepository</code> using the ORM; <code>StripePaymentGateway</code> implements
<code>PaymentGateway</code>; <code>SmtpNotifier</code> implements <code>Notifier</code>.
Each wraps a real technology behind the port.</li>
</ul>
<strong>Dependency direction:</strong> now the domain depends on <em>nothing external</em>
— it only references its own port interfaces. The ORM, Stripe, and SMTP adapters depend
<em>inward</em> on the domain's interfaces (they implement them). The arrows all point
toward the center. Wiring (which concrete adapter implements which port) happens at the
edge, via dependency injection at startup.
<br><br>
<strong>What got easier:</strong> (1) <em>Testing</em> — <code>placeOrder</code> can now
be unit-tested with in-memory fake adapters (a fake repo, a fake gateway that records the
charge, a fake notifier), no database or network, fast and deterministic; you test the
<em>business rules</em> directly. (2) <em>Swapping infrastructure</em> — move to a
different database or payment provider by writing one new adapter, no domain change; the
Stripe→another-provider migration is contained. (3) <em>Domain clarity</em> — the order
rules live in one clean place, not smeared through ORM and SDK calls; the "position"
coupling of <code>charge(amount, card)</code> is gone behind a typed port.
<br><br>
<strong>What got heavier:</strong> more interfaces and more files (three ports, three
adapters, a use case, plus mapping between domain objects and the ORM's entities and the
HTTP DTOs); more indirection to trace ("where's the real save?" → follow the port to the
adapter); a wiring/DI setup at the edge; and a learning curve for anyone expecting "the
service just calls the repository." For a rich order domain that must be tested and
outlive its current tech, that's a good trade.
<br><br>
<strong>When the original was the better call:</strong> if this were a <em>thin
CRUD</em> feature with essentially no business logic — "take the form, write a row" — then
there's no real domain to protect, the infrastructure is stable, and the ports/adapters
ceremony guards nothing while adding files, indirection, and mapping overhead. There, the
simpler direct design ships faster and is easier to read. The judgment (Lesson 2): apply
the <em>principle</em> — at minimum, don't let the domain import the ORM/SDK directly, so
you keep testability — at a <em>dose</em> proportional to how much genuine domain logic and
infrastructure volatility exists. Full hexagonal for a rich, long-lived domain; a light
touch (or nothing) for thin CRUD. The anti-pattern in <em>both</em> directions:
hexagonalizing a CRUD app (over-engineering) and letting a rich domain import the database
directly (under-engineering — untestable, DB changes ripple into business rules).
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Hexagonal / ports & adapters (the original) | Alistair Cockburn — <https://alistair.cockburn.us/hexagonal-architecture/> |
| Clean Architecture & the Dependency Rule | Robert C. Martin — <https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html> |
| Onion Architecture | Jeffrey Palermo — <https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/> |
| Dependency Inversion Principle | <https://en.wikipedia.org/wiki/Dependency_inversion_principle> |
| Layered architecture & its trade-offs | *Fundamentals of Software Architecture*, Richards & Ford (Ch. 10) |

---

## Checkpoint

**Q1.** In a naive layered architecture the business layer calls the data layer. What's
the hidden problem with that dependency direction, and how does dependency inversion (the
core of hexagonal/clean) fix it?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The hidden problem: "business layer calls data layer" means the <strong>business/domain
logic depends on the database/data layer</strong> — the dependency arrow points from your
most valuable, most stable code (the business rules) toward your most volatile detail (the
specific database, ORM, schema). Two bad consequences follow. (1) <em>Changes leak
upward:</em> a database or ORM change ripples <em>into</em> the domain, because the domain
references the data layer's types and calls — so the thing that should be most protected is
coupled to the thing most likely to change (a violation of Lesson 5's rule that
dependencies should point from volatile toward stable). (2) <em>Untestable in isolation:</em>
because the domain depends on the real data layer, you can't test the business rules without
a real database — tests become slow, brittle, and infrastructure-bound.
<br><br>
<strong>Dependency inversion fixes it</strong> by flipping the arrow. Instead of the domain
depending on the concrete data layer, the <em>domain defines an interface</em> (a port —
e.g., <code>OrderRepository</code>) describing <em>what it needs</em> ("save/load an
Order") in its own terms, knowing nothing about Postgres or the ORM. The data layer then
<em>implements</em> that interface (a <code>PostgresOrderRepository</code> adapter). Now the
concrete database code depends on the domain's interface — the dependency points
<em>inward, from the detail toward the domain</em> — and the domain depends on nothing
external, only on its own abstraction. Result: (1) the database is a swappable detail behind
the port, so DB/ORM changes stay in the adapter and never touch the domain; and (2) you can
test the domain by plugging in a fake in-memory adapter, no database needed. The word
"inversion" refers exactly to this: the naive dependency (domain → database) is
<em>inverted</em> (database → domain's interface ← domain) by introducing the abstraction
that both sides now point at, with the domain owning it.
</details>

**Q2.** Hexagonal/clean architecture is powerful but frequently over-applied. Give the
factors that make it worth the ceremony, the situation where it's overkill, and the mature
"apply the principle at the right dose" middle ground.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Worth the ceremony when:</strong> (1) the system has <em>rich, valuable business
logic</em> — real domain rules worth isolating, testing thoroughly, and keeping legible
(not just moving data in and out of tables); (2) the domain must <em>outlive its current
infrastructure</em> — a long-lived system where the database, framework, or external
providers may well change, so protecting the core from those swaps pays off over years; (3)
<em>testability matters</em> — you want fast, isolated tests of business rules without
standing up databases and networks; (4) you integrate <em>swappable or external</em>
systems (payment providers, third-party APIs) you want behind adapters/anti-corruption
layers. The more of these hold, the more the indirection earns its cost.
<br><br>
<strong>Overkill when:</strong> the system is a <em>thin CRUD</em> app — essentially "take
the request, write a row, read it back" — with little or no genuine domain logic and stable
infrastructure. There's nothing valuable to protect at the center, so the ports, adapters,
mapping layers, and DI wiring guard an empty core: they add files, indirection, and mapping
overhead while buying almost nothing. Full hexagonal here is over-engineering (Lesson 35),
slowing delivery and obscuring simple code behind abstraction.
<br><br>
<strong>The mature middle ground:</strong> treat it as a <em>principle</em> to apply at a
<em>dose</em>, not an all-or-nothing framework. The principle — <em>don't let the domain
depend on volatile infrastructure; put an interface between them</em> — can be honored
lightly: at minimum, keep the business logic from directly importing the ORM/SDK (define a
repository/gateway interface), which alone preserves testability and swappability, without
necessarily building the full concentric-circles ceremony, elaborate DTO-to-entity mapping,
and layer-upon-layer of indirection. Scale the ceremony to the domain: a rich, long-lived
core gets the full treatment; a modestly-logic'd service gets a light interface-at-the-edge;
pure CRUD gets little or none. The failure is treating hexagonal as a religion applied
uniformly — which over-engineers the CRUD and (if people rebel against the ceremony)
sometimes leaves the rich domains under-protected. Match the structure to how much domain
and volatility actually exist — which is just Lesson 2's "everything is a trade-off, tied to
context" applied to internal structure.
</details>

---

## Homework

Look at one component/service in your current system with real business logic. Trace its
dependencies: does the business logic directly import and depend on the database/ORM, the
web framework, or external SDKs — or is it isolated behind interfaces? Try the test: *could
you unit-test the core logic with no database and no network?* If not, that's the coupling to
fix. Sketch the ports the domain would define and the adapters that would implement them, and
decide honestly what dose is right — full hexagonal, a light interface-at-the-edge, or (if
it's thin CRUD) leaving it direct. Note what testing pain the current coupling causes today.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise makes the abstract dependency rule concrete on real code. A strong response:
<br><br>
<strong>Runs the testability litmus test.</strong> "Can I unit-test the core logic with no
database and no network?" is the sharpest single diagnostic, and the common finding is
<em>no</em> — the business logic imports the ORM entities and calls the database directly (or
news up an SDK client inline), so testing it requires a real database/network, which is why
the tests are slow, flaky, or missing altogether. Naming that concrete pain (slow test suite,
tests that need a DB container, logic that's effectively untested because testing it is too
hard) grounds the whole lesson: the coupling isn't a theoretical impurity, it's why testing
hurts today.
<br><br>
<strong>Identifies the ports and the dose.</strong> A good answer sketches the specific
interfaces the domain would own (a repository, a gateway, a notifier — whatever volatile
things it currently reaches for) and which are driving vs driven, and then makes the
<em>dose</em> judgment honestly: if there's substantial, long-lived domain logic and volatile
infrastructure, the full ports-and-adapters treatment is justified and would immediately buy
testability; if it's mostly CRUD with a bit of logic, the right move is the light touch —
extract just enough interface that the domain no longer imports the ORM/SDK, restoring
testability without the full ceremony; if it's pure CRUD, leaving it direct is the correct,
non-dogmatic call. The mark of good judgment is <em>not</em> concluding "hexagonalize
everything" but matching the structure to the amount of domain and volatility — and, ideally,
identifying the <em>one</em> component where inverting the database dependency would most
improve testability, as a concrete, high-value first step rather than a sweeping rewrite.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-layer-tier"></a>**layer / tier** — a horizontal slice by technical role (presentation, business, data); a *tier* is a physically separate layer.
- <a id="w-dependency-inversion"></a>**dependency inversion** — making high-level code depend on an abstraction, and the low-level detail depend on that same abstraction, so the arrow points *away* from the details.
- <a id="w-port"></a>**port** — an interface the domain defines for something it needs (a "driven" port) or something that drives it (a "driving" port).
- <a id="w-adapter"></a>**adapter** — a concrete implementation of a port that plugs a real technology (a database, a web framework) into the domain.
- <a id="w-domain-business-logic"></a>**domain / business logic** — the core rules of the application, ideally independent of any framework or infrastructure.
- <a id="w-infrastructure"></a>**infrastructure** — the plumbing: databases, message brokers, web servers, external APIs.
- <a id="w-indirection"></a>**indirection** — an extra layer of abstraction between two things; buys flexibility, costs directness.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 11 — Microservices →](lesson-11-microservices){: .btn .btn-primary }
