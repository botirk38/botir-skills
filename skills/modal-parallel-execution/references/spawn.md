# Spawn (Fire-and-Forget)

Distilled from Modal guides on async execution.

## .spawn()

Launch a Function invocation without blocking:

```python
handle = f.spawn(arg1, arg2)
```

Returns a `FunctionHandle` immediately. The Function runs asynchronously.

## Retrieving Results

```python
handle = f.spawn(data)
# ... do other work ...
result = handle.get()  # blocks until complete
```

### Error handling

```python
try:
    result = handle.get()
except Exception as e:
    print(f"Remote function failed: {e}")
```

## Batch Spawning

```python
handles = [process.spawn(item) for item in items]
results = [h.get() for h in handles]
```

This is similar to `.map()` but gives you more control over when results are collected.

## Use Cases

### Background processing

```python
@app.function()
@modal.fastapi_endpoint(method="POST")
def api(data: dict):
    # Start processing in background
    handle = heavy_compute.spawn(data)
    # Return immediately
    return {"status": "processing", "task_id": handle.object_id}
```

### Parallel with progress tracking

```python
handles = []
for item in work_items:
    handles.append(task.spawn(item))

completed = 0
for h in handles:
    h.get()
    completed += 1
    print(f"Progress: {completed}/{len(handles)}")
```

## spawn vs map

| Feature | .spawn() | .map() |
|---------|---------|--------|
| Returns | FunctionHandle | Iterator of results |
| Blocking | No (until .get()) | Yes (iterates results) |
| Error handling | Per-handle | Per-iteration |
| Best for | Background tasks, manual control | Batch parallel processing |

## Tips

- Use `.spawn()` when you need to do other work before collecting results
- Use `.map()` when you just want all results from a batch
- `.get()` re-raises any exception from the remote Function
- `handle.object_id` can be used for tracking/logging
