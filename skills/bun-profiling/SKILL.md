---
name: bun-profiling
description: Profile Bun applications using built-in CPU and heap profilers, markdown reports, async stack traces, and memory pressure monitoring. Use when debugging performance issues, memory leaks, or slow execution.
compatibility: Requires Bun 1.4+
allowed-tools: ["Bash", "Write", "Read"]
metadata:
  author: dale
  category: bun-runtime
  tags: "bun, profiling, performance, cpu, heap, memory, debugging"
---

# Bun Profiling & Debugging

Bun ships with built-in profilers that output standard formats compatible with Chrome DevTools, plus Markdown reports for terminal and LLM-friendly analysis.

## What's new in Bun 1.4

- **`--cpu-prof-md`** — CPU profile as Markdown for terminal/LLM analysis (1.3.7)
- **`--heap-prof-md`** — Heap profile as Markdown (1.3.7)
- **`BUN_CPU_PROFILE=1`** — Profile processes you can't pass flags to (1.4.0)
- **`node:inspector` Profiler API** — start/stop CPU profiles programmatically (1.4.0)
- **Async stack traces** — errors from async APIs point to the `await` in your code (1.3.12)
- **`process.on("memoryPressure")`** — OS memory pressure events (1.4.0)
- **`--no-orphans`** — exit when parent dies, SIGKILL descendants (1.3.2)

## CPU Profiling

### CLI: `.cpuprofile` for Chrome DevTools

```bash
bun --cpu-prof ./app.ts
```

Outputs a `.cpuprofile` file. Open in Chrome DevTools or VS Code.

### CLI: Markdown CPU Report

```bash
bun --cpu-prof-md ./app.ts
```

Outputs a Markdown report with:
- Top 10 hot functions by self time
- Call tree (total time)
- Function details with callers

### Environment Variable Profiling

Profile a process you can't pass flags to:

```bash
BUN_CPU_PROFILE=1 node app.js
# or
BUN_CPU_PROFILE=1 bun run app.ts
```

### Programmatic Profiling

```typescript
import { Session } from "node:inspector";

const session = new Session();
session.connect();

// Start profiling
session.post("Profiler.enable");
session.post("Profiler.start");

// ... run your code ...

// Stop and get profile
const { profile } = await new Promise((resolve) => {
  session.post("Profiler.stop", (err, result) => resolve(result));
});

session.disconnect();
```

## Heap Profiling

### CLI: `.heapsnapshot` for Chrome DevTools

```bash
bun --heap-prof ./app.ts
```

Outputs a V8-compatible `.heapsnapshot`. Open in Chrome DevTools.

### CLI: Markdown Heap Report

```bash
bun --heap-prof-md ./app.ts
```

Outputs a Markdown report with:
- Top 50 types by retained size
- Largest objects
- GC roots
- Retention chains

## Memory Pressure (Bun 1.4.0+)

React to OS memory pressure events:

```typescript
process.on("memoryPressure", (level) => {
  if (level === "critical") {
    cache.clear();
    pool.drainIdle();
  } else {
    cache.shrink(0.5);
  }
});
```

- **macOS**: `kqueue` with `EVFILT_MEMORYSTATUS` — `level` is `"warning"` or `"critical"`
- **Linux**: PSI trigger — `level` is `"critical"`
- **Windows**: `CreateMemoryResourceNotification` — `level` is `"critical"`

## Async Stack Traces (Bun 1.3.12+)

Errors from async native APIs point back to the `await` in your code:

```typescript
// Before Bun 1.3.12: stack trace points at internal native frames
// After Bun 1.3.12: stack trace points at the `await` in your code

async function loadData() {
  const data = await fs.promises.readFile("data.json"); // ← points here
  return JSON.parse(data);
}
```

Works with:
- `fs.promises`
- `fetch()`
- S3
- DNS
- `crypto`

## Orphan Process Management (Bun 1.3.2+)

```bash
# Exit when parent process dies
bun --no-orphans app.ts
```

Useful for CI/CD and containerized environments.

## Performance Comparison (Bun 1.4.0+)

| Metric | Bun 1.3 | Bun 1.4 | Improvement |
|--------|---------|---------|-------------|
| Idle CPU (hello world) | Baseline | -5x | 5x lower |
| HTTP server memory | 135 MB | 81 MB | -40% |
| Windows startup | 39 ms | 15.5 ms | 2.5x faster |
| Binary size (Windows) | 93.9 MB | 84.8 MB | -10% |

## Debugging Workflow

1. **CPU bottleneck**: Run with `--cpu-prof-md`, identify hot functions
2. **Memory leak**: Run with `--heap-prof-md`, check for growing object counts
3. **Memory pressure**: Add `process.on("memoryPressure")` handler
4. **Async errors**: Check stack traces — should point to `await` lines
5. **Orphan processes**: Use `--no-orphans` in CI

## Completion Checklist

- ✅ CPU profile generated (if performance issue)
- ✅ Heap profile generated (if memory issue)
- ✅ Memory pressure handler added (if applicable)
- ✅ Async stack traces verified
- ✅ Performance improvements measured
