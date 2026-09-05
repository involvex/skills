---
name: bun-build
description: Create optimized production bundles with Bun's native bundler. Use when building applications for production, optimizing bundle sizes, setting up multi-environment builds, or replacing webpack/esbuild/rollup.
compatibility: Requires Bun 1.4+
allowed-tools: ["Bash", "Write", "Read"]
metadata:
  author: dale
  category: bun-runtime
  tags: "bun, build, bundler, production, optimization"
---

# Bun Production Build Configuration

Set up production builds using Bun's native bundler — fast, optimized bundle creation without webpack or esbuild.

## What's new in Bun 1.4

- **Standalone executables**: `Bun.build({ compile: true })` compiles to a standalone binary (1.3.10)
- **Self-contained HTML**: `--target=browser` produces a self-contained HTML file (1.3.10)
- **Bundle analysis as Markdown**: `--metafile-md` writes LLM-friendly bundle analysis (1.3.8)
- **Bytecode compilation**: `--compile --bytecode` for cross-compilation (1.4.1)
- **Tree-shaking via dynamic import**: dynamic `import()` now participates in tree-shaking
- **Barrel import optimization**: re-export chains are optimized automatically
- **Virtual filesystem**: `Bun.build({ files })` injects virtual files into the bundle
- **Smaller/faster compiled executables**: up to 17% smaller binaries, faster startup

## Quick Reference

For detailed patterns, see:
- **Build Targets**: [targets.md](references/targets.md) — Browser, Node.js, library, CLI, standalone executable, self-contained HTML
- **Optimization**: [optimization.md](references/optimization.md) — Tree shaking, code splitting, bundle analysis, size limits
- **Plugins**: [plugins.md](references/plugins.md) — Custom loaders and transformations

## Core Workflow

### 1. Check Prerequisites

```bash
# Verify Bun installation
bun --version

# Check project structure
ls -la package.json src/
```

### 2. Determine Build Requirements

Ask the user about their build needs:

- **Application Type**: Frontend SPA, Node.js backend, CLI tool, library, or standalone executable
- **Target Platform**: Browser, Node.js, Bun runtime, Cloudflare Workers, or self-contained HTML
- **Output Format**: ESM (modern), CommonJS (legacy), IIFE, or standalone binary
- **Special needs**: Image optimization, markdown bundling, virtual files

### 3. Create Basic Build Script

Create `build.ts` in project root:

```typescript
#!/usr/bin/env bun

const result = await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist',
  target: 'browser', // or 'node', 'bun'
  format: 'esm', // or 'cjs', 'iife'
  minify: true,
  splitting: true,
  sourcemap: 'external',
});

if (!result.success) {
  console.error('Build failed');
  for (const message of result.logs) {
    console.error(message);
  }
  process.exit(1);
}

console.log('✅ Build successful');
console.log(`📦 ${result.outputs.length} files generated`);

// Show bundle sizes
for (const output of result.outputs) {
  const size = (output.size / 1024).toFixed(2);
  console.log(`  ${output.path} - ${size} KB`);
}
```

### 4. Configure for Target Platform

**For Browser/Frontend:**

```typescript
await Bun.build({
  entrypoints: ['./src/index.tsx'],
  outdir: './dist',
  target: 'browser',
  format: 'esm',
  minify: true,
  splitting: true,
  define: {
    'process.env.NODE_ENV': '"production"',
  },
  loader: {
    '.png': 'file',
    '.svg': 'dataurl',
    '.css': 'css',
  },
});
```

**For Node.js Backend:**

```typescript
await Bun.build({
  entrypoints: ['./src/server.ts'],
  outdir: './dist',
  target: 'node',
  format: 'esm',
  minify: true,
  external: ['*'], // Don't bundle node_modules
});
```

**For libraries, CLI tools, standalone executables, and other targets**, see [targets.md](references/targets.md).

### 5. Standalone Executable (Bun 1.3.10+)

Compile your app to a standalone binary:

```typescript
#!/usr/bin/env bun

const result = await Bun.build({
  entrypoints: ['./src/cli.ts'],
  compile: {
    target: 'bun',        // 'bun', 'node', 'browser'
    outfile: './dist/myapp',
    format: 'esm',
    minify: true,
  },
});

if (!result.success) {
  console.error('Build failed');
  process.exit(1);
}

console.log('✅ Standalone executable built: ./dist/myapp');
console.log(`📦 Size: ${(result.outputs[0].size / 1024 / 1024).toFixed(1)} MB`);
```

