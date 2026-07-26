---
title: "Lesson 34 — The Architect as Communicator & Influencer"
nav_order: 2
parent: "Phase 8: The Architect in Practice"
---

[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }

---

# Lesson 34: The Architect as Communicator & Influencer

{: .note }
> **Words to know**
> - **influence without authority** — getting people to do something when you don't manage them; the architect's normal condition.
> - **selling the why** — persuading with the reasoning and trade-offs behind a decision, not decreeing the *what*.
> - **buy-in** — genuine agreement and commitment from the people who'll implement, not mere compliance.
> - **ivory tower** — the disconnected architect who dictates from above and never touches the code; the anti-pattern.
> - **disagree and commit** — after a fair hearing, backing a decision you argued against, so the team can move.
> - **architecture guild / review** — a forum where architects and senior engineers align on standards and review designs together.
> - **gardener, not dictator** — the model of an architect who cultivates good decisions across teams rather than commanding them.

## Concept

Here is the humbling truth of the job: **a technically brilliant architecture that the organization
doesn't build is worth nothing.** The architect's decisions are implemented by teams the architect
usually doesn't manage — so architecture is not just a technical act, it's a **social** one, and the
scarce skill isn't producing the right design, it's getting the right design *adopted*. You almost never
have the authority to command it; you have to earn it through influence.

```
   THE DECISION IS WORTHLESS UNTIL IT'S ADOPTED

   IVORY TOWER (fails)                    GARDENER (works)
   ┌──────────────────────────┐           ┌──────────────────────────────────┐
   │ architect decrees the WHAT│           │ architect sells the WHY + trade-offs│
   │ from above, never codes   │           │ meets each audience where it is:   │
   │ throws design over wall   │           │   exec · PM · engineer             │
   │  → teams resist / ignore  │           │ influence WITHOUT authority        │
   │  → "not invented here"    │           │ disagree & commit · wrong gracefully│
   │  → the design dies        │           │  → teams OWN it → it gets built    │
   └──────────────────────────┘           └──────────────────────────────────┘
       power you don't have                    trust & reasoning you build
```

The shift is from *deciding* to *influencing*. You **sell the why** (the reasoning and the trade-offs)
rather than decree the what — because people build what they *understand and believe in*, not what
they're told. You **meet each audience where they are** (an exec, a PM, and an engineer need the same
decision explained three different ways). You stay out of the **ivory tower** by keeping your hands close
enough to the ground that your designs are credible and buildable. And you accept that being an architect
is being a **gardener, not a dictator** — you cultivate good decisions across teams you don't own, which
means you must be able to be *disagreed with*, to *disagree and commit*, and to be *wrong gracefully*.
*(This lesson pairs with the entire [Leadership]({{ '/leadership/learning-plan.html' | relative_url }})
track's communication and influence phases — the people-judgment half of the architect's job.)*

## Going Deeper

**Architecture is a social act: influence without authority.** In most organizations the architect
doesn't manage the teams that implement the architecture — you have responsibility without command
authority. So the currency is **influence**, not power: your designs get built because people trust your
judgment and understand your reasoning, not because you can order them to. This means the classic
technical-leadership skill (Leadership Phase 7, influence without authority) is *central* to
architecture, not incidental. The architect who says "I decided, so build it" and expects compliance is
operating on authority they don't have, and their designs quietly die of resistance, foot-dragging, and
"not invented here."

