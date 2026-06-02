# Sandbox Filesystem

Distilled from Modal guide on Sandbox filesystem access.

## Filesystem API

The filesystem API provides reliable file operations on running Sandboxes.

### Write and Read Text

```python
sb.filesystem.write_text("Hello World!\n", "/tmp/test.txt")
contents = sb.filesystem.read_text("/tmp/test.txt")
```

### Write and Read Bytes

```python
sb.filesystem.write_bytes(b"\x00\x01\x02", "/tmp/data.bin")
data = sb.filesystem.read_bytes("/tmp/data.bin")
```

### Copy Files

```python
# Local → Sandbox
sb.filesystem.copy_from_local("local-file.txt", "/tmp/remote.txt")

# Sandbox → Local
sb.filesystem.copy_to_local("/tmp/remote.txt", "local-copy.txt")
```

### Directory Operations

```python
# Create directory
sb.filesystem.make_directory("/tmp/project/results")

# List files
for entry in sb.filesystem.list_files("/tmp/project"):
    print(entry.name, entry.type.value, entry.size)

# Check file info
info = sb.filesystem.stat("/tmp/file.txt")
print(info.size, info.type)

# Remove files/dirs
sb.filesystem.remove("/tmp/project", recursive=True)
```

## Volume Mounting

Attach persistent Volumes to Sandboxes:

```python
volume = modal.Volume.from_name("shared-data", create_if_missing=True)

sb = modal.Sandbox.create(
    app=app,
    volumes={"/data": volume},
)

# Write to volume from sandbox
p = sb.exec("bash", "-c", "echo 'result' > /data/output.txt")
p.wait()
```

Volume data persists after Sandbox termination.

## Tips

- Use filesystem API for ad-hoc file operations
- Use Volumes for persistent data that outlives the Sandbox
- `copy_from_local` / `copy_to_local` handle streaming internally
- Always check file existence before reading in scripts
