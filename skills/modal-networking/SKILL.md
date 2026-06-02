---
name: modal-networking
description: Networking on Modal — proxies for static IPs, port forwarding, region selection, and container networking. Use when configuring outbound IP addresses, exposing ports, selecting regions for latency, or connecting containers.
---

# Modal Networking

Use this skill when configuring network behavior for Modal Functions, including static IPs, port forwarding, region selection, and inter-container communication.

## When to Use This Skill

- Need a static outbound IP for allow-listing
- Exposing internal container ports to the internet
- Running Functions near specific cloud regions
- Optimizing network latency
- Connecting Functions to external private networks

## Proxies (Static IP)

Route all container traffic through static IP addresses:

```python
@app.function(proxy=modal.Proxy.from_name("my-proxy"))
def call_external_api():
    import requests
    # All traffic exits through proxy's static IP
    return requests.get("https://api.example.com").json()
```

### Setup

1. Create a Proxy in workspace Settings (Team/Enterprise plans)
2. Each Proxy has up to 5 static IPs
3. Allow-list these IPs in your firewall

### With Sandboxes

```python
sb = modal.Sandbox.create(
    app=app,
    proxy=modal.Proxy.from_name("my-proxy"),
)
```

## Port Forwarding (modal.forward)

Expose a container port to the internet temporarily:

```python
@app.function()
def serve():
    with modal.forward(8080) as tunnel:
        print(f"Public URL: {tunnel.url}")
        start_server(port=8080)
        # Server accessible at tunnel.url while context is active
```

Useful for debugging, temporary services, or webhook receivers.

## Region Selection

### Container region

Run containers in specific geographic regions:

```python
@app.function(region=["us-west"])
def west_coast_function():
    ...
```

### Available regions

| Broad | Narrow |
|-------|--------|
| `us` | `us-east`, `us-central`, `us-south`, `us-west` |
| `eu` | `eu-west`, `eu-north`, `eu-south` |
| `ap` | `ap-northeast`, `ap-southeast`, `ap-south`, `ap-melbourne`, `jp`, `au` |
| `uk` | — |
| `ca` | — |

Broader regions are cheaper (1.5x) than narrow (1.75x).

### Routing region

Control where inputs are routed through:

```python
@app.function(routing_region="eu-west")
def eu_function():
    ...
```

Options: `us-east` (default), `us-west`, `eu-west`, `ap-south`.

## Container Networking

- Containers in the same App cannot directly connect to each other by IP
- Use Volumes, Dicts, or Queues for inter-Function communication
- External network access is available by default
- DNS resolution works normally

## Symptom Triage

### "External API rejects requests"
- Use a Proxy for static IP allow-listing
- Verify the proxy is online in workspace Settings

### "High latency to external database"
- Use `region=` to run near the database
- Use `routing_region=` to route through a closer region

### "Need to expose a development server"
- Use `modal.forward(port)` context manager
- Use Sandbox tunnels for Sandbox-based servers

## Reference Map

- `references/proxies.md` — Proxy setup, static IPs, WireGuard
- `references/port-forwarding.md` — modal.forward, tunnels
- `references/region-selection.md` — container and routing regions, pricing
- `references/container-networking.md` — network behavior, DNS, inter-container comms

## Guardrails

- Proxies require Team or Enterprise plan
- Region selection adds cost multipliers (1.5x–1.75x)
- `routing_region` cannot be changed after initial deployment
- Use broader regions for better availability and lower cost
- Proxy traffic is encrypted with WireGuard
