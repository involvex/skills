# Agents.md

This document provides guidance for AI agents working with this codebase. It covers useful commands, technologies used across the project, and best practices to follow.

## Table of Contents

- [Useful Commands](#useful-commands)
- [Technologies](#technologies)
- [Best Practices and Guidelines](#best-practices-and-guidelines)

---

## Useful Commands

### Git Commands

```bash
# Clone and setup
git clone <repository-url>
cd <repository-name>

# Branch management
git checkout -b <branch-name>
git checkout <branch-name>
git branch -d <branch-name>

# Commit workflow
git add .
git commit -m "feat: add new feature"
git push origin <branch-name>

# Pull and merge
git pull origin <branch-name>
git merge <branch-name>

# View status
git status
git diff
git log --oneline -10
```

### Package Managers

```bash
# npm
npm install
npm install <package-name>
npm run dev
npm run build
npm test
npm run lint

# pnpm
pnpm install
pnpm add <package-name>
pnpm dev
pnpm build

# bun
bun install
bun add <package-name>
bun run dev
bun run build
bun test
```

### Node.js and TypeScript

```bash
# Node version management
nvm install 20
nvm use 20

# TypeScript
npx tsc --init
npx tsc --watch
npx tsc --noEmit

# ESLint
npx eslint .
npx eslint --fix .

# Prettier
npx prettier --write .
npx prettier --check .
```

### Docker

```bash
# Build and run
docker build -t <image-name> .
docker run -p 3000:3000 <image-name>

# Docker Compose
docker-compose up
docker-compose up -d
docker-compose down

# Cleanup
docker system prune
docker container prune
```

### Cloudflare CLI (Wrangler)

```bash
# Workers
npx wrangler dev
npx wrangler deploy
npx wrangler pages dev

# D1 Database
npx wrangler d1 execute <database-name> --local --command="SELECT * FROM table"

# KV
npx wrangler kv:key put <key> <value> --namespace-id=<namespace-id>
```

### Capacitor

```bash
# Add platforms
npx capacitor add android
npx capacitor add ios

# Sync
npx capacitor sync

# Build
npx capacitor build android
npx capacitor build ios

# Open native IDE
npx capacitor open android
npx capacitor open ios
```

### Expo

```bash
# Start development
npx expo start
npx expo start --web
npx expo start --android
npx expo start --ios

# Build
npx expo prebuild
npx expo run:android
npx expo run:ios

# EAS Build
eas build -p android
eas build -p ios
```

---

## Technologies

### Frontend Frameworks

| Technology   | Description                                        | Common Use Cases                |
| ------------ | -------------------------------------------------- | ------------------------------- |
| React        | UI library for building component-based interfaces | Web applications, SPAs          |
| Next.js      | React framework with SSR/SSG capabilities          | Full-stack web apps, e-commerce |
| React Native | Cross-platform mobile framework                    | iOS/Android mobile apps         |
| Expo         | React Native development platform                  | Faster mobile development       |
| Flutter      | Google's cross-platform framework                  | Cross-platform mobile apps      |

### Backend Technologies

| Technology | Description                  | Common Use Cases          |
| ---------- | ---------------------------- | ------------------------- |
| Node.js    | JavaScript runtime           | APIs, real-time apps      |
| Express    | Node.js web framework        | REST APIs, middleware     |
| GraphQL    | Query language for APIs      | Flexible data fetching    |
| Python     | General-purpose language     | Backend services, AI/ML   |
| Go         | Systems programming language | High-performance services |
| Swift      | iOS/macOS development        | iOS native apps           |
| Kotlin     | Android development          | Android native apps       |

### Databases and Storage

| Technology    | Description              | Type                 |
| ------------- | ------------------------ | -------------------- |
| PostgreSQL    | Relational database      | SQL                  |
| SQLite        | Lightweight SQL database | Embedded             |
| Prisma        | ORM for Node.js          | Query builder        |
| NeonDB        | Serverless Postgres      | Cloud SQL            |
| Supabase      | Firebase alternative     | Backend-as-a-Service |
| Cloudflare D1 | Serverless SQLite        | Edge SQL             |
| Cloudflare KV | Key-value store          | NoSQL                |
| Cloudflare R2 | S3-compatible storage    | Object storage       |
| Redis         | In-memory data store     | Caching, sessions    |

### Cloud Platforms

| Platform   | Services Used                   |
| ---------- | ------------------------------- |
| Cloudflare | Workers, Pages, D1, KV, R2, AI  |
| Vercel     | Next.js hosting, Edge functions |
| AWS        | Lambda, S3, RDS, EC2            |
| Firebase   | Auth, Firestore, Functions      |

### DevOps and Tools

| Category         | Tools                               |
| ---------------- | ----------------------------------- |
| Containerization | Docker, Kubernetes                  |
| CI/CD            | GitHub Actions, CircleCI, GitLab CI |
| IaC              | Terraform, Pulumi                   |
| Monitoring       | Datadog, Sentry, CloudWatch         |

### AI and Agent Frameworks

| Framework                    | Purpose                         |
| ---------------------------- | ------------------------------- |
| GitHub Copilot SDK           | Building AI agent applications  |
| MCP (Model Context Protocol) | Connecting AI to external tools |
| OpenAI API                   | LLM integration                 |
| Anthropic Claude             | AI assistant integration        |

---

## Best Practices and Guidelines

### General Development

1. **Use Version Control**
   - Commit frequently with descriptive messages
   - Use feature branches for new work
   - Write meaningful commit messages following conventional commits format

2. **Environment Configuration**
   - Never commit secrets or credentials
   - Use `.env` files for local development
   - Follow twelve-factor app methodology for config

3. **Code Quality**
   - Write clean, readable code
   - Follow language-specific style guides
   - Use linters and formatters
   - Write tests for critical functionality

### TypeScript Best Practices

```typescript
// Prefer explicit types over 'any'
function greet(name: string): string {
  return `Hello, ${name}`;
}

// Use interfaces for object shapes
interface User {
  id: string;
  name: string;
  email: string;
}

// Use type aliases for unions
type Status = "pending" | "active" | "completed";

// Avoid null/undefined when possible
function getUser(id: string): User | null {
  // Return null instead of undefined
}
```

### React Best Practices

1. **Component Structure**
   - Keep components small and focused
   - Use composition over inheritance
   - Extract reusable logic into custom hooks

2. **State Management**
   - Use appropriate state solution for the scope
   - Local state: `useState`
   - Server state: React Query/SWR
   - Global state: Zustand, Redux Toolkit, or Context

3. **Performance**
   - Memoize expensive computations
   - Use `React.memo` for pure components
   - Lazy load heavy components with `React.lazy`

```typescript
// Good: Component composition
function Page() {
  return (
    <Layout>
      <Sidebar />
      <Content />
    </Layout>
  );
}

// Good: Custom hook for reusable logic
function useUser(userId: string) {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  return user;
}
```

### Cloudflare Workers Best Practices

1. **Performance**
   - Minimize `await` where not needed to avoid waterfalls
   - Use `Promise.all()` for independent operations
   - Cache responses appropriately

2. **Storage**
   - Use KV for key-value data
   - Use D1 for relational data
   - Use Durable Objects for stateful coordination

3. **Security**
   - Validate all inputs
   - Use proper CORS headers
   - Never log sensitive data

```typescript
// Good: Parallel fetching
export default {
  async fetch(request) {
    const [user, config] = await Promise.all([fetchUser(), fetchConfig()]);

    return new Response(JSON.stringify({ user, config }));
  },
};

// Bad: Sequential fetching
async function fetch(request) {
  const user = await fetchUser();
  const config = await fetchConfig();
  return new Response(JSON.stringify({ user, config }));
}
```

### Git Workflow

1. **Branch Naming**
   - `feature/description` - New features
   - `fix/description` - Bug fixes
   - `refactor/description` - Code improvements
   - `docs/description` - Documentation

2. **Commit Messages**

   ```
   feat: add user authentication
   fix: resolve login redirect issue
   refactor: simplify payment processing
   docs: update API documentation
   ```

3. **Pull Requests**
   - Keep PRs small and focused
   - Write clear descriptions
   - Include testing steps

### Security Guidelines

1. **Input Validation**
   - Validate all user inputs
   - Use parameterized queries
   - Sanitize data before rendering

2. **Authentication**
   - Use secure authentication methods
   - Implement proper session management
   - Store passwords with proper hashing

3. **Secrets Management**
   - Never commit secrets to git
   - Use environment variables for configuration
   - Rotate secrets regularly

### Testing Best Practices

1. **Unit Tests**
   - Test individual functions and components
   - Mock external dependencies
   - Aim for high coverage on business logic

2. **Integration Tests**
   - Test component interactions
   - Test API endpoints
   - Use realistic test data

3. **E2E Tests**
   - Test critical user flows
   - Use realistic browser automation
   - Run against staging environments

```typescript
// Example: Simple unit test
function sum(a: number, b: number): number {
  return a + b;
}

describe("sum", () => {
  it("adds two numbers", () => {
    expect(sum(2, 3)).toBe(5);
  });
});
```

### Performance Guidelines

1. **Frontend**
   - Minimize bundle size
   - Use code splitting
   - Optimize images and assets
   - Implement proper caching

2. **Backend**
   - Use connection pooling
   - Implement caching where appropriate
   - Optimize database queries
   - Use appropriate data structures

3. **Network**
   - Minimize requests
   - Compress responses
   - Use CDNs for static assets

### Documentation

1. **Code Comments**
   - Explain _why_, not _what_
   - Document complex algorithms
   - Keep comments up to date

2. **README Files**
   - Project overview
   - Setup instructions
   - Common commands
   - API documentation

3. **API Documentation**
   - Endpoint descriptions
   - Request/response formats
   - Error codes
   - Authentication requirements

---

## Quick Reference

### Common File Patterns

| File                 | Purpose                          |
| -------------------- | -------------------------------- |
| `package.json`       | Node.js dependencies and scripts |
| `tsconfig.json`      | TypeScript configuration         |
| `.eslintrc.json`     | ESLint rules                     |
| `docker-compose.yml` | Docker services definition       |
| `wrangler.toml`      | Cloudflare Workers config        |
| `app.json`           | Expo/React Native config         |

### Environment Variables

```bash
# Common variables
NODE_ENV=development
DATABASE_URL=postgresql://...
API_KEY=your-api-key
SECRET_KEY=your-secret
```

### Checking Available Skills

When working on a task, check if a relevant skill exists:

- **Refactoring**: Use the `refactor` skill for code improvements
- **Code Review**: Use the `code-reviewer` skill for reviews
- **Frontend Design**: Use `frontend-design` skill for UI tasks
- **Debugging**: Use `systematic-debugging` skill for bug fixes
- **Cloudflare**: Use `cloudflare` skill for Workers development
- **React**: Use `react-best-practices` for React optimization

---

_Last updated: March 2026_