Run it:
```bash
chmod +x ./dist/myapp
./dist/myapp --help
```

Cross-compile with bytecode (Bun 1.4.1+):
```bash
bun build ./src/cli.ts --compile --bytecode --target=bun-linux-x64 --outfile=myapp
```

For all compile options and targets, see [targets.md](references/targets.md).

### 6. Self-Contained HTML (Bun 1.3.10+)

Bundle your app into a single self-contained HTML file:

```bash
bun build ./src/index.tsx --target=browser --outdir=./dist --outfile=app.html
```

Or in `build.ts`:
```typescript
await Bun.build({
  entrypoints: ['./src/index.tsx'],
  outdir: './dist',
  target: 'browser',
  format: 'iife',
  minify: true,
  // Output is a single HTML file when using --target=browser with outfile
});
```

### 7. Bundle Analysis (Bun 1.3.8+)

Get LLM-friendly bundle analysis as Markdown:

```bash
bun build ./src/index.ts --outdir ./dist --metafile-md=./dist/meta.md
```

Or in `build.ts`:
```typescript
const result = await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist',
  minify: true,
  splitting: true,
  metafile: true,  // Required for --metafile-md
});

if (!result.success) {
  process.exit(1);
}

// Write Markdown analysis
await Bun.write(
  './dist/meta.md',
  result.metafile
    ? await Bun.build({ ...options, 'metafile-md': './dist/meta.md' })
    : 'No metafile available',
);
```

The Markdown report shows: largest modules, entry point analysis, dependency chains, and full module graph.

For advanced bundle analysis, see [optimization.md](references/optimization.md).

### 8. Add Production Optimizations

```typescript
await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist',
  target: 'browser',

  // Maximum minification
  minify: {
    whitespace: true,
    identifiers: true,
    syntax: true,
  },

  // Code splitting for optimal caching
  splitting: true,

  // Content hashing for cache busting
  naming: {
    entry: '[dir]/[name].[hash].[ext]',
    chunk: 'chunks/[name].[hash].[ext]',
    asset: 'assets/[name].[hash].[ext]',
  },

  // Source maps for debugging
  sourcemap: 'external',
});
```

For advanced optimizations (virtual filesystem, tree-shaking via dynamic import, barrel optimization), see [optimization.md](references/optimization.md).

### 9. Environment-Specific Builds

Create `build-env.ts`:

```typescript
#!/usr/bin/env bun

const env = process.env.NODE_ENV || 'development';

const configs = {
  development: {
    minify: false,
    sourcemap: 'inline',
    define: {
      'process.env.NODE_ENV': '"development"',
      'process.env.API_URL': '"http://localhost:3000"',
    },
  },
  production: {
    minify: true,
    sourcemap: 'external',
    define: {
      'process.env.NODE_ENV': '"production"',
      'process.env.API_URL': '"https://api.example.com"',
    },
  },
};

const config = configs[env as keyof typeof configs];

const result = await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist',
  target: 'browser',
  format: 'esm',
  splitting: true,
  ...config,
});

if (!result.success) {
  console.error('❌ Build failed');
  process.exit(1);
}

console.log(`✅ ${env} build successful`);
```

Run with:
```bash
NODE_ENV=production bun run build-env.ts
```

### 10. Update package.json

Add build scripts:

```json
{
  "scripts": {
    "build": "bun run build.ts",
    "build:dev": "NODE_ENV=development bun run build-env.ts",
    "build:prod": "NODE_ENV=production bun run build-env.ts",
    "build:watch": "bun run build.ts --watch",
    "build:analyze": "bun run build.ts --metafile-md=./dist/meta.md",
    "build:exe": "bun run build-exe.ts",
    "clean": "rm -rf dist"
  }
}
```

**For libraries**, also add:

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

### 11. Generate Type Declarations (Libraries)

For libraries, generate TypeScript declarations:

```typescript
// build-lib-with-types.ts
import { $ } from 'bun';

// Build JavaScript
await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist',
  target: 'node',
  format: 'esm',
  minify: true,
});

// Generate type declarations
await $`tsc --declaration --emitDeclarationOnly --outDir dist`;

console.log('✅ Built library with type declarations');
```

