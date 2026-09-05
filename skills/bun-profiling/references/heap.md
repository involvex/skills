# Heap Profiling Reference

Profile memory usage with Bun's built-in heap profiler.

## --heap-prof

Generate a `.heapsnapshot` file compatible with Chrome DevTools:

```bash
bun --heap-prof ./app.ts
```

Open in Chrome DevTools → Memory tab → Load snapshot.

## --heap-prof-md

Generate a Markdown heap profile report:

```bash
bun --heap-prof-md ./app.ts
```

Output includes:
- Total heap size and object count
- Top 50 types by retained size
- Largest objects
- GC roots
- Object reference chains

Example output:
```markdown
# Bun Heap Profile

| Metric | Value |
|--------|-------|
| Total Heap Size | 4.2 MB |
| Total Objects | 121,116 |
| Unique Types | 67 |
| GC Roots | 427 |

## Top 10 Types by Retained Size

| Type | Count | Retained Size |
|------|-------|---------------|
| `string` | 119,883 | 4.1 MB |
| `GlobalObject` | 1 | 83.1 KB |
| `Function` | 319 | 61.5 KB |
```

## Finding Memory Leaks

1. Take a heap snapshot after app startup
2. Perform the operation suspected of leaking
3. Take another heap snapshot
4. Compare — look for growing object counts of specific types

Common leak sources:
- Event listeners not removed
- Closures capturing large objects
- Caches without eviction
- Circular references (though GC handles these)

## Quick Commands

```bash
# Find all Function objects
grep '| `Function`' heap.md

# Find all GC roots
grep 'gcroot=1' heap.md

# Find specific object
grep '| 12345 |' heap.md
```

## When to Use

| Tool | When |
|---|---|
| `--heap-prof` | Deep dive in DevTools |
| `--heap-prof-md` | Quick analysis, share with team/LLM |
| `grep` on Markdown | Find specific types or objects |
