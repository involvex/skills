# Bun Workspaces Reference

Workspace patterns for monorepos with `bun install`.

## Basic Workspace Config

```json
{
  "name": "my-monorepo",
  "workspaces": ["packages/*", "apps/*"]
}
```

## Install All Workspaces

```bash
bun install
```

## Filter Workspaces

```bash
# Specific workspace
bun install --filter="./packages/web"

# Multiple workspaces
bun install --filter="./packages/web" --filter="./packages/api"

# All workspaces
bun install --filter "*"

# Exclude workspace
bun install --filter "!./packages/legacy"
```

## Run Scripts in Workspaces

```bash
# Run script in all workspaces
bun run --parallel --filter '*' build

# Run in specific workspace
bun run --filter "./packages/web" dev

# Run in workspaces matching pattern
bun run --parallel --filter "apps-*" test
```

## Workspace Dependencies

```json
{
  "name": "packages/web",
  "dependencies": {
    "@my-org/api": "workspace:*"
  }
}
```

## Cross-Workspace Scripts

```bash
# Run test in all workspaces in parallel
bun run --parallel --filter '*' test

# Run build sequentially (respects dependencies)
bun run --sequential --filter '*' build
```

## Project Structure

```
monorepo/
├── package.json          # Root with workspaces
├── packages/
│   ├── web/
│   │   ├── package.json
│   │   └── src/
│   ├── api/
│   │   ├── package.json
│   └── shared/
│       ├── package.json
└── apps/
    └── docs/
        ├── package.json
```

## Best Practices

1. Keep workspace packages focused and independent
2. Use `workspace:*` protocol for internal dependencies
3. Use `--filter` to avoid installing everything
4. Use `--parallel` for independent tasks, `--sequential` for dependent ones
