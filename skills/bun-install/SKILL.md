---
name: bun-install
description: Install and manage dependencies with Bun's fast package manager. Use when installing packages, managing workspaces, troubleshooting lockfiles, or using workspace filtering.
compatibility: Requires Bun 1.4+
allowed-tools: ["Bash", "Write", "Read"]
metadata:
  author: dale
  category: bun-runtime
  tags: "bun, install, dependencies, workspaces, lockfile, linker"
---

# Bun Install & Dependency Management

Bun's package manager is 3-10x faster than npm, with a binary lockfile (`bun.lockb`) and workspace support.

## What's new in Bun 1.4

- **`--linker=isolated`** — pnpm-style isolated `node_modules` (1.3.14)
- **Global virtual store** — 7x faster installs by sharing packages across workspaces (1.3.14)
- **`--offline` / `--prefer-offline`** — install without network or prefer cache (1.4.1)
- **`bun audit fix`** — scan and fix vulnerabilities (1.4.0)
- **`bun dedupe`** — deduplicate dependencies (1.4.0)
- **`bun prune`** — remove extraneous packages (1.4.0)
- **`bun why`** — understand why a dependency is installed (1.2.19)
- **`bun outdated`** — check for outdated packages (1.2.19)
- **`--filter`** — run commands in workspace subsets (1.2.16)
- **Self-contained workspace `node_modules`** — each workspace gets its own `node_modules` (1.4.1)

## Core Workflow

### 1. Check Prerequisites

```bash
# Verify Bun installation
bun --version

# Check existing lockfile
ls -la bun.lockb package.json
```

### 2. Install Dependencies

```bash
# Install all dependencies
bun install

# Install with frozen lockfile (CI)
bun install --frozen-lockfile

# Install offline (no network)
bun install --offline

# Prefer offline, fall back to network
bun install --prefer-offline
```

### 3. Add / Remove Packages

```bash
# Add dependency
bun add express

# Add dev dependency
bun add -d @types/express

# Add from specific workspace (monorepo)
bun add react --workspace=apps/web

# Remove dependency
bun remove express

# Update dependency
bun update express
```

### 4. Workspace Commands

```bash
# Run script in all workspaces
bun run --filter '*' build

# Run script matching a pattern
bun run --filter "apps/*" test

# Filter by dependency
bun run --filter "...web" dev  # Workspaces that depend on "web"

# Parallel execution
bun run --parallel --filter '*' build

# Sequential execution
bun run --sequential --filter '*' test
```

### 5. Audit and Fix

```bash
# Scan for vulnerabilities
bun audit

# Fix vulnerabilities automatically
bun audit fix

# Deduplicate dependencies
bun dedupe

# Prune extraneous packages
bun prune

# Check why a package is installed
bun why lodash

# Check outdated packages
bun outdated

# Update outdated packages
bun outdated --update
```

### 6. Linker Configuration

#### Hoisted Linker (Default)

```bash
# Default — hoisted like npm
bun install --linker=hoisted
```

#### Isolated Linker (pnpm-style)

```bash
# pnpm-style isolated node_modules
bun install --linker=isolated
```

Configure in `bunfig.toml`:
```toml
[install]
linker = "isolated"  # or "hoisted"
```

Isolated linker benefits:
- Strict dependency isolation
- Prevents phantom dependencies
- Faster installs with global virtual store

### 7. Global Virtual Store (1.3.14+)

Enable for 7x faster installs in workspaces:

```toml
# bunfig.toml
[install]
linker = "isolated"
isolatedStore = true  # Global virtual store
```

### 8. Workspace Configuration

In `package.json`:
```json
{
  "workspaces": [
    "packages/*",
    "apps/*"
  ]
}
```

Or in `bunfig.toml`:
```toml
[workspace]
protocols = ["workspace:*"]  # Resolve workspace: protocol
```

### 9. bunfig.toml Configuration

```toml
[install]
# Linker mode
linker = "hoisted"  # or "isolated"

# Global virtual store (1.3.14+)
isolatedStore = true

# Cache directory
cacheDir = "~/.bun/install/cache"

# Frozen lockfile for CI
frozenLockfile = false

# Exact versions (no caret/tilde)
exact = false

# Use system CA certificates
useSystemCAs = true

# Production install
production = false
```

## Migration from npm/pnpm/yarn

| npm | Bun | Notes |
|-----|-----|-------|
| `npm install` | `bun install` | Same behavior |
| `npm install --frozen-lockfile` | `bun install --frozen-lockfile` | Same flag |
| `npm install --offline` | `bun install --offline` | Same flag (1.4.1+) |
| `npm audit fix` | `bun audit fix` | Same behavior |
| `npm dedupe` | `bun dedupe` | Same behavior |
| `npm prune` | `bun prune` | Same behavior |
| `pnpm install --frozen-lockfile` | `bun install --frozen-lockfile` | Same flag |
| `yarn install --frozen-lockfile` | `bun install --frozen-lockfile` | Same flag |

## Performance Notes

- 3-10x faster installs than npm
- Binary lockfile (`bun.lockb`) is faster to read/write
- Global virtual store reduces disk usage and speeds up workspace installs
- Streams tarballs to disk using 17x less memory (1.3.13)

## Troubleshooting

### Lockfile Issues

```bash
# Regenerate lockfile
rm bun.lockb
bun install

# Verify lockfile integrity
bun install --frozen-lockfile
```

### Workspace Resolution Issues

```bash
# Ensure workspaces are defined in package.json
# Check that workspace packages have correct names

# Re-install with verbose output
bun install --verbose
```

### Offline Install Fails

```bash
# Ensure cache exists
ls ~/.bun/install/cache

# Fall back to online install
bun install --prefer-offline
```
