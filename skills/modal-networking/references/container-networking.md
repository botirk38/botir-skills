# Container Networking

Distilled from Modal guide on container networking.

## Default Behavior

- Containers have full outbound internet access
- DNS resolution works normally
- Each container gets its own network namespace

## Inter-Container Communication

Containers in the same App cannot directly connect to each other by IP or hostname. Instead, use:

### Function calls

```python
@app.function()
def process(data):
    return transform(data)

@app.function()
def orchestrate():
    results = process.map(data_list)
```

### Volumes

Shared filesystem for data exchange:

```python
volume = modal.Volume.from_name("shared", create_if_missing=True)

@app.function(volumes={"/shared": volume})
def writer():
    with open("/shared/output.json", "w") as f:
        json.dump(data, f)

@app.function(volumes={"/shared": volume})
def reader():
    volume.reload()
    with open("/shared/output.json") as f:
        return json.load(f)
```

### Dict and Queue

In-memory distributed data structures:

```python
d = modal.Dict.from_name("cache", create_if_missing=True)
d["key"] = value
result = d["key"]

q = modal.Queue.from_name("tasks", create_if_missing=True)
q.put(item)
item = q.get()
```

## Network Isolation

- Containers are isolated via gVisor
- No shared network namespace between containers
- Each container has its own IP (not guaranteed static)
- Use Proxy for static outbound IPs

## Tips

- Don't attempt direct container-to-container TCP connections
- Use `.remote()`, `.map()`, Volumes, Dicts, or Queues for coordination
- For static IPs, use Proxies
- Containers can reach any public endpoint by default
