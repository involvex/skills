# Memory Pressure Reference

Handle low-memory events from the operating system.

## process.on("memoryPressure")

```typescript
process.on("memoryPressure", (level) => {
  if (level === "critical") {
    cache.clear();
    pool.drainIdle();
    console.warn("Memory pressure: freed caches");
  } else {
    cache.shrink();
  }
});
```

## Levels

| Level | Meaning | Platforms |
|---|---|---|
| `"warning"` | Memory is getting low | macOS |
| `"critical"` | Memory is critically low | macOS, Linux, Windows |

## Platform Details

| Platform | Mechanism | Levels |
|---|---|---|
| macOS | `kqueue` + `EVFILT_MEMORYSTATUS` | `warning`, `critical` |
| Linux | `/proc/pressure/memory` PSI trigger | `critical` only |
| Windows | `CreateMemoryResourceNotification` | `critical` only |

## Use Cases

- Clear in-memory caches before OS kills the process
- Close idle database connections
- Stop idle workers
- Free large buffers

## Example: Cache Management

```typescript
const cache = new Map<string, { data: Buffer; size: number }>();
let totalSize = 0;
const MAX_CACHE_SIZE = 100 * 1024 * 1024; // 100 MB

process.on("memoryPressure", (level) => {
  if (level === "critical") {
    // Clear entire cache
    cache.clear();
    totalSize = 0;
  } else if (level === "warning") {
    // Shrink cache to 50%
    const target = MAX_CACHE_SIZE / 2;
    for (const [key, entry] of cache) {
      if (totalSize <= target) break;
      cache.delete(key);
      totalSize -= entry.size;
    }
  }
});
```

## Example: Connection Pool

```typescript
const pool = new ConnectionPool();

process.on("memoryPressure", async (level) => {
  if (level === "critical") {
    await pool.drainIdle();
    console.log(`Drained idle connections under memory pressure`);
  }
});
```

## When to Use

- Long-running services
- Applications with large in-memory caches
- Memory-intensive batch processing
- Any Bun process that should gracefully handle low memory
