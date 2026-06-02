# Lifecycle Hooks

Distilled from Modal guide on container lifecycle hooks.

## @modal.enter()

Runs once when a new container starts, before any inputs are processed.

```python
@modal.enter()
def setup(self):
    self.model = load_model()
    self.tokenizer = load_tokenizer()
    self.cache = {}
```

### Use cases

- Loading ML models into GPU memory
- Establishing database connections
- Initializing caches or state
- Downloading configuration

### Execution timing

- Runs after the container image is ready
- Runs before any `@modal.method()` calls
- Runs once per container lifetime (not per input)

## @modal.exit()

Runs when a container is shutting down.

```python
@modal.exit()
def cleanup(self):
    self.db.close()
    save_metrics(self.collected_metrics)
```

### Triggers

- Container idle timeout (scaledown_window exceeded)
- Manual termination
- Preemption (SIGTERM)
- App redeployment (old containers shut down)

### Limitations

- Best-effort: may not complete if container is forcefully killed
- Keep cleanup logic fast
- Don't depend on exit running for correctness

## Execution Order

1. Container boots (image + global scope)
2. `@modal.enter()` runs
3. Inputs processed via `@modal.method()` calls (repeated)
4. `@modal.exit()` runs on shutdown

## Multiple Enter Methods

You can have multiple `@modal.enter()` methods. They run in declaration order:

```python
@modal.enter()
def load_model(self):
    self.model = load_model()

@modal.enter()
def connect_db(self):
    self.db = connect_database()
```

## Combining with Lifecycle

```python
@app.cls(gpu="A100", volumes={"/models": volume})
class InferenceService:
    @modal.enter()
    def setup(self):
        self.model = load_model("/models/llama")
        self.request_count = 0

    @modal.method()
    def predict(self, prompt):
        self.request_count += 1
        return self.model(prompt)

    @modal.exit()
    def report(self):
        print(f"Processed {self.request_count} requests")
```

## Tips

- `@modal.enter()` is the right place for one-time expensive setup
- Prefer `@modal.enter()` over `__init__` for Modal-specific setup
- Keep `@modal.exit()` lightweight and fast
- State set in `@modal.enter()` persists across all method calls on that container
