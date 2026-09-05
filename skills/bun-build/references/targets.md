# Build Targets and Configurations

Complete configurations for different build targets using Bun's native bundler.

## Browser/Frontend Build

```typescript
// build-browser.ts
await Bun.build({
  entrypoints: ['./src/index.tsx'],
  outdir: './dist',
  target: 'browser',
  format: 'esm',
  minify: {
    whitespace: true,
    identifiers: true,
    syntax: true,
  },
  splitting: true,
  sourcemap: 'external',
  external: [], // Bundle everything
  define: {
    'process.env.NODE_ENV': '"production"',
    'process.env.API_URL': '"https://api.example.com"',
  },
  loader: {
    '.png': 'file',
    '.jpg': 'file',
    '.svg': 'dataurl',
    '.css': 'css',
  },
});
```

## Node.js Backend Build

```typescript
// build-node.ts
await Bun.build({
  entrypoints: ['./src/server.ts'],
  outdir: './dist',
  target: 'node',
  format: 'esm',
  minify: true,
  sourcemap: 'inline',
  external: ['*'], // Don't bundle node_modules
  // Or be explicit:
  // external: ['express', 'mongodb', 'redis'],
});
```

## Library Build (Dual Format)

```typescript
// build-library.ts

// ESM build
await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist/esm',
  target: 'node',
  format: 'esm',
  minify: true,
  sourcemap: 'external',
  external: ['*'],
});

// CommonJS build
await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist/cjs',
  target: 'node',
  format: 'cjs',
  minify: true,
  sourcemap: 'external',
  external: ['*'],
});

console.log('✅ Built ESM and CJS formats');
```

Update `package.json`:
```json
{
  "type": "module",
  "main": "./dist/cjs/index.js",
  "module": "./dist/esm/index.js",
  "types": "./dist/esm/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/esm/index.js",
      "require": "./dist/cjs/index.js",
      "types": "./dist/esm/index.d.ts"
    }
  },
  "files": ["dist"]
}
```

## CLI Tool Build

```typescript
// build-cli.ts
await Bun.build({
  entrypoints: ['./src/cli.ts'],
  outdir: './dist',
  target: 'bun',
  format: 'esm',
  minify: true,
  // Bundle everything for single-file distribution
  external: [],
});

// Make executable
import { chmod } from 'fs/promises';
await chmod('./dist/cli.js', 0o755);

console.log('✅ CLI built and made executable');
```

Update `package.json`:
```json
{
  "bin": {
    "your-cli-name": "./dist/cli.js"
  }
}
```

## Cloudflare Workers

```typescript
// build-worker.ts
await Bun.build({
  entrypoints: ['./src/worker.ts'],
  outdir: './dist',
  target: 'browser',
  format: 'esm',
  minify: true,
  external: [],
});
```

## Bun Runtime Target

```typescript
// build-bun.ts
await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist',
  target: 'bun',
  format: 'esm',
  minify: true,
  // Can use Bun-specific features
});
```

## Standalone Executable (Bun 1.3.10+)

Compile to a standalone binary:

```typescript
// build-exe.ts
const result = await Bun.build({
  entrypoints: ['./src/cli.ts'],
  compile: {
    target: 'bun',           // 'bun' | 'node' | 'browser'
    outfile: './dist/myapp', // Output binary path
    format: 'esm',
    minify: true,
    bytecode: false,         // true for bytecode compilation (1.4.1+)
  },
});

if (!result.success) {
  console.error('❌ Build failed');
  process.exit(1);
}

console.log('✅ Standalone executable built: ./dist/myapp');
console.log(`📦 Size: ${(result.outputs[0].size / 1024 / 1024).toFixed(1)} MB`);
```

Cross-compile with bytecode (Bun 1.4.1+):
```bash
# Compile for a different platform
bun build ./src/cli.ts --compile --target=bun-linux-x64 --outfile=myapp-linux

# Compile with bytecode for smaller size
bun build ./src/cli.ts --compile --bytecode --target=bun-darwin-arm64 --outfile=myapp-mac
```

Available targets:
- `bun` — Current platform
- `bun-linux-x64`, `bun-linux-arm64`
- `bun-darwin-arm64`, `bun-darwin-x64`
- `bun-windows-x64`, `bun-windows-arm64`
- `bun-android-arm64`, `bun-android-x64`
- `node` — Node.js standalone
- `browser` — Self-contained HTML

## Self-Contained HTML (Bun 1.3.10+)

Bundle your app into a single self-contained HTML file:

```bash
# CLI
bun build ./src/index.tsx --target=browser --outdir=./dist --outfile=app.html
```

```typescript
// build-html.ts
await Bun.build({
  entrypoints: ['./src/index.tsx'],
  outdir: './dist',
  target: 'browser',
  format: 'iife',
  minify: true,
});
```

When using `--target=browser` with an `outfile`, Bun produces a single self-contained HTML file with all assets embedded.

## Target Comparison

| Target | Use Case | Node APIs | Browser APIs | Bun APIs | Output |
|--------|----------|-----------|--------------|----------|--------|
| `browser` | Frontend apps | ❌ | ✅ | ❌ | JS/CSS/assets |
| `node` | Backend apps | ✅ | ❌ | ❌ | JS |
| `bun` | Bun-specific apps | ✅ | ❌ | ✅ | JS |
| `bun-*` | Standalone executable | ✅ | ❌ | ✅ | Binary |
| `browser` + outfile | Self-contained HTML | ❌ | ✅ | ❌ | Single HTML |

## Format Options

### ESM (Recommended)

```typescript
{
  format: 'esm',  // Modern, tree-shakeable
}
```

Output:
```javascript
export default function() {}
export { foo, bar };
```

### CommonJS

```typescript
{
  format: 'cjs',  // Legacy Node.js
}
```

Output:
```javascript
module.exports = function() {}
exports.foo = foo;
```

### IIFE (Browser Scripts)

```typescript
{
  format: 'iife',  // Self-contained browser script
}
```

Output:
```javascript
(function() {
  // Your code
})();
```

## Virtual Filesystem (Bun 1.3.6+)

Inject virtual files into the bundle without creating them on disk:

```typescript
await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist',
  files: {
    './version.txt': '1.0.0',
    './config.json': JSON.stringify({ api: 'https://api.example.com' }),
    './README.md': await Bun.file('./README.md').text(),
  },
});
```

These files are accessible via `import` or `Bun.file` at runtime.
