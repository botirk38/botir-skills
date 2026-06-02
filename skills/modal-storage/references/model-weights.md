# Model Weight Management

Distilled from Modal guide on storing model weights.

## Recommended: Store in a Volume

### Download from a Modal Function

```python
volume = modal.Volume.from_name("model-weights", create_if_missing=True)

@app.function(gpu="any", volumes={"/models": volume})
def download_model():
    from huggingface_hub import snapshot_download
    snapshot_download("meta-llama/Llama-3-8B", local_dir="/models/llama-3-8b")
```

### Upload from client

```python
volume = modal.Volume.from_name("model-weights", create_if_missing=True)

@app.local_entrypoint()
def upload(local_path: str, remote_path: str):
    with volume.batch_upload() as upload:
        upload.put_directory(local_path, remote_path)
```

Or via CLI:

```bash
modal volume put model-weights ./model-dir /models/my-model
```

### Use with CloudBucketMount

If weights are in S3/GCS, mount the bucket directly:

```python
@app.function(
    volumes={"/models": modal.CloudBucketMount("model-bucket", secret=creds)}
)
def inference():
    model = load("/models/my-model")
```

## Loading Weights Efficiently

### Use @modal.enter() for one-time loading

```python
@app.cls(gpu="A100", volumes={"/models": volume})
class Model:
    @modal.enter()
    def setup(self):
        self.model = load_model("/models/llama-3-8b")

    @modal.method()
    def predict(self, prompt):
        return self.model(prompt)
```

Weights load once per container, not per input.

## Alternative: Store in Image

```python
def download():
    from huggingface_hub import snapshot_download
    snapshot_download("model-id", local_dir="/models")

image = (
    modal.Image.debian_slim()
    .pip_install("huggingface_hub")
    .run_function(download)
)
```

Performance is similar to Volumes, but less flexible:
- Image rebuilds trigger re-downloads
- Volumes allow updates without image changes

## Hugging Face Hub Pattern

```python
volume = modal.Volume.from_name("hf-models", create_if_missing=True)
HF_SECRET = modal.Secret.from_name("huggingface")  # HF_TOKEN

@app.function(
    volumes={"/models": volume},
    secrets=[HF_SECRET],
)
def download(model_id: str):
    from huggingface_hub import snapshot_download
    snapshot_download(model_id, local_dir=f"/models/{model_id}")
```

## Tips

- Volume storage is recommended over Image storage for model weights
- Use `@modal.enter()` to load weights once per container
- Pre-download weights to a Volume before deploying inference
- Use Secrets for authenticated model downloads (e.g., Hugging Face gated models)
- For very large models, check download speed with `modal volume` CLI
