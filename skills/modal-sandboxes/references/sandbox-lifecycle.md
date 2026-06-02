# Sandbox Lifecycle

Distilled from Modal guide on Sandboxes.

## Lifecycle Events

1. **Created** — Sandbox registered, no resources allocated
2. **Scheduled** — Worker provisioning (CPU, memory, GPU, volumes)
3. **Started** — Container running, `exec()` available
4. **Ready** — Readiness probe passed (if configured)
5. **Completed** — Entrypoint exited successfully
6. **Terminated** — Manually terminated via `sb.terminate()`
7. **Error** — Entrypoint failed or resource error

## Timeouts

```python
sb = modal.Sandbox.create(app=app, timeout=600)  # 10 min max lifetime
```

The Sandbox is terminated after the timeout regardless of activity.

Per-command timeouts:

```python
p = sb.exec("long-running-command", timeout=30)
```

## Reconnecting to Sandboxes

Look up an existing Sandbox by ID:

```python
sb = modal.Sandbox.from_id(sandbox_id)
```

Useful for long-running Sandboxes where the client may disconnect.

## Entrypoint vs exec()

### No entrypoint (default)

Sandbox starts with no process running. Use `exec()` to run commands:

```python
sb = modal.Sandbox.create(app=app)
p = sb.exec("python", "script.py")
```

### With entrypoint

```python
sb = modal.Sandbox.create("python", "server.py", app=app)
# The entrypoint starts immediately
# Use exec() for additional commands
```

## Readiness Probes

Wait for a service to be ready before proceeding:

```python
sb = modal.Sandbox.create(
    "python", "server.py",
    app=app,
    readiness_probe=modal.Sandbox.ReadinessProbe(port=8080, path="/health"),
)
# sb is in "Ready" state once /health returns 200
```

## Cleanup Patterns

### Explicit cleanup

```python
sb = modal.Sandbox.create(app=app)
try:
    p = sb.exec("make", "test")
    p.wait()
finally:
    sb.terminate()
    sb.detach()
```

### detach()

Releases the client's reference to the Sandbox. The Sandbox continues running until timeout or termination.

```python
sb.detach()  # Sandbox keeps running
```

## Tips

- Always set timeouts to prevent runaway Sandboxes
- Use `from_id()` to reconnect after client disconnects
- Use readiness probes for services that take time to start
- Call both `terminate()` and `detach()` for full cleanup
