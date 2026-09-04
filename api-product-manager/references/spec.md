# Spec Stage

A stage for product managers (not just engineers) who need to turn a plain-language
description of an API into a real OpenAPI 3.0 spec file - something engineering can
actually review, not just a bullet list.

The person using this often knows the *product* side (who the API is for, what it needs
to let customers do) but not necessarily REST/OpenAPI conventions. Your job is to do the
translation, ask only what's truly ambiguous, and default sensibly on the rest - while
explaining the defaults so the PM can push back on anything that doesn't fit.

## Step 1: Capture the API description

If the user already described the API in the conversation (including in an earlier PRFAQ
or design doc from this skill), extract what you can from it before asking anything. Look
for:

- **Purpose** - what problem does this API solve, for whom?
- **Resources** - the "nouns" of the API (e.g., orders, users, shipments). These become
  the main paths.
- **Actions** - what can be done to each resource (read, list, create, update, delete,
  or a custom action like "cancel" or "retry")?
- **Audience** - internal, partner, or public? This affects auth strength, rate limiting
  defaults, and how much the spec needs to "stand on its own" for a stranger reading it.

## Step 2: Ask only what's missing

Don't run a long questionnaire if the description already answers most of this. Ask about
what's genuinely unclear, briefly, and default the rest with a note. Common gaps:

- **Fields per resource** - if not given, propose a reasonable minimal set (id, created_at,
  updated_at, plus the 2-4 fields the user's description implies) and flag them as
  suggestions to confirm, not facts.
- **Auth** - default to API key via header (`X-API-Key`) for partner/public APIs unless
  the user specifies OAuth2, bearer tokens, or "internal only, no auth."
- **Pagination** - default to cursor-based pagination (`limit` + `cursor` query params)
  for any `list` endpoint, unless the user says otherwise.
- **Versioning** - default to a path-based version prefix (`/v1/...`) unless the user
  already has a convention.
- **Rate limiting** - default to per-API-key limits, tiered by maturity/audience:
  internal APIs can go unlimited or very high; public APIs need an explicit default
  (propose 60 requests/minute unless the user has a number) with room for higher tiers
  later. Every endpoint should document a `429` response and the rate-limit headers
  (see Step 4) - don't leave rate limiting as prose-only, it needs to be in the spec
  itself so both the client and the gateway config can reference the same numbers.
- **Errors** - default to a consistent error schema across all endpoints (see Step 4).

If the user answers "not sure" or gives no preference, proceed with the default and say
so explicitly in your reply - don't silently assume.

## Step 3: Apply API-as-product conventions

These aren't arbitrary - they reflect standard practice for APIs meant to be consumed by
someone other than the team that built them (see `references/conventions.md` for the
reasoning behind each one, useful if the user asks "why"):

- Collection names are plural nouns (`/orders`, not `/order`).
- Every list endpoint supports pagination and returns a consistent envelope
  (`{ "data": [...], "next_cursor": "..." }`).
- Every error response uses one shared schema (`{ "error": { "code", "message" } }`),
  referenced via `components/schemas/Error` rather than repeated per endpoint.
- Every endpoint has a `summary`, and non-trivial ones get a `description` - this is what
  turns into the "API reference" a developer actually reads.
- Rate limiting and auth are documented in `components/securitySchemes` and referenced
  per-path, not just described in prose.
- Every response includes rate-limit headers as `components/headers`, referenced from
  every `200`/`201` response: `X-RateLimit-Limit`, `X-RateLimit-Remaining`,
  `X-RateLimit-Reset`. Every endpoint also declares a `429` response using the shared
  `Error` schema, so a client can handle rate limiting generically instead of per-endpoint.

## Step 4: Generate the OpenAPI YAML

Produce a complete, valid OpenAPI 3.0.3 file:

```yaml
openapi: 3.0.3
info:
  title: <API name>
  version: 1.0.0
  description: <one or two sentences on purpose and audience>
servers:
  - url: https://api.example.com/v1
paths:
  /orders:
    get:
      summary: List orders
      parameters:
        - name: limit
          in: query
          schema: { type: integer, default: 20 }
        - name: cursor
          in: query
          schema: { type: string }
      responses:
        '200':
          description: A page of orders
          headers:
            X-RateLimit-Limit: { $ref: '#/components/headers/RateLimitLimit' }
            X-RateLimit-Remaining: { $ref: '#/components/headers/RateLimitRemaining' }
            X-RateLimit-Reset: { $ref: '#/components/headers/RateLimitReset' }
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items: { $ref: '#/components/schemas/Order' }
                  next_cursor: { type: string, nullable: true }
        '429':
          description: Rate limit exceeded
          headers:
            X-RateLimit-Reset: { $ref: '#/components/headers/RateLimitReset' }
          content:
            application/json:
              schema: { $ref: '#/components/schemas/Error' }
    post:
      summary: Create an order
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/OrderInput' }
      responses:
        '201':
          description: Order created
          content:
            application/json:
              schema: { $ref: '#/components/schemas/Order' }
        '400':
          description: Invalid request
          content:
            application/json:
              schema: { $ref: '#/components/schemas/Error' }
components:
  securitySchemes:
    ApiKeyAuth:
      type: apiKey
      in: header
      name: X-API-Key
  headers:
    RateLimitLimit:
      description: Requests allowed per window for this API key
      schema: { type: integer }
    RateLimitRemaining:
      description: Requests remaining in the current window
      schema: { type: integer }
    RateLimitReset:
      description: Unix timestamp when the current window resets
      schema: { type: integer }
  schemas:
    Order:
      type: object
      properties:
        id: { type: string }
        created_at: { type: string, format: date-time }
        # ...resource-specific fields
    OrderInput:
      type: object
      properties:
        # ...fields required to create the resource
    Error:
      type: object
      properties:
        error:
          type: object
          properties:
            code: { type: string }
            message: { type: string }
security:
  - ApiKeyAuth: []
```

Adapt resource names, fields, and endpoints to what the user actually described - the
above is the shape, not the content. Cover every resource and action from Step 1. Include
`GET /{resource}`, `GET /{resource}/{id}`, `POST /{resource}`, `PATCH /{resource}/{id}`,
and `DELETE /{resource}/{id}` by default for each resource, and add custom action
endpoints (e.g. `POST /orders/{id}/cancel`) for anything that isn't plain CRUD.

## Step 5: Save, present, and summarize

- Save the file as `<api-name>.yaml` (kebab-case) under `/mnt/user-data/outputs/`.
- Use `present_files` so the user can open/download it.
- After presenting, give a short plain-language summary (3-5 bullets) of the choices you
  made by default (auth method, pagination style, versioning) so the PM can flag anything
  that doesn't match their actual constraints - don't make them read the YAML to find out.
- Do not restate the whole spec in prose - the file is the artifact.

## Notes

- This produces a **first-draft skeleton** meant to accelerate a conversation with
  engineering, not a final contract. Say this explicitly the first time you present a spec
  in a conversation.
- If the user's description implies more than ~6-8 resources, check in before generating
  everything at once - confirm the resource list first, then generate.
- Once the spec is settled, the metrics stage (`references/metrics.md`) is the natural
  next step - offer it, don't jump to it unprompted.
