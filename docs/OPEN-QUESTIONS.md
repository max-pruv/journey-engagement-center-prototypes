# Open questions

Things the prototype decided one way but that are genuinely open, plus known gaps. Ordered roughly
by how much they'd change if answered differently.

## Product

**Which tab lands first in Engagements?** Currently **AI engagement**, on the reasoning that if you
land on Custom you never discover the themes or the mode education. For daily operation the Custom
tab is the one you'd want. One-line change in `selectTab`'s default.

**Now that a play is individually switchable, is eight themes still the right granularity?** A play's
switch (§3b of PRODUCT.md) answers the narrower question — a merchant who wants *Cart abandonment*
and not *Welcome* no longer has to decline the whole Consideration & browse theme. What it doesn't
answer: Consideration & browse still carries four quite different plays, and Loyalty moments carries
three near-identical ones. Splitting or merging themes is cheap in the data and is a separate
question from whether each play can be turned off.

**Is "What we're willing to give" right as a dropdown?** The tone field became a prefilled textarea;
this one stayed a closed list, because it's the field the guardrails have to be able to bound. If it
should also be free text, the guardrail ceiling has to become the thing that's enforced rather than
the field.

**How is the modelled potential for an off theme (or an off play) actually computed?** The prototype
shows estimates for themes and plays that are off and labels them as modelled on comparable
merchants. That number needs a real method, and a confidence, before it goes in front of a merchant —
an estimate that's wrong in the optimistic direction is worse than dashes.

**Do ACE modes need to be per-theme?** Right now the mode is global and themes are on/off. A
merchant might reasonably want Automatic on replenishment and Manual acceptance on service recovery.
That's a small data change (`mode` on each theme) and a real conversation about whether it's too
much control.

**Where does the 48-hour expiry on pending approvals come from?** It's asserted in the Manual
acceptance copy. It's a sensible default, not a decision anyone made.

**Campaigns vs Engagements boundary.** The prototype says: recurring belongs in Engagements, one-offs
in Campaigns. The strategy docs say Campaigns stays as-is on its own surface and is not being
rebuilt. Those two statements are compatible today but will collide the moment a campaign wants a
trigger.

## Not built

- **Brand profile** is a nav placeholder. Performance was removed — Reporting covers it.
- **Reporting** has no date-range behaviour; the selector is decorative. The dimensions are
  independent partitions with their own bases, so totals differ between them; in the real product
  they would all partition the same account.
- **Onboarding** (`onboarding.html`) is scripted: the chips prefill an answer and any typed text is
  accepted, but nothing is parsed. It argues for a shape, not a flow.
- **Rule builder** rows render but don't compose or affect the audience estimate.
- **The campaign wizard** does not parse what the merchant types — chips prefill a canned answer
  and free text is accepted but not read. Same honesty gap as onboarding.html.
- **Shopper profile** is a stand-in panel. In the product it opens in a new tab; the prototype says
  so rather than faking it.
- **Reject** in Conversations does not mutate state yet. Approve & send now moves the message to Live and updates the pending counts.
- **Dark mode.** Tokens exist for it in Axiom; the prototype only implements light.
- **Save is decorative everywhere except the custom-engagement studio.** That one does write back:
  editing a trigger or instruction and pressing Save updates `ENG[i]` and re-renders the table, and
  "New engagement" appends a real row. A theme's and a play's on/off switches do persist for the
  session, the same as any other toggle in the prototype — it's specifically the *instruction text*
  in a theme's or play's studio that doesn't write back, and neither does anything in Guardrails,
  Lifecycle or Loyalty.
- **`TEST_BODY`** is the last panel that predates the studio, now reached only from the Lifecycle
  stage tester. `NEW_BODY` was folded into `studioForNew()`; this one could go the same way.

## Data-model questions the UI is quietly asserting

These are places where the prototype had to invent a shape to be clickable, and the real answer
belongs to engineering:

- An instruction is versioned (`v4 · edited 3 days ago`) and resettable to default. That implies a
  default that's owned centrally and can change under a merchant who hasn't customized.
- A theme carries an optional instruction. That makes the theme a first-class configurable object,
  not just a filter on the agent's remit.
- Exclusions are evaluated as *audience* membership, separately from suppression as *timing*. Those
  are two different joins and probably two different services.
- A play carries its own instruction *and* sits under a theme that carries one. The UI says the play
  instruction is the specific one and the theme instruction covers everything underneath, but it
  never shows them resolved together — which is deliberate (PRODUCT.md §4 killed the "resolved
  instruction set" preview), and also means the precedence is asserted rather than demonstrated. The
  same question now applies one level down: a play's own switch and its theme's switch are ANDed
  together (`playLive`) rather than shown as a single resolved on/off, on the same "instructions ask,
  guardrails decide"-style reasoning — worth revisiting if a merchant is ever confused about why a
  play they turned on still isn't running.
- A play is identified by its position (`themeIndex.playIndex` in `data-playinstr` and `data-play`).
  Real plays need stable ids, because a merchant's customized instruction or on/off state has to
  survive Gorgias reordering or inserting plays in a theme.
- Reporting's `source` dimension now reads ACE / Custom / Campaign, with ACE carrying the volume the
  standard engagements used to report under `Default`. That is the right model — those sends *are*
  ACE now — but it means historical reporting spans a definition change, which someone has to decide
  how to present.
