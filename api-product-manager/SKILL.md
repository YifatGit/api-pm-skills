---
name: api-product-manager
description: Helps an API product manager take a public, partner, or internal API through its full lifecycle - deciding whether to build it, designing it, specifying it, and monitoring it - producing a real artifact at each stage instead of generic advice. Covers four artifact types depending on what's asked - (1) a PRFAQ/proposal doc when pitching or deciding whether to build an API ("should we build an API for X", "draft a PRFAQ", "pitch this API idea", "is this worth building"), (2) a technical design doc once the decision is made ("write a design doc for this API", "how should we architect this", "what are the tradeoffs here", "what needs security or governance sign-off"), (3) an OpenAPI 3.0 spec turning a plain-language description into endpoints/schema ("build an API for X", "spec out this API", "draft an OpenAPI file", "what endpoints would I need for Y"), and (4) deployable monitoring config (Datadog Monitor JSON or PrometheusRule YAML) plus a product-analytics tracking plan ("set up monitoring for this API", "what should I track", "build a tracking plan", "alert on API errors or latency"). Use this any time a product manager is working on a public, partner, or internal API anywhere in this lifecycle, even if they don't name the artifact type or say "API" explicitly - describing an API idea, asking what to measure, or asking for endpoints is enough to trigger it.
---

# API Product Manager

One skill covering an API's whole lifecycle, for product managers who need real
artifacts - not a spreadsheet or a bullet list nobody opens again. The lifecycle runs:

**PRFAQ (should we build it) → Design doc (how will it work) → Spec (the contract) →
Metrics kit (how do we know it's working)**

Each stage is a self-contained reference file below. Figure out which stage the user
needs, read that one file, and follow it. Don't load the others unless the conversation
actually moves into that stage.

## Step 0: Pick the stage

| The user is... | Read |
|---|---|
| Pitching an idea, deciding whether to build an API, asking "is this worth it" | `references/prfaq.md` |
| Past the "should we" question, needs architecture/tradeoffs/security review | `references/design-doc.md` |
| Describing resources/endpoints and wanting a spec, schema, or OpenAPI file | `references/spec.md` |
| Instrumenting, monitoring, or tracking an API already built or being built | `references/metrics.md` |

If it's genuinely unclear which stage fits, ask in one short question rather than
guessing - the four artifacts are different enough that picking wrong wastes a full
turn. If the user wants the whole thing end-to-end ("take this API idea from scratch to
launch"), walk the stages in lifecycle order, producing one artifact at a time, and
carry context forward between stages rather than re-interviewing at each one.

## Principles that apply at every stage

These hold regardless of which reference file you're following - they're what makes the
output usable by the people who receive it, not just impressive-looking:

- **Pull from context first, ask only what's missing, in one pass.** If an earlier stage
  already produced a PRFAQ, design doc, or spec in this conversation, reuse its problem
  statement, resources, audience, and decisions - don't re-run the interview.
- **Never hide gaps with a silent guess.** Where a reference file says to default
  something, say so explicitly in your reply. Where it says to leave something as an open
  question, actually leave it open rather than inventing an answer - a document that's
  honest about what's unresolved is more useful than one that looks complete but isn't.
- **Save the artifact as a real file, present it, then summarize in a few bullets** what
  you assumed or defaulted, so the user can spot anything that doesn't match their actual
  constraints without reading the whole file.
- **Point at the next stage, don't jump to it unprompted.** After finishing one artifact,
  mention what naturally comes next in the lifecycle and offer to do it - let the user
  decide when to move forward.
- **If the user entered mid-lifecycle, name the stage(s) they skipped too, not just the
  next one forward.** E.g. if they asked for a spec directly, the PRFAQ and design doc
  were skipped - say so and offer them explicitly, rather than only pointing ahead to
  metrics. Otherwise a user who never asked to skip a stage may not realize it's missing
  until they go looking for it.

## Reference files

- **`references/prfaq.md`** - PRFAQ (Press Release + FAQ) proposal document: the
  Amazon-style "should we build this" artifact, before any design or spec exists.
- **`references/design-doc.md`** - Technical design doc: architecture and resource model,
  design alternatives considered (auth, pagination, protocol), non-functional
  requirements, versioning strategy, a security & governance review checklist, and a
  rollout plan.
- **`references/spec.md`** - Turns a plain-language API description into a complete
  OpenAPI 3.0 YAML spec. See `references/conventions.md` for the reasoning behind its
  default conventions (pagination style, error schema, etc.), useful if the user or an
  engineering reviewer asks "why."
- **`references/metrics.md`** - Deployable infrastructure monitors (Datadog Monitor JSON
  or PrometheusRule YAML) and a product-analytics tracking plan (JSON, for
  Amplitude/Mixpanel), covering activation and retention events.