**Sell the why and the trade-offs, not the what.** The instinct is to communicate the *decision* ("use a
saga here"). But people implement what they *believe in*, and belief comes from understanding the
**reasoning**: the driver it serves, the alternatives considered, and the trade-off consciously made
(exactly the ADR content, Lesson 29). When you sell the *why* — "we split the data for independent
deployability, which cost us the shared transaction, so we need a saga; here's why that trade is worth it
here" — the team can *evaluate* it, *improve* it, and *own* it. When you decree the *what*, you get, at
best, compliance without understanding (they'll implement it wrong the first time they hit a case you
didn't specify) and, at worst, resentment. Selling the why also makes you *correctable*: if your
reasoning has a flaw, letting people see it means they can catch it — which is a feature, not a
vulnerability.

{: .warning }
> **The ivory tower vs the architect who stays close to the ground**
> The most damaging architect stereotype is the <strong>ivory-tower architect</strong>: someone who
> dictates designs from above, produces diagrams disconnected from reality, doesn't write code or feel
> the consequences of their decisions, and "throws the design over the wall" for others to build. It
> fails on two levels — the designs are <em>worse</em> (out of touch with real constraints, current
> tooling, and the actual pain of the code), and they're <em>resisted</em> (engineers rightly distrust
> decisions from someone who won't live with them). The antidote isn't that the architect must code
> full-time (Lesson 12 — that's a different balance), but that they stay <strong>close enough to the
> ground</strong> to keep their designs credible and buildable: pairing occasionally, doing a spike,
> reviewing real PRs, feeling the friction. Credibility is earned by proximity to the work; an architect
> who has lost touch with the code has lost the basis of their influence. Prefer being the architect who
> still gets their hands dirty over the one who only draws boxes.

**Meet each audience where they are.** The same architectural decision must be communicated *differently*
to different audiences, because they care about different things (Lesson 28's "views for audiences,"
applied to conversation):
- **Executives** care about business impact, cost, risk, and time — frame the decision in those terms
  ("this lets us hold Black Friday without a re-platform; it costs X and buys us Y"), not in technical
  detail.
- **Product managers** care about what it enables and constrains for the roadmap and the user — frame it
  as capability and trade-off in their terms.
- **Engineers** care about the technical substance, the reasoning, and how it affects their work — give
  them the real depth, the alternatives, and room to push back.
Using the same pitch for all three fails all three. Translating a decision into each audience's concerns
is a core architect skill — and it's *not* dumbing down; it's respecting what each audience actually needs
to decide.

**Running reviews and guilds; disagree-and-commit; being wrong gracefully.** Influence at scale happens
through *forums*: an **architecture review** or **guild** where designs are discussed and standards
aligned collaboratively (not handed down) builds shared ownership and catches problems early (Lesson 30's
evaluation, done socially). Two dispositions make the whole thing work. **Disagree and commit:** after a
fair hearing, the architect (and everyone) must be able to fully back a decision they argued against, so
the team can move — endless relitigation is its own failure. And **being wrong gracefully:** an architect
who can't be corrected, who defends a decision past the evidence to protect ego, destroys the trust their
influence runs on; conversely, an architect who says "you're right, I missed that, let's change it" *gains*
credibility. The whole stance is **gardener, not dictator** — you cultivate good decisions across teams by
creating the conditions (shared reasoning, forums, standards, trust) in which they grow, rather than
commanding outputs you have no authority to command. *(All of this is the [Leadership]({{
'/leadership/learning-plan.html' | relative_url }}) track's material — influence, feedback,
disagree-and-commit — which is why an architect needs the people-judgment as much as the technical.)*

---

## Lab — Design Exercise

**The situation:** You've proposed a **shared platform** — a common service (say, a shared
authentication + user-profile service, or a shared data-ingestion pipeline) that two senior teams would
adopt instead of each maintaining their own. You believe it's the right call (less duplication, one place
to secure and operate, consistency). But **both senior teams are skeptical**, for *different* reasons:
- **Team A** is worried about **losing control and velocity** — depending on a shared service run by
  someone else means their roadmap is now hostage to another team's priorities and reliability.
- **Team B** is skeptical it will actually **fit their needs** — they think the shared service will be a
  lowest-common-denominator that doesn't handle their specific requirements, and they'll be worse off.

You do *not* manage these teams. **Plan how you'd build buy-in** — explicitly *not* how you'd overrule
them.

**Your response:**

<details>
<summary>Show Model Answer</summary>
<br>
The skill is treating this as an influence problem, addressing each team's <em>actual, different</em>
concern, and resisting the urge to invoke authority you don't have. A strong answer:
<br><br>
<strong>Start by listening, not pitching.</strong> Before defending the platform, genuinely understand
each team's objection — because they're different and both may be <em>right</em>. Team A's fear (losing
velocity/control to a dependency) and Team B's fear (poor fit) are legitimate risks of shared platforms,
not obstacles to steamroll. Treating them as real surfaces whether the platform is actually a good idea
and earns the right to be heard. (If, on listening, you discover the platform genuinely doesn't fit, the
mature move is to <em>change your proposal</em> — being wrong gracefully — not to push a bad design
through.)
<br><br>
<strong>Sell the why, and address each concern specifically:</strong>
<ul>
<li><em>To Team A (control/velocity):</em> the answer isn't "trust me," it's <em>de-risking the
dependency</em>. Concrete commitments: strong SLAs on the shared service, a clear roadmap-input process
so they're not hostage to it, good self-service so they're not blocked waiting on the platform team, and
perhaps a governance seat. Show that the shared platform <em>increases</em> their velocity for the
undifferentiated part (they stop maintaining their own auth — Lesson 33) so they can spend their capacity
on their actual product. Address the real fear, don't dismiss it.</li>
<li><em>To Team B (fit):</em> the answer is to <em>involve them in the design</em> so it isn't a
lowest-common-denominator imposed on them — bring their requirements in as first-class inputs, design
extension points for team-specific needs, and maybe pilot with them so it's shaped by a real hard case.
Co-designing turns "a thing done to us" into "a thing we helped build," which is both better fit and
genuine buy-in.</li>
</ul>
<strong>Meet each audience where they are and use a forum.</strong> Frame the value in each team's terms
(velocity reclaimed for A, fit + influence for B), and run the discussion as a collaborative
<em>architecture review</em>, not a verdict — let them shape it, poke holes, and co-own the outcome.
Consider a <em>pilot / incremental adoption</em> (strangler-style, Lesson 32) so the platform earns trust
by working for one real case before anyone bets their roadmap on it — evidence beats argument.
<br><br>
<strong>Be willing to lose, and to disagree-and-commit.</strong> Because you don't manage these teams,
the only durable path is genuine buy-in — and that means accepting they might convince <em>you</em> that
the platform is wrong (or wrong for now), and being willing to change or drop it gracefully. If, after a
fair process, the org decides to proceed and one team still disagrees, ask for disagree-and-commit — but
you can only credibly ask that after they've been genuinely heard and their concerns addressed. The whole
plan is <strong>gardener, not dictator</strong>: you cultivate the conditions (heard concerns, co-design,
de-risked dependency, a pilot, shared reasoning) under which buy-in grows, rather than trying to command
adoption you have no authority to command — because a shared platform that teams resent will be worked
around, starved, or sabotaged, and a technically-correct platform nobody genuinely adopts is worth
nothing.
</details>

