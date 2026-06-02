# Port Forwarding

Distilled from Modal API reference on modal.forward.

## modal.forward

Context manager that exposes a container port to the public internet:

```python
@app.function()
def serve():
    with modal.forward(8080) as tunnel:
        print(f"URL: {tunnel.url}")
        # Start server on port 8080
        run_server(port=8080)
```

### Tunnel object

The `tunnel` object provides:
- `tunnel.url` — public HTTPS URL
- `tunnel.host` — hostname
- `tunnel.port` — mapped port

## Use Cases

- Temporary debugging endpoints
- Webhook receivers for development
- Exposing internal services during testing
- Ad-hoc file serving

## With Sandboxes

```python
sb = modal.Sandbox.create(
    app=app,
    tunnels={8080: modal.Tunnel()},
)

# After creating the sandbox
tunnel = sb.tunnels()[8080]
print(f"URL: {tunnel.url}")
```

## Lifetime

The tunnel is active only while:
- The `modal.forward` context manager is open (for Functions)
- The Sandbox is running (for Sandbox tunnels)

## Tips

- Use for development and debugging, not production traffic
- For production HTTP endpoints, use `@modal.fastapi_endpoint` or `@modal.asgi_app`
- Tunnels provide HTTPS URLs automatically
- Port forwarding works for any protocol over TCP
