# Modal Integrations — Reference

## Datadog

### Setup

1. Navigate to Modal dashboard → Settings → Integrations → Datadog
2. Enter your Datadog API key and site (e.g., `datadoghq.com`, `datadoghq.eu`)
3. Save — metrics and logs start flowing immediately

### What gets sent

- Function invocation counts and durations
- Container lifecycle events (start, stop, OOM, timeout)
- Custom logs from your Functions
- GPU utilization metrics

## OpenTelemetry

### Setup

1. Navigate to Modal dashboard → Settings → Integrations → OpenTelemetry
2. Enter your OTLP endpoint URL (e.g., `https://api.honeycomb.io`)
3. Add optional headers (e.g., `x-honeycomb-team: YOUR_API_KEY`)
4. Save

### Compatible backends

- Honeycomb
- Jaeger
- Grafana Tempo
- Datadog (via OTLP ingestion)
- Any OTLP-compatible collector

### Exported data

Modal exports OpenTelemetry spans for:
- Function invocations (including cold start vs warm)
- Container lifecycle (startup, shutdown, preemption)
- Image builds

## Slack Notifications (Beta)

### Setup

1. Modal dashboard → Settings → Integrations → Slack
2. Click "Connect to Slack" and authorize
3. Select notification channel

### What triggers notifications

- Scheduled function failures
- Function timeouts
- Container OOM kills

## SSO Integrations

### Okta SSO

For Okta SSO integration:
1. Create SAML app in Okta admin console
2. Set ACS URL and Entity ID from Modal settings
3. Upload IdP metadata XML to Modal
4. Enable SSO enforcement in workspace settings

### Custom SAML SSO

Modal supports any SAML 2.0 identity provider:
1. Configure your IdP with Modal's SP metadata
2. Upload IdP metadata to Modal dashboard
3. Map SAML attributes (email, name)
4. Enable enforcement

SSO is available on Enterprise plans.
