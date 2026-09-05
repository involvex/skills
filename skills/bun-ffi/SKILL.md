---
name: bun-ffi
description: High-performance foreign function interface with Bun's built-in FFI. Use when calling C libraries from TypeScript, binding native functions, or working with dlopen, C strings, and pointers. 3× faster than previous Bun versions and other JS FFI solutions.
compatibility: Requires Bun 1.4+
allowed-tools: ["Bash", "Write", "Read"]
metadata:
  author: dale
  category: bun-runtime
  tags: "bun, ffi, dlopen, native, c, pointers, performance"
---

# Bun FFI — Foreign Function Interface

Call C libraries directly from TypeScript with Bun's built-in FFI. 3× faster than previous Bun versions, with native JSC FFI replacing TinyCC.

## Quick Reference

| Task | Pattern |
|---|---|
| Load library | `dlopen("libname.so", symbols)` |
| Call function | `lib.fn(args)` |
| C string return | `returns: "cstring"` |
| Buffer + length | `args: ["buffer", "buffer_length"]` |
| By-value struct | Pack into `bigint`, use `u64` |

## Workflow

### 1. Basic FFI Setup

```typescript
import { dlopen } from "bun:ffi";

const { symbols } = dlopen("libc.so.6", {
  strlen: {
    args: ["ptr"],
    returns: "u32",
  },
  strcpy: {
    args: ["ptr", "ptr"],
    returns: "ptr",
  },
});

const length = symbols.strlen(cStringPtr);
```

### 2. FFI Types

| FFI Type | C Type | TypeScript |
|---|---|---|
| `"ptr"` | `void*`, `char*`, struct pointers | `Pointer` |
| `"u8"` | `uint8_t`, `BYTE` | `number` |
| `"u16"` | `uint16_t`, `WORD` | `number` |
| `"u32"` | `uint32_t`, `DWORD` | `number` |
| `"u64"` | `uint64_t`, handles | `bigint` |
| `"i8"` | `int8_t` | `number` |
| `"i16"` | `int16_t`, `SHORT` | `number` |
| `"i32"` | `int32_t`, `INT` | `number` |
| `"i64"` | `int64_t` | `bigint` |
| `"f32"` | `float` | `number` |
| `"f64"` | `double` | `number` |
| `"bool"` | `bool` | `boolean` |
| `"cstring"` | `char*` (null-terminated) | `string` |
| `"void"` | `void` | `void` |

### 3. Buffer Length Pattern (Bun 1.4+)

Pass a TypedArray's length alongside its pointer so they can't disagree:

```typescript
const { symbols } = dlopen("libhash.so", {
  hash: { args: ["buffer", "buffer_length"], returns: "cstring" },
});

const data = new Uint8Array([1, 2, 3, 4]);
const digest = symbols.hash(data, data);
typeof digest; // "string"
```

### 4. C String Returns

```typescript
const { symbols } = dlopen("libc.so.6", {
  getenv: { args: ["cstring"], returns: "cstring" },
});

const value = symbols.getenv("HOME");
// NULL returns `null`, not a C string
```

### 5. Pointer Types

```typescript
// Local buffer (caller allocates)
const buffer = new Uint8Array(1024);
const result = symbols.write(buffer, buffer.length);

// Remote/opaque pointer (address from another process)
const remotePtr: bigint = 0x7FFF_1234_5678n;
```

### 6. Performance

Bun 1.4 FFI runs on native JavaScriptCore FFI, replacing TinyCC:

| Operation | Bun 1.3 | Bun 1.4 | Speedup |
|---|---|---|---|
| no-op call | 2.13 ns | 0.70 ns | 3.0× |
| `new CString(ptr)` | 92.5 ns | 24.1 ns | 3.8× |

When a call site gets hot, the JIT compiles it into a direct call to the C function, passing unboxed values in registers.

### 7. Lifetime Management

```typescript
const lib = dlopen("libfoo.so", { ... });

// Keep library alive
using _ = lib; // Symbol.dispose unloads the library

// Or manually
lib.unload();
```

## Common Patterns

### String Arguments

```typescript
const { symbols } = dlopen("libc.so.6", {
  printf: { args: ["cstring", ...], returns: "i32" },
});

symbols.printf("Hello %s\n", "world");
```

### Callbacks

```typescript
const { symbols } = dlopen("libfoo.so", {
  setCallback: { args: ["ptr"], returns: "void" },
});

const callback = new CFunction((value: number) => {
  console.log("Callback:", value);
  return 0;
}, "i32", ["i32"]);

symbols.setCallback(callback);
```

### Structs (by value)

```typescript
// Pack small structs into bigint
type POINT = bigint; // 8-byte struct: x (i32) + y (i32)

const point: POINT = (0n << 32n) | 0n; // x=0, y=0

const { symbols } = dlopen("libfoo.so", {
  distance: { args: ["u64", "u64"], returns: "f64" },
});

const d = symbols.distance(pointA, pointB);
```

## Troubleshooting

### Symbol not found

```typescript
// Check exports first
const lib = dlopen("libfoo.so", {});
console.log(Object.keys(lib.symbols));
```

### Type mismatch

- `Pointer` for local buffers
- `bigint` for handles and remote addresses
- Never mix `ptr` and `u64` for the same parameter

### Segfaults

- Verify argument types match C signature exactly
- Check buffer sizes are sufficient
- Ensure pointers are valid (not freed)

## Completion Checklist

- ✅ Library loaded with `dlopen`
- ✅ Symbols declared with correct types
- ✅ Arguments match C signature
- ✅ Return values handled correctly
- ✅ Lifetime management configured
