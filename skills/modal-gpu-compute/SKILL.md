---
name: modal-gpu-compute
description: GPU acceleration on Modal — selecting GPUs, multi-GPU, fallbacks, preemption handling, and cost optimization. Use when running ML inference, training, or any GPU-accelerated workload.
---

# Modal GPU Compute

Use this skill when configuring GPU resources for Modal Functions, choosing GPU types, or handling preemption and cost trade-offs.

## When to Use This Skill

- Running ML inference or training on GPUs
- Choosing between GPU types (T4, L4, A10, L40S, A100, H100, H200, B200)
- Configuring multi-GPU containers
- Handling preemptible/spot GPU pricing
- Optimizing GPU cost vs performance

## GPU Types Available

| GPU | Architecture | VRAM | Best For |
|-----|-------------|------|----------|
| T4 | Turing | 16 GB | Light inference, dev |
| L4 | Ada Lovelace | 24 GB | Inference, video |
| A10 | Ampere | 24 GB | Inference |
| L40S | Ada Lovelace | 48 GB | Inference (excellent cost/perf) |
| A100-40GB | Ampere | 40 GB | Training, large inference |
| A100-80GB | Ampere | 80 GB | Large models |
| RTX-PRO-6000 | Ada Lovelace | 48 GB | Professional workloads |
| H100 | Hopper | 80 GB | High-perf training/inference |
| H200 | Hopper | 141 GB | Memory-bound workloads |
| B200 | Blackwell | 192 GB | Flagship performance |

## Quickstart

```python
@app.function(gpu="A100")
def train():
    import torch
    assert torch.cuda.is_available()
```

## Specifying GPU Count

Append `:n` for multi-GPU (up to 8 for most types):

```python
@app.function(gpu="H100:8")
def train_large_model():
    ...
```

## GPU Fallbacks

Specify a list of acceptable GPUs for availability:

```python
@app.function(gpu=["H100", "A100-80GB", "A100-40GB"])
def flexible_inference():
    ...
```

Modal tries GPUs in order, using the first available.

## Picking a GPU

- **Start with L40S** for inference — best cost/performance ratio, 48 GB VRAM
- **A100-80GB** for training or models >40 GB
- **H100/H200** for maximum throughput and large batch training
- **B200** for flagship performance (requires CUDA 13.0+)
- **T4/L4** for light inference or dev/testing

Check bottleneck type: memory-bound workloads (small batch LLM) don't benefit as much from faster GPUs — VRAM matters more than FLOPs.

## Automatic GPU Upgrades

- `gpu="H100"` may be upgraded to H200 at no extra cost
- `gpu="A100"` may be upgraded to A100-80GB at no extra cost
- Use `gpu="H100!"` to disable automatic upgrade (e.g., for benchmarking)
- Use `gpu="B200+"` to allow B200 or B300 at B200 pricing

## Symptom Triage

### "GPU not available in container"
- Ensure `gpu=` parameter is set on the Function decorator
- CUDA drivers are pre-installed; verify with `nvidia-smi`

### "CUDA out of memory"
- Use a GPU with more VRAM (A100-80GB, H200, B200)
- Reduce batch size or use model parallelism with multi-GPU

### "Slow cold starts on GPU Functions"
- Use `min_containers` to keep GPU containers warm
- Download model weights to a Volume ahead of time (not during boot)

## Reference Map

- `references/gpu-types-selection.md` — detailed GPU comparison and selection guide
- `references/multi-gpu.md` — multi-GPU configuration and use cases
- `references/preemption.md` — preemptible containers and handling interruptions

## Guardrails

- Know your bottleneck before upgrading GPU — memory-bound vs compute-bound
- Don't request more GPUs than needed; 8×H100 is expensive
- Use GPU fallback lists for better availability
- Download model weights ahead of time (Volume or Image) to avoid slow cold starts
