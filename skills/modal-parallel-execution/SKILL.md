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
