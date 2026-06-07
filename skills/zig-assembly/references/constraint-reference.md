# Assembly Constraint Reference

## Output Constraints

| Prefix | Meaning |
|--------|---------|
| `=` | Write-only output |
| `+` | Read-write (also input) |

| Constraint | Description |
|------------|-------------|
| `={rax}` | Specific named register |
| `=r` | Any general-purpose register |
| `=m` | Memory operand |
| `=x` | Any SSE register |

## Input Constraints

| Constraint | Description |
|------------|-------------|
| `{rax}` | Specific named register |
| `r` | Any general-purpose register |
| `m` | Memory operand |
| `i` | Immediate integer |
| `x` | Any SSE register |
| `0`, `1`... | Same register as output N |

## Clobber Flags

| Clobber | Meaning |
|---------|---------|
| `.memory` | Assembly accesses memory not in constraints |
| `.cc` | Modifies condition flags |
| `.{reg}` | Specific register is clobbered |

## x86_64 Register Names

### General Purpose

```
rax, rbx, rcx, rdx       (64-bit)
eax, ebx, ecx, edx       (32-bit)
ax, bx, cx, dx           (16-bit)
al, bl, cl, dl           (8-bit low)
ah, bh, ch, dh           (8-bit high)
rsi, rdi, rsp, rbp       (64-bit)
r8-r15                   (64-bit)
```

### SSE/AVX

```
xmm0-xmm15    (128-bit SSE)
ymm0-ymm15    (256-bit AVX)
zmm0-zmm31    (512-bit AVX-512)
```

## Linux Syscall Convention (x86_64)

| Register | Purpose |
|----------|---------|
| `rax` | Syscall number |
| `rdi` | Arg 1 |
| `rsi` | Arg 2 |
| `rdx` | Arg 3 |
| `r10` | Arg 4 |
| `r8` | Arg 5 |
| `r9` | Arg 6 |
| `rax` | Return value |
| `rcx`, `r11` | Clobbered by kernel |

## ARM64 (AArch64) Register Names

```
x0-x30    (64-bit general purpose)
w0-w30    (32-bit, lower half of xN)
sp        (stack pointer)
lr (x30)  (link register)
```

### ARM64 Syscall Convention

| Register | Purpose |
|----------|---------|
| `x8` | Syscall number |
| `x0`-`x5` | Arguments |
| `x0` | Return value |

## Template Syntax

Assembly templates use `%` for operand substitution:

```zig
asm ("add %[b], %[a]"
    : [a] "+r" (value),
    : [b] "r" (addend),
);
```

Literal `%` is written as `%%`.

## Multi-Line Templates

Use Zig multiline string literals:

```zig
asm volatile (
    \\push %rbp
    \\mov %rsp, %rbp
    \\sub $16, %rsp
    \\call some_function
    \\add $16, %rsp
    \\pop %rbp
);
```
