# Terminal and PTY Integration

Drive real terminals from the browser using `Bun.Terminal` and `Bun.spawn`.

## Basic PTY Server

```typescript
import { Hono } from 'hono';
import { serveStatic } from 'hono/bun';

const app = new Hono();

// Serve terminal UI
app.get('/terminal', (c) => {
  return c.html(Bun.file('public/terminal.html'));
});

const server = Bun.serve({
  port: 3000,
  fetch: app.fetch,
});
```

## WebSocket + PTY Integration

```typescript
import { Hono } from 'hono';

const app = new Hono();

app.get('/terminal', (c) => {
  const upgradeHeader = c.req.header('upgrade');
  if (upgradeHeader !== 'websocket') {
    return c.text('Expected WebSocket', 426);
  }

  // Upgrade to WebSocket
  const ws = Bun.serve({
    port: 0, // Not used here
    fetch: () => new Response(),
  });

  // Spawn PTY
  const proc = Bun.spawn(['bash'], {
    terminal: {
      cols: 80,
      rows: 24,
      data(term, data) {
        ws.send(data);
      },
    },
  });

  // Handle WebSocket messages → PTY input
  // (In practice, use a proper WebSocket upgrade with Hono's ws adapter)

  // Handle PTY output → WebSocket
  // data() callback above handles this

  return ws;
});
```

## Terminal UI (HTML + JS)

```html
<!-- public/terminal.html -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Terminal</title>
  <style>
    body { margin: 0; background: #1a1a1a; }
    #terminal { width: 100vw; height: 100vh; }
  </style>
</head>
<body>
  <div id="terminal"></div>
  <script type="module">
    const ws = new WebSocket(`ws://${location.host}/terminal`);
    const term = document.getElementById('terminal');

    ws.binaryType = 'arraybuffer';
    ws.onmessage = (e) => {
      term.textContent += new TextDecoder().decode(e.data);
    };

    term.addEventListener('keydown', (e) => {
      ws.send(e.key);
    });
  </script>
</body>
</html>
```

## Resize Handling

```typescript
const proc = Bun.spawn(['bash'], {
  terminal: {
    cols: 80,
    rows: 24,
    data(term, data) {
      ws.send(data);
    },
  },
});

// Resize PTY when browser window resizes
window.addEventListener('resize', () => {
  const cols = Math.floor(window.innerWidth / 8);
  const rows = Math.floor(window.innerHeight / 16);
  proc.terminal.resize(cols, rows);
});
```

## Supported Platforms

- Linux: Uses native PTY
- macOS: Uses native PTY
- Windows: Uses ConPTY (Windows 10 1809+)

No `node-pty` needed. No native addon compilation.

## Use Cases

1. **Web-based terminals**: VS Code Server, Gitpod, StackBlitz
2. **Developer tools**: In-browser database shells, API explorers
3. **Education**: Interactive coding environments
4. **DevOps**: In-browser server management

## Performance Notes

- PTY output is streamed directly to WebSocket
- Backpressure is handled automatically by Bun's native streams
- Memory usage stays low even with long-running terminal sessions
