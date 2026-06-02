# Dockerfiles and Registries

Distilled from Modal guide on Images.

## From Dockerfile

```python
image = modal.Image.from_dockerfile("./Dockerfile")
```

Modal builds the Dockerfile and uses the result as the base image. All standard Dockerfile instructions work.

You can add `context_mount` to include local files in the build context:

```python
image = modal.Image.from_dockerfile(
    "./Dockerfile",
    context_mount=modal.Mount.from_local_dir(".", remote_path="/build")
)
```

## From Public Registry

```python
image = modal.Image.from_registry("python:3.13-slim")
image = modal.Image.from_registry("nvidia/cuda:12.4.0-devel-ubuntu22.04", add_python="3.11")
```

Use `add_python` to install Python in images that don't include it.

## From AWS ECR

```python
image = modal.Image.from_aws_ecr(
    "123456789.dkr.ecr.us-east-1.amazonaws.com/my-image:latest",
    secret=modal.Secret.from_name("aws-ecr-creds")
)
```

## From GCP Artifact Registry

```python
image = modal.Image.from_gcp_artifact_registry(
    "us-docker.pkg.dev/project/repo/image:tag",
    secret=modal.Secret.from_name("gcp-creds")
)
```

## Extending Any Base

After `from_registry` or `from_dockerfile`, you can chain additional methods:

```python
image = (
    modal.Image.from_registry("python:3.13-slim")
    .apt_install("ffmpeg")
    .pip_install("torch")
    .run_commands("echo 'ready'")
)
```

## Tips

- Use `from_registry` with NVIDIA images for full CUDA toolkit access
- Private registries require a `secret` parameter with credentials
- `add_python` only needed when the base image lacks Python
- Prefer Modal's built-in methods (pip_install, apt_install) over Dockerfile when possible for better caching
