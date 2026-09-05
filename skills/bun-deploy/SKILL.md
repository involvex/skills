---
name: bun-deploy
description: Deploy Bun applications as standalone executables, self-contained HTML bundles, or containerized services. Use when building for production, creating CLI binaries, or deploying Bun apps.
compatibility: Requires Bun 1.4+
allowed-tools: ["Bash", "Write", "Read"]
metadata:
  author: dale
  category: bun-runtime
  tags: [bun, deploy, compile, executable, docker, standalone, production]
---

# Bun Deployment

Deploy Bun applications as standalone executables, self-contained HTML bundles, or containerized services.

## What's new in Bun 1.4

- **`--compile --target=bun-*`** — compile to standalone executable (1.4.0)
- **`--bytecode`** — cross-compile to bytecode for faster startup (1.4.1)
- **`--target=browser`** — self-contained HTML bundle (1.3.10)
- **Binary size reduced** — Windows binary 10% smaller (1.4.0)
- **50% faster startup on Windows** — 15.5ms vs 39ms (1.4.0)

## Deployment Options

| Target | Output | Use Case |
|--------|--------|----------|
| Source | `.ts` / `.js` files | Server deployment (Node.js-compatible) |
| Standalone executable | `.exe` / binary | CLI tools, desktop apps |
| Self-contained HTML | `.html` file | Static hosting, demos |
| Docker container | `Dockerfile` | Server deployment |
| Bytecode | `.bun` files | Fast-start, cross-platform |

## Core Workflow

### 1. Choose Deployment Target

Ask the user:
- **CLI tool?** → Standalone executable (`--compile`)
- **Web app?** → Docker container or source deployment
- **Static site?** → Self-contained HTML (`--target=browser`)
- **Library?** → Source or bundled ESM/CJS

### 2. Standalone Executable (CLI Tools)

```bash
# Build executable for current platform
bun build ./src/cli.ts --compile --outfile ./dist/cli.exe

# Build for specific platform
bun build ./src/cli.ts --compile --target=bun-windows --outfile ./dist/cli.exe
bun build ./src/cli.ts --compile --target=bun-linux --outfile ./dist/cli
bun build ./src/cli.ts --compile --target=bun-macos --outfile ./dist/cli
```

Available targets:
| Target | Platform |
|--------|----------|
| `bun` | Current platform |
| `bun-windows` | Windows x64/arm64 |
| `bun-linux` | Linux x64/arm64 |
| `bun-macos` | macOS arm64/x64 |

### 3. Bytecode Compilation (1.4.1+)

Cross-compile to bytecode for faster startup:

```bash
# Compile to bytecode
bun build ./src/index.ts --bytecode --outdir ./dist

# Cross-compile bytecode for another platform
bun build ./src/index.ts --bytecode --target=bun-linux --outdir ./dist
```

### 4. Self-Contained HTML (Static Sites)

```bash
# Build self-contained HTML
bun build ./src/index.ts --target=browser --outdir ./dist
```

Produces a single `.html` file with the runtime and bundle inlined. No server required.

### 5. Docker Deployment

```dockerfile
# Dockerfile
FROM oven/bun:1.4.1-alpine

WORKDIR /app

# Install dependencies
COPY package.json bun.lockb ./
RUN bun install --frozen-lockfile

# Copy source
COPY . .

# Build (optional)
RUN bun run build

# Run
EXPOSE 3000
CMD ["bun", "run", "start"]
```

Build and run:
```bash
docker build -t my-bun-app .
docker run -p 3000:3000 my-bun-app
```

### 6. Source Deployment (Node.js-compatible)

Deploy as regular JavaScript/TypeScript:

```bash
# Build for Node.js target
bun build ./src/server.ts --outdir ./dist --target=node --format=esm

# Run with Node.js or Bun
node dist/server.js
# or
bun run dist/server.js
```

### 7. Package.json Scripts

```json
{
  "scripts": {
    "dev": "bun run --hot src/index.ts",
    "build": "bun build src/index.ts --outdir=dist --minify",
    "build:exe": "bun build src/cli.ts --compile --outfile=dist/cli.exe",
    "build:html": "bun build src/index.ts --target=browser --outdir=dist",
    "build:bytecode": "bun build src/index.ts --bytecode --outdir=dist",
    "start": "bun run dist/index.js",
    "docker:build": "docker build -t my-bun-app .",
    "docker:run": "docker run -p 3000:3000 my-bun-app"
  }
}
```

## Performance Characteristics

| Deployment | Startup | Memory | Binary Size |
|-----------|---------|--------|-------------|
| Source (bun run) | 15.5ms | 16.8 MB | — |
| Standalone exe | ~5ms | ~8 MB | 75-85 MB |
| Self-contained HTML | N/A | Browser | ~2 MB |
| Docker (bun:alpine) | 15.5ms | 16.8 MB | ~80 MB image |

## Troubleshooting

### Compile fails

```bash
# Ensure entry file exists and is valid TypeScript
bun build ./src/cli.ts --compile --outfile ./dist/cli.exe

# Check for native dependencies (not supported in standalone)
# Standalone executables cannot use native addons
```

### Binary too large

```bash
# Use --bytecode for smaller distribution
bun build ./src/cli.ts --compile --bytecode --outfile ./dist/cli
```

### Docker image too large

```dockerfile
# Use Alpine-based image
FROM oven/bun:1.4.1-alpine

# Multi-stage build for even smaller images
FROM oven/bun:1.4.1-alpine AS builder
WORKDIR /app
COPY package.json bun.lockb ./
RUN bun install --frozen-lockfile
COPY . .
RUN bun run build

FROM oven/bun:1.4.1-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
CMD ["bun", "run", "start"]
```

## Completion Checklist

- ✅ Deployment target chosen
- ✅ Build command verified
- ✅ Binary/bundle tested
- ✅ Dockerfile created (if applicable)
- ✅ CI/CD pipeline updated
