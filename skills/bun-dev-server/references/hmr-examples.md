# HMR and Dev Server Patterns

This document provides HMR implementation patterns for Bun development servers.

## Built-in HMR (Recommended, Bun 1.4+)

The recommended approach is to use `bun run --hot`, which handles HMR automatically:

```bash
# Start dev server with built-in HMR
bun run --hot server.ts
```

**No custom WebSocket code needed.** Bun watches for file changes and triggers browser reloads automatically. React Fast Refresh works out of the box.

### Why built-in HMR?

- **Zero setup**: No WebSocket server, no file watcher, no client-side script
- **React Fast Refresh**: Preserves component state during edits
- **CSS HMR**: Styles update without full reload
- **Automatic**: Works with `import.meta.hot` for framework-agnostic acceptance

```typescript
// In any module — runs when the module is hot-updated
if (import.meta.hot) {
  import.meta.hot.accept(() => {
    console.log('Module updated');
  });

  import.meta.hot.dispose((data) => {
    // Cleanup before the module is replaced
    console.log('Cleaning up before update');
  });
}
```

## Legacy: Custom WebSocket HMR

If you need custom HMR behavior, here is the manual pattern:

```typescript
// server.ts
import type { ServerWebSocket } from "bun";

const clients = new Set<ServerWebSocket<unknown>>();

const server = Bun.serve({
  port: 3000,

  fetch(request, server) {
    const url = new URL(request.url);

    if (url.pathname === "/_hmr") {
      server.upgrade(request);
      return undefined;
    }

    return new Response(Bun.file("index.html"));
  },

  websocket: {
    open(ws) {
      clients.add(ws);
    },
    close(ws) {
      clients.delete(ws);
    },
    message() {},
  },
});

// File watcher
const watcher = Bun.file.watch("./src");
for await (const event of watcher) {
  for (const client of clients) {
    client.send(JSON.stringify({ type: "reload", file: event.path }));
  }
}
```

```html
<!-- Client-side HMR -->
<script>
  const ws = new WebSocket('ws://localhost:3000/_hmr');
  ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    if (data.type === 'reload') {
      window.location.reload();
    }
  };
</script>
```

> **Note**: This manual pattern is kept for reference and advanced use cases. Prefer `bun run --hot` for standard development workflows.

## React Fast Refresh with Built-in HMR

React Fast Refresh works automatically with `bun run --hot`:

```tsx
// App.tsx
export default function App() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}
```

Edit the component — the counter state is preserved. No extra configuration needed.

For manual control:

```typescript
if (import.meta.hot) {
  import.meta.hot.accept((newModule) => {
    // Replace the current module with the updated one
  });

  import.meta.hot.dispose((data) => {
    // Preserve state across updates
    data.count = count;
  });
}
```

## CSS HMR (No Page Reload)

CSS updates automatically with `bun run --hot` — no custom code needed.

For manual CSS injection:

```typescript
// server.ts
const cssWatcher = Bun.file.watch("./src/**/*.css");

for await (const event of cssWatcher) {
  if (event.kind === "change") {
    const cssContent = await Bun.file(event.path).text();
    for (const client of clients) {
      client.send(JSON.stringify({
        type: "css-update",
        path: event.path,
        content: cssContent
      }));
    }
  }
}
```

```javascript
// Client-side CSS injection
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.type === 'css-update') {
    let styleTag = document.querySelector(`style[data-path="${data.path}"]`);
    if (!styleTag) {
      styleTag = document.createElement('style');
      styleTag.setAttribute('data-path', data.path);
      document.head.appendChild(styleTag);
    }
    styleTag.textContent = data.content;
  }
};
```

## Server-Side Code Reload

Reload server-side code without dropping connections:

```typescript
// hot-server.ts
let requestHandler = (await import('./routes.ts')).default;

const server = Bun.serve({
  port: 3000,
  async fetch(request) {
    return requestHandler(request);
  },
});

// Watch server code
const serverWatcher = Bun.file.watch("./routes.ts");
for await (const event of serverWatcher) {
  console.log('🔄 Reloading server code...');
  const newModule = await import(`./routes.ts?t=${Date.now()}`);
  requestHandler = newModule.default;
  console.log('✅ Server code reloaded');
}
```

## Terminal App with Bun.Terminal (Bun 1.3.5+)

Drive a terminal emulator via WebSocket:

