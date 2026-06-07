# Linking Libraries

## System Libraries

```zig
// build.zig
exe.linkLibC();                        // link libc
exe.linkSystemLibrary("pthread");      // -lpthread
exe.linkSystemLibrary("ssl");          // -lssl
exe.linkSystemLibrary("crypto");       // -lcrypto
```

## Static vs Dynamic Linking

```zig
// Prefer static linking (portable binary)
exe.linkSystemLibrary2("z", .{ .preferred_link_mode = .static });

// Force dynamic
exe.linkSystemLibrary2("dl", .{ .preferred_link_mode = .dynamic });
```

## Compiling C Sources

```zig
exe.addCSourceFiles(.{
    .files = &.{
        "src/impl.c",
        "src/utils.c",
    },
    .flags = &.{
        "-std=c11",
        "-Wall",
        "-Wextra",
        "-O2",
        "-fno-strict-aliasing",
    },
});

// Add include directories
exe.addIncludePath(b.path("include"));
exe.addIncludePath(b.path("vendor/include"));
```

## Compiling C++ Sources

```zig
exe.addCSourceFiles(.{
    .files = &.{ "src/engine.cpp" },
    .flags = &.{ "-std=c++17", "-fno-exceptions" },
});
exe.linkLibCpp();  // link libc++ or libstdc++
```

## Vendored Libraries

For libraries checked into your repo:

```zig
const lib = b.addStaticLibrary(.{
    .name = "vendor-lib",
    .target = target,
    .optimize = optimize,
});
lib.addCSourceFiles(.{
    .root = b.path("vendor/lib"),
    .files = &.{ "src/core.c", "src/util.c" },
    .flags = &.{ "-DLIB_STATIC" },
});
lib.addIncludePath(b.path("vendor/lib/include"));
lib.linkLibC();

exe.linkLibrary(lib);
exe.addIncludePath(b.path("vendor/lib/include"));
```

## pkg-config Integration

```zig
// Automatically find include paths and link flags
exe.linkSystemLibrary("gtk4");
// Zig's build system uses pkg-config when available
```

## Framework Linking (macOS)

```zig
if (target.result.os.tag == .macos) {
    exe.linkFramework("CoreFoundation");
    exe.linkFramework("Security");
}
```

## Building a C Library from Zig

Export Zig functions and build as shared/static library:

```zig
// src/lib.zig
export fn my_init() c_int { return 0; }
export fn my_process(data: [*]const u8, len: usize) c_int {
    const slice = data[0..len];
    _ = slice;
    return 0;
}

// build.zig
const lib = b.addSharedLibrary(.{
    .name = "mylib",
    .root_module = b.createModule(.{
        .root_source_file = b.path("src/lib.zig"),
        .target = target,
        .optimize = optimize,
    }),
});
lib.linkLibC();
b.installArtifact(lib);

// Also install the header
b.installFile("include/mylib.h", "include/mylib.h");
```

## Object File Mixing

Link pre-compiled object files:

```zig
exe.addObjectFile(b.path("prebuilt/helper.o"));
exe.addObjectFile(.{ .cwd_relative = "/usr/lib/libspecial.a" });
```
