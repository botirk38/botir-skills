# GPU Images and CUDA

Distilled from Modal guide on using CUDA.

## CUDA Stack on Modal

The CUDA stack has multiple layers. Modal pre-installs the lower layers:

| Level | Component | Pre-installed? |
|-------|-----------|---------------|
| 0 | Kernel-mode driver | Yes (host) |
| 1 | User-mode driver API (`libcuda.so`, `nvidia-smi`) | Yes |
| 2 | CUDA Toolkit (`nvcc`, `cudnn`, `nvrtc`) | No — install via pip or base image |

## Simple GPU Setup

For most workloads, just pip_install the library. Libraries like `torch` bundle CUDA deps:

```python
image = modal.Image.debian_slim().pip_install("torch")

@app.function(gpu="A100", image=image)
def train():
    import torch
    assert torch.cuda.is_available()
```

## Bleeding-Edge Libraries (flash-attn, etc.)

Some libraries need the full CUDA Toolkit. Options:

### Use an NVIDIA base image

```python
image = (
    modal.Image.from_registry("nvidia/cuda:12.4.0-devel-ubuntu22.04", add_python="3.11")
    .pip_install("flash-attn", extra_options="--no-build-isolation")
)
```

### Use Modal's CUDA installer

```python
image = (
    modal.Image.debian_slim()
    .pip_install("nvidia-cuda-nvcc-cu12", "nvidia-cuda-runtime-cu12")
    .pip_install("flash-attn")
)
```

## Verifying GPU Access

```python
@app.function(gpu="any")
def check():
    import subprocess
    output = subprocess.check_output(["nvidia-smi"], text=True)
    print(output)
```

## CUDA Version

Modal provides CUDA Driver API version 13.0 (driver version 580.95.05). This is forward-compatible with all CUDA Toolkit versions up to 13.0.

## Tips

- `torch`, `tensorflow`, `jax` bundle their CUDA deps — just pip_install them
- For `flash-attn`: use `--no-build-isolation` and ensure CUDA toolkit is available
- Use `from_registry` with NVIDIA images when you need `nvcc` for compilation
- The `add_python` parameter on `from_registry` installs Python in the NVIDIA image
