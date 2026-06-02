# Concurrent Inputs

Distilled from Modal guide on concurrent inputs.

## What Is Input Concurrency

By default, each container processes one input at a time. With `@modal.concurrent`, a single container handles multiple inputs simultaneously.

## When to Use

- I/O-bound workloads (API calls, database queries, file downloads)
- Web servers handling multiple requests
- Any workload where the container is mostly waiting

## Configuration

### On Functions

```python
@app.function()
@modal.concurrent(max_inputs=10)
def fetch_data(url):
    import requests
    return requests.get(url).json()
```

### On Class Methods

```python
@app.cls(gpu="A100")
class Model:
    @modal.enter()
    def setup(self):
        self.model = load_model()

    @modal.method()
    @modal.concurrent(max_inputs=100, target_inputs=50)
    def predict(self, x):
        return self.model(x)
```

## Parameters

### max_inputs

Maximum concurrent inputs per container. Set based on your workload:
- I/O-bound: higher values (50-200)
- CPU-bound: keep low (1-4)

### target_inputs

The concurrency level at which the autoscaler starts new containers. Default: 50% of max_inputs.

- Lower target → more containers, lower latency
- Higher target → fewer containers, higher utilization

## Interaction with Autoscaling

The autoscaler considers both concurrent slots and current load:
- If all slots are occupied, new containers start
- `target_inputs` controls the scaling trigger point
- `buffer_containers` adds extra headroom

## Tips

- Use for I/O-bound work, not CPU/GPU-bound work
- GPU inference with batching: use `@modal.batched` instead
- Test with realistic load to find optimal `max_inputs`
- Monitor container CPU/memory to avoid overloading
