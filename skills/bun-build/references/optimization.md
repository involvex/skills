# Build Optimization Strategies

## Code Splitting

Enable code splitting for optimal caching:

```typescript
await Bun.build({
  entrypoints: [
    './src/index.ts',
    './src/admin.ts', // Separate entry for admin panel
  ],
  outdir: './dist',
  target: 'browser',
  splitting: true, // Enable code splitting
  naming: {
    entry: '[dir]/[name].[ext]',
    chunk: 'chunks/[name]-[hash].[ext]',
    asset: 'assets/[name]-[hash].[ext]',
  },
});
```

## Tree Shaking

Tree shaking is automatic, but you can help:

```typescript
await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist',
  target: 'browser',
  minify: true,

  // Remove debug code
  define: {
    'process.env.DEBUG': 'false',
    '__DEV__': 'false',
  },

  // Mark side-effect-free packages
  external: [],
});
```

Use named exports for better tree shaking:

```typescript
// ❌ Harder to tree shake
export default { foo, bar, baz };

// ✅ Tree shakeable
export { foo, bar, baz };
```

### Dynamic Import Tree Shaking (Bun 1.4+)

Dynamic `import()` now participates in tree-shaking. Use it for code splitting:

```typescript
// Dynamic imports are analyzed for tree-shaking
async function loadFeature() {
  const module = await import('./heavy-feature.ts');
  return module.doWork();
}

// Only loaded modules are included in the bundle
// Unused dynamic imports are tree-shaken
```

### Barrel Import Optimization (Bun 1.4+)

Re-export chains (barrel files) are optimized automatically:

```typescript
// index.ts - barrel file
export { foo } from './foo';
export { bar } from './bar';
export { baz } from './baz';

// Bun automatically optimizes the import chain
// No need for manual re-export optimization
```

## Bundle Analysis with Markdown (Bun 1.3.8+)

Generate LLM-friendly bundle analysis as Markdown:

```bash
# CLI
bun build ./src/index.ts --outdir ./dist --metafile-md=./dist/meta.md
```

```typescript
// build-analyze.ts
const result = await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist',
  minify: true,
  splitting: true,
  sourcemap: 'external',
  metafile: true, // Required for metafile-md
});

if (result.success && result.metafile) {
  // Write Markdown analysis
  await Bun.write('./dist/meta.md', result.metafile);
}
```

The Markdown report includes:
- **Largest Modules**: Top contributors to bundle size
- **Entry Point Analysis**: What each entry point loads
- **Dependency Chains**: Why each module is included
- **Full Module Graph**: Complete dependency information

Example output:
```markdown
# Bundle Analysis Report

## Quick Summary
| Metric | Value |
| Total output size | 56.1 KB |

## Largest Modules by Output Contribution
| Output Bytes | % of Total | Module |
| 55.74 KB | 99.4% | `node_modules/marked/lib/marked.esm.js` |
| 113 bytes | 0.2% | `src/escape.ts` |
```

## Minification

### Basic Minification

```typescript
await Bun.build({
  minify: true,  // Simple boolean
});
```

### Granular Minification

```typescript
await Bun.build({
  minify: {
    whitespace: true,    // Remove whitespace
    identifiers: true,   // Shorten variable names
    syntax: true,        // Simplify syntax
  },
});
```

## Source Maps

```typescript
await Bun.build({
  sourcemap: 'external',  // Separate .map files
  // or
  sourcemap: 'inline',    // Inline in bundle
  // or
  sourcemap: 'none',      // No source maps
});
```

## Asset Loaders

```typescript
await Bun.build({
  loader: {
    '.png': 'file',     // Copy file, return path
    '.jpg': 'file',
    '.svg': 'dataurl',  // Inline as data URL
    '.css': 'css',      // Process as CSS
    '.txt': 'text',     // Inline as string
    '.json': 'json',    // Inline and parse JSON
    '.md': 'text',      // Inline markdown (Bun 1.3.8+)
  },

  // Public path for assets
  publicPath: '/static/',
});
```

## Virtual Filesystem (Bun 1.3.6+)

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

## Environment-Specific Builds

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
  staging: {
    minify: true,
    sourcemap: 'external',
    define: {
      'process.env.NODE_ENV': '"staging"',
      'process.env.API_URL': '"https://staging-api.example.com"',
    },
  },
  production: {
    minify: {
      whitespace: true,
      identifiers: true,
      syntax: true,
    },
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

## Performance Tips

1. **Use --hot for development**: Faster than full rebuilds
2. **Enable code splitting**: Better caching
3. **Externalize dependencies**: Don't bundle node_modules for backend
4. **Use proper loaders**: 'dataurl' for small files, 'file' for large
5. **Enable minification**: Only in production
6. **Use --metafile-md**: Analyze bundle bloat with LLM-friendly output
7. **Leverage tree-shaking**: Use named exports and dynamic imports strategically

## Watch Mode

```typescript
// build-watch.ts
import { watch } from 'fs';

async function build() {
  console.log('🔨 Building...');

  const result = await Bun.build({
    entrypoints: ['./src/index.ts'],
    outdir: './dist',
    target: 'browser',
    minify: false,
    sourcemap: 'inline',
  });

  if (result.success) {
    console.log('✅ Build complete');
  } else {
    console.error('❌ Build failed');
  }
}

// Initial build
await build();

// Watch for changes
watch('./src', { recursive: true }, async (event, filename) => {
  if (filename?.endsWith('.ts') || filename?.endsWith('.tsx')) {
    console.log(`\n📝 ${filename} changed`);
    await build();
  }
});

console.log('\n👀 Watching for changes...');
```

## Bundle Size Limits

Enforce size limits:

```typescript
const MAX_SIZES = {
  total: 500 * 1024,     // 500 KB total
  chunk: 200 * 1024,     // 200 KB per chunk
};

for (const output of result.outputs) {
  if (output.size > MAX_SIZES.chunk) {
    console.error(`❌ Chunk too large: ${output.path}`);
    process.exit(1);
  }
}

const totalSize = result.outputs.reduce((sum, o) => sum + o.size, 0);
if (totalSize > MAX_SIZES.total) {
  console.error(`❌ Total bundle too large: ${totalSize} bytes`);
  process.exit(1);
}
```

## Standalone Executable Optimization (Bun 1.3.10+)

For CLI tools, compile to a standalone binary:

```typescript
const result = await Bun.build({
  entrypoints: ['./src/cli.ts'],
  compile: {
    target: 'bun',
    outfile: './dist/myapp',
    format: 'esm',
    minify: true,
    bytecode: false, // Set true for smaller size (1.4.1+)
  },
});
```

- Binaries are up to 17% smaller in Bun 1.4
- Startup is 50% faster on Windows, 2× faster on Linux
- Use `--bytecode` for additional size reduction (cross-compilation support in 1.4.1)
