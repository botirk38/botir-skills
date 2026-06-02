---
name: modal-fundamentals
description: Core primitives for building and running Modal applications. Use when creating Apps, defining Functions, writing entrypoints, deploying, or managing environments and CLI workflows.
---

# Modal Fundamentals

Use this skill when building, running, or deploying Modal applications from scratch or when you need to understand the core execution model.

## When to Use This Skill

- Creating a new Modal App or Function
- Running ephemeral apps with `modal run` or `modal serve`
- Deploying persistent apps with `modal deploy`
- Writing `local_entrypoint` or remote entrypoints
- Managing Modal environments (dev/staging/prod)
- Understanding the App/Function/container relationship
- Working with the Modal CLI or Python client
- Using async Modal APIs
- Structuring Modal projects

## Core Concepts

### App

`modal.App` groups one or more Functions for atomic deployment.

```python
import modal

app = modal.App(name="my-app")

@app.function()
def f():
    print("Hello world!")
```

### Function

Each `Function` is an independent autoscaling unit. Zero containers run (zero cost) when idle.

### Entrypoints

- `@app.local_entrypoint()` — runs locally, typically calls `.remote()` on Functions
- A single `@app.function()` can be the entrypoint if no `local_entrypoint` exists
- `modal run script.py::app.func_name` to specify a particular function

Argument parsing is automatic for primitive types:

```python
@app.local_entrypoint()
def main(foo: int, bar: str):
    some_function.remote(foo, bar)
```

## Execution Modes

| Command | Behavior |
|---------|----------|
| `modal run script.py` | Ephemeral app, stops when script exits |
| `modal run --detach script.py` | Ephemeral but keeps running after client disconnects |
| `modal serve script.py` | Ephemeral with live-reload for web endpoint development |
| `modal deploy script.py` | Persistent deployed app |

### Programmatic execution

```python
with modal.enable_output():
    with app.run():
        some_function.remote()
```

## Deployment

```python
app.deploy(name="my-app", strategy="rolling")
```

Strategies: `rolling` (default, gradual traffic shift) or `recreate` (terminate all, then start new).

### Environments

Environments isolate resources (Secrets, Volumes, deployed Apps):

```bash
modal environment create staging
modal run --env staging script.py
modal deploy --env production script.py
```

Set default: `modal config set-environment staging`

## Async API

All Modal methods have async counterparts via `.aio` suffix:

```python
# Sync
result = f.remote(arg)

# Async
result = await f.remote.aio(arg)

# Async volumes
await vol.read_file.aio("/path")
```

Async functions run concurrent inputs as asyncio tasks (not threads). Sync functions use threading.

## Global Variables

Container-scope globals persist across invocations. Use `modal.is_local()` for conditional init:

```python
if modal.is_local():
    weights = None  # don't download locally
else:
    weights = download_weights()
```

## Project Structure

Recommended layout:
```
my-modal-project/
├── my_module/
│   ├── __init__.py
│   ├── app.py       # modal.App() and function definitions
│   └── utils.py
├── pyproject.toml
└── .modal/
    └── environment   # default environment name
```

Deploy with: `modal deploy -m my_module.app`

Key rules:
- `include_source=True` (default) adds files reachable from entrypoint to container
- Use `modal.Mount` to add non-importable files explicitly
- Set `MODAL_ENVIRONMENT` env var or `.modal/environment` file for default environment

## Go SDK

The Go SDK can call deployed Functions, create Sandboxes, and manage resources — but cannot define Functions.

```go
import modal "github.com/modal-labs/modal-client/go"

// Create client
mc, _ := modal.NewClient()

// Reference an App
app, _ := mc.Apps.FromName(ctx, "my-app", &modal.AppFromNameParams{CreateIfMissing: true})

// Call a deployed Function
fn, _ := mc.Functions.FromName(ctx, "my-app", "my_function", nil)
result, _ := fn.Remote(ctx, []any{"arg1"}, nil)
```

Install: `go get -u github.com/modal-labs/modal-client/go` (requires Go 1.23+)

## TypeScript SDK

The TypeScript SDK mirrors Go capabilities — call Functions, create Sandboxes, manage resources.

```typescript
import { ModalClient } from "modal";

const modal = new ModalClient();

// Reference an App
const app = await modal.apps.fromName("my-app", { createIfMissing: true });

// Call a deployed Function
const fn = await modal.functions.fromName("my-app", "my_function");
const result = await fn.remote(["arg1"]);
```

Install: `npm install modal` (requires Node.js 22+)

### SDK Feature Parity

| Feature | Python | Go | TypeScript |
|---------|--------|-----|-----------|
| Define Functions | Yes | No | No |
| Call Functions | Yes | Yes | Yes |
| Sandboxes | Yes | Yes | Yes |
| Volumes | Yes | Yes | Yes |
| Queues | Yes | Yes | Yes |
| Secrets | Yes | Yes | Yes |
| Images | Yes | Yes | Yes |
| Scheduling | Yes | No | No |
| Web endpoints | Yes | No | No |

## Developing with LLMs

When using AI assistants to write Modal code:
- Install `modal` locally for type checking
- Provide Modal Python SDK stubs for autocomplete
- Use `modal run` for rapid iteration (not full deployments)
- Use `modal serve` for web endpoint development with live reload

## Symptom Triage

### "modal run does nothing"
- Ensure you have a `@app.local_entrypoint()` or exactly one `@app.function()`
- Check `modal setup` was completed

### "App deploys but Functions don't run"
- Scheduled Functions need `modal deploy`, not `modal run`
- Non-scheduled Functions need explicit invocation (web URL, `.remote()`, etc.)

### "Logs are missing"
- Use `modal.enable_output()` context manager for programmatic runs
- Check the Modal dashboard for deployed app logs

### "Import error: module not found in container"
- Ensure `include_source=True` (default) and file is importable from entrypoint
- Or add explicit `modal.Mount.from_local_dir()` in image definition

## Reference Map

- `references/apps-and-functions.md` — App/Function lifecycle, naming, entrypoints
- `references/cli-commands.md` — CLI reference for run, serve, deploy, app, environment
- `references/environments.md` — Environment isolation and configuration
- `references/local-development.md` — Local dev workflows, serve, hot reload

## Guardrails

- Everything is code — no YAML configuration files
- `modal.Stub` is removed since Modal 1.0; use `modal.App`
- Protect `app.run()` in `if __name__ == "__main__"` blocks to avoid running in containers
- Use `--detach` for long-running ephemeral jobs where the client may disconnect
- Use `modal.is_local()` to guard local-only code paths
- Use `.aio` suffix for async operations, not raw `await` on sync methods
