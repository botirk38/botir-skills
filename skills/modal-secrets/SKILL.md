---
name: modal-secrets
description: Secure credential and secret management on Modal. Use when providing API keys, database credentials, or tokens to Modal Functions, managing secrets via dashboard/CLI/code, or configuring OIDC authentication.
---

# Modal Secrets

Use this skill when managing credentials, API keys, tokens, or other sensitive configuration for Modal Functions.

## When to Use This Skill

- Providing API keys or credentials to remote Functions
- Managing secrets via dashboard, CLI, or code
- Using .env files with Modal
- Configuring OIDC for cloud provider authentication
- Sharing secrets across Functions or Apps

## Quick Start

```python
@app.function(secrets=[modal.Secret.from_name("my-secret")])
def f():
    import os
    api_key = os.environ["API_KEY"]
```

## Creating Secrets

### Dashboard (recommended for persistent secrets)

Go to [modal.com/secrets](https://modal.com/secrets). Templates available for common services (AWS, HuggingFace, Postgres, etc.).

### CLI

```bash
modal secret create my-secret KEY1=value1 KEY2=value2
modal secret list
modal secret delete my-secret
```

### Programmatically

```python
modal.Secret.objects.create("my-secret", {"KEY": "value"})
```

## Using Secrets

### from_name (reference dashboard/CLI secrets)

```python
@app.function(secrets=[modal.Secret.from_name("aws-creds")])
def f():
    key = os.environ["AWS_ACCESS_KEY_ID"]
```

### from_dict (inline for dev/testing)

```python
@app.function(secrets=[modal.Secret.from_dict({"FOO": "bar"})])
def f():
    print(os.environ["FOO"])
```

### from_dotenv (.env files)

```python
@app.function(secrets=[modal.Secret.from_dotenv()])
def f():
    print(os.environ["DATABASE_URL"])
```

### from_local_environ (forward local env vars)

```python
@app.function(secrets=[modal.Secret.from_local_environ(["API_KEY", "DB_URL"])])
def f():
    print(os.environ["API_KEY"])
```

### Multiple secrets

```python
@app.function(secrets=[
    modal.Secret.from_name("aws"),
    modal.Secret.from_name("db"),
])
def f(): ...
```

Applied in order — later secrets override earlier ones on key collision.

## App-Level Secrets

Inject secrets into all Functions in an App:

```python
app = modal.App(secrets=[modal.Secret.from_name("shared-secret")])
```

## Limits

- Key names: <=16,384 chars, letters/digits/underscores, cannot start with digit
- Values: <=32,768 chars
- For larger values, write to a Volume and read at runtime

## Symptom Triage

### "KeyError for environment variable"
- Check secret name matches: `modal secret list`
- Verify key name in the secret matches what you access
- Ensure `secrets=[...]` is in the function decorator

### "Secret not found"
- Check environment: secrets are environment-scoped
- Use `--env` flag or `environment_name=` parameter

## Reference Map

- `references/secrets-management.md` — CRUD operations, dashboard, CLI, code
- `references/oidc-integration.md` — OIDC for AWS, keyless authentication

## Guardrails

- Never hardcode secrets in source code
- Use `from_name` for production; `from_dict` for dev/testing only
- Secrets are environment-scoped — check your active environment
- Use the dashboard for secure secret editing (values are encrypted)
