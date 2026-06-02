# Parametrization

Distilled from Modal guide on parametrized functions.

## What Is Parametrization

Parametrize a class so each unique combination of parameters gets its own autoscaling container pool.

## Defining Parameters

```python
@app.cls()
class Worker:
    user_id: str = modal.parameter()
    model_size: str = modal.parameter(default="small")
    batch_size: int = modal.parameter(default=32)

    @modal.enter()
    def setup(self):
        self.model = load_model(self.model_size)

    @modal.method()
    def process(self, data):
        return self.model(data)
```

`modal.parameter()` creates keyword-only constructor arguments.

## Creating Instances

```python
# Each unique combo = separate container pool
w1 = Worker(user_id="alice", model_size="large")
w2 = Worker(user_id="bob")   # model_size defaults to "small"

# These route to different container pools
w1.process.remote(data)
w2.process.remote(data)

# Same parameters = same pool
w3 = Worker(user_id="alice", model_size="large")  # reuses w1's pool
```

## Constraints

- Total parameter size limited to 16 KiB
- Parameters must be serializable
- Type annotations are required
- Parameters are keyword-only in the constructor

## with_options (Dynamic Override)

Override class configuration at invocation time:

```python
# Reference a deployed class
Model = modal.Cls.from_name("my-app", "Model")

# Override GPU and scaling
FastModel = Model.with_options(gpu="H100", max_containers=20)
FastModel().predict.remote("hello")
```

### Stackable overrides

```python
Model.with_options(gpu="H100").with_options(scaledown_window=300)
```

### with_concurrency

```python
Model.with_options(gpu="A100").with_concurrency(max_inputs=100)
```

### with_batching

```python
Model.with_options(gpu="A100").with_batching(max_batch_size=64, wait_ms=100)
```

## Use Cases

- **Per-user isolation**: separate container pools per user
- **A/B testing**: different model versions in separate pools
- **Multi-tenancy**: tenant-specific configurations
- **Dynamic GPU selection**: override GPU type based on request

## Tips

- Parameters determine the container pool — same params = same pool
- Use defaults for common configurations
- Combine with `@modal.enter()` to load parameter-specific resources
- `with_options` creates new pools, not modifying existing ones
