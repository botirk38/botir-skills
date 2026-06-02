---
name: modal-gpu-compute
description: GPU acceleration on Modal — selecting GPUs, multi-GPU, multi-node clusters, CPU/memory/disk config, preemption, and cost optimization. Use when running ML inference, training, configuring compute resources, or any GPU-accelerated workload.
---

# Modal GPU Compute

Use this skill when configuring GPU resources, CPU/memory/disk, multi-node clusters, or handling preemption and cost trade-offs.

## When to Use This Skill

- Running ML inference or training on GPUs
- Choosing between GPU types (T4, L4, A10, L40S, A100, H100, H200, B200)
- Configuring multi-GPU containers or multi-node clusters
- Configuring CPU, memory, and disk resources
- Handling preemptible/spot GPU pricing
- Optimizing GPU cost vs performance
- Multi-node distributed training

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

## CPU, Memory, and Disk

Default: 0.125 CPU cores, 128 MiB memory. Containers can burst above if resources available.

```python
@app.function(
    cpu=8.0,            # 8 physical cores
    memory=32768,       # 32 GiB RAM
    ephemeral_disk=1000_000,  # ~1 TiB disk
)
def heavy_processing():
    ...
```

### Resource limits (request, limit) tuples

```python
@app.function(
    cpu=(1.0, 4.0),           # request 1 core, soft limit at 4
    memory=(1024, 2048),      # request 1 GiB, OOM kill at 2 GiB
)
def f():
    ...
```

Default CPU soft limit: 16 cores above request. Disk default quota: 512 GiB (max 3 TiB).

Billing: charged on max(request, actual usage). Disk increases memory billing at 20:1 ratio.

## Go SDK — GPU Sandboxes

```go
mc, _ := modal.NewClient()
app, _ := mc.Apps.FromName(ctx, "ml-app", &modal.AppFromNameParams{CreateIfMissing: true})

image := mc.Images.FromRegistry("nvidia/cuda:12.4.0-runtime-ubuntu22.04", nil)
image = image.DockerfileCommands([]string{
    "RUN pip install torch transformers",
}, &modal.ImageDockerfileCommandsParams{GPU: "A100"})

sb, _ := mc.Sandboxes.Create(ctx, app, image, &modal.SandboxCreateParams{
    GPU:       "A100",
    MemoryMiB: 16384,
    Timeout:   30 * time.Minute,
})

// Call GPU-accelerated function
fn, _ := mc.Functions.FromName(ctx, "ml-app", "infer", nil)
result, _ := fn.Remote(ctx, []any{"prompt text"}, nil)
```

## TypeScript SDK — GPU Sandboxes

```typescript
import { ModalClient } from "modal";
const modal = new ModalClient();

const app = await modal.apps.fromName("ml-app", { createIfMissing: true });
const image = modal.images
    .fromRegistry("nvidia/cuda:12.4.0-runtime-ubuntu22.04")
    .dockerfileCommands(["RUN pip install torch transformers"], { gpu: "A100" });

const sb = await modal.sandboxes.create(app, image, {
    gpu: "A100",
    memoryMiB: 16384,
    timeoutMs: 1800000,
});

const fn = await modal.functions.fromName("ml-app", "infer");
const result = await fn.remote(["prompt text"]);
```

GPU string format: `"T4"`, `"A100"`, `"H100:8"`, `"any"`, or arrays `["A100", "H100"]` for fallback.

## Multi-Node Clusters (Beta)

For distributed training across multiple nodes using `@modal.experimental.clustered`:

```python
import modal
import modal.experimental

@app.function(gpu="H100:8", timeout=60*60*24)
@modal.experimental.clustered(size=4, rdma=True)
def train_model():
    info = modal.experimental.get_cluster_info()
    # info.rank, info.cluster_id, info.container_ips
    ...
```

### Key features

- Gang scheduling: all nodes scheduled together or not at all
- 50 Gbps IPv6 private network + 3200 Gbps RDMA (RoCE)
- Up to 64 GPU devices across nodes
- Integrates with `torchrun` for PyTorch distributed training
- Only rank 0 output is returned to caller

### Cluster info

```python
info = modal.experimental.get_cluster_info()
info.rank                # container rank (0 = leader)
info.cluster_id          # unique cluster ID
info.container_ips       # IPv6 addresses sorted by rank
info.container_ipv4_ips  # IPv4 addresses sorted by rank
```

### Fault tolerance

- If rank 0 fails, the input is marked failed (regardless of other ranks)
- If any container is preempted, all are terminated and retried
- Use `retries=modal.Retries(...)` for automatic retry on preemption

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

### "OOM kill (container killed)"
- Set `memory=(request, limit)` with higher limit
- Or increase `memory` request

### "Disk write error (OSError)"
- Request larger `ephemeral_disk` (max 3 TiB)

## Reference Map

- `references/gpu-types-selection.md` — detailed GPU comparison and selection guide
- `references/multi-gpu.md` — multi-GPU and multi-node configuration
- `references/preemption.md` — preemptible containers and handling interruptions

## Guardrails

- Know your bottleneck before upgrading GPU — memory-bound vs compute-bound
- Don't request more GPUs than needed; 8×H100 is expensive
- Use GPU fallback lists for better availability
- Download model weights ahead of time (Volume or Image) to avoid slow cold starts
- Multi-node clusters require 8 GPUs per node (e.g., `H100:8`)
- CPU/memory billing: max(request, actual usage) — don't over-request
