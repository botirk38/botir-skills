# Type Reflection

## @typeInfo

Returns a `std.builtin.Type` union describing any type:

```zig
const info = @typeInfo(MyStruct);
switch (info) {
    .@"struct" => |s| {
        for (s.fields) |field| {
            std.debug.print("field: {s}\n", .{field.name});
        }
    },
    else => {},
}
```

### Struct Info

```zig
const StructInfo = struct {
    fields: []const StructField,
    decls: []const Declaration,
    is_tuple: bool,
};

const StructField = struct {
    name: [:0]const u8,
    type: type,
    default_value_ptr: ?*const anyopaque,
    is_comptime: bool,
    alignment: comptime_int,
};
```

### Enum Info

```zig
const EnumInfo = struct {
    tag_type: type,
    fields: []const EnumField,
    is_exhaustive: bool,
};
```

### Union Info

```zig
const UnionInfo = struct {
    tag_type: ?type,
    fields: []const UnionField,
};
```

## @TypeOf

Returns the type of an expression:

```zig
const x: i32 = 42;
const T = @TypeOf(x);  // i32

fn identity(val: anytype) @TypeOf(val) {
    return val;
}
```

## @typeName

Returns a human-readable type name as a string:

```zig
const name = @typeName(std.ArrayList(u8));
// "array_list.ArrayListAligned(u8,null)"
```

## @hasDecl / @hasField

Check if a type has a declaration or field:

```zig
if (@hasDecl(MyType, "init")) {
    return MyType.init();
}

if (@hasField(MyStruct, "name")) {
    // field exists
}
```

## @field

Access a field by comptime-known name:

```zig
fn getField(obj: anytype, comptime name: []const u8) @TypeOf(@field(obj, name)) {
    return @field(obj, name);
}
```

## @fieldParentPtr

Get pointer to containing struct from a field pointer:

```zig
const Node = struct {
    data: i32,
    hook: Hook,
};

fn fromHook(hook: *Hook) *Node {
    return @fieldParentPtr(hook, "hook");
}
```

## Practical Examples

### Serialization

```zig
fn serialize(writer: anytype, value: anytype) !void {
    const T = @TypeOf(value);
    switch (@typeInfo(T)) {
        .int => try writer.writeInt(T, value, .little),
        .@"struct" => |info| {
            inline for (info.fields) |field| {
                try serialize(writer, @field(value, field.name));
            }
        },
        .optional => {
            if (value) |v| {
                try writer.writeByte(1);
                try serialize(writer, v);
            } else {
                try writer.writeByte(0);
            }
        },
        else => @compileError("unsupported type: " ++ @typeName(T)),
    }
}
```

### Interface Pattern (vtable)

```zig
fn Writer(comptime Context: type) type {
    return struct {
        context: Context,
        writeFn: *const fn (Context, []const u8) error{IoError}!usize,

        pub fn write(self: @This(), bytes: []const u8) !usize {
            return self.writeFn(self.context, bytes);
        }
    };
}
```

### Compile-Time String Processing

```zig
fn comptimeJoin(comptime parts: []const []const u8, comptime sep: []const u8) []const u8 {
    comptime {
        var len: usize = 0;
        for (parts, 0..) |part, i| {
            len += part.len;
            if (i < parts.len - 1) len += sep.len;
        }
        var buf: [len]u8 = undefined;
        var pos: usize = 0;
        for (parts, 0..) |part, i| {
            @memcpy(buf[pos..][0..part.len], part);
            pos += part.len;
            if (i < parts.len - 1) {
                @memcpy(buf[pos..][0..sep.len], sep);
                pos += sep.len;
            }
        }
        return &buf;
    }
}
```
