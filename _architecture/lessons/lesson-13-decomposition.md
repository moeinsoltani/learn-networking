---
title: "Lesson 13 — Service Granularity & Decomposition"
nav_order: 5
parent: "Phase 3: Architectural Styles"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 13: Service Granularity & Decomposition

{: .note }
> **Words to know**
> - **granularity** — how big or small your services are; coarse-grained = few big ones, fine-grained = many small ones.
> - **disintegrator** — a force that argues for *splitting* something into separate services.
> - **integrator** — a force that argues for *keeping* things together in one service.
> - **nano-service** — a service so small it does almost nothing useful alone; a granularity anti-pattern.
> - **chatty / chattiness** — many back-and-forth network calls to do one logical operation; a sign services are split too finely.
> - **decompose** — to break a system (usually a monolith) into components/services.

## Concept

"How big should a service be?" is the question that sinks most microservice efforts. The
wrong answers are equally common: too coarse (you didn't really gain independence) and too
fine ("nano-services" so small that a single business operation requires ten chatty network
calls, reintroducing all the coupling as latency and failure). There's no magic size — the
right granularity is a **balance of opposing forces**.

```
   FORCES THAT PUSH APART                 FORCES THAT PULL TOGETHER
   (disintegrators — reasons to SPLIT)    (integrators — reasons to MERGE)
   ────────────────────────────────      ────────────────────────────────
   • different scaling needs              • a shared database / data that
   • fault isolation needed                 must stay transactionally consistent
   • different rate of change             • a workflow that's one unit of work
   • different security/compliance zone   • chatty back-and-forth (would become
   • separate team ownership                network calls)
   • different technology needs           • strong data dependency between them

            The right granularity is the BALANCE point, not the extreme.
            "Smallest possible" is a mistake as surely as "one giant service."
```

The mental model (from *Software Architecture: The Hard Parts*): for any candidate
boundary, list the **disintegrator** forces (reasons to pull it apart) and the
**integrator** forces (reasons to keep it together), and let them argue. Split when the
disintegrators clearly win; keep together when the integrators do. "Microservices" does
*not* mean "as small as possible" — it means "the size where the forces balance," which is
often bigger than beginners think.

## Going Deeper

**The disintegrators — legitimate reasons to split:**
- **Differential scalability** — two capabilities have very different load profiles and
  should scale independently (a read-heavy catalog vs a write-heavy order stream).
- **Fault isolation** — one capability's failure must not take down another (payments
  should survive the recommendation engine falling over).
- **Different rate/reason of change** — a part that changes daily (pricing/promotions)
  vs one that's stable for years shouldn't be chained together in one deploy.
- **Security / compliance isolation** — payment-card or health data belongs in a tightly-
  controlled service with a smaller attack surface and its own audit boundary.
