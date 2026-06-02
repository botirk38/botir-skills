---
name: modal-workspace-admin
description: Workspace management, environments, RBAC, service users, billing, and SSO on Modal. Use when configuring team access, managing environments, setting up CI/CD service accounts, or administering workspace settings.
---

# Modal Workspace Administration

Use this skill when managing workspaces, environments, team access, service users, or billing on Modal.

## When to Use This Skill

- Setting up workspace environments (dev/staging/prod)
- Configuring role-based access control (RBAC)
- Creating service users for CI/CD
- Managing workspace membership and roles
- Understanding billing and pricing
- Setting up SSO (Okta, SAML)
- Managing deployments and rollbacks

## Workspaces

A workspace groups all Modal resources (Apps, Functions, Volumes, Secrets, etc.) for a team. Each workspace has an isolated namespace.

### Workspace roles

| Role | Permissions |
|------|------------|
| Owner | Full access including billing, workspace management, all environments |
| Manager | Same as Owner, except cannot modify Owner role |
| Member | Deploy and manage Apps; no billing or workspace settings access |

## Environments

Environments isolate Modal resources within a workspace (e.g., dev, staging, production).

```bash
# Create environment
modal environment create staging

# List environments
modal environment list

# Use specific environment
modal deploy --env staging my_app.py

# Set default environment
export MODAL_ENVIRONMENT=staging
```

### Restricted environments (RBAC)

```bash
# Create restricted environment
modal environment create --restricted production
```

When an environment is restricted:
- All Members get **Viewer** access (read-only)
- Owners/Managers get **Contributor** access automatically
- Specific users can be granted Contributor access

| Environment Role | Permissions |
|-----------------|------------|
| Viewer | Read-only: dashboards, logs, metrics, config |
| Contributor | Full read/write access to the environment |

## Service Users

Programmatic accounts for CI/CD and automated workflows.

### Create

1. Workspace settings → Tokens → New Service User
2. Name must be lowercase alphanumeric (hyphens/underscores allowed)
3. Save the `MODAL_TOKEN_ID` and `MODAL_TOKEN_SECRET` — shown only once

### Use in CI/CD

```bash
export MODAL_TOKEN_ID=your-token-id
export MODAL_TOKEN_SECRET=your-token-secret
modal deploy your_app.py
```

### Security best practices

- Store tokens in CI/CD platform's secrets manager
- Assign service users Contributor role only on needed environments
- Use RBAC to isolate production from development

## Continuous Deployment

### GitHub Actions

```yaml
name: CI/CD
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      MODAL_TOKEN_ID: ${{ secrets.MODAL_TOKEN_ID }}
      MODAL_TOKEN_SECRET: ${{ secrets.MODAL_TOKEN_SECRET }}
    steps:
      - uses: actions/checkout@v6
      - uses: actions/setup-python@v5
        with:
          python-version: "3.10"
      - run: pip install modal
      - run: modal deploy -m my_package.my_file
```

For multiple environments: set `MODAL_ENVIRONMENT=xyz` in the workflow.

## Deployment Management

### Creating deployments

```bash
modal deploy -m my_package.my_file
```

### Key behaviors

- Redeploying an existing App increments its version
- Zero-downtime deployments: traffic gradually transitions to new version
- Old containers finish processing current requests before stopping
- Image build errors abort deployment with no change to running App

### Rollbacks

```bash
# Via CLI or dashboard — rollback to a previous version
# Available on Team and Enterprise plans
```

Rollbacks increment the version number but reset App state to a previous version.

### Stopping

```bash
modal app stop <app-name>
```

Stopping is destructive — stopped Apps must be redeployed from source.

## Go/TypeScript SDK Note

Workspace administration (RBAC, environments, service users, SSO) is managed via the Modal dashboard and CLI.

Go/TS SDKs authenticate using the same tokens (`MODAL_TOKEN_ID` / `MODAL_TOKEN_SECRET`):

```go
// Go — uses ~/.modal.toml or env vars automatically
mc, _ := modal.NewClient()
```

```typescript
// TypeScript — uses ~/.modal.toml or env vars automatically
const modal = new ModalClient();
```

Both SDKs can target specific environments:

```go
mc, _ := modal.NewClientWithOptions(&modal.ClientParams{
    Environment: "staging",
})
```

```typescript
const modal = new ModalClient({ environment: "staging" });
```

For CI/CD, you can use Go or TypeScript SDKs alongside `modal deploy` (Python) — e.g., a Go service that orchestrates Sandboxes while Python defines the Functions.

## Billing

Modal charges per second of resource usage:
- CPU cores
- Memory (MiB)
- GPU (per GPU-second by type)
- Disk (increases memory billing at 20:1 ratio)

You are billed for whichever is higher: your request or actual usage.

## References

- [rbac-reference.md](references/rbac-reference.md) — RBAC setup and environment roles
- [service-users-reference.md](references/service-users-reference.md) — Service user management

## Guardrails

- Service user tokens are long-lived — store securely, never in source code
- Use restricted environments to protect production
- Stopping an app is destructive and cannot be undone
- Always test deployments in a staging environment first
- Rollbacks require Team or Enterprise plan
