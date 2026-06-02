---
name: modal-sandboxes
description: Secure, isolated containers for executing untrusted or agent-generated code on Modal — including snapshots, Docker-in-sandbox, and network controls. Use when running LLM-generated code, building coding agents, snapshotting state, or executing arbitrary user code safely.
---

# Modal Sandboxes

Use this skill when creating isolated containers for untrusted code execution, building coding agents, snapshotting sandbox state, or running arbitrary commands in secure environments.

## When to Use This Skill

- Executing LLM-generated code safely
- Building coding agent infrastructure
- Running untrusted user code in isolation
- Creating temporary environments for testing
- Executing git operations (clone, test, lint) in isolation
- Snapshotting sandbox state (filesystem, directory, memory)
- Running Docker containers inside sandboxes

## Quick Start

```python
import modal

app = modal.App.lookup("my-sandbox-app", create_if_missing=True)
sb = modal.Sandbox.create(app=app)

p = sb.exec("python", "-c", "print('hello')", timeout=3)
print(p.stdout.read())

sb.terminate()
sb.detach()
```

No `modal run` needed — run directly with `python script.py`.

## Sandbox Lifecycle

1. **Created** — registered, no resources allocated yet
2. **Scheduled** — worker provisioning resources
3. **Started** — container running, ready for `exec()`
4. **Ready** — readiness probe succeeded (if configured)
5. **Completed/Terminated/Error** — final states

```python
sb = modal.Sandbox.create(app=app, timeout=600)

# Execute commands
p = sb.exec("bash", "-c", "echo hello")
p.wait()

# Terminate when done
sb.terminate()
sb.detach()  # release client reference
```

## Running Commands

```python
# Simple command
p = sb.exec("python", "-c", "print(42)", timeout=5)
stdout = p.stdout.read()

# Stream output
p = sb.exec("bash", "-c", "for i in {1..5}; do echo $i; sleep 1; done")
for line in p.stdout:
    print(line, end="")

# Check exit code
p.wait()
print(p.returncode)
```

## Filesystem API

```python
# Write files
sb.filesystem.write_text("content", "/tmp/file.txt")

# Read files
text = sb.filesystem.read_text("/tmp/file.txt")

# Copy to/from local
sb.filesystem.copy_from_local("local.txt", "/tmp/remote.txt")
sb.filesystem.copy_to_local("/tmp/remote.txt", "local-copy.txt")

# List and manage
entries = sb.filesystem.list_files("/tmp")
info = sb.filesystem.stat("/tmp/file.txt")
sb.filesystem.make_directory("/tmp/project")
sb.filesystem.remove("/tmp/project", recursive=True)
```

## Configuration

```python
sb = modal.Sandbox.create(
    app=app,
    image=modal.Image.debian_slim().pip_install("requests"),
    gpu="T4",
    cpu=2,
    memory=4096,
    timeout=300,
    secrets=[modal.Secret.from_name("api-key")],
    volumes={"/data": modal.Volume.from_name("shared-vol")},
    region=["us-west"],
    proxy=modal.Proxy.from_name("my-proxy"),
)
```

## Multi-Language Support

Sandboxes support Python, JavaScript/TypeScript, and Go clients.

## Symptom Triage

### "Sandbox command hangs"
- Always set `timeout` on `exec()` calls
- Check if the command expects stdin input
- Use `sb.terminate()` to force-stop

### "Cannot read files from Sandbox"
- Use the filesystem API, not volume mounts, for ad-hoc file access
- Ensure the file was actually created by checking with `sb.exec("ls", "/path")`

### "Sandbox internet access blocked"
- Internet access is available by default
- Check if proxy/firewall is configured

## Sandbox Snapshots

Capture and reuse sandbox state:

### Filesystem snapshot → Image

```python
# Install packages, configure environment
sb.exec("pip", "install", "pandas", "numpy").wait()
sb.exec("bash", "-c", "echo 'setup done' > /tmp/ready").wait()

# Capture full filesystem as a Modal Image
img = sb.snapshot_filesystem()

# Reuse in new Sandboxes — everything installed is already there
sb2 = modal.Sandbox.create(image=img, app=app)
```

Retained indefinitely. Snapshot is a full `modal.Image`.

### Directory snapshot

