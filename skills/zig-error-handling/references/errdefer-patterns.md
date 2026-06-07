# errdefer Patterns

## Basic errdefer

Runs cleanup only when the enclosing function returns an error:

```zig
fn allocateAndInit(allocator: Allocator) !*Widget {
    const widget = try allocator.create(Widget);
    errdefer allocator.destroy(widget);

    widget.* = try Widget.init();
    return widget;
}
```

If `Widget.init()` fails, `allocator.destroy(widget)` runs.
If everything succeeds, `errdefer` does NOT run (caller owns the widget).

## errdefer vs defer

```zig
fn example(allocator: Allocator) !*Resource {
    const file = try openFile();
    defer file.close();  // ALWAYS closes file (success or error)

    const buf = try allocator.alloc(u8, 1024);
    errdefer allocator.free(buf);  // ONLY frees on error

    try fillBuffer(file, buf);
    return makeResource(buf);  // caller now owns buf
}
```

Rule of thumb:
- `defer`: cleanup resources this function owns permanently
- `errdefer`: cleanup resources that will be transferred to caller on success

## errdefer with Captures

`errdefer` can capture the error value:

```zig
fn process() !void {
    errdefer |err| {
        std.log.err("process failed: {s}", .{@errorName(err)});
    };

    try stepOne();
    try stepTwo();
    try stepThree();
}
```

## Multi-Resource Initialization

```zig
fn initSystem(allocator: Allocator) !System {
    const db = try Database.open(allocator);
    errdefer db.close();

    const cache = try Cache.init(allocator);
    errdefer cache.deinit();

    const server = try Server.bind(allocator, db, cache);
    errdefer server.shutdown();

    return System{ .db = db, .cache = cache, .server = server };
}
```

Each `errdefer` protects against failures in subsequent steps. On success, the caller takes ownership of all three resources.

## errdefer Ordering

`errdefer`s execute in reverse declaration order (LIFO), same as `defer`:

```zig
fn example() !void {
    errdefer std.debug.print("3\n", .{});
    errdefer std.debug.print("2\n", .{});
    errdefer std.debug.print("1\n", .{});
    return error.Fail;
}
// Prints: 1, 2, 3
```

## Common Pattern: Init/Deinit with errdefer

```zig
const Connection = struct {
    socket: Socket,
    buffer: []u8,
    allocator: Allocator,

    pub fn init(allocator: Allocator, addr: Address) !Connection {
        const socket = try Socket.connect(addr);
        errdefer socket.close();

        const buffer = try allocator.alloc(u8, 4096);
        errdefer allocator.free(buffer);

        try socket.handshake();

        return .{
            .socket = socket,
            .buffer = buffer,
            .allocator = allocator,
        };
    }

    pub fn deinit(self: *Connection) void {
        self.allocator.free(self.buffer);
        self.socket.close();
    }
};
```

## Avoid errdefer in Loops

`errdefer` in a loop body can be tricky — it only protects one iteration:

```zig
// CAREFUL: each errdefer only covers its iteration
fn initAll(allocator: Allocator, n: usize) ![]Widget {
    const widgets = try allocator.alloc(Widget, n);
    errdefer allocator.free(widgets);

    var initialized: usize = 0;
    errdefer for (widgets[0..initialized]) |*w| w.deinit();

    for (widgets) |*w| {
        w.* = try Widget.init();
        initialized += 1;
    }

    return widgets;
}
```
