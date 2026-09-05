# Bun.WebView — Headless Browser Automation

Headless browser automation built into Bun. No Puppeteer or Playwright install needed.

## Quick Start

```typescript
await using view = new Bun.WebView({ width: 800, height: 600 });
await view.navigate("https://bun.sh");
await view.click("a[href='/docs']");
const title = await view.evaluate("document.title");
await Bun.write("page.png", await view.screenshot());
```

## Features

- Navigate, click, scroll, run JavaScript
- Take screenshots (returns `Blob`)
- Real user input for clicks and scrolls
- Extends `EventTarget`
- `.cdp(method, params?)` escape hatch for raw Chrome DevTools Protocol commands
- macOS: uses system WebKit (nothing to install)
- macOS/Linux/Windows: can also drive installed Chrome, Chromium, or Edge

## API Reference

### Constructor

```typescript
new Bun.WebView(options?: {
  width?: number;
  height?: number;
  headless?: boolean; // default true
  browser?: 'chromium' | 'webkit'; // default: auto
});
```

### Navigation

```typescript
await view.navigate("https://bun.sh");
await view.goBack();
await view.goForward();
await view.reload();
```

### Interaction

```typescript
// Click
await view.click("a[href='/docs']");
await view.click({ x: 100, y: 200 }); // Coordinates

// Type text
await view.type("input[name='search']", "hello world");

// Scroll
await view.scroll(0, 500); // x, y
await view.scrollToBottom();
```

### JavaScript Evaluation

```typescript
const title = await view.evaluate("document.title");
const count = await view.evaluate("document.querySelectorAll('a').length");

// With arguments
const result = await view.evaluate((selector: string) => {
  return document.querySelector(selector)?.textContent;
}, ".main-content");
```

### Screenshots

```typescript
// Full viewport
const blob = await view.screenshot();
await Bun.write("page.png", blob);

// Specific element
const blob = await view.screenshot({ selector: ".chart" });

// Full page (scroll and stitch)
const blob = await view.screenshot({ fullPage: true });
```

### Chrome DevTools Protocol

```typescript
// Raw CDP command
const result = await view.cdp("DOM.getDocument", {});
const nodeId = result.root.nodeId;

// Navigate via CDP
await view.cdp("Page.navigate", { url: "https://bun.sh" });
```

### Events

```typescript
view.addEventListener("load", () => {
  console.log("Page loaded");
});

view.addEventListener("console", (event) => {
  console.log("Console:", event.detail);
});

view.addEventListener("error", (event) => {
  console.error("Error:", event.detail);
});
```

## Common Patterns

### Screenshot Capture

```typescript
await using view = new Bun.WebView({ width: 1920, height: 1080 });
await view.navigate("https://example.com");
await Bun.sleep(1000); // Wait for render
const screenshot = await view.screenshot();
```

### Web Scraping

```typescript
await using view = new Bun.WebView();
await view.navigate("https://news.ycombinator.com");

const titles = await view.evaluate(() => {
  return Array.from(document.querySelectorAll('.titleline > a'))
    .map(a => ({ title: a.textContent, url: a.href }));
});
```

### Form Submission

```typescript
await using view = new Bun.WebView();
await view.navigate("https://example.com/login");
await view.type("#username", "admin");
await view.type("#password", "secret");
await view.click("button[type='submit']");

// Wait for navigation
await view.waitForNavigation();
```

## Migration from Puppeteer

| Puppeteer | Bun.WebView |
|-----------|-------------|
| `puppeteer.launch()` | `new Bun.WebView()` |
| `page.goto(url)` | `view.navigate(url)` |
| `page.click(selector)` | `view.click(selector)` |
| `page.type(selector, text)` | `view.type(selector, text)` |
| `page.screenshot()` | `view.screenshot()` |
| `page.evaluate(fn)` | `view.evaluate(fn)` |
| `page.setViewport({width, height})` | `new Bun.WebView({width, height})` |

## Performance Notes

- No browser download required on macOS (uses WebKit)
- On macOS/Linux/Windows, can use system Chrome/Chromium/Edge
- Screenshots return `Blob` — use `Bun.write()` to save
- Memory efficient — view is disposed with `using`
