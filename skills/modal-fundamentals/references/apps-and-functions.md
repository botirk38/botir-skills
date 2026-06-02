# Apps and Functions

Distilled from the Modal guide on Apps, Functions, and entrypoints.

## App

- `modal.App(name="my-app")` — the deployment unit
- Groups Functions for atomic deployment
- Named via constructor; re-deploying by name updates in place
- `app.app_id` property to get the running app's ID

### Ephemeral vs Deployed

| Type | Created by | Lifetime |
|------|-----------|----------|
| Ephemeral | `modal run`, `app.run()` | Until script exits (or `--detach`) |
| Deployed | `modal deploy` | Until manually stopped |

### App constructor defaults

```python
app = modal.App(
    name="my-app",
    image=modal.Image.debian_slim(),   # default image for all Functions
    secrets=[modal.Secret.from_name("shared")],  # injected into all Functions
    volumes={"/data": modal.Volume.from_name("vol")},  # mounted in all Functions
)
```

## Function

- Decorated with `@app.function()`
- Each Function scales independently
- Scales to zero by default when idle
- Can be invoked with `.remote()`, `.local()`, `.map()`, `.starmap()`, `.spawn()`

### Invocation patterns

```python
result = f.remote(arg)         # synchronous remote call
handle = f.spawn(arg)          # fire-and-forget, returns FunctionHandle
results = f.map([1, 2, 3])    # parallel map over inputs
result = f.local(arg)          # run locally (for testing)
```

## Entrypoints

### local_entrypoint

Runs on your local machine. Typically orchestrates remote calls:

```python
@app.local_entrypoint()
def main(count: int = 10):
    for result in my_func.map(range(count)):
        print(result)
```

Primitive-typed arguments become CLI flags automatically.

### Remote entrypoint

If there's exactly one `@app.function()` and no `local_entrypoint`, that function becomes the entrypoint for `modal run`.

### Specifying entrypoint explicitly

```bash
modal run script.py::app.specific_function
```

## App.lookup

Look up a deployed App by name (useful for Sandboxes):

```python
app = modal.App.lookup("my-app", create_if_missing=True)
```

## Outputs and Logging

- `modal.enable_output()` context manager for programmatic runs
- Deployed app logs visible on the Modal dashboard
- Container stdout/stderr captured automatically
