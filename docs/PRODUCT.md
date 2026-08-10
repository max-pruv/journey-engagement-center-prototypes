# The product argument

This prototype exists to settle a handful of questions about how a merchant configures AI Journey.
Each section below is a decision, why it was made, and — where it matters — what it replaced. The
"replaced" part is the useful part: several of these were wrong on the first pass and the reason
they were wrong is the reason the current shape holds.

---

## 1. ACE is one switch, not a library of scenarios

**Decision.** ACE (Autonomous Customer Engagement) is a mode, not a set of things you turn on.
Behind the switch, a chain of agents watches your data, finds a moment worth acting on, decides
whether *this specific person* should hear from you, writes the message, gets checked by guardrails,
and learns from the outcome. The merchant configures none of that.

**What it replaced.** A first pass modelled ACE as a feed of "opportunities" you activate one by
one, and ACE-sourced rows sitting in the engagements table alongside the configured ones. Both were
wrong for the same reason: they turned an autonomous system back into a configuration surface. If
the merchant has to pick the use cases, the machine isn't deciding — they are.

**Consequence.** There is no "Opportunities for you" screen anywhere. Every ACE decision is
per-shopper and unique to that shopper, so it belongs in a log of conversations, not a backlog of
templates.

## 2. The three modes are Off / Manual acceptance / Automatic

**Decision.** The modes describe *who presses send*, and are rendered as three full cards with a
one-line brief and a **Learn more** that carries the real explanation — what it does, what you'll
see, when to pick it, and the agent chain that runs underneath in every mode.

| Mode | Meaning |
| --- | --- |
| **Off** | ACE stops looking. Nothing queued is held; it drops, so nothing goes out days late |
| **Manual acceptance** | It does every bit of the work, then waits. Nothing sends without you. Items expire after 48h rather than going out stale |
| **Automatic** | Same work, no waiting. Guardrails still run last. You review after instead of before |

**What it replaced.** "Dry run" — which described a testing posture rather than an operating one.
The distinction that matters to a merchant is not *is this real* but *do I press send*. Also
replaced a four-level ladder (Off / Suggest / AI-supported / AI-driven), which drew a distinction
between "suggest" and "supported" that nobody could name.

**Why the cards, not a segmented control.** This is the single highest-stakes choice in the product
and the one that needs the most education. It had a 28px-tall control; now it has a third of the
screen and a Learn more panel.

## 3. Domains are the only dial — and refusing one has a visible price

**Decision.** Eight domains bound what ACE may work on: replenishment, retention, cross-sell,
service recovery, subscriptions, loyalty moments, consideration, seasonal. Turn one off and ACE
stops looking there entirely.

**Why they exist.** Enterprise control. A merchant who cannot bound an autonomous system will not
switch it on, and "all or nothing" is not a real choice at that size.

**The tension, and the resolution.** Domains give control but create a blind spot: you can't see
what you're declining. So a domain that is off does **not** show dashes. It shows its modelled
potential — shoppers reachable, GMV, conversion, opt-out — in purple italics, with a header tag
like `2 off · ≈$19,400 GMV left on the table`. The estimate is explicitly labelled as modelled on
comparable merchants rather than measured on you. Declining a domain stays a choice, but an
informed one.

**Metrics on every domain row:** shoppers reached, GMV, conversion rate, opt-out rate. Those four,
because they are the four a merchant actually argues about.

**Per-domain instruction.** A domain can carry its own optional instruction, for merchants who need
personalisation on one domain without touching the rest of the account. Same one-instruction rule
as everywhere else.

## 4. One instruction per unit, and it is a skill

**Decision.** Every engagement and every domain carries exactly **one** instruction. It is a skill
the agent loads when that thing fires. It has a badge: `Default` or `Customized`, and a
"Reset to default" when it has been edited.

**What it replaced.** A four-layer stack (brand → centre → mode → trigger) with a "resolved
instruction set" preview. Conceptually tidy, practically wrong: it invented a hierarchy the system
doesn't have, and it made every instruction feel like a negotiation with three invisible parents.

**The line that survived, and matters:** *instructions ask; guardrails decide.* An instruction can
request anything; the guardrails run last and can refuse it. That distinction is what makes the
model defensible in a demo, and it is shown literally in the live test (§6).

## 5. Naming: AI engagement / Custom engagement

**Decision.** The two tabs inside Engagements are **AI engagement** and **Custom engagement**. The
column that says who built a scenario is **Built by**, with values `Default` (pre-built) and
`Custom` (written by the merchant).

**What it replaced.** "Scenarios you own", which implied the merchant owns only half of this. They
own all of it. The name made the AI half feel like someone else's.

## 6. Opening an engagement opens a studio, not a summary

