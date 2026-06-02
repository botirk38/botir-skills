# GPU Types and Selection

Distilled from Modal guide on GPU acceleration.

## Available GPUs

### Blackwell Architecture
- **B200**: 192 GB HBM3e. Flagship. Requires CUDA 13.0+.
- **B200+**: Allows B200 or B300 at B200 pricing.

### Hopper Architecture
- **H200**: 141 GB HBM3e, 4.8 TB/s bandwidth. 1.75x memory and 1.4x bandwidth over H100.
- **H100**: 80 GB HBM3. SXM variant. `gpu="H100"` may auto-upgrade to H200.
- **H100!**: Explicitly prevents H200 auto-upgrade.

### Ampere Architecture
- **A100-80GB**: 80 GB HBM2e. Workhorse for training.
- **A100-40GB**: 40 GB HBM2e. `gpu="A100"` may auto-upgrade to 80 GB.
- **A10**: 24 GB. Up to 4 GPUs per container.

### Ada Lovelace Architecture
- **L40S**: 48 GB. Excellent inference cost/performance.
- **L4**: 24 GB. Up to 8 GPUs.
- **RTX-PRO-6000**: 48 GB. Professional workloads.

### Turing Architecture
- **T4**: 16 GB. Budget option for light inference.

## Selection Guide

| Use Case | Recommended GPU |
|----------|----------------|
| Quick dev/testing | T4 or L4 |
| Standard inference (<24 GB) | L4 or A10 |
| Best inference value (<=48 GB) | L40S |
| Training (<=40 GB) | A100-40GB |
| Training (<=80 GB) | A100-80GB or H100 |
| Large models (>80 GB) | H200 or multi-GPU |
| Max performance | B200 |

## GPU Fallback Lists

```python
@app.function(gpu=["H100", "A100-80GB"])
def inference():
    ...
```

GPUs are tried in order. Improves availability at the cost of deterministic performance.

## Cost Considerations

- Per-second billing
- More powerful GPUs cost more per second but may finish faster
- Memory-bound workloads (small batch LLM) don't benefit much from faster GPUs
- Use `gpu="any"` for cheapest available GPU when type doesn't matter
