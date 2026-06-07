---
name: zig-functions
description: Function declarations, calling conventions, function pointers, and parameter passing in Zig. Use when defining APIs, working with callbacks, or understanding Zig's function semantics.
---

# Zig Functions

Use this skill when defining functions, working with function pointers, or understanding Zig's parameter passing and calling conventions.

## When to Use This Skill

- Declaring functions with various parameter types
- Using `anytype` for generic parameters
- Working with function pointers and callbacks
- Understanding calling conventions
- Using inline functions
- Function reflection at comptime

## Function Declaration

```zig
// Basic function
fn add(a: i32, b: i32) i32 {
    return a + b;
}

// Public (visible to importers)
pub fn process(data: []const u8) !void {
    // ...
}

// Function with no return value
fn logMessage(msg: []const u8) void {
    std.debug.print("{s}\n", .{msg});
}
```

## Parameter Passing

### By Value (Copy)

Primitives are always copied. Structs/unions may be passed by reference internally (compiler decides), but parameters are always immutable:

```zig
fn distance(a: Point, b: Point) f32 {
    // a and b are immutable here
    const dx = a.x - b.x;
    const dy = a.y - b.y;
    return @sqrt(dx * dx + dy * dy);
}
```

### Pointer Parameters (Mutation)

```zig
fn increment(ptr: *i32) void {
    ptr.* += 1;
}

fn fillBuffer(buf: []u8) void {
    @memset(buf, 0xFF);
}
```

## anytype (Compile-Time Duck Typing)

```zig
fn print(writer: anytype, data: []const u8) !void {
    try writer.writeAll(data);
}

// Works with any type that has .writeAll()
```

Return type can depend on parameter type:

```zig
fn double(x: anytype) @TypeOf(x) {
    return x + x;
}
```

## Function Pointers

```zig
const MathFn = *const fn (i32, i32) i32;

fn apply(f: MathFn, a: i32, b: i32) i32 {
    return f(a, b);
}

const result = apply(&add, 3, 4);  // 7
```

## Calling Conventions

```zig
// Default (Zig calling convention)
fn zigFunc() void {}

// C calling convention (for FFI)
export fn cFunc(x: c_int) c_int {
    return x + 1;
}

// Naked (no prologue/epilogue — for assembly)
fn nakedFunc() callconv(.naked) noreturn {
    asm volatile ("syscall");
    unreachable;
}

// Windows API convention
extern "kernel32" fn ExitProcess(code: u32) callconv(.winapi) noreturn;
```

## Inline Functions

Forced inlining (compile error if not possible):

```zig
inline fn fastAdd(a: u32, b: u32) u32 {
    return a + b;
}
```

Inline functions can propagate comptime-known arguments:

```zig
inline fn isZero(x: anytype) bool {
    return x == 0;
}
// If x is comptime-known, the branch is eliminated
```

## @branchHint

Optimization hint for unlikely/cold paths:

```zig
fn handleError(err: Error) noreturn {
    @branchHint(.cold);
    std.log.err("fatal: {}", .{err});
    std.process.exit(1);
}
```

## Function Reflection

```zig
const info = @typeInfo(@TypeOf(myFunc)).@"fn";
info.params.len;        // number of parameters
info.return_type;       // return type (or null if generic)
info.is_var_args;       // C variadic?
info.calling_convention; // .auto, .c, .naked, etc.
```

## Methods (Self Parameter)

Methods are just functions in a struct namespace:

```zig
const Counter = struct {
    count: u32 = 0,

    pub fn increment(self: *Counter) void {
        self.count += 1;
    }

    pub fn value(self: Counter) u32 {
        return self.count;
    }

    pub fn reset(self: *Counter) void {
        self.count = 0;
    }
};

var c = Counter{};
c.increment();       // dot-call syntax
Counter.increment(&c); // equivalent explicit call
```

## Extern Functions

Link against external libraries:

```zig
extern "c" fn printf(fmt: [*:0]const u8, ...) c_int;
extern "c" fn malloc(size: usize) ?*anyopaque;
```
