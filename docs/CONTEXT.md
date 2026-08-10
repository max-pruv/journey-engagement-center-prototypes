# Where the thinking came from

## Source material

The product model was reconstructed from Gorgias Notion documents, in August 2026. If you are
picking this up later, **re-read these rather than trusting this summary** — several were in
Discovery status and will have moved.

| Document | What it contributed |
| --- | --- |
| **AI Journey — Leadership Meeting** (the 7 July session) | The core architecture: *Engagement Center is the container; Default, Custom and ACE are the three engagement types inside it.* Also the UX principles — "Default: simple toggles · Custom: the merchant defines intent · ACE: activatable without feeling like an opaque black box" |
| **AI Journey: H2 2026 Direction & Prioritization** | The three trigger types (base always-on / AI-sourced / client-custom in natural language), and "a merchant UI with guardrails" as an explicit deliverable |
| **AI Journey — Autonomous Customer Engagement** (P1) | The marketer as *director* setting goals and guardrails; Discover → Decide → Activate; two-level learning |
| **Decide and Activate Per-Shopper Lifecycle Journeys** (P2) | Deterministic guardrails as hard constraints — frequency, suppression, quiet hours, consent. The event-source inventory with its honest gaps: 3PL, loyalty, advertising are not wired; on-site is partial |
| **AI Journey Pricing v0**, **AI Journey FAQs** | Vocabulary and framing |

Granola was the original ask for meeting context. Its local cache is encrypted as of v6
(`cache-v6.json.enc`, key in the macOS Keychain, non-standard scheme) and could not be read. The
same meetings were recoverable from Notion, which is where the material above came from.

## What is real and what is invented

**Real:** the architecture, the three engagement types, the guardrail categories, the signal-source
gaps, the "instructions ask, guardrails decide" distinction, and the strategic framing.

**Invented — all of it:** every number, every shopper, every conversation, every merchant name. The
demo account is **Alpine Supply Co.**, a fiction.

The first version of the prototype used the real alpha merchant names from the recruitment tracker.
Those were **deliberately replaced** when the repo was made public, and should stay replaced. If you
need a demo account, invent another one.

## The palette

The three-colour categorical set is `#0d6cf2` / `#c35e4a` / `#7e55f6` — Axiom core steps chosen by
running candidates through the dataviz validator until one passed all six checks. It was not picked
by eye, and it should not be changed by eye. See WORKFLOW.md.

Note that the categorical order here was chosen for this prototype. If Axiom has since published a
canonical categorical theme, that one wins.

## Design system

Axiom light-theme semantic tokens, Inter, the 2/4/6/8/12/16/24/32/48 spacing scale, and the standard
page shell: grey-50 page background, a rounded white content container with a 1px border, a sticky
`PageHeader` that grows a bottom border on scroll.

The prototype hand-rolls the components in plain CSS rather than importing `@gorgias/axiom`, because
it has to be a single portable file. The token values and the component behaviour follow the real
system; the implementations do not. **Do not treat this CSS as a reference for production work** —
build against the real package.

## Publication

The repo is public and served by GitHub Pages, at Max's request, so the prototype can be shared by
link. The content is a UI prototype with fictional data; it still describes unreleased product
direction. That trade-off was made knowingly. If it needs to stop being public, the repo can be
flipped private and the same file published as a private Artifact instead.