- **Separate team ownership** — a distinct team that needs to deploy autonomously
  (Conway's Law again — the org reason, often the strongest).

**The integrators — legitimate reasons to keep together:**
- **Transactional data consistency** — if two operations must be atomic (both happen or
  neither), splitting them means giving up ACID and adopting a saga (Lesson 17), which is a
  lot of complexity — a strong reason to keep them in one service with one database.
- **Strong data dependency** — if two capabilities constantly read/write the same data,
  splitting them just turns local data access into network calls (chattiness).
- **Chattiness** — if doing one logical operation would require many round-trips between the
  two candidates, they're too intertwined to split cheaply — the network cost and latency
  would dominate.
- **A single workflow / unit of work** — steps that are really one cohesive process are
  often clearer and more reliable as one service.

The decision is disintegrators vs integrators at *each* candidate seam — not a global "make
everything a service."

{: .note }
> **Start from bounded contexts, and decompose the monolith, don't green-field it**
> Two practical anchors. (1) The best <em>starting</em> boundaries are your <strong>bounded
> contexts</strong> (Lesson 7) — they're already high-cohesion, loosely-coupled units with
> their own consistent model, which is exactly what a good service is. When people ask "how
> do I find service boundaries?", "along the bounded contexts" is the first answer, then
> refine with the disintegrator/integrator forces. (2) The safest <em>way</em> to get there
> is <strong>component-based decomposition</strong> of an existing modular monolith: first
> make clean components/modules <em>inside</em> the monolith (Lesson 9), let their boundaries
> and dependencies prove themselves, and only then extract the ones that a disintegrator
> force justifies — rather than guessing service boundaries up front on a green field, where
> you understand the domain least and a wrong boundary is brutally expensive.

**Two failure modes, symmetric.** Get granularity wrong in one direction and you get a
**distributed monolith** (Lesson 11) — services too coarse or split along the wrong seams, so
they're coupled and can't deploy independently. Get it wrong the other way and you get a
**nano-service swamp** — services so fine-grained that every business operation is a storm of
chatty network calls, every change spans five services, and the coordination overhead exceeds
any benefit. Both are worse than a well-modularized monolith. The right granularity threads
between them, and it's found by force-balancing at each seam, not by picking a target service
count.

**Granularity can and should evolve.** You don't have to get it perfect up front — and you
can't. Boundaries that are cheap to move (within a modular monolith) let you defer the hard
service-granularity decisions to the last responsible moment (Lesson 4), and evolutionary
architecture (Lesson 31) means you can merge two services that turned out too fine, or split
one that grew too coarse, as the forces change. The worst move is treating an early
granularity guess as permanent — especially a fine-grained one made before the domain was
understood.

---

## Lab — Design Exercise

**The situation:** A team decomposing an e-learning platform has proposed these four
candidate services. For each, decide whether the granularity is right, too coarse, or too
fine, using the disintegrator/integrator forces:

1. **`UserService`** — owns everything about users: authentication, profile, preferences,
   *and* the user's course enrollments, *and* their payment history, *and* their forum posts.
2. **`EnrollmentService`** and **`ProgressService`** — two separate services. Enrollment
   records which courses a user is enrolled in; Progress records how far they are in each
   course. Every "resume my course" operation calls Enrollment to check access, then Progress
   to get the position, then Enrollment again to log activity — many round-trips, sharing
   most of the same data.
3. **`PaymentService`** — handles all payment processing, card data, and PCI-compliance
   concerns, separate from everything else.
4. **`FirstNameService`** and **`LastNameService`** — separate services, each storing and
   serving one field of a user's name.

**For each, name the dominant forces and give your verdict.** Then say what the *right*
decomposition of the user-related capabilities looks like.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is applying the two sets of forces at each seam. A strong answer:
<br><br>
<strong>1. <code>UserService</code> owning auth + profile + enrollments + payment history +
forum posts → too coarse (a mini-monolith).</strong> Dominant forces are
<em>disintegrators</em> being ignored: these are different bounded contexts with different
change rates, scaling, security, and likely different teams. Payment history has PCI/security
concerns that shouldn't share a service with forum posts; enrollments belong to the learning
domain, not the identity domain; forum posts are a different capability entirely. Bundling them
means one deploy for five unrelated concerns, one shared database coupling them, and a broad
attack/audit surface. <em>Verdict:</em> split by bounded context — a real
<code>Identity/User</code> service (auth, profile, preferences), with enrollments, payments,
and forum as their own contexts.
<br><br>
<strong>2. <code>EnrollmentService</code> + <code>ProgressService</code> as separate services
→ too fine; should be merged (or at least co-located).</strong> Dominant forces are
<em>integrators</em>: they share most of the same data (both keyed on user+course), they have a
<em>strong data dependency</em>, and the "resume my course" flow is <strong>chatty</strong>
(Enrollment → Progress → Enrollment for one operation), turning what would be local calls into
multiple network round-trips with added latency and new failure points. There's no stated
disintegrator (no different scaling, no separate team, no security boundary) to justify the
split. This is a classic over-decomposition that created a distributed-monolith seam.
<em>Verdict:</em> merge into one <code>LearningProgress</code>/<code>Enrollment</code> service
(enrollment + progress together), so the "resume" operation is local and transactional.
<br><br>
<strong>3. <code>PaymentService</code> separate → right, arguably the best-justified split.</strong>
Multiple strong <em>disintegrators</em>: <em>security/compliance isolation</em> (PCI — you want
card data and payment logic in a tightly-controlled service with a minimal attack surface and
its own audit boundary), likely <em>different rate of change</em> and possibly a specialized
team, and <em>fault isolation</em>. The integrator forces are weak (payment doesn't need to
share a transaction with forum posts). <em>Verdict:</em> keep it separate — this is exactly the
kind of boundary where the disintegrators clearly win.
<br><br>
<strong>4. <code>FirstNameService</code> + <code>LastNameService</code> → absurdly too fine
(the nano-service anti-pattern).</strong> There is <em>no</em> disintegrator — first and last
name have identical scaling, change rate, security, and ownership, and are always used
together — and every integrator applies (same data, same lifecycle, maximal chattiness). Two
services to serve two fields of one concept means every "get the user's name" is two network
calls for zero benefit; it's pure overhead and coordination cost. <em>Verdict:</em> these
aren't services at all — they're two fields of the User profile, inside the Identity service.
<br><br>
<strong>The right decomposition of the user-related capabilities:</strong> follow the bounded
contexts, letting the forces refine them:
<ul>
<li><code>Identity/User</code> — auth, profile (including <em>both</em> name fields),
preferences. One cohesive identity context.</li>
<li><code>Enrollment & Progress</code> — one service (they're welded by shared data and a
chatty, transactional flow).</li>
<li><code>Payments</code> — separate, for security/compliance isolation.</li>
<li><code>Forum</code> — its own context if it's a real capability (different domain, possibly
different team/scaling); otherwise a module.</li>
</ul>
<strong>The meta-lesson:</strong> right granularity is neither "one big <code>UserService</code>"
(too coarse — a mini-monolith bundling unrelated contexts) nor "a service per field" (too fine
— nano-services and chattiness). It's found by asking, at each seam, whether the disintegrator
forces (different scaling/change/security/team) beat the integrator forces (shared data,
transactional consistency, chattiness) — and letting genuine bounded contexts be the starting
skeleton. The best-justified split (Payments) is driven by a strong disintegrator (security);
the worst splits (Enrollment/Progress, First/Last name) ignored overwhelming integrators. That's
the whole discipline.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Disintegrators & integrators (granularity forces) | *Software Architecture: The Hard Parts*, Ford, Richards et al. (Ch. 7) |
| Service boundaries from bounded contexts | Sam Newman, *Building Microservices* (2e), Ch. 2 |
| Decomposing a monolith (component-based) | Newman, *Monolith to Microservices* |
| Nano-services anti-pattern | <https://en.wikipedia.org/wiki/Microservices> (see "granularity" critiques) |

