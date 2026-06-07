# Allocators

## The Allocator Interface

All allocators implement `std.mem.Allocator`:

```zig
const Allocator = struct {
    pub fn alloc(self: Allocator, comptime T: type, n: usize) ![]T;
    pub fn free(self: Allocator, memory: []T) void;
    pub fn create(self: Allocator, comptime T: type) !*T;
    pub fn destroy(self: Allocator, ptr: *T) void;
    pub fn realloc(self: Allocator, old: []T, new_len: usize) ![]T;
    pub fn resize(self: Allocator, old: []T, new_len: usize) ?usize;
};
```

## Standard Allocators

### page_allocator

Direct OS page allocation. Simple but coarse-grained:

```zig
const allocator = std.heap.page_allocator;
const buf = try allocator.alloc(u8, 4096);
defer allocator.free(buf);
```

### FixedBufferAllocator

Stack-backed, no heap allocation:

```zig
var buffer: [1024]u8 = undefined;
var fba = std.heap.FixedBufferAllocator.init(&buffer);
const allocator = fba.allocator();

const data = try allocator.alloc(u8, 100);
// Fails with error.OutOfMemory if buffer exhausted
```

### ArenaAllocator

Bulk-free pattern — individual frees are no-ops:

```zig
var arena = std.heap.ArenaAllocator.init(std.heap.page_allocator);
defer arena.deinit();  // frees everything at once

const allocator = arena.allocator();
_ = try allocator.alloc(u8, 1000);
_ = try allocator.alloc(u8, 2000);
// All freed when arena.deinit() is called
```

Reset for reuse (e.g., per-frame in a game loop):
```zig
_ = arena.reset(.retain_capacity);  // keep backing memory
// or
_ = arena.reset(.free_all);         // return memory to OS
```

### DebugAllocator (General Purpose)

Full-featured allocator with safety checks:

```zig
var gpa: std.heap.DebugAllocator(.{}) = .init;
defer _ = gpa.deinit();
const allocator = gpa.allocator();
```

Features: double-free detection, use-after-free detection, leak reporting.

### c_allocator

Wraps libc malloc/free (only available when linking libc):

```zig
const allocator = std.heap.c_allocator;
```

### testing.allocator

For use in tests — reports memory leaks as test failures:

```zig
test "no leaks" {
    const allocator = std.testing.allocator;
    const buf = try allocator.alloc(u8, 10);
    defer allocator.free(buf);
}
```

## Passing Allocators

Convention: accept `Allocator` as a parameter, never use globals:

```zig
pub fn ArrayList(comptime T: type) type {
    return struct {
        items: []T,
        capacity: usize,
        allocator: std.mem.Allocator,

        pub fn init(allocator: std.mem.Allocator) @This() {
            return .{ .items = &.{}, .capacity = 0, .allocator = allocator };
        }

        pub fn deinit(self: *@This()) void {
            self.allocator.free(self.items.ptr[0..self.capacity]);
        }

        pub fn append(self: *@This(), item: T) !void {
            // ... grow if needed using self.allocator
        }
    };
}
```

## Handling Allocation Failure

Zig treats `OutOfMemory` as a regular error — no exceptions, no panics:

```zig
const data = allocator.alloc(u8, n) catch |err| switch (err) {
    error.OutOfMemory => return error.OutOfMemory,
};
// or simply:
const data = try allocator.alloc(u8, n);
```
