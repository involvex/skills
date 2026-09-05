# Bun.Terminal — PTY Support

Built-in pseudo-terminal for driving interactive CLI programs from JavaScript.

## Quick Start

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
```

## API Reference

### Spawning with Terminal

```typescript
const proc = Bun.spawn(["bash"], {
  terminal: {
    cols: 80,
    rows: 24,
    data(term, data) {
      // PTY output callback
      console.log(data);
    },
  },
});
```

### Writing to Terminal

```typescript
proc.terminal?.write("ls -la\n");
proc.terminal?.write("\x03"); // Ctrl+C
```

### Resizing Terminal

```typescript
proc.terminal?.resize(120, 40);
```

### Terminal Options

```typescript
const proc = Bun.spawn(["bash"], {
  terminal: {
    cols: 80,           // Columns (default: 80)
    rows: 24,           // Rows (default: 24)
    cwd: "./project",   // Working directory
    env: { ... },       // Environment variables
    data(term, data) {  // Output callback
      ws.send(data);    // Send to WebSocket, etc.
    },
  },
});
```

## WebSocket Terminal Server

Serve a terminal in the browser:

```typescript
const server = Bun.serve({
  port: 3000,
  fetch(req, server) {
    const url = new URL(req.url);

    if (url.pathname === '/terminal') {
      const upgraded = server.upgrade(req);
      if (upgraded) return undefined;
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

      ws.terminal = proc.terminal;

      ws.addEventListener('message', (event) => {
        proc.terminal?.write(event.data as string);
      });

      ws.addEventListener('close', () => {
        proc.kill();
      });
    },
  },
});
```

## Common Patterns

### Interactive Shell

```typescript
const proc = Bun.spawn(['bash'], {
  terminal: {
    cols: 80,
    rows: 24,
    data(term, data) {
      process.stdout.write(data);
    },
  },
  stdin: 'pipe',
  stdout: 'inherit',
  stderr: 'inherit',
});

// Write commands
proc.terminal.write("npm install\n");

// Wait for exit
await proc.exited;
```

### Running a Command

```typescript
const proc = Bun.spawn(['npm', 'run', 'build'], {
  terminal: {
    data(term, data) {
      console.log(data);
    },
  },
});

await proc.exited;
console.log(`Exit code: ${proc.exitCode}`);
```

## Performance Notes

- No `node-pty` dependency needed
- Uses ConPTY on Windows, `forkpty` on Linux/macOS
- Works on Linux, macOS, and Windows
