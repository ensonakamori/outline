# Outline Architecture Overview

**High-level system design and architecture patterns.**

**Documentation Date:** November 18, 2025

---

## System Architecture

Outline is a full-stack monorepo with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                        Client (Browser)                      │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ React App (app/)                                        │ │
│  │ - MobX State Management                                 │ │
│  │ - ProseMirror Editor                                    │ │
│  │ - Styled Components                                     │ │
│  └────────────────────────────────────────────────────────┘ │
└───────────────┬─────────────────────────────────────────────┘
                │ HTTP/WebSocket
┌───────────────┴─────────────────────────────────────────────┐
│                  Server (Node.js)                            │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐│
│  │  Web Service    │  │  Collaboration  │  │  Websockets  ││
│  │  (Koa)          │  │  (Hocuspocus)   │  │  (Socket.io) ││
│  └────────┬────────┘  └────────┬────────┘  └──────┬───────┘│
│           │                    │                    │        │
│  ┌────────┴────────────────────┴────────────────────┴─────┐ │
│  │          Shared Business Logic                          │ │
│  │  Commands | Policies | Presenters | Models              │ │
│  └─────────────────────────┬──────────────────────────────┘ │
└────────────────────────────┼─────────────────────────────────┘
                             │
         ┌───────────────────┴──────────────────┐
         │                                       │
    ┌────┴─────┐  ┌──────────┐  ┌────────────┐ │
    │PostgreSQL│  │  Redis   │  │ S3 Storage │ │
    │ Database │  │  Cache   │  │   Files    │ │
    └──────────┘  └──────────┘  └────────────┘ │
                                                │
                  ┌─────────────────────────────┘
                  │
            ┌─────┴──────┐
            │   Worker   │
            │ Background │
            │    Jobs    │
            └────────────┘
```

---

## Frontend Architecture (React + MobX)

### Component Hierarchy

```
App (Root)
├── Theme Provider
├── I18n Provider
├── Router
│   ├── Authenticated Routes
│   │   ├── Document Scene
│   │   ├── Collection Scene
│   │   ├── Settings Scene
│   │   └── ...
│   └── Public Routes
│       ├── Login Scene
│       └── Share Scene
└── Global Components
    ├── Sidebar
    ├── CommandBar
    └── Toasts
```

### State Management (MobX 4 - Decorator Pattern)

**Store Structure:**
```
RootStore
├── AuthStore          (current user, authentication)
├── DocumentsStore     (document collection)
├── CollectionsStore   (collection tree)
├── UsersStore         (team users)
├── UiStore            (modals, sidebar, toasts)
└── PoliciesStore      (permissions)
```

**Pattern:**
```typescript
class DocumentsStore {
  @observable documents = new Map()
  
  @action
  add(document: Document) {
    this.documents.set(document.id, document)
  }
  
  @computed
  get orderedDocuments() {
    return Array.from(this.documents.values())
      .sort((a, b) => a.title.localeCompare(b.title))
  }
}
```

⚠️ **Note:** This uses MobX 4 decorators. Modern MobX 6 uses `makeObservable`.

---

## Backend Architecture (Koa + Sequelize)

### Layered Architecture

```
HTTP Request
    ↓
┌───────────────────────────────┐
│   Middleware Stack            │
│ - Authentication              │
│ - Rate Limiting               │
│ - CSRF Protection             │
└───────────────┬───────────────┘
                ↓
┌───────────────────────────────┐
│   Route Handler               │
│ - Parse request               │
│ - Validate input              │
└───────────────┬───────────────┘
                ↓
┌───────────────────────────────┐
│   Policy Check                │
│ - authorize(user, action, resource)
└───────────────┬───────────────┘
                ↓
┌───────────────────────────────┐
│   Command Execution           │
│ - Business logic              │
│ - Database operations         │
│ - Emit events                 │
└───────────────┬───────────────┘
                ↓
┌───────────────────────────────┐
│   Presenter                   │
│ - Format response             │
│ - Remove sensitive data       │
└───────────────┬───────────────┘
                ↓
         HTTP Response
```

### Command Pattern

Commands encapsulate business logic:

**Example:** `documentCreator.ts`
```typescript
export default async function documentCreator({
  title, text, user, collection, ...
}: DocumentCreatorProps) {
  // Validate
  // Create document
  // Create backlinks
  // Create revision
  // Emit event
  return document
}
```

Used by:
- API endpoint (`POST /api/documents`)
- Background jobs (imports)
- Tests

### Event-Driven Architecture

```
Model Change (e.g., document.save())
    ↓
Event Emitted (e.g., "documents.update")
    ↓
