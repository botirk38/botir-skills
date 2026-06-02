---
name: modal-parallel-execution
description: Parallel execution patterns on Modal — map, starmap, spawn, generators, error handling, and retries. Use when running batch jobs in parallel, fan-out/fan-in patterns, fire-and-forget tasks, or streaming results from generators.
---

# Modal Parallel Execution

Use this skill when running work in parallel across containers, using fan-out patterns, or handling errors and retries in distributed execution.

## When to Use This Skill

- Running batch processing across many containers
- Fan-out / fan-in patterns
- Fire-and-forget background tasks
- Streaming results from remote generators
- Configuring retries for transient failures
- Processing large datasets in parallel
- Building job queues for async processing
- Dataset ingestion pipelines

## Execution Patterns

### .remote() — Single call

```python
result = f.remote(arg)
```

Synchronous remote call. Blocks until result returns.

### .map() — Parallel batch

```python
results = list(f.map([1, 2, 3, 4, 5]))
```

Distributes inputs across containers in parallel. Results are ordered by default.

```python
# Unordered (returns results as they complete)
results = list(f.map(inputs, order_outputs=False))
```

### .starmap() — Multi-argument parallel

```python
results = list(f.starmap([(a1, b1), (a2, b2), (a3, b3)]))
```

Each tuple is unpacked as positional arguments.

### .spawn() — Fire-and-forget

```python
handle = f.spawn(arg)
# ... do other work ...
result = handle.get()  # retrieve when needed
```

Returns a `FunctionHandle` for later retrieval. The Function runs asynchronously.

### .remote_gen() — Remote generator

```python
for chunk in f.remote_gen(arg):
    print(chunk)
```

Streams results from a generator Function.

## Error Handling and Retries

### Retries

```python
@app.function(retries=modal.Retries(max_retries=3, backoff_coefficient=2.0))
def flaky_operation():
    ...
```

Parameters:
- `max_retries` — number of retry attempts
- `backoff_coefficient` — multiplier for wait time between retries
- `initial_delay` — seconds before first retry

### Error handling in .map()

```python
results = []
for result in f.map(inputs):
    try:
        results.append(result)
    except Exception as e:
        print(f"Failed: {e}")
```

Exceptions from remote Functions are re-raised on the caller side.

## Generators

Modal Functions can be generators that yield results incrementally:

```python
@app.function()
def process_large_file(path):
    with open(path) as f:
        for line in f:
            yield transform(line)

# Consume remotely
for result in process_large_file.remote_gen("/data/big.csv"):
    print(result)
```

## Real-World Patterns

### Batch processing with map

```python
@app.function()
def process_image(image_path):
    return resize_and_upload(image_path)

@app.local_entrypoint()
def main():
    image_paths = list_all_images()
    results = list(process_image.map(image_paths))
    print(f"Processed {len(results)} images")
```

### Fan-out with spawn

```python
@app.local_entrypoint()
def main():
    handles = [expensive_task.spawn(i) for i in range(100)]
    results = [h.get() for h in handles]
```

### Pipeline chaining

```python
@app.function()
def step1(data):
    return preprocess(data)

@app.function(gpu="A100")
def step2(preprocessed):
    return model_inference(preprocessed)

@app.local_entrypoint()
def pipeline():
    preprocessed = list(step1.map(raw_data))
    results = list(step2.map(preprocessed))
```

## Go SDK — Calling Deployed Functions

```go
mc, _ := modal.NewClient()
fn, _ := mc.Functions.FromName(ctx, "my-app", "process_item", nil)

// Single remote call
result, _ := fn.Remote(ctx, []any{"arg1"}, nil)

// Parallel spawns (fan-out)
var calls []*modal.FunctionCall
for i := 0; i < 100; i++ {
    call, _ := fn.Spawn(ctx, []any{i}, nil)
    calls = append(calls, call)
}
// Collect results (fan-in)
for _, call := range calls {
    result, _ := call.Get(ctx, nil)
    fmt.Println(result)
}

// Autoscaler control
minC := uint32(2)
maxC := uint32(20)
fn.UpdateAutoscaler(ctx, &modal.FunctionUpdateAutoscalerParams{
    MinContainers: &minC,
    MaxContainers: &maxC,
})
```

## TypeScript SDK — Calling Deployed Functions

```typescript
import { ModalClient } from "modal";
const modal = new ModalClient();

const fn = await modal.functions.fromName("my-app", "process_item");

// Single remote call
const result = await fn.remote(["arg1"]);

// Parallel spawns (fan-out)
const calls = await Promise.all(
    Array.from({ length: 100 }, (_, i) => fn.spawn([i]))
);
// Collect results (fan-in)
const results = await Promise.all(calls.map(c => c.get()));

// Autoscaler control
await fn.updateAutoscaler({
    minContainers: 2,
    maxContainers: 20,
    bufferContainers: 3,
});

// Class methods
const model = await modal.cls.fromName("my-app", "Model");
const instance = await model.create({ gpu: "A100" });
const prediction = await instance.method("predict").remote(["input"]);
```

## Job Queue Pattern

Use `modal.Queue` + `modal.Function.spawn()` for async job processing:

```python
import modal

app = modal.App()
job_queue = modal.Queue.from_name("jobs", create_if_missing=True)
results = modal.Dict.from_name("results", create_if_missing=True)

@app.function()
def process_job(job_id: str, data: dict):
    result = heavy_computation(data)
    results[job_id] = result
    return result

# Submit from a web endpoint
@app.function()
@modal.fastapi_endpoint(method="POST")
def submit(data: dict):
    import uuid
    job_id = str(uuid.uuid4())
    process_job.spawn(job_id, data)
    return {"job_id": job_id}
```

### Dataset Ingestion

For large-scale data processing:

```python
@app.function()
def process_shard(shard_path: str):
    import pyarrow.parquet as pq
    table = pq.read_table(shard_path)
    return transform(table)

@app.local_entrypoint()
def ingest():
    shard_paths = list_s3_shards("s3://bucket/data/")
    results = list(process_shard.map(shard_paths))
```

Use `order_outputs=False` and `return_exceptions=True` for resilient ingestion.

## Timeouts

```python
@app.function(timeout=600)   # max 10 min per invocation
def slow_task():
    ...
```

Default timeout: 300s (5 min). Max: 86,400s (24h). Container-level timeout kills the process.

## Symptom Triage

### ".map() is slow to start"
- Containers need to cold-start; use `min_containers` or `buffer_containers`
- Check if image build is the bottleneck

### "Some .map() inputs fail"
- Add `retries` to handle transient failures
- Catch exceptions in the map loop
- Check container logs for the specific failures

### ".spawn() result never arrives"
- Ensure you call `handle.get()` to retrieve the result
- Check if the Function errored (exceptions are raised on `.get()`)

### "Function times out"
- Increase `timeout` parameter
- Check if work is I/O-bound (consider async or concurrent inputs)

## Reference Map

- `references/map-starmap.md` — parallel batch execution, ordering, error handling
- `references/spawn.md` — fire-and-forget, FunctionHandle, async patterns
- `references/generators.md` — remote_gen, streaming, generator Functions
- `references/error-handling-retries.md` — Retries, exception propagation, timeouts

## Guardrails

- `.map()` results are ordered by default; use `order_outputs=False` for speed
- `.spawn()` results must be explicitly retrieved with `.get()`
- Remote exceptions are re-raised on the caller side
- Retries apply per-input, not per-container
- Generator Functions must `yield`, not `return` collections
- Default timeout is 5 min — always set explicitly for long-running jobs
- Use `return_exceptions=True` in `.map()` for resilient batch processing
