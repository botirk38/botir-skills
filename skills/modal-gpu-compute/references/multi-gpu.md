# Multi-GPU Configuration

Distilled from Modal guide on GPU acceleration.

## Requesting Multiple GPUs

Append `:n` to the GPU string:

```python
@app.function(gpu="H100:8")
def train_llama():
    ...
```

## Limits

| GPU | Max Count |
|-----|-----------|
| B200, H200, H100, A100, L4, T4, L40S | Up to 8 |
| A10 | Up to 4 |

All GPUs in a multi-GPU container are on the same physical machine (NVLink/NVSwitch connected).

## Considerations

- Requesting >2 GPUs usually results in longer wait times for allocation
- Multi-GPU primarily useful for:
  - Models too large for single GPU VRAM (tensor parallelism)
  - Data-parallel training
  - Running multi-GPU inference engines (vLLM, SGLang with tensor parallelism)

## Example: 8×H100 for Large Model

```python
@app.function(gpu="H100:8")
def serve_405b():
    # vLLM with tensor_parallel_size=8
    ...
```

## Tips

- Only request multi-GPU when single GPU VRAM is insufficient
- Check if quantization (FP8, INT4) can fit the model on fewer GPUs first
- Multi-GPU adds inter-GPU communication overhead
- Use the fewest GPUs that fit your model to minimize cost
