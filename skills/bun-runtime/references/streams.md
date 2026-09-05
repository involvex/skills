# Native Streams Reference

Bun 1.4 ships native `ReadableStream`, `WritableStream`, and `TransformStream` — up to 7× faster throughput than Node.js.

## What's Native Now

- `ReadableStream`, `WritableStream`, `TransformStream` — 100% Web Platform Tests
- `CompressionStream`, `DecompressionStream` — native implementation
- `TextDecoderStream`, `TextEncoderStream` — native implementation
- `Response.clone()` / `Request.clone()` — zero-copy body sharing
- `Readable.toWeb()` / `Writable.toWeb()` from `node:stream`

## CompressionStream

```typescript
const source = new ReadableStream({
  pull(c) { c.enqueue(new Uint8Array(65536)); c.close(); }
});

const compressed = source.pipeThrough(new CompressionStream("gzip"));
const response = new Response(compressed, {
  headers: { "Content-Encoding": "gzip" }
});
```

Performance: 152 MB/s compression (vs 135 MB/s Node.js)

## DecompressionStream

```typescript
const response = await fetch("https://example.com/data.gz");
const decompressed = response.body.pipeThrough(new DecompressionStream("gzip"));
const text = await new Response(decompressed).text();
```

Performance: 2,291 MB/s decompression (vs 491 MB/s Node.js)

## TextDecoderStream / TextEncoderStream

```typescript
// Decode UTF-8 bytes to strings
const textStream = byteStream.pipeThrough(new TextDecoderStream());
for await (const chunk of textStream) {
  console.log(chunk); // string
}

// Encode strings to UTF-8 bytes
const byteStream = textStream.pipeThrough(new TextEncoderStream());
```

Performance: 1,963 MB/s encoding, 1,489 MB/s decoding

## Zero-Copy clone()

```typescript
const res = await fetch("https://example.com/large-file");
const clone = res.clone(); // Shares body chunks — no full copy

const [a, b] = await Promise.all([res.arrayBuffer(), clone.arrayBuffer()]);
```

Memory savings: up to 155 MB vs 311 MB in Bun 1.3

## Backpressure

All streams respect backpressure automatically:

```typescript
return new Response(
  new ReadableStream({
    pull(controller) {
      // Only called when connection is ready for more data
      controller.enqueue(new Uint8Array(65536));
    },
  }),
);
```

Works with: `fetch()`, `Bun.spawn`, `Bun.file().stream()`, `Blob.stream()`, `CompressionStream`, `TransformStream`

## node:stream Interop

```typescript
import { Readable, Writable } from "node:stream";

// Convert Node.js stream to Web stream
const webStream = Readable.toWeb(fs.createReadStream("file.txt"));

// Convert Web stream to Node.js stream
const nodeStream = Writable.toWeb(fs.createWriteStream("output.txt"));
```

## Pipeline Benchmarks

| Pipeline | Bun 1.4 | Bun 1.3 | Node.js 26 | Deno 2.9 |
|---|---|---|---|---|
| Download (fetch → gzip → decode) | 1,519 MB/s | n/a | 204 MB/s | 530 MB/s |
| Upload (file → gzip → fetch) | 179 MB/s | n/a | 78 MB/s | 137 MB/s |
| Transcode (file → decode → encode → file) | 132 MB/s | 116 MB/s | 52 MB/s | 91 MB/s |
| Subprocess (fetch → cat → for-await) | 751 MB/s | 505 MB/s | 256 MB/s | 170 MB/s |

## When to Use

- Large file uploads/downloads
- Real-time data processing
- Compression/decompression
- Streaming APIs
- Any I/O-heavy application
