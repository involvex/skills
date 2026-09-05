# Bun.markdown — Markdown Parser

Built-in CommonMark-compliant Markdown parser.

## Quick Start

```typescript
// HTML output
const html = Bun.markdown.html("# Hello **world**");
// "<h1>Hello <strong>world</strong></h1>\n"

// React elements
export default function Page() {
  return Bun.markdown.react(readme);
}

// Terminal ANSI output
const ansi = Bun.markdown.render("# Hello\n\n**bold**", {
  heading: (children) => `\x1b[1;4m${children}\x1b[0m\n`,
  paragraph: (children) => children + "\n",
  strong: (children) => `\x1b[1m${children}\x1b[22m`,
});
```

## API Reference

### Bun.markdown.html()

Convert Markdown to HTML string:

```typescript
const html = Bun.markdown.html(markdownString);
```

### Bun.markdown.react()

Convert Markdown to React elements:

```typescript
import { BunMarkdown } from "bun";

function Page({ content }: { content: string }) {
  return <BunMarkdown>{content}</BunMarkdown>;
}

// Or use the static method
const elements = Bun.markdown.react(markdownString);
```

Custom components:
```typescript
function CustomHeading({ children, level }: { children: React.ReactNode; level: number }) {
  return <h{level} className="custom-heading">{children}</h{level}>;
}

Bun.markdown.react(markdownString, {
  heading: (children, { level }) => <CustomHeading level={level}>{children}</CustomHeading>,
});
```

### Bun.markdown.render()

Convert Markdown with custom renderers per element:

```typescript
const output = Bun.markdown.render(markdownString, {
  heading: (children, { level }) => {
    return `\x1b[1;4m${children}\x1b[0m\n`;
  },
  paragraph: (children) => children + "\n",
  strong: (children) => `\x1b[1m${children}\x1b[22m`,
  link: (children, { url }) => `\x1b[4;34m${children}\x1b[0m (\x1b[34m${url}\x1b[0m)`,
  code: (children) => `\x1b[48;5;235m${children}\x1b[0m`,
});
```

## Supported Features

- GFM tables
- Strikethrough
- Task lists
- Autolinks
- Linear-time parsing (safe against adversarial input)

## Bundler Loader

Use `.md` files as importable modules:

```typescript
import readme from './README.md';
const html = Bun.markdown.html(readme);
```

Or configure in `Bun.build`:
```typescript
await Bun.build({
  entrypoints: ['./src/index.ts'],
  loader: {
    '.md': 'text', // Import .md as text
  },
});
```

## Security Note

The HTML output is **not sanitized**. Raw HTML, event-handler attributes, and `javascript:` hrefs pass through verbatim. Sanitize if rendering untrusted input.
