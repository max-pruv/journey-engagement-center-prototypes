# CLAUDE.md

A clickable UI/UX prototype for configuring **AI Journey** on a merchant account, in the Gorgias
**Axiom** design system. Two self-contained pages — `index.html` (the product) and `onboarding.html`
(a conversational first-run setup) — no build, no dependencies, no network calls. Public at
https://max-pruv.github.io/journey-engagement-center-prototypes/ — pushing to `main` deploys.

Owner: Max Pruvost, VP of Product, R&D. The prototype is a **spec you can operate**, used to align
people in meetings — so copy is part of the deliverable, not decoration.

## Read these before changing anything

- `docs/PRODUCT.md` — every design decision, why it was made, and what it replaced. Several were
  wrong on the first pass; the "what it replaced" notes are the load-bearing part.
- `docs/CODEBASE.md` — code map, data objects, conventions.
- `docs/WORKFLOW.md` — the verification runbook. **Not optional; see below.**
- `docs/OPEN-QUESTIONS.md` — what's deliberately unresolved.
- `docs/CONTEXT.md` — source material, and what's real vs invented.

## The three things that will bite you

1. **The boot chain has no error handling.** If one render function throws, every render after it
   never runs, and the symptom is blank screens with no error surfaced to the user. This has happened
   twice, both times after deleting a DOM node a render function still wrote to. Run the isolation
   probe in `docs/WORKFLOW.md` after every change — all keys `ok`, all counts non-zero.

2. **A `file://` preview pane serves a stale snapshot** if you edited via shell (`python3`, `sed`)
   rather than an editor tool. Force a reload from disk before believing what you see. Console error
   buffers also survive reloads — trust the probe over the error list.

3. **Structural edits are safer as line-range splices with asserts on both boundaries.** The markup
   has a lot of near-identical blocks; every splice that skipped an assert landed in the wrong place.

Also: check the JS parses (`node -e …`, one-liner in WORKFLOW.md) before opening the page, and after
pushing confirm the live page is **byte-identical** to local — Pages gives no signal and takes up to
90 seconds.

## Non-negotiables

- **All data is fictional.** The demo account is "Alpine Supply Co.". Real alpha merchant names were
  deliberately removed when the repo went public. Do not reintroduce them.
- **Axiom tokens only** — no hex colours, no off-scale spacing, type through the `.h-*` / `.t-*`
  classes.
- **The chart palette is validated on ALL pairs**, not chosen by eye:
  `#7e55f6` / `#c35e4a` / `#149db8` / `#0e4ea7`. Re-run the validator with `--pairs all` if you
  touch it. Colour is bound to the entity, never to its rank.
- **Everything in Reporting derives from six weekly primitives per member.** Never hand-write a
  total; if the chart and the table can disagree, the change is wrong. The same rule now binds the
  theme table: `domTotals()` folds the plays, so `ACE_DOMAINS` carries no metrics of its own.
- **New interactive elements use `data-*` attributes and the delegated document click handler**, not
  per-node listeners — they must survive a re-render.

## The product model in six lines

- **ACE is a switch, not a scenario library.** Off / Manual acceptance / Automatic. The only
  account-wide dial is *themes*, and a theme that's off still shows what it would have been worth.
- **Two layers under the switch: a theme, and the plays inside it.** The standard engagements live
  there now — *Consideration & browse* holds Cart abandonment, Session abandonment, Browse
  abandonment, Welcome. A play is readable, instructable, and has its own switch — but that switch
  can only narrow *within* an allowed theme; the theme's own switch still overrides every play under
  it. Theme figures are folded from the plays, never hand-written. Click anywhere on a theme's row to
  expand or collapse the plays under it.
- **A custom engagement is When + What** — the trigger you own, then the skill. The trigger is
  a sentence first ("when someone buys a bike…"), same as campaign audience; exact rules are the
  fallback. Tab two is custom-only; nothing pre-built lives there any more.
- **One skill per unit.** Badged `Default` or `Customized`.
- **Skills ask; guardrails decide.** Guardrails run last and can refuse anything.
- **Suppression is timing; exclusion is people.** Two different blocks, deliberately.
- **Loyalty is the lever that's cheaper than margin** — which is why it lives inside the Engagement
  Center.
- **KPI cards are read-only and account-wide** — deliberately independent of the report below them.
- **Conversations is one row per person**, led by why it started and what the engine reasoned from.
  Approvals live there; there is no separate queue.
- **Every page is a link, tabs included.** `goto(page)` syncs `location.hash`; Engagements and
  Loyalty go one level deeper (`#engagements/scenarios`) so a link can point at one tab, not just
  the page — useful given the prototype's whole job is alignment in meetings.

Full vocabulary table at the end of `docs/PRODUCT.md` — use it, several terms were chosen against a
worse alternative.
