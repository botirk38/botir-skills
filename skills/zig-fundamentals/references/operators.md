# Operators

## Arithmetic Operators

| Operator | Description | Overflow behavior |
|----------|-------------|-------------------|
| `a + b` | Addition | Illegal in safe mode |
| `a - b` | Subtraction | Illegal in safe mode |
| `a * b` | Multiplication | Illegal in safe mode |
| `a / b` | Division | Illegal if divisor is 0 |
| `a % b` | Remainder | Illegal if divisor is 0 |
| `-a` | Negation | Illegal if `a` is min value |

### Wrapping variants (defined overflow)

| Operator | Description |
|----------|-------------|
| `a +% b` | Wrapping addition |
| `a -% b` | Wrapping subtraction |
| `a *% b` | Wrapping multiplication |

### Saturating variants (clamps at min/max)

| Operator | Description |
|----------|-------------|
| `a +\| b` | Saturating addition |
| `a -\| b` | Saturating subtraction |
| `a *\| b` | Saturating multiplication |

## Bit Manipulation

```zig
const a: u8 = 0b1100;
const b: u8 = 0b1010;

const and_result = a & b;    // 0b1000
const or_result = a | b;     // 0b1110
const xor_result = a ^ b;    // 0b0110
const not_result = ~a;       // 0b11110011

const shifted = a << 2;      // 0b110000
const right = a >> 1;        // 0b0110
```

## Comparison

All comparison operators return `bool`:

```zig
x == y    // equal
x != y    // not equal
x < y     // less than
x > y     // greater than
x <= y    // less or equal
x >= y    // greater or equal
```

## Boolean Operators

```zig
const result = (a > 0) and (b > 0);   // short-circuit AND
const either = (a > 0) or (b > 0);    // short-circuit OR
const negated = !condition;             // NOT
```

## Optional/Error Operators

```zig
// orelse — unwrap optional with fallback
const val = optional_value orelse 0;

// catch — unwrap error union with fallback
const val = might_fail() catch |err| handle(err);

// try — propagate error to caller
const val = try might_fail();
```

## Pointer Operators

```zig
const ptr = &x;      // address-of
const val = ptr.*;   // dereference
```

## Precedence (high to low)

1. Grouping: `()`
2. Postfix: `.`, `.*`, `[i]`, function calls
3. Prefix: `!`, `-`, `~`, `&`, `@builtins`
4. Multiplication: `*`, `/`, `%`
5. Addition: `+`, `-`
6. Bit shifts: `<<`, `>>`
7. Bitwise: `&`, `|`, `^`
8. Comparison: `==`, `!=`, `<`, `>`, `<=`, `>=`
9. Boolean: `and`, `or`
10. Assignment: `=`, `+=`, etc.

When in doubt, use explicit parentheses.