---

## Checkpoint

**Q1.** Explain the disintegrator/integrator framework for deciding service granularity, and
use it to explain why "make services as small as possible" is wrong. Give one strong
disintegrator and one strong integrator.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The framework: for any candidate boundary (a place you're considering splitting into separate
services, or keeping together), list the <strong>disintegrator</strong> forces — reasons to
pull apart — and the <strong>integrator</strong> forces — reasons to keep together — and let
them argue. Split when the disintegrators clearly win; keep together when the integrators do.
Granularity isn't a target number; it's the balance point found seam by seam.
<br><br>
A strong <strong>disintegrator</strong>: <em>differential scalability</em> (or fault
isolation, or security isolation, or separate-team ownership) — e.g., a read-heavy catalog and
a write-heavy order stream have such different load profiles that they should scale
independently, which one combined service can't do. A strong <strong>integrator</strong>:
<em>transactional data consistency / strong shared-data dependency</em> — e.g., two operations
that must be atomic and constantly touch the same data; splitting them means giving up ACID for
a saga (Lesson 17) <em>and</em> turning local reads into chatty network calls, so the forces to
keep them together are overwhelming.
<br><br>
Why "as small as possible" is wrong: it treats one force (fine-grained independence) as the
only goal and ignores the integrators entirely. Pushed to the extreme you get
<strong>nano-services</strong> — services so small that a single business operation requires a
storm of network calls between them (chattiness), every change spans many services, shared data
gets fragmented across service boundaries (breaking transactions and forcing sagas everywhere),
and coordination overhead swamps any benefit. That's the nano-service swamp: all the distributed
tax, none of the payoff — often strictly worse than a modular monolith. "Small" is not the
objective; <em>the right size for the forces</em> is, and that's frequently coarser than
beginners assume. The discipline is to split only where a disintegrator genuinely wins and to
resist splitting where integrators (shared data, transactions, chattiness) dominate — which is
exactly how you avoid both the too-coarse distributed monolith and the too-fine nano-service
swamp.
</details>

