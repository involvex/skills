# Bun Install Linker Reference

Choose between hoisted (default) and isolated (pnpm-style) linking.

## Hoisted (Default)

The default mode. Similar to npm/yarn — packages are hoisted to the root `node_modules`.

```bash
bun install --linker=hoisted
```

**Pros:**
- Fast installs
- Compatible with most tooling
- Familiar to npm/yarn users

**Cons:**
- Can have phantom dependencies
- Less isolation between workspaces

## Isolated (pnpm-style)

Packages are stored in isolated `node_modules` with strict dependency isolation.

```bash
bun install --linker=isolated
```

Or in `bunfig.toml`:

```toml
[install]
linker = "isolated"
```

**Pros:**
- Strict isolation — no phantom dependencies
- Similar to pnpm's layout
- Better for monorepos

**Cons:**
- Slightly slower installs
- Some tools may need configuration

## Global Virtual Store

Both modes support the global store for faster installs:

```bash
bun install --global-store
```

```toml
[install]
globalStore = true
```

The global store deduplicates packages across all projects.

## Comparison

| Feature | Hoisted | Isolated |
|---|---|---|
| Speed | Fast | Slightly slower |
| Isolation | Moderate | Strict |
| Disk usage | Higher | Lower (deduped) |
| pnpm compatibility | Low | High |
| npm compatibility | High | Moderate |

## When to Use Isolated

- Monorepos with many workspaces
- Teams migrating from pnpm
- Need strict dependency isolation
- Want to avoid phantom dependencies

## When to Use Hoisted

- Simple single-package projects
- Need maximum install speed
- Using tools that expect hoisted layout
- Migrating from npm/yarn
