# Autoscaling

Distilled from Modal guide on scaling out.

## How Autoscaling Works

Every Modal Function has an autoscaling container pool managed by Modal. The autoscaler:

- Spins up new containers when inputs queue and no capacity is available
- Spins down containers when resources idle
- Makes decisions quickly and frequently
- Scales to zero by default

## Parameters

```python
@app.function(
    max_containers=100,
    min_containers=1,
    buffer_containers=2,
    scaledown_window=300,
)
def f(): ...
```

### max_containers

Upper limit on concurrent containers. Protects against runaway costs.

### min_containers

Floor on warm containers. Prevents scale-to-zero. Containers stay warm even with no inputs. You pay for idle time.

### buffer_containers

Extra containers above current demand while the Function is active. Reduces latency for incoming bursts.

### scaledown_window

Max idle time (seconds) before shutdown. Range: 2–1200 (20 minutes). Default: 60s.

Containers may be shut down before the full window if the Function is substantially over-provisioned.

## Dynamic Autoscaler Updates

Update scaling parameters at runtime without redeploying:

```python
f = modal.Function.from_name("my-app", "my-function")
f.update_autoscaler(max_containers=50, buffer_containers=5)
```

Useful for responding to traffic spikes or scaling down for maintenance.

## Parallel Execution Patterns

### .map() for batch parallelism

```python
results = list(f.map(inputs))
```

Automatically scales containers to process inputs in parallel.

### .starmap() for multi-arg parallelism

```python
results = list(f.starmap([(a1, b1), (a2, b2)]))
```

### .spawn() for fire-and-forget

```python
handle = f.spawn(arg)
result = handle.get()  # retrieve later
```

## Cost vs Latency Trade-offs

| Strategy | Cost Impact | Latency Impact |
|----------|------------|----------------|
| `min_containers=0` (default) | Lowest | Cold starts possible |
| `min_containers=1` | Always-on cost | No cold starts |
| `buffer_containers=N` | Higher during activity | Smoother scaling |
| `scaledown_window=300` | Higher idle costs | Fewer cold restarts |
