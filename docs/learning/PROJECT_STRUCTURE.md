# Outline Project Structure

**Quick Reference:** Where to find everything in the Outline codebase.

**Documentation Date:** November 18, 2025

---

## Directory Overview

```
outline/
├── app/                    # 🎨 Frontend (React + MobX)
├── server/                 # ⚙️ Backend (Koa + Sequelize)
├── shared/                 # 🔄 Shared code (universal)
├── plugins/                # 🔌 Plugin system
├── public/                 # 📁 Static assets
├── build/                  # 📦 Compiled output (gitignored)
├── data/                   # 💾 Local file storage (gitignored)
├── docs/                   # 📚 Documentation
├── patches/                # 🩹 Package patches
├── __mocks__/              # 🧪 Global test mocks
└── .github/                # 🤖 CI/CD workflows
```

---

## Frontend (`app/`)

**Purpose:** React-based single-page application

```
app/
├── index.tsx                   # Entry point, renders <App />
├── App.tsx                     # Root component with providers
│
├── routes/                     # React Router configuration
│   └── index.tsx              # Main routes (authenticated vs public)
│
├── scenes/                     # Page-level components (route handlers)
│   ├── Document/              # [app/scenes/Document](../../app/scenes/Document/)
│   │   ├── index.tsx          # Document viewer/editor
│   │   └── components/        # Document-specific components
│   ├── Collection/            # Collection management
│   ├── Settings/              # Settings pages
│   ├── Login/                 # Authentication pages
│   ├── Search/                # Search interface
│   ├── Home/                  # Dashboard
│   └── ...                    # 20+ scene directories
│
├── components/                 # Reusable UI components (60+)
│   ├── Avatar/                # [app/components/Avatar](../../app/components/Avatar/)
│   ├── Button/
│   ├── CommandBar/            # Command palette (Cmd+K)
│   ├── Editor/                # Document editor wrapper
│   ├── Flex/                  # Layout primitives
│   ├── Modal/
│   ├── Sidebar/               # Navigation sidebar
│   ├── Tooltip/
│   ├── primitives/            # Low-level UI building blocks
│   └── ...
│
├── stores/                     # MobX state management (30+ stores)
│   ├── RootStore.ts           # [app/stores/RootStore.ts](../../app/stores/RootStore.ts)
│   │                          # Root store, initializes all stores
│   ├── AuthStore.ts           # Authentication state
│   ├── DocumentsStore.ts      # Document collection store
│   ├── CollectionsStore.ts    # Collection tree store
│   ├── UiStore.ts             # UI state (modals, toasts, sidebars)
│   ├── PoliciesStore.ts       # Permission checks
│   └── ...                    # Domain-specific stores
│
├── models/                     # Frontend data models (25+ models)
│   ├── base/
│   │   └── Model.ts           # Base model with save/delete/refresh
│   ├── Document.ts            # Document model (~17KB)
│   ├── Collection.ts          # Collection with tree structure
│   ├── User.ts
│   ├── Team.ts
│   └── ...
│
├── editor/                     # Shared editor components
│   ├── components/            # Editor UI (mentions, embeds, etc.)
│   ├── extensions/            # Custom editor extensions
│   └── menus/                 # Editor menus
│
├── actions/                    # User actions/commands
│   └── definitions/           # Action definitions for command palette
│
├── hooks/                      # React hooks (30+)
│   ├── useCurrentTeam.ts
│   ├── useCurrentUser.ts
│   ├── useStores.ts           # Access MobX stores
│   └── ...
│
├── utils/                      # Utility functions
│   ├── dates.ts
│   ├── files.ts
│   ├── routeHelpers.ts
│   └── ...
│
├── menus/                      # Context menus
├── styles/                     # Global styles
│   ├── global.ts              # Global CSS
│   └── theme.ts               # Theme configuration
│
└── test/                       # Test setup
    └── setup.ts               # Jest configuration
```

