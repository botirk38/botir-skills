# Environments

Distilled from Modal guide on managing environments.

## What Are Environments

Environments are isolated namespaces for Modal resources. Each environment has its own:

- Deployed Apps
- Secrets
- Volumes
- NetworkFileSystems (deprecated)

This enables dev/staging/production separation without resource conflicts.

## Creating and Managing

```bash
modal environment list
modal environment create dev
modal environment create staging
modal environment create production
modal environment delete dev
```

## Using Environments

### CLI

```bash
modal run --env staging script.py
modal deploy --env production script.py
```

### Default environment

```bash
modal config set-environment staging
```

All subsequent commands use `staging` unless overridden with `--env`.

### In Code

```python
modal.Secret.from_name("my-secret", environment_name="production")
modal.Volume.from_name("my-vol", environment_name="staging")
```

## Best Practices

- Use separate environments for dev and production
- Keep production Secrets isolated from development
- Set a safe default (e.g., dev) to prevent accidental production changes
- Deployed Apps in one environment are invisible to another
