---
name: bun-dev-server
description: Set up high-performance development servers with Hot Module Replacement. Use when creating dev servers for web apps, setting up React Fast Refresh, or configuring API servers with live reload.
compatibility: Requires Bun 1.4+
allowed-tools: ["Bash", "Write", "Read"]
metadata:
  author: dale
  category: bun-runtime
  tags: [bun, dev-server, hmr, react, hot-reload]
---

# Bun Development Server Setup

Set up high-performance development servers using `Bun.serve` with built-in HMR, React Fast Refresh, and native Bun 1.4 features.

## What's new in Bun 1.4

- **Built-in HMR**: `bun run --hot` handles HMR natively — no custom WebSocket code needed
- **`routes` in `Bun.serve`** (1.2.16): Declarative file-based routing for static files
- **HTTP/2 support**: `Bun.serve({ http: { ... } })` for HTTP/2 with TLS
- **Backpressure**: `ReadableStream` request/response bodies auto-pause when sockets fill
- **`WebSocket.pause()`/`resume()`** (1.4.1): Control WebSocket message flow
- **`Bun.Terminal`** (1.3.5, improved 1.4.0): Built-in PTY for driving terminal apps
- **50% faster startup on Windows**, lower memory usage
- **Native streams**: `ReadableStream`/`WritableStream` up to 7× faster

## Quick Reference

For detailed patterns, see:
- **HMR Examples**: [hmr-examples.md](references/hmr-examples.md) — Built-in HMR, custom HMR, CSS HMR, framework-specific patterns
- **Advanced patterns**: `Bun.Terminal`, HTTP/2, backpressure, WebSocket pause/resume

## Core Workflow

### 1. Determine Server Type

Ask the user what type of development server they need:

- **React/Frontend App**: SPA with React Fast Refresh
- **API Server**: REST/GraphQL API with auto-reload
- **Full-Stack App**: Frontend + API combined
- **Static Server**: File server with live reload
- **Terminal App**: PTY-driven terminal in the browser

### 2. Check Prerequisites

```bash
# Verify Bun installation
bun --version

# Check if project has package.json
ls -la package.json
```

If no package.json exists, suggest running `bun init` first.

### 3. Install Dependencies

**For React Apps:**
```bash
bun add react react-dom
bun add -d @types/react @types/react-dom
```

**For API with Hono (recommended):**
```bash
bun add hono
```

**For Full-Stack:**
```bash
bun add react react-dom hono
bun add -d @types/react @types/react-dom
```

### 4. Create Server Configuration

#### React Development Server (with built-in HMR)

Create `server.ts` in the project root:

```typescript
import { Hono } from 'hono';
import { serveStatic } from 'hono/bun';

const app = new Hono();

// Serve static files from public/
app.use('/*', serveStatic({ root: './public' }));

// SPA fallback — serve index.html for all non-file routes
app.get('*', (c) => c.html(Bun.file('public/index.html')));

const server = Bun.serve({
  port: 3000,
  fetch: app.fetch,
});

console.log(`🚀 Dev server running at http://localhost:${server.port}`);
```

Start with HMR:
```bash
bun run --hot server.ts
```

**Bun's `--hot` flag handles HMR automatically** — no custom WebSocket code needed. It watches for file changes and reloads the browser. For React, React Fast Refresh works out of the box.

> **Note**: The old pattern of manually implementing HMR with WebSockets and `Bun.file.watch()` is no longer necessary. `bun run --hot` is the recommended approach. See [hmr-examples.md](references/hmr-examples.md) for the legacy pattern and advanced customization.

Create `public/index.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Bun + React App</title>
</head>
<body>
  <div id="root"></div>
  <script type="module" src="/src/index.tsx"></script>
</body>
</html>
```

Create `src/index.tsx`:
```typescript
import { render } from 'react-dom';
import App from './App';

const root = document.getElementById('root');
render(<App />, root);
```

Create `src/App.tsx`:
```typescript
export default function App() {
  return (
    <div>
      <h1>Welcome to Bun + React!</h1>
      <p>Edit src/App.tsx to see HMR in action</p>
    </div>
  );
}
```

#### API Server with Hono

Create `server.ts`:
```typescript
import { Hono } from 'hono';
import { cors } from 'hono/cors';
import { logger } from 'hono/logger';

