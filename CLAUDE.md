# CLAUDE.md

A clickable UI/UX prototype for configuring **AI Journey** on a merchant account, in the Gorgias
**Axiom** design system. One self-contained `index.html`, ~2,550 lines, no build, no dependencies,
no network calls. Public at
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
- **The chart palette is validated**, not chosen by eye: `#0d6cf2` / `#c35e4a` / `#7e55f6`. If you
  change a series colour, re-run the dataviz validator and keep all six checks passing.
- **New interactive elements use `data-*` attributes and the delegated document click handler**, not
  per-node listeners — they must survive a re-render.

## The product model in six lines

- **ACE is a switch, not a scenario library.** Off / Manual acceptance / Automatic. The only dial is
  *domains*, and a domain that's off still shows what it would have been worth.
- **One instruction per unit**, and it's a skill. Badged `Default` or `Customized`.
- **Instructions ask; guardrails decide.** Guardrails run last and can refuse anything.
- **Suppression is timing; exclusion is people.** Two different blocks, deliberately.
- **Loyalty is the lever that's cheaper than margin** — which is why it lives inside the Engagement
  Center.
- **Conversations is one row per person**, led by why it started and what the engine reasoned from.
  Approvals live there; there is no separate queue.

Full vocabulary table at the end of `docs/PRODUCT.md` — use it, several terms were chosen against a
worse alternative.
