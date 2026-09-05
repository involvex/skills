# Bun.Archive — tar.gz Archives

Create and extract tar.gz archives natively.

## Quick Start

```typescript
// Create archive
const archive = new Bun.Archive("./output.tar.gz");
await archive.addDirectory("./src");
await archive.addFile("./README.md", "README content");
await archive.close();

// Extract archive
const archive = await Bun.Archive.open("./output.tar.gz");
await archive.extract("./extracted");
archive.close();
```

## API Reference

### Constructor

```typescript
// Create new archive
const archive = new Bun.Archive(filePath: string);
```

### Static Methods

```typescript
// Open existing archive
const archive = await Bun.Archive.open("./archive.tar.gz");
```

### Adding Content

```typescript
// Add a directory (recursive)
await archive.addDirectory("./src", {
  prefix: "src/", // Optional prefix inside archive
});

// Add a file with content
await archive.addFile("./README.md", "File content as string");

// Add a file from buffer
await archive.addFile("./data.bin", buffer);

// Add from file path
await archive.addFileFromPath("./large-file.iso");
```

### Extracting

```typescript
// Extract all to directory
await archive.extract("./output");

// Extract with options
await archive.extract("./output", {
  strip: 1, // Strip first path component
});
```

### Closing

```typescript
await archive.close();
// Always close to finalize the archive
```

## Common Patterns

### Backup Directory

```typescript
async function backup(sourceDir: string, outputPath: string) {
  const archive = new Bun.Archive(outputPath);
  await archive.addDirectory(sourceDir);
  await archive.close();
  console.log(`Backup created: ${outputPath}`);
}
```

### Extract with Validation

```typescript
async function safeExtract(archivePath: string, outputDir: string) {
  const archive = await Bun.Archive.open(archivePath);

  // Validate before extracting
  const entries = await archive.list();
  for (const entry of entries) {
    if (entry.path.includes("..")) {
      throw new Error(`Path traversal detected: ${entry.path}`);
    }
  }

  await archive.extract(outputDir);
  archive.close();
}
```

### Create from Memory

```typescript
const archive = new Bun.Archive("./config.tar.gz");
await archive.addFile("config.json", JSON.stringify(config));
await archive.addFile("README.md", "# Configuration");
await archive.close();
```

## Performance Notes

- gzip compression built-in
- No native addon required
- Streaming for large files
