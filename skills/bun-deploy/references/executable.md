# Standalone Executable Reference

Compile Bun apps to standalone executables with `--compile`.

## Basic Usage

```bash
bun build ./src/cli.ts --compile --outfile ./dist/myapp
```

Or programmatically:

```typescript
await Bun.build({
  entrypoints: ['./src/cli.ts'],
  compile: {
    target: 'bun',
    outfile: './dist/myapp',
  },
});
```

## With Options

```bash
bun build ./src/cli.ts \
  --compile \
  --target=bun-linux-x64 \
  --outfile=./dist/myapp \
  --minify
```

## Cross-Compilation

```bash
# Compile for Linux from macOS
bun build ./src/cli.ts --compile --target=bun-linux-x64 --outfile myapp-linux

# Compile for Windows from macOS
bun build ./src/cli.ts --compile --target=bun-windows-x64 --outfile myapp.exe

# Compile for macOS ARM from Intel
bun build ./src/cli.ts --compile --target=bun-darwin-arm64 --outfile myapp-arm64
```

## Bytecode Compilation (Bun 1.4.1+)

```bash
bun build ./src/cli.ts --compile --bytecode --target=bun-linux-x64 --outfile myapp
```

Benefits:
- Smaller binary size
- Faster startup
- Source code not easily reversible

## With TypeScript

```bash
# TypeScript works out of the box
bun build ./src/cli.ts --compile --outfile myapp
```

## With Dependencies

```bash
# All dependencies are bundled
bun build ./src/cli.ts --compile --outfile myapp
```

## Package.json Script

```json
{
  "scripts": {
    "build:cli": "bun build ./src/cli.ts --compile --outfile ./dist/myapp",
    "build:cli:linux": "bun build ./src/cli.ts --compile --target=bun-linux-x64 --outfile ./dist/myapp-linux",
    "build:cli:win": "bun build ./src/cli.ts --compile --target=bun-windows-x64 --outfile ./dist/myapp.exe"
  }
}
```

## When to Use

- CLI tools
- Worker processes
- Scheduled jobs
- Any app that needs to run without Bun installed

## Binary Size

| Platform | Bun 1.4 | Bun 1.3 |
|---|---|---|
| Linux x64 | 77.0 MB | 88.5 MB |
| Linux arm64 | 76.8 MB | 87.6 MB |
| Windows x64 | 84.8 MB | 93.9 MB |
| Windows arm64 | 75.1 MB | 90.2 MB |

With `--bytecode`, binaries are even smaller.
