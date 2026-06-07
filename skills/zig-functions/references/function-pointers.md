# Function Pointers

## Declaring Function Pointer Types

```zig
// Pointer to a function taking two i32 and returning i32
const BinaryOp = *const fn (i32, i32) i32;

// Pointer to a function that can fail
const Callback = *const fn ([]const u8) anyerror!void;

// Mutable function pointer
var handler: *const fn () void = &defaultHandler;
```

## Obtaining Function Pointers

```zig
fn add(a: i32, b: i32) i32 { return a + b; }
fn sub(a: i32, b: i32) i32 { return a - b; }

const op: BinaryOp = &add;
const result = op(3, 4);  // 7
```

## Function Pointer Tables (vtables)

```zig
const VTable = struct {
    read: *const fn (*anyopaque, []u8) usize,
    write: *const fn (*anyopaque, []const u8) usize,
    close: *const fn (*anyopaque) void,
};

const Stream = struct {
    vtable: *const VTable,
    context: *anyopaque,

    pub fn read(self: Stream, buf: []u8) usize {
        return self.vtable.read(self.context, buf);
    }

    pub fn write(self: Stream, data: []const u8) usize {
        return self.vtable.write(self.context, data);
    }
};
```

## Callbacks

```zig
fn sort(items: []i32, lessThan: *const fn (i32, i32) bool) void {
    // Use lessThan for comparisons
    for (items[0 .. items.len - 1], 0..) |_, i| {
        for (items[i + 1 ..]) |_, j| {
            if (lessThan(items[i + j + 1], items[i])) {
                std.mem.swap(i32, &items[i], &items[i + j + 1]);
            }
        }
    }
}

fn ascending(a: i32, b: i32) bool { return a < b; }
fn descending(a: i32, b: i32) bool { return a > b; }

sort(&data, &ascending);
sort(&data, &descending);
```

## Function Pointers vs anytype

| Feature | Function Pointer | anytype |
|---------|-----------------|---------|
| Runtime dispatch | Yes | No (comptime) |
| Stored in data structures | Yes | No |
| Performance | Indirect call | Inlined |
| Type erasure | Yes | No |

Use function pointers when:
- You need runtime polymorphism
- Storing callbacks in structs
- Building plugin systems

Use `anytype` when:
- You want zero-cost abstraction
- The concrete type is known at compile time
- Maximum performance matters

## Calling Convention Compatibility

Function pointers carry their calling convention:

```zig
// C-compatible function pointer (for passing to C code)
const CCallback = *const fn (c_int) callconv(.c) void;

extern "c" fn register_callback(cb: CCallback) void;
```

## Null Function Pointers

Use optional function pointers for nullable callbacks:

```zig
const OptionalFn = ?*const fn () void;

var callback: OptionalFn = null;
callback = &myHandler;

if (callback) |cb| {
    cb();
}
```
