# Modal Queue — Reference

## API

```python
class Queue(modal.object.Object)
```

### Creation

```python
# Named (persisted)
q = modal.Queue.from_name("my-queue", create_if_missing=True)

# Ephemeral
with modal.Queue.ephemeral() as q:
    q.put("task")
```

### Put/Get

```python
q.put(value)                              # enqueue to default partition
q.put(value, partition="key")             # enqueue to named partition
q.put(value, partition="key", partition_ttl=60)  # custom TTL in seconds

item = q.get()                            # blocking dequeue (default partition)
item = q.get(timeout=5)                   # blocks up to 5s, raises queue.Empty
item = q.get(block=False)                 # immediate, raises queue.Empty if empty
item = q.get(partition="key")             # dequeue from named partition
```

### Batch operations

```python
q.put_many([1, 2, 3])                    # enqueue multiple items
items = q.get_many(3, timeout=5)          # dequeue up to 3 items
```

### Iteration

```python
# Read items immutably (does not remove them)
for item in q.iterate():
    print(item)
```

### Async

```python
await q.put.aio(value)
item = await q.get.aio()
await q.put_many.aio([1, 2, 3])
items = await q.get_many.aio(3)
```

## Partitions

Queues have independent FIFO partitions keyed by string. The default partition is an empty string.

Each partition:
- Has its own FIFO ordering
- Has independent TTL (default 24h after last put)
- Can hold up to 5,000 items

A single Queue supports up to 100,000 partitions.

## Limits

| Limit | Value |
|-------|-------|
| Max partitions per Queue | 100,000 |
| Max items per partition | 5,000 |
| Max item size | 1 MiB |
| Default partition TTL | 24 hours |

## Serialization

Objects serialized via `cloudpickle`, same as Dict. The deserializing environment needs matching libraries.

## Exceptions

```python
import queue

try:
    item = q.get(timeout=5)
except queue.Empty:
    print("Queue empty after timeout")

try:
    q.put(item, block=True, timeout=5)
except queue.Full:
    print("Queue full after timeout")
```

## Durability

Queues are backed by a replicated in-memory database. Persistence is likely but **not guaranteed**. Use Queues for active inter-function communication, not long-term storage. Contact Modal support if you need durable queues.
