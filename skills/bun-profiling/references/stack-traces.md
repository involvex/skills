# Async Stack Traces and Inspector Reference

## Async Stack Traces

Bun 1.4+ provides async stack traces that point back to the `await` in your code, not at native frames.

```typescript
async function main() {
  await fs.promises.readFile("file.txt"); // Error points here
}
```

Works for:
- `fs.promises`
- `fetch()`
- S3 operations
- DNS lookups
- `crypto` operations
- `Bun.file()`

## node:inspector Profiler API

Use the `Profiler` class for programmatic profiling:

```typescript
import { Profiler } from "node:inspector";

const profiler = new Profiler();

// Start profiling
profiler.start();

// ... do work ...

// Stop and get profile
const profile = await profiler.stop();
console.log(profile); // CPU profile data
```

## Performance Monitoring

```typescript
// Monitor event loop lag
setInterval(() => {
  const lag = performance.now() - Date.now();
  if (lag > 100) {
    console.warn(`Event loop lag: ${lag}ms`);
  }
}, 1000);
```

## Debugging Tips

1. **Use async stack traces** — they show exactly where the `await` is
2. **Use `--cpu-prof-md`** for quick terminal analysis
3. **Use Chrome DevTools** for deep profiling
4. **Use `node:inspector`** when you need programmatic control
