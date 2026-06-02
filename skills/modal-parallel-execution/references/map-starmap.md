# Map and Starmap

Distilled from Modal guides on parallel execution.

## .map()

Distribute a single-argument function across inputs in parallel:

```python
@app.function()
def square(x):
    return x ** 2

@app.local_entrypoint()
def main():
    results = list(square.map(range(100)))
    print(results)  # [0, 1, 4, 9, ...]
```

### Ordering

By default, results are returned in input order:

```python
results = list(f.map([3, 1, 2]))  # returns in order [f(3), f(1), f(2)]
```

For faster results (unordered):

```python
results = list(f.map(inputs, order_outputs=False))
```

### Error handling

Exceptions from individual inputs propagate to the caller:

```python
for result in f.map(inputs):
    try:
        process(result)
    except Exception as e:
        log_failure(e)
```

## .starmap()

For functions taking multiple arguments:

```python
@app.function()
def add(a, b):
    return a + b

@app.local_entrypoint()
def main():
    pairs = [(1, 2), (3, 4), (5, 6)]
    results = list(add.starmap(pairs))
    print(results)  # [3, 7, 11]
```

### With keyword arguments

```python
@app.function()
def process(data, config):
    ...

# Each tuple unpacked as positional args
inputs = [(data1, config1), (data2, config2)]
results = list(process.starmap(inputs))
```

## Scaling Behavior

- `.map()` and `.starmap()` trigger the autoscaler
- More inputs → more containers (up to `max_containers`)
- Empty input → no containers started
- The caller blocks until all results are collected (or iterate lazily)

## Lazy Iteration

Results are streamed as they complete. Iterate lazily for efficiency:

```python
for result in f.map(large_input_list):
    # Process each result as it arrives
    save(result)
```

Don't wrap in `list()` unless you need all results in memory.

## Combining with Streaming Endpoints

```python
@app.function(image=image)
@modal.fastapi_endpoint()
def parallel_api():
    from fastapi.responses import StreamingResponse
    return StreamingResponse(
        f.map(inputs, order_outputs=False),
        media_type="text/plain"
    )
```

## Tips

- Use `.map()` for single-arg parallel work
- Use `.starmap()` for multi-arg parallel work
- Iterate lazily when possible (don't `list()` unless needed)
- Set `order_outputs=False` when order doesn't matter for better performance
