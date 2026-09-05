---
name: bun-test
description: Configure Bun's built-in test runner with Jest-compatible APIs. Use when setting up testing infrastructure, writing unit/integration/snapshot tests, migrating from Jest, or configuring test coverage. 3-10x faster than Jest.
compatibility: Requires Bun 1.4+
allowed-tools: ["Bash", "Write", "Read"]
metadata:
  author: dale
  category: bun-runtime
  tags: "bun, testing, jest, unit-tests, tdd, parallel, coverage"
---

# Bun Test Configuration

Set up Bun's built-in test runner with Jest-compatible APIs and significantly faster execution (3-10x faster than Jest).

## What's new in Bun 1.4

- **`--parallel`** — run tests in parallel across CPU cores (1.3.13)
- **`--isolate`** — run each test file in its own process (1.3.13)
- **`--shard`** — split test suite across CI runners (1.3.13)
- **`--changed`** — run only tests affected by changed files (1.3.13)
- **`--grep`** — filter tests by name pattern (1.3.6)
- **`--only-failures`** / **`--rerun-failures`** — rerun only failing tests (1.3.1)
- **`retry`** / **`repeats`** — per-test retry and repeat (1.3.9)
- **Fake timers** — `useFakeTimers()` for time-dependent tests (1.3.4)
- **`vi` global** — global `vi` mock function (1.3.1)
- **`onTestFinished` hook** — post-test assertions (1.3.2)
- **`--pass-with-no-tests`** — don't fail when no tests match (1.3.1)
- **JUnit XML output** — `--reporter=junit` (1.3.13)

## Quick Reference

For detailed patterns, see:
- **Jest Migration**: [references/jest-migration.md](references/jest-migration.md) — Complete Jest to Bun migration guide
- **Mocking**: [references/mocking.md](references/mocking.md) — Mock functions, spies, module mocking
- **Examples**: [references/examples.md](references/examples.md) — Test patterns for APIs, databases, async code

## Core Workflow

### 1. Check Prerequisites

```bash
# Verify Bun installation
bun --version

# Check if project exists
ls -la package.json
```

### 2. Create Test Directory Structure

```
project/
├── src/
│   └── utils.ts
├── tests/
│   ├── unit/              # Unit tests
│   ├── integration/       # Integration tests
│   ├── fixtures/          # Test data
│   └── setup.ts          # Global setup
├── package.json
└── bunfig.toml           # Test configuration
```

### 3. Configure Bun Test

Create `bunfig.toml` in project root:

```toml
[test]
# Preload files before running tests
preload = ["./tests/setup.ts"]

# Code coverage
coverage = true
coverageDir = "coverage"
coverageThreshold = 80

# Timeouts (in milliseconds)
timeout = 5000

# Bail after first failure
bail = false

# Run tests in parallel (Bun 1.3.13+)
parallel = true

# Reporter
reporter = "verbose" # or "junit", "tap", "json"
```

### 4. Create Test Setup File

Create `tests/setup.ts`:

```typescript
import { beforeAll, afterAll, beforeEach, afterEach } from "bun:test";

// Global test setup
beforeAll(() => {
  console.log("🧪 Starting test suite");
  process.env.NODE_ENV = "test";
});

afterAll(() => {
  console.log("✅ Test suite complete");
});

// Reset mocks before each test
beforeEach(() => {
  // Clear mock state
});

afterEach(() => {
  // Cleanup after each test
});

// Global test utilities
globalThis.testHelpers = {
  wait: (ms: number) => new Promise(resolve => setTimeout(resolve, ms)),
};
```

### 5. Write First Test

Create `tests/unit/example.test.ts`:

