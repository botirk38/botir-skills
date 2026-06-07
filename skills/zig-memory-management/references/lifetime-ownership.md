# Lifetime and Ownership

## Ownership Rules

Zig has no borrow checker, but follows clear conventions:

1. **If a function allocates, it documents who must free**
2. **Slices don't own their data** — they're borrowed views
3. **defer/errdefer ensure cleanup** on all paths

## Common Patterns

### Caller-Owns-Result

```zig
/// Caller must free the returned slice with `allocator.free()`.
pub fn readFile(allocator: Allocator, path: []const u8) ![]u8 {
    const file = try std.fs.cwd().openFile(path, .{});
    defer file.close();
    return file.readToEndAlloc(allocator, std.math.maxInt(usize));
}

// Usage:
const data = try readFile(allocator, "config.json");
defer allocator.free(data);
```

### Init/Deinit Pattern

```zig
const ArrayList = struct {
    items: []u8,
    allocator: Allocator,

    pub fn init(allocator: Allocator) ArrayList {
        return .{ .items = &.{}, .allocator = allocator };
    }

    pub fn deinit(self: *ArrayList) void {
        if (self.items.len > 0) {
            self.allocator.free(self.items);
        }
        self.* = undefined;  // poison after free
    }
};
```

### Arena for Temporary Allocations

```zig
fn processRequest(permanent: Allocator, request: Request) !Response {
    var arena = std.heap.ArenaAllocator.init(permanent);
    defer arena.deinit();
    const tmp = arena.allocator();

    // All temp allocations freed at end of function
    const parsed = try parse(tmp, request.body);
    const result = try transform(tmp, parsed);
    
    // Only the response is allocated with the permanent allocator
    return try Response.init(permanent, result);
}
```

## Stack vs Heap

### Stack (automatic lifetime)

```zig
fn example() void {
    var buf: [1024]u8 = undefined;  // stack-allocated
    // Automatically freed when function returns
    // WARNING: don't return pointers to stack memory!
}
```

### Heap (manual lifetime)

```zig
fn example(allocator: Allocator) !*Node {
    const node = try allocator.create(Node);
    // Lives until explicitly destroyed
    return node;
}
```

## Dangling Pointer Prevention

Zig helps prevent dangling pointers through:

1. **No returning pointers to locals** — compile error
2. **Slices from stack arrays** — compile error if escaping scope
3. **Use-after-free detection** — DebugAllocator fills freed memory with `0xAA`

```zig
fn bad() *i32 {
    var x: i32 = 42;
    return &x;  // COMPILE ERROR: pointer to local
}
```

## @memcpy and @memset

```zig
// Copy memory
@memcpy(dest[0..src.len], src);

// Fill memory
@memset(buf, 0);           // zero-fill
@memset(buf, undefined);   // mark as undefined (debug aid)
```

## Sentinel Values and Null

```zig
// Optional pointers are same size as regular pointers
// null uses address 0 (which is normally illegal)
var ptr: ?*Node = null;
ptr = &some_node;

if (ptr) |p| {
    // p is *Node (guaranteed non-null)
    use(p);
}
```
