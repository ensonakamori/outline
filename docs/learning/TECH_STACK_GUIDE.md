# Technology Stack Guide

**Deep dive into every technology used in Outline.**

**Documentation Date:** November 18, 2025
**Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)

---

## Core Stack

This guide provides context and learning resources for each technology. For version analysis and currency notes, see [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md).

---

## Frontend Technologies

### React 17.0.2

⚠️ **Project uses React 17** (Latest: React 19.2 - Nov 2025)

**What it is:** JavaScript library for building user interfaces

**Why Outline uses it:**
- Component-based architecture
- Large ecosystem
- Strong TypeScript support
- Mature and stable

**How it's used:**
- All UI components in `app/components/`
- Pages in `app/scenes/`
- React Router for navigation

**Key patterns in this project:**
- Function components with hooks
- React Router v5 (not v6)
- No Server Components (React 18+)
- No Suspense for data fetching

🧠 **Mental Model:** Components are like reusable HTML templates that can accept data (props) and manage their own state (hooks).

**Learn more:**
- Docs: https://react.dev/
- ⚠️ Latest docs show React 19 patterns; some differ from v17

---

### MobX 4.15.4

⚠️ **Project uses MobX 4 decorators** (Latest: MobX 6.15 - Nov 2025)

**What it is:** State management library using reactive programming

**Why Outline uses it:**
- Less boilerplate than Redux
- Automatic re-rendering
- Object-oriented approach fits well with models

**How it's used:**
- Stores in `app/stores/`
- Models in `app/models/`
- `@observable`, `@action`, `@computed` decorators

**Decorators pattern (MobX 4):**
```typescript
class DocumentsStore {
  @observable documents = new Map()
  
  @action
  add(doc) {
    this.documents.set(doc.id, doc)
  }
  
  @computed
  get sortedDocs() {
    return Array.from(this.documents.values()).sort(...)
  }
}
```

🌉 **Bridge from React:** MobX is like Redux but:
- No reducers (just modify state directly in `@action`)
- No selectors (use `@computed` instead)
- No connect() (use `observer()` HOC)
- Automatic subscription (no manual selectors)

⚠️ **Note:** Modern MobX 6 uses `makeObservable()` instead of decorators.

**Learn more:**
- Docs: https://mobx.js.org/
- MobX 4 patterns: Look for "decorator" examples
- ⚠️ Latest docs show MobX 6 patterns

---

### Styled Components 5.3.11

⚠️ **Project uses v5** (Latest: v6.1.15 - Nov 2025)

**What it is:** CSS-in-JS library

**Why Outline uses it:**
- Scoped styles (no global CSS conflicts)
- Dynamic styling based on props
- Theme support
- Server-side rendering

**How it's used:**
- Component styles co-located with components
- Global theme in `app/styles/theme.ts`

**Example:**
```typescript
const Button = styled.button<{ primary?: boolean }>`
  background: ${props => props.primary ? 'blue' : 'gray'};
  padding: 10px 20px;
`
```

🧠 **Mental Model:** Write CSS inside JavaScript with template literals.

**Learn more:**
- Docs: https://styled-components.com/

---

### ProseMirror

✅ **Current and actively maintained**

**What it is:** Toolkit for building rich text editors

**Why Outline uses it:**
- Highly customizable
- Excellent collaborative editing support (Yjs)
- Used by Notion, Atlassian, NYTimes

**How it's used:**
- Editor code in `shared/editor/`
- Custom nodes and marks
- Plugins for features

🧠 **Mental Model:** ProseMirror is like a framework for editors, not a ready-made editor. You build your editor using its primitives.

**Core concepts:**
- **Schema:** Defines document structure
- **State:** Current editor state
- **View:** DOM representation
- **Transactions:** State changes
- **Plugins:** Add functionality

**Learn more:**
- Docs: https://prosemirror.net/
- Guide: https://prosemirror.net/docs/guide/

---

## Backend Technologies

### Node.js 22.x

⚠️ **Project uses Node 22** (Latest LTS: Node 24 - Nov 2025)

**What it is:** JavaScript runtime built on Chrome's V8 engine

**Why Outline uses it:**
- JavaScript everywhere (frontend + backend)
- Huge ecosystem (npm)
- Non-blocking I/O (good for I/O-heavy apps)

**How it's used:**
- Runs backend server
- Build tools
- Development servers

✅ **Node 22 is still supported** until October 2027.

**Learn more:**
- Docs: https://nodejs.org/docs/latest-v22.x/api/

---

### Koa 3.0.3

✅ **Current** (Latest: 3.1.0 - Nov 2025)

**What it is:** Minimalist web framework for Node.js (by Express team)

**Why Outline uses it:**
- Async/await support (cleaner than Express callbacks)
- Lightweight (small core)
- Middleware composition

**How it's used:**
- Main server in `server/index.ts`
- Routes in `server/routes/`
- Middleware in `server/middlewares/`

**Example:**
```typescript
app.use(async (ctx, next) => {
  const start = Date.now()
  await next()
  const ms = Date.now() - start
  console.log(`${ctx.method} ${ctx.url} - ${ms}ms`)
})
```

🌉 **Bridge from Express:** Koa is like Express but:
- Uses `async/await` instead of callbacks
- `ctx` (context) instead of `req`/`res`
- Cleaner error handling (try/catch)

**Learn more:**
- Docs: https://koajs.com/

---

### Sequelize 6.37.7

✅ **Current stable**

**What it is:** ORM (Object-Relational Mapper) for Node.js

