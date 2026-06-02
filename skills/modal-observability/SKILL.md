---
name: modal-observability
description: Monitoring, metrics, and integrations on Modal — Datadog, OpenTelemetry, audit logs, GPU metrics, Slack notifications, and troubleshooting. Use when setting up monitoring, debugging performance, or integrating with observability platforms.
---

# Modal Observability

Use this skill when setting up monitoring, debugging performance issues, integrating with observability platforms, or auditing workspace activity.

## When to Use This Skill

- Connecting Modal to Datadog or OpenTelemetry
- Monitoring GPU utilization, power, and temperature
- Setting up Slack notifications for failures
- Reviewing audit logs for compliance
- Debugging container crashes or OOM errors
- Understanding GPU health and metrics

## GPU Metrics

Modal exposes GPU metrics on the dashboard:

| Metric | What It Measures |
|--------|-----------------|
| GPU Utilization % | Time spent executing CUDA kernels (same as `nvidia-smi`) |
| GPU Power Utilization % | Fraction of max power draw — best proxy for actual work |
| GPU Temperature | Die temperature in Celsius |
| GPU Memory Used | Allocated GPU memory in bytes |

**Key insight**: GPU utilization % only measures kernel occupancy, not compute efficiency. Power utilization is a better proxy for actual work being done.

For profiling, use PyTorch profiler or NSight — see Modal examples for profiling guidance.

## GPU Health

Modal monitors GPUs at the hardware level:

- Detects ECC memory errors and other hardware faults
- Automatically retries work on healthy GPUs when hardware issues detected
- No configuration needed — health monitoring is built in

## Datadog Integration

Send Modal metrics and logs to Datadog:

1. Get Datadog API key and site URL
2. In Modal dashboard → Settings → Integrations → Datadog
3. Enter API key and site
4. Metrics and logs flow automatically

Modal sends function invocation metrics, container lifecycle events, and custom logs to Datadog.

## OpenTelemetry Integration

Export traces to any OpenTelemetry-compatible backend:

1. In Modal dashboard → Settings → Integrations → OpenTelemetry
2. Provide your OTLP endpoint URL and optional headers
3. Modal exports spans for function invocations, container lifecycle, etc.

Supported backends: Honeycomb, Jaeger, Datadog (via OTLP), Grafana Tempo, etc.

## Slack Notifications (Beta)

Get Slack alerts when scheduled functions fail:

1. Modal dashboard → Settings → Integrations → Slack
2. Connect your Slack workspace
3. Choose channel for notifications
4. Alerts fire on function failure, timeout, or OOM

## Audit Logs (Enterprise)

Append-only record of workspace-level actions: who did what, when, from where.

### Key fields

| Field | Description |
|-------|-------------|
| `action` | e.g., `secret.create`, `app.deploy`, `volume.delete` |
| `actor` | User or service user |
| `targets` | Affected resource(s) by ID |
| `context.ip_address` | Client IP |
| `context.source` | `web` (dashboard) or `sdk` (CLI/client) |
| `status` | success or failure |

### Filtering

Filter with `key:value` pairs in the search bar. Negate with `-`:

```
action:secret.create                    # All secret creations
-status:success                          # All non-successes
action:volume.delete -actor_type:service # Volume deletes by humans
```

### Recorded actions

Includes: `app.deploy`, `app.stop`, `app.rollback`, `secret.create`, `secret.delete`, `volume.create`, `volume.delete`, `token.create`, `token.delete`, `environment.create`, `member.set_role`, and more.

**Note**: Container runtime activity (function invocations, sandbox exec) is NOT audited — only workspace-level actions.

## Go SDK — Telemetry via gRPC Interceptors

```go
import (
    modal "github.com/modal-labs/modal-client/go"
    "google.golang.org/grpc"
)

// Custom interceptor for tracing/metrics
func loggingInterceptor(ctx context.Context, method string, req, reply any,
    cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
    start := time.Now()
    err := invoker(ctx, method, req, reply, cc, opts...)
    log.Printf("RPC %s took %v", method, time.Since(start))
    return err
}

mc, _ := modal.NewClientWithOptions(&modal.ClientParams{
    UnaryInterceptors: []grpc.UnaryClientInterceptor{loggingInterceptor},
})
```

## TypeScript SDK — Telemetry via gRPC Middleware

```typescript
import { ModalClient } from "modal";

const modal = new ModalClient({
    grpcMiddleware: [
        async (call, options, next) => {
            const start = Date.now();
            const result = await next(call, options);
            console.log(`RPC ${call.method} took ${Date.now() - start}ms`);
            return result;
        },
    ],
});
```

Both SDKs support custom gRPC interceptors/middleware for integrating with OpenTelemetry, Datadog, or custom observability pipelines.

## Troubleshooting Guide

### Common Issues

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| OOM kill | Memory limit exceeded | Increase `memory` request or set `memory=(request, limit)` |
| Container timeout | Function exceeds `timeout` | Increase timeout or optimize |
| Cold start too slow | Large image or imports | Use memory snapshots, optimize image layers |
| GPU not detected | Wrong image or missing CUDA | Use `modal.Image.debian_slim()` with proper CUDA setup |
| Import error in container | Package not in image | Add to image definition |
| Disk write error (`OSError`) | Disk quota exceeded | Request larger `ephemeral_disk` |

### Deployment overlay

Enable "Show Deployments" toggle on metric charts to overlay deployment markers on graphs — correlate metric changes with deployments.

## References

- [integrations.md](references/integrations.md) — Datadog, OpenTelemetry, Slack setup details
- [audit-logs.md](references/audit-logs.md) — Audit log actions and filtering
- [gpu-monitoring.md](references/gpu-monitoring.md) — GPU metrics and health details

## Guardrails

- GPU utilization % is not a measure of compute efficiency — use power draw instead
- Audit logs record workspace actions, not runtime activity
- Slack notifications require Beta access
- OpenTelemetry exports add minor overhead; keep sampling rate reasonable
