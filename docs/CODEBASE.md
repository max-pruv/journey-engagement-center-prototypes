# Codebase map

One file: `index.html`, ~2,550 lines, no build, no dependencies, no network calls. Three zones in
order: **tokens + CSS**, **markup for every screen**, **data + render functions**.

Line numbers below drift as soon as you edit. Grep for the marker instead — every section has one.

## Layout of the file

| Zone | Find it with | What's there |
| --- | --- | --- |
| Tokens | `AXIOM TOKENS` | Axiom core colours, semantic tokens, spacing scale, the validated dataviz palette |
| CSS | after `*{box-sizing` | Utilities, then components, each with a comment header |
| Markup | `<!-- ======== ` | One `<section class="page" id="page-*">` per screen |
| Data | `const [A-Z_]* =` | Every list the UI renders from |
| Render | `function render*` | One per screen or block |
| Interaction | `/* ACE — the switch` onwards | Delegated click handler, filters, tabs, nav |
| Boot | `/* boot */` | The render chain — **read the failure mode below** |

## Screens

Each is a `<section class="page">`; `goto(page)` unhides one and hides the rest.

| id | Nav item | Rendered by |
| --- | --- | --- |
| `page-overview` | Reporting | `renderReport` → `renderTiles`, `renderRevTable`, `stackedBars`/`lines`/`hBars`; plus `renderBench` |
| `page-engagements` | Engagements | `renderAce` (AI tab), `renderEng` (Custom tab) |
| `page-campaigns` | Campaigns | `renderCampaigns` |
| `page-campaign-new` | (via Create) | `renderWiz`, `renderRules`, `renderVariants` |
| `page-guardrails` | Guardrails | `renderGuardrails` (master/detail over `GR_CATS`) → `renderSuppression`, `renderExclusions` |
| `page-lifecycle` | Lifecycle | `renderLifecycle` |
| `page-conversations` | Conversations | `renderConv`, `renderThread` |
| `page-loyalty` | Loyalty | `renderLoyalty` — overview tab plus a master/detail over tiers, earn rules and rewards |
| `page-soon` | — | placeholder for anything not built |

`NAV_OF` maps a page to the nav item that should look active — that's how `campaign-new` keeps
Campaigns highlighted.

## Data objects

Change these, not the markup, when you want different content.

| Const | Drives |
| --- | --- |
| `ENG` | The Custom engagement table. `s` is `Default` or `Custom`, `ins` is `Default` or `Customized`, `st` is `on`/`off`/`setup` |
| `ACE_DOMAINS` | The domain table. Real metrics (`reach`, `gmv`, `cvr`, `opt`) **and** the modelled ones used when the domain is off (`potReach`, `potGmv`, `potCvr`, `potOpt`) |
| `ACE_MODES` | The three mode cards. `sum` is the headline, `brief` the one-liner on the card, `det`/`see`/`who` fill the Learn more panel |
| `ACE_CHAIN` | The six agent steps, shown inside Learn more |
| `CONV` | The conversations inbox. `why` is the plain-language reason, `reason` the key/value reasoning rows, `thread` the messages, `stop` the guardrail that blocked it |
| `STAGES` | The seven lifecycle stages. `lever` holds indexes into `LEVERS`; `capN`/`capU` are the message cap value and unit |
| `LEVERS` / `LEVER_CFG` | Lever names, and per-lever configuration fields keyed by lever index |
| `TONE_TEXT` | The prefilled tone textarea, keyed by stage name |
| `OBJECTIVES`, `DISCOUNTS`, `CAP_UNITS` | The dropdown option lists on a lifecycle stage |
| `SUPPRESSION` / `EXCLUSIONS` | The two guardrail blocks — timing vs people. `locked: true` means it can't be toggled |
| `TIERS`, `EARN`, `REWARDS` | Loyalty. Tiers carry structured `cond` rows so the editor opens populated |
| `DIMS` | The reporting dimensions. Each member carries a weekly base, a growth rate and **its own rates** — every number on the page is derived from those, so no two figures can disagree |
| `METRICS` | The ten outbound metrics, each with a kind (`count` / `money` / `rate`) and a direction |
| `GR_CATS` | The seven guardrail categories driving the master/detail |
| `WEEKS`, `BENCH` | The 12-week axis and the benchmark rows |
| `CAMPAIGNS`, `VARIANTS`, `RULES`, `WIZ`, `WIZ_CHECKS` | Campaigns list and the wizard |
| `SIM_SHOPPERS` | The shoppers you can run a live test against |
| `PREV_*` | Word pools for `genPreview(n)`, the pre-launch campaign preview |

## The reporting engine

`DIMS[dim].members[i]` holds a weekly base, a growth rate and per-member rates. `prim(m,i,w)`
expands that into six primitives for one week — reached, replies, clicks, orders, optouts, gmv.
Everything else is derived:

