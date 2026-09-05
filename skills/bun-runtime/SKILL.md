---
name: bun-runtime
description: Use Bun's built-in runtime APIs — Bun.Image, Bun.WebView, Bun.markdown, Bun.cron, Bun.Terminal, Bun.redis, Bun.SQL, Bun.Archive, JSON5/JSONL/JSONC, streams, and more. Use when image processing, browser automation, markdown rendering, scheduled jobs, PTY, Redis, SQL, archives, or native Bun APIs are needed.
compatibility: Requires Bun 1.4+
allowed-tools: ["Bash", "Write", "Read"]
metadata:
  author: dale
  category: bun-runtime
  tags: [bun, runtime, native-apis, image, webview, markdown, cron, terminal, redis, sql, archive]
---

# Bun Native Runtime APIs

Bun 1.4 ships a comprehensive set of built-in runtime APIs that replace entire categories of npm dependencies. Use these for faster, zero-dependency implementations.

## Quick Reference

| API | Replaces | Version |
|---|---|---|
| `Bun.Image` | `sharp`, `jimp`, `gm` | 1.3.14+ |
| `Bun.WebView` | `puppeteer`, `playwright` | 1.3.12+ |
| `Bun.markdown` | `marked`, `markdown-it` | 1.3.8+ |
| `Bun.cron()` | `node-cron`, `cron` | 1.3.11+ |
| `Bun.Terminal` | `node-pty` | 1.3.5+ |
| `Bun.redis` | `ioredis`, `redis` | 1.2.9+ |
| `Bun.SQL` | `pg`, `mysql`, `sqlite3` | 1.2.21+ |
| `Bun.Archive` | `tar`, `archiver` | 1.3.6+ |
| `Bun.JSON5` | `json5` | 1.3.7+ |
| `Bun.JSONL` | `jsonl-parse`, `ndjson` | 1.3.7+ |
| `Bun.JSONC` | `jsonc-parser` | 1.3.6+ |
| `Bun.wrapAnsi` | `wrap-ansi` | 1.3.7+ |
| `Bun.stripANSI` | `strip-ansi` | 1.3.11+ |
| `process.on("memoryPressure")` | — | 1.4.0+ |
| Native streams | `readable-stream` | 1.4.0+ |
| `CompressionStream`/`DecompressionStream` | `pako`, `fflate` | 1.3.3+, native 1.4.0+ |

## Core APIs

### Bun.Image — Image Processing (1.3.14+)

Decode, resize, rotate, and encode images without `sharp` or any native addon.

```typescript
// Resize and convert
await Bun.file("photo.jpg")
  .image()
  .resize(1024, 1024, { fit: "inside" })
  .rotate(90)
  .webp({ quality: 85 })
  .write("thumb.webp");

// Stream into a Response
return new Response(new Bun.Image(upload).resize(200).jpeg());

// Get image metadata
const image = new Bun.Image(await Bun.file("photo.jpg").arrayBuffer());
console.log(image.width, image.height);

// Supported formats: JPEG, PNG, WebP, GIF, BMP
// HEIC, AVIF, TIFF: macOS and Windows only
```

**Key methods:**
- `.resize(width, height, options?)` — Resize with fit modes: `"inside"`, `"outside"`, `"fill"`, `"stretch"`
- `.rotate(degrees)` — Rotate 90, 180, 270
- `.flip(horizontal?, vertical?)` — Flip image
- `.jpeg(options?)` — Encode as JPEG
- `.png(options?)` — Encode as PNG
- `.webp(options?)` — Encode as WebP (quality 0-100)
- `.gif(options?)` — Encode as GIF
- `.bmp()` — Encode as BMP
- `.toBuffer()` — Get encoded buffer
- `.toFile(path)` — Save to file
- `.arrayBuffer()` — Get raw pixels as ArrayBuffer

### Bun.WebView — Headless Browser Automation (1.3.12+)

Navigate, click, scroll, run JavaScript, and take screenshots without Puppeteer or Playwright.

```typescript
await using view = new Bun.WebView({ width: 800, height: 600 });
await view.navigate("https://bun.sh");
await view.click("a[href='/docs']");
const title = await view.evaluate("document.title");
await Bun.write("page.png", await view.screenshot());

// Raw CDP access
await view.cdp("Browser.getVersion");
```