**Why Outline uses it:**
- TypeScript support (via sequelize-typescript)
- Migrations
- Query builder
- Supports PostgreSQL features

**How it's used:**
- Models in `server/models/`
- Migrations in `server/migrations/`
- Decorators for model definition

**Example:**
```typescript
@Table
class Document extends Model {
  @Column
  title: string
  
  @BelongsTo(() => User)
  user: User
}
```

🌉 **Bridge from other ORMs:**
- Like TypeORM but older API
- Like Prisma but no schema file (defined in code)
- Like ActiveRecord (Rails) but for JavaScript

**Learn more:**
- Docs: https://sequelize.org/

---

### PostgreSQL

✅ **Industry standard**

**What it is:** Open-source relational database

**Why Outline uses it:**
- Full-text search (tsvector/tsquery)
- JSONB support
- Arrays and advanced types
- Reliable and mature

**How it's used:**
- Primary data store
- Full-text search for documents
- Stores collaborative editing history

**Key features used:**
- Full-text search
- JSONB columns
- Row-level security (via team isolation)
- Indexes

**Learn more:**
- Docs: https://www.postgresql.org/docs/

---

### Redis

✅ **Current and widely used**

**What it is:** In-memory data store

**Why Outline uses it:**
- Fast caching
- Job queue (Bull)
- Session storage
- Pub/sub for WebSockets

**How it's used:**
- Cache layer
- Bull job queue
- Session management
- WebSocket message broker

**Learn more:**
- Docs: https://redis.io/docs/

---

## Real-Time Technologies

### Yjs 13.6.1

✅ **Current**

**What it is:** CRDT (Conflict-free Replicated Data Type) for collaborative editing

**Why Outline uses it:**
- Automatic conflict resolution
- Offline-first
- Integrates with ProseMirror

**How it works:**
- Each client has a Y.Doc
- Changes are Y.js operations
- Operations are commutative (order doesn't matter)
- Automatic merging

🧠 **Mental Model:** Like Git for real-time documents - everyone can edit simultaneously and changes merge automatically.

**Learn more:**
- Docs: https://docs.yjs.dev/

---

### Socket.io 4.8.1

✅ **Current**

**What it is:** WebSocket library with fallbacks

**Why Outline uses it:**
- Real-time notifications
- Presence awareness
- Automatic reconnection

**How it's used:**
- Real-time updates (document changes, user presence)
- Separate from Yjs collaboration (which uses Hocuspocus)

**Learn more:**
- Docs: https://socket.io/

---

## Build Tools

### TypeScript 5.9.2

✅ **Current**

**What it is:** Typed superset of JavaScript

**Why Outline uses it:**
- Type safety
- Better IDE support
- Catches errors early

**How it's used:**
- All code is TypeScript
- `tsconfig.json` configuration
- Compiled to JavaScript for execution

**Learn more:**
- Docs: https://www.typescriptlang.org/

---

### Vite (Custom: rolldown-vite)

🔍 **Custom build** (Standard Vite: 7.2.2 - Nov 2025)

**What it is:** Fast build tool using ES modules

**Why Outline uses rolldown-vite:**
- Rolldown is a Rust-based bundler (experimental)
- Faster than standard Vite
- Likely for performance improvements

**How it's used:**
- Frontend dev server (`yarn vite:dev`)
- Production builds (`yarn vite:build`)
- Hot module replacement (HMR)

⚠️ **Note:** This is a custom fork, not standard Vite.

**Learn more:**
- Vite Docs: https://vite.dev/
- Rolldown: https://github.com/rolldown-rs/rolldown

---

## Testing

### Jest 29.7.0

✅ **Current**

**What it is:** JavaScript testing framework

**Why Outline uses it:**
- Good TypeScript support
- Rich matchers
- Mocking capabilities

**How it's used:**
- All `.test.ts` files
- Separate projects for app/server/shared
- JSDOM for React testing

**Learn more:**
- Docs: https://jestjs.io/

---

## Background Jobs

### Bull 4.16.5

✅ **Current**

**What it is:** Job queue built on Redis

**Why Outline uses it:**
- Reliable job processing
- Retries and backoff
- Priority queues
- Web dashboard

**How it's used:**
- Background tasks in `server/queues/tasks/`
- Event processors in `server/queues/processors/`
- Job dashboard via Bull Board

**Learn more:**
- Docs: https://github.com/OptimalBits/bull

---

## Learning Path

**If you're new to:**

**React:**
1. Read official tutorial
2. Explore `app/components/` (simple components first)
3. Note: Uses React 17 patterns (not 19)

**MobX:**
1. Read MobX docs (look for decorator examples)
2. Study `app/stores/RootStore.ts`
3. Note: Uses MobX 4 decorators

**Koa:**
1. Read Koa docs
2. Study `server/routes/api/` (simple endpoints)
3. Compare to Express if familiar

**Sequelize:**
1. Read Sequelize docs
2. Study `server/models/` (start with simple models)
3. Understand decorators

**ProseMirror:**
1. Read ProseMirror guide (it's complex!)
2. Study `shared/editor/nodes/` (start with simple nodes)
3. Build a small editor as practice

---

## Next Steps

- [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) - React + MobX details
- [Backend Architecture](./BACKEND_ARCHITECTURE.md) - Koa + Sequelize details
- [Database Architecture](./DATABASE_ARCHITECTURE.md) - Data model details
- [How-To Guide](./HOW_TO_GUIDE.md) - Practical examples

---

*Last Updated: November 18, 2025*
