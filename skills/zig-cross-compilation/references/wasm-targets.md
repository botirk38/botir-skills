# WebAssembly Targets

## Target Variants

| Target | Runtime | Use Case |
|--------|---------|----------|
| `wasm32-wasi` | WASI runtimes (wasmtime, wasmer) | CLI tools, server-side |
| `wasm32-freestanding` | Browser / custom host | Web apps, plugins |
| `wasm64-wasi` | 64-bit address space | Large memory apps |

## WASI Target

Full standard library support (file I/O, time, random):

```zig
const std = @import("std");

pub fn main() !void {
    const file = try std.fs.cwd().createFile("output.txt", .{});
    defer file.close();
    try file.writer().writeAll("Hello from WASI!");
}
```

Build and run:
```sh
zig build-exe main.zig -target wasm32-wasi
wasmtime main.wasm --dir .
```

## Freestanding WASM (Browser)

No std library. Export functions for the host:

```zig
// lib.zig
export fn init() void {
    // Called by JavaScript on load
}

export fn update(dt: f32) void {
    // Game loop tick
    _ = dt;
}

export fn alloc(len: usize) [*]u8 {
    // Simple bump allocator for host communication
    const slice = buffer[offset .. offset + len];
    offset += len;
    return slice.ptr;
}

var buffer: [65536]u8 = undefined;
var offset: usize = 0;
```

Build:
```sh
zig build-lib lib.zig -target wasm32-freestanding -O ReleaseSmall --export-memory
```

## Importing Host Functions

```zig
// Declare functions provided by the host (JavaScript)
extern "env" fn consoleLog(ptr: [*]const u8, len: usize) void;
extern "env" fn getTime() f64;

export fn run() void {
    const msg = "Hello from Zig WASM!";
    consoleLog(msg.ptr, msg.len);
}
```

JavaScript host:
```javascript
const importObject = {
    env: {
        consoleLog(ptr, len) {
            const bytes = new Uint8Array(instance.exports.memory.buffer, ptr, len);
            console.log(new TextDecoder().decode(bytes));
        },
        getTime() { return performance.now(); },
    },
};
const { instance } = await WebAssembly.instantiate(wasmModule, importObject);
instance.exports.run();
```

## Memory Management in WASM

WASM has linear memory. Zig can manage it:

```zig
// Use page_allocator (maps to memory.grow)
const allocator = std.heap.page_allocator;

// Or fixed buffer for simple cases
var buf: [64 * 1024]u8 = undefined;
var fba = std.heap.FixedBufferAllocator.init(&buf);
```

## Optimization for WASM

```sh
# Minimize binary size
zig build-lib lib.zig -target wasm32-freestanding \
    -O ReleaseSmall \
    -fstrip \
    -fsingle-threaded \
    --export-memory

# Further reduce with wasm-opt (from binaryen)
wasm-opt -Oz lib.wasm -o lib.opt.wasm
```

## WASM Features

Enable/disable CPU features:

```zig
// build.zig
const exe = b.addExecutable(.{
    .name = "app",
    .root_module = b.createModule(.{
        .root_source_file = b.path("src/main.zig"),
        .target = b.resolveTargetQuery(.{
            .cpu_arch = .wasm32,
            .os_tag = .freestanding,
            .cpu_features_add = .{ .simd128 = true, .bulk_memory = true },
        }),
    }),
});
```

## Testing WASM Builds

```sh
# Run WASI tests
zig test src/lib.zig -target wasm32-wasi

# Use wasmtime integration
zig build test -Dtarget=wasm32-wasi -fwasmtime
```