```typescript
import { describe, it, expect, test } from "bun:test";

// Simple test
test("addition works", () => {
  expect(1 + 1).toBe(2);
});

// Describe blocks for organization
describe("Array utilities", () => {
  it("should filter even numbers", () => {
    const numbers = [1, 2, 3, 4, 5, 6];
    const evens = numbers.filter(n => n % 2 === 0);

    expect(evens).toEqual([2, 4, 6]);
    expect(evens).toHaveLength(3);
  });
});

// Async tests
describe("Async operations", () => {
  it("should handle promises", async () => {
    const result = await Promise.resolve(42);
    expect(result).toBe(42);
  });
});
```

For more test examples (API testing, database testing, etc.), see [references/examples.md](references/examples.md).

### 6. Run Tests

```bash
# Run all tests
bun test

# Run specific file
bun test tests/unit/utils.test.ts

# Run in parallel (Bun 1.3.13+)
bun test --parallel

# Run each file in isolation (Bun 1.3.13+)
bun test --isolate

# Watch mode
bun test --watch

# With coverage
bun test --coverage

# Filter by name pattern (Bun 1.3.6+)
bun test --grep="should handle"

# Run only changed tests (Bun 1.3.13+)
bun test --changed

# Run only previously failing tests (Bun 1.3.1+)
bun test --only-failures

# Shard tests across CI runners (Bun 1.3.13+)
bun test --shard=2/4

# Pass even if no tests match (Bun 1.3.1+)
bun test --pass-with-no-tests

# JUnit XML output (Bun 1.3.13+)
bun test --reporter=junit --coverage
```

### 7. Add Mocking

```typescript
import { describe, it, expect, mock, spyOn } from "bun:test";

describe("Mock functions", () => {
  it("should create mock functions", () => {
    const mockFn = mock((x: number) => x * 2);

    const result = mockFn(5);

    expect(result).toBe(10);
    expect(mockFn).toHaveBeenCalledTimes(1);
    expect(mockFn).toHaveBeenCalledWith(5);
  });

  it("should spy on methods", () => {
    const obj = {
      method: (x: number) => x * 2,
    };

    const spy = spyOn(obj, "method");

    obj.method(5);

    expect(spy).toHaveBeenCalledWith(5);
    expect(spy).toHaveReturnedWith(10);
  });
});
```

For advanced mocking patterns, see [references/mocking.md](references/mocking.md).

### 8. Use Fake Timers (Bun 1.3.4+)

```typescript
import { useFakeTimers } from "bun:test";

test("should control time", () => {
  const clock = useFakeTimers();

  setTimeout(() => {
    console.log("Delayed");
  }, 1000);

  // Fast-forward 1 second
  clock.tick(1000);

  clock.restore();
});
```

### 9. Use Retry and Repeats (Bun 1.3.9+)

```typescript
test("should retry on failure", async () => {
  // Retry up to 3 times
  test.retry(3);

  // Or repeat 5 times
  test.repeat(5);

  const result = await fetchFromFlakyService();
  expect(result).toBeDefined();
});
```

### 10. Global `vi` (Bun 1.3.1+)

Bun test exposes a global `vi` object compatible with vitest:

```typescript
test("should use vi", () => {
  const mockFn = vi.fn(() => "mocked");
  mockFn();
  expect(mockFn).toHaveBeenCalled();
});
```

### 11. Update package.json

Add test scripts:

```json
{
  "scripts": {
    "test": "bun test",
    "test:watch": "bun test --watch",
    "test:coverage": "bun test --coverage",
    "test:parallel": "bun test --parallel",
    "test:changed": "bun test --changed",
    "test:ui": "bun test --coverage --reporter=html"
  }
}
```

## Jest Migration

If migrating from Jest, see [references/jest-migration.md](references/jest-migration.md) for:
- Import updates (`@jest/globals` → `bun:test`)
- Mock syntax changes (`jest.fn()` → `mock()`)
- Configuration migration
- Compatibility notes

**Key changes:**
```typescript
// Before (Jest)
import { describe, it, expect } from '@jest/globals';
const mockFn = jest.fn();

// After (Bun)
import { describe, it, expect, mock } from 'bun:test';
const mockFn = mock();
```

## Common Test Patterns

### Testing Functions

