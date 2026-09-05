# Audit and Cleanup Reference

Security auditing, deduplication, and cleanup commands for `bun install`.

## Security Audit

```bash
# Scan for vulnerabilities
bun audit

# Output:
# ┌───────────────┬──────────────────────────────────────────────────────────────┐
# │ moderate      │ Prototype Pollution in lodash                          │
# │ ...           │ ...                                                        │
# └───────────────┴──────────────────────────────────────────────────────────────┘
```

## Auto-Fix Vulnerabilities

```bash
# Fix all fixable vulnerabilities
bun audit fix

# Fix only production dependencies
bun audit fix --production

# Dry run (show what would be fixed)
bun audit fix --dry-run
```

## Dedupe

Flatten nested dependencies to reduce duplication:

```bash
bun dedupe
```

This merges duplicate packages in `node_modules` and updates `bun.lockb`.

## Prune

Remove extraneous packages not in `package.json`:

```bash
bun prune
```

## Why

Check why a package is installed:

```bash
bun why <package-name>
```

Output shows the dependency chain that brought in the package.

## Outdated

Check for outdated packages:

```bash
# All packages
bun outdated

# Specific package
bun outdated <package-name>

# JSON output
bun outdated --json
```

## Update Dependencies

```bash
# Update all packages within semver ranges
bun update

# Update specific package
bun update <package-name>

# Update to latest (ignore lockfile ranges)
bun update <package-name> --latest

# Update all to latest
bun update --latest
```

## CI/CD Integration

```bash
# Install with audit in CI
bun install --frozen-lockfile
bun audit

# Auto-fix in CI (if desired)
bun audit fix
bun install --frozen-lockfile
```

## Package Manager Info

```bash
# View package metadata
bun pm view <package-name>

# List installed packages
bun pm list

# List outdated packages
bun outdated
```