const app = new Hono();

// Middleware
app.use('*', cors());
app.use('*', logger());

// Routes
app.get('/', (c) => {
  return c.json({ message: 'Welcome to Bun API' });
});

app.get('/api/health', (c) => {
  return c.json({ status: 'ok', timestamp: Date.now() });
});

// Example POST endpoint
app.post('/api/users', async (c) => {
  const body = await c.req.json();
  return c.json({ created: true, data: body }, 201);
});

// Start server
const server = Bun.serve({
  port: process.env.PORT || 3000,
  fetch: app.fetch,
});

console.log(`🚀 API server running at http://localhost:${server.port}`);
```

#### Full-Stack Server

Create `server.ts`:
```typescript
import { Hono } from 'hono';
import { serveStatic } from 'hono/bun';

const app = new Hono();

// API routes
const api = new Hono();

api.get('/health', (c) => c.json({ status: 'ok' }));
api.get('/users', (c) => c.json({ users: [] }));

app.route('/api', api);

// Serve static files
app.use('/*', serveStatic({ root: './public' }));

// SPA fallback
app.get('*', (c) => c.html(Bun.file('public/index.html')));

const server = Bun.serve({
  port: 3000,
  fetch: app.fetch,
});

console.log(`🚀 Full-stack server at http://localhost:${server.port}`);
```

#### Terminal App with Bun.Terminal (Bun 1.3.5+)

Drive a terminal emulator in the browser:

```typescript
const server = Bun.serve({
  port: 3000,
  fetch(req, server) {
    const url = new URL(req.url);

    // WebSocket for terminal
    if (url.pathname === '/terminal') {
      const upgraded = server.upgrade(req);
      if (upgraded) return undefined;
    }

    return new Response(Bun.file('public/terminal.html'));
  },

  websocket: {
    open(ws) {
      // Spawn a PTY when client connects
      const proc = Bun.spawn(['bash'], {
        terminal: {
          cols: 80,
          rows: 24,
          data(term, data) {
            ws.send(data);
          },
        },
      });

      ws.data.pty = proc;

      ws.addEventListener('message', (event) => {
        proc.terminal?.write(event.data as string);
      });
    },

    close(ws) {
      ws.data.pty?.kill();
    },
  },
});
```

See [hmr-examples.md](references/hmr-examples.md) for the full terminal HTML client.

### 5. Advanced: HTTP/2

Enable HTTP/2 with TLS:

```typescript
import { readFileSync } from 'fs';

const server = Bun.serve({
  port: 3000,
  tls: {
    cert: readFileSync('./localhost.pem'),
    key: readFileSync('./localhost-key.pem'),
  },
  http: {
    // Enable HTTP/2
    http2: true,
  },
  fetch: app.fetch,
});

console.log(`🔒 HTTP/2 server at https://localhost:${server.port}`);
```

### 6. Advanced: WebSocket Pause/Resume (Bun 1.4.1+)

Control WebSocket message flow:

```typescript
const server = Bun.serve({
  port: 3000,
  fetch(req, server) {
    return server.upgrade(req);
  },

  websocket: {
    open(ws) {
      // Send data rapidly
      const interval = setInterval(() => {
        if (!ws.isPaused) {
          ws.send(JSON.stringify({ tick: Date.now() }));
        }
      }, 100);

      ws.data.interval = interval;
    },

    close(ws) {
      clearInterval(ws.data.interval);
    },

    // Pause is called automatically when the client can't keep up
    pause(ws) {
      console.log('Client paused — backpressure');
    },

    // Resume when client catches up
    resume(ws) {
      console.log('Client resumed');
    },
  },
});
```

### 7. Environment Configuration

Create `.env.development`:
```bash
# Server
PORT=3000
NODE_ENV=development

# API
API_URL=http://localhost:3000/api
```

Create `.env.production`:
```bash
# Server
PORT=8080
NODE_ENV=production