---

## Further Reading

| Topic | Source |
|---|---|
| Influence without authority | [Leadership Phase 7]({{ '/leadership/lessons/phase-07-influence.html' | relative_url }}) |
| Building buy-in | [Leadership Lesson 36]({{ '/leadership/lessons/lesson-36-building-buy-in.html' | relative_url }}) |
| The architect elevator (staying connected) | Gregor Hohpe — <https://architectelevator.com/> |
| Selling the why; architecture as communication | *Fundamentals of Software Architecture*, Richards & Ford (soft skills chapters) |
| Disagree and commit | <https://en.wikipedia.org/wiki/Disagree_and_commit> |
| Running design reviews (people side) | [Leadership Lesson 9]({{ '/leadership/lessons/lesson-09-design-reviews.html' | relative_url }}) |

---

## Checkpoint

**Q1.** Why is architecture "a social act," and why do you sell the *why* and trade-offs rather than
decree the *what*?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>Architecture is a social act</strong> because the architect's decisions are implemented by teams
the architect usually doesn't manage — so a technically brilliant architecture the organization doesn't
actually build is worth <em>nothing</em>. The scarce skill isn't producing the right design, it's getting
it <em>adopted</em>, and since you rarely have the authority to command adoption, the currency is
<strong>influence, not power</strong>: your designs get built because people trust your judgment and
understand your reasoning. The architect who says "I decided, build it" is operating on authority they
don't have, and their designs die of resistance, foot-dragging, and "not invented here." This makes the
people-judgment (influence, communication, being disagreed with) central to architecture, not a soft
add-on.
<br><br>
<strong>Sell the why, not the what,</strong> because people implement what they <em>understand and
believe in</em>, not what they're merely told. When you communicate only the decision ("use a saga"), you
get compliance without understanding — the team implements it wrong the first time they hit a case you
didn't specify, and feels no ownership. When you sell the <em>reasoning and the trade-off</em> ("we split
the data for independent deploys, which cost the shared transaction, so we need a saga, and here's why
that trade is worth it here"), the team can evaluate it, improve it, and <em>own</em> it — they're now
equipped to make good decisions in the cases you didn't foresee, because they understand the intent.
Selling the why also makes you <em>correctable</em>: exposing your reasoning lets people catch a flaw in
it, which is a feature — it's how the design gets better and how you keep the trust your influence depends
on. Decreeing the what hides the reasoning, prevents ownership, and forfeits the correction.
</details>