- `metricAt(key,m,i,w)` — one member, one week
- `metricTotal(key,m,i)` — one member over the period
- `fromTotals(key,totals)` — a metric from summed primitives

**Rates are always re-derived from period totals, never averaged across weeks.** That is why the
chart, the tiles and the revenue table can never disagree — there is one source of truth and three
projections of it. There is a test for this in WORKFLOW.md.

`repSeries()` takes the top four members and folds the rest into `Other`, re-deriving Other's
values from primitives so its rates stay honest.

**Colour is bound to the member, not to its rank.** Assignment happens once at load: the four
biggest members of each dimension take the four categorical hues, everything else is `--dv-other`.
Changing metric or view can never repaint a series.

## Mutable state

Six variables, all module-scope: `aceState`, `engFilter`, `stg` (selected lifecycle stage),
`convFilter`/`convSel`/`convQ`, `wizStep`, `studio`. Every one of them is followed by a `render*`
call — there is no reactive layer, so **if you mutate state you must re-render**.

## Panels

`openPanel(title, sub, bodyHTML, footHTML)` fills the single `#panel`. `closePanel()` hides it.

The studio is a wide two-column variant: `panel.classList.add("wide")` before `openPanel`, and the
body contains `.studio` with two `.studioCol`. `openPanel` strips `wide` when the body isn't a
studio, and `closePanel` clears it after the transition — so a normal panel never inherits the
wide width.

Entry points: `studioForEng(i)` for an engagement, `studioForDomain(i)` for a domain, `openLearn(k)`
for a mode, `openShoppers(stage)` → `openShopper(s)`, `openTier(i)` (`null` for a new tier).

## The live test

`simOpener(ctx, shopper)` produces the first message. `simReply(text)` matches the merchant's typed
reply against keyword groups (sizing / price / problem / refusal / anything else) and returns
`{msg, note}` — the note is the reasoning line under the thread. `studioSay()` appends and schedules
the reply; `drawStudio()` repaints.

To add a behaviour to the simulation, add a branch to `simReply`. Keep the `note` honest about which
rule produced the answer — that note is the whole point of the feature.

## Charts

Hand-rolled inline SVG, no library. `stackedBars` and `lines` **measure `host.clientWidth`** and
generate at 1:1 so text doesn't get downscaled; `drawCharts()` re-runs on a debounced resize. Both
have hover tooltips through a shared `#tip`, and both have a table view via the `data-view`
segmented control.

**Palette.** `--dv-1 #7e55f6` (purple, ACE) · `--dv-2 #c35e4a` (coral) · `--dv-3 #149db8` (teal,
Default) · `--dv-4 #0e4ea7` (blue), plus `--dv-other #b3b8c1`. Axiom steps validated **on all
pairs**, not just adjacent ones — the earlier blue/purple pairing failed the normal-vision floor
(ΔE 11.5, under 15) and was genuinely hard to tell apart in the line chart. The set is on
`<body data-palette>`. **If you change a series colour, re-run the validator** (see WORKFLOW.md);
don't eyeball it.

**Stat tiles** match the Gorgias metric card: label, info icon and a `+`, a large tabular number,
a delta whose direction comes from a rotated arrow rather than from colour, and a full-bleed
`sparkline()` with a gradient fill. They are **read-only and account-wide** — deliberately
independent of the report below them.

Stacked bars put the 2px gap on the *top* of each segment except the topmost, so the bottom segment
stays anchored to the baseline. Only the topmost segment gets the 4px rounded end.

## Conventions

- No hex colours in new CSS — use the semantic tokens.
- Spacing only from the scale: `--xxxxs` 2 → `--xxl` 48.
- Type through the `.h-*` / `.t-*` classes; never a raw font-size.
- Chart text wears ink tokens, never the series colour. Identity comes from the legend, the swatch
  and the direct label — never colour alone.
- New interactive things use `data-*` attributes and the delegated document click handler, not
  per-node listeners, so they survive a re-render.
- Copy is plain English, no em-dash-free rule but no jargon either. It reads like a colleague
  explaining the product, because half the prototype's job is the copy.

## Ranked and table views

`hBars` is a single-measure chart, so it uses one hue — ranking is magnitude, not identity. The
table view is not a nicety: the dataviz rules require a table alternative wherever colour carries
meaning.

## The campaign preview

`genPreview(n)` builds n plausible conversations from word pools, splitting the variants the way the
wave would and marking roughly one in nine as deferred by a collision. It is index-driven, so the
same n always produces the same set. `openPreviewConv(i)` opens one as a full thread with the
instruction that produced it. Nothing about it is a simulation of the model — it exists so a
merchant can see the shape and tone of what a campaign would send before committing.

## Known dead ends

`NEW_BODY` and `TEST_BODY` are template strings for the "New engagement" panel and the page-level
Test buttons. They predate the studio and overlap with it. If you touch that area, consider folding
the New engagement flow into a studio variant rather than keeping two shapes.
