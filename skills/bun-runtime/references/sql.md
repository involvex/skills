# Bun.SQL — Database Client

Unified SQL API for PostgreSQL, MySQL, and SQLite.

## Quick Start

```typescript
import { SQL } from "bun";

// PostgreSQL
const pg = new SQL("postgres://user:pass@localhost/db");
const users = await pg`SELECT * FROM users WHERE age > ${18}`;

// MySQL
const mysql = new SQL("mysql://user:pass@localhost/db");
const rows = await mysql`SHOW TABLES`;

// SQLite
const sqlite = new SQL(":memory:");
await sqlite`CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)`;
```

## Connection Strings

### PostgreSQL

```typescript
const sql = new SQL("postgres://user:password@localhost:5432/mydb");
// With options
const sql = new SQL({
  url: "postgres://user:pass@localhost/mydb",
  max: 10, // Connection pool size
});
```

### MySQL

```typescript
const sql = new SQL("mysql://user:password@localhost:3306/mydb");
```

### SQLite

```typescript
const sql = new SQL(":memory:");
// or
const sql = new SQL("./database.sqlite");
```

## Queries

### Tagged Template Literals

```typescript
const name = "Alice";
const age = 30;

const users = await sql`
  SELECT id, name, email
  FROM users
  WHERE name = ${name} AND age > ${age}
`;
```

Tagged templates automatically escape values — no SQL injection risk.

### Raw Queries

```typescript
const result = await sql`SELECT * FROM users`;
for (const row of result) {
  console.log(row.name);
}
```

### Transactions

```typescript
await sql.begin();
try {
  await sql`INSERT INTO users (name) VALUES ('Alice')`;
  await sql`INSERT INTO profiles (user_id) VALUES (LAST_INSERT_ID())`;
  await sql.commit();
} catch (error) {
  await sql.rollback();
  throw error;
}
```

## Common Patterns

### Select with Pagination

```typescript
const page = 1;
const limit = 20;
const offset = (page - 1) * limit;

const users = await sql`
  SELECT * FROM users
  ORDER BY created_at DESC
  LIMIT ${limit} OFFSET ${offset}
`;
```

### Insert and Get ID

```typescript
const result = await sql`
  INSERT INTO users (name, email)
  VALUES (${name}, ${email})
`;

const id = result.lastInsertRowid;
```

### Batch Operations

```typescript
const users = [
  { name: "Alice", email: "alice@example.com" },
  { name: "Bob", email: "bob@example.com" },
];

for (const user of users) {
  await sql`INSERT INTO users (name, email) VALUES (${user.name}, ${user.email})`;
}
```

### SQLite Specific

```typescript
// In-memory database
const sqlite = new SQL(":memory:");

// File-based
const sqlite = new SQL("./app.sqlite");

// Enable WAL mode
await sqlite`PRAGMA journal_mode = WAL`;
```

## Performance Notes

- Single API across PostgreSQL, MySQL, and SQLite
- Connection pooling built-in
- Prepared statements cached automatically
- Lower overhead than `pg` / `mysql2` packages
