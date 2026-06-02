---
name: modal-sandboxes
description: Secure, isolated containers for executing untrusted or agent-generated code on Modal. Use when running LLM-generated code, building coding agents, creating isolated test environments, or executing arbitrary user code safely.
---

# Modal Sandboxes

Use this skill when creating isolated containers for untrusted code execution, building coding agents, or running arbitrary commands in secure environments.

## When to Use This Skill

- Executing LLM-generated code safely
- Building coding agent infrastructure
- Running untrusted user code in isolation
- Creating temporary environments for testing
- Executing git operations (clone, test, lint) in isolation

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
