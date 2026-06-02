# Image Basics

Distilled from Modal guide on Images.

## What Are Images

Containers run from stored filesystem "snapshots" called images. Modal Functions run in Debian Linux containers with a basic Python installation by default.

## Method Chaining

Images are built by chaining methods from a base. Each method adds a layer:

```python
image = (
    modal.Image.debian_slim(python_version="3.13")
    .apt_install("git", "ffmpeg")
    .uv_pip_install("torch==2.8.0", "transformers")
    .env({"CACHE_DIR": "/cache"})
    .run_commands("mkdir -p /cache")
)
```

Layer ordering matters for caching: put stable, slow layers (apt_install) before volatile layers (add_local_dir).

## Python Package Installation

### uv_pip_install (recommended)

```python
image.uv_pip_install("pandas==2.2.0", "numpy")
```

Uses `uv` for faster installs. Falls back to `pip_install` if issues arise.

### pip_install

```python
image.pip_install("pandas==2.2.0", "numpy")
```

Standard pip. Slower but more compatible.

### From requirements files

```python
image.uv_pip_install_from_requirements("requirements.txt")
image.uv_pip_install_from_pyproject("pyproject.toml")
```

## System Packages

```python
image.apt_install("ffmpeg", "libsndfile1", "curl")
```

## Environment Variables

```python
image.env({"MY_VAR": "value", "ANOTHER": "val2"})
```

## Run Commands

Execute shell commands during image build:

```python
image.run_commands(
    "git clone https://github.com/org/repo /opt/repo",
    "cd /opt/repo && make install"
)
```

## Run Python Functions During Build

```python
def setup():
    import something
    something.download_data()

image = (
    modal.Image.debian_slim()
    .pip_install("something")
    .run_function(setup)
)
```

The function runs during image build. Useful for downloading models or data.

Pass `secrets` or `volumes` to `run_function` if the setup needs credentials or persistent storage.

## Image Caching

- Modal caches image layers
- Unchanged layers are reused across deployments
- Changing a layer invalidates all downstream layers
- Pin versions to maximize cache hits

## Python Version

```python
modal.Image.debian_slim(python_version="3.13")
```

By default, matches your local Python minor version.

## Per-Function Images

Each Function can have its own image. No virtual environment juggling:

```python
@app.function(image=datascience_image)
def analyze(): ...

@app.function(image=web_image)
def serve(): ...
```
