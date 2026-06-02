# Modal Audit Logs — Reference

Audit logs are available on the Enterprise plan.

## Fields

| Field | Description |
|-------|-------------|
| `action` | Kind of change (e.g., `secret.create`, `app.deploy`) |
| `actor` | User or service user who initiated the action |
| `targets` | Resource(s) affected, recorded by ID |
| `context.environment` | Scoped environment |
| `context.ip_address` | Client IP address |
| `context.source` | `web` (dashboard) or `sdk` (CLI/client) |
| `status` | success or failure |
| `metadata` | Action-specific extra fields |

## Filtering

Use `key:value` pairs in search bar. Negate with `-` prefix.

| Filter | Matches |
|--------|---------|
| `action:secret.create` | All secret creations |
| `-status:success` | All non-successes |
| `action:volume.delete -actor_type:service` | Volume deletes by non-service users |

## Action Catalog

### App actions
- `app.deploy` — App deployed via `modal deploy` or `App.lookup`
- `app.rollback` — App rolled back to earlier version
- `app.rollover` — App rolled over (version redeployed)
- `app.run` — Ephemeral App started with `modal run`/`modal serve`
- `app.stop` — App stopped from dashboard or CLI

### Resource actions
- `secret.create`, `secret.delete`, `secret.get`, `secret.update`
- `volume.create`, `volume.delete`, `volume.get`
- `dict.create`, `dict.get`
- `queue.create`, `queue.get`
- `image.delete`
- `nfs.create`, `nfs.get`

### Workspace actions
- `member.set_role` — Workspace role changed
- `member.delete` — Member removed
- `invite.create` — Workspace invite generated
- `environment.create`, `environment.delete`, `environment.update`
- `environment.update_member` — Per-environment role changed

### Security actions
- `token.create`, `token.delete` — API token management
- `proxy.create`, `proxy.delete`, `proxy.add_ip`
- `domain.create`, `domain.delete`
- `access_grant.create`, `access_grant.approve`, `access_grant.revoke`
- `workspace.set_budget`

**Note**: Container runtime activity (function invocations, sandbox exec calls) is NOT audited.