```python
# Capture a specific directory
snapshot_id = sb.snapshot_directory("/workspace/project")

# Restore into new sandbox
sb2 = modal.Sandbox.create(app=app)
sb2.restore_directory(snapshot_id, "/workspace/project")
```

Retained 30 days.

### Memory snapshot (Beta)

```python
# Capture full process memory state (like hibernation)
snapshot_id = sb.snapshot_memory()

# Resume from memory state — all in-flight processes continue
sb2 = modal.Sandbox.restore_memory(snapshot_id, app=app)
```

Retained 7 days.

## Docker in Sandboxes (Alpha)

Run Docker containers inside Modal Sandboxes using the Sysbox runtime:

```python
sb = modal.Sandbox.create(
    image=modal.Image.from_registry("nestybox/ubuntu-noble-systemd-docker"),
    encrypted_ports=[443],
    timeout=3600,
    memory=4096,
    cpu=4,
    app=app,
)

# Wait for Docker daemon
sb.exec("bash", "-c", "while ! docker info; do sleep 1; done").wait()

# Now use Docker normally inside the sandbox
sb.exec("docker", "pull", "nginx:latest").wait()
sb.exec("docker", "run", "-d", "-p", "80:80", "nginx:latest").wait()
```

### Caveats

- Uses Sysbox runtime for nested container isolation
- Higher resource requirements than standard sandboxes
- Alpha feature — may change

## Go SDK

```go
mc, _ := modal.NewClient()
app, _ := mc.Apps.FromName(ctx, "my-app", &modal.AppFromNameParams{CreateIfMissing: true})
image := mc.Images.FromRegistry("python:3.11", nil)

// Create sandbox with GPU and timeout
sb, _ := mc.Sandboxes.Create(ctx, app, image, &modal.SandboxCreateParams{
    GPU:       "T4",
    Timeout:   10 * time.Minute,
    MemoryMiB: 4096,
})
defer sb.Terminate(ctx, nil)

// Execute command
p, _ := sb.Exec(ctx, []string{"python", "-c", "print('hello')"}, nil)
stdout, _ := io.ReadAll(p.Stdout)
fmt.Println(string(stdout))

// Snapshot filesystem as Image
img, _ := sb.SnapshotFilesystem(ctx)
```

## TypeScript SDK

```typescript
import { ModalClient } from "modal";
const modal = new ModalClient();

const app = await modal.apps.fromName("my-app", { createIfMissing: true });
const image = modal.images.fromRegistry("python:3.13");

// Create sandbox
const sb = await modal.sandboxes.create(app, image, {
    gpu: "T4",
    timeoutMs: 600000,
    memoryMiB: 4096,
});

// Execute command
const proc = await sb.exec(["python", "-c", "print('hello')"]);
console.log(await proc.stdout.readText());

// Write and read files
const f = await sb.open("/tmp/data.txt", "w");
await f.write("some data");
await f.close();

// Snapshot filesystem
const snapshot = await sb.snapshotFilesystem();
await sb.terminate();
```

## Network Controls

```python
sb = modal.Sandbox.create(
    app=app,
    block_network=True,                          # no internet
    outbound_cidr_allowlist=["1.2.3.0/24"],      # whitelist outbound
    inbound_cidr_allowlist=["10.0.0.0/8"],       # whitelist inbound
    encrypted_ports=[443],                        # expose port via TLS tunnel
    unencrypted_ports=[8080],                     # expose port without TLS
)
```

### Access exposed ports

```python
tunnel = sb.tunnels()[443]
print(tunnel.url)  # https://<hash>.modal.run
```

## Reference Map

- `references/sandbox-basics.md` — creation, execution, lifecycle
- `references/sandbox-filesystem.md` — filesystem API, file operations
- `references/sandbox-networking.md` — tunnels, internet access, ports
- `references/sandbox-lifecycle.md` — events, probes, reconnection

## Guardrails

- Always set timeouts on exec() calls to prevent hanging
- Use `sb.terminate()` + `sb.detach()` when done
- Sandboxes require an App (use `App.lookup` for standalone scripts)
- gVisor sandboxing provides security isolation
- Each Sandbox is a separate container with its own filesystem
- Filesystem snapshots are retained indefinitely; memory snapshots expire in 7 days
- Docker-in-Sandbox is Alpha and has higher resource overhead