## Build Options Reference

### Target

- **`browser`**: For web applications (includes browser globals)
- **`node`**: For Node.js applications (assumes Node.js APIs)
- **`bun`**: For Bun runtime (optimized for Bun-specific features)
- **`bun-linux-x64`**, **`bun-linux-arm64`**, **`bun-darwin-arm64`**, etc.: For standalone executables

### Format

- **`esm`**: ES Modules (modern, tree-shakeable) — Recommended
- **`cjs`**: CommonJS (legacy Node.js)
- **`iife`**: Immediately Invoked Function Expression (browser scripts)

### Compile Options (Bun 1.3.10+)

```typescript
compile: {
  target: 'bun',           // 'bun' | 'node' | 'browser'
  outfile: './dist/app',   // Output file path
  format: 'esm',           // Output format
  minify: true,            // Minify output
  bytecode: false,         // Compile to bytecode (1.4.1+)
}
```

### Minification

```typescript
minify: true                  // Basic minification
minify: {                     // Granular control
  whitespace: true,
  identifiers: true,
  syntax: true,
}
```

### Source Maps

- **`external`**: Separate .map files (production)
- **`inline`**: Inline in bundle (development)
- **`none`**: No source maps

### Virtual Filesystem (Bun 1.3.6+)

Inject virtual files into the bundle:

```typescript
await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist',
  files: {
    './version.txt': '1.0.0',
    './config.json': JSON.stringify({ api: 'https://api.example.com' }),
  },
});
```

### Bundle Analysis

```typescript
await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist',
  metafile: true,  // Generate metafile for analysis
});
```

Or via CLI:
```bash
bun build ./src/index.ts --outdir ./dist --metafile-md=./dist/meta.md
```

## Verification

After building:

```bash
# 1. Check output directory
ls -lh dist/

# 2. Verify bundle size
du -sh dist/*

# 3. Test bundle
bun run dist/index.js

# 4. Check for errors
echo $?  # Should be 0

# 5. View bundle analysis (if generated)
cat dist/meta.md
```

## Common Build Patterns

**Watch mode for development:**

```typescript
import { watch } from 'fs';

async function build() {
  await Bun.build({
    entrypoints: ['./src/index.ts'],
    outdir: './dist',
    minify: false,
  });
}

await build();

watch('./src', { recursive: true }, async (event, filename) => {
  if (filename?.endsWith('.ts')) {
    console.log(`Rebuilding...`);
    await build();
  }
});
```

**Custom asset loaders:**

```typescript
loader: {
  '.png': 'file',     // Copy file, return path
  '.svg': 'dataurl',  // Inline as data URL
  '.txt': 'text',     // Inline as string
  '.json': 'json',    // Parse and inline
  '.md': 'text',      // Inline markdown as string
}
```

**For custom plugins and advanced transformations**, see [plugins.md](references/plugins.md).

## Troubleshooting

**Build fails:**
```typescript
if (!result.success) {
  for (const log of result.logs) {
    console.error(log);
  }
}
```

**Bundle too large:**
See [optimization.md](references/optimization.md) for:
- Bundle analysis with `--metafile-md`
- Code splitting
- Tree shaking through dynamic import
- Size limits

**Module not found:**
Check `external` configuration:
```typescript
external: ['*']         // Exclude all node_modules
external: ['react']     // Exclude specific packages
external: []            // Bundle everything
```

**Compiled executable too large:**
```typescript
compile: {
  target: 'bun',
  bytecode: true,  // Use bytecode for smaller size
  minify: true,
}
```

## Completion Checklist

- ✅ Build script created
- ✅ Target platform configured
- ✅ Minification enabled
- ✅ Source maps configured
- ✅ Environment-specific builds set up
- ✅ Package.json scripts added
- ✅ Build tested successfully
- ✅ Bundle size verified
- ✅ Bundle analysis generated (if applicable)

## Next Steps

After basic build setup:

1. **Standalone executable**: Add `--compile` for CLI tools
2. **Self-contained HTML**: Use `--target=browser` for single-file demos
3. **Optimization**: Add bundle analysis and size limits
4. **CI/CD**: Automate builds in your pipeline
5. **Type Checking**: Add pre-build type checking
6. **Testing**: Run tests before building
7. **Deployment**: Integrate with containerization

For detailed implementations, see the reference files linked above.
