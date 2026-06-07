# Build Modes

## Debug (Default)

- No optimizations
- Full runtime safety checks (bounds, overflow, null)
- Full debug info and stack traces
- Fast compilation

```sh
zig build-exe main.zig  # Debug is default
```

Best for: development, testing, debugging.

## ReleaseSafe

- Full optimizations
- Runtime safety checks KEPT (bounds, overflow)
- Minimal debug info
- Crashes instead of undefined behavior

```sh
zig build-exe main.zig -O ReleaseSafe
```

Best for: production servers, security-critical code.

## ReleaseFast

- Maximum speed optimizations
- No runtime safety checks
- No debug info
- Undefined behavior on violations (like C)

```sh
zig build-exe main.zig -O ReleaseFast
```

Best for: performance-critical applications, games, HPC.

## ReleaseSmall

- Size-focused optimizations
- No runtime safety checks
- No debug info
- Smallest binary size

```sh
zig build-exe main.zig -O ReleaseSmall -fstrip -fsingle-threaded
```

Best for: embedded systems, WebAssembly, size-constrained environments.

## Per-Scope Safety Override

Override safety for specific code blocks:

```zig
fn hotPath(data: []u8) void {
    @setRuntimeSafety(false);  // disable safety in this function
    // Unchecked operations for performance
    for (data) |*byte| {
        byte.* +%= 1;  // wrapping add (no check anyway)
    }
}

fn safePath(data: []u8) void {
    // Default safety based on build mode
    for (data) |*byte| {
        byte.* += 1;  // checked in ReleaseSafe, unchecked in ReleaseFast
    }
}
```

## Checking Build Mode at Comptime

```zig
const builtin = @import("builtin");

const is_debug = builtin.mode == .Debug;
const is_safe = builtin.mode == .Debug or builtin.mode == .ReleaseSafe;

fn log(msg: []const u8) void {
    if (is_debug) {
        std.debug.print("{s}\n", .{msg});
    }
}
```

## Mixed Mode (Build System)

Different optimization per compilation unit:

```zig
pub fn build(b: *std.Build) void {
    // Main app in ReleaseSafe
    const exe = b.addExecutable(.{
        .name = "app",
        .root_module = b.createModule(.{
            .root_source_file = b.path("src/main.zig"),
            .target = target,
            .optimize = .ReleaseSafe,
        }),
    });

    // Hot inner loop as a separate module in ReleaseFast
    const simd_mod = b.createModule(.{
        .root_source_file = b.path("src/simd_kernel.zig"),
        .target = target,
        .optimize = .ReleaseFast,
    });
    exe.root_module.addImport("simd_kernel", simd_mod);
}
```

## Additional Flags

```sh
-fstrip          # Strip debug symbols from binary
-fsingle-threaded  # Disable threading support (smaller binary)
-fno-llvm        # Use Zig's self-hosted backend (faster compile, less optimal code)
-femit-asm       # Emit assembly alongside object
```

## Comparing Binary Sizes (x86_64-linux, hello world)

| Mode | Flags | Size |
|------|-------|------|
| Debug | (none) | ~2.7 MB |
| ReleaseSafe | (none) | ~180 KB |
| ReleaseFast | `-fstrip` | ~8 KB |
| ReleaseSmall | `-fstrip -fsingle-threaded` | ~5 KB |
