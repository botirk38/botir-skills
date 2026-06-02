---
name: modal-classes-lifecycle
description: Class-based Modal Functions with lifecycle hooks and parametrization. Use when loading models once per container, sharing state between method calls, parametrizing Functions by user/config, or managing container startup/shutdown.
---

# Modal Classes and Lifecycle

Use this skill when using class-based Functions for stateful workloads, lifecycle management, or parametrization.

## When to Use This Skill

- Loading an ML model once per container (not per request)
- Sharing state between method calls
- Running setup/teardown logic per container
- Creating separate container pools per parameter set (e.g., per user)
- Combining GPU inference with lifecycle hooks

## Core Pattern: Class with Lifecycle

```python
import modal

app = modal.App()

@app.cls(gpu="A100")
class Model:
    @modal.enter()
    def setup(self):
        self.model = load_model()

    @modal.method()
    def predict(self, prompt: str) -> str:
        return self.model(prompt)

    @modal.exit()
    def cleanup(self):
        del self.model
```

### Calling

```python
@app.local_entrypoint()
def main():
    model = Model()
    result = model.predict.remote("hello")
```

## Lifecycle Hooks

### @modal.enter()

Runs once when a container starts. Use for expensive initialization:

```python
@modal.enter()
def setup(self):
    self.model = load_model("/models/llama")
    self.tokenizer = load_tokenizer()
```

### @modal.exit()

Runs when a container shuts down. Use for cleanup:

```python
@modal.exit()
def teardown(self):
    save_metrics(self.metrics)
```

`@modal.exit()` also runs on SIGTERM (preemption).

### @modal.method()

Exposes a class method as an invokable Function:

```python
@modal.method()
def predict(self, x):
    return self.model(x)
```

## Parametrization

Create separate container pools per parameter combination:

```python
@app.cls()
class Worker:
    user_id: str = modal.parameter()
    model_size: str = modal.parameter(default="small")

    @modal.enter()
    def setup(self):
        self.model = load_model(self.model_size)

    @modal.method()
    def process(self, data):
        return self.model(data)
```

### Calling parametrized classes

```python
w1 = Worker(user_id="alice", model_size="large")
w1.process.remote(data)  # runs in alice/large pool

w2 = Worker(user_id="bob")
w2.process.remote(data)  # runs in bob/small pool (separate containers)
```

Each unique parameter combination gets its own autoscaling pool.

## with_options (Runtime Override)

Override class configuration dynamically:

```python
Model = modal.Cls.from_name("my-app", "Model")
FastModel = Model.with_options(gpu="H100", max_containers=10)
FastModel().predict.remote("hello")
```

Stackable: `Model.with_options(gpu="H100").with_options(scaledown_window=300)`

## Symptom Triage

### "Model loads on every request"
- Use `@modal.enter()` to load once per container
- Move model loading from `__init__` or method body to `@modal.enter()`

### "Container shutdown is abrupt"
- Use `@modal.exit()` for graceful cleanup
- `@modal.exit()` receives SIGTERM on preemption

### "Each user shares a container pool"
- Use `modal.parameter()` to create per-user pools
- Each unique parameter combination scales independently

## Reference Map

- `references/classes-and-methods.md` — @app.cls, @modal.method, invocation patterns
- `references/lifecycle-hooks.md` — @modal.enter, @modal.exit, execution order
- `references/parametrization.md` — modal.parameter, per-user pools, with_options

## Guardrails

- `@modal.enter()` runs once per container, not once per input
- `@modal.exit()` is best-effort on container shutdown
- Parameters are limited to 16 KiB total
- Use `@modal.method()` (not `@app.function()`) for class methods
- `@app.cls()` replaces `@app.function()` for the class decorator