# API
API_URL=https://api.example.com
```

Load environment in `server.ts`:
```typescript
// Environment is loaded automatically by Bun
const isDev = process.env.NODE_ENV === 'development';
const port = process.env.PORT || 3000;
```

### 8. Update package.json Scripts

Add development scripts:

```json
{
  "scripts": {
    "dev": "bun run --hot server.ts",
    "dev:watch": "bun run --watch server.ts",
    "start": "NODE_ENV=production bun run server.ts",
    "build": "bun build src/index.tsx --outdir=dist --minify",
    "clean": "rm -rf dist"
  }
}
```

**Script explanations:**
- `dev`: Run with HMR (auto-reloads on file changes)
- `dev:watch`: Watch mode (faster, but doesn't reload on crash)
- `start`: Production mode
- `build`: Build frontend for production

### 9. Configure TypeScript

Update `tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "types": ["@types/bun"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "allowImportingTsExtensions": true,
    "noEmit": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*", "server.ts"]
}
```

### 10. Create Project Structure

Generate complete project structure:
```
project/
├── server.ts              # Development server
├── src/
│   ├── index.tsx         # App entry point
│   ├── App.tsx           # Main component
│   ├── components/       # React components
│   └── styles/           # CSS files
├── public/
│   ├── index.html        # HTML template
│   └── assets/           # Static assets
├── .env.development
├── .env.production
├── package.json
├── tsconfig.json
└── README.md
```

### 11. Proxy Configuration (for existing backends)

If user needs to proxy API requests to another server:

```typescript
const app = new Hono();

// Proxy /api requests to backend
app.all('/api/*', async (c) => {
  const url = new URL(c.req.url);
  const backendUrl = `http://localhost:8080${url.pathname}${url.search}`;

  const response = await fetch(backendUrl, {
    method: c.req.method,
    headers: c.raw.headers,
    body: c.req.method !== 'GET' ? await c.req.raw.text() : undefined,
  });

  return new Response(response.body, {
    status: response.status,
    headers: response.headers,
  });
});
```

## Testing the Setup

After creation, guide user to test:

```bash
# 1. Start dev server with HMR
bun run --hot server.ts

# 2. Open browser
open http://localhost:3000

# 3. Make a change to src/App.tsx
# 4. Verify HMR updates the page automatically

# 5. Test API endpoints
curl http://localhost:3000/api/health
```

## Troubleshooting

### HMR not working

```bash
# Check server is running with --hot flag
bun run --hot server.ts

# Verify no firewall blocking port 3000
# Check browser console for errors
```

### Port already in use

```bash
# Find process using port 3000
Get-NetTCPConnection -LocalPort 3000 -ErrorAction SilentlyContinue | Stop-Process -Force

# Or use a different port
PORT=3001 bun run --hot server.ts
```

### CORS issues

Add CORS headers to server:
```typescript
app.use('*', cors({
  origin: 'http://localhost:3000',
  credentials: true,
}));
```

## Performance Tips

1. **Use `--hot` for development**: Built-in HMR is faster than custom implementations
2. **Minimize file watcher scope**: `--hot` watches the current directory by default
3. **Use HTTP/2**: Enable for faster parallel loading
4. **Cache static assets**: Add Cache-Control headers
5. **Use backpressure**: Bun's native streams auto-pause when clients slow down

```typescript
app.use('/assets/*', async (c, next) => {
  await next();
  c.header('Cache-Control', 'public, max-age=31536000');
});
```

## Completion Checklist

- ✅ Development server created
- ✅ HMR configured (using `--hot`)
- ✅ Environment variables set up
- ✅ Package.json scripts added
- ✅ Project structure organized
- ✅ TypeScript configured
- ✅ Browser successfully connects
- ✅ File changes trigger HMR reload

## Next Steps

Suggest to the user:
1. Add error boundaries for better error handling
2. Set up ESLint and Prettier
3. Configure path aliases in tsconfig.json
4. Add development vs production builds
5. Consider adding `bun-test` for testing
6. Explore `Bun.Terminal` for terminal-based apps
7. Use `bun build --compile` for standalone distribution

For detailed implementations, see the reference files linked above.
