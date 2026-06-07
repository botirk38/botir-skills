# Loops

## for Loops

### Basic iteration

```zig
const items = [_]i32{ 1, 2, 3, 4, 5 };
for (items) |item| {
    std.debug.print("{}\n", .{item});
}
```

### With index

```zig
for (items, 0..) |item, idx| {
    std.debug.print("[{}] = {}\n", .{ idx, item });
}
```

### Multiple sequences (zip)

```zig
const keys = [_][]const u8{ "a", "b", "c" };
const vals = [_]i32{ 1, 2, 3 };
for (keys, vals) |key, val| {
    map.put(key, val);
}
```

### Range iteration

```zig
for (0..100) |i| {
    buffer[i] = @intCast(i);
}
```

### Pointer capture (mutation)

```zig
for (&array) |*elem| {
    elem.* *= 2;
}
```

### Labeled for

```zig
outer: for (matrix) |row| {
    for (row) |cell| {
        if (cell == target) break :outer;
    }
}
```

### inline for

Unrolls at compile time:

```zig
inline for (@typeInfo(T).@"struct".fields) |field| {
    std.debug.print("{s}\n", .{field.name});
}
```

## while Loops

### Basic while

```zig
var i: usize = 0;
while (i < items.len) : (i += 1) {
    process(items[i]);
}
```

The continue expression `(i += 1)` runs on every iteration including `continue`.

### while with optionals

Unwraps until null:

```zig
var node: ?*Node = head;
while (node) |n| : (node = n.next) {
    process(n.data);
}
```

### while with error unions

```zig
while (stream.next()) |item| {
    try process(item);
} else |err| {
    if (err != error.EndOfStream) return err;
}
```

### Labeled while

```zig
retry: while (attempts < max_attempts) : (attempts += 1) {
    const result = connect() catch {
        continue :retry;
    };
    return result;
}
```

### inline while

```zig
comptime var i = 0;
inline while (i < fields.len) : (i += 1) {
    // Body is duplicated for each value of i
}
```

## Loop Else Clauses

`for` and `while` can have `else` that runs when the loop completes without `break`:

```zig
const found = for (items) |item| {
    if (item == target) break item;
} else null;  // no break occurred
```

## break and continue

- `break` exits the innermost loop (or labeled loop)
- `break :label value` exits a labeled block/loop with a value
- `continue` skips to the next iteration
- `continue :label` continues a labeled outer loop

```zig
const result = for (items) |item| {
    if (matches(item)) break item;
} else @as(?Item, null);
```
