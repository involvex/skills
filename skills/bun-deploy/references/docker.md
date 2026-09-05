# Docker Deployment Reference

Containerize Bun applications with official Docker images.

## Official Image

```dockerfile
FROM oven/bun:1.4

WORKDIR /app

# Copy package files
COPY package.json bun.lockb ./

# Install dependencies
RUN bun install --frozen-lockfile

# Copy source
COPY . .

# Run
CMD ["bun", "run", "start"]
```

## Multi-Stage Build

```dockerfile
# Build stage
FROM oven/bun:1.4 AS builder
WORKDIR /app
COPY package.json bun.lockb ./
RUN bun install --frozen-lockfile
COPY . .
RUN bun run build

# Production stage
FROM oven/bun:1.4 AS runner
WORKDIR /app

ENV NODE_ENV=production

RUN bun install --frozen-lockfile --production
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

CMD ["bun", "run", "start"]
```

## Alpine Variant

```dockerfile
FROM oven/bun:1.4-alpine

WORKDIR /app
COPY package.json bun.lockb ./
RUN bun install --frozen-lockfile
COPY . .

CMD ["bun", "run", "start"]
```

## With Standalone Executable

```dockerfile
FROM oven/bun:1.4 AS builder
WORKDIR /app
COPY . .
RUN bun build ./src/cli.ts --compile --target=bun-linux-x64 --outfile /app/myapp

FROM debian:bookworm-slim
COPY --from=builder /app/myapp /usr/local/bin/myapp
CMD ["myapp"]
```

## Docker Compose

```yaml
version: "3.8"
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://db:5432/myapp
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=myapp
```

## Best Practices

1. **Use multi-stage builds**: Separate build and runtime
2. **Use `--frozen-lockfile`**: Reproducible installs
3. **Use specific Bun version**: Pin `oven/bun:1.4.1` not `latest`
4. **Use Alpine**: Smaller image size
5. **Run as non-root**: Add `USER` directive

## Image Sizes

| Image | Size |
|---|---|
| `oven/bun:1.4` | ~84 MB |
| `oven/bun:1.4-alpine` | ~40 MB |
| `oven/bun:1.4-slim` | ~60 MB |
