# Self-Contained HTML Reference

Package your app as a single self-contained HTML file with `--target=browser`.

## Basic Usage

```bash
bun build ./src/index.tsx --target=browser --outfile ./dist/app.html
```

Or programmatically:

```typescript
await Bun.build({
  entrypoints: ['./src/index.tsx'],
  outfile: './dist/app.html',
  target: 'browser',
  format: 'iife',
  minify: true,
});
```

## With Loaders

```bash
bun build ./src/index.tsx \
  --target=browser \
  --outfile=./dist/app.html \
  --loader .svg=dataurl \
  --loader .css=css
```

## Package.json Script

```json
{
  "scripts": {
    "build:demo": "bun build ./src/demo.tsx --target=browser --outfile ./dist/demo.html",
    "build:widget": "bun build ./src/widget.ts --target=browser --outfile ./dist/widget.html --minify"
  }
}
```

## Use Cases

- Interactive demos
- Embeddable widgets
- Portable apps (run anywhere with a browser)
- Code sandboxes
- Offline tools

## Example: Interactive Demo

```typescript
// src/demo.tsx
import { h, render } from "preact";

function App() {
  return (
    <div>
      <h1>Hello from Bun!</h1>
      <Counter />
    </div>
  );
}

function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>;
}

render(<App />, document.getElementById("root"));
```

Build:
```bash
bun build ./src/demo.tsx --target=browser --outfile ./dist/demo.html
```

Open `demo.html` in any browser — it works offline.

## Example: Embeddable Widget

```typescript
// src/widget.ts
export function initWidget(container: HTMLElement) {
  container.innerHTML = `<div class="widget">Hello!</div>`;
}
```

Build and embed:
```html
<script src="widget.html"></script>
<div id="widget"></div>
<script>
  // widget.html exposes initWidget globally
  initWidget(document.getElementById("widget"));
</script>
```

## Limitations

- No server-side code (everything runs in browser)
- No file system access
- No Node.js APIs
- Bundle size limited by browser memory
