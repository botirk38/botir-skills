# Sandbox Networking

Distilled from Modal guides on Sandbox networking and internet access.

## Internet Access

Sandboxes have internet access by default. You can disable it:

```python
sb = modal.Sandbox.create(
    app=app,
    block_network=True,  # no outbound internet
)
```

## Tunnels (Exposing Ports)

Expose a port from the Sandbox to the internet:

```python
sb = modal.Sandbox.create(app=app)

# Start a web server inside the sandbox
p = sb.exec("python", "-m", "http.server", "8080")

# Create a tunnel to expose port 8080
tunnel = sb.tunnels()[8080]
print(f"URL: {tunnel.url}")
```

### Configuring tunnels at creation

```python
from modal import Sandbox

sb = Sandbox.create(
    app=app,
    tunnels={8080: modal.Tunnel()},
)
```

## Proxy Support

Route all Sandbox traffic through a static IP proxy:

```python
sb = modal.Sandbox.create(
    app=app,
    proxy=modal.Proxy.from_name("my-proxy"),
)
```

All outbound traffic uses the proxy's static IP.

## Region Selection

Run Sandboxes in specific regions:

```python
sb = modal.Sandbox.create(
    app=app,
    region=["us-west"],
)
```

Useful for latency-sensitive operations near external databases.

## DNS and Networking

- Sandboxes have full DNS resolution
- Can reach any public endpoint by default
- Use `block_network=True` for complete isolation
- Containers within the same App cannot directly communicate (use Volumes or queues)

## Tips

- Use tunnels to expose development servers or debugging interfaces
- Use `block_network=True` for untrusted code that shouldn't access the internet
- Combine with Proxy for consistent outbound IP addresses
