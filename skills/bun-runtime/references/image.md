# Bun.Image — Image Processing API

Decode, resize, rotate, and encode images. No `sharp` needed.

## Supported Formats

| Format | Decode | Encode | Notes |
|--------|--------|--------|-------|
| JPEG | ✅ | ✅ | |
| PNG | ✅ | ✅ | |
| WebP | ✅ | ✅ | |
| GIF | ✅ | ✅ | |
| BMP | ✅ | ✅ | |
| HEIC | ✅ | ✅ | macOS and Windows only |
| AVIF | ✅ | ✅ | macOS and Windows only |
| TIFF | ✅ | ✅ | macOS and Windows only |

## Quick Start

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
```

## API Reference

### Creating an Image

```typescript
// From file
const image = Bun.Image.fromFile("photo.jpg");

// From buffer
const image = new Bun.Image(buffer);

// From Blob
const image = new Bun.Image(blob);

// From another image (copy)
const copy = new Bun.Image(existingImage);
```

### Resize

```typescript
image.resize(width: number, height: number, options?: {
  fit?: "inside" | "outside" | "fill" | "contain" | "cover";
  withoutEnlargement?: boolean; // Don't upscale
  withoutReduction?: boolean;   // Don't downscale
});
```

### Rotate

```typescript
image.rotate(degrees: number); // 90, 180, 270
```

### Flip

```typescript
image.flip(horizontal: boolean, vertical: boolean);
```

### Crop

```typescript
image.crop(x: number, y: number, width: number, height: number);
```

### Encoding

```typescript
// JPEG
image.jpeg({ quality: 85 });

// PNG
image.png({ compressionLevel: 9 });

// WebP
image.webp({ quality: 85 });

// GIF
image.gif();
```

### Writing Output

```typescript
// Write to file
await image.write("output.webp");

// Get buffer
const buffer = await image.buffer();

// Get Blob
const blob = image.blob();

// Get ArrayBuffer
const arrayBuffer = await image.arrayBuffer();
```

### Metadata

```typescript
image.width;   // number
image.height;  // number
```

## Common Patterns

### Thumbnail Generation

```typescript
async function generateThumbnail(inputPath: string, outputPath: string, size: number) {
  await Bun.file(inputPath)
    .image()
    .resize(size, size, { fit: "inside", withoutEnlargement: true })
    .webp({ quality: 80 })
    .write(outputPath);
}
```

### Image Upload Handler

```typescript
app.post('/upload', async (c) => {
  const formData = await c.req.formData();
  const file = formData.get('image') as File;

  const thumb = await Bun.file(file)
    .image()
    .resize(400, 400, { fit: 'inside' })
    .webp({ quality: 85 });

  c.header('Content-Type', 'image/webp');
  return thumb;
});
```

### Batch Processing

```typescript
const files = await glob('*.jpg');
for (const file of files) {
  await Bun.file(file)
    .image()
    .resize(800, 600)
    .webp({ quality: 80 })
    .write(file.replace(/\.jpg$/, '.webp'));
}
```

## Performance Notes

- On a 1080p PNG resized to 400×400 JPEG, `Bun.Image` is 1.38× faster than `sharp`
- On JPEG to WebP, it's 1.19× faster
- ICC color profiles like Display P3 survive transcoding
- No native addon required — pure Zig implementation inside Bun
