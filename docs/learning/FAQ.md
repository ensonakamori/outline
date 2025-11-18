# Frequently Asked Questions (FAQ)

**Quick answers to common questions about Outline development.**

**Documentation Date:** November 18, 2025

---

## Getting Started

### Q: What Node.js version should I use?

**A:** Use Node.js 22.x (specified in `.nvmrc`). The latest LTS is Node 24 (Nov 2025), but Outline is tested with Node 22 and will be upgraded in the future.

```bash
nvm install 22
nvm use 22
```

### Q: Why does the project use React 17 instead of React 19?

**A:** React 17 is used for stability. React 19 (latest as of Nov 2025) introduced major breaking changes including:
- New compiler
- Actions API
- Server Components
- Removal of forwardRef (used extensively in this codebase)

Upgrading would require significant refactoring. See [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md) for details.

### Q: Why MobX 4 instead of MobX 6?

**A:** MobX 4 uses decorators (`@observable`, `@action`) which provide a cleaner API for this codebase. MobX 6 uses `makeObservable` which is more verbose. The entire frontend would need refactoring to upgrade.

### Q: Do I need Docker to develop?

**A:** No, but it's recommended for PostgreSQL and Redis. You can install them locally:
- PostgreSQL: `brew install postgresql@14`
- Redis: `brew install redis`

Or use Docker: `docker-compose up -d redis postgres`

### Q: Why is the build so slow?

**A:** First build compiles TypeScript, bundles frontend, and processes i18n. Subsequent builds are faster with caching. Use `yarn dev:watch` for hot reload during development.

---

## Architecture

### Q: What's the difference between `app/`, `server/`, and `shared/`?