**Key Files:**
- [app/index.tsx](../../app/index.tsx) - Entry point
- [app/stores/RootStore.ts](../../app/stores/RootStore.ts#L1-L50) - State initialization
- [app/routes/index.tsx](../../app/routes/index.tsx) - Routing configuration

---

## Backend (`server/`)

**Purpose:** Koa-based REST API + WebSocket server

```
server/
├── index.ts                    # [server/index.ts](../../server/index.ts)
│                              # Entry point, starts Koa with throng
│
├── services/                   # Service modules
│   ├── web.ts                 # [server/services/web.ts](../../server/services/web.ts)
│   │                          # Web service (serves app + API)
│   ├── worker.ts              # Background job worker
│   ├── collaboration.ts       # Real-time collaboration (Hocuspocus/Yjs)
│   ├── websockets.ts          # WebSocket service (Socket.io)
│   └── cron.ts                # Scheduled tasks
│
├── routes/                     # HTTP routes
│   ├── index.ts               # [server/routes/index.ts](../../server/routes/index.ts)
│   │                          # Main web routes (static, SSR)
│   ├── api/                   # REST API endpoints
│   │   ├── index.ts           # API router setup
│   │   ├── documents/         # /api/documents.*
│   │   ├── collections/       # /api/collections.*
│   │   ├── users/             # /api/users.*
│   │   ├── auth/              # /api/auth.*
│   │   ├── attachments/       # /api/attachments.*
│   │   ├── shares/            # /api/shares.*
│   │   └── ...                # 30+ resource routes
│   ├── auth/                  # OAuth authentication routes
│   │   ├── providers/         # OAuth provider implementations
│   │   └── ...
│   └── oauth/                 # OAuth2 provider (Outline as OAuth server)
│
├── models/                     # Database models (Sequelize, 40+)
│   ├── base/
│   │   └── Model.ts           # [server/models/base/Model.ts](../../server/models/base/Model.ts)
│   │                          # Base model with events, soft deletes
│   ├── Document.ts            # [server/models/Document.ts](../../server/models/Document.ts)
│   │                          # Document model (~34KB, complex)
│   ├── Collection.ts          # Collection with tree structure
│   ├── User.ts                # User model (~21KB)
│   ├── Team.ts                # Multi-tenancy
│   ├── ApiKey.ts              # API authentication
│   ├── Attachment.ts          # File attachments
│   ├── Comment.ts             # Document comments
│   └── ...                    # 40+ models
│
├── commands/                   # Business logic (Command pattern)
│   ├── documentCreator.ts     # [server/commands/documentCreator.ts](../../server/commands/documentCreator.ts)
│   ├── documentUpdater.ts     # Updates documents
│   ├── documentMover.ts       # Moves documents
│   ├── accountProvisioner.ts  # User account provisioning
│   └── ...                    # 20+ command classes
│
├── policies/                   # Authorization (CanCan-style)
│   ├── cancan.ts              # [server/policies/cancan.ts](../../server/policies/cancan.ts)
│   │                          # Authorization framework
│   ├── document.ts            # Document permissions
│   ├── collection.ts          # Collection permissions
│   ├── user.ts                # User permissions
│   └── ...                    # Policy per model
│
├── presenters/                 # API response formatters
│   ├── document.ts            # [server/presenters/document.ts](../../server/presenters/document.ts)
│   ├── collection.ts          # Formats collection for API
│   ├── user.ts                # Formats user (removes sensitive data)
│   └── ...                    # Presenter per model
│
├── queues/                     # Background job system (Bull + Redis)
│   ├── processors/            # Job processors (event handlers)
│   │   ├── ImportsProcessor.ts
│   │   ├── EmailsProcessor.ts
│   │   ├── WebsocketsProcessor.ts
│   │   ├── NotificationsProcessor.ts
│   │   └── ...                # 20+ processors
│   └── tasks/                 # Async/scheduled tasks
│       ├── DocumentImportTask.ts
│       ├── CleanupDeletedDocumentsTask.ts
│       ├── ExportTask.ts
│       └── ...                # 50+ task classes
│
├── middlewares/                # Koa middlewares
│   ├── authentication.ts      # [server/middlewares/authentication.ts](../../server/middlewares/authentication.ts)
│   │                          # JWT/session auth
│   ├── rateLimiter.ts         # Rate limiting
│   ├── csrf.ts                # CSRF protection
│   ├── editor.ts              # Editor permissions
│   ├── passport.ts            # Passport.js integration
│   └── ...                    # 15+ middlewares
│
├── storage/                    # Storage abstraction
│   ├── database.ts            # Sequelize setup
│   ├── redis.ts               # Redis client
│   └── files/                 # File storage
│       ├── S3Storage.ts       # S3-compatible storage
│       └── LocalStorage.ts    # Local filesystem
│
├── migrations/                 # Database migrations (200+)
│   ├── 20160101000000-initial.js
│   ├── 20160102000000-add-teams.js
│   └── ...                    # Chronological order
│
├── emails/                     # Email templates
│   └── templates/             # React email templates
│       ├── WelcomeEmail.tsx
│       ├── InviteEmail.tsx
│       └── ...
│
├── collaboration/              # Real-time collaboration
│   ├── HocuspocusProvider.ts
│   └── extensions/
│
├── utils/                      # Utility functions
│   ├── PluginManager.ts       # [server/utils/PluginManager.ts](../../server/utils/PluginManager.ts)
│   │                          # Plugin system loader
│   ├── Logger.ts              # Logging
│   ├── metrics.ts             # Metrics collection
│   └── ...
│
├── config/                     # Configuration
│   └── bullboard.ts           # Bull queue dashboard
│
├── logging/                    # Logging infrastructure
├── onboarding/                 # Onboarding flows
├── env.ts                      # [server/env.ts](../../server/env.ts)
│                              # Environment variable definitions
└── test/                       # Test fixtures
    ├── factories/             # Test data factories
    ├── fixtures/              # Test fixtures
    └── setup.ts               # Jest setup
```

**Key Files:**
- [server/index.ts](../../server/index.ts#L1-L50) - Entry point
- [server/models/index.ts](../../server/models/index.ts) - Model registry
- [server/routes/api/index.ts](../../server/routes/api/index.ts) - API routes

---

## Shared (`shared/`)

**Purpose:** Code shared between frontend and backend

```
shared/
├── types.ts                    # [shared/types.ts](../../shared/types.ts)
│                              # Shared TypeScript types/enums
├── constants.ts                # Shared constants
├── validations.ts              # Shared validation logic
├── schema.ts                   # Zod schemas
│
├── editor/                     # ProseMirror editor (used by both)
│   ├── nodes/                 # Document nodes
│   │   ├── Heading.ts         # Heading node
│   │   ├── Paragraph.ts       # Paragraph node
│   │   ├── CodeBlock.ts       # Code block node
│   │   └── ...                # 30+ node types
│   ├── marks/                 # Text marks
│   │   ├── Bold.ts
│   │   ├── Italic.ts
│   │   ├── Link.ts
│   │   └── ...
│   ├── extensions/            # Editor extensions
│   ├── commands/              # Editor commands
│   ├── plugins/               # ProseMirror plugins
│   ├── lib/                   # Editor utilities
│   └── components/            # Editor React components
│
├── utils/                      # Shared utilities
│   ├── parseTitle.ts          # Extract title from markdown
│   ├── slugify.ts             # URL-safe slugs
│   └── ...
│
├── helpers/                    # Helper functions
├── hooks/                      # Shared React hooks
├── components/                 # Shared React components
├── collaboration/              # Collaboration utilities
│
└── i18n/                       # Internationalization
    ├── index.ts               # i18n configuration
    └── locales/               # 27 language translations
        ├── en_US/
        ├── fr_FR/
        ├── de_DE/
        └── ...
```

---

## Plugins (`plugins/`)

**Purpose:** Modular plugin system for extensibility

```
plugins/
├── slack/                      # Slack integration plugin
│   ├── plugin.json            # Plugin metadata
│   ├── client/                # Frontend components
│   │   ├── SlackButton.tsx
│   │   └── ...
│   ├── server/                # Backend logic
│   │   ├── index.ts           # Plugin registration
│   │   ├── auth/              # OAuth authentication
│   │   ├── api/               # API endpoints
│   │   ├── processors/        # Event processors
│   │   └── presenters/        # Response formatters
│   └── shared/                # Shared types/utils
│
├── google/                     # Google OAuth
├── azure/                      # Azure AD OAuth
├── oidc/                       # Generic OIDC provider
├── email/                      # Email authentication
├── storage/                    # S3-compatible storage
├── webhooks/                   # Webhook integration
├── notion/                     # Notion import
├── linear/                     # Linear integration
├── github/                     # GitHub integration
├── googleanalytics/            # Google Analytics
├── matomo/                     # Matomo analytics
├── umami/                      # Umami analytics
└── ...                         # More plugins

Each plugin follows the structure:
plugin-name/
├── plugin.json                 # Metadata
├── client/                     # Frontend code (optional)
├── server/                     # Backend code
│   ├── index.ts               # Registers hooks
│   ├── auth/                  # Auth provider (if applicable)
│   ├── api/                   # API routes (if applicable)
│   └── ...
└── shared/                     # Shared code (optional)
```

**Plugin Hooks:**
- `Hook.AuthProvider` - OAuth providers
- `Hook.API` - API routes
- `Hook.Processor` - Event processors
- `Hook.Task` - Background tasks
- `Hook.EmailTemplate` - Email templates
- `Hook.IssueProvider` - Issue tracking

**See:** [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md) for plugin development

---

## Build Output (`build/`)

**Generated by:** `yarn build`

```
build/
├── app/                        # Frontend bundle (Vite output)
│   ├── index.html             # Entry HTML
│   ├── assets/
│   │   ├── index-[hash].js    # Main bundle
│   │   ├── index-[hash].css   # Styles
│   │   └── ...                # Chunks, images, fonts
│   └── manifest.json          # PWA manifest
│
├── server/                     # Backend compiled (Babel output)
│   ├── index.js               # Entry point
│   ├── routes/
│   ├── models/
│   └── ...                    # Mirrored structure from server/
│
└── shared/                     # Shared code compiled
    ├── editor/
    └── i18n/
        └── locales/           # Copied from shared/i18n/locales/
```

---

## Configuration Files

| File | Purpose |
|------|---------|
| `.env` | Environment variables (gitignored) |
| `.env.sample` | Template for `.env` |
| `package.json` | Dependencies, scripts, monorepo config |
| `yarn.lock` | Locked dependency versions |
| `tsconfig.json` | TypeScript configuration |
| `.jestconfig.json` | Jest test configuration |
| `vite.config.ts` | Frontend build configuration |
| `build.js` | Backend build script |
| `docker-compose.yml` | Docker services (Redis, PostgreSQL) |
| `Dockerfile` | Production Docker image |
| `.sequelizerc` | Sequelize CLI configuration |
| `.oxlintrc.json` | Oxlint configuration |
| `.prettierrc` | Prettier configuration |
| `.gitignore` | Git ignored files |

---

## Finding Things Quickly

### "I want to find..."

**Frontend Components:**
```bash
# Search in app/components/
ls app/components/

# Find component by name
find app/components -name "*Button*"
```

**API Endpoints:**
```bash
# All API routes
ls server/routes/api/

# Find specific endpoint
grep -r "documents.create" server/routes/api/
```

**Database Models:**
```bash
# All models
ls server/models/

# Find model
ls server/models/ | grep Document
```

**Tests:**
```bash
# All test files
find . -name "*.test.ts" | grep -v node_modules

# Tests for specific file
find . -name "Document.test.ts"
```

**Editor Nodes:**
```bash
# ProseMirror nodes
ls shared/editor/nodes/
```

**Translations:**
```bash
# All locales
ls shared/i18n/locales/

# English translations
cat shared/i18n/locales/en_US/translation.json
```

---

## File Naming Conventions

### Frontend (app/)

**Components:**
- PascalCase: `Avatar.tsx`, `CommandBar.tsx`
- Index files for directory components: `Sidebar/index.tsx`
- Tests colocated: `Avatar.test.tsx`
- Styles (if separate): `Avatar.styles.ts`

**Models:**
- PascalCase singular: `Document.ts`, `Collection.ts`

**Stores:**
- PascalCase plural + "Store": `DocumentsStore.ts`, `CollectionsStore.ts`

**Hooks:**
- camelCase with "use" prefix: `useCurrentUser.ts`, `useStores.ts`

**Utils:**
- camelCase: `routeHelpers.ts`, `dateUtils.ts`

### Backend (server/)

**Models:**
- PascalCase singular: `Document.ts`, `User.ts`
- Sequelize decorators: `@Table`, `@Column`

**Routes:**
- camelCase plurals: `documents.ts`, `collections.ts`
- Directory per resource: `api/documents/documents.ts`

**Commands:**
- camelCase + action: `documentCreator.ts`, `documentUpdater.ts`

**Migrations:**
- Timestamp + kebab-case: `20230101000000-add-comments.js`

**Tasks:**
- PascalCase + "Task": `DocumentImportTask.ts`

**Processors:**
- PascalCase + "Processor": `EmailsProcessor.ts`

---

## Import Aliases

TypeScript path aliases (defined in `tsconfig.json`):

```typescript
// Frontend
import { Document } from "~/models/Document"      // app/models/Document
import type { User } from "~/types"                // app/types

// Backend
import { Document } from "@server/models/Document" // server/models/Document
import { documentCreator } from "@server/commands" // server/commands

// Shared
import { Editor } from "@shared/editor"            // shared/editor
import type { DocumentData } from "@shared/types"  // shared/types
```

---

## Next Steps

- **Understand the Architecture:** [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)
- **Learn the Technologies:** [TECH_STACK_GUIDE.md](./TECH_STACK_GUIDE.md)
- **Trace a Request:** [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md)
- **Frontend Deep Dive:** [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md)
- **Backend Deep Dive:** [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)

---

*Last Updated: November 18, 2025*
