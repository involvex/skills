# Bun.redis — Redis Client

Built-in Redis client. Pub/Sub supported.

## Quick Start

```typescript
const redis = new Bun.Redis("redis://localhost:6379");

// Key-value operations
await redis.set("key", "value", { ex: 3600 });
const value = await redis.get("key");

// Pub/Sub
redis.subscribe("channel", (message) => {
  console.log("Received:", message);
});
```

## API Reference

### Constructor

```typescript
const redis = new Bun.Redis(url: string);
// or
const redis = new Bun.Redis({
  host: "localhost",
  port: 6379,
  password?: string,
  db?: number,
});
```

### String Operations

```typescript
await redis.set("key", "value");
await redis.set("key", "value", { ex: 3600 }); // With TTL
await redis.set("key", "value", { nx: true }); // Only if not exists

const value = await redis.get("key");
const exists = await redis.exists("key");
await redis.del("key");
await redis.expire("key", 3600);
await redis.ttl("key");
```

### Hash Operations

```typescript
await redis.hset("user:1", { name: "Alice", age: "30" });
const name = await redis.hget("user:1", "name");
const all = await redis.hgetall("user:1");
await redis.hdel("user:1", "age");
```

### List Operations

```typescript
await redis.lpush("queue", "item1", "item2");
await redis.rpush("queue", "item3");
const items = await redis.lrange("queue", 0, -1);
await redis.lpop("queue");
```

### Set Operations

```typescript
await redis.sadd("tags", "typescript", "bun");
const members = await redis.smembers("tags");
await redis.srem("tags", "typescript");
```

### Pub/Sub

```typescript
// Subscribe
redis.subscribe("channel", (message) => {
  console.log("Received:", message);
});

// Publish
await redis.publish("channel", "hello world");
```

### Batch Commands

```typescript
const results = await redis.batch([
  ["get", "key1"],
  ["get", "key2"],
  ["set", "key3", "value"],
]);
```

## Common Patterns

### Caching

```typescript
async function getUser(id: string) {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);

  const user = await db.users.find(id);
  await redis.setex(`user:${id}`, 3600, JSON.stringify(user));
  return user;
}
```

### Rate Limiting

```typescript
const key = `ratelimit:${ip}`;
const count = await redis.incr(key);
if (count === 1) await redis.expire(key, 60);

if (count > 100) {
  return new Response("Rate limit exceeded", { status: 429 });
}
```

## Performance Notes

- No native addon required — pure Zig implementation
- Connection pooling built-in
- Lower latency than `ioredis` for simple operations
