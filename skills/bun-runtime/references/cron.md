# Bun.cron() — Scheduled Jobs

OS-level cron scheduling and in-process job scheduler.

## Quick Start

```typescript
// OS-level cron job (crontab on Linux, launchd on macOS, Task Scheduler on Windows)
await Bun.cron("./worker.ts", "30 2 * * MON", "weekly-report");

// In-process scheduler
using job = Bun.cron("*/5 * * * *", async () => {
  await cleanupTempFiles();
});

// Parse cron expression
const next = Bun.cron.parse("*/15 * * * *");
```

## API Reference

### Bun.cron(path, expression, name?)

Register an OS-level scheduled job that runs a file:

```typescript
await Bun.cron(
  path: string,       // Path to the worker file
  expression: string, // Cron expression
  name?: string       // Job name
);
```

The worker file exports a `scheduled` handler:

```typescript
// worker.ts
export default {
  async scheduled(controller) {
    // controller.cron === "30 2 * * 1"
    // controller.scheduledTime === 1737340200000
    await doWork();
  },
};
```

### Bun.cron(expression, callback)

In-process scheduler (no system cron involved):

```typescript
using job = Bun.cron("*/5 * * * *", async () => {
  await cleanupTempFiles();
});
```

Jobs never overlap. Use `using` to stop the job when it goes out of scope:

```typescript
{
  using job = Bun.cron("*/5 * * * *", async () => {
    await doWork();
  });
  // Job runs in this scope...
}
// Job is stopped here automatically
```

### Job Object

```typescript
const job = Bun.cron("0 * * * *", async () => {});
job.cron;    // "0 * * * *"
job.unref(); // Allow process to exit without waiting
job.stop();  // Cancel the job
```

### Bun.cron.parse()

Parse a cron expression and get the next matching UTC Date:

```typescript
const next = Bun.cron.parse("*/15 * * * *");
console.log(next.toISOString()); // 2026-01-01T00:00:00.000Z

// With a from timestamp
const next = Bun.cron.parse("0 9 * * 1-5", {
  from: new Date("2026-01-01"),
});
```

## Cron Syntax

Standard 5-field cron syntax:

```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6) (Sunday to Saturday)
│ │ │ │ │
* * * * *
```

Named days: `MON`, `TUE`, `WED`, `THU`, `FRI`, `SAT`, `SUN`

Special strings: `@yearly`, `@monthly`, `@weekly`, `@daily`, `@hourly`

## Timezone Support

```typescript
// Schedule in a specific timezone
await Bun.cron("0 9 * * 1-5", async () => {
  // Runs at 9 AM in the specified timezone
}, "morning-report", { tz: "America/New_York" });
```

## Common Patterns

### Daily Cleanup

```typescript
using job = Bun.cron("0 3 * * *", async () => {
  await cleanupTempFiles();
  await clearExpiredSessions();
});
```

### Every 15 Minutes

```typescript
using job = Bun.cron("*/15 * * * *", async () => {
  await syncExternalData();
});
```

### Weekdays at 9 AM

```typescript
using job = Bun.cron("0 9 * * 1-5", async () => {
  await sendDailyReport();
});
```

### Migration from node-cron

| node-cron | Bun.cron() |
|-----------|-----------|
| `cron.schedule('*/5 * * * *', fn)` | `using job = Bun.cron('*/5 * * * *', fn)` |
| `cron.stop()` | `job.stop()` |
| — | `job.unref()` |
| `cron.parse(expr)` | `Bun.cron.parse(expr)` |
