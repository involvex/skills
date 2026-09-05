---
name: node-to-bun
description: Migrate Node.js projects to Bun with compatibility analysis. Use when converting existing npm/pnpm/yarn projects to Bun or auditing dependencies for Bun compatibility.
compatibility: Requires Bun 1.4+
allowed-tools: ["Bash", "Read", "Grep", "Write"]
metadata:
  author: daleseo
  category: bun-runtime
  tags: [bun, migration, nodejs, node-to-bun, compatibility, dependencies]
---

# Node.js to Bun Migration

Migrate an existing Node.js project to Bun 1.4+. This involves analyzing dependencies, updating configurations, and leveraging Bun's native APIs to replace npm packages.

## What's new in Bun 1.4

Bun 1.4 dramatically expands what you can drop in from npm — and what you can delete entirely:

- **Playwright** runs on Bun via `connectOverCDP()` and `playwright test`
- **vitest** runs under Bun with `--coverage`, threads, and forks pools
- **Next.js 16** works with `bun --bun next build` and Turbopack
- **OpenTelemetry** http/fs instrumentation works with `node:http` and `node:fs`
- **dd-trace** and `@datadog/pprof` work via V8 C++ API implementations
- **Node.js 26.3.0** compatibility: +1,517 newly passing tests
- **3× faster `bun:ffi`** for native bindings

## Migration Workflow

### 1. Pre-Migration Analysis

**Check if Bun is installed:**
```bash
bun --version
```

**Analyze current project:**
```bash
# Check Node.js version
node --version

# Check package manager
ls -la | grep -E "package-lock.json|yarn.lock|pnpm-lock.yaml"
```

Read `package.json` to understand the project structure.

### 2. Dependency Compatibility Check

**Read and analyze all dependencies** from `package.json`:

```bash
cat package.json
```

**Check for packages that Bun 1.4 can replace entirely:**

| npm Package | Bun 1.4 Replacement | Notes |
|---|---|---|
| `sharp` | `Bun.Image` | Built-in image processing. JPEG, PNG, WebP, GIF, BMP, HEIC, AVIF, TIFF. 1.38× faster than sharp on resize. |
| `puppeteer` / `playwright` | `Bun.WebView` | Headless browser automation built in. Navigate, click, screenshot, CDP escape hatch. No install needed on macOS (uses system WebKit). |
| `marked` / `markdown-it` / `showdown` | `Bun.markdown` | Built-in CommonMark parser. `.html()`, `.react()`, `.render()` for terminal output. |
| `node-cron` / `cron` | `Bun.cron()` | OS-level cron (crontab/launchd/Task Scheduler) + in-process scheduler. Cloudflare Workers-style `scheduled()` handler. |
| `node-pty` | `Bun.Terminal` | Built-in PTY. Drive `bash`, `vim`, `htop` from JS without native addons. |
| `ioredis` / `redis` | `Bun.redis` | Built-in Redis client with Pub/Sub. |
| `pg` / `mysql` / `mysql2` | `Bun.SQL` | Unified SQL API supporting PostgreSQL, MySQL, and SQLite. |
| `tar` / `archiver` / `decompress` | `Bun.Archive` | Create/extract tar.gz archives natively. |
| `json5` / `json5-promised` | `Bun.JSON5` | Built-in JSON5 parser. |
| `jsonl-parse` / `ndjson` | `Bun.JSONL` | Built-in JSONL parser. |
| `strip-ansi` / `wrap-ansi` | `Bun.stripANSI` / `Bun.wrapAnsi` | SIMD-accelerated. 33–88× faster than `wrap-ansi`. |
| `dotenv` | Built-in | Bun loads `.env` files automatically. |
| `concurrently` / `npm-run-all` | `bun run --parallel` | Run multiple scripts concurrently with name-prefixed output. |
| `nodemon` / `tsx` / `ts-node` | `bun run --hot` / `bun run --watch` | Built-in file watching and TypeScript execution. |
| `jest` / `vitest` | `bun test` | Built-in test runner. 3–10× faster than Jest. Parallel, isolated, sharded. |
| `esbuild` / `swc` / `webpack` | `Bun.build()` | Native bundler. Up to 3× faster than esbuild. |
| `vite` | `bun build` + `bun run --hot` | Bun's bundler + dev server replaces Vite for many use cases. |
| `tsc` (standalone) | Built-in | Bun ships TypeScript compiler natively. |

**Known incompatible native modules (check against current dependencies):**

- `bcrypt` → Use `bcryptjs` or `@node-rs/bcrypt` instead
- `sharp` → Use `Bun.Image` (see above)
- `node-canvas` → Limited support, check version compatibility
- `sqlite3` → Use `bun:sqlite` instead
- `node-gyp` dependent packages → May require alternative pure JS versions
- `fsevents` → macOS-specific, usually optional dependency