```typescript
import { test, expect } from "bun:test";

function add(a: number, b: number): number {
  return a + b;
}

test("add function", () => {
  expect(add(2, 3)).toBe(5);
  expect(add(-1, 1)).toBe(0);
});
```

### Testing Errors

```typescript
test("should throw errors", () => {
  const throwError = () => {
    throw new Error("Something went wrong");
  };

  expect(throwError).toThrow("Something went wrong");
  expect(throwError).toThrow(Error);
});

test("should reject promises", async () => {
  const asyncReject = async () => {
    throw new Error("Async error");
  };

  await expect(asyncReject()).rejects.toThrow("Async error");
});
```

### Snapshot Testing

```typescript
test("should match snapshot", () => {
  const data = {
    id: 1,
    name: "Test User",
    email: "test@example.com",
  };

  expect(data).toMatchSnapshot();
});

test("should match inline snapshot", () => {
  const config = { theme: "dark", language: "en" };

  expect(config).toMatchInlineSnapshot(`
    {
      "theme": "dark",
      "language": "en"
    }
  `);
});
```

## Matchers Reference

Common matchers available:

```typescript
// Equality
expect(value).toBe(expected);           // ===
expect(value).toEqual(expected);        // Deep equality

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeDefined();
expect(value).toBeUndefined();

// Numbers
expect(number).toBeGreaterThan(3);
expect(number).toBeLessThan(5);

// Strings
expect(string).toMatch(/pattern/);
expect(string).toContain("substring");

// Arrays
expect(array).toContain(item);
expect(array).toHaveLength(3);

// Objects
expect(object).toHaveProperty("key");
expect(object).toMatchObject({ subset });

// Promises
await expect(promise).resolves.toBe(value);
await expect(promise).rejects.toThrow();

// Mock functions
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledTimes(3);
expect(mockFn).toHaveBeenCalledWith(arg1, arg2);
```

## Test Organization

### Setup and Teardown

```typescript
import { beforeAll, afterAll, beforeEach, afterEach, describe, it } from "bun:test";

describe("User service", () => {
  let db: Database;

  beforeAll(async () => {
    // Setup before all tests
    db = await connectToDatabase();
  });

  afterAll(async () => {
    // Cleanup after all tests
    await db.close();
  });

  beforeEach(async () => {
    // Reset before each test
    await db.clear();
  });

  it("should create user", async () => {
    const user = await db.users.create({ name: "Test" });
    expect(user.id).toBeDefined();
  });
});
```

## Coverage Configuration

View coverage report:

```bash
# Generate coverage
bun test --coverage

# View HTML report
bun test --coverage --reporter=html
open coverage/index.html
```

Set coverage thresholds in `bunfig.toml`:
```toml
[test]
coverage = true
coverageThreshold = 80  # Fail if coverage < 80%
```

## Debugging Tests

```bash
# Run with debugger
bun test --inspect

# Verbose output
bun test --verbose

# Show all test results
bun test --reporter=tap

# Filter by filename
bun test tests/unit/
```

## Performance

Bun test is significantly faster than Jest:

- **Jest**: ~15 seconds for 100 tests
- **Bun**: ~2 seconds for 100 tests

**5-10x faster execution!**

With `--parallel`, it scales across all CPU cores.

## Completion Checklist

- ✅ Test directory structure created
- ✅ `bunfig.toml` configured
- ✅ Test setup file created
- ✅ Example tests written
- ✅ Package.json scripts updated
- ✅ Tests run successfully
- ✅ Coverage configured (if needed)
- ✅ Parallel/isolated/shard configured (if needed)

## Next Steps

After basic setup:

1. **Write tests**: Add tests for critical business logic
2. **CI/CD**: Configure parallel/sharded tests in your pipeline
3. **Coverage**: Set up coverage reporting with thresholds
4. **Pre-commit**: Add pre-commit hooks to run tests
5. **Documentation**: Document testing patterns for the team

For detailed implementations, see the reference files linked above.
