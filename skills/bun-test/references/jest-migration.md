# Migrating from Jest to Bun Test

Complete guide for migrating Jest test suites to Bun's built-in test runner.

## API Compatibility

Bun test provides Jest-compatible APIs for seamless migration:

| Jest API | Bun Test | Status |
|----------|----------|--------|
| `describe`, `it`, `test` | ✅ Identical | Fully supported |
| `expect` with matchers | ✅ Identical | Most matchers supported |
| `beforeAll`, `afterAll` | ✅ Identical | Fully supported |
| `beforeEach`, `afterEach` | ✅ Identical | Fully supported |
| `jest.fn()` | `mock()` | Different import |
| `jest.spyOn()` | `spyOn()` | Different import |
| Snapshot testing | ✅ Identical | Fully supported |
| Async testing | ✅ Identical | Fully supported |
| `jest.useFakeTimers()` | `useFakeTimers()` | Different import (Bun 1.3.4+) |
| `jest.retryTimes()` | `test.retry()` | Per-test retry (Bun 1.3.9+) |

## Migration Steps

### 1. Update Imports

**Before (Jest):**
```typescript
import { describe, it, expect } from '@jest/globals';
import { jest } from '@jest/globals';
```

**After (Bun):**
```typescript
import { describe, it, expect, mock, spyOn } from 'bun:test';
```

### 2. Update Mock Syntax

**Before (Jest):**
```typescript
const mockFn = jest.fn();
jest.fn((x) => x * 2);
jest.spyOn(obj, 'method');
```

**After (Bun):**
```typescript
const mockFn = mock();
mock((x) => x * 2);
spyOn(obj, 'method');
```

### 3. Update Configuration

**Remove Jest config files:**
```bash
rm jest.config.js
rm jest.setup.js
```

**Create bunfig.toml:**
```toml
[test]
preload = ["./tests/setup.ts"]
coverage = true
coverageDir = "coverage"
coverageThreshold = 80
timeout = 5000
parallel = true
```

### 4. Update package.json

**Before:**
```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "devDependencies": {
    "jest": "^29.0.0",
    "@types/jest": "^29.0.0"
  }
}
```

**After:**
```json
{
  "scripts": {
    "test": "bun test",
    "test:watch": "bun test --watch",
    "test:coverage": "bun test --coverage",
    "test:parallel": "bun test --parallel"
  }
}
```

Remove Jest dependencies:
```bash
bun remove jest @types/jest ts-jest
```

## Feature Comparison

| Feature | Jest | Bun Test |
|---------|------|----------|
| Speed | Baseline | 5-10x faster |
| Parallel execution | Worker pools | Native `--parallel` |
| Snapshot testing | ✅ | ✅ |
| Mock functions | `jest.fn()` | `mock()` |
| Module mocking | `jest.mock()` | Manual/DI patterns |
| Fake timers | `jest.useFakeTimers()` | `useFakeTimers()` (1.3.4+) |
| Coverage | ✅ | ✅ |
| Watch mode | ✅ | ✅ |
| Retry | Limited | `test.retry()` (1.3.9+) |
| Sharding | `--shard` | `--shard` (1.3.13+) |

## New Bun Test Features

Take advantage of Bun-specific features:

### Parallel Execution (Bun 1.3.13+)

```bash
# Run all tests in parallel across CPU cores
bun test --parallel

# Run each file in isolation (separate process)
bun test --isolate
```

### Sharding for CI (Bun 1.3.13+)

```bash
# Split tests across 4 CI runners
bun test --shard=1/4  # Runner 1
bun test --shard=2/4  # Runner 2
bun test --shard=3/4  # Runner 3
bun test --shard=4/4  # Runner 4
```

### Only-Failures Mode (Bun 1.3.1+)

```bash
# Run only tests that failed last time
bun test --only-failures

# Rerun failures up to N times
bun test --rerun-failures=3
```

### Changed Files Only (Bun 1.3.13+)

```bash
# Run only tests affected by changed files
bun test --changed
```

### Grep Filter (Bun 1.3.6+)

```bash
# Run tests matching a pattern
bun test --grep="should handle"
```

## Migration Checklist

- [ ] Update imports (`@jest/globals` → `bun:test`)
- [ ] Replace `jest.fn()` with `mock()`
- [ ] Replace `jest.spyOn()` with `spyOn()`
- [ ] Remove Jest config files
- [ ] Create `bunfig.toml`
- [ ] Update package.json scripts
- [ ] Remove Jest dependencies
- [ ] Run tests to verify
- [ ] Enable `--parallel` for faster CI
- [ ] Update CI/CD pipelines
- [ ] Update documentation

## Performance Comparison

Bun test is significantly faster:

```bash
# Jest
npm test  # ~15 seconds for 100 tests

# Bun
bun test  # ~2 seconds for 100 tests
bun test --parallel  # ~0.5 seconds with parallelism
```

**5-10x faster execution, up to 30x with parallel!**