**Check workspace configuration** (for monorepos):
```bash
# Check if workspaces are defined
grep -n "workspaces" package.json
```

### 3. Generate Compatibility Report

Create a migration report file `BUN_MIGRATION_REPORT.md`:

```markdown
# Bun Migration Analysis Report

## Project Overview
- **Name**: [project name]
- **Current Node Version**: [version]
- **Package Manager**: [npm/yarn/pnpm]
- **Project Type**: [app/library/monorepo]

## Dependency Analysis

### ✅ Compatible Dependencies
[List dependencies that work without changes]

### 🔄 Replaceable with Bun Native APIs
[List npm packages that Bun 1.4 replaces]

| npm Package | Bun Replacement | Savings |
|---|---|---|
| sharp | Bun.Image | ~1 dependency, faster |
| marked | Bun.markdown | ~1 dependency |
| node-cron | Bun.cron() | ~1 dependency |
| ioredis | Bun.redis | ~1 dependency |
| pg/mysql | Bun.SQL | ~1-2 dependencies |

### ⚠️ Potentially Incompatible Dependencies
[List dependencies that may have issues]

**Recommended Actions:**
- [Specific migration steps for each incompatible dependency]

## Configuration Changes Needed

### package.json
- [ ] Update scripts to use `bun` instead of `npm`/`yarn`
- [ ] Remove replaceable dependencies
- [ ] Review and update `engines` field
- [ ] Check `type` field (ESM vs CommonJS)

### tsconfig.json
- [ ] Update `moduleResolution` to `"bundler"`
- [ ] Add `@types/bun` to types array
- [ ] Set `allowImportingTsExtensions` to `true`
- [ ] Add `verbatimModuleSyntax` and `noImplicitOverride`

### Build Configuration
- [ ] Review webpack/rollup/esbuild config (may use Bun.build)
- [ ] Update test runner config (use Bun test)

## Migration Steps

1. Install Bun dependencies
2. Update configuration files
3. Replace npm packages with Bun native APIs
4. Run tests to verify compatibility
5. Update CI/CD pipelines
6. Update documentation

## Risk Assessment

**Low Risk:**
[List low-risk changes]

**Medium Risk:**
[List items needing testing]

**High Risk:**
[List critical compatibility concerns]
```

### 4. Backup Current State

Before making changes:

```bash
# Create backup branch if in git repo
git branch -c backup-before-bun-migration

# Or suggest user commits current state
git add -A
git commit -m "Backup before Bun migration"
```

### 5. Update package.json

**Read current package.json:**
```typescript
// Read and parse package.json
```

**Update scripts** to use Bun:

```json
{
  "scripts": {
    "dev": "bun run --hot src/index.ts",
    "start": "bun run src/index.ts",
    "build": "bun build src/index.ts --outdir=dist",
    "test": "bun test",
    "typecheck": "bun run --bun tsc --noEmit",
    "lint": "bun run --bun eslint ."
  }
}
```

**Update engines field:**
```json
{
  "engines": {
    "bun": ">=1.4.0"
  }
}
```

**Remove replaceable dependencies:**
```bash
# Remove packages that Bun replaces
bun remove sharp marked node-cron ioredis pg mysql
bun remove concurrently jest ts-node nodemon dotenv
bun remove tar archiver node-pty redis json5
```

**For libraries, add exports field** if not present:
```json
{
  "type": "module",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    }
  }
}
```

### 6. Replace npm Packages with Bun Native APIs

**Before:**
```typescript
// Image processing
import sharp from 'sharp';
await sharp('photo.jpg').resize(1024, 1024).toFile('thumb.jpg');

// Markdown parsing
import { marked } from 'marked';
const html = marked.parse('# Hello');

// Cron jobs
import cron from 'node-cron';
cron.schedule('*/5 * * * *', doWork);

// Redis
import Redis from 'ioredis';
const redis = new Redis();

// SQL
import { Pool } from 'pg';
const pool = new Pool();

// PTY
import pty from 'node-pty';
const proc = pty.spawn('bash', [], { cols: 80, rows: 24 });
```

**After:**
```typescript
// Image processing — Bun.Image (1.3.14+)
await Bun.file("photo.jpg")
  .image()
  .resize(1024, 1024)
  .toFile("thumb.jpg");

// Markdown — Bun.markdown (1.3.8+)
const html = Bun.markdown.html("# Hello");

// Cron — Bun.cron() (1.3.11+)
await Bun.cron("./worker.ts", "*/5 * * * *", "my-job");

// Redis — Bun.redis (1.2.9+)
const redis = new Bun.Redis("redis://localhost:6379");

// SQL — Bun.SQL (1.2.21+)
const sql = Bun.sql`postgres://localhost/mydb`;

