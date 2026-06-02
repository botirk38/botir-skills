# OIDC Integration

Distilled from Modal guides on cloud bucket mounts and authentication.

## What Is OIDC

OpenID Connect (OIDC) allows Modal to generate short-lived identity tokens for authenticating to cloud providers without manual credential management.

## AWS OIDC

### Setup

1. Configure AWS to trust Modal's OIDC provider
2. Create an IAM role with appropriate permissions
3. Reference the role ARN in your Modal code

### Usage with CloudBucketMount

```python
@app.function(
    volumes={
        "/data": modal.CloudBucketMount(
            bucket_name="my-bucket",
            oidc_auth_role_arn="arn:aws:iam::123456789:role/modal-s3-role"
        )
    }
)
def process():
    import os
    print(os.listdir("/data"))
```

No secrets needed — Modal generates tokens automatically.

## Benefits

- No manual token rotation
- Short-lived tokens limit exposure window
- No secrets to manage or accidentally leak
- Works with AWS services that support OIDC federation

## When to Use

- Prefer OIDC over static AWS credentials when possible
- Especially useful for S3 bucket access
- Reduces secret management overhead
- More secure than long-lived access keys

## Limitations

- Currently supported for AWS
- Requires IAM role configuration in AWS
- Role must trust Modal's OIDC issuer
