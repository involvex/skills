# CPU Profiling Reference

Profile CPU usage with Bun's built-in profiler.

## --cpu-prof

Generate a `.cpuprofile` file compatible with Chrome DevTools and VS Code:

```bash
bun --cpu-prof ./app.ts
```

Opens in:
- Chrome DevTools → Performance tab → Load profile
- VS Code → Run and Debug → Load `.cpuprofile`

## --cpu-prof-md

Generate a Markdown CPU profile report — optimized for terminal viewing and LLM prompts:

```bash
bun --cpu-prof-md ./app.ts
```

Output includes:
- Top functions by self time
- Call tree (total time)
- Function details with callers

Example output:
```markdown
# CPU Profile

| Duration | Samples | Interval | Functions |
| -------- | ------- | -------- | --------- |
| 304.9ms  | 279     | 1.0ms    | 6         |

**Top 10:** `tokenize` 39.1%, `escapeHtml` 25.6%, `render` 15.8%

## Hot Functions (Self Time)

| Self% | Self | Total% | Total | Function | Location |
| -----: | ---: | -----: | ----: | -------- | -------- |
| 39.1% | 119ms | 39.1% | 119ms | `tokenize` | `app.ts:14` |
| 25.6% | 78ms | 25.6% | 78ms | `escapeHtml` | `app.ts:5` |
```

## BUN_CPU_PROFILE

Enable CPU profiling via environment variable for processes you can't pass flags to:

```bash
BUN_CPU_PROFILE=1 bun ./app.ts
```

Writes `.cpuprofile` on exit. Useful for:
- Workers started by a framework
- Long-running services
- Processes managed by PM2/systemd

## node:inspector API

Start/stop profiling programmatically:

```typescript
import { Session } from "node:inspector";

const session = new Session();
session.connect();

session.post("Profiler.enable");
session.post("Profiler.start");

// ... do work ...

session.post("Profiler.stop");
const result = await new Promise((resolve) => {
  session.once("Profiler.stopProfile", resolve);
});

session.post("Profiler.disable");
session.disconnect();
```

## When to Use

| Tool | When |
|---|---|
| `--cpu-prof` | Need full profile in DevTools |
| `--cpu-prof-md` | Quick terminal analysis, paste into bug report/LLM |
| `BUN_CPU_PROFILE=1` | Can't modify command line args |
| `node:inspector` | Need programmatic control |
