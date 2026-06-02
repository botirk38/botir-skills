# Dynamic Batching

Distilled from Modal guide on dynamic batching.

## What Is Dynamic Batching

Individual callers send single inputs. Modal collects them into batches for efficient GPU processing. The function receives and returns lists.

## Configuration

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

## Parameters

### max_batch_size

Maximum inputs collected into a single batch. Set based on GPU memory and throughput.

### wait_ms

Maximum time to wait for more inputs before processing the batch. Lower values reduce latency but may produce smaller batches.

## How Callers Use It

Callers send individual inputs as if the function takes a single input:

```python
# Each call sends one input
result = Model().predict.remote("hello")
```

Modal transparently batches multiple concurrent calls.

## Function Signature

The batched function must:
- Accept a `list` of inputs
- Return a `list` of outputs of the same length
- Output order must match input order

```python
@modal.batched(max_batch_size=32, wait_ms=50)
def predict(self, texts: list[str]) -> list[dict]:
    embeddings = self.model.encode(texts)
    return [{"embedding": e.tolist()} for e in embeddings]
```

## When to Use

- GPU inference where batch processing is more efficient
- Embedding generation
- Any workload where processing N items together is faster than N × 1 item

## Tips

- Increase `max_batch_size` for better GPU utilization
- Decrease `wait_ms` for lower latency (at the cost of smaller batches)
- Combine with `@modal.concurrent` for handling many simultaneous callers
- Monitor GPU utilization to tune batch size
