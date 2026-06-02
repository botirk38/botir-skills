---
name: modal-scaling-performance
description: Autoscaling, cold start optimization, memory snapshots, concurrent inputs, dynamic batching, and high-performance LLM inference on Modal. Use when tuning scaling, reducing latency, increasing throughput, or optimizing cost.
---

# Modal Scaling and Performance

Use this skill when tuning autoscaler behavior, reducing cold start latency, configuring concurrency, enabling dynamic batching, or using memory snapshots for faster starts.

## When to Use This Skill

- Cold starts are too slow for your use case
- Optimizing cost vs latency trade-offs
- Configuring container scaling limits
- Enabling concurrent input processing per container
- Using dynamic batching for inference workloads
- Using memory snapshots for sub-second cold starts
- High-performance LLM inference optimization
- Dynamic function configuration at runtime
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

## Memory Snapshots

Skip initialization work on most container boots — 3-10x faster cold starts:

```python
@app.cls(enable_memory_snapshot=True)
class Model:
    @modal.enter(snap=True)
    def load(self):
        self.model = load_model()  # captured in snapshot

    @modal.enter()
    def post_snapshot(self):
        ...  # runs after snapshot restore (not captured)
```

Enable with `enable_memory_snapshot=True` on deployed apps (`modal deploy`).

### GPU Memory Snapshots (Alpha)

Also capture GPU state for even faster GPU cold starts:

```python
@app.cls(
    gpu="h100",
    enable_memory_snapshot=True,
    experimental_options={"enable_gpu_snapshot": True},
)
class Llm:
    @modal.enter(snap=True)
    def init(self):
        self.pipeline = pipeline(model="Qwen/Qwen3-1.7B", device_map="cuda")
        # Warmup recommended — run sample inference to capture compiled state
        self.pipeline([{"role": "user", "content": "warmup"}])
```

### Caveats

- Memory snapshots are worker-type-specific (6 snapshots for CPU, 2-3 for specific GPU)
- Randomness is frozen in snapshots — re-seed after restore if needed
- GPU snapshots have driver compatibility limits; review docs before using

## Go SDK — Autoscaler Control

```go
mc, _ := modal.NewClient()
fn, _ := mc.Functions.FromName(ctx, "my-app", "my_function", nil)

// Update autoscaler at runtime
minC := uint32(2)
maxC := uint32(50)
bufC := uint32(5)
fn.UpdateAutoscaler(ctx, &modal.FunctionUpdateAutoscalerParams{
    MinContainers:    &minC,
    MaxContainers:    &maxC,
    BufferContainers: &bufC,
})
```

## TypeScript SDK — Autoscaler Control

```typescript
import { ModalClient } from "modal";
const modal = new ModalClient();

const fn = await modal.functions.fromName("my-app", "my_function");

// Update autoscaler at runtime
await fn.updateAutoscaler({
    minContainers: 2,
    maxContainers: 50,
    bufferContainers: 5,
    scaledownWindow: 300,
});
```

Note: Defining autoscaler config (decorators, concurrency, batching, memory snapshots) is Python-only. Go/TS SDKs can update runtime autoscaler parameters for deployed Functions.

## Dynamic Function Configuration

Change compute resources per call at runtime:

```python
# Change GPU for a specific invocation
result = f.with_options(gpu="H100").remote(data)

# Dynamic concurrency
concurrent_f = f.with_concurrency(max_inputs=32)

# Compose options
concurrent_f.with_options(gpu="H100").remote(...)
```

Each distinct configuration gets its own autoscaling container pool. Avoid too many fine-grained configs.

## High-Performance LLM Inference

### Throughput optimization (batch jobs)

- Use vLLM for best throughput (continuous batching)
- FP8 quantization on Hopper+ GPUs (not FP4)
- Single-GPU-per-replica for simplicity when model fits
- Use `.spawn_map` + `--detach` for fire-and-forget batch jobs
- Store results in Volumes or external databases

### Latency optimization (chatbots)

- Minimize TTFT (time-to-first-token) and TPOT (time-per-output-token)
- Use streaming responses to improve perceived latency
- Speculative decoding for faster generation
- Smaller quantized models for memory-bound decode
- `min_containers` to avoid cold start on first request

### Cold start optimization (bursty traffic)

- Memory snapshots (`enable_memory_snapshot=True`) for fast container starts
- GPU memory snapshots for GPU-heavy initialization
- Model weights on Volumes (not downloaded at boot)
- `min_containers` + `buffer_containers` for warm capacity

## Symptom Triage

### "First request is slow, subsequent ones are fast"
- Cold start problem; use `min_containers`, `scaledown_window`, or memory snapshots
- Move model loading to `@modal.enter()` or build-time

### "Throughput is low for I/O-bound work"
- Enable `@modal.concurrent` to handle multiple inputs per container
- Increase `max_inputs` for I/O-heavy workloads

### "GPU underutilized"
- Use `@modal.batched` to accumulate inputs into efficient batches
- Increase `max_batch_size` for better GPU utilization

### "Cold start still slow after optimization"
- Enable memory snapshots (`enable_memory_snapshot=True`)
- For GPU: also enable GPU memory snapshots
- Pre-warm with sample inference in `@modal.enter(snap=True)`

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
- Memory snapshots only work with `modal deploy` (not `modal run`)
- GPU memory snapshots are Alpha — test compatibility before production use
- `with_options()` creates separate container pools — don't create too many variants
- Tune `target_inputs` and `buffer_containers` based on traffic patterns
