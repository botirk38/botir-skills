# Modal Dict — Reference

## API

```python
class Dict(modal.object.Object)
```

### Creation

```python
# Named (persisted across deployments)
kv = modal.Dict.from_name("my-dict", create_if_missing=True)

# Ephemeral (scoped to context manager)
with modal.Dict.ephemeral() as d:
    d["key"] = "value"

# Lookup existing
kv = modal.Dict.from_name("existing-dict")
```

### Read/Write

```python
kv["key"] = value         # set (__setitem__)
value = kv["key"]         # get (__getitem__), raises KeyError
del kv["key"]             # delete (__delitem__)
"key" in kv               # contains (__contains__)
len(kv)                   # length (__len__)

# Method-based (supports .aio for async)
kv.put("key", value)
value = kv.get("key")     # returns None if missing (unlike bracket)
kv.pop("key")             # get and remove
kv.clear()                # remove all entries
```

### Async

```python
await kv.put.aio("key", value)
value = await kv.get.aio("key")
await kv.pop.aio("key")
```

### Locking

```python
async with kv.lock("my-lock-key"):
    # Only one container can hold this lock at a time
    val = await kv.get.aio("counter")
    await kv.put.aio("counter", val + 1)
```

## Serialization

All keys and values serialized via `cloudpickle`. Any serializable Python object works as key or value — including lambdas, modules, and custom classes. The deserializing environment must have the same libraries installed.

## Limits

- **Per-object size**: 100 MiB max
- **Entries per update**: 10,000 max
- **Recommended object size**: < 5 MiB
- **Inactivity expiry**: 7 days with no reads or writes

## Caveats

- Mutable value updates (e.g., `kv["list"].append(x)`) are **local only** — the Dict is not updated until you re-assign: `kv["list"] = updated_list`
- Chained updates like `kv["outer"]["inner"] = val` do not propagate
- Network latency: ~tens of ms per operation. Use `.aio` to avoid blocking
