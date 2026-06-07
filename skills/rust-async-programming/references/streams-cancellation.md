# Async Streams & Cancellation Safety

## Async Streams

### The Stream Trait

```rust
pub trait Stream {
    type Item;
    fn poll_next(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Option<Self::Item>>;
}
```

### Creating Streams

```rust
use tokio_stream::StreamExt;

// From iterator
let stream = tokio_stream::iter(vec![1, 2, 3]);

// From channel
let (tx, rx) = tokio::sync::mpsc::channel(100);
let stream = tokio_stream::wrappers::ReceiverStream::new(rx);

// Custom with async-stream crate
use async_stream::stream;
let s = stream! {
    for i in 0..10 {
        tokio::time::sleep(Duration::from_millis(100)).await;
        yield i;
    }
};
```

### Stream Adaptors

```rust
use tokio_stream::StreamExt;

let results: Vec<_> = stream
    .filter(|x| x % 2 == 0)
    .map(|x| x * 2)
    .take(5)
    .timeout(Duration::from_secs(1))  // per-item timeout
    .collect()
    .await;
```

### Consuming Streams

```rust
// while let
while let Some(item) = stream.next().await {
    process(item);
}

// Collect
let all: Vec<_> = stream.collect().await;

// Fold
let sum = stream.fold(0, |acc, x| async move { acc + x }).await;
```

### Buffered Concurrency

Process stream items concurrently with bounded parallelism:

```rust
use futures::stream::StreamExt;

let results: Vec<_> = stream::iter(urls)
    .map(|url| async move { fetch(url).await })
    .buffer_unordered(10)  // up to 10 concurrent fetches
    .collect()
    .await;
```

## Cancellation Safety

### What is Cancellation?

When a future is dropped before completion (e.g., in `select!`), it's "cancelled." Any work in progress is lost.

### select! and Cancellation

```rust
tokio::select! {
    data = read_stream(&mut stream) => { ... }
    _ = shutdown.recv() => { return; }
}
// If shutdown fires, read_stream is DROPPED
// If read_stream partially read bytes, they're LOST
```

### Cancellation-Safe Operations

| Safe | Unsafe |
|------|--------|
| `TcpStream::read()` | Reading into a buffer you manage |
| `channel::recv()` | Multi-step protocol sequences |
| `sleep()` | Transactions (partial commit) |
| `JoinHandle::await` | Buffered reads (`BufReader`) |

### Making Code Cancellation-Safe

```rust
// UNSAFE: partial read lost on cancellation
async fn read_exact(stream: &mut TcpStream, buf: &mut [u8]) {
    let mut pos = 0;
    while pos < buf.len() {
        let n = stream.read(&mut buf[pos..]).await?;
        pos += n;  // if cancelled here, pos is lost
    }
}

// SAFE: use select! only on cancellation-safe operations
loop {
    tokio::select! {
        result = stream.read(&mut buf) => {
            // Each iteration is self-contained
            let n = result?;
            process(&buf[..n]);
        }
        _ = shutdown.recv() => break,
    }
}
```

### Fuse Pattern

Prevent polling a completed future:

```rust
use futures::future::FusedFuture;
use futures::FutureExt;

let mut operation = some_operation().fuse();
let mut interval = tokio::time::interval(Duration::from_secs(1));

loop {
    tokio::select! {
        result = &mut operation, if !operation.is_terminated() => {
            handle(result);
        }
        _ = interval.tick() => {
            println!("still waiting...");
        }
    }
}
```

## Structured Concurrency

### TaskTracker (tokio)

```rust
use tokio_util::task::TaskTracker;

let tracker = TaskTracker::new();

for item in items {
    tracker.spawn(async move { process(item).await });
}

tracker.close();          // no more tasks can be spawned
tracker.wait().await;     // wait for all to complete
```

### Graceful Cancellation via CancellationToken

```rust
use tokio_util::sync::CancellationToken;

let token = CancellationToken::new();
let child_token = token.child_token();

tokio::spawn(async move {
    tokio::select! {
        _ = do_work() => {}
        _ = child_token.cancelled() => {
            // cleanup
        }
    }
});

// Later: cancel all tasks
token.cancel();
```

## Drop as Cancellation

In Rust, dropping a future IS cancellation. No special API needed:

```rust
// Option 1: Cancel by dropping the handle
let handle = tokio::spawn(long_running_task());
drop(handle);  // task continues but we won't get the result
```

```rust
// Option 2: Abort explicitly
let handle = tokio::spawn(long_running_task());
handle.abort();  // task is cancelled immediately
match handle.await {
    Ok(result) => { /* completed before abort */ }
    Err(e) if e.is_cancelled() => { /* was aborted */ }
    Err(e) => { /* panicked */ }
}
```
