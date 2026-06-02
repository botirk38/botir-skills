# Volumes

Distilled from Modal guide on Volumes.

## Creating Volumes

### In code

```python
volume = modal.Volume.from_name("my-vol", create_if_missing=True)
```

### CLI

```bash
modal volume create my-vol
```

## Mounting in Functions

```python
@app.function(volumes={"/data": volume})
def process():
    # /data is backed by the Volume
    with open("/data/output.txt", "w") as f:
        f.write("result")
```

Multiple Functions can mount the same Volume at different or same paths.

## Volume CLI

```bash
modal volume list                          # list all volumes
modal volume ls my-vol /path               # list files
modal volume put my-vol local/file remote  # upload
modal volume get my-vol remote local/file  # download
modal volume rm my-vol /path               # delete files
```

## Batch Upload (Client-Side)

Push files from your local machine:

```python
volume = modal.Volume.from_name("my-vol", create_if_missing=True)

with volume.batch_upload() as upload:
    upload.put_directory("local/dir", "remote/dir")
    upload.put_file("local/file.bin", "remote/file.bin")
```

## Reading Updates

Volumes are eventually consistent. To see writes from other containers:

```python
volume.reload()  # refresh local cache
```

Call `reload()` before reading if another Function may have written recently.

## Snapshots

Volumes support point-in-time snapshots:

```python
snapshot = volume.snapshot()
# Later, restore or read from snapshot
```

## Performance Characteristics

- Optimized for write-once, read-many
- Built-in caching at container level
- Distributed across regions
- Chunked uploads for large files
- Not optimized for frequent small writes or random access patterns

## Volume with Image Builds

Mount volumes during image build for downloading large assets:

```python
def download():
    from huggingface_hub import snapshot_download
    snapshot_download("model-id", local_dir="/vol/model")

image = (
    modal.Image.debian_slim()
    .pip_install("huggingface_hub")
    .run_function(download, volumes={"/vol": volume})
)
```

## Tips

- Use `create_if_missing=True` for convenience
- Call `volume.reload()` before reads in multi-writer scenarios
- Use batch_upload for large local-to-remote transfers
- Use Volumes for model weights (better than Image storage for flexibility)
