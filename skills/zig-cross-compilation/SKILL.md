---
name: zig-cross-compilation
description: Cross-compilation, build modes, and target platforms in Zig including WebAssembly, freestanding, and WASI. Use when building for different architectures or optimizing release binaries.
---

# Zig Cross-Compilation

Use this skill when building for different targets, creating release builds, or targeting WebAssembly/embedded platforms.

## When to Use This Skill

- Cross-compiling for different architectures (ARM, RISC-V, x86)
- Targeting WebAssembly (WASM/WASI)
- Building freestanding binaries (no OS)
- Choosing build modes (Debug, ReleaseFast, ReleaseSafe, ReleaseSmall)
- Creating portable static binaries
- Targeting embedded systems

## Cross-Compilation Basics

Zig is a cross-compiler out of the box — no external toolchains needed:

```sh
# Build for Linux ARM64
zig build-exe main.zig -target aarch64-linux-gnu

# Build for Windows x86_64
zig build-exe main.zig -target x86_64-windows-msvc

# Build for macOS ARM (Apple Silicon)
zig build-exe main.zig -target aarch64-macos

# Build for RISC-V 64
zig build-exe main.zig -target riscv64-linux-gnu
```

## Build Modes

| Mode | Optimizations | Safety Checks | Debug Info | Binary Size |
|------|--------------|---------------|------------|-------------|
| Debug (default) | None | Full | Full | Large |
| ReleaseSafe | Yes | Full | Minimal | Medium |
| ReleaseFast | Max | None | None | Medium |
| ReleaseSmall | Size-focused | None | None | Small |

```sh
zig build-exe main.zig -O ReleaseFast
zig build-exe main.zig -O ReleaseSmall
zig build-exe main.zig -O ReleaseSafe
```

### In build.zig

```zig
pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    const exe = b.addExecutable(.{
        .name = "app",
        .root_module = b.createModule(.{
            .root_source_file = b.path("src/main.zig"),
            .target = target,
            .optimize = optimize,
        }),
    });
    b.installArtifact(exe);
}
```

Usage:
```sh
zig build -Dtarget=aarch64-linux-gnu -Doptimize=ReleaseFast
```

## WebAssembly

### WASI (WebAssembly System Interface)

```zig
// src/main.zig
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    try stdout.writeAll("Hello from WASM!\n");
}
```

Build:
```sh
zig build-exe src/main.zig -target wasm32-wasi
# Produces main.wasm
wasmtime main.wasm  # run with a WASI runtime
```

### Browser WebAssembly (freestanding)

```zig
// Export functions for JavaScript:
export fn add(a: i32, b: i32) i32 {
    return a + b;
}

export fn fibonacci(n: u32) u32 {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

Build:
```sh
zig build-lib lib.zig -target wasm32-freestanding -O ReleaseSmall
```

JavaScript usage:
```javascript
const { instance } = await WebAssembly.instantiateStreaming(fetch("lib.wasm"));
console.log(instance.exports.add(2, 3));  // 5
```

## Freestanding (No OS)

For bare-metal/embedded targets:

```zig
const builtin = @import("builtin");

// No std available in freestanding
pub fn _start() noreturn {
    // Direct hardware interaction
    const uart: *volatile u8 = @ptrFromInt(0x1000_0000);
    uart.* = 'H';
    uart.* = 'i';
    while (true) {}
}

// Must provide panic handler
pub const panic = std.debug.SimplePanic(.{});
// Or manually:
// pub fn panic(msg: []const u8, _: ?*std.builtin.StackTrace, _: ?usize) noreturn {
//     while (true) {}
// }
```

Build:
```sh
zig build-exe boot.zig -target arm-freestanding-none
```

## Static Binaries

Create fully static executables (no dynamic linker needed):

```sh
# Linux static binary (uses musl libc)
zig build-exe main.zig -target x86_64-linux-musl -O ReleaseSmall -fstrip

# Size optimization
zig build-exe main.zig -O ReleaseSmall -fstrip -fsingle-threaded
```

## Multi-Target Release

Build for multiple targets in build.zig:

```zig
pub fn build(b: *std.Build) void {
    const targets = [_]std.Target.Query{
        .{ .cpu_arch = .x86_64, .os_tag = .linux, .abi = .musl },
        .{ .cpu_arch = .aarch64, .os_tag = .linux, .abi = .musl },
        .{ .cpu_arch = .x86_64, .os_tag = .windows },
        .{ .cpu_arch = .aarch64, .os_tag = .macos },
    };

    for (targets) |t| {
        const exe = b.addExecutable(.{
            .name = "app",
            .root_module = b.createModule(.{
                .root_source_file = b.path("src/main.zig"),
                .target = b.resolveTargetQuery(t),
                .optimize = .ReleaseSafe,
            }),
        });
        b.installArtifact(exe);
    }
}
```

## Conditional Compilation

```zig
const builtin = @import("builtin");

const is_linux = builtin.os.tag == .linux;
const is_wasm = builtin.cpu.arch == .wasm32;
const is_debug = builtin.mode == .Debug;

// Platform-specific implementation
const impl = if (is_linux)
    @import("linux_impl.zig")
else if (builtin.os.tag == .windows)
    @import("windows_impl.zig")
else
    @import("generic_impl.zig");
```

## Target Query

Available target components:
- **CPU arch**: `x86_64`, `aarch64`, `arm`, `riscv64`, `wasm32`, `mips`
- **OS**: `linux`, `windows`, `macos`, `freebsd`, `freestanding`
- **ABI**: `gnu`, `musl`, `msvc`, `none`, `eabi`

List all supported targets:
```sh
zig targets
```
