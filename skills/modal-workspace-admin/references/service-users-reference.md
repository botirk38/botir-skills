# Service Users — Reference

## What Are Service Users?

Programmatic accounts for automated systems to interact with Modal. Ideal for:
- CI/CD pipelines
- Automated deployments
- Scheduled infrastructure management

## Creation

1. Go to workspace tokens settings page
2. Click "New Service User"
3. Enter name (lowercase alphanumeric, hyphens/underscores allowed)
4. Click "Create"
5. **Save credentials immediately** — `MODAL_TOKEN_ID` and `MODAL_TOKEN_SECRET` shown only once

## Usage

```bash
export MODAL_TOKEN_ID=your-token-id
export MODAL_TOKEN_SECRET=your-token-secret

# Now use Modal CLI normally
modal deploy your_app.py
modal run your_script.py
```

## Permissions

Service users have the same base permissions as workspace Members:
- Can deploy and manage Apps
- Cannot access billing or workspace settings
- Subject to RBAC environment restrictions

## Deletion

1. Go to tokens settings page
2. Hover over the service user row
3. Click "Delete"

## Security Best Practices

1. **Store tokens in secrets manager** — never hardcode in source
2. **Use restricted environments** — grant Contributor role only on needed environments
3. **Rotate tokens periodically** — delete old service users, create new ones
4. **One service user per pipeline** — easier to audit and revoke
5. **Minimal permissions** — use RBAC to limit blast radius
