---
title: "Lesson 27 — Cloud & Deployment Architecture"
nav_order: 5
parent: "Phase 6: Cross-Cutting Quality Attributes"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 27: Cloud & Deployment Architecture

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

For most of software history, "deployment" was a thing that happened *after* architecture — an
ops problem, someone else's job, downstream of the design. That is no longer true. **Deployability
is now a first-class quality attribute** (Lesson 3): how fast, how safely, and how often you can get
a change into production is a property you *design for*, and it shapes the structure as much as
performance or security does. A design that can only be released by hand, at midnight, with downtime,
is a *worse architecture* than one that ships continuously with zero downtime — even if the
boxes-and-arrows look identical.

**Deployment is part of the design, not an afterthought.**

In the old model, you designed, built, and then threw the result over the wall
to operations — where deploying was manual, risky, rare, and involved downtime.
The design had nothing to say about any of that.

Now the design *includes* how the system is built, shipped, and run, and a
handful of properties do most of the work:

| Property | Why it matters |
|---|---|
| **Stateless services** | You can scale and replace instances freely |
| **Configuration in the environment** | The same build runs in any stage |
| **Immutable infrastructure / IaC** | Rebuild rather than patch; environments stop drifting |
| **Blue-green and canary releases** | Release without downtime, and roll back cheaply |
| **Managed services** | Rent the undifferentiated heavy lifting |
| **Cost** | A design *output*, not something free that arrives with the bill |

The framing worth adopting: **deployability is a quality attribute** (Lesson
03), and like any other, you design for it deliberately or you discover you did
not have it at the worst possible moment.