**A:**
- **app/** - Frontend React code (runs in browser)
- **server/** - Backend Koa code (runs in Node.js)
- **shared/** - Universal code (runs in both)

The ProseMirror editor lives in `shared/` because it's used both client-side and server-side for document processing.

### Q: What is the Command pattern used in `server/commands/`?

**A:** Commands encapsulate business logic (e.g., creating a document) in reusable classes. This separates business logic from HTTP routes and allows the same logic to be used in API endpoints, background jobs, and tests.

Example: `documentCreator.ts` is used by:
- API endpoint: `POST /api/documents`
- Background job: Import processor
- Tests: Document creation tests

### Q: How does authorization work?

**A:** Authorization uses a policy-based system (similar to CanCan):
- **Policies** (in `server/policies/`) define who can do what
- **Middleware** checks policies before allowing actions
- **Frontend** uses PoliciesStore to show/hide UI elements

Example:
```typescript
// Backend
const can = authorize(user, "update", document)

// Frontend
if (policies.can("update", document)) {
  // Show edit button
}
```

### Q: What is the plugin system?

**A:** Plugins are modular extensions in `plugins/` directory. Each plugin can provide:
- Authentication providers (OAuth)
- API endpoints
- Background processors
- Email templates
- Storage adapters

Plugins auto-load on startup via `PluginManager`. See [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md).

### Q: How does real-time collaboration work?

**A:** Outline uses **Yjs** (CRDT) + **Hocuspocus** (WebSocket server):
1. Users connect via WebSocket
2. Edits are broadcast as Y.js operations
3. Conflicts are automatically resolved
4. Changes sync to all connected clients
5. Documents saved to PostgreSQL periodically

See: [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md)

---

## Development

### Q: Do I need to rebuild after every change?

**A:** No! Use `yarn dev:watch` for hot reload:
- Frontend: Vite HMR (instant updates)
- Backend: Nodemon restarts on file changes

Only rebuild for production: `yarn build`

### Q: How do I add a new API endpoint?

**A:** See [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md#add-api-endpoint), but quick steps:
1. Create route file in `server/routes/api/[resource]/`
2. Add route to `server/routes/api/index.ts`
3. Create command in `server/commands/`
4. Add policy check
5. Add presenter for response
6. Write tests

### Q: How do I add a new database field?

**A:**
1. Create migration: `yarn db:create-migration --name add-field`
2. Edit migration file in `server/migrations/`
3. Add column to model in `server/models/`
4. Run migration: `yarn db:migrate`
5. Update presenters if needed

### Q: Where do uploaded files go?

**A:**
- **Development:** `./data/` (local filesystem)
- **Production:** S3-compatible storage (configurable via env vars)

### Q: How do background jobs work?

**A:** Bull + Redis queue system:
1. Model emits event (e.g., `document.update`)
2. Processor (`queues/processors/`) handles event
3. Processor creates task (`queues/tasks/`)
4. Worker process executes task
5. Job completes or retries on failure

### Q: How do I debug the backend?

**A:**
```bash
# Enable debug logs
DEBUG=* yarn dev:backend

# Or specific categories
DEBUG=database,http yarn dev:backend

# Use Node inspector (Chrome DevTools)
yarn dev:backend
# Then open chrome://inspect
```

### Q: How do I debug the frontend?

**A:**
- React DevTools browser extension
- MobX DevTools (search npm for "mobx-devtools-mst")
- Browser debugger: Add `debugger` statement
- Console: `window.stores` to access stores

---

## Testing

### Q: How do I run tests?

**A:**
```bash
# All tests
yarn test

# Frontend only
yarn test:app

# Backend only
yarn test:server

# Specific file
yarn test path/to/file.test.ts

# Watch mode
yarn test path/to/file.test.ts --watch
```

### Q: Do I need to run migrations for tests?

**A:** No, test database is created automatically on first `yarn test`. It creates a separate `outline_test` database.

### Q: How do I add test data?

**A:** Use factories in `server/test/factories/`:
```typescript
import { buildUser, buildDocument } from "@server/test/factories"

const user = await buildUser()
const document = await buildDocument({ userId: user.id })
```

---

## Database

### Q: How do I reset the database?

**A:**
```bash
yarn db:reset  # Drops, creates, and migrates
```

Or manually:
```bash
psql postgres
DROP DATABASE outline;
CREATE DATABASE outline;
\q
yarn db:migrate
```

### Q: How do I rollback a migration?

**A:**
```bash
yarn db:rollback  # Rolls back last migration
```

### Q: What's the difference between soft delete and hard delete?

**A:**
- **Soft delete:** Sets `deletedAt` timestamp (model still in DB)
- **Hard delete:** Permanently removes from DB

Most models use soft delete (Sequelize paranoid mode). This allows:
- Undo/restore functionality
- Audit trails
- Data recovery

### Q: How does multi-tenancy work?

**A:** Every model belongs to a **Team**. Database queries automatically filter by team:
```typescript
// Sequelize automatically adds: WHERE teamId = currentUser.teamId
Document.findAll()  // Only returns documents for current team
```

This ensures team isolation (team A can't access team B's data).

---

## Deployment

### Q: How do I deploy to production?

**A:** See official docs: https://docs.getoutline.com/s/hosting/

Quick overview:
1. Set `NODE_ENV=production`
2. Configure PostgreSQL and Redis
3. Set secrets (`SECRET_KEY`, `UTILS_SECRET`)
4. Run migrations: `yarn db:migrate`
5. Build: `yarn build`
6. Start: `yarn start`

### Q: What environment variables are required?

**A:** Minimum:
```env
NODE_ENV=production
URL=https://your-domain.com
DATABASE_URL=postgres://...
REDIS_URL=redis://...
SECRET_KEY=...
UTILS_SECRET=...
```

Plus one authentication provider (Google, Slack, OIDC, or Email/SMTP).

See `.env.sample` for all options.

### Q: Can I use SQLite instead of PostgreSQL?

**A:** No, Outline requires PostgreSQL for:
- Full-text search (tsvector/tsquery)
- JSONB support
- Array columns
- Advanced indexing

### Q: Can I use a different cache instead of Redis?

**A:** No, Redis is required for:
- Bull job queue
- Session storage
- WebSocket scaling
- Cache

---

## Common Errors

### Q: "Port 3000 already in use"

**A:**
```bash
# Find process
lsof -i :3000

# Kill it
kill -9 <PID>

# Or use different port
PORT=3001 yarn dev:watch
```

### Q: "Database connection failed"

**A:**
1. Check PostgreSQL is running: `psql -U user -d outline`
2. Verify `DATABASE_URL` in `.env`
3. For local dev, add `PGSSLMODE=disable` to `.env`
4. Check Docker: `docker-compose ps`

### Q: "Redis connection failed"

**A:**
```bash
# Check Redis is running
redis-cli ping  # Should return "PONG"

# Start Redis
brew services start redis  # macOS
sudo systemctl start redis  # Linux
docker-compose up -d redis  # Docker
```

### Q: "Module not found" errors

**A:**
```bash
# Reinstall dependencies
rm -rf node_modules
yarn install

# Clear cache
yarn clean
yarn build
```

### Q: "Out of memory" during build

**A:**
```bash
export NODE_OPTIONS="--max-old-space-size=4096"
yarn build
```

### Q: "Migration already executed" error

**A:** Migration was already run. To re-run:
```bash
yarn db:rollback  # Undo last migration
yarn db:migrate   # Re-run
```

---

## Code Style

### Q: What code style does Outline use?

**A:**
- **Formatter:** Prettier (automatic)
- **Linter:** oxlint (faster than ESLint)
- **Pre-commit:** Husky + lint-staged

Run before committing:
```bash
yarn format  # Format all files
yarn lint    # Lint all files
```

### Q: Should I use semicolons?

**A:** Prettier adds them automatically. Don't worry about it.

### Q: Tabs or spaces?

**A:** 2 spaces (configured in Prettier).

### Q: How do I name files?

**A:**
- Components: PascalCase (`Avatar.tsx`)
- Utils: camelCase (`dateUtils.ts`)
- Models: PascalCase singular (`Document.ts`)
- Tests: Same as file + `.test.ts` (`Document.test.ts`)

---

## Performance

### Q: Why is the app slow in development?

**A:** Development mode includes:
- Source maps
- Hot module replacement
- Verbose logging
- No minification

Production builds are much faster: `yarn build && yarn start`

### Q: How do I profile performance?

**A:**
**Frontend:**
- React DevTools Profiler
- Chrome DevTools Performance tab
- Lighthouse

**Backend:**
- `DEBUG=database` to see query times
- Datadog APM (if configured)
- Node.js `--prof` flag

### Q: How can I reduce bundle size?

**A:** Bundle size is optimized by Vite:
- Code splitting (automatic)
- Tree shaking
- Minification
- Lazy loading routes

Check bundle stats:
```bash
yarn vite:build --mode analyze
```

---

## Contributing

### Q: How do I contribute?

**A:** See [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md):
1. Find an issue (look for `good first issue` label)
2. Comment on the issue to claim it
3. Fork the repo
4. Create a branch
5. Make changes + write tests
6. Submit PR
7. Address review feedback

### Q: What makes a good pull request?

**A:**
- ✅ Clear description of what and why
- ✅ Tests included
- ✅ Follows code style (yarn lint)
- ✅ Small and focused (one feature/fix per PR)
- ✅ No breaking changes (unless discussed)

### Q: How long until my PR is reviewed?

**A:** Maintainers review PRs as time allows. Expect 1-7 days. Complex PRs may take longer. Be patient and responsive to feedback.

---

## Technology Choices

### Q: Why Koa instead of Express?

**A:** Koa uses async/await instead of callbacks, resulting in cleaner code and better error handling.

### Q: Why Sequelize instead of Prisma/TypeORM?

**A:** Sequelize was chosen early in the project (2016). Migration to Prisma would be a large effort. Sequelize works well for Outline's needs.

### Q: Why MobX instead of Redux/Zustand?

**A:** MobX provides automatic reactivity with less boilerplate than Redux. The decorator API (MobX 4) is ergonomic for this codebase.

### Q: Why ProseMirror instead of Slate/Lexical?

**A:** ProseMirror is mature, performant, and powers editors like Notion and Atlassian. It has excellent real-time collaboration support (Y.js integration).

### Q: Why PostgreSQL instead of MySQL/MongoDB?

**A:** PostgreSQL provides:
- Excellent full-text search
- JSONB support (flexible schemas)
- Array columns
- Mature, reliable, open-source

---

## Migration & Upgrades

### Q: Is there a plan to upgrade to React 19?

**A:** Not immediately. React 19 requires significant refactoring:
- Replace all forwardRef usage
- Update to new patterns
- Test thoroughly

This is a large effort that would need to be carefully planned.

### Q: Is there a plan to upgrade to MobX 6?

**A:** Similar to React, MobX 6 would require refactoring all stores to use `makeObservable` instead of decorators. No immediate plans.

### Q: Can I use the latest Node.js LTS?

**A:** You can try Node 24, but the project is tested with Node 22. Report any issues you find.

---

## Learning Resources

### Q: Where can I learn more about the technologies used?

**A:** See [TECH_STACK_GUIDE.md](./TECH_STACK_GUIDE.md) for links to official docs.

**Key resources:**
- React: https://react.dev/
- MobX: https://mobx.js.org/
- Koa: https://koajs.com/
- Sequelize: https://sequelize.org/
- ProseMirror: https://prosemirror.net/
- Yjs: https://docs.yjs.dev/

### Q: Are there any video tutorials?

**A:** Not official ones for this codebase. The best way to learn is:
1. Read this documentation
2. Explore the code
3. Try the [exercises](./EXERCISES.md)
4. Make small contributions

### Q: Who can I ask for help?

**A:**
- GitHub Discussions: https://github.com/outline/outline/discussions
- GitHub Issues (for bugs/features)
- Discord/Slack (if available - check repo)

---

## More Questions?

Check these guides:
- [Getting Started](./GETTING_STARTED.md)
- [Architecture Overview](./ARCHITECTURE_OVERVIEW.md)
- [Tech Stack Guide](./TECH_STACK_GUIDE.md)
- [How-To Guide](./HOW_TO_GUIDE.md)
- [Debugging Guide](./DEBUGGING_GUIDE.md)

Or ask in GitHub Discussions!

---

*Last Updated: November 18, 2025*
