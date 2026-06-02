# Preemption

Distilled from Modal guide on preemption.

## What Is Preemption

Preemptible containers can be interrupted by Modal when capacity is needed. In exchange, they cost less.

## Enabling Preemption

```python
@app.function(gpu="A100", allow_preemption=True)
def train():
    ...
```

## Handling Preemption

When a container is preempted:
1. The function receives a `SIGTERM` signal
2. The container has a grace period to save state
3. The input is retried on a new container (if retries are configured)

### Graceful handling with @modal.exit

```python
@app.cls(gpu="A100", allow_preemption=True)
class Trainer:
    @modal.enter()
    def setup(self):
        self.checkpoint = load_checkpoint()

    @modal.exit()
    def save(self):
        save_checkpoint(self.checkpoint)

    @modal.method()
    def train_step(self, batch):
        ...
```

### Signal handling

```python
import signal

def handle_preemption(signum, frame):
    save_state()

signal.signal(signal.SIGTERM, handle_preemption)
```

## Best Practices

- Use preemption for fault-tolerant batch workloads (training with checkpoints)
- Save checkpoints to a Volume regularly
- Combine with `retries` for automatic retry on preemption
- Don't use preemption for latency-sensitive serving workloads
- Use `@modal.exit()` lifecycle hook for cleanup

## Cost vs Reliability Trade-off

- Preemptible: cheaper but may be interrupted
- Non-preemptible (default): more expensive, guaranteed execution
- For training: preemptible + checkpointing is cost-effective
- For inference serving: use non-preemptible for reliability
