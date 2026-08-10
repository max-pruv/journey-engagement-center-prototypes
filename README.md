# Journey Engagement Center Prototypes

Clickable UI/UX prototype for configuring **AI Journey** on a merchant account, built with the
Gorgias **Axiom** design system (tokens, spacing scale, component patterns) as a single
self-contained HTML file.

**Live:** https://max-pruv.github.io/journey-engagement-center-prototypes/

All data is fictional. The demo account (“Alpine Supply Co.”) and every number, shopper and
conversation in it are invented for the mockup.

## The story it tells

| Surface | What it is | What the merchant does |
| --- | --- | --- |
| **Overview** | Volume by source, conversion by source, revenue/GMV table, benchmark vs category | Reads |
| **Engagements** | **ACE** (one switch, domains as the only dial) on top, then the scenarios the merchant owns — Default and Custom in one list | Switches ACE on; turns scenarios on/off; edits one instruction each |
| **Campaigns** | Past campaigns, then a builder: audience by rules *or* natural language, and three instructions A/B-tested for the send | Creates one-offs |
| **Guardrails** | Frequency, quiet hours, suppression, what may be offered, what may be said, volume & proof | Sets the hard limits |
| **Lifecycle** | What defines each stage, and how each stage is treated: objective, tone, what we're willing to give, levers, caps | Defines the model |
| **Activity** | One row per person: what was decided, why, what was sent — or why it wasn't | Reads, approves in dry run |
| **Loyalty** | Tiers mapped to lifecycle stages, earn rules, reward catalogue | Gives the engine levers cheaper than margin |

## Three ideas the prototype is arguing for

1. **ACE is a switch, not a scenario library.** Behind it, a chain of agents watches, finds,
   decides, writes, gets checked by guardrails, and learns. The merchant configures nothing.
   The only dial is *domains* — turn one off and ACE stops looking there.
2. **One instruction per engagement.** It's a skill attached to that context, carrying a badge
   for whether it's still the default or has been customized. Instructions ask; guardrails decide.
3. **Default and Custom deserve the same UI.** The only thing that differs is that a custom
   engagement is created by describing it in natural language.

## Design notes

- Axiom light-theme semantic tokens; Inter; the 2/4/6/8/12/16/24/32/48 spacing scale.
- Categorical chart palette validated on the six checks (lightness band, chroma floor,
  CVD ΔE, normal-vision floor, contrast): `#0d6cf2` Default · `#c35e4a` Custom · `#7e55f6` ACE.
- Charts are hand-rolled inline SVG with hover tooltips, direct labels and a table view — no
  chart library, no external requests.

## Running it

Open `index.html`. No build, no dependencies, no network calls.
