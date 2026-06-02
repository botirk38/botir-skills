# Proxies

Distilled from Modal guide on Proxies.

## What Are Proxies

Modal Proxies provide static outbound IP addresses for containers. All traffic between containers and the Proxy is encrypted with WireGuard.

Built on [vprox](https://github.com/modal-labs/vprox), an open-source Modal project.

## Creating Proxies

1. Go to workspace Settings → Proxies
2. Create a new Proxy (Team/Enterprise plans)
3. Each Proxy has up to 5 static IPs
4. Team: 1 Proxy max; Enterprise: 3 Proxies max

## Using with Functions

```python
@app.function(proxy=modal.Proxy.from_name("my-proxy"))
def call_api():
    import subprocess
    # Always uses the proxy's static IP
    subprocess.run(["curl", "-s", "ifconfig.me"])
```

## Using with Sandboxes

```python
sb = modal.Sandbox.create(
    app=app,
    proxy=modal.Proxy.from_name("my-proxy"),
)
```

## Performance

WireGuard adds encryption overhead and latency. For throughput issues, add more IP addresses to the Proxy.

## Multiple IPs

Up to 5 IPs per Proxy. Modal randomly picks one per Function invocation. More IPs = better throughput.

Add IPs: Settings → Proxies → select Proxy → Add IP.

## Unique per Workspace

Proxies and their IPs are not shared between workspaces.

## Tips

- Allow-list all Proxy IPs in your firewall
- Add more IPs if throughput is insufficient
- Proxies are workspace-unique — no IP sharing
- Contact support@modal.com for higher limits
