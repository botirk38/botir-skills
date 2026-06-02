---
name: modal-storage
description: Persistent and cloud storage on Modal — Volumes, CloudBucketMount (S3/GCS/R2), and model weight management. Use when storing data, sharing files between Functions, managing ML model weights, or mounting cloud buckets.
---

# Modal Storage

Use this skill when working with persistent storage, sharing data between Functions, managing model weights, or mounting external cloud storage.

## When to Use This Skill

- Storing and loading ML model weights
- Sharing files between Functions
- Persisting data across container restarts
- Mounting S3, GCS, or Cloudflare R2 buckets
- Managing large datasets for batch processing

## Storage Options

| Type | Best For | Mutability | Performance |
|------|----------|-----------|-------------|
| Volume | Model weights, shared data | Write-once, read-many | High (distributed, cached) |
| CloudBucketMount | Existing cloud data | Read/write via mountpoint | Depends on bucket |
| NetworkFileSystem | *(deprecated)* | Read/write | Moderate |

## Volumes (Recommended)

### Create and use

```python
volume = modal.Volume.from_name("my-data", create_if_missing=True)

@app.function(volumes={"/data": volume})
def process():
    with open("/data/result.txt", "w") as f:
        f.write("hello")
```

### CLI management

```bash
modal volume create my-vol
modal volume ls my-vol /
modal volume put my-vol local/file remote/path
modal volume get my-vol remote/path local/file
```

### Key characteristics

- Distributed: accessible from any region
- Optimized for write-once, read-many
- Built-in caching and chunking
- Multiple Functions can mount the same Volume

## Model Weight Pattern

```python
volume = modal.Volume.from_name("model-weights", create_if_missing=True)
MODEL_DIR = "/models"

@app.function(volumes={MODEL_DIR: volume})
def download_model(model_id):
    from huggingface_hub import snapshot_download
    snapshot_download(model_id, local_dir=f"{MODEL_DIR}/{model_id}")

@app.cls(gpu="A100", volumes={MODEL_DIR: volume})
class Model:
    @modal.enter()
    def load(self):
        self.model = load_from_disk(f"{MODEL_DIR}/my-model")

    @modal.method()
    def predict(self, prompt):
        return self.model(prompt)
```

## CloudBucketMount (S3/GCS/R2)

```python
@app.function(
    volumes={
        "/bucket": modal.CloudBucketMount(
            "my-bucket",
            secret=modal.Secret.from_name("aws-creds")
        )
    }
)
def read_data():
    import os
    files = os.listdir("/bucket")
```

Supports AWS S3, Google Cloud Storage, and Cloudflare R2.

## Symptom Triage

### "FileNotFoundError on Volume"
- Check the Volume name matches: `modal volume ls vol-name /`
- Ensure `create_if_missing=True` on first use
- Verify the mount path in `volumes=` dict

### "Slow model loading"
- Store weights in a Volume (not downloaded at boot time)
- Use `@modal.enter()` to load once per container
- Pre-download to Volume via a separate Function

### "CloudBucketMount errors"
- Verify IAM permissions (read, write, list)
- Check `AWS_REGION` matches bucket region
- See `references/cloud-bucket-mounts.md` for troubleshooting

## Reference Map

- `references/volumes.md` — Volume lifecycle, CLI, batch upload, snapshots
- `references/cloud-bucket-mounts.md` — S3, GCS, R2 mounting, OIDC, key_prefix
- `references/model-weights.md` — model weight storage patterns and optimization

## Guardrails

- Volumes are optimized for write-once, read-many — not a general-purpose filesystem
- Use `volume.reload()` in long-running containers to see updates from other Functions
- CloudBucketMount inherits mountpoint-s3 limitations
- Don't store secrets in Volumes — use Modal Secrets instead