**Q2.** What is the ivory-tower anti-pattern and its antidote, and why must you communicate the same
decision differently to executives, PMs, and engineers?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
<strong>The ivory-tower architect</strong> dictates designs from above while disconnected from the actual
work — doesn't write code or feel the consequences, produces diagrams out of touch with real constraints
and tooling, and throws the design "over the wall" for others to build. It fails twice: the designs are
<em>worse</em> (uninformed by the real pain, friction, and constraints of the code), and they're
<em>resisted</em> (engineers rightly distrust decisions from someone who won't live with them). The
<strong>antidote</strong> is staying <em>close enough to the ground</em> to keep designs credible and
buildable — pairing occasionally, doing a spike, reviewing real PRs, feeling the friction. It's not that
the architect must code full-time (that's a separate balance, Lesson 12), but that <em>credibility is
earned by proximity to the work</em>: an architect who has lost touch with the code has lost the basis of
their influence. Better to be the architect who still gets their hands dirty than the one who only draws
boxes.
<br><br>
<strong>Why tailor the communication:</strong> executives, PMs, and engineers care about different things,
so the same decision must be framed in each one's concerns (the conversational version of Lesson 28's
"views for audiences"). <em>Executives</em> care about business impact, cost, risk, and timing — frame it
as "this lets us hold Black Friday without a re-platform; costs X, buys Y." <em>PMs</em> care about what
it enables/constrains for the roadmap and users — frame it as capability and trade-off in their terms.
<em>Engineers</em> care about the technical substance and reasoning — give them the real depth,
alternatives, and room to push back. One pitch for all three fails all three (too technical for the exec,
too vague for the engineer). Translating a decision into each audience's actual concerns isn't dumbing
down — it's respecting what each needs in order to say yes, and it's how you get buy-in across the whole
organization rather than from just one part of it.
</details>

---

## Homework

Think of an architectural decision or proposal of yours that *didn't* get adopted, or was adopted
grudgingly and half-heartedly. Honestly diagnose why through this lesson's lens: did you sell the *why*
and the trade-offs, or decree the *what*? Did you meet each audience where they were, or use one pitch?
Did the implementing teams — whom you don't manage — feel heard and involved, or overruled? Were you
operating on authority you didn't actually have? Then pick a *current* proposal you need adopted and plan
the influence campaign: whose buy-in you need, each person's actual concern, how you'd frame it for each
audience, what forum you'd use, and where you'd be genuinely willing to be wrong or to compromise. Bonus:
notice whether you've drifted toward the ivory tower — when did you last feel the friction of the code
your designs create?

**Your answer:**

<details>
<summary>Show Answer</summary>
<br>
A strong response is honest about a real failure and treats adoption as an influence problem.
<br><br>
<strong>Diagnoses the failed adoption truthfully.</strong> The common, uncomfortable finding: the design
was <em>right</em> but the architect decreed the <em>what</em> without selling the <em>why</em>, used one
technical pitch for every audience (losing the execs and the PMs), and — the big one — expected teams they
don't manage to comply on authority they didn't have, so the design met resistance or grudging
half-adoption. Recognizing that a technically-correct design that isn't adopted is a <em>failure</em>, and
that the failure was social not technical, is the growth.
<br><br>
<strong>Plans a real influence campaign.</strong> For a current proposal, the architect's move is to map
it as people, not just boxes: whose buy-in is actually needed, each stakeholder's <em>specific</em>
concern (control? fit? cost? risk?), how the decision should be framed for each audience (business terms
for execs, capability/roadmap for PMs, technical depth for engineers), which forum builds shared ownership
(an architecture review, a pilot), and — crucially — where they're genuinely willing to be convinced they're
wrong or to compromise. Planning to <em>listen and co-design</em> rather than to <em>win</em> is the
gardener-not-dictator stance.
<br><br>
<strong>Checks for ivory-tower drift.</strong> The honest self-assessment — when did I last feel the real
friction of the code my designs create; have my diagrams drifted from reality? — guards the credibility
that all the influence depends on. The takeaway a good answer reaches: the technical decision is worthless
until it's adopted; adoption is an influence problem solved by selling the why, meeting each audience where
it is, involving the people who'll build it, staying close enough to the ground to be credible, and being
able to disagree-and-commit and be wrong gracefully — the people-judgment half of the architect's job that
pairs with everything technical in this track.
</details>

---

<!-- nav-next -->
[← Home]({{ '/' | relative_url }}){: .btn .btn-outline }
[Next: Lesson 35 — Architecture Anti-Patterns & Pitfalls →](lesson-35-antipatterns){: .btn .btn-primary }
