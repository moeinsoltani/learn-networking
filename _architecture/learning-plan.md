---
title: Learning Plan
nav_order: 2
---

# Software Architecture Learning Plan

A curriculum for the **Senior Software Developer → Software Architect** transition.
You already know how to build software well. This track is about the *other* job:
making the significant, hard-to-reverse decisions that shape a whole system —
reasoning about quality attributes and trade-offs, choosing structure and styles,
taming distributed systems and data, and then documenting, evaluating, and evolving
what you've decided. It ends with an end-to-end design capstone.

**Lab format:** this track has no terminal. Each lab is a **design exercise** — a
realistic situation ("your monolith's checkout is the bottleneck; the business
wants Black Friday to hold") where you sketch a design, name the trade-offs, or
write the decision, then reveal a model answer that explains the reasoning, the
common mistakes, and the phrasing an architect uses. The point is not one right
answer — it is learning to *reason like an architect*: everything is a trade-off,
and the job is making the trade-offs explicit.

**How this track relates to the others.** Architecture sits on top of the systems
tracks and next to leadership:
- **[Engineering Leadership]({{ '/leadership/learning-plan.html' | relative_url }})** teaches the *people* judgment (influence, delegation, running design reviews). This track teaches the *technical* judgment. An architect needs both; they cross-link throughout (ADRs, trade-offs, design reviews).
- **[Security & Identity]({{ '/security/learning-plan.html' | relative_url }})**, **[Linux Networking]({{ '/networking/learning-plan.html' | relative_url }})**, and **[Operating Systems]({{ '/os/learning-plan.html' | relative_url }})** are the ground truth beneath the boxes-and-arrows. This track cites them where a diagram hides real mechanics.

**Core references** (cited throughout the Further Reading tables): *Fundamentals of
Software Architecture* and *Software Architecture: The Hard Parts* (Richards &
Ford), *Designing Data-Intensive Applications* (Kleppmann), *Building Evolutionary
Architectures* (Ford, Parsons, Kua), *Domain-Driven Design* (Evans) and
*Implementing DDD* (Vernon), *Release It!* (Nygard), *Building Microservices*
(Newman), *Documenting Software Architectures* (Clements et al.), the C4 model
(c4model.com), the AWS/Azure architecture centers, and Martin Fowler's
bliki (martinfowler.com).

---

## [Phase 1: The Architect's Role & Mindset](lessons/phase-01-role-mindset.html)

### [Lesson 1: What a software architect actually does](lessons/lesson-01-what-architects-do.html)
**Goal:** Redefine your output: from features you build to structural decisions that shape the whole system.
**Topics:** Architecture as "the decisions that are hard to change"; the architect's real deliverables (decisions, constraints, guardrails, shared understanding — not just diagrams); the many flavors (application, solution, enterprise, "architect" as a role vs an activity); why the title is less important than the work; the myth of the ivory-tower architect vs the architect who still touches the ground.
**Design exercise:** Given a team's list of recent decisions, sort which were "architectural" (significant + hard to reverse) and which were not — and defend the line you drew.

### [Lesson 2: Architectural thinking & the nature of trade-offs](lessons/lesson-02-architectural-thinking.html)
**Goal:** Internalize the one sentence that defines the job: *everything in architecture is a trade-off*.
**Topics:** "It depends" as the honest answer (and how to make it useful); breadth over depth; seeing the "-ilities" behind every requirement; second-order effects; the difference between a *decision* and a *guess*; making trade-offs explicit instead of implicit; why "best practice" is context-free and therefore suspect.
**Design exercise:** For "should we add a cache here?", write the trade-off analysis — what it buys, what it costs, and what would have to be true for it to be right.

### [Lesson 3: Architecture characteristics — the "-ilities"](lessons/lesson-03-quality-attributes.html)
**Goal:** Learn the vocabulary of quality attributes and why they, not features, drive architecture.
**Topics:** Functional requirements vs *quality attributes* (scalability, availability, performance, security, maintainability, testability, deployability, …); operational vs structural vs cross-cutting characteristics; the "you can't have them all" reality (they conflict); making them measurable (a scenario with a stimulus, response, and measure); why an architecture is *for* its characteristics.
**Design exercise:** Turn three vague requirements ("it should be fast", "it should be reliable") into measurable quality-attribute scenarios.

### [Lesson 4: Architectural drivers & the "architecturally significant"](lessons/lesson-04-architectural-drivers.html)
**Goal:** Find the small set of things that actually shape the architecture, and ignore the rest — for now.
**Topics:** The four drivers (quality attributes, key functional requirements, constraints, and architectural concerns); Architecturally Significant Requirements (ASRs); prioritizing (important × difficult); constraints you can't negotiate (regulation, team, budget, deadline, existing systems); the "last responsible moment" for a decision; distinguishing drivers from noise.
**Design exercise:** From a one-page product brief, extract the 5 architecturally significant requirements and justify why the other ten don't shape the architecture.

---

## [Phase 2: Foundations of Structure](lessons/phase-02-foundations.html)

### [Lesson 5: Coupling, cohesion & connascence](lessons/lesson-05-coupling-cohesion.html)
**Goal:** Master the two forces that govern every structural decision you'll ever make.
**Topics:** Cohesion (things that change together live together) and coupling (things that shouldn't need each other, don't); afferent vs efferent coupling; connascence as a precise vocabulary for coupling (static: name, type, position; dynamic: execution, timing, value); the goal — high cohesion, low & *loose* coupling; why "no coupling" is a fantasy (you're choosing *which* coupling).
**Design exercise:** Given two module designs for the same feature, identify the coupling in each and argue which is more changeable — and why.

### [Lesson 6: Modularity & component boundaries](lessons/lesson-06-modularity-boundaries.html)
**Goal:** Decide where the lines go — the single highest-leverage act of architecture.
**Topics:** What a component/module is; drawing boundaries by change axis and by team; the "align boundaries with the domain, not the technical layer" principle; the dependency direction rule (dependencies point toward stability/abstraction); the cost of a wrong boundary (it leaks everywhere); why a boundary you can't defend at runtime is only a suggestion.
**Design exercise:** Given a tangled feature spanning "orders, payments, inventory, notifications", propose the component boundaries and the dependencies between them.

### [Lesson 7: Domain-Driven Design essentials](lessons/lesson-07-domain-driven-design.html)
**Goal:** Use the domain — not the database or the framework — as the primary source of structure.
**Topics:** Ubiquitous language; bounded contexts (the same word means different things in different contexts — and that's fine); context mapping (partnership, customer-supplier, conformist, anti-corruption layer); strategic vs tactical DDD; subdomains (core / supporting / generic) and where to spend your best people; why bounded contexts are the natural seams for services.
**Design exercise:** For an e-commerce system, identify three bounded contexts, show where "Customer" means different things, and place an anti-corruption layer.

### [Lesson 8: A map of architectural styles](lessons/lesson-08-styles-overview.html)
**Goal:** Get the lay of the land before diving in — the menu of styles and what each optimizes for.
**Topics:** Monolithic vs distributed as the first fork; the major styles (layered, modular monolith, pipeline, microkernel/plugin, service-based, event-driven, microservices, space-based) and the one characteristic each is *for*; the "big ball of mud" as the default you get by not choosing; hybrids are the norm; how to *choose* a starting style from the drivers.
**Design exercise:** For three different products (a startup MVP, a bank's core ledger, a media-streaming backend), pick a starting architectural style and justify it from the drivers.

---

## [Phase 3: Architectural Styles](lessons/phase-03-styles.html)

### [Lesson 9: Monoliths & the modular monolith](lessons/lesson-09-monoliths.html)
**Goal:** Take the monolith seriously — it's the right default far more often than the internet admits.
**Topics:** The monolith's real strengths (simplicity, one deploy, transactions, refactoring, no network); why they *rot* (missing internal boundaries → big ball of mud); the **modular monolith** as the antidote (enforced internal modules, one deployable); "monolith first" (Fowler); when a monolith genuinely stops fitting; how a well-modularized monolith makes a later split cheap.
**Design exercise:** A 3-year-old monolith is a tangle. Propose how to introduce module boundaries *without* splitting into services — and what would enforce them.

### [Lesson 10: Layered, hexagonal & clean architecture](lessons/lesson-10-layered-hexagonal.html)
**Goal:** Understand the family of "keep the domain independent of the plumbing" styles.
**Topics:** Layered/n-tier (and its trap: the domain depending on the database); the dependency inversion that fixes it; **hexagonal / ports & adapters** (the domain at the center, adapters at the edges); clean/onion architecture as the same idea; where the value is (testability, swappable infrastructure) and where the cost is (indirection, ceremony); when it's overkill.
**Design exercise:** Redraw a layered design where business logic calls the ORM directly as a ports-and-adapters design — and name what got easier and what got heavier.

### [Lesson 11: Microservices](lessons/lesson-11-microservices.html)
**Goal:** Understand microservices honestly — the benefits, and the enormous bill that comes with them.
**Topics:** What microservices actually are (independently deployable, owned by a team, bounded-context-sized); what they buy (independent deploy, scale, and tech; team autonomy; fault isolation); the cost (distributed-systems tax — everything in Phase 4); "you must be this tall" prerequisites (deployment automation, observability, org maturity — Fowler's premium); the distributed monolith as the worst outcome; microservices as an *organizational* solution as much as technical.
**Design exercise:** A team of 6 wants to split their working monolith into 20 microservices for a product with modest traffic. Advise them — and write the honest pushback.

### [Lesson 12: Event-driven architecture](lessons/lesson-12-event-driven.html)
**Goal:** Learn the style that trades control-flow legibility for decoupling and scale.
**Topics:** Events vs commands vs messages; the broker/topology (pub-sub, brokers, event streams); choreography vs orchestration; the huge upside (temporal decoupling, extensibility, scale) and the sharp edges (eventual consistency, no linear flow to read, debugging across hops, error handling and dead letters); event notification vs event-carried state transfer vs event sourcing; when events fit and when they hide the system from you.
**Design exercise:** Redesign a synchronous "place order → charge → reserve stock → email" chain as event-driven — and enumerate the new failure modes you just signed up for.

### [Lesson 13: Service granularity & decomposition](lessons/lesson-13-decomposition.html)
**Goal:** Answer the question that sinks most microservice efforts: *how big is a service?*
**Topics:** Granularity disintegrators (reasons to split: scale, fault isolation, differing change rate, security, team) vs integrators (reasons to keep together: transactions, data dependency, shared workflow, chatty calls); the sweet spot is a balance, not "smallest possible"; decomposition approaches (by bounded context, by capability; component-based decomposition of a monolith first); how getting granularity wrong creates either a distributed monolith or a nano-service swamp.
**Design exercise:** Given four candidate services, apply the disintegrator/integrator forces to decide which to split and which to merge.

---

## [Phase 4: Distributed Systems](lessons/phase-04-distributed.html)

### [Lesson 14: The fallacies of distributed computing](lessons/lesson-14-fallacies.html)
**Goal:** Unlearn the assumptions that quietly poison every naive distributed design.
**Topics:** The eight fallacies (network is reliable, latency is zero, bandwidth is infinite, network is secure, topology doesn't change, one admin, transport cost is zero, network is homogeneous); each fallacy mapped to a real production failure; the mental shift — a remote call is *nothing like* a local call; why "it's just a function call over the network" is the most expensive lie in our field.
**Design exercise:** Take a design that assumes calls always succeed instantly, and annotate every remote call with the fallacy it's ignoring and the mitigation it needs.

### [Lesson 15: CAP, PACELC & consistency models](lessons/lesson-15-cap-consistency.html)
**Goal:** Reason precisely about the consistency-vs-availability trade-off instead of hand-waving it.
**Topics:** CAP stated correctly (during a *partition*, choose consistency or availability — not "pick 2 of 3 always"); PACELC (and *else*, latency vs consistency, the choice you make every day even without partitions); the consistency spectrum (strong, linearizable, causal, read-your-writes, eventual); what "eventual consistency" actually means for a user; matching the model to the business (a bank balance vs a like count).
**Design exercise:** For three data items (account balance, product review, "last seen" timestamp), choose a consistency model and justify it against the business cost of being wrong.

### [Lesson 16: Communication styles — sync, async, REST, gRPC, messaging](lessons/lesson-16-communication.html)
**Goal:** Choose how services talk — the decision that determines coupling and resilience.
**Topics:** Synchronous request/response (REST, gRPC) vs asynchronous messaging (queues, streams); the coupling difference (temporal coupling: the callee must be up *now*); protocol trade-offs (REST's ubiquity vs gRPC's performance and contracts vs GraphQL's client flexibility); message queues (point-to-point) vs pub-sub vs event streams; the "prefer async where you can tolerate it" heuristic and its limits; the orchestration vs choreography choice again, at the wire level.
**Design exercise:** For four inter-service interactions with different latency and reliability needs, choose sync-REST, sync-gRPC, or async-messaging for each and justify.

### [Lesson 17: Distributed data — transactions, sagas, outbox, idempotency](lessons/lesson-17-distributed-data.html)
**Goal:** Face the hardest part: you gave up the database transaction the moment you split the data.
**Topics:** Why 2-phase commit doesn't scale (and the coupling it reintroduces); the **saga** pattern (orchestrated vs choreographed) and compensating transactions; the dual-write problem and the **transactional outbox**; **idempotency** as survival (retries are guaranteed, so every consumer must be idempotent); delivery semantics (at-least-once is the realistic default, exactly-once is mostly a lie); eventual consistency as a business conversation, not just a technical one.
**Design exercise:** Design "book flight + hotel + car; if any fails, undo the rest" as a saga — with the compensating actions and the idempotency keys spelled out.

### [Lesson 18: Reliability & resilience patterns](lessons/lesson-18-resilience.html)
**Goal:** Design for the failures you *know* are coming, because in a distributed system they always come.
**Topics:** Failure is normal, not exceptional; timeouts (the one everyone forgets); retries with backoff + jitter (and why naive retries cause retry storms); the **circuit breaker** (stop hammering a downed dependency); **bulkheads** (isolate so one failure can't sink everything); graceful degradation and fallbacks; load shedding and backpressure; the difference between fault, error, and failure (Nygard); designing the *blast radius*.
**Design exercise:** A payment service's slow dependency is taking down the whole checkout via thread exhaustion. Prescribe the resilience patterns, in priority order, and explain what each contains.

---

## [Phase 5: Data & Scale](lessons/phase-05-data-scale.html)

### [Lesson 19: Choosing a data store — polyglot persistence](lessons/lesson-19-choosing-datastore.html)
**Goal:** Match the store to the access pattern instead of reaching for the same database every time.
**Topics:** The data-model families (relational, document, key-value, wide-column, graph, search, time-series) and the access pattern each is *for*; the questions that decide (read/write ratio, query shape, consistency need, scale, relationships); "start with Postgres" as an honest default and when to deviate; polyglot persistence and its operational cost; why the *access pattern*, not the data, should drive the choice; NoSQL's real trade (flexibility/scale for lost joins and transactions).
**Design exercise:** For a system with a user profile, a product catalog with faceted search, a shopping-cart, and a social graph, pick a store for each and justify from the access pattern.

### [Lesson 20: Event sourcing & CQRS](lessons/lesson-20-event-sourcing-cqrs.html)
**Goal:** Understand two powerful, frequently-misapplied patterns — and when they earn their complexity.
**Topics:** **CQRS** (separate the write model from the read model — different shapes, different stores, scaled independently); **event sourcing** (store the sequence of events, not the current state; state is a fold over events); the huge benefits (audit, temporal queries, rebuild-any-view, natural fit with events) and the real costs (eventual consistency, versioning events forever, no easy "just query the table", steep learning curve); why they're independent (you can do either alone); the "you probably don't need event sourcing" honesty.
**Design exercise:** Decide whether a bank ledger, a CMS, and an IoT telemetry pipeline should use event sourcing — and defend each yes/no from the trade-offs.

### [Lesson 21: Caching strategies](lessons/lesson-21-caching.html)
**Goal:** Wield the sharpest double-edged sword in performance work without cutting yourself.
**Topics:** Why cache (latency, load, cost) and the two hard problems (invalidation, and coherence); caching layers (client, CDN, gateway, application, database); patterns (cache-aside, read-through, write-through, write-behind); TTL vs explicit invalidation; the stampede/thundering-herd problem; staleness as a *deliberate* trade-off tied to a consistency model (Lesson 15); when a cache hides a design problem you should fix instead.
**Design exercise:** A product page hammers the database. Design a caching strategy across the layers — and name exactly what staleness each layer introduces and whether the business can tolerate it.

### [Lesson 22: Scaling data — partitioning, sharding, replication](lessons/lesson-22-scaling-data.html)
**Goal:** Scale past the single database — the point where most systems actually break.
**Topics:** Vertical vs horizontal scaling; read replicas (and the replication lag that reintroduces consistency questions); **partitioning/sharding** (by key — and how a bad shard key creates hot spots and cross-shard queries that ruin you); the CAP tax on writes; when to reach for a distributed database vs shard yourself; the "denormalize for the read path" trade; capacity thinking (know your numbers before you scale).
**Design exercise:** A single Postgres is at its write ceiling for a multi-tenant SaaS. Walk the scaling ladder (replica → partition → shard) and choose a shard key, defending it against hot-spot and cross-shard-query risk.

---

## [Phase 6: Cross-Cutting Quality Attributes](lessons/phase-06-cross-cutting.html)

### [Lesson 23: Scalability & performance architecture](lessons/lesson-23-scalability-performance.html)
**Goal:** Design for load deliberately — and know the difference between fast and scalable.
**Topics:** Latency vs throughput (and why optimizing one can hurt the other); scalability as a *shape* (linear? does adding hardware help?); statelessness as the enabler of horizontal scale; the async/queue-based load-leveling pattern; identifying the bottleneck before optimizing (Amdahl, USE method); back-of-the-envelope capacity math; performance as a budget you spend, not a feature you add later; the "premature scaling is a debt too" reminder.
**Design exercise:** Given a latency budget for a request that fans out to four services, allocate the budget across the hops and find where it will blow — before writing code.

### [Lesson 24: Security architecture & threat modeling](lessons/lesson-24-security-architecture.html)
**Goal:** Build security into the structure instead of bolting it on — the architect's non-negotiable.
**Topics:** Defense in depth and the principle of least privilege; the trust boundary as an architectural concept; **threat modeling** (STRIDE, "what can go wrong here?") as a design activity; authN vs authZ placement (gateway? service? both?); secrets and key management as architecture; **zero trust** ("never trust the network"); the blast-radius / segmentation mindset; security as a quality attribute that trades against usability and cost. *(Cross-links to the [Security & Identity]({{ '/security/learning-plan.html' | relative_url }}) track for the mechanics.)*
**Design exercise:** Draw the trust boundaries for a system with a public API, internal services, and a third-party payment provider, and run a quick STRIDE pass on the riskiest boundary.

### [Lesson 25: Observability architecture](lessons/lesson-25-observability.html)
**Goal:** Design a system you can *understand in production* — because a distributed system you can't see is a system you can't operate.
**Topics:** Monitoring (known-unknowns) vs observability (unknown-unknowns); the three pillars (logs, metrics, traces) and what each answers; **distributed tracing** and the correlation/trace ID as an architectural requirement (thread it from the edge); structured logging; the RED and USE method for metrics; SLIs/SLOs/error budgets as the language of reliability with the business; why observability must be designed in, not added after an outage.
**Design exercise:** For a 5-service request path, specify what you'd instrument (traces, key metrics, log correlation) so that "checkout is slow" can be diagnosed in minutes, not hours.

### [Lesson 26: API design & management](lessons/lesson-26-api-design.html)
**Goal:** Treat the API as a contract and a product — because it's the most expensive thing to change once others depend on it.
**Topics:** APIs as one-way doors (public contracts you can't unship); designing for the consumer; REST resource design, and when RPC/gRPC or GraphQL fit better; **versioning** strategies (URI, header, and the "never break v1" discipline); backward/forward compatibility (expand-contract, tolerant reader); the API **gateway** (cross-cutting concerns at the edge: auth, rate limiting, routing); contract testing so services can evolve independently; documentation as part of the deliverable.
**Design exercise:** You must add a required field to a widely-used public API without breaking existing clients. Design the compatible evolution and the deprecation path.

### [Lesson 27: Cloud & deployment architecture](lessons/lesson-27-cloud-deployment.html)
**Goal:** Design how the system is built, deployed, and run — deployability is a first-class quality attribute now.
**Topics:** The **twelve-factor app** as a baseline; containers and orchestration (why they exist, what they change architecturally); infrastructure as code and immutable infrastructure; deployment strategies (blue-green, canary, rolling) and what each buys; stateless services + externalized state; managed services and the build-vs-rent trade at the infra layer; cost as an architectural concern (the cloud bill is a design output); multi-region and the availability/complexity trade. *(Grounded by the [Virtualization]({{ '/virtualization/learning-plan.html' | relative_url }}) and [Networking]({{ '/networking/learning-plan.html' | relative_url }}) tracks.)*
**Design exercise:** Take a stateful monolith deployed by hand and design its path to a twelve-factor, container-deployed service with zero-downtime releases — naming the hardest step.

---

## [Phase 7: Documenting, Evaluating & Evolving Architecture](lessons/phase-07-documenting-evolving.html)

### [Lesson 28: Documenting architecture — the C4 model & views](lessons/lesson-28-documenting-c4.html)
**Goal:** Communicate a design so others can build it, review it, and remember it — the deliverable that outlives you.
**Topics:** Why "one giant diagram" fails (it serves no audience); **views for audiences** (the 4+1 idea); the **C4 model** (Context → Container → Component → Code — zoom levels, not more boxes); diagrams that carry meaning (a legend, consistent notation, one abstraction level per diagram); text-based diagramming (diagrams-as-code) for versioning; the arc42 / views-and-beyond idea of a documentation package; documenting the *why*, not just the *what*.
**Design exercise:** For a system you know, sketch the C4 Context and Container diagrams (in words/ASCII) and identify which stakeholder each serves.

### [Lesson 29: ADRs & capturing decisions](lessons/lesson-29-adrs.html)
**Goal:** Make your decisions legible and durable so the *why* survives past the meeting. *(Deepens [Leadership Lesson 7]({{ '/leadership/lessons/lesson-07-adrs.html' | relative_url }}) with the architect's lens.)*
**Topics:** The ADR structure (context / decision / consequences / alternatives-considered); one-way vs two-way doors setting the deliberation budget; recording the *rejected* options (the most valuable part); status lifecycle (proposed / accepted / superseded); keeping ADRs next to the code; ADRs as onboarding, as anti-relitigation, and as the raw material for evaluating an architecture later.
**Design exercise:** Write a full ADR (with alternatives and consequences) for a real architectural decision from your past — then critique whether the deliberation matched the door-type.

### [Lesson 30: Evaluating architecture — ATAM, trade-offs & fitness functions](lessons/lesson-30-evaluating.html)
**Goal:** Judge an architecture *before* you've built it (and continuously after) instead of discovering the flaws in production.
**Topics:** Scenario-based evaluation (ATAM in spirit: quality-attribute scenarios → find the sensitivity points, trade-off points, and risks); the "what would break this?" review stance; **architecture fitness functions** (automated, continuous tests of a quality attribute — e.g., "no cyclic dependencies", "p99 < 200ms", "no service calls the DB of another"); making the implicit architecture testable; reviewing others' architectures kindly and usefully (cross-link to leadership design reviews).
**Design exercise:** Given a proposed design and its top three quality attributes, run a lightweight evaluation: write two scenarios per attribute and identify the biggest risk and one trade-off point.

### [Lesson 31: Evolutionary architecture & managing change](lessons/lesson-31-evolutionary.html)
**Goal:** Design for change as a first-class requirement, because the one certainty is that the requirements will move.
**Topics:** Architecture as a verb (guided, incremental change) not a noun (a fixed blueprint); the "last responsible moment" and deferring irreversible decisions; fitness functions as the guardrails that let you evolve safely; keeping options open (the real value of good boundaries and loose coupling); avoiding the big-rewrite trap; the "just enough, just in time" architecture; postponing accidental complexity.
**Design exercise:** A design bakes in a specific message broker everywhere. Show how to keep the decision reversible, and write the fitness function that would catch a leak of the broker into the domain.

### [Lesson 32: Modernizing legacy — the strangler fig & friends](lessons/lesson-32-modernization.html)
**Goal:** Change a running system without a big-bang rewrite — the situation you'll actually inherit.
**Topics:** Why big-bang rewrites usually fail (the second-system effect, the moving target, the risk); the **strangler fig** (grow the new around the old, redirect route by route, delete the old when starved); the branch-by-abstraction and anti-corruption-layer tools; carving a service out of a monolith safely (seams, the database is the hard part); parallel run and reconciliation; sequencing by risk and value; knowing when *not* to modernize.
**Design exercise:** Plan the strangler-fig extraction of a "notifications" capability out of a legacy monolith — the sequence, the seam, and how you'd handle the shared database.

---

## [Phase 8: The Architect in Practice](lessons/phase-08-in-practice.html)

### [Lesson 33: Build vs buy & technology selection](lessons/lesson-33-build-vs-buy.html)
**Goal:** Make the "should we even build this?" and "which technology?" calls with judgment instead of résumé-driven or hype-driven reflexes.
**Topics:** Build vs buy vs open-source vs don't (opportunity cost as the lens); is it your *core* domain (build) or generic (buy)?; total cost of ownership (the sticker price is the smallest part); evaluating technology honestly (fit to the driver, maturity, team skill, operational burden, community, lock-in) vs the hype cycle and "résumé-driven development"; running a spike/bake-off; the cost of every new technology added (the "boring technology" argument).
**Design exercise:** The team wants to adopt a trendy new database for a non-core feature. Run the build-vs-buy and technology-selection analysis and write the recommendation.

### [Lesson 34: The architect as communicator & influencer](lessons/lesson-34-architect-as-communicator.html)
**Goal:** Get an architecture *adopted* — the technical decision is worthless if the org doesn't build it. *(Pairs with the whole [Leadership]({{ '/leadership/learning-plan.html' | relative_url }}) communication and influence phases.)*
**Topics:** Architecture is a social act: influence without authority (you rarely own the teams that implement); selling the *why* and the trade-offs, not decreeing the *what*; the architect who codes vs the ivory tower; running an architecture review or guild; meeting each audience where they are (exec, PM, engineer); disagree-and-commit; being wrong gracefully; the "architect as gardener, not dictator" model.
**Design exercise:** Two senior teams are skeptical of your proposed shared platform, for two different reasons. Plan how you'd build buy-in (not how you'd overrule them).

### [Lesson 35: Architecture anti-patterns & pitfalls](lessons/lesson-35-antipatterns.html)
**Goal:** Recognize the classic failure modes by name so you can catch them early — in your own designs first.
**Topics:** The big ball of mud; the distributed monolith (microservices with monolith coupling — worst of both); accidental complexity and over-engineering (gold-plating, speculative generality, YAGNI); the resume-driven / hype-driven design; premature optimization *and* premature scaling; the "architect's dream, developer's nightmare"; analysis paralysis and the ivory tower; leaky abstractions; vendor lock-in by accident; the second-system effect.
**Design exercise:** Given a design riddled with three anti-patterns, name each one, explain the harm, and propose the corrective.

### [Lesson 36: Capstone — design a system end to end](lessons/lesson-36-capstone.html)
**Goal:** Put the whole track together: take a real-world brief from requirements to a defended architecture.
**Topics:** The full arc — drivers & ASRs → quality attributes → style choice → bounded contexts & boundaries → data & consistency → communication & resilience → cross-cutting (security, observability, scale) → deployment → documentation (C4 + key ADRs) → evaluation & evolution plan → the trade-offs you consciously made. A synthesis, not new material.
**Design exercise (capstone):** Design the architecture for a given real-world product brief end to end — produce the C4 Context/Container sketch, three key ADRs, the top quality attributes with scenarios, and an honest list of the trade-offs and risks. A cumulative model answer walks the full reasoning.

---

*All 36 lessons written = track complete. Update CLAUDE.md's index as each lands.*
