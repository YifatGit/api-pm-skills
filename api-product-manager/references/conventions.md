# Why these conventions

Short reference for when the user (or engineering) asks "why did you do it this way?"

- **Plural collection names** - matches the way nearly every public API reference
  documents resources; makes the spec instantly familiar to any developer reading it,
  which shortens onboarding time.
- **Cursor pagination over offset** - offset pagination breaks when records are added or
  removed between pages (skipped or duplicated results). Cursor pagination is stable and
  is the default for most modern public APIs.
- **One shared Error schema** - a developer integrating your API should only need to write
  one error-handling code path, not one per endpoint. Inconsistent error shapes are one of
  the most common developer complaints in API feedback.
- **API key via header by default** - simplest auth model to start with for partner/public
  APIs; upgrade to OAuth2 later if third parties need delegated access (e.g. "log in with
  your account" flows) rather than a shared secret.
- **Path-based versioning (`/v1/...`)** - most visible versioning scheme; a breaking change
  is obvious from the URL alone, which matters when planning deprecation and migration
  windows down the line.
- **`summary` and `description` on every path** - these fields are what auto-generated API
  reference tools (Swagger UI, Redoc, Postman) pull directly into the developer-facing
  documentation. Skipping them means shipping an API reference with blank pages.
