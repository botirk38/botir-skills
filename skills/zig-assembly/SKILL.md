---
name: zig-assembly
description: Inline and global assembly in Zig. Use when writing performance-critical code, hardware interaction, or system-level programming that requires direct machine code control.
---

# Zig Assembly

Use this skill when you need direct control over machine code generation, SIMD intrinsics unavailable through builtins, or low-level system programming.

## When to Use This Skill

- Writing syscall wrappers
- Performance-critical inner loops
- Hardware register manipulation
- Implementing CPU-specific features
- Custom calling convention bridges
- Interrupt handlers

## Inline Assembly

```zig
pub fn syscall1(number: usize, arg1: usize) usize {
    return asm volatile ("syscall"
        : [ret] "={rax}" (-> usize),
        : [number] "{rax}" (number),
          [arg1] "{rdi}" (arg1),
        : .{ .rcx = true, .r11 = true }
    );
}
```

## Syntax Structure

```
asm [volatile] ("assembly template"
    : output_constraints,
    : input_constraints,
    : clobber_list
)
```

### Output Constraints

```zig
// Single output (returned as value)
: [name] "=constraint" (-> Type)

// Store to variable
: [name] "=constraint" (variable)
```

Constraint characters:
- `={reg}` — specific register (e.g., `={rax}`, `={xmm0}`)
- `=r` — any general-purpose register
- `=m` — memory location

### Input Constraints

```zig
: [name] "{reg}" (value)   // specific register
: [name] "r" (value)       // any GPR
: [name] "i" (value)       // immediate value
: [name] "m" (value)       // memory reference
```

### Clobbers

Registers modified by the assembly that aren't outputs:

```zig
: .{ .rcx = true, .r11 = true, .memory = true, .cc = true }
```

- `.memory` — assembly reads/writes memory not listed in inputs/outputs
- `.cc` — condition codes (flags register) modified

## Examples

### Hello World (Linux x86_64)

```zig
pub fn main() noreturn {
    const msg = "hello world\n";
    _ = asm volatile ("syscall"
        : [ret] "={rax}" (-> usize),
        : [number] "{rax}" (@as(usize, 1)),  // SYS_write
          [fd] "{rdi}" (@as(usize, 1)),      // stdout
          [buf] "{rsi}" (@intFromPtr(msg.ptr)),
          [len] "{rdx}" (msg.len),
        : .{ .rcx = true, .r11 = true }
    );

    asm volatile ("syscall"
        :
        : [number] "{rax}" (@as(usize, 60)),  // SYS_exit
          [code] "{rdi}" (@as(usize, 0)),
        : .{ .rcx = true, .r11 = true }
    );
    unreachable;
}
```

### CPUID

```zig
fn cpuid(leaf: u32) struct { eax: u32, ebx: u32, ecx: u32, edx: u32 } {
    var eax: u32 = undefined;
    var ebx: u32 = undefined;
    var ecx: u32 = undefined;
    var edx: u32 = undefined;
    asm volatile ("cpuid"
        : [eax] "={eax}" (eax),
          [ebx] "={ebx}" (ebx),
          [ecx] "={ecx}" (ecx),
          [edx] "={edx}" (edx),
        : [leaf] "{eax}" (leaf),
          [subleaf] "{ecx}" (@as(u32, 0)),
    );
    return .{ .eax = eax, .ebx = ebx, .ecx = ecx, .edx = edx };
}
```

### Read Timestamp Counter

```zig
fn rdtsc() u64 {
    var low: u32 = undefined;
    var high: u32 = undefined;
    asm volatile ("rdtsc"
        : [low] "={eax}" (low),
          [high] "={edx}" (high),
    );
    return (@as(u64, high) << 32) | low;
}
```

## volatile

- Without `volatile`: compiler may reorder, duplicate, or eliminate the asm
- With `volatile`: always emitted exactly once in order (use for I/O, syscalls, timing)

## Global Assembly

For assembly that needs to define symbols visible to the linker:

```zig
comptime {
    asm (
        \\.globl my_asm_func
        \\.type my_asm_func, @function
        \\my_asm_func:
        \\    mov %rdi, %rax
        \\    ret
    );
}

extern fn my_asm_func(x: u64) u64;
```

## Naked Functions

For writing entire functions in assembly:

```zig
fn _start() callconv(.naked) noreturn {
    asm volatile (
        \\xor %ebp, %ebp
        \\mov %rsp, %rdi
        \\call main
    );
    unreachable;
}
```

No prologue/epilogue is generated. You control the entire stack frame.
