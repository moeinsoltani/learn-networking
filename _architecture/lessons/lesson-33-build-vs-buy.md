---
title: "Lesson 33 — Build vs Buy & Technology Selection"
nav_order: 1
parent: "Phase 8: The Architect in Practice"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 33: Build vs Buy & Technology Selection

*Words marked ° are explained in plain English in [Words to Know](#words-to-know) at the end of the lesson.*

## Concept

Two of the most consequential decisions an architect makes aren't *how* to build something — they're
**whether to build it at all**, and **which technology** to use. Both are routinely made badly, for the
same reason: by reflex (résumé-driven, hype-driven, "we always use X") instead of by judgment. The
governing lens for both is **opportunity cost**[°](#w-opportunity-cost): every hour and every technology you spend on this is an
hour and a slot you *don't* spend on something else, so the question is never "can we build it?" (you
usually can) but "is this the best use of our scarce building capacity?"

Two decisions, one lens — **opportunity cost**.

**Build versus buy** turns on a single question: *is this your core domain?*
If yes, **build** it — that is your edge, and outsourcing it outsources your
advantage. If no, **buy, rent, or adopt open source**, because it is
undifferentiated work that will not win you a customer. Judge on **total cost
of ownership** rather than sticker price, and include the exit cost and the
lock-in in that total.

**Technology selection** turns on fit to the drivers from Lesson 04, plus a
handful of practical filters: maturity, your team's existing skill, the
operational burden it adds, the health of its community, lock-in, and again
TCO.

What both decisions are defending against is choosing for hype or for the CV. A
useful discipline: treat **every new technology as spending one token** from a
small budget. A system with three unfamiliar technologies in it is not three
times as modern; it is one team learning three things while trying to ship.

Two heuristics do most of the work. For **build vs buy**[°](#w-build-vs-buy): build what is your **core domain** — your
actual competitive advantage, the thing customers pay *you* for (Lesson 7's DDD subdomains) — and buy or
rent everything **generic** (auth, email, payments, search infrastructure), because building
undifferentiated capability is spending your best people on something a vendor already does better. For
**technology selection**: choose for **fit to the driver** (Lesson 4) and honest operational reality —
maturity, your team's skill, the operational burden, community, and **lock-in**[°](#w-lock-in) — weighed against the
**boring-technology** discipline that every novel technology is a cost, not a free upgrade. Both
decisions are judged on **total cost of ownership**, not the sticker price — because the sticker price is
always the smallest part.

## Going Deeper

**Build vs buy vs open-source vs don't — through opportunity cost.** The options are more than a binary:
**build** it, **buy** a commercial product/SaaS, **rent** a managed service (Lesson 27), adopt
**open-source**, or **don't do it at all** (the most-overlooked option — is this even worth doing?). The
lens that cuts through it is **opportunity cost**: your team's building capacity is finite and precious,
so spending it on something that isn't your differentiator is a loss even if the build "succeeds." The
DDD frame (Lesson 7) makes it concrete: is this your **core** subdomain (the reason your business wins —
*build* it, invest your best people, because a bought version can't be your advantage), or a
**supporting/generic** subdomain (email delivery, authentication, a CMS, a payment gateway — *buy/rent*
it, because it's table stakes, not edge)? Building your own auth system or message queue is almost always
opportunity-cost negative: you pour senior effort into a solved problem and starve the thing that
actually differentiates you.

**Total cost of ownership — the sticker price is the smallest part.** The naive build-vs-buy compares a
license fee to "free" (we'll build it). This is wrong on both sides. **Buying** costs more than the
license: integration, learning, operational dependence, and the *exit* cost (lock-in). **Building** costs
vastly more than the initial development: you now own it *forever* — maintenance, bug fixes, security
patches, upgrades, on-call, documentation, the ramp-up of every future engineer, and the opportunity cost
of all that ongoing effort. The honest comparison is **lifetime TCO** on both sides. A "free" in-house
build that consumes two engineers' attention indefinitely is often far more expensive than a paid SaaS —
and the reverse can also be true for a core capability. The discipline is counting the *whole* cost, not
the visible up-front one.

{: .warning }
> **Evaluate technology honestly — fit and reality over hype and résumé**
> New technology gets chosen for bad reasons constantly. The failure modes:
> - <strong>**Résumé-driven development**[°](#w-resume-driven-development)</strong> — picking the exciting new database/framework/language
>   because it's fun or good for the CV, not because it fits the problem. The tell: the technology was
>   chosen <em>before</em> the problem was understood.
> - <strong>Hype-cycle chasing</strong> — adopting what's trending (the new thing everyone's blogging
>   about) at its peak of inflated expectations, before its real trade-offs are known.
> The honest evaluation weighs, against the actual <strong>driver</strong> (Lesson 4, the ASR this
> technology must serve):
> - <strong>Fit</strong> — does it actually solve <em>this</em> problem well, or are you forcing it?
> - <strong>Maturity</strong> — proven in production at your scale, or bleeding-edge with unknown
>   failure modes?
> - <strong>Team skill</strong> — can your team operate and debug it at 3am, or is it a knowledge cliff?
> - <strong>Operational burden</strong> — what does running it actually cost (Lesson 27)?
> - <strong>Community & support</strong> — is there a healthy ecosystem, or will you be alone with it?
> - <strong>Lock-in</strong> — how hard is it to leave if it doesn't work out?
> The <strong>"**boring technology**[°](#w-boring-technology)"</strong> argument (Dan McKinley): treat novelty as a scarce resource —
> you get a few "innovation tokens" to spend on genuinely new tech; spend them where the novelty is
> <em>essential</em> to your core, and use proven, well-understood ("boring") technology for everything
> else, because boring tech has known failure modes, deep documentation, and a hiring pool. Every new
> technology you add is a permanent tax on operations, hiring, and cognitive load — the cost of "one
> more thing to run and understand" is real and recurring.

**Spikes and bake-offs — reduce uncertainty before committing.** When a technology decision is genuinely
uncertain and consequential (a one-way door, Lesson 2), don't decide from blog posts and vendor decks —
run a **spike** (a small, time-boxed experiment to answer a specific question: "can it handle our write
pattern?") or a **bake-off** (a head-to-head trial of two candidates against your real workload).
Time-box it, define the question and success criteria up front, and use a realistic workload — the goal
is to convert an argument-from-opinion into evidence, cheaply, before the expensive commitment. This is
evaluation (Lesson 30) applied to technology choice.

**The cost of every new technology, and the honest recommendation.** Adding a technology is never free —
it's a permanent addition to what your team must run, monitor, patch, hire for, and hold in their heads.
So the bar for "yes, add this" should be real fit to a real driver, not novelty. When you write the
**recommendation**, an architect's version names the option chosen *and the trade-offs accepted*, weighs
TCO on both sides, states whether it's core (build) or generic (buy), flags the lock-in, and is honest
about what it's giving up — the same discipline as an ADR (Lesson 29). *(This pairs with getting the
recommendation adopted — Lesson 34.)*

---

## Lab — Design Exercise

**The situation:** Your team wants to adopt a **trendy new database** (a recently-popular NoSQL store
everyone's talking about) for a **non-core feature** — an internal analytics/reporting dashboard that a
handful of internal users look at. The current stack is Postgres, which the team knows well. The
engineers are excited about the new database; one has been reading about it and wants to use it.

**Run the build-vs-buy and technology-selection analysis, and write the recommendation.** Address: is
this the right place to spend an "innovation token"? What's the TCO of adding a new database? What would
change your answer? Frame it so the excited engineer can hear it.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is separating the technology's merits from whether <em>this</em> is the right place to adopt it,
and writing a recommendation that's honest without crushing the engineer's enthusiasm. A strong answer:
<br><br>
<strong>Frame it against the driver, not the excitement.</strong> The first question isn't "is this
database good?" (it may be excellent) but "what problem does the analytics dashboard actually have that
Postgres can't solve?" For a handful of internal users looking at a reporting dashboard, the honest answer
is almost certainly <em>none</em> — Postgres handles this workload comfortably. The technology was chosen
before a problem that needs it was identified: the tell of <strong>résumé-driven / hype-driven</strong>
selection. So on <em>fit to the driver</em>, it fails — there's no driver demanding it.
<br><br>
<strong>Count the real TCO of adding a database.</strong> Adopting a new datastore is not free even if the
license is: it's a <em>permanent</em> new thing the team must operate, monitor, back up, patch, secure,
and debug at 3am; a new failure mode in production; a new thing every future engineer must learn; and a
new item on the on-call runbook — <em>forever</em>, in exchange for a non-core internal dashboard. Against
that recurring cost, the benefit (a slightly nicer fit for one internal feature, and some excited
engineers) is small. This is opportunity-cost and TCO negative.
<br><br>
<strong>Spend the innovation token wisely.</strong> The boring-technology argument: you get a few
"innovation tokens" for genuinely new tech; spend them where novelty is <em>essential to your core</em>,
and use boring/proven tech (here, the Postgres the team already runs) for undifferentiated things like an
internal dashboard. A non-core reporting feature is exactly the <em>wrong</em> place to spend a scarce
token — you'd take on a permanent operational and cognitive cost for something that doesn't differentiate
you at all.
<br><br>
<strong>What would change the answer.</strong> Be specific and fair: <em>if</em> the dashboard had a real
access pattern Postgres genuinely struggles with (say, a data-model or scale need that maps to what the
new database is <em>for</em> — Lesson 19), <em>and</em> that need was significant, then it becomes a real
fit-to-driver case worth a spike. Or if the team's strategy is to <em>deliberately</em> build expertise in
this database for a <em>future core</em> need, a low-risk internal feature could be a reasonable, conscious
place to learn it — but that should be an <em>explicit</em> decision (an ADR), named as such, not smuggled
in as "let's use the cool DB." Naming the conditions that would flip the recommendation shows it's
judgment, not reflexive conservatism.
<br><br>
<strong>The recommendation, framed so the engineer can hear it.</strong> "This database looks genuinely
interesting, and I don't want to shut down the enthusiasm — but adopting a new datastore is a permanent
operational and cognitive cost the whole team carries, and for an internal dashboard that Postgres serves
fine, I can't justify spending one of our scarce innovation tokens here. If you can point to a specific
thing Postgres can't do for this workload, let's run a small time-boxed spike and see real numbers —
that'd change my mind. And if part of the goal is to learn this DB for something bigger coming, let's make
<em>that</em> the explicit decision and pick the lowest-risk place to do it. For now, my recommendation is
Postgres for the dashboard, and let's find a better home for the new tech where it actually earns its
keep." — This validates the person, ties the decision to fit/TCO/tokens rather than authority, keeps a
door open (a spike, a future core use), and models that <em>everything is a trade-off</em> (Lesson 2) and
<em>every new technology is a cost</em>. (Getting it adopted gracefully is Lesson 34.)
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Choose boring technology | Dan McKinley — <https://mcfunley.com/choose-boring-technology> |
| Build vs buy / core vs generic | *Domain-Driven Design*, Evans (subdomains) ; Lesson 7 |
| Total cost of ownership | <https://en.wikipedia.org/wiki/Total_cost_of_ownership> |
| Résumé-driven development (avoiding) | <https://martinfowler.com/bliki/TechnicalDebt.html> (opportunity cost lens) |
| Spikes / time-boxed experiments | <https://www.thoughtworks.com/insights/blog/evolutionary-architecture> |
| Getting the recommendation adopted | [Lesson 34 — The architect as communicator](lesson-34-architect-as-communicator) |

---

## Checkpoint

**Q1.** What's the core-vs-generic heuristic for build-vs-buy, why is opportunity cost the right lens, and
why is TCO — not the sticker price — how you judge it?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Core vs generic:</strong> build what is your <em>core domain</em> — the capability that is your
competitive advantage, the reason customers choose you (DDD, Lesson 7) — because a bought/generic version
can't <em>be</em> your differentiator, so this is where your best people's effort actually pays off. Buy
or rent everything <em>generic/supporting</em> — auth, email, payments, search infrastructure, a CMS —
because it's undifferentiated table stakes that a vendor already does well; building it yourself pours
senior effort into a solved problem while starving the thing that actually makes you win.
<br><br>
<strong>Why opportunity cost is the lens:</strong> the question is almost never "can we build it?" (you
usually can) but "is this the best use of our <em>finite</em> building capacity?" Every hour spent
building something is an hour not spent on something else, so building an undifferentiated capability is a
loss <em>even if the build succeeds</em> — you got a thing you could have rented, at the price of not
building your differentiator. Opportunity cost reframes the decision from "is it possible / is it cheaper
up front" to "what are we giving up by spending our scarce capacity here," which is the question that
actually matters for a team that can only build so much.
<br><br>
<strong>Why TCO, not sticker price:</strong> the sticker price (a license fee, or "free, we'll build it")
is the smallest and most visible part of the cost. <em>Buying</em> also costs integration, learning,
operational dependence, and exit/lock-in. <em>Building</em> costs <em>vastly</em> more than the initial
development, because you own it forever: maintenance, security patches, upgrades, on-call, documentation,
and every future engineer's ramp-up — plus the opportunity cost of all that ongoing effort. So a "free"
in-house build can be far more expensive over its life than a paid SaaS, and the only honest comparison is
<em>lifetime total cost of ownership on both sides</em>, not the up-front number that happens to be easy
to see.
</details>

**Q2.** What is the "boring technology" argument and the innovation-token idea, and what should an honest
technology evaluation weigh instead of hype?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Boring technology / innovation tokens:</strong> the argument (Dan McKinley) is that novelty is a
scarce, expensive resource, so you should treat yourself as having only a few "innovation tokens" to
spend on genuinely new/unproven technology. Spend those tokens where the novelty is <em>essential</em> —
usually your core, where a new capability is a real advantage — and use <strong>boring</strong> (proven,
well-understood, widely-used) technology for everything else. Boring tech isn't inferior; it has known
failure modes, deep documentation, mature tooling, and a hiring pool, so it's <em>cheaper to run and
staff</em>. The insight behind it: every new technology you add is a <em>permanent</em> tax — one more
thing to operate, monitor, patch, hire for, and hold in the team's collective head — so the cost of
novelty is recurring and easy to underestimate, and most systems should be mostly boring with novelty
reserved for where it truly earns its keep.
<br><br>
<strong>What an honest evaluation weighs (against the driver, not the hype):</strong> instead of "it's
trending" or "it'd be fun / good on my CV" (hype-cycle and résumé-driven development — the tell is that
the tech was picked before the problem was understood), evaluate against the actual <em>driver</em>
(Lesson 4) the technology must serve:
<ul>
<li><em>Fit</em> — does it genuinely solve <em>this</em> problem, or are you forcing it?</li>
<li><em>Maturity</em> — proven at your scale, or bleeding-edge with unknown failure modes?</li>
<li><em>Team skill</em> — can your team run and debug it under pressure, or is it a knowledge cliff?</li>
<li><em>Operational burden</em> — what does running it actually cost?</li>
<li><em>Community & support</em> — healthy ecosystem, or alone with it?</li>
<li><em>Lock-in</em> — how hard to leave if it fails?</li>
</ul>
And when genuinely uncertain and consequential, replace opinion with evidence via a time-boxed
<em>spike</em> or a <em>bake-off</em> against a realistic workload before committing. The goal is to
choose for the problem, not for the excitement — because everything is a trade-off and every new
technology is a cost.
</details>

---

## Homework

Find a build-vs-buy decision in your system's past (or present): something your team built that's
arguably generic (an in-house auth system, a homegrown queue, a custom deploy tool), or something you
bought that's arguably core. Assess it honestly through opportunity cost and TCO: was it your
differentiator or table stakes, and did the *lifetime* cost (maintenance, on-call, ramp-up, the features
not built instead) get counted at decision time? Separately, look at your stack for a technology chosen
for hype or résumé reasons rather than fit — and one where "boring" would have served better. Identify the
single most expensive build-vs-buy or technology mismatch you carry, and what you'd do about it now.
Bonus: for a technology decision you're facing, design the spike that would replace the argument with
evidence.

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
A strong response applies the two lenses unsentimentally to real choices the team lives with.
<br><br>
<strong>Finds the mis-scoped build or buy.</strong> The instructive finding is usually something
<em>generic</em> the team built in-house (auth, a queue, a job scheduler, a deploy tool) that now costs
ongoing senior attention for no differentiation — the classic opportunity-cost and TCO error, where "free
to build" hid a permanent maintenance and on-call bill and the real cost was the features <em>not</em>
built instead. (Occasionally the reverse: something core that was bought and now can't be the advantage it
needed to be.) Recognizing that the <em>lifetime</em> cost — not the up-front build — is what made it a
bad trade is the insight.
<br><br>
<strong>Names a hype/résumé-driven technology choice.</strong> Being honest about a technology adopted for
excitement rather than fit — and one where boring, proven tech would have served better with less
operational and cognitive tax — is the maturity. The tell to look for: was the technology chosen before a
problem that needed it was identified?
<br><br>
<strong>Picks the most expensive mismatch and a move.</strong> The architect's discipline is to identify
the single highest-cost mismatch carried today and a realistic response — often either retiring an
in-house generic build in favour of a managed/bought equivalent (reclaiming the maintenance capacity), or
containing/replacing a poorly-fitting hyped technology. The bonus (designing a real spike — a specific
question, success criteria, realistic workload, time-box) shows the move from opinion to evidence. The
takeaway a good answer reaches: whether to build and which technology to use are among the highest-leverage
architectural decisions, both are usually made by reflex, and the antidotes are the same — judge by
opportunity cost and lifetime TCO, build your differentiator and rent the rest, spend novelty only where
it's essential, and replace argument with a cheap spike when the stakes are real.
</details>

---

## Words to Know

*Simple definitions and pronunciations for the terms marked ° above.*

- <a id="w-build-vs-buy"></a>**build vs buy** — decide whether to build a capability yourself, buy/rent it (SaaS, managed service, commercial product), or adopt open-source — or not do it at all.
- <a id="w-core-vs-generic-domain"></a>**core vs generic (domain)** — your *core* domain is your competitive advantage (build it); *generic/supporting* is undifferentiated (buy it). From DDD (Lesson 7).
- <a id="w-tco-total-cost-of-ownership"></a>**TCO (total cost of ownership)** — the full lifetime cost, not the sticker price: integration, operations, maintenance, upgrades, training, exit.
- <a id="w-opportunity-cost"></a>**opportunity cost** — what you *don't* get to build because you spent the time on this instead; the real lens for build-vs-buy.
- <a id="w-lock-in"></a>**lock-in** — how hard it is to leave a technology/vendor later; a cost you pay at the worst possible time.
- <a id="w-resume-driven-development"></a>**résumé-driven development** — choosing tech because it's exciting or looks good on a CV, not because it fits the problem.
- <a id="w-boring-technology"></a>**boring technology** — the argument for proven, well-understood tech; every novel technology spends a scarce "innovation token".
- <a id="w-spike-bake-off"></a>**spike / bake-off** — a small time-boxed experiment (or head-to-head trial) to reduce uncertainty before committing.

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 34 — The Architect as Communicator & Influencer →](lesson-34-architect-as-communicator){: .btn .btn-primary }