**Decision.** Clicking an engagement (or its Test button, or a domain's instruction) opens a wide
two-column panel: the skill on the left, a **live testable conversation** on the right. You pick a
shopper, run it, then reply *as the shopper* and watch what comes back — with the reasoning shown
under the thread.

**What it replaced.** "What it produced last time" — a static transcript. It told you nothing you
could act on and couldn't be interrogated.

**Why it earns the space.** It is where the guardrail model stops being a claim:

> *"what size should I get?"* → "Medium is the one — it sits under the tights without bunching…"
> **Note:** Answered the sizing question before selling. No offer used.

> *"any chance of a discount?"* → "I can't drop the price on this one, but shipping's on us this
> week — and you're 60 points from a free-shipping reward anyway."
> **Note:** Discount withheld: guardrail caps the AI at 10% and the loyalty lever was cheaper.

The second one demonstrates §4, §9 and §10 in a single exchange.

## 7. Two tabs, not three — and the switch is never inside a tab

**Decision.** Engagements has two tabs, with a sticky strip above them carrying ACE's state and,
in Manual acceptance, a "Review N waiting" action.

**Why three tabs were wrong.** Default / Custom / ACE split *the same kind of object* into
arbitrary buckets. That's a filter wearing a tab's clothing — and it's why Custom ended up looking
different from Default for no reason.

**Why two are right.** They separate two natures of work: a switch you trust, and a list you
maintain. That is a real boundary.

**The condition.** A mode this important cannot be hidden behind tab selection, so its state lives
in the strip and is readable from either tab.

## 8. Campaign creation is a three-step conversation

**Decision.** Who → What → How, as a stepper. Each step opens with a question the agent asks, you
answer in plain language, and the answer expands into a structured result you can edit. Audience
can be re-read **as rules** if you'd rather be exact. Instructions are A/B tested: three approaches,
removable and addable, and a rollout rule — send 10%, wait an hour, send the winner to the rest,
unless nothing leads by more than 5%, in which case hold and say so.

**What it replaced.** One dense screen with everything on it.

**Order matters:** "Describe it" is the first and default audience mode; rules are the fallback.

## 9. Guardrails separate timing from people

**Decision.** Two distinct blocks.

- **Suppression** is about *timing*: don't message this person *right now* (open ticket, recent
  refund, live conversation, frequency, quiet hours).
- **Who we never contact** is about *people*: they are out of every audience, permanently or until
  cleared. Left a review of 3★ or less. Rated a support conversation 1 or 2 — AI **or** human.
  Opened a dispute. Asked not to be contacted in words, without a formal STOP. Returned more than
  half of what they bought. Wholesale, press, employees, test accounts. Flagged do-not-market.

Three suppression rules and two exclusion rules are locked and cannot be turned off.

## 10. Levers are configured objects, not labels

**Decision.** Each lifecycle stage carries an ordered list of levers, and each lever is an object
you configure. The discount lever has type (% / fixed / free shipping), value, code strategy
(unique per shopper / shared pool), expiry and minimum order. Points boost has a multiplier and a
duration. Early access has a window. Tier fast-track has a jump and a hold period.

Levers are badged `cheap` or `costs margin`, and the guardrails carry an explicit order of
preference: answer the objection → early access → points boost → free shipping → **discount last**.

**What it replaced.** Chips with lever names on them, which implied a choice the merchant couldn't
actually make.

## 11. Lifecycle is "who is this" plus "how do we treat them"

**Decision.** Master/detail over seven stages: Visitor, Lead, First-time buyer, Repeat, VIP,
At risk, Dormant. Each carries what puts someone there, and then how they're treated: objective
(a dropdown of specific outcomes, not free text), tone (a **prefilled textarea** per stage), what
we're willing to give, a message cap as a value plus a timeframe unit, the lever objects, and
which sources may touch the stage.

**What it replaced.** A "your primary goal this quarter" section — removed. There isn't one goal;
the goal is revenue. And free-text fields where a dropdown of known-good options is kinder.

**"See the N shoppers"** opens a searchable drawer; clicking a shopper opens their profile. In the
real product that is a new tab, and the prototype says so.

## 12. Conversations is a marketing inbox

**Decision.** Not a table of events — an inbox. List on the left with search and filters (Waiting
for you / Live / Closed / Stopped, and a source filter), thread on the right. Every thread opens
with:

1. **Why this conversation started**, in plain language, badged with source and domain
2. **What it was reasoning from** — lifecycle stage, loyalty state, timing chosen, lever picked
3. The conversation, or the exact guardrail that stopped it

Approvals live here. There is no separate approval queue; "Waiting for you" is a filter.

**Why loyalty appears in the reasoning.** It is the cheapest lever and the least obvious one. Seeing
*"Member · 900 points, enough for free shipping → points before discount, costs you $6.40 instead
of $12"* is what makes the loyalty integration legible.

## 13. Loyalty is inside the Engagement Center on purpose

**Decision.** Tiers are editable — entry conditions you compose (spend in 12 months, orders,
points balance, returns rate), earn rate, perks, and the lifecycle stage the tier maps to, so "VIP"
means one thing across the product. Each tier reports members, AOV, LTV, repeat rate and points
outstanding. Each reward shows **your** cost next to its points cost, and carries a toggle for
whether the AI may offer it unprompted.

**The argument.** A discount costs margin every time. Points, early access and a tier bump cost far
less, and on the at-risk cohort they converted better. Loyalty is what lets the engine be generous
without being expensive — which is why it sits here and not in a separate product area.

---

## Vocabulary

Use these consistently; several of them were chosen against a worse alternative.

| Term | Means | Not |
| --- | --- | --- |
| **Engagement** | Anything that reaches out to a shopper | "flow", "journey" |
| **AI engagement / ACE** | The autonomous mode | "AI-found engagements", a list of scenarios |
| **Custom engagement** | The scenarios the merchant defines | "scenarios you own" |
| **Domain** | The bound on what ACE may work on | "category", "use case" |
| **Instruction** | The single skill attached to a unit | "prompt", "instruction stack" |
| **Guardrail** | A hard limit no instruction can unlock | "setting" |
| **Suppression** | Don't message them *now* | exclusion |
| **Exclusion** | Never contact them at all | suppression |
| **Lever** | A configured thing the engine may offer | "incentive" |
| **Manual acceptance / Automatic** | Who presses send | "dry run", "AI-supported" |
