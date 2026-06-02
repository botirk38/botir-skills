# Error Handling and Retries

Distilled from Modal guides on error handling.

## Exception Propagation

Exceptions from remote Functions are re-raised on the caller side:

```python
@app.function()
def risky():
    raise ValueError("something went wrong")

@app.local_entrypoint()
def main():
    try:
        risky.remote()
    except ValueError as e:
        print(f"Remote error: {e}")
```

## Retries

Configure automatic retries for transient failures:

```python
@app.function(retries=modal.Retries(
    max_retries=3,
    backoff_coefficient=2.0,
    initial_delay=1.0,
))
def flaky_api_call():
    response = requests.get("https://unreliable-api.com")
    response.raise_for_status()
    return response.json()
```

### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `max_retries` | — | Number of retry attempts |
| `backoff_coefficient` | 1.0 | Multiplier for delay between retries |
| `initial_delay` | 0.0 | Seconds before first retry |

### Retry behavior

- Retries apply per-input (not per-container)
- Failed input may retry on a different container
- Each retry starts fresh (no state from previous attempt)

### Simple retry syntax

```python
@app.function(retries=3)  # shorthand for max_retries=3
def f():
    ...
```

## Timeouts

Set maximum execution time per input:

```python
@app.function(timeout=300)  # 5 minutes
def long_running():
    ...
```

Timeout triggers a `TimeoutError` on the caller side.

## Error Handling in .map()

```python
results = []
errors = []
for i, input_val in enumerate(inputs):
    try:
        result = None
        for r in f.map([input_val]):
            result = r
        results.append(result)
    except Exception as e:
        errors.append((i, e))
```

Or handle inline:

```python
for result in f.map(inputs):
    try:
        process(result)
    except Exception as e:
        log_error(e)
```

## Combining Retries and Preemption

For preemptible containers, retries handle interruptions:

```python
@app.function(
    gpu="A100",
    allow_preemption=True,
    retries=modal.Retries(max_retries=3, initial_delay=5.0),
)
def train_step(batch):
    ...
```

## Tips

- Use retries for transient failures (network, rate limits)
- Don't retry for deterministic errors (bad input, logic bugs)
- Set timeouts to prevent hanging Functions
- Combine retries + preemption for cost-effective GPU work
- Catch exceptions from `.remote()` and `.map()` on the caller side
