# Bun ANSI Utilities

SIMD-accelerated ANSI string operations. 33–88× faster than JavaScript alternatives.

## Bun.wrapAnsi() — Wrap Text Preserving ANSI Codes

```typescript
// Wrap text to terminal width, preserving ANSI escape codes
const wrapped = Bun.wrapAnsi(ansiString, 80);

// Example with colored text
const colored = "\x1b[31mThis is a long red string that needs wrapping\x1b[0m";
const wrapped = Bun.wrapAnsi(colored, 40);
```

## Bun.stripANSI — Remove ANSI Escape Codes

```typescript
const clean = Bun.stripANSI("\x1b[31mred text\x1b[0m");
// "red text"

// Useful for measuring string length without ANSI codes
const displayLength = Bun.stripANSI(ansiString).length;
```

## Bun.sliceAnsi — Slice ANSI Strings

```typescript
// Slice an ANSI string preserving escape codes
const sliced = Bun.sliceAnsi(ansiString, 0, 20);
// Returns first 20 visible characters, with ANSI codes intact
```

## When to Use

| Task | Bun API | npm Package |
|---|---|---|
| Wrap colored text | `Bun.wrapAnsi()` | `wrap-ansi` |
| Remove ANSI codes | `Bun.stripANSI` | `strip-ansi` |
| Slice ANSI text | `Bun.sliceAnsi` | — |

## Performance

- `Bun.wrapAnsi()`: 33–88× faster than `wrap-ansi`
- `Bun.stripANSI`: Uses SIMD for bulk escape sequence removal
- `Bun.sliceAnsi`: Grapheme-aware, handles multi-byte characters correctly
