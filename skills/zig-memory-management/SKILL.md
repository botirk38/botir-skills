---
name: zig-memory-management
description: Memory management in Zig including allocators, pointers, slices, alignment, and ownership patterns. Use when allocating memory, working with pointers/slices, or choosing the right allocator.
---

# Zig Memory Management

Use this skill when allocating memory, choosing allocators, working with pointers and slices, or understanding Zig's ownership model.

## When to Use This Skill

- Choosing an allocator for your use case
- Working with pointers (single-item and many-item)
- Using slices for dynamic-length views
- Understanding alignment requirements
- Implementing custom allocators
- Managing lifetime and ownership

## Core Philosophy

Zig has no garbage collector, no hidden allocations, and no default allocator. Functions that need to allocate memory accept an `Allocator` parameter, making allocation explicit and configurable.

## Choosing an Allocator

| Use Case | Allocator |
|-----------|-----------|
| Library code | Accept `Allocator` parameter |
| Linking libc | `std.heap.c_allocator` |
| Known max size | `std.heap.FixedBufferAllocator` |
| CLI tool (free-all-at-end) | `std.heap.ArenaAllocator` |
| Cyclic patterns (game loop, web request) | `std.heap.ArenaAllocator` |
| Testing | `std.testing.allocator` (detects leaks) |
| General purpose | `std.heap.DebugAllocator` |
| Page-level | `std.heap.page_allocator` |

## Basic Allocation

```zig
const allocator = std.heap.page_allocator;

// Allocate a slice
const buf = try allocator.alloc(u8, 1024);
defer allocator.free(buf);

// Allocate a single item
const ptr = try allocator.create(MyStruct);
defer allocator.destroy(ptr);
```

## Arena Allocator

Free everything at once — no individual frees needed:

```zig
var arena = std.heap.ArenaAllocator.init(std.heap.page_allocator);
defer arena.deinit();

const allocator = arena.allocator();
const data = try allocator.alloc(u8, 4096);
// No need to free `data` — arena.deinit() frees all
```

## Pointers

### Single-Item Pointer: `*T`

```zig
var x: i32 = 42;
const ptr: *i32 = &x;
ptr.* = 100;            // dereference and assign
```

### Many-Item Pointer: `[*]T`

```zig
const many: [*]const u8 = slice.ptr;
const byte = many[5];   // index access
```

### Pointer Arithmetic

Only many-item pointers support arithmetic:
```zig
const next = many_ptr + 1;
const prev = many_ptr - 1;
```

## Slices

A slice is a fat pointer: `{ ptr: [*]T, len: usize }`.

```zig
var array = [_]u8{ 1, 2, 3, 4, 5 };
const slice: []u8 = array[1..4];  // [2, 3, 4]
slice.len;   // 3
slice.ptr;   // pointer to element at index 1
```

### Sentinel-Terminated

```zig
const c_str: [:0]const u8 = "hello";  // null-terminated slice
const arr: [5:0]u8 = .{ 1, 2, 3, 4, 5 };  // sentinel-terminated array
```

## Alignment

```zig
const aligned_ptr: *align(16) u8 = @alignCast(ptr);

// Specify alignment in allocations
const buf = try allocator.alignedAlloc(u8, 64, 4096);
```

Every pointer type has an alignment. Misaligned access is illegal behavior.

## Ownership Patterns

1. **Caller owns**: function returns allocated memory, caller must free
2. **Callee owns**: function takes ownership of passed allocation
3. **Borrowed**: function borrows a slice/pointer, doesn't free

```zig
// Caller owns the result
fn concat(allocator: Allocator, a: []const u8, b: []const u8) ![]u8 {
    const result = try allocator.alloc(u8, a.len + b.len);
    @memcpy(result[0..a.len], a);
    @memcpy(result[a.len..], b);
    return result;
}
```

## Where Are the Bytes?

| Data | Location |
|------|----------|
| String literals | Global constant (read-only) |
| `comptime var` | Doesn't exist at runtime |
| Local `var` | Stack |
| `allocator.alloc()` | Heap |
| Thread-local | Thread-local storage |
| `@import` result | Compile-time |
