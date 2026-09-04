# Metrics Stage

Turns "what should we measure for this API" into files that get deployed or implemented,
not a spreadsheet nobody opens again. Two outputs, two different owners:

1. **Infrastructure monitors** (Datadog JSON or Prometheus) → goes to whoever owns the
   observability stack, often platform/SRE.
2. **Product analytics tracking plan** (JSON) → goes to the engineer implementing
   `.track()` calls in the API/SDK code.

This stage assumes the API's shape (resources, endpoints, audience) is already known -
either from earlier in the conversation (e.g., the spec or design-doc stage's output) or
from a quick description the user gives now. Don't re-derive it if it's already
available in context.

## Step 1: Confirm scope, don't over-ask

You need three things. Get them from context first; only ask for what's missing:

- **API name and key endpoints/resources** (for labeling monitors and events)
- **Audience/maturity** (internal / partner / public) - affects alert thresholds and
  urgency. Public APIs generally need tighter latency/error thresholds since customers
  notice immediately.
- **Which observability tool** the team actually uses for infra metrics. This varies
  by team, so always ask rather than assume:
  - Datadog → generate plain Datadog Monitor JSON (see Step 2). This works for *any*
    team - paste into Datadog's UI ("New Monitor" → import) or POST it to the Monitors
    API. No IaC tooling required.
  - Prometheus/Grafana → generate PrometheusRule YAML (this one is inherently
    config-as-code, no plain-JSON equivalent worth generating)
  - New Relic or other → generate the Datadog JSON version as the closest common pattern
    and say explicitly that field names need adapting to their tool, rather than guessing
    wrong syntax
  - Unsure → default to Datadog Monitor JSON, and say so

Product analytics tool (Amplitude/Mixpanel) - if not stated, ask once; the two have
compatible-enough event/property models that the same tracking plan JSON works for either
with minor renaming.

## Step 2: Infrastructure monitors

Cover five metrics by default, unless the user has different priorities: **uptime**,
**p90/p99 latency**, **error rate**, **requests per minute**, and **rate-limit-429 rate**.
These are the ones that, per the API metrics framework, form the baseline "is this even
usable" layer underneath product and business metrics. 429 rate is its own signal,
separate from general error rate: a spike can mean an integration bug on the customer's
side, a bot/abuse pattern, or - if it's a *sustained* elevated rate from one API key -
a customer who's outgrowing their tier and is a upsell/tier-upgrade conversation, not
an incident. Route these two cases differently in the alert (see below).

### Datadog - Monitor JSON (default)

This is the primary format - works for anyone with Datadog access, regardless of how
their org manages infrastructure. Each object below can be imported individually via
the Datadog UI or POSTed to `https://api.datadoghq.com/api/v1/monitor`:

```json
[
  {
    "name": "<api-name> - Error rate above threshold",
    "type": "metric alert",
    "query": "sum(last_5m):sum:trace.http.request.errors{service:<api-name>}.as_count() / sum:trace.http.request.hits{service:<api-name>}.as_count() > 0.02",
    "message": "Error rate for <api-name> exceeded 2% over 5 minutes. @slack-<team-channel>",
    "tags": ["service:<api-name>", "layer:infrastructure"],
    "options": { "thresholds": { "critical": 0.02, "warning": 0.01 } }
  },
  {
    "name": "<api-name> - p99 latency above threshold",
    "type": "metric alert",
    "query": "percentile(last_5m):p99:trace.http.request.duration{service:<api-name>} > 1000",
    "message": "p99 latency for <api-name> exceeded 1000ms. @slack-<team-channel>",
    "tags": ["service:<api-name>", "layer:infrastructure"],
    "options": { "thresholds": { "critical": 1000, "warning": 750 } }
  },
  {
    "name": "<api-name> - Availability drop",
    "type": "metric alert",
    "query": "avg(last_5m):avg:synthetics.test_run{check:<api-name>-uptime} < 0.99",
    "message": "<api-name> availability dropped below 99% over 5 minutes. @pagerduty-<team>",
    "tags": ["service:<api-name>", "layer:infrastructure"],
    "options": { "thresholds": { "critical": 0.99 } }
  },
  {
    "name": "<api-name> - Sustained spike in 429 responses",
    "type": "metric alert",
    "query": "sum(last_15m):sum:trace.http.request.hits{service:<api-name>,http.status_code:429}.as_count() / sum:trace.http.request.hits{service:<api-name>}.as_count() > 0.05",
    "message": "429 rate for <api-name> exceeded 5% over 15 minutes. Check whether this is one or two API keys (a customer to reach out to about upgrading their tier) or broad-based (likely abuse or limits set too low). @slack-<team-channel>",
    "tags": ["service:<api-name>", "layer:infrastructure"],
    "options": { "thresholds": { "critical": 0.05, "warning": 0.02 } }
  },
  {
    "name": "<api-name> - Abnormal request volume",
    "type": "query alert",
    "query": "avg(last_15m):anomalies(avg:trace.http.request.hits{service:<api-name>}, 'basic', 2, direction='both') >= 1",
    "message": "Request volume for <api-name> is anomalous - could mean an outage, a bad deploy, or an integration partner misbehaving. @slack-<team-channel>",
    "tags": ["service:<api-name>", "layer:infrastructure"]
  }
]
```

Adjust thresholds based on audience: tighten (e.g., error rate critical → 0.01, latency →
500ms) for public APIs; loosen for internal-only APIs with lower stakes.


### Prometheus (PrometheusRule)

Use this instead when the user's team runs Prometheus/Grafana:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: <api-name>-alerts
  labels:
    layer: infrastructure
spec:
  groups:
    - name: <api-name>.rules
      rules:
        - alert: <ApiName>HighErrorRate
          expr: sum(rate(http_requests_total{service="<api-name>",status=~"5.."}[5m])) / sum(rate(http_requests_total{service="<api-name>"}[5m])) > 0.02
          for: 5m
          labels: { severity: critical }
          annotations:
            summary: "Error rate above 2% for <api-name>"
        - alert: <ApiName>HighLatency
          expr: histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket{service="<api-name>"}[5m])) by (le)) > 1
          for: 5m
          labels: { severity: warning }
          annotations:
            summary: "p99 latency above 1s for <api-name>"
        - alert: <ApiName>LowAvailability
          expr: avg_over_time(up{service="<api-name>"}[5m]) < 0.99
          for: 5m
          labels: { severity: critical }
          annotations:
            summary: "Availability below 99% for <api-name>"