**Q2.** Why is "start from bounded contexts, then decompose a modular monolith" safer than
designing fine-grained services up front on a green-field project?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
Two reinforcing reasons — one about <em>where</em> the boundaries go, one about <em>when</em>
you commit to them.
<br><br>
<strong>Bounded contexts give you the right seams.</strong> A bounded context (Lesson 7) is
already a high-cohesion, loosely-coupled unit with its own consistent model and language —
which is precisely the property a good service boundary needs. Starting from contexts means
your service boundaries follow the natural fault lines of the domain (where coupling is
genuinely low and cohesion high), rather than arbitrary or technical lines that cut across how
the system actually changes. It's the difference between splitting rock along its grain vs
across it.
<br><br>
<strong>Decomposing a modular monolith defers the irreversible commitment to when you know
most.</strong> On a green field you understand the domain <em>least</em> — the boundaries
you'd draw are guesses, and a wrong <em>service</em> boundary is one of the most expensive
mistakes to fix (splitting/merging distributed services with their own data is brutal, whereas
moving a boundary <em>within</em> a monolith is a refactor). So designing fine-grained services
up front means locking in high-consequence, hard-to-reverse decisions at the moment of maximum
ignorance — and fine granularity multiplies the number of these guesses. The safer path: build
a <em>modular monolith</em> first, where the boundaries are cheap to move, let them
<em>prove themselves</em> as the domain clarifies and the real disintegrator forces emerge
(this part actually needs to scale separately; this part is now owned by a second team), and
only <em>then</em> extract the specific components a real force justifies — a small, informed,
low-risk step at a time. This respects the last-responsible-moment principle (Lesson 4) for the
irreversible decision (physical service boundaries) while keeping the reversible one (logical
module boundaries) open and adjustable. In short: bounded contexts point you at the right
lines, and monolith-first decomposition lets you commit to the expensive version of those lines
only once you've seen that they hold — instead of betting the architecture on green-field
guesses that fine granularity makes both more numerous and more costly to get wrong.
</details>

---

## Homework

Take your system (or a system you know) and pick two service (or module) boundaries: one you
suspect is *too coarse* (a service bundling unrelated concerns that should be split) and one you
suspect is *too fine* (services that are chatty or share data and should probably be merged).
For each, list the disintegrator and integrator forces and reach a verdict. If your system is a
monolith, instead pick the two boundaries you'd extract *first* if a driver forced it, and the
two you'd *never* extract (because integrators dominate) — and say which forces decide each.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
The exercise trains force-balancing on real seams. A strong response:
<br><br>
<strong>The too-coarse candidate</strong> is usually a "kitchen-sink" service (a
<code>UserService</code>/<code>CoreService</code>/<code>PlatformService</code> that has accreted
several unrelated capabilities) where the disintegrators are being ignored — different change
rates, different scaling, or a security-sensitive concern (payments, PII) sharing a service with
unrelated ones. A good answer names the specific disintegrator that justifies a split (often
security isolation or separate-team ownership) and proposes splitting along the bounded contexts
inside it.
<br><br>
<strong>The too-fine candidate</strong> is the more instructive find: two services that are
<em>chatty</em> (one logical operation = several cross-service calls) and/or share most of their
data, with no real disintegrator justifying the separation — a distributed-monolith seam or an
outright nano-service. The honest verdict is usually "merge them," and recognizing that the
split is buying network latency and coupling for no benefit is exactly the lesson. (The tell is
often a single user action that fans out into a burst of internal calls, or two services that are
always changed and deployed together.)
<br><br>
<strong>For a monolith:</strong> naming the "extract first" candidates (the modules with a real
disintegrator waiting — one that will need independent scaling, or a second team, or security
isolation like payments) versus the "never extract" ones (modules welded by transactional
consistency, shared data, or chattiness — where integrators dominate) is precisely the
granularity judgment applied prospectively. The best answers show that the decision is
<em>per-seam</em> and force-driven, not global: some boundaries clearly want to be services
(strong disintegrators), some clearly never should be (strong integrators), and the skill is
telling which is which by weighing the two force sets — the same discipline whether you're
splitting a monolith or fixing an over-decomposed distributed system.
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 14 — The Fallacies of Distributed Computing →](lesson-14-fallacies){: .btn .btn-primary }