// PTY — Bun.Terminal (1.3.5+)
const proc = Bun.spawn(['bash'], {
  terminal: { cols: 80, rows: 24, data(term, data) {} }
});
```

### 7. Update tsconfig.json

**Read current tsconfig:**
```bash
cat tsconfig.json
```

**Apply Bun-specific updates:**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "types": ["@types/bun"],
    "lib": ["ES2022"],
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "allowImportingTsExtensions": true,
    "noEmit": true,
    "verbatimModuleSyntax": true,
    "noImplicitOverride": true
  }
}
```

**Key changes explained:**
- `moduleResolution: "bundler"` → Uses Bun's module resolution
- `types: ["@types/bun"]` → Adds Bun's TypeScript definitions (replaces `bun-types`)
- `allowImportingTsExtensions: true` → Allows importing `.ts` files directly
- `noEmit: true` → Bun runs TypeScript directly, no compilation needed
- `verbatimModuleSyntax: true` → Cleaner ESM/CJS interop (TypeScript 5.x)
- `noImplicitOverride: true` → Safer class inheritance

### 8. Handle Workspace Configuration

**For monorepos with workspaces:**

Verify workspace configuration is compatible:

```json
{
  "workspaces": [
    "packages/*",
    "apps/*"
  ]
}
```

Bun supports the same workspace syntax as npm/yarn/pnpm.

**Check workspace dependencies:**
```bash
# Verify workspace structure
find . -name "package.json" -not -path "*/node_modules/*"
```

### 9. Install Dependencies with Bun

**Remove old lockfiles:**
```bash
rm -f package-lock.json yarn.lock pnpm-lock.yaml
```

**Install with Bun:**
```bash
bun install
```

