# RBAC — Reference

RBAC is available on Team and Enterprise plans.

## Two-Level Role System

### Workspace Roles

| Role | Billing | Workspace Settings | All Environments | Assign Roles |
|------|---------|-------------------|------------------|-------------|
| Owner | Yes | Yes | Full access | All roles |
| Manager | Yes | Yes | Full access | All except Owner |
| Member | No | No | Unrestricted: full; Restricted: Viewer | No |

### Environment Roles

Only apply to restricted environments:

| Role | Read (dashboards, logs, metrics) | Write (deploy, manage) |
|------|--------------------------------|----------------------|
| Viewer | Yes | No |
| Contributor | Yes | Yes |

## Setting Up Restricted Environments

1. **Enable RBAC**: Requires Team or Enterprise plan
2. **Create restricted environment**:
   ```bash
   modal environment create --restricted production
   ```
   Or restrict existing: Settings → Environments → Make Restricted
3. **Configure access**: Settings → Environments → Manage → Add users with Contributor role

## Automatic Access

- All Members → Viewer on restricted environments
- Owners/Managers → Contributor on all restricted environments
- Service users follow same rules as human members

## Environment CLI Commands

```bash
modal environment create <name>              # Create unrestricted
modal environment create --restricted <name>  # Create restricted
modal environment list                        # List all environments
modal environment delete <name>               # Delete environment
```
