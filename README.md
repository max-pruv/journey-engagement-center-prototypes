# Journey Engagement Center Prototypes

A clickable UI/UX prototype for configuring **AI Journey** on a merchant account, built with the
Gorgias **Axiom** design system as a single self-contained HTML file.

**Live:** https://max-pruv.github.io/journey-engagement-center-prototypes/

Its job is to make an argument about how the product should work, in a form you can click through
in a meeting. It is not a codebase to ship — it is a spec you can operate.

> All data is fictional. The demo account, "Sample Store Co.", and every number, shopper,
> conversation and merchant name in it are invented. See [docs/CONTEXT.md](docs/CONTEXT.md) for
> what came from real source material and what didn't.

## Where to start

| If you want to… | Read |
| --- | --- |
| Understand the product argument and why each screen looks like this | [docs/PRODUCT.md](docs/PRODUCT.md) |
| Change the prototype without breaking it | [docs/CODEBASE.md](docs/CODEBASE.md) |
| Verify and deploy | [docs/WORKFLOW.md](docs/WORKFLOW.md) |
| Know what's still undecided | [docs/OPEN-QUESTIONS.md](docs/OPEN-QUESTIONS.md) |
| See where the thinking came from | [docs/CONTEXT.md](docs/CONTEXT.md) |

If you are an AI agent picking this up, read [CLAUDE.md](CLAUDE.md) first — it is the short version
of all of the above, plus the failure modes that have actually bitten in this repo.

## Two pages

| File | What it is |
| --- | --- |
| `index.html` | The product itself |
| `onboarding.html` | A conversational first-run setup — it infers the store profile, recommends each decision, and teaches ACE through an interactive video tour before asking for the autonomy mode |

## Running it

```bash
open index.html
```

No build, no dependencies, no network calls. Tokens and CSS, then the markup for every screen, then
the data and render functions.

## The eight surfaces

| Surface | What it is |
| --- | --- |
| **Reporting** | Read-only KPI cards, then a report you drive: any metric × any dimension × smooth interactive time series / ranked / table |
| **Opportunities** | A conversational action inbox for detected campaigns, configuration changes, ACE modes, lifecycle gaps and loyalty improvements |
| **Engagements** | Two tabs. **AI engagement** — ACE: one mode switch plus domains. **Custom engagement** — the scenarios the merchant defines, pre-built or self-written |
| **Campaigns** | AI-detected opportunities, past campaigns, then a fully conversational build — Who / What / How as a status bar, everything else is question and answer. The story becomes a detailed instruction; the first 30 outbound messages can be reviewed as a scrollable send-order feed before launch |
| **Guardrails** | Detected opportunities above seven categories: frequency, timing, suppression, exclusions, offers, content, volume & proof |
| **Lifecycle** | Detected opportunities, then what defines each stage and how it is treated — objective, tone, what we're willing to give, configurable levers |
| **Conversations** | A marketing inbox. One row is one person: why it started, what the engine reasoned from, what was sent |
| **Loyalty** | Detected opportunities, then two tabs — an overview of where members sit, and a configuration master/detail for tiers, earn rules and rewards |

## The three claims it is making

1. **ACE is a switch, not a scenario library.** Behind it, a chain of agents watches, finds, decides,
   writes, gets checked by guardrails, and learns. The merchant configures nothing. The only dial is
   *domains* — and a domain that is off still shows what it would have been worth.
2. **One instruction per unit.** It's a skill attached to that context, carrying a badge for whether
   it's still the default or has been customized. Instructions ask; guardrails decide.
3. **Loyalty is what lets the engine be generous without being expensive.** Points, early access and
   a tier bump cost far less than margin, and on the at-risk cohort they win.