Queued in Redis (Bull)
    ↓
Processor Handles Event
    ↓
Tasks Executed (email, webhooks, search indexing)
```

---

## Multi-Service Architecture

Outline runs multiple services:

**1. Web Service** (`services/web.ts`)
- Serves static assets
- Handles API requests
- Server-side rendering

**2. Collaboration Service** (`services/collaboration.ts`)
- Hocuspocus server (Yjs WebSocket)
- Real-time document editing
- CRDT conflict resolution

**3. WebSocket Service** (`services/websockets.ts`)
- Socket.io server
- Real-time notifications
- Presence awareness

**4. Worker Service** (`services/worker.ts`)
- Background job processing
- Email sending
- Document imports
- Search indexing

**5. Cron Service** (`services/cron.ts`)
- Scheduled tasks
- Data cleanup
- Metrics collection

All services share code (models, commands, policies) but run in separate processes.

---

## Database Architecture

### Multi-Tenancy

Every record belongs to a **Team**:
```sql
WHERE teamId = <current-user-team-id>
```

Sequelize automatically adds this filter to all queries.

### Key Models

- **Team** - Tenant (organization)
- **User** - User account (belongs to team)
- **Collection** - Document hierarchy (tree structure)
- **Document** - Content (with collaborative editing)
- **Revision** - Document version history
- **Share** - Public sharing links
- **ApiKey** - API authentication

### Relationships

```
Team
├── Users (1:many)
├── Collections (1:many)
│   └── Documents (1:many)
│       ├── Revisions (1:many)
│       ├── Backlinks (many:many)
│       └── Comments (1:many)
└── ...
```

---

## Real-Time Collaboration

Uses **Yjs** (CRDT) for conflict-free collaborative editing:

```
User A types "Hello"           User B types "World"
    ↓                               ↓
Y.js Document (Local)         Y.js Document (Local)
    ↓                               ↓
WebSocket → Hocuspocus Server ← WebSocket
                ↓
        Merge Operations (CRDT)
                ↓
        Broadcast to All Clients
                ↓
    ┌───────────┴───────────┐
    ↓                       ↓
User A sees "HelloWorld"  User B sees "HelloWorld"
```

**Features:**
- Automatic conflict resolution
- Offline support (sync when reconnected)
- Cursor awareness
- Persisted to PostgreSQL

---

## Plugin System

Plugins extend functionality via hooks:

```typescript
// Plugin registration
export default {
  name: "slack",
  init() {
    PluginManager.registerHook(Hook.AuthProvider, SlackAuth)
    PluginManager.registerHook(Hook.API, SlackAPI)
    PluginManager.registerHook(Hook.Processor, SlackProcessor)
  }
}
```

**Available Hooks:**
- `Hook.AuthProvider` - OAuth providers
- `Hook.API` - API routes
- `Hook.Processor` - Event processors
- `Hook.Task` - Background tasks
- `Hook.EmailTemplate` - Email templates

See: [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md)

---

## Security

### Authentication
- OAuth 2.0 (Google, Slack, Azure, OIDC)
- Magic links (email)
- API keys
- JWT tokens (session)

### Authorization
- Policy-based (CanCan pattern)
- Row-level security (team isolation)
- Role-based (admin, member, viewer)

### Protection
- CSRF tokens
- Rate limiting
- Input validation (Zod schemas)
- SQL injection prevention (Sequelize ORM)
- XSS prevention (sanitized output)

---

## Data Flow Examples

See [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md) for detailed flows.

---

## Technology Stack Summary

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Frontend | React 17 | UI components |
| State | MobX 4 | State management |
| Styling | Styled Components 5 | CSS-in-JS |
| Editor | ProseMirror | Rich text editing |
| Backend | Koa 3 | HTTP server |
| ORM | Sequelize 6 | Database abstraction |
| Database | PostgreSQL | Data storage |
| Cache/Queue | Redis | Caching + jobs |
| Jobs | Bull | Background processing |
| Realtime | Socket.io + Yjs | WebSockets + CRDT |
| Build | Vite + Babel | Bundling |
| Tests | Jest | Testing |

**Full analysis:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)

---

## Next Steps

- [Project Structure](./PROJECT_STRUCTURE.md) - Find files
- [Data Flow Guide](./DATA_FLOW_GUIDE.md) - Trace requests
- [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) - React deep dive
- [Backend Architecture](./BACKEND_ARCHITECTURE.md) - Koa deep dive
- [Database Architecture](./DATABASE_ARCHITECTURE.md) - Data models

---

*Last Updated: November 18, 2025*
