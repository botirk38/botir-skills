# Secrets Management

Distilled from Modal guide on Secrets and API reference.

## Dashboard

The Modal dashboard ([modal.com/secrets](https://modal.com/secrets)) is the primary UI for managing secrets:
- Create with templates (AWS, HuggingFace, Postgres, etc.)
- Edit existing secrets
- View secret metadata (not values)

## CLI

```bash
modal secret list                                    # list all
modal secret create name KEY1=val1 KEY2=val2         # create
modal secret create name KEY="$ENV_VAR"              # from env
modal secret delete name                             # delete
```

## Programmatic CRUD (v1.1.2+)

### Create

```python
modal.Secret.objects.create("my-secret", {"KEY": "value"})
modal.Secret.objects.create("my-secret", env_dict, allow_existing=True)  # no-op if exists
```

### List

```python
secrets = modal.Secret.objects.list()
for s in secrets:
    print(s.name)

# Filter
recent = modal.Secret.objects.list(max_objects=10, created_before="2025-01-01")
```

### Delete

```python
modal.Secret.objects.delete("my-secret")
modal.Secret.objects.delete("my-secret", allow_missing=True)
```

## Secret Construction

### from_name

```python
secret = modal.Secret.from_name("dashboard-secret")
secret = modal.Secret.from_name("secret", environment_name="production")
```

### from_dict

```python
secret = modal.Secret.from_dict({"KEY": "value", "OTHER": None})  # None values ignored
```

### from_dotenv

```python
secret = modal.Secret.from_dotenv()                   # reads ./.env
secret = modal.Secret.from_dotenv(path="/app")         # custom dir
secret = modal.Secret.from_dotenv(filename=".env.prod") # custom filename
```

### from_local_environ

```python
secret = modal.Secret.from_local_environ(["API_KEY", "DB_URL"])
```

Forwards specific local environment variables to the container.

## Environment Scoping

Secrets are scoped to environments:

```python
modal.Secret.from_name("secret", environment_name="dev")
modal.Secret.objects.list(environment_name="staging")
```

Defaults to the active environment.

## Injection Order

Multiple secrets applied in list order. Later values override earlier on collision:

```python
secrets=[modal.Secret.from_name("base"), modal.Secret.from_name("override")]
# If both have KEY, "override"'s value wins
```
