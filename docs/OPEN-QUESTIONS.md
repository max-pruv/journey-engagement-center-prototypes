# Open questions

Things the prototype decided one way but that are genuinely open, plus known gaps. Ordered roughly
by how much they'd change if answered differently.

## Product

**Which tab lands first in Engagements?** Currently **AI engagement**, on the reasoning that if you
land on Custom you never discover the domains or the mode education. For daily operation the Custom
tab is the one you'd want. One-line change in `selectTab`'s default.

**Does "Custom engagement" as a tab name collide with "Custom" as a Built-by value?** It does,
slightly: a tab called Custom engagement contains rows built by `Default`. The alternative is
renaming the badge values to `Gorgias` / `You`, which reads better but abandons the vocabulary in the
strategy docs. Left as-is deliberately; it's two string replacements either way.

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

- **Performance** and **Brand profile** are nav placeholders.
- **Overview** has no date-range behaviour; the selector is decorative.
- **Rule builder** rows render but don't compose or affect the audience estimate.
- **Shopper profile** is a stand-in panel. In the product it opens in a new tab; the prototype says
  so rather than faking it.
- **Approve / Reject** in Conversations don't mutate state.
- **Dark mode.** Tokens exist for it in Axiom; the prototype only implements light.
- The `NEW_BODY` "New engagement" flow and the studio are two shapes for overlapping jobs. Folding
  the former into a studio variant would remove a concept.

## Data-model questions the UI is quietly asserting

These are places where the prototype had to invent a shape to be clickable, and the real answer
belongs to engineering:

- An instruction is versioned (`v4 · edited 3 days ago`) and resettable to default. That implies a
  default that's owned centrally and can change under a merchant who hasn't customized.
- A domain carries an optional instruction. That makes the domain a first-class configurable object,
  not just a filter on the agent's remit.
- Exclusions are evaluated as *audience* membership, separately from suppression as *timing*. Those
  are two different joins and probably two different services.
- The `Built by` distinction (`Default` / `Custom`) survives after a merchant edits a pre-built
  engagement's instruction — you can be `Default`-built with a `Customized` instruction. That's the
  right model but it's two independent flags, not one.
