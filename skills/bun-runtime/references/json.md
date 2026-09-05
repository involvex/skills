# Bun.JSON5 / Bun.JSONL / Bun.JSONC — Data Format Parsers

Built-in parsers for non-standard JSON formats.

## Bun.JSON5 — JSON with JSON5 Extensions

```typescript
// Parse JSON5: comments, trailing commas, unquoted keys
const obj = Bun.JSON5.parse(`
  {
    // This is a comment
    key: "value",
    num: 42,
    arr: [1, 2, 3,],
  }
`);

// Stringify to JSON5
const str = Bun.JSON5.stringify({ key: "value", num: 42 });
```

## Bun.JSONL — Newline-Delimited JSON

```typescript
// Parse JSONL
const lines = Bun.JSONL.parse(`
  {"id": 1, "name": "Alice"}
  {"id": 2, "name": "Bob"}
  {"id": 3, "name": "Charlie"}
`);
// [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }, ...]

// Stringify to JSONL
const jsonl = Bun.JSONL.stringify([
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
]);
// '{"id":1,"name":"Alice"}\n{"id":2,"name":"Bob"}\n'
```

## Bun.JSONC — JSON with Comments

```typescript
// Parse JSONC (JSON with // and /* */ comments)
const config = Bun.JSONC.parse(`
  {
    // Server config
    "port": 3000,   // HTTP port
    /* Multi-line comment
       spanning multiple lines */
    "host": "localhost",
  }
`);
```

## Use Cases

| Format | Use Case | Example |
|---|---|---|
| JSON5 | Config files with comments | `.bunfig.toml` alternatives |
| JSONL | Logs, bulk data import | NDJSON files |
| JSONC | VS Code settings, tsconfig | `settings.json`, `tsconfig.json` |

## Performance

All three parsers are native (written in Zig) and significantly faster than JavaScript alternatives.
