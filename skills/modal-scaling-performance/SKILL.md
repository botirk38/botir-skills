---
name: modal-scaling-performance
description: Autoscaling, cold start optimization, concurrent inputs, and dynamic batching on Modal. Use when tuning container scaling behavior, reducing latency, increasing throughput, or optimizing cost.
---

# Modal Scaling and Performance

Use this skill when tuning autoscaler behavior, reducing cold start latency, configuring concurrency, or enabling dynamic batching.

## When to Use This Skill

- Cold starts are too slow for your use case
- Optimizing cost vs latency trade-offs
- Configuring container scaling limits
- Enabling concurrent input processing per container
- Using dynamic batching for inference workloads
- Understanding Modal's autoscaler behavior

## Autoscaler Basics

Every Function has an autoscaling container pool. The autoscaler:
- Spins up containers when inputs queue
- Spins down idle containers
- Scales to zero by default when no inputs

## Key Scaling Parameters

```python
@app.function(
    max_containers=100,       # upper limit on containers
    min_containers=1,         # keep warm even when idle
    buffer_containers=2,      # extra containers while active
    scaledown_window=300,     # max idle time before shutdown (seconds, 2-1200)
)
def f():
    ...
```

| Parameter | Default | Effect |
|-----------|---------|--------|
| `max_containers` | None (unlimited) | Cap concurrent containers |
| `min_containers` | 0 | Keep N containers warm always |
| `buffer_containers` | 0 | Extra warm containers while active |
| `scaledown_window` | 60 | Max seconds a container can idle |

## Cold Start Optimization

Two sources of latency during cold starts:
1. **Queue time** — waiting for a container to become ready
2. **Initialization time** — first-invocation setup work

### Reduce queue time

- `min_containers=1` — never scale to zero
- `buffer_containers=2` — keep spare capacity
- `scaledown_window=300` — keep containers warm longer

### Reduce initialization time

- Move model downloads to build time (Volume or Image)
- Use `@modal.enter()` for one-time setup per container
- Minimize global scope imports
- Use lightweight base images

## Concurrent Inputs

Process multiple inputs in a single container simultaneously:

```python
@app.function()
@modal.concurrent(max_inputs=10)
def process(x):
    ...
```

Useful for I/O-bound workloads where one container can handle multiple requests.

### With classes

```python
@app.cls()
class Model:
    @modal.enter()
    def setup(self):
        self.model = load_model()

    @modal.method()
    @modal.concurrent(max_inputs=100, target_inputs=50)
    def predict(self, x):
        return self.model(x)
```

`target_inputs` controls when the autoscaler starts new containers (default: 50% of max).

## Dynamic Batching

Automatically batch individual requests for GPU-efficient processing:

```python
@app.cls(gpu="A100")
class Model:
    @modal.enter()
    def setup(self):
        self.model = load_model()

    @modal.method()
    @modal.batched(max_batch_size=64, wait_ms=100)
    def predict(self, inputs: list[str]) -> list[str]:
        return self.model.batch_predict(inputs)
```

Individual callers send one input; Modal collects them into batches.

## Symptom Triage

### "First request is slow, subsequent ones are fast"
- Cold start problem; use `min_containers` or `scaledown_window`
- Move model loading to `@modal.enter()` or build-time

### "Throughput is low for I/O-bound work"
- Enable `@modal.concurrent` to handle multiple inputs per container
- Increase `max_inputs` for I/O-heavy workloads

### "GPU underutilized"
- Use `@modal.batched` to accumulate inputs into efficient batches
- Increase `max_batch_size` for better GPU utilization

## Reference Map

- `references/autoscaling.md` — autoscaler behavior, parameters, dynamic updates
- `references/cold-start.md` — cold start sources and optimization techniques
- `references/concurrent-inputs.md` — concurrent input configuration
- `references/dynamic-batching.md` — batching for inference workloads

## Guardrails

- `min_containers` > 0 means you pay even when idle
- Higher `scaledown_window` keeps containers warm longer but costs more
- `@modal.concurrent` is for I/O-bound work; CPU/GPU-bound work won't benefit
- `@modal.batched` function must accept and return lists
- Tune `target_inputs` and `buffer_containers` based on traffic patterns
