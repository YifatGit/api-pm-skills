# Design Doc Stage

Produces one document: a technical design doc for an API that's already been decided on
(or is far enough along that "should we build this" isn't the open question anymore). This
sits between the PRFAQ and the spec in the lifecycle.

A design doc's job is different from a spec: a spec picks defaults and moves on (see
`references/spec.md`), but a design doc has to *show its work* - name the alternatives that
were considered and why one was picked, so an engineering or security reviewer can push
back on the reasoning, not just the output. If a PRFAQ or spec already exists earlier in
this conversation, pull the problem, customer, resources, and endpoints from there rather
than re-asking - don't restart the interview from scratch.

## Step 1: Confirm what's already decided vs. still open

Pull from context first. You need:

- **The problem and customer** (from the PRFAQ if one exists, or ask in one pass if not)
- **Resources and actions** (from a spec if one exists, or a plain description if not -
  doesn't need to be endpoint-complete yet, that's what happens in Step 2/3 of this doc
  and gets finalized in the spec stage afterward)
- **Audience/maturity** (internal / partner / public) - this changes how much the security
  and versioning sections need to defend against a hostile or unknown caller, versus a
  trusted internal one
- **Target maturity stage for this launch** (Beta, Early Access/Limited Release, or GA) -
  ask if unclear; it changes how firm the rollout plan needs to be

If something is genuinely undecided, don't invent a decision to fill the section - write
it as an open question instead (see Step 3). A design doc that hides unresolved questions
just moves the argument to the review meeting instead of surfacing it here.

## Step 2: Write the design doc

Use this structure. Skip a section only if it plainly doesn't apply (e.g., no sync/async
choice to make) - don't pad sections that have nothing real to say.

**1. Overview & goals**
One paragraph: what this API does, for whom, and the goal it serves. Link back explicitly
to the PRFAQ's problem statement if one exists in this conversation.

**2. Requirements**
Split functional (what it must let someone do - pull from Step 1) from non-functional.
Frame non-functional requirements using the Key Success Factors so nothing important gets
skipped by accident: performance (latency/throughput targets), scalability (expected load
and growth), reliability (uptime target), security, usability (how hard is this to
integrate), flexibility (room to extend later). State a rough target for each where one
exists (e.g., "p99 < 500ms"); mark "TBD, needs input from X" where it doesn't - a guessed
number is worse than an honest gap here.

**3. Resource & data model / architecture**
The "nouns" of the API, their relationships, and a short description of the request flow
(sync vs. async, and why). This doesn't need diagrams-as-code - plain prose or a simple
list of resources and how they relate is enough for a v1 design doc.

**4. Design decisions & alternatives considered**
This is the section that distinguishes a design doc from a spec. For each significant
choice, give the decision *and* the alternative(s) considered and why they were rejected -
one or two sentences each, not an essay:
- **Auth model** (API key vs. OAuth2 vs. mTLS, etc.)
- **Pagination style** (cursor vs. offset)
- **Protocol** (REST vs. RPC vs. something else) if it's not obviously REST
- Anything else non-obvious to this specific API (e.g., webhook delivery guarantees,
  idempotency handling for write endpoints)

**5. Versioning strategy**
State the versioning scheme (default: path-based, `/v1/...`, matching the spec stage's
default - flag it if this API needs to diverge) and define what counts as a breaking
change versus a safe additive change for this specific API, so engineering and consumers
share a definition before the first breaking change shows up. Anchor the launch to a point
on the maturity ladder - **Beta → Early Access/Limited Release → GA** - and say which stage
this design targets and roughly what would need to be true to move to the next one. This
section is about the versioning *strategy* going forward, not a deprecation or sunset plan
for retiring anything - don't add a deprecation timeline unless the user explicitly asks
for one.

**6. Security & governance review**
A checklist, not prose, so a reviewer can scan it fast:
- Authn/authz model and where credentials/tokens are issued and rotated
- Scopes/permissions - what can this API do on the caller's behalf, and can it be scoped
  narrower than "all or nothing"
- Data classification - does any request/response body carry PII or other sensitive data,
  and what handling that implies
- Abuse posture - rate limiting approach and what happens on sustained abuse (tie to the
  metrics stage's 429-rate monitor if that stage's been used already)
- Sign-offs needed before this can move to Beta/GA - name the functions typically involved
  (security review, legal/compliance if PII is in scope, platform/SRE for on-call
  readiness) rather than specific people, since the user knows their own org chart

**7. Rollout plan**
Target maturity stage (from Step 1), who needs to be aligned before starting build
(stakeholders - typically eng, security, docs/DX, and support), and a pointer to the fact
that a testing strategy exists (doesn't need to be written out here in full).

**8. Open questions & risks**
List them plainly. A design doc with no open questions was probably not looked at hard
enough - if Step 1 surfaced any unresolved items, they belong here, not smoothed over.

## Step 3: Save, present, and summarize

- Save as `<api-name>-design-doc.md`.
- Use `present_files`.
- Summarize in 3-5 bullets which sections rest on real input versus which are placeholders
  or open questions the user still needs to resolve - same principle as the PRFAQ stage:
  an unflagged guess in a design doc is worse than an honest gap.
- Point at the spec stage (`references/spec.md`) as the natural next step once the design
  is reviewed - don't generate the spec unprompted here, the same way the PRFAQ stage
  defers to this one rather than jumping ahead.

## Notes

- This is a review-forcing document aimed at an engineering/security audience, not a
  pitch - resist smoothing over unresolved tradeoffs the way you would for a PRFAQ's
  press-release section.
- If the user's request is really just "give me the endpoints," they may want the spec
  stage directly instead - offer that if the ask turns out to be lighter than a full
  design doc once you hear the details.