**Key methods:**
- `.navigate(url)` — Navigate to URL
- `.click(selector)` — Click element (real user input)
- `.scroll(x, y)` — Scroll page
- `.evaluate(js)` — Run JavaScript, return result
- `.screenshot(options?)` — Get Blob screenshot
- `.cdp(method, params?)` — Raw Chrome DevTools Protocol
- `.close()` — Dispose the WebView

**Options:**
```typescript
new Bun.WebView({
  width: 800,
  height: 600,
  url: "https://bun.sh",  // Initial URL
  backgroundColor: "#ffffff",
})
```

**Platform notes:**
- macOS: Uses system WebKit (no install needed)
- macOS/Linux/Windows: Can also drive Chrome, Chromium, or Edge

### Bun.markdown — Markdown Processing (1.3.8+)

Built-in CommonMark-compliant Markdown parser.

```typescript
// HTML output
const html = Bun.markdown.html("# Hello **world**");
// "<h1>Hello <strong>world</strong></h1>\n"

// Terminal/ANSI output
const ansi = Bun.markdown.render("# Hello\n\n**bold**", {
  heading: (children) => `\x1b[1;4m${children}\x1b[0m\n`,
  paragraph: (children) => children + "\n",
  strong: (children) => `\x1b[1m${children}\x1b[22m`,
});

// React elements
const elements = Bun.markdown.react(readmeContent);
```

**Key methods:**
- `.html(markdown)` → HTML string
- `.render(markdown, callbacks)` → Custom rendering (terminal, etc.)
- `.react(markdown, components?)` → React elements

**Supported:** GFM tables, strikethrough, task lists, autolinks, linear-time parsing.

**Warning:** HTML output is not sanitized. Raw HTML, event handlers, and `javascript:` hrefs pass through verbatim.

### Bun.cron() — Scheduled Jobs (1.3.11+)

OS-level cron jobs and in-process scheduling with Cloudflare Workers-style handlers.

```typescript
// OS-level cron job
await Bun.cron("./worker.ts", "30 2 * * MON", "weekly-report");

// In-process scheduler (no system cron)
using job = Bun.cron("*/5 * * * *", async () => {
  await cleanupTempFiles();
});

// Parse cron expressions
const next = Bun.cron.parse("*/15 * * * *");
console.log(next.toISOString()); // Next matching UTC date

// worker.ts — Cloudflare Workers-style handler
export default {
  async scheduled(controller) {
    console.log(controller.cron);        // "30 2 * * 1"
    console.log(controller.scheduledTime); // timestamp
    await doWork();
  },
};
```

**Key methods:**
- `Bun.cron(path, expression, name?)` — OS-level cron job
- `Bun.cron(expression, fn)` — In-process scheduler
- `Bun.cron.parse(expression)` — Parse cron → next Date
- `job.stop()` — Cancel scheduled job
- `job.unref()` — Allow process exit

**Features:**
- Jobs never overlap
- `using` stops the job on scope exit
- Timezone support: `Bun.cron(expr, fn, { tz: "America/New_York" })`
- Standard 5-field cron + named days + `@daily`, `@hourly`, etc.

### Bun.Terminal — PTY Support (1.3.5+)

Drive terminal applications from JavaScript without `node-pty`.

```typescript
const proc = Bun.spawn(["bash"], {
  terminal: {
    cols: 80,
    rows: 24,
    data(term, data) {
      process.stdout.write(data);
    },
  },
});

proc.terminal.write("echo Hello from PTY!\n");
proc.terminal.resize(120, 40);
```

**Key features:**
- Works on Linux, macOS, and Windows
- `data(term, data)` callback receives terminal output
- `.write(text)` sends input to the PTY
- `.resize(cols, rows)` resizes the terminal
- Supports colors, Unicode, cursor control

### Bun.redis — Redis Client (1.2.9+)

Built-in Redis client with Pub/Sub support.

```typescript
const redis = new Bun.Redis("redis://localhost:6379");

// Key-value operations
await redis.set("key", "value");
const val = await redis.get("key");

// Pub/Sub
const sub = redis.subscribe("channel");
sub.addEventListener("message", (event) => {
  console.log("Received:", event.data);
});
await redis.publish("channel", "hello");

// Hash operations
await redis.hset("user:1", { name: "Alice", age: "30" });
const user = await redis.hgetall("user:1");
```

### Bun.SQL — Unified SQL (1.2.21+)

PostgreSQL, MySQL, and SQLite with a unified API.

```typescript
// PostgreSQL
const pg = Bun.sql`postgres://localhost/mydb`;
const users = await pg`SELECT * FROM users WHERE age > ${18}`;

// MySQL
const mysql = Bun.sql`mysql://localhost/mydb`;
const rows = await mysql`SHOW TABLES`;

// SQLite (built-in, no server needed)
const sqlite = Bun.sql`sqlite:mydb.db`;
await sqlite`CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)`;
await sqlite`INSERT INTO users VALUES (${1}, ${"Alice"})`;
const user = await sqlite`SELECT * FROM users WHERE id = ${1}`;

// Prepared statements
const stmt = await sqlite.prepare("SELECT * FROM users WHERE id = ?");
const user = await stmt.get(1);
```

### Bun.Archive — Tar Archives (1.3.6+)

Create and extract tar.gz archives natively.

```typescript
// Create a tar.gz archive
const archive = new Bun.Archive();
archive.addFile("src/index.ts", await Bun.file("src/index.ts").text());
archive.addFile("package.json", await Bun.file("package.json").text());
await archive.write("dist/app.tar.gz");

// Extract a tar.gz archive
const archive = await Bun.Archive.from("dist/app.tar.gz");
await archive.extract("./output/");

// Stream extraction
const response = await fetch("https://example.com/archive.tar.gz");
const archive = Bun.Archive.from(await response.arrayBuffer());
await archive.extract("./output/");
```

### Bun.JSON5 / JSONL / JSONC — Extended JSON Parsers (1.3.6–1.3.7+)

```typescript
// JSON5 — parse JSON with comments and trailing commas
const obj = Bun.JSON5.parse(`{
  // This is a comment
  name: "Alice",
  age: 30,  // trailing comma
}`);

// JSONL — parse line-delimited JSON
const lines = Bun.JSONL.parse(`{"name": "Alice"}
{"name": "Bob"}
{"name": "Carol"}`);
// [{ name: "Alice" }, { name: "Bob" }, { name: "Carol" }]

// JSONC — parse JSON with comments (like VS Code settings)
const config = Bun.JSONC.parse(`{
  // Enable TypeScript
  "typescript.enabled": true,
  "editor.fontSize": 14,  // pixels
}`);
```

### Bun.wrapAnsi / Bun.stripANSI — ANSI String Operations (1.3.7–1.3.11+)

```typescript
// Wrap text to terminal width (33–88× faster than wrap-ansi)
const wrapped = Bun.wrapAnsi(longAnsiString, 80);

// Strip ANSI escape codes
const plain = Bun.stripANSI("\x1b[1mBold\x1b[0m text");
// "Bold text"
```

### process.on("memoryPressure") — Memory Warnings (1.4.0+)

React to OS memory pressure before the process is killed:

```typescript
process.on("memoryPressure", (level) => {
  if (level === "warning") {
    console.log("⚠️ Memory pressure — clearing caches");
    cache.clear();
    pool.drainIdle();
  } else if (level === "critical") {
    console.log("🚨 Critical memory — freeing everything");
    cache.clear();
    closeIdleConnections();
  }
});
```

Works on macOS (`kqueue`), Linux (`/proc/pressure/memory`), and Windows (`CreateMemoryResourceNotification`).

## Native Streams and Bodies (1.4.0+)

`ReadableStream`, `WritableStream`, and `TransformStream` are now native — up to 7× faster, 100% Web Platform Tests.

```typescript
// Fetch → decompress → decode
const text = await fetch(url)
  .then(r => r.body.pipeThrough(new DecompressionStream("gzip")))
  .then(r => r.pipeThrough(new TextDecoderStream()))
  .then(r => r.getReader())
  .then(reader => reader.read())
  .then(({ done, value }) => done ? null : new TextDecoder().decode(value));

// Upload with compression
const body = Bun.file("data.json")
  .stream()
  .pipeThrough(new TextEncoderStream())
  .pipeThrough(new CompressionStream("gzip"));

await fetch(url, { method: "POST", body });
```

**Performance (Bun 1.4 vs Node.js 26):**

| Pipeline | Bun 1.4 | Node.js 26 |
|---|---|---|
| DecompressionStream | 2,291 MB/s | 491 MB/s |
| TextEncoderStream | 1,963 MB/s | 75 MB/s |
| Subprocess pipe | 751 MB/s | 256 MB/s |

### Zero-Copy Response/Request Clone (1.4.0+)

`Response.clone()` and `Request.clone()` share body chunks — no full copy:

```typescript
const res = await fetch(url);
const clone = res.clone();

// Both can be read concurrently without doubling memory
const [a, b] = await Promise.all([res.arrayBuffer(), clone.arrayBuffer()]);
```

### Backpressure (1.4.0+)

`Bun.serve` automatically pauses `ReadableStream` bodies when the connection can't accept more data:

```typescript
Bun.serve({
  routes: {
    "/": () => new Response(new ReadableStream({
      pull(controller) {
        controller.enqueue(new Uint8Array(65536));
        // Automatically pauses when socket buffer fills
      },
    })),
  },
});
```

Works with: `CompressionStream`, `DecompressionStream`, `TextEncoderStream`, `TextDecoderStream`, `HTMLRewriter`, `child_process`, `Bun.spawn`, `Bun.file(path).stream()`, `Blob.stream()`.

## Runtime Performance APIs

### Async Stack Traces (1.3.7+)

Errors from async native APIs point back to the `await` in your code:

```typescript
async function getUser() {
  const res = await fetch("https://api.example.com/user");
  return res.json(); // Error points here, not deep in fetch internals
}
```

### Profiling

```bash
# CPU profile (Chrome DevTools compatible)
bun --cpu-prof ./app.ts

# CPU profile as Markdown (LLM-friendly)
bun --cpu-prof-md ./app.ts

# Heap snapshot (Chrome DevTools compatible)
bun --heap-prof ./app.ts

# Heap profile as Markdown
bun --heap-prof-md ./app.ts

# Profile a running process (can't pass flags)
BUN_CPU_PROFILE=1 node app.js
```

## Additional Native APIs

### Bun.hash — Hashing (1.2.21+)

```typescript
// RapidHash (fastest)
const hash = Bun.hash.rapidhash("hello world");

// CRC32
const crc = Bun.hash.crc32("hello world");

// Fast hash (general purpose)
const h = Bun.hash("hello world");
```

### Bun.sliceAnsi — ANSI-Aware String Slicing (1.3.11+)

```typescript
// Slice a string with ANSI codes, preserving escape sequences
const sliced = Bun.sliceAnsi("\x1b[1mHello world\x1b[0m", 0, 5);
// "\x1b[1mHello\x1b[0m" (correctly handles escape codes)
```

### Bun.Glob — File Globbing (1.2.16+)

```typescript
const files = await Bun.Glob.scan("src/**/*.ts", {
  cwd: import.meta.dir,
});
for await (const file of files) {
  console.log(file);
}
```

### process.execve() (1.3.14+)

Replace the current process with a new program:

```typescript
process.execve("/usr/bin/node", ["node", "server.js"], {
  ...process.env,
  NODE_ENV: "production",
});
```

### process.on("uncaughtException") / process.on("unhandledRejection") (1.1.8+)

```typescript
process.on("uncaughtException", (error) => {
  console.error("Uncaught:", error);
  // Cleanup and exit
  process.exit(1);
});

process.on("unhandledRejection", (reason) => {
  console.error("Unhandled rejection:", reason);
});
```

## Completion Checklist

When using Bun native APIs:

- ✅ Identified replaceable npm packages
- ✅ Replaced with Bun native APIs
- ✅ Verified behavior matches original
- ✅ Removed old npm dependencies
- ✅ Updated imports
- ✅ Tests passing
- ✅ Bundle size reduced

## Next Steps

1. Audit `package.json` for more replaceable dependencies
2. Explore `Bun.Image` for image processing pipelines
3. Try `Bun.WebView` for headless testing/automation
4. Use `Bun.SQL` for database access without ORM overhead
5. Set up `process.on("memoryPressure")` for production resilience
