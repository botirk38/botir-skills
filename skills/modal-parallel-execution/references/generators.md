# Generators

Distilled from Modal guides on streaming and generators.

## Generator Functions

Modal Functions can `yield` results incrementally:

```python
@app.function()
def process_chunks(file_path):
    with open(file_path) as f:
        for chunk in read_chunks(f):
            yield transform(chunk)
```

## Consuming Remote Generators

### .remote_gen()

Stream results from a generator Function:

```python
for chunk in process_chunks.remote_gen("/data/large.csv"):
    print(chunk)
```

Results arrive as they are yielded, not all at once.

### With streaming endpoints

```python
@app.function(gpu="A100")
def generate_tokens(prompt):
    for token in model.generate(prompt):
        yield token

@app.function(image=image)
@modal.fastapi_endpoint()
def stream(prompt: str):
    from fastapi.responses import StreamingResponse
    return StreamingResponse(
        generate_tokens.remote_gen(prompt),
        media_type="text/event-stream"
    )
```

## Generator vs Regular Functions

| Aspect | Regular | Generator |
|--------|---------|-----------|
| Return | Single value | Multiple values (yield) |
| Remote call | `.remote()` | `.remote_gen()` |
| Memory | Full result in memory | Streamed incrementally |
| Best for | Discrete results | Large/streaming data |

## Async Generators

```python
@app.function()
async def async_process():
    async for item in async_source():
        yield transform(item)
```

## Combining with .map()

Generator return values from `.map()` are collected per-input:

```python
@app.function()
def process(x):
    return x * 2  # regular return for map

for result in process.map(range(100)):
    print(result)
```

For streaming per-input results, use `.remote_gen()` in a loop.

## Tips

- Use generators for large outputs that should stream
- `.remote_gen()` is the remote equivalent of iterating a local generator
- Combine with `StreamingResponse` for SSE endpoints
- Memory-efficient for large datasets (no full result in memory)
