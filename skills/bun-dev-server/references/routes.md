# Bun.serve Routes Reference

Complete reference for `Bun.serve({ routes })` and related patterns in Bun 1.4+.

## Basic Routes

```typescript
const server = Bun.serve({
  port: 3000,
  routes: {
    '/': Bun.file('public/index.html'),
    '/about': Bun.file('public/about.html'),
    '/api/health': () => new Response(JSON.stringify({ status: 'ok' })),
  },
});
```

Routes map path patterns to responses. Each route can be:
- A `Blob`/`File`/`Response` object
- A function returning a `Response`
- A handler function `(req: Request) => Response`

## SPA Fallback

Catch-all for client-side routing:

```typescript
const server = Bun.serve({
  port: 3000,
  routes: {
    '/': Bun.file('public/index.html'),
    '/{...}': Bun.file('public/index.html'), // SPA fallback
  },
});
```

The `/{...}` pattern matches any unmatched route.

## Static File Serving

Serve files with automatic MIME types:

```typescript
const server = Bun.serve({
  port: 3000,
  routes: {
    // Serve specific files
    '/favicon.ico': Bun.file('public/favicon.ico'),
    '/robots.txt': Bun.file('public/robots.txt'),

    // Directory serving
    '/static/{...}': Bun.file('public/static/{...}'),
  },
});
```

## Dynamic Routes

Capture path parameters:

```typescript
const server = Bun.serve({
  port: 3000,
  routes: {
    '/api/users/:id': (req) => {
      const id = new URL(req.url).pathname.split('/').pop();
      return new Response(JSON.stringify({ userId: id }));
    },
  },
});
```

## Headers and Status

Return custom headers and status codes:

```typescript
const server = Bun.serve({
  port: 3000,
  routes: {
    '/api/data': () => {
      return new Response(JSON.stringify({ data: 'value' }), {
        status: 200,
        headers: {
          'Content-Type': 'application/json',
          'Cache-Control': 'no-cache',
        },
      });
    },
  },
});
```

## CORS Headers

```typescript
const corsHeaders = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type, Authorization',
};

const server = Bun.serve({
  port: 3000,
  routes: {
    '/api/data': () => new Response(JSON.stringify({ data: 'value' }), {
      headers: corsHeaders,
    }),
    // Handle preflight
    'OPTIONS/{...}': () => new Response(null, { status: 204, headers: corsHeaders }),
  },
});
```

## Redirects

```typescript
const server = Bun.serve({
  port: 3000,
  routes: {
    '/old-page': () => new Response(null, {
      status: 302,
      headers: { Location: '/new-page' },
    }),
    '/old-page-2': () => Response.redirect('/new-page'),
  },
});
```

## Error Handling

```typescript
const server = Bun.serve({
  port: 3000,
  routes: {
    '/': Bun.file('public/index.html'),
    '/{...}': () => new Response('Not Found', { status: 404 }),
  },
});
```

## Combining Routes with Hono

For complex APIs, use `routes` for static files and Hono for dynamic routes:

```typescript
import { Hono } from 'hono';
import { serveStatic } from 'hono/bun';

const app = new Hono();

// API routes with Hono
app.get('/api/users', (c) => c.json({ users: [] }));
app.get('/api/users/:id', (c) => c.json({ id: c.req.param('id') }));

// Static files
app.use('/*', serveStatic({ root: './public' }));

// SPA fallback
app.get('*', (c) => c.html(Bun.file('public/index.html')));

const server = Bun.serve({
  port: 3000,
  fetch: app.fetch,
});
```

## Middleware with Routes

```typescript
const server = Bun.serve({
  port: 3000,
  routes: {
    '/api/{...}': async (req) => {
      // Middleware: logging
      console.log(`${req.method} ${new URL(req.url).pathname}`);

      // Middleware: auth check
      const auth = req.headers.get('Authorization');
      if (!auth) {
        return new Response('Unauthorized', { status: 401 });
      }

      // Route handler
      return new Response(JSON.stringify({ data: 'protected' }));
    },
  },
});
```

## Performance Tips

1. **Use `routes` for static files**: Faster than manual `fetch` handlers
2. **Order matters**: Routes are matched in order — put specific routes first
3. **Use `/{...}` for SPA fallback**: Only one catch-all needed
4. **Avoid dynamic imports in routes**: Preload modules at startup