The organizing idea is that the way a system is **built, deployed, and operated** is now inseparable
from its architecture. The **twelve-factor app**[°](#w-twelve-factor-app) gives the baseline; statelessness and externalized state
make horizontal scale and safe releases possible; containers and **orchestration**[°](#w-orchestration) change what a
"deployable unit" is; and the cloud turns decisions that used to be capital purchases (a data center,
a database cluster) into architectural trade-offs you make continuously — including **cost**, which
in the cloud is a direct output of your design.

## Going Deeper

**The twelve-factor app — the deployability baseline.** The [twelve-factor](https://12factor.net/)
methodology is a checklist of properties that make a service cloud-friendly and easy to operate. The
load-bearing ones for an architect: **config in the environment** (never hard-code URLs, credentials,
or per-stage values — the *same build artifact* runs in dev, staging, and prod, differing only by
environment config), **stateless, share-nothing processes** (keep no session or file state in the
process; push state to a datastore or cache), **disposability** (processes start fast and shut down
gracefully, so they can be killed and replaced at will), **dev/prod parity** (keep the environments
as similar as possible), and **logs as event streams** (write to stdout; let the platform route them
— Lesson 25). These aren't ceremony: each one directly enables scaling, zero-downtime releases, and
observability. Statelessness in particular is the linchpin — it's what lets *any* instance serve
*any* request, which is the precondition for horizontal scale (Lesson 23) and for replacing instances
during a deploy.

**Containers and orchestration — what changed architecturally.** A container packages the app *and
its dependencies* into one immutable image, so "works on my machine" becomes "works everywhere the
image runs." That standardized unit is what makes **orchestration** (Kubernetes and friends) possible:
the orchestrator schedules containers onto machines, restarts failed ones, scales them up and down,
and does rolling releases — turning "run my service reliably" into a declarative spec. The
architectural shift is that your *deployable unit* is now a self-contained image, and much of the
resilience you used to build by hand (health checks, restarts, autoscaling — Lesson 18) is provided
by the platform. The cost is real operational complexity; the platform is itself a system to run and
understand. *(The [Virtualization]({{ '/virtualization/learning-plan.html' | relative_url }}) and
[Networking]({{ '/networking/learning-plan.html' | relative_url }}) tracks are the ground truth
beneath containers and cluster networking.)*

**Immutable infrastructure & infrastructure as code.** The old model produced **snowflake servers** —
long-lived machines patched and tweaked by hand until each one was unique, undocumented, and terrifying
to touch. **Immutable infrastructure**[°](#w-immutable-infrastructure) replaces that: you never modify a running server; to change
anything you build a *new* image and replace the old instance. Combined with **infrastructure as code**
(IaC — the servers, networks, and policies defined in version-controlled files and applied by a tool
like Terraform), your whole environment becomes reproducible, reviewable, and diffable. The
architectural payoff: environments stop drifting, rollback is "redeploy the previous image," and
your infrastructure is subject to the same review and testing discipline as your code.

{: .warning }
> **Deployment strategies — releasing without downtime**
> Zero-downtime release is an architectural capability, not a deployment-script detail. The main
> strategies and what each buys:
> - <strong>Rolling</strong> — replace instances a few at a time; old and new versions run
>   simultaneously during the roll. Simple and the orchestrator default, but it means <em>both
>   versions serve traffic at once</em>, so your API and database changes must be
>   <strong>backward compatible</strong> (Lesson 26's expand–contract — this is why compatibility is
>   an architectural concern, not just an API one).
> - <strong>Blue-green</strong> — stand up the full new version ("green") alongside the current one
>   ("blue"), switch traffic over all at once, keep blue ready for instant rollback. Fast, clean
>   rollback; costs double the capacity during the switch.
> - <strong>Canary</strong> — release to a small slice of traffic (1%, then 5%, then 25%…), watch the
>   metrics (Lesson 25), and roll forward or back based on real signal. The safest for risky changes;
>   needs good observability and traffic-splitting to work.
> All three depend on <strong>**stateless services**[°](#w-stateless-service) + externalized state</strong> and
> <strong>backward-compatible changes</strong>. If instances hold state, or a new version's DB schema
> breaks the old version still running, none of these are safe. Deployability constrains the design.

**Managed services and the build-vs-rent trade at the infra layer.** The cloud lets you *rent* the
undifferentiated heavy lifting — a managed database, queue, cache, or object store — instead of
running your own. The trade mirrors build-vs-buy (Lesson 33) but at the infrastructure layer: a
managed service removes enormous operational burden (patching, backups, replication, failover) at the
cost of money, some control/flexibility, and **lock-in** risk. The architect's default: rent the
generic infrastructure that isn't your competitive advantage (you gain little by operating your own
Postgres), and reserve "build/self-operate" for the rare cases where a managed option genuinely
doesn't fit or the cost/lock-in is unacceptable. Running your own version of something a provider
operates well is often just operational cost with no architectural return.

**Cost and multi-region are design outputs.** In the cloud, the bill is a direct consequence of
architectural choices — chatty inter-service calls, over-provisioned instances, egress traffic, and
data-store choices all show up as line items, so **cost is a quality attribute** you trade against
the others (a beautifully redundant multi-region design that no one will pay for is not a good design).
And **multi-region** is the clearest availability/complexity trade there is: spreading across regions
buys you survival of a whole-region outage, at the price of data replication across high latency, the
consistency questions of Lesson 15, and a large jump in operational complexity. Reach for it when the
availability requirement (an ASR, Lesson 4) genuinely demands it — not by default.

---

## Lab — Design Exercise

**The situation:** You've inherited a **stateful monolith** deployed by hand: a single VM that a
senior engineer SSHes into, pulls the latest code onto, and restarts — taking the site down for a few
minutes each release, which is why releases happen roughly once a month at night. Local files are
written to the VM's disk (user uploads, session data, a search index), configuration lives in a file
edited directly on the box, and there's exactly one of it. The business now wants **frequent,
zero-downtime releases**.

**Design its path to a twelve-factor, container-deployed service with zero-downtime releases.** Walk
the transformation, and name the **single hardest step** and why.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is sequencing the transformation and recognizing that the <em>state on the local disk</em>
— not the containerization — is the hard part. A strong answer:
<br><br>
<strong>1. Externalize configuration (twelve-factor: config in the environment).</strong> Move the
config off the edited-on-the-box file into environment variables / a config service, so the exact same
build artifact can run anywhere and differs only by environment. This is a prerequisite for having
more than one identical instance and for immutable images.
<br><br>
<strong>2. Externalize state — the crux.</strong> Today the process holds state on its local disk:
user uploads, session data, a search index. As long as that's true, you <em>cannot</em> run a second
instance or replace the instance without losing or splitting state — which kills both horizontal scale
and zero-downtime release. So: move <em>uploads</em> to object storage (e.g. S3-style), move
<em>sessions</em> to a shared store (a cache/DB) or make them stateless tokens (Security track), and
move the <em>search index</em> to a managed/standalone search service. Only once the service is
<strong>stateless</strong> can any instance serve any request and instances be replaced freely.
<br><br>
<strong>3. Containerize (immutable, disposable unit).</strong> Package the app + dependencies into an
image so it runs identically everywhere and starts/stops cleanly. Now the deployable unit is an
immutable artifact, not "whatever is currently on the VM."
<br><br>
<strong>4. Automate the infrastructure and the pipeline (IaC + CI/CD).</strong> Define the
environment as code so it's reproducible and reviewable; build and deploy through a pipeline instead of
SSH-and-pull. Run <em>more than one instance</em> behind a load balancer (now possible because the
service is stateless).
<br><br>
<strong>5. Adopt a zero-downtime deployment strategy.</strong> With multiple stateless instances, use
rolling or blue-green releases so new instances come up and healthy, take traffic, and old ones drain
— no downtime. Because old and new versions overlap during the release, make schema/API changes
<strong>backward compatible</strong> (expand–contract, Lesson 26) — this is the coupling between
deployment strategy and design.
<br><br>
<strong>The single hardest step: externalizing state (step 2).</strong> Containerizing and automating
are mostly mechanical; the genuinely hard, risky work is prying state out of the local process —
migrating existing uploads to object storage, reworking session handling, and standing up/migrating the
search index — because it touches data (which can be lost), changes application code, and is the
<em>precondition</em> that everything else (multiple instances, disposability, zero-downtime release)
depends on. A stateful process is fundamentally un-scalable and un-replaceable; until the state is
externalized, no amount of containerization buys zero-downtime release. The mature answer names this
explicitly: the container is easy, the <em>statelessness</em> is the architecture change, and it should
be done carefully (parallel run, migrate data, verify) before the deployment modernization can pay off.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| The twelve-factor app | <https://12factor.net/> |
| Immutable infrastructure | <https://martinfowler.com/bliki/ImmutableServer.html> |
| Blue-green deployment | <https://martinfowler.com/bliki/BlueGreenDeployment.html> |
| Canary release | <https://martinfowler.com/bliki/CanaryRelease.html> |
| Deployment strategies & cloud patterns | <https://learn.microsoft.com/azure/architecture/guide/> ; <https://aws.amazon.com/architecture/well-architected/> |
| Containers & orchestration (background) | [Virtualization track]({{ '/virtualization/learning-plan.html' | relative_url }}) |

---

## Checkpoint

**Q1.** Why is deployability now a first-class quality attribute, and why is statelessness the linchpin
that makes horizontal scaling and zero-downtime releases possible?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Deployability as a quality attribute:</strong> how fast, safely, and frequently you can get a
change into production is a measurable property of the system (Lesson 3), and it has enormous business
value — it determines how quickly you can fix bugs, respond to the market, and iterate. A design that
can only be released manually, rarely, with downtime is genuinely <em>worse</em> than one that ships
continuously with zero downtime, even if the component diagram is identical. So how the system is
built, deployed, and run is now something you <em>design for</em>, not something ops handles after the
fact — twelve-factor practices, statelessness, immutable infrastructure, and deployment strategy are
architectural choices.
<br><br>
<strong>Statelessness as the linchpin:</strong> a <em>stateless</em> service keeps no per-request or
session state in its own memory or local disk — any state lives in an external store. That property is
what makes two things possible. <em>Horizontal scaling</em> (Lesson 23): if any instance can serve any
request, you can add instances behind a load balancer and they share the load; if instances held state,
a given user's requests would have to go to a specific instance (sticky), and you couldn't freely add
or remove capacity. <em>Zero-downtime release</em>: rolling/blue-green/canary all work by standing up
new instances and draining old ones; that's only safe if instances are interchangeable and disposable —
i.e., stateless — because killing an instance must not lose state or strand a user. A stateful process,
by contrast, is a single point that can't be replaced or multiplied without losing or splitting its
state. That's why externalizing state (to a datastore, cache, or object storage) is the precondition
for both scale and safe deployment — it's the architectural change that unlocks the rest.
</details>

**Q2.** Compare rolling, blue-green, and canary deployments — what each buys and costs — and explain
why backward-compatible changes are required for all of them.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Rolling:</strong> replace instances in batches, so old and new versions run at the same time
during the roll. Cheap (no extra capacity beyond a small buffer) and the orchestrator default, but
rollback is slower (roll back the same way) and both versions serve traffic simultaneously.
<br><br>
<strong>Blue-green:</strong> stand up the entire new version ("green") beside the running one ("blue"),
switch all traffic at once, keep blue warm for instant rollback. Buys <em>fast, clean rollback</em> and
a clear cutover; costs roughly <em>double the capacity</em> during the transition.
<br><br>
<strong>Canary:</strong> release to a small percentage of traffic, watch real metrics (Lesson 25), and
progressively increase or roll back. Buys the <em>safest</em> release for risky changes (you catch
problems while blast radius is 1% of users); costs the need for good observability and traffic-splitting
infrastructure, and takes longer.
<br><br>
<strong>Why all three need backward-compatible changes:</strong> every one of these strategies has a
window where the <em>old and new versions run at the same time</em> against the <em>same database and
the same clients</em>. Rolling: both versions serve traffic throughout the roll. Blue-green: both exist
around the switch (and share the DB). Canary: explicitly, some traffic hits old, some hits new, for as
long as the canary runs. So the new version must not make a change that breaks the still-running old
version — most critically, a <strong>database schema change</strong> the old code can't handle, or an
<strong>API change</strong> that breaks other services or clients still calling the old shape. This is
exactly the expand–contract / parallel-change discipline from Lesson 26, applied to deployment:
add-then-migrate-then-remove, never break-in-place. It's why deployment strategy and API/data
compatibility are the same architectural concern — you can't have zero-downtime releases without
backward-compatible evolution, because zero-downtime <em>means</em> two versions coexist.
</details>

---

## Homework

Take a service you operate and assess its deployment architecture against twelve-factor and
zero-downtime. How is config handled — in the environment, or baked into builds / edited on boxes? Is
the service genuinely stateless, or does it hold session/file/cache state that would prevent running a
second instance? How are releases done — automated pipeline or manual steps; zero-downtime or a
maintenance window; can you roll back in seconds? Is the infrastructure defined as code and immutable,
or are there snowflake servers no one dares rebuild? Which infrastructure pieces are managed (rented)
vs self-operated, and is any self-operated piece just operational cost with no architectural return?
Identify the single biggest deployability risk (often: hidden local state, or manual/risky releases)
and the one change you'd make first.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
A strong response applies deployability-as-quality-attribute thinking to a real service.
<br><br>
<strong>Finds where state hides.</strong> The most common and most damaging finding is <em>hidden
local state</em> — files written to the instance's disk, in-memory sessions or caches, a background
job that assumes it's the only one running — which silently makes the service un-scalable and
un-safe-to-replace even if everything else looks modern. Naming exactly what state lives locally, and
that it's the thing blocking multiple instances and zero-downtime release, is the key insight (it
mirrors the lab).
<br><br>
<strong>Assesses the release process honestly.</strong> Is there an automated pipeline, or manual SSH
steps? Is there downtime? Can you roll back in seconds, or is rollback a scramble? Slow, manual, risky
releases are a real architectural weakness (they make the team ship rarely and fear changes), not just
an ops annoyance — recognizing that connects deployability to velocity and reliability.
<br><br>
<strong>Checks config, immutability, and managed-vs-self.</strong> Config in the environment (same
artifact everywhere) vs baked/edited-on-box; immutable images + IaC vs snowflake servers you're afraid
to rebuild; and whether any self-operated infrastructure (a hand-run database, queue, or search
cluster) is just operational burden that a managed service would remove with no architectural downside.
<br><br>
<strong>Names the one first move.</strong> The architect's discipline is to pick the highest-leverage
single change — usually either (a) externalizing the hidden state so the service can finally be
stateless (unlocking scale <em>and</em> zero-downtime), or (b) automating the release into a
zero-downtime pipeline, or (c) getting config out of the box and infra into code. The takeaway a good
answer reaches: deployability is designed, not inherited; statelessness + externalized state is the
foundation everything else (scaling, safe releases) stands on; and the biggest wins usually come from
removing hidden local state and manual release steps rather than from adopting a fancier platform.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-twelve-factor-app"></a>**twelve-factor app** — a set of practices (config in the environment, stateless processes, disposability…) that make a service easy to deploy, scale, and run.
- <a id="w-stateless-service"></a>**stateless service** — a service that keeps no per-request state in its own memory/disk; any instance can serve any request, so you can add or kill instances freely.
- <a id="w-immutable-infrastructure"></a>**immutable infrastructure** — you never patch a running server; you replace it with a freshly built one. No "snowflake" servers drifting apart.
- <a id="w-iac-infrastructure-as-code"></a>**IaC (infrastructure as code)** — the servers, networks, and config defined in version-controlled files, applied by a tool — not clicked together by hand.
- <a id="w-blue-green-canary-rolling"></a>**blue-green / canary / rolling** — deployment strategies for releasing a new version without downtime and with controlled risk.
- <a id="w-orchestration"></a>**orchestration** — a system (e.g. Kubernetes) that schedules containers onto machines, restarts failed ones, and scales them.
- <a id="w-build-vs-rent-managed-service"></a>**build vs rent (managed service)** — using a provider's managed database/queue instead of running your own; you trade control and cost for operational burden.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 28 — Documenting Architecture: the C4 Model & Views →](lesson-28-documenting-c4){: .btn .btn-primary }