```

## Step 3: Product analytics tracking plan

Generate a tracking plan JSON that maps directly to the customer journey funnel
(discovery → activation → integration → retention). Focus on the events that are cheap
to instrument and high-signal - don't propose tracking everything.

```json
{
  "api_name": "<api-name>",
  "events": [
    {
      "name": "api_key_created",
      "trigger": "User generates their first (or a new) API key in the developer portal",
      "properties": {
        "user_id": "string",
        "org_id": "string",
        "environment": "string (sandbox | production)"
      },
      "funnel_stage": "activation"
    },
    {
      "name": "first_api_call",
      "trigger": "First successful authenticated request from a given api_key, ever",
      "properties": {
        "api_key_id": "string",
        "endpoint": "string",
        "time_since_key_created_seconds": "number"
      },
      "funnel_stage": "activation",
      "notes": "time_since_key_created_seconds is your Time to First Hello World (TTFHW) metric directly."
    },
    {
      "name": "endpoint_called",
      "trigger": "Every API request (consider sampling at scale)",
      "properties": {
        "api_key_id": "string",
        "endpoint": "string",
        "method": "string",
        "status_code": "number",
        "response_time_ms": "number"
      },
      "funnel_stage": "engagement",
      "notes": "High volume - sample or aggregate before sending if using a paid-per-event tool."
    },
    {
      "name": "rate_limit_hit",
      "trigger": "A request from a given api_key is rejected with 429",
      "properties": {
        "api_key_id": "string",
        "endpoint": "string",
        "requests_in_window": "number",
        "limit": "number"
      },
      "funnel_stage": "engagement",
      "notes": "Sample rather than send per-request if volume is high. A key that hits this repeatedly over several days is a strong upsell/tier-upgrade signal, worth a saved segment in Amplitude/Mixpanel rather than just an alert."
    },
    {
      "name": "api_key_revoked",
      "trigger": "User deletes/revokes an API key",
      "properties": {
        "api_key_id": "string",
        "days_active": "number"
      },
      "funnel_stage": "churn_signal"
    }
  ],
  "derived_metrics": {
    "activation_rate": "count(distinct api_key_id with >=1 first_api_call) / count(distinct api_key_created)",
    "time_to_first_value": "avg(first_api_call.time_since_key_created_seconds)",
    "dau_wau_mau": "distinct api_key_id in endpoint_called, grouped by day/week/month",
    "retention": "cohort of api_key_created, % with endpoint_called in week N",
    "keys_repeatedly_rate_limited": "count(distinct api_key_id with rate_limit_hit on >=3 distinct days in trailing 7 days) - candidate list for a tier-upgrade outreach"
  }
}
```

Adapt event names/properties to the resources named in Step 1 (e.g., if the API is about
orders, `endpoint_called` might be split into `order_created`, `order_cancelled` if the
user wants resource-level granularity instead of generic endpoint tracking - ask if unclear
which grain they want).

## Step 4: Business metrics - point, don't fabricate

Do not generate config for MRR, churn, or CAC - these come from billing/subscription
systems (Stripe, an internal billing service) or CRM, not from event instrumentation.
Instead, add a short note in your reply naming where each one typically lives, so the
user knows who to talk to rather than expecting a file:

- Revenue/MRR → billing system (Stripe, Chargebee, internal)
- CAC → marketing/finance, tied to acquisition channel data
- Churn → derived from billing (cancellations) or from the `api_key_revoked` /
  inactivity signal above, cross-referenced with account status

## Step 5: Save, present, summarize

- Save infra config as `<api-name>-monitors.json` (Datadog default) or
  `<api-name>-alerts.yaml` (Prometheus).
- Save tracking plan as `<api-name>-tracking-plan.json`.
- Present both via `present_files`.
- Summarize in 3-4 bullets: which thresholds you set and why (tie to audience/maturity),
  and flag anything that's a placeholder the user must fill in (Slack channel names,
  PagerDuty routing, etc.) - don't bury placeholders silently in the file.
