# Sandbox Basics

Distilled from Modal guide on Sandboxes.

## What Are Sandboxes

Sandboxes are secure containers for executing untrusted code at runtime. Unlike Functions (declared at App definition time), Sandboxes are created dynamically.

Use cases:
- LLM-generated code execution
- Isolated test/lint environments
- User code execution (SaaS platforms)
- Git operations in clean environments

## Creating a Sandbox

```python
import modal

# From standalone script (no modal run needed)
app = modal.App.lookup("my-app", create_if_missing=True)
sb = modal.Sandbox.create(app=app)
```

### With custom image

```python
image = modal.Image.debian_slim().pip_install("numpy", "pandas")
sb = modal.Sandbox.create(app=app, image=image)
```

### With resources

```python
sb = modal.Sandbox.create(
    app=app,
    image=image,
    gpu="T4",
    cpu=4,
    memory=8192,
    timeout=600,
)
```

## Executing Commands

### Simple execution

```python
p = sb.exec("echo", "hello", timeout=5)
stdout = p.stdout.read()
stderr = p.stderr.read()
print(stdout)
```

### Streaming output

```python
p = sb.exec("python", "-c", "for i in range(10): print(i)")
for line in p.stdout:
    print(line, end="")
```

### Wait and check exit code

```python
p = sb.exec("make", "test", timeout=120)
p.wait()
if p.returncode != 0:
    print("Tests failed!")
    print(p.stderr.read())
```

### Writing to stdin

```python
p = sb.exec("python", "-i")
p.stdin.write("print(2+2)\n")
p.stdin.write("exit()\n")
p.stdin.eof()
print(p.stdout.read())
```

## Cleanup

```python
sb.terminate()  # stop the container
sb.detach()     # release client reference
```

Or use as context manager (if supported):

```python
# Always clean up, even on errors
try:
    sb = modal.Sandbox.create(app=app)
    # ... use sandbox
finally:
    sb.terminate()
    sb.detach()
```

## Tips

- No `modal run` needed for Sandbox scripts
- Set timeouts on all exec() calls
- Use `App.lookup` with `create_if_missing=True` for Sandbox Apps
- Sandboxes run with gVisor for security isolation
