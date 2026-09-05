# Process Management Reference

## --no-orphans

Kill all child processes when the parent exits:

```bash
bun --no-orphans ./app.ts
```

On Linux, macOS, and Windows:
- Bun exits when its parent dies
- SIGKILLs every descendant on exit

Useful for:
- Development servers
- Worker processes
- CI/CD pipelines

## --no-env-file

Skip automatic `.env` loading:

```bash
bun --no-env-file ./app.ts
```

Or in `bunfig.toml`:

```toml
[run]
env = false
```

## Process Events

```typescript
// Memory pressure
process.on("memoryPressure", (level) => {
  console.log(`Memory pressure: ${level}`);
});

// Exit handling
process.on("exit", (code) => {
  console.log(`Process exiting with code ${code}`);
});
```

## Process Info

```typescript
// Memory usage
const mem = process.memoryUsage();
console.log(mem.rss, mem.heapUsed, mem.external);

// CPU usage
const cpu = process.cpuUsage();
console.log(cpu.user, cpu.system);
```