```typescript
// server.ts
const server = Bun.serve({
  port: 3000,
  fetch(req, server) {
    const url = new URL(req.url);
    if (url.pathname === '/terminal') {
      return server.upgrade(req);
    }
    return new Response(Bun.file('public/terminal.html'));
  },

  websocket: {
    open(ws) {
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

```html
<!-- public/terminal.html -->
<!DOCTYPE html>
<html>
<head>
  <title>Bun Terminal</title>
  <style>
    body { background: #1a1a1a; color: #fff; font-family: monospace; }
    #terminal { padding: 20px; white-space: pre; }
  </style>
</head>
<body>
  <div id="terminal">Connecting...</div>
  <script>
    const ws = new WebSocket('ws://localhost:3000/terminal');
    const terminal = document.getElementById('terminal');

    ws.onmessage = (event) => {
      terminal.textContent += event.data;
    };

    document.addEventListener('keydown', (e) => {
      ws.send(e.key);
    });
  </script>
</body>
</html>
```

## WebSocket Pause/Resume (Bun 1.4.1+)

Handle backpressure in WebSocket connections:

```typescript
const server = Bun.serve({
  port: 3000,
  fetch(req, server) {
    return server.upgrade(req);
  },

  websocket: {
    open(ws) {
      // Rapid data producer
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

    pause(ws) {
      console.log('Client buffer full — pausing');
      // Bun automatically pauses when the client can't keep up
    },

    resume(ws) {
      console.log('Client caught up — resuming');
    },
  },
});
```

## Error Overlay

Display runtime errors in browser:

```typescript
// error-overlay.ts
export function showErrorOverlay(error: Error) {
  const overlay = document.createElement('div');
  overlay.id = 'error-overlay';
  overlay.style.cssText = `
    position: fixed; top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(0, 0, 0, 0.9); color: #fff; padding: 20px;
    font-family: monospace; z-index: 999999; overflow: auto;
  `;

  overlay.innerHTML = `
    <h1 style="color: #ff5555;">Runtime Error</h1>
    <pre style="color: #ffff55;">${error.message}</pre>
    <pre style="color: #888;">${error.stack}</pre>
    <button onclick="this.parentElement.remove()">Dismiss</button>
  `;

  document.body.appendChild(overlay);
}

// Listen for errors
window.addEventListener('error', (event) => {
  if (import.meta.env.DEV) {
    showErrorOverlay(event.error);
  }
});
```

## Vue 3 HMR

Hot Module Replacement for Vue components:

```typescript
// hmr-vue.ts
import { createApp } from 'vue';

let app = createApp(App);
app.mount('#app');

if (import.meta.hot) {
  import.meta.hot.accept('./App.vue', (newModule) => {
    app.unmount();
    app = createApp(newModule.default);
    app.mount('#app');
  });
}
```

## Svelte HMR

```typescript
// hmr-svelte.ts
import App from './App.svelte';

let app = new App({
  target: document.getElementById('app')!
});

if (import.meta.hot) {
  import.meta.hot.accept();
  import.meta.hot.dispose(() => {
    app.$destroy();
  });
}
```

## Smart File Watching

Only reload affected modules:

```typescript
// dependency-graph.ts
class DependencyGraph {
  private graph = new Map<string, Set<string>>();

  addDependency(parent: string, child: string) {
    if (!this.graph.has(parent)) {
      this.graph.set(parent, new Set());
    }
    this.graph.get(parent)!.add(child);
  }

  getAffectedModules(changedFile: string): Set<string> {
    const affected = new Set<string>();
    const queue = [changedFile];

    while (queue.length > 0) {
      const file = queue.shift()!;
      affected.add(file);

      for (const [parent, children] of this.graph) {
        if (children.has(file) && !affected.has(parent)) {
          queue.push(parent);
        }
      }
    }

    return affected;
  }
}

const graph = new DependencyGraph();
const watcher = Bun.file.watch("./src");
for await (const event of watcher) {
  const affected = graph.getAffectedModules(event.path);
  for (const client of clients) {
    client.send(JSON.stringify({
      type: "update",
      modules: Array.from(affected)
    }));
  }
}
```

## Production HMR Disable

Ensure HMR is disabled in production:

```typescript
// config.ts
export const HMR_ENABLED = process.env.NODE_ENV === 'development';

// server.ts
if (HMR_ENABLED) {
  setupHMR();
}
```

## Performance Optimization

Debounce file changes to avoid excessive reloads:

```typescript
function debounce<T extends (...args: any[]) => void>(
  fn: T,
  delay: number
): T {
  let timeout: Timer;

  return ((...args: Parameters<T>) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => fn(...args), delay);
  }) as T;
}

const notifyClients = debounce((file: string) => {
  for (const client of clients) {
    client.send(JSON.stringify({ type: "reload", file }));
  }
}, 100);
```

## Resources

- [Bun HMR Documentation](https://bun.sh/docs/runtime/hot)
- [React Fast Refresh](https://github.com/facebook/react/tree/main/packages/react-refresh)
- [Vite HMR API](https://vitejs.dev/guide/api-hmr.html)
