# PRFAQ Stage

Produces one real document: a PRFAQ (Press Release + FAQ) for a proposed API. This is
the pre-engineering artifact - it exists to force a clear answer to "should we build
this" before anyone writes a design doc, a spec, or a line of code. If a design doc or
spec already exists earlier in this conversation, treat this as happening *before* those
in the lifecycle (skip straight to the design-doc stage if a spec or design already
exists and the decision to build clearly wasn't in question).

## Step 1: Get the essentials, not a full interview

You need four things. Extract from context first; ask only what's missing, in one pass,
not one question at a time:

- **The problem** - what pain does the target customer have today, without this API?
- **The customer** - who exactly (internal team, partner type, or public developer
  segment)? Vague answers here ("developers") are a signal to push back gently - ask
  what kind of developer, at what kind of company, doing what.
- **Why now** - what changed, or what's the cost of waiting? (Competitive pressure,
  a partner asking for it, an internal API that's clearly wanted externally, etc.)
- **Rough scope** - 2-4 sentences on what the API would let someone do. Doesn't need to
  be endpoint-level yet - that's what the spec stage is for, later.

If the user genuinely doesn't know one of these yet (e.g., "why now" isn't clear), don't
force an answer - write the FAQ section honestly as an open question the team needs to
resolve, rather than inventing a justification. A PRFAQ that surfaces "we don't actually
know why now" is doing its job.

## Step 2: Write the Press Release (~half a page)

Written as if the API already launched and succeeded. Customer-facing language, no
internal jargon, no engineering terms. This is the section that forces clarity - if you
can't write this in plain language, the value proposition isn't clear yet, and that's
worth saying to the user rather than papering over with vague language.

Structure:
1. **Headline** - one sentence, what the API does and for whom
2. **Sub-headline** - the concrete benefit, in the customer's terms
3. **Opening paragraph** - the problem, and the API as the solution, dated as if launched
4. **Quote from a (illustrative, clearly-labeled-as-hypothetical) customer** - what changed
   for them. Label it plainly as illustrative (e.g., "A team integrating this API might
   say:") - never present a fabricated quote as if it were real, attributed testimony.
5. **How to get started** - one sentence pointing at the developer portal/docs (even if
   they don't exist yet - this is aspirational, showing the intended experience)

## Step 3: Write the FAQ

Cover these categories, adapting specific questions to what's actually relevant - skip
categories that don't apply rather than padding:

**Customer & problem**
- Who is this for, specifically?
- What do they do today without this API? Why is that painful?
- How do we know this problem is real (data, requests, competitive signal)?

**Why this, why now**
- What's the cost of not building this?
- What's changed recently that makes this the right time?

**Scope**
- What's explicitly in scope for v1?
- What's explicitly out of scope (and why - defer, never, or someone else's job)?

**Success criteria**
- How will we know this worked, 3 and 12 months after launch? Pull directly from the
  infrastructure/product/business metrics framework if the metrics stage has already run
  earlier in this conversation, or propose 2-3 candidate metrics per layer if not - keep
  it to the ones that matter for a v1 decision, not the full list.

**Risks & open questions**
- What could make this fail or underperform?
- What are we still not sure about? (Explicitly list unresolved questions rather than
  hiding them - a PRFAQ with no open questions usually means they weren't looked for.)

**Anticipated engineering questions**
- Rough auth/scale/dependency questions an engineering reviewer would ask, so the team
  isn't caught flat-footed in the review meeting. Keep this section honest about what's
  *not* yet known - "TBD, needs engineering input" is a valid and better answer than a
  guessed one.

## Step 4: Save, present, and flag gaps

- Save as `<api-name>-prfaq.md`.
- Use `present_files`.
- After presenting, call out in 2-3 bullets which sections were written from real input
  versus which are placeholders/open questions the user still needs to resolve before
  this goes to a review meeting - a PRFAQ with unflagged guesses is worse than one that's
  honest about its gaps.

## Notes

- This is a decision-forcing document, not a marketing document. Resist the urge to make
  everything sound good - if scope is unclear or the "why now" is weak, the FAQ should
  say so plainly rather than smoothing it over.
- Offer, but don't push: after presenting, mention that the design-doc stage
  (`references/design-doc.md`) is the natural next step once the proposal is approved -
  don't generate a design doc or spec unprompted here.
