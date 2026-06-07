---
name: zig-concurrency
description: Concurrency primitives in Zig including threads, atomics, SIMD vectors, and synchronization. Use when writing multi-threaded code, using atomic operations, or leveraging SIMD.
---

# Zig Concurrency

Use this skill when writing multi-threaded programs, using atomic operations, or leveraging SIMD vectors for data parallelism.

## When to Use This Skill

- Spawning and managing threads
- Using atomic operations for lock-free programming
- Working with SIMD vectors
- Thread-local storage
- Synchronization primitives (mutex, etc.)
- Single-threaded build considerations

## Threads

```zig
const std = @import("std");

pub fn main() !void {
    var threads: [4]std.Thread = undefined;

    for (&threads, 0..) |*t, i| {
        t.* = try std.Thread.spawn(.{}, worker, .{i});
    }

    for (threads) |t| {
        t.join();
    }
}

fn worker(id: usize) void {
    std.debug.print("Worker {} started\n", .{id});
    // Do work...
}
```

### Thread Configuration

```zig
const thread = try std.Thread.spawn(.{
    .stack_size = 8 * 1024 * 1024,  // 8MB stack
}, workerFn, .{arg1, arg2});
```

## Atomic Operations

```zig
const std = @import("std");

var counter = std.atomic.Value(u32).init(0);

fn increment() void {
    _ = counter.fetchAdd(1, .seq_cst);
}

fn load() u32 {
    return counter.load(.acquire);
}

fn store(val: u32) void {
    counter.store(val, .release);
}
```

### Memory Orderings

| Ordering | Use Case |
|----------|----------|
| `.relaxed` | Counters, statistics (no synchronization) |
| `.acquire` | Reading shared data after a flag |
| `.release` | Publishing shared data before a flag |
| `.acq_rel` | Read-modify-write on synchronization vars |
| `.seq_cst` | Default, strongest (sequential consistency) |

### Compare-and-Swap

```zig
fn tryIncrement(expected: u32) bool {
    return counter.cmpxchgWeak(expected, expected + 1, .seq_cst, .seq_cst) == null;
}

// Strong version (no spurious failures)
const result = counter.cmpxchgStrong(old, new, .acq_rel, .acquire);
```

## Thread-Local Variables

```zig
threadlocal var tls_buffer: [4096]u8 = undefined;
threadlocal var tls_count: u32 = 0;

fn perThreadWork() void {
    tls_count += 1;
    // Each thread has its own copy
}
```

In single-threaded builds (`-fsingle-threaded`), threadlocal variables become regular globals.

## SIMD Vectors

Hardware-accelerated parallel operations:

```zig
const Vec4f = @Vector(4, f32);

fn addVectors(a: Vec4f, b: Vec4f) Vec4f {
    return a + b;  // 4 additions in one instruction
}

fn dotProduct(a: Vec4f, b: Vec4f) f32 {
    const product = a * b;
    return @reduce(.Add, product);
}
```

### Vector Operations

```zig
const v: @Vector(8, i32) = .{ 1, 2, 3, 4, 5, 6, 7, 8 };

// Element-wise arithmetic
const doubled = v * @as(@Vector(8, i32), @splat(2));

// Comparison (returns vector of bool)
const mask = v > @as(@Vector(8, i32), @splat(4));

// Select elements based on mask
const result = @select(i32, mask, v, @as(@Vector(8, i32), @splat(0)));
// result = { 0, 0, 0, 0, 5, 6, 7, 8 }

// Shuffle
const shuffled = @shuffle(i32, v, undefined, .{ 3, 2, 1, 0, 7, 6, 5, 4 });

// Reduce
const sum = @reduce(.Add, v);  // 36
const max = @reduce(.Max, v);  // 8
```

### Converting Between Vectors and Arrays

```zig
const arr = [4]f32{ 1.0, 2.0, 3.0, 4.0 };
const vec: @Vector(4, f32) = arr;  // array to vector
const back: [4]f32 = vec;          // vector to array
```

## Mutex

```zig
const std = @import("std");

var mutex = std.Thread.Mutex{};
var shared_data: u64 = 0;

fn safeIncrement() void {
    mutex.lock();
    defer mutex.unlock();
    shared_data += 1;
}
```

## Single-Threaded Builds

Compile with `-fsingle-threaded` for:
- Threadlocal becomes regular global
- Mutexes become no-ops
- Atomic operations become regular loads/stores
- Smaller, faster binary