This creates `bun.lockb` (Bun's binary lockfile).

**For workspaces:**
```bash
bun install --frozen-lockfile  # Equivalent to npm ci
```

### 10. Update Test Configuration

**If using Jest, migrate to Bun test:**

Create `bunfig.toml` for test configuration:

```toml
[test]
preload = ["./tests/setup.ts"]
coverage = true
coverageThreshold = 0.8

# New in Bun 1.3.13+
parallel = true
```

**Update test files:**
- Replace `import { test, expect } from '@jest/globals'`
- With `import { test, expect } from 'bun:test'`

**Jest compatibility notes:**
- Most Jest APIs work out of the box
- `jest.fn()` → Use `mock()` or `vi.fn()` from `bun:test`
- `jest.useFakeTimers()` → Use `useFakeTimers()` from `bun:test`
- Snapshot testing works the same
- Coverage reports may differ slightly

### 11. Update Environment Configuration

**Check .env files:**
```bash
ls -la | grep .env
```

Bun loads `.env` files automatically (same as dotenv package).

**Update environment loading code:**
- Remove `require('dotenv').config()`
- Bun loads `.env` by default

### 12. Update Build Configuration

**If using webpack/rollup/esbuild:**

Consider replacing with Bun's built-in bundler:

```typescript
// bun-build.ts
await Bun.build({
  entrypoints: ['./src/index.ts'],
  outdir: './dist',
  minify: true,
  splitting: true,
  sourcemap: 'external',
  target: 'bun',
});
```

**Update build script in package.json:**
```json
{
  "scripts": {
    "build": "bun run bun-build.ts"
  }
}
```

### 13. Update CI/CD Configuration

**GitHub Actions example:**

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
        with:
          bun-version: latest

      - run: bun install --frozen-lockfile

      - run: bun test --parallel

      - name: Type check
        run: bun run --bun tsc --noEmit

      - name: Build
        run: bun run build
```

### 14. Verify Migration with New Bun 1.4 Features

Run these commands to verify migration and leverage new features:

```bash
# 1. Check dependencies installed correctly
bun install

# 2. Run type checking
bun run --bun tsc --noEmit

# 3. Run tests (with parallel execution)
bun test --parallel

# 4. Try development server with HMR
bun run --hot src/index.ts

# 5. Test production build
bun run build

# 6. Try new Bun 1.4 APIs
bun repl
# > Bun.Image
# > Bun.WebView
# > Bun.markdown
# > Bun.cron
```

### 15. Replace Common npm Patterns with Bun 1.4 APIs

After migration, audit the codebase for Bun 1.4 replacement opportunities:

```bash
# Find sharp usage → replace with Bun.Image
grep -r "import.*sharp" src/

# Find marked/markdown-it → replace with Bun.markdown
grep -r "import.*marked\|markdown-it\|showdown" src/

# Find node-cron → replace with Bun.cron()
grep -r "import.*node-cron\|from 'cron'" src/

# Find ioredis → replace with Bun.redis
grep -r "import.*ioredis\|from 'redis'" src/

# Find pg/mysql → replace with Bun.SQL
grep -r "import.*pg\|from 'pg'\|from 'mysql'" src/

# Find node-pty → replace with Bun.Terminal
grep -r "import.*node-pty\|from 'node-pty'" src/
```

## Bun 1.4 Compatibility Matrix

| Framework/Tool | Bun 1.4 Status | Notes |
|---|---|---|
| **Playwright** | ✅ Runs on Bun | `connectOverCDP()`, Chromium on Windows |
| **vitest** | ✅ Runs on Bun | `--coverage`, threads, forks pools |
| **Next.js 16** | ✅ Works | `bun --bun next build` with Turbopack |
| **OpenTelemetry** | ✅ Works | `@opentelemetry/instrumentation-http/fs` |
| **dd-trace** | ✅ Works | `@datadog/pprof` profiles continuously |
| **Nuxt** | ✅ Works | HMR + Nuxt DevTools |
| **testcontainers** | ✅ Works | `container.exec()` |
| **@grpc/grpc-js** | ✅ Works | Servers behind Envoy |
| **TypeORM** | ✅ Works | Decorator settings in tsconfig |
| **nock** | ✅ Works | Intercepts `http` and `https` |
| **Fastify inject()** | ✅ Works | `light-my-request` works too |
| **happy-dom** | ✅ Works | No longer breaks `console.log` |
| **piscina** | ✅ Works | Worker thread pool |

## Common Migration Issues & Solutions

### Issue: Native Module Incompatibility

**Symptoms:**
```
error: Cannot find module "bcrypt"
```

**Solution:**
```bash
# Replace with pure JavaScript alternative
bun remove bcrypt
bun add bcryptjs

# Update imports
# Before: import bcrypt from 'bcrypt';
# After: import bcrypt from 'bcryptjs';
```

### Issue: ESM/CommonJS Conflicts

**Symptoms:**
```
error: require() of ES Module not supported
```

**Solution:**

Add to `package.json`:
```json
{
  "type": "module"
}
```

### Issue: Path Alias Resolution

**Symptoms:**
```
error: Cannot resolve "@/components"
```

**Solution:**

Verify `tsconfig.json` paths match:
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

Bun respects TypeScript path aliases automatically.

### Issue: Test Failures

**Symptoms:**
```
error: jest is not defined
```

**Solution:**

Update test imports:
```typescript
// Before
import { describe, it, expect } from '@jest/globals';

// After
import { describe, it, expect } from 'bun:test';
```

## Migration Checklist

Present this checklist to the user:

- [ ] Bun installed and verified (1.4+)
- [ ] Dependency compatibility analyzed
- [ ] Migration report reviewed
- [ ] Current state backed up (git commit/branch)
- [ ] Replaceable npm packages identified
- [ ] `package.json` scripts updated
- [ ] Replaceable dependencies removed
- [ ] `tsconfig.json` configured for Bun 1.4
- [ ] Old lockfiles removed
- [ ] Dependencies installed with `bun install`
- [ ] Bun native APIs replacing npm packages
- [ ] Test configuration migrated
- [ ] Tests passing with `bun test --parallel`
- [ ] Build process verified
- [ ] CI/CD updated for Bun
- [ ] Documentation updated
- [ ] Team notified of migration

## Post-Migration Performance Verification

After migration, help user verify performance improvements:

```bash
# Compare install times
time bun install  # Should be 3-10x faster than npm

# Compare test execution
time bun test --parallel     # Should be faster than Jest

# Compare startup time
time bun run src/index.ts  # Should be 50% faster on Windows

# Try new native APIs
bun repl
# > Bun.Image("photo.jpg").resize(1024).webp().write("thumb.webp")
# > Bun.markdown.html("# Hello **world**")
# > Bun.cron.parse("*/15 * * * *")
```

## Rollback Procedure

If migration encounters critical issues:

```bash
# Return to backup branch
git checkout backup-before-bun-migration

# Or restore original state
git reset --hard HEAD~1

# Reinstall original dependencies
npm install  # or yarn/pnpm
```

## Completion

Once migration is complete, provide summary:
- Migration status (success/partial/issues)
- List of npm packages replaced with Bun native APIs
- Performance improvements observed
- Any remaining manual steps
- Links to Bun documentation for ongoing development
