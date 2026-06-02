# Cold Start Performance

Distilled from Modal guide on cold start performance.

## What Is a Cold Start

A cold start occurs when no warm container is available and a new one must be started. Two sources of added latency:

1. **Queue time** — input waits for a container to become ready
2. **Initialization** — extra work on first invocation (imports, model loading)

## Reducing Queue Time

### Boot faster

Modal's container stack boots in ~1 second. But before warmup completes:
- Global scope code (imports) runs
- `@modal.enter()` methods execute

Optimizations:
- Download models ahead of time (to a Volume or during image build)
- Minimize global scope imports
- Use lightweight base images

### Keep more warm containers

#### scaledown_window

```python
@app.function(scaledown_window=300)
def f(): ...
```

Keep containers idle longer (2–1200 seconds). Default: 60s.

#### min_containers

```python
@app.function(min_containers=1)
def f(): ...
```

Never scale to zero. You pay for idle time.

#### buffer_containers

```python
@app.function(buffer_containers=2)
def f(): ...
```

Keep extra containers ready while the Function is active. Helps absorb bursts.

## Reducing Initialization Time

### Move model downloads to build time

```python
volume = modal.Volume.from_name("models", create_if_missing=True)

def download():
    from huggingface_hub import snapshot_download
    snapshot_download("meta-llama/Llama-3-8B", local_dir="/models/llama")

image = modal.Image.debian_slim().pip_install("huggingface_hub").run_function(
    download, volumes={"/models": volume}
)
```

### Use @modal.enter() for one-time setup

```python
@app.cls(gpu="A100", volumes={"/models": volume})
class Model:
    @modal.enter()
    def load(self):
        self.model = load_from_disk("/models/llama")

    @modal.method()
    def predict(self, prompt):
        return self.model(prompt)
```

Model loads once per container, not per input.

### Lazy imports

Import heavy libraries inside function bodies:

```python
@app.function()
def f():
    import torch  # only imported when container handles first input
    ...
```

## Measuring Cold Starts

- Check the Modal dashboard for container boot times
- Look for inputs in "pending" state (queue time)
- Compare first vs subsequent invocation latency
