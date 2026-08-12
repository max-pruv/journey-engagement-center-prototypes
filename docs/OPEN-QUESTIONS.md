# Open questions

Things the prototype decided one way but that are genuinely open, plus known gaps. Ordered roughly
by how much they'd change if answered differently.

## Product

**Which tab lands first in Engagements?** Currently **AI engagement**, on the reasoning that if you
land on Custom you never discover the domains or the mode education. For daily operation the Custom
tab is the one you'd want. One-line change in `selectTab`'s default.

**Should a play be individually switchable after all?** The prototype says no, hard: the dial is the
domain, and a play with its own toggle is the scenario library coming back through the side door. But
a merchant who wants *Cart abandonment* and not *Welcome* now has to decline the whole
Consideration & browse domain. The honest answers are either "that's the trade, bound it at the
domain" or "domains are too coarse and there should be more of them". Adding per-play toggles is the
answer that feels helpful and quietly undoes the restructure — see PRODUCT.md §3b before reaching for
it.

**Are eight domains the right granularity now that the plays are visible?** Consideration & browse
carries four quite different plays; Loyalty moments carries three near-identical ones. Splitting or
merging domains is cheap in the data and is probably the real answer to the question above.

**Is "What we're willing to give" right as a dropdown?** The tone field became a prefilled textarea;
this one stayed a closed list, because it's the field the guardrails have to be able to bound. If it
should also be free text, the guardrail ceiling has to become the thing that's enforced rather than
the field.

**How is the modelled potential for an off domain actually computed?** The prototype shows estimates
for domains that are off and labels them as modelled on comparable merchants. That number needs a
real method, and a confidence, before it goes in front of a merchant — an estimate that's wrong in
the optimistic direction is worse than dashes.

**Do ACE modes need to be per-domain?** Right now the mode is global and domains are on/off. A
merchant might reasonably want Automatic on replenishment and Manual acceptance on service recovery.
That's a small data change (`mode` on each domain) and a real conversation about whether it's too
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
  "New engagement" appends a real row. A play's instruction and a domain's do not persist, and
  neither does anything in Guardrails, Lifecycle or Loyalty.
- **`TEST_BODY`** is the last panel that predates the studio, now reached only from the Lifecycle
  stage tester. `NEW_BODY` was folded into `studioForNew()`; this one could go the same way.

## Data-model questions the UI is quietly asserting

These are places where the prototype had to invent a shape to be clickable, and the real answer
belongs to engineering:

- An instruction is versioned (`v4 · edited 3 days ago`) and resettable to default. That implies a
  default that's owned centrally and can change under a merchant who hasn't customized.
- A domain carries an optional instruction. That makes the domain a first-class configurable object,
  not just a filter on the agent's remit.
- Exclusions are evaluated as *audience* membership, separately from suppression as *timing*. Those
  are two different joins and probably two different services.
- A play carries its own instruction *and* sits under a domain that carries one. The UI says the play
  instruction is the specific one and the domain instruction covers everything underneath, but it
  never shows them resolved together — which is deliberate (PRODUCT.md §4 killed the "resolved
  instruction set" preview), and also means the precedence is asserted rather than demonstrated.
- A play is identified by its position (`domainIndex.playIndex` in `data-playinstr`). Real plays need
  stable ids, because a merchant's customized instruction has to survive Gorgias reordering or
  inserting plays in a domain.
- Reporting's `source` dimension now reads ACE / Custom / Campaign, with ACE carrying the volume the
  standard engagements used to report under `Default`. That is the right model — those sends *are*
  ACE now — but it means historical reporting spans a definition change, which someone has to decide
  how to present.
