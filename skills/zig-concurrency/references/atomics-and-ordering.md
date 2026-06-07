# Atomics and Memory Ordering

## std.atomic.Value

The primary type for atomic variables:

```zig
const Counter = std.atomic.Value(u64);
var global_counter = Counter.init(0);
```

### Operations

```zig
// Load
const val = counter.load(.acquire);

// Store
counter.store(42, .release);

// Fetch-and-modify (returns old value)
const old = counter.fetchAdd(1, .seq_cst);
const old2 = counter.fetchSub(1, .seq_cst);
const old3 = counter.fetchAnd(mask, .seq_cst);
const old4 = counter.fetchOr(bits, .seq_cst);
const old5 = counter.fetchXor(bits, .seq_cst);

// Compare-and-exchange
const result = counter.cmpxchgStrong(
    expected,   // expected current value
    desired,    // value to set if current == expected
    .acq_rel,   // success ordering
    .acquire,    // failure ordering
);
// result is null on success, or the actual value on failure
```

## Memory Ordering Explained

### .relaxed

No synchronization. Only guarantees atomicity of the operation itself:

```zig
// Good for: statistics counters, progress indicators
var stats_counter = std.atomic.Value(u64).init(0);
_ = stats_counter.fetchAdd(1, .relaxed);
```

### .acquire / .release

Paired for producer-consumer patterns:

```zig
var data: [1024]u8 = undefined;
var ready = std.atomic.Value(bool).init(false);

// Producer thread:
@memcpy(&data, payload);
ready.store(true, .release);  // all writes above are visible to acquirer

// Consumer thread:
while (!ready.load(.acquire)) {}  // spins until ready
// All writes by producer are now visible
useData(&data);
```

### .acq_rel

Combined acquire+release for read-modify-write:

```zig
// Used when the RMW both reads shared state and publishes new state
_ = counter.fetchAdd(1, .acq_rel);
```

### .seq_cst

Sequential consistency — all threads see operations in the same total order:

```zig
// Safest but slowest. Use when you need a total order across threads.
counter.store(val, .seq_cst);
```

## Fence

Memory fence without an atomic variable:

```zig
std.atomic.fence(.release);  // all prior writes become visible
std.atomic.fence(.acquire);  // all subsequent reads see prior writes
```

## Spinlock Pattern

```zig
const Spinlock = struct {
    locked: std.atomic.Value(bool) = std.atomic.Value(bool).init(false),

    pub fn lock(self: *Spinlock) void {
        while (self.locked.cmpxchgWeak(false, true, .acquire, .relaxed) != null) {
            std.atomic.spinLoopHint();
        }
    }

    pub fn unlock(self: *Spinlock) void {
        self.locked.store(false, .release);
    }
};
```

## Lock-Free Queue (Single Producer, Single Consumer)

```zig
fn SpscQueue(comptime T: type, comptime capacity: usize) type {
    return struct {
        buffer: [capacity]T = undefined,
        head: std.atomic.Value(usize) = .init(0),
        tail: std.atomic.Value(usize) = .init(0),

        pub fn push(self: *@This(), item: T) bool {
            const tail = self.tail.load(.relaxed);
            const next = (tail + 1) % capacity;
            if (next == self.head.load(.acquire)) return false; // full
            self.buffer[tail] = item;
            self.tail.store(next, .release);
            return true;
        }

        pub fn pop(self: *@This()) ?T {
            const head = self.head.load(.relaxed);
            if (head == self.tail.load(.acquire)) return null; // empty
            const item = self.buffer[head];
            self.head.store((head + 1) % capacity, .release);
            return item;
        }
    };
}
```

## @atomicRmw / @atomicLoad / @atomicStore (Builtins)

Lower-level builtins (prefer `std.atomic.Value` for most use):

```zig
var x: u32 = 0;
_ = @atomicRmw(u32, &x, .Add, 1, .seq_cst);
const val = @atomicLoad(u32, &x, .acquire);
@atomicStore(u32, &x, 42, .release);
```
