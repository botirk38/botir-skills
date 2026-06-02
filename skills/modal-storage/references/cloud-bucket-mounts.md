# Cloud Bucket Mounts

Distilled from Modal guide on cloud bucket mounts.

## Overview

`modal.CloudBucketMount` mounts cloud storage as a filesystem. Supports:
- AWS S3
- Google Cloud Storage (GCS)
- Cloudflare R2 (S3-compatible)

Built on AWS mountpoint technology.

## S3

```python
s3_creds = modal.Secret.from_dict({
    "AWS_ACCESS_KEY_ID": "...",
    "AWS_SECRET_ACCESS_KEY": "...",
    "AWS_REGION": "us-east-1"
})

@app.function(
    volumes={"/data": modal.CloudBucketMount("my-bucket", secret=s3_creds)}
)
def process():
    import os
    print(os.listdir("/data"))
```

### Specify region

Always include `AWS_REGION` in the secret to avoid auto-detection failures when container and bucket are in different regions.

### Temporary credentials

Include `AWS_SESSION_TOKEN` in the secret. Tokens expire and must be manually refreshed.

### OIDC (no manual tokens)

```python
@app.function(
    volumes={
        "/data": modal.CloudBucketMount(
            bucket_name="my-bucket",
            oidc_auth_role_arn="arn:aws:iam::123456789:role/modal-role"
        )
    }
)
def process(): ...
```

Requires AWS IAM role configured to trust Modal's OIDC provider.

## Google Cloud Storage

```python
gcs_creds = modal.Secret.from_name("gcs-credentials")

@app.function(
    volumes={"/gcs": modal.CloudBucketMount("my-gcs-bucket", secret=gcs_creds)}
)
def process(): ...
```

## Cloudflare R2

S3-compatible. Use the same pattern as S3 with R2 API tokens that have read/write/list permissions.

## Options

### Subdirectory mounting

```python
modal.CloudBucketMount("bucket", key_prefix="path/to/dir/", secret=creds)
```

`key_prefix` must end with `/`. Only files under that prefix are mounted.

### Read-only mode

```python
modal.CloudBucketMount("bucket", read_only=True, secret=creds)
```

### Requester-pays buckets

```python
modal.CloudBucketMount("bucket", requester_pays=True, secret=creds)
```

## IAM Permissions (S3)

Minimum required permissions:
- `s3:GetObject`
- `s3:PutObject`
- `s3:ListBucket`
- `s3:DeleteObject` (for writes)
- `s3:GetBucketLocation` (for region detection)

## Limitations

Inherits from AWS mountpoint:
- Not a full POSIX filesystem
- Random writes and appends may not work
- Best for sequential read/write patterns
- See mountpoint documentation for full details
