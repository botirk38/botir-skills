---
name: modal-container-images
description: Define container environments for Modal Functions using Image method chaining. Use when installing Python packages, system dependencies, adding local files, using Dockerfiles, configuring CUDA/GPU images, or building custom container images.
---

# Modal Container Images

Use this skill when defining the environment (packages, system deps, files, GPU libraries) that Modal Functions run in.

## When to Use This Skill

- Installing Python packages for remote Functions
- Adding system packages (apt)
- Bringing local files/code into containers
- Using existing Dockerfiles or registry images
- Setting up CUDA/GPU-accelerated environments
- Optimizing image build times and layer caching

## Core Workflow

1. Start from a base image (usually `modal.Image.debian_slim()`)
2. Chain methods to add packages, files, and commands
3. Assign to Functions via `@app.function(image=my_image)` or set as App default

```python
image = (
    modal.Image.debian_slim(python_version="3.13")
    .apt_install("git")
    .uv_pip_install("torch<3")
    .env({"MY_VAR": "value"})
    .run_commands("echo 'ready'")
)

@app.function(image=image)
def f():
    import torch
```

## Image Methods (Build Order Matters)

### Python packages

```python
image.uv_pip_install("pandas==2.2.0", "numpy")   # fast, uses uv
image.pip_install("pandas==2.2.0", "numpy")       # fallback, uses pip
image.uv_pip_install_from_requirements("requirements.txt")
image.uv_pip_install_from_pyproject("pyproject.toml")
```

Pin versions tightly for reproducibility: `"torch==2.8.0"` over `"torch<3"`.

### System packages

```python
image.apt_install("ffmpeg", "libsndfile1")
```

### Local files and code

```python
image.add_local_dir("./config", remote_path="/root/config")
image.add_local_file("model.bin", remote_path="/root/model.bin")
image.add_local_python_source("my_module")  # by module name
```

By default, local files are added at container startup (fast redeploy). Use `copy=True` to bake into the image layer.

### Environment variables

```python
image.env({"TRANSFORMERS_CACHE": "/vol/cache"})
```

### Run arbitrary commands

```python
image.run_commands("git clone https://github.com/org/repo")
```

### Run a Python function during build

```python
def download_model():
    from huggingface_hub import snapshot_download
    snapshot_download("meta-llama/Llama-3-8B")

image = modal.Image.debian_slim().pip_install("huggingface_hub").run_function(download_model)
```

## Alternative Base Images

```python
modal.Image.from_registry("nvidia/cuda:12.4.0-devel-ubuntu22.04")
modal.Image.from_dockerfile("./Dockerfile")
modal.Image.from_gcp_artifact_registry("us-docker.pkg.dev/project/repo/image:tag")
modal.Image.from_aws_ecr("123456789.dkr.ecr.us-east-1.amazonaws.com/my-image:tag")
```

## Symptom Triage

### "Module not found" in container
- Add the package via `uv_pip_install` or `pip_install`
- For local code, use `add_local_python_source("module_name")`
- Import inside the function body, not at module top level

### Slow image builds
- Order methods so slow/stable layers come first (apt_install before pip_install)
- Pin versions to maximize layer cache hits
- Use `uv_pip_install` over `pip_install` for faster installs
- Use `add_local_dir` without `copy=True` for frequently changing files

### GPU library issues
- See `references/gpu-images-cuda.md` for CUDA stack details
- Use `from_registry` with NVIDIA base images for bleeding-edge CUDA
- `torch` bundles CUDA deps — just `pip_install("torch")`

## Reference Map

- `references/image-basics.md` — method chaining, layer caching, base images
- `references/gpu-images-cuda.md` — CUDA stack, torch, flash-attn setup
- `references/dockerfiles-registries.md` — Dockerfile, ECR, GCP AR, custom registries
- `references/local-files-mounts.md` — add_local_dir, add_local_file, add_local_python_source

## Guardrails

- Import heavy libraries inside function bodies, not at module top level
- Pin dependency versions for reproducible builds
- Prefer `uv_pip_install` over `pip_install` for speed
- Each Function can have its own image — no need for virtual environments
- Images are cached and reused across deployments when unchanged
