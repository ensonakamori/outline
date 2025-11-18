# Outline Learning Path

**Welcome to Outline!** This is your complete guide to understanding and contributing to the Outline codebase.

**Documentation Date:** November 18, 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)

---

## What is Outline?

Outline is an open-source, self-hosted collaborative knowledge base and wiki designed for modern teams. It combines the best of Notion, Confluence, and Google Docs with:

- **Real-time Collaboration:** Multiple people editing simultaneously (powered by Yjs CRDTs)
- **Beautiful Editor:** Rich text editing with Markdown support (ProseMirror)
- **Powerful Search:** Full-text search with PostgreSQL
- **Extensible:** Plugin system for auth providers, integrations, and storage
- **Self-Hosted:** Run on your own infrastructure

---

## About This Documentation

This documentation is written for **mid-level developers** who:
- ✅ Are proficient in JavaScript/TypeScript and React
- ⚠️ May be new to some technologies used in THIS specific project
- 🎯 Want to contribute within their first 2-3 weeks
- 📚 Learn best through code examples and connections to familiar concepts

### How to Use This Documentation

1. **Start Here:** Begin with the [Getting Started Guide](./GETTING_STARTED.md)
2. **Understand the System:** Read the [Architecture Overview](./ARCHITECTURE_OVERVIEW.md)
3. **Dive Deeper:** Choose guides based on what you're working on
4. **Practice:** Try the [Hands-On Exercises](./EXERCISES.md)
5. **Contribute:** Follow the [First Contributions Guide](./FIRST_CONTRIBUTIONS.md)

---

## Important: Technology Stack Notes (Nov 2025)

⚠️ **This project intentionally uses some older versions** of popular libraries for stability. The patterns you see here may differ from the latest tutorials online.

| Technology | Project Version | Latest (Nov 2025) | Status |
|------------|----------------|-------------------|---------|
| React | 17.0.2 | 19.2.0 | 🚨 Using v17 patterns |
| MobX | 4.15.4 | 6.15.0 | 🚨 Using v4 decorators |
| Node.js | 22.x | 24.x LTS | ⚠️ One version behind |
| TypeScript | 5.9.2 | 5.9.3 | ✅ Current |
| Koa | 3.0.3 | 3.1.0 | ✅ Current |
| Sequelize | 6.37.7 | 6.37.7 | ✅ Current |

**What this means for you:**
- Learn the patterns in THIS project first (they're stable and production-tested)
- Be aware that React 17 and MobX 4 patterns differ from current best practices
- See [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md) for comprehensive version analysis

---

## Learning Path

### 🚀 Phase 1: Getting Started (Day 1)

**Goal:** Get Outline running locally and understand the basics

1. [**Getting Started Guide**](./GETTING_STARTED.md)
   - Prerequisites and installation
   - First run and setup
   - Development environment
   - Troubleshooting common issues

2. [**Project Structure**](./PROJECT_STRUCTURE.md)
   - Directory organization (app/, server/, shared/, plugins/)
   - File naming conventions
   - Entry points and build output

3. [**Technology Stack Guide**](./TECH_STACK_GUIDE.md)
   - Overview of all technologies
   - Why each was chosen
   - How they work together
   - Analogies to familiar tools

---

### 🏗️ Phase 2: Understanding the System (Days 2-3)

**Goal:** Grasp the overall architecture and data flow

4. [**Architecture Overview**](./ARCHITECTURE_OVERVIEW.md)
   - High-level system design
   - Frontend architecture (React + MobX)
   - Backend architecture (Koa + Sequelize)
   - Real-time collaboration (Yjs)
   - Multi-service design
   - Visual diagrams

5. [**Data Flow Guide**](./DATA_FLOW_GUIDE.md)
   - Request lifecycle
   - API request flow
   - Real-time collaboration flow
   - Background job flow
   - Authentication flow
   - Sequence diagrams

---

### 🎨 Phase 3: Frontend Deep Dive (Days 4-5)

**Goal:** Master the React + MobX frontend

6. [**Frontend Architecture**](./FRONTEND_ARCHITECTURE.md)
   - React components structure
   - MobX 4 state management (decorator patterns)
   - Stores and models
   - React Router v5
   - ProseMirror editor
   - Styled Components v5
   - Code examples

---

### ⚙️ Phase 4: Backend Deep Dive (Days 6-8)

**Goal:** Understand the Koa backend and database

7. [**Backend Architecture**](./BACKEND_ARCHITECTURE.md)
   - Koa server setup
   - Middleware stack
   - Command pattern
   - Policy-based authorization
   - Presenters
   - Event-driven architecture
   - Multi-service design

8. [**Database Architecture**](./DATABASE_ARCHITECTURE.md)
   - Sequelize ORM patterns
   - Model structure and relationships
   - Migration strategy
   - Full-text search
   - Multi-tenancy
   - Query optimization

9. [**Database Schema**](./DATABASE_SCHEMA.md)
   - Complete schema documentation
   - Table relationships
   - Key queries

---

### 🔌 Phase 5: Integrations & Plugins (Days 9-10)

**Goal:** Learn the plugin system and integrations

10. [**Integration Guide**](./INTEGRATION_GUIDE.md)
    - Plugin system architecture
    - Creating custom plugins
    - OAuth providers
    - Webhook integrations
    - Third-party integrations

---

### 📝 Phase 6: Practical Skills (Days 11-14)

**Goal:** Learn how to build features and write code

11. [**Patterns and Conventions**](./PATTERNS_AND_CONVENTIONS.md)
    - Code style and naming
    - MobX patterns (decorator-based)
    - React patterns
    - Async/await patterns
    - Error handling
    - Testing patterns

12. [**How-To Guide**](./HOW_TO_GUIDE.md)
    - How to add a new API endpoint
    - How to create a new model
    - How to add a new React component
    - How to add a migration
    - How to create a background job
    - How to add authentication provider

13. [**Development Workflow**](./DEVELOPMENT_WORKFLOW.md)
    - Git workflow and branches
    - Commit conventions
    - PR process and code review
    - Local development tips
    - Debugging techniques

14. [**Code Tours**](./CODE_TOURS.md)
    - Tour 1: Follow a document creation
    - Tour 2: Real-time collaboration flow
    - Tour 3: Authentication flow
    - Tour 4: Search functionality
    - Tour 5: Background job processing

---

### 🧪 Phase 7: Testing & Quality (Days 15-16)

**Goal:** Write tests and ensure code quality

15. [**Testing Guide**](./TESTING_GUIDE.md)
    - Jest configuration
    - Unit testing patterns
    - Integration testing
    - Component testing
    - API testing
    - Mocking strategies

16. [**Debugging Guide**](./DEBUGGING_GUIDE.md)
    - Frontend debugging (React DevTools, MobX DevTools)
    - Backend debugging (Node inspector)
    - Database query debugging
    - WebSocket debugging
    - Performance debugging

---

### 🔒 Phase 8: Security & APIs (Days 17-18)

**Goal:** Understand security and API design

17. [**Security Guide**](./SECURITY_GUIDE.md)
    - Authentication architecture
    - Authorization (policies)
    - CSRF protection
    - Rate limiting
    - Input validation
    - Security best practices

18. [**API Documentation**](./API_DOCUMENTATION.md)
    - REST API endpoints
    - Authentication
    - Request/response formats
    - Error handling
    - Rate limits
    - Examples

---

### 🎓 Phase 9: Practice & Contribute (Days 19-21)

**Goal:** Build something and make your first contribution

19. [**Hands-On Exercises**](./EXERCISES.md)
    - Exercise 1: Create a simple model and API
    - Exercise 2: Add a frontend component
    - Exercise 3: Implement a background job
    - Exercise 4: Add a plugin
    - Exercise 5: Write comprehensive tests

20. [**First Contributions Guide**](./FIRST_CONTRIBUTIONS.md)
    - Good first issues
    - Beginner-friendly areas
    - Code review checklist
    - Documentation updates

---

### 📚 Reference Materials

21. [**FAQ**](./FAQ.md)
    - Common questions
    - Troubleshooting
    - Architecture decisions
    - Technology choices

22. [**Technology Stack Research**](./TECH_STACK_RESEARCH.md)
    - Comprehensive version analysis
    - Current vs. outdated patterns
    - Migration considerations
    - Official documentation links

---

## Quick Reference

### Common Commands

```bash
# Development
yarn dev:watch              # Frontend + backend hot reload
yarn dev:backend            # Backend only
yarn vite:dev              # Frontend only

# Testing
yarn test                  # Run all tests
yarn test:app             # Frontend tests
yarn test:server          # Backend tests

# Database
yarn db:migrate           # Run migrations
yarn db:rollback          # Rollback last migration

# Build
yarn build                # Production build
yarn start                # Start production server

# Linting
yarn lint                 # Lint all code
yarn format              # Format with Prettier
```

### Key Directories

```
outline/
├── app/                    # Frontend React application
│   ├── components/        # Reusable UI components
│   ├── scenes/            # Page-level components
│   ├── stores/            # MobX state management
│   └── models/            # Frontend data models
├── server/                 # Backend Koa application
│   ├── routes/            # API and web routes
│   ├── models/            # Database models
│   ├── commands/          # Business logic
│   ├── policies/          # Authorization
│   └── queues/            # Background jobs
├── shared/                 # Shared code
│   └── editor/            # ProseMirror editor
└── plugins/                # Plugin system
    ├── slack/             # Slack integration
    ├── google/            # Google OAuth
    └── ...                # More plugins
```

### Important Files

| File | Purpose |
|------|---------|
| `app/index.tsx` | Frontend entry point |
| `server/index.ts` | Backend entry point |
| `app/stores/RootStore.ts` | Frontend state |
| `server/models/index.ts` | Model registry |
| `vite.config.ts` | Frontend build |
| `build.js` | Backend build |
| `.jestconfig.json` | Test configuration |

---

## Learning Tips

### For React Developers

If you're coming from React but new to MobX:
- MobX is like Redux but with less boilerplate
- Stores are like Redux stores but with automatic subscriptions
- `@observable` is like `useState` but for classes
- `@action` is like a Redux action creator
- `@computed` is like `useMemo` but automatic

**Key Difference:** This project uses MobX 4 with decorators, while modern MobX 6 uses `makeObservable`. See [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) for details.

### For Backend Developers

If you're new to Koa:
- Koa is like Express but with async/await instead of callbacks
- Middleware uses `ctx` (context) instead of `req` and `res`
- Error handling uses try/catch instead of error callbacks
- Routing is similar to Express Router

### For Database Developers

If you're new to Sequelize:
- Sequelize is like an ORM (similar to TypeORM, Prisma, etc.)
- Models are defined with TypeScript decorators
- Migrations are separate files (like Knex, ActiveRecord)
- Queries use a fluent API (like query builders)

---

## Symbols Used in Documentation

Throughout the documentation, you'll see these symbols:

- ✅ **CURRENT (Nov 2025)** - Pattern/approach is up-to-date
- ⚠️ **OUTDATED PATTERN** - Works but newer approaches exist
- 🚨 **DEPRECATED** - Should not be used in new code
- 🆕 **NEW IN 2025** - Feature/pattern introduced recently
- 🔍 **NEEDS INVESTIGATION** - Requires deeper exploration
- ⚠️ **UNCLEAR** - Not fully understood from code
- ❓ **ASSUMPTION** - Based on inference, needs verification
- 🚧 **TODO** - Placeholder needing information

- 🧠 **Mental Model** - Simplified way to think about it
- 🌉 **Bridge from React/JS** - Analogies to familiar concepts
- 💡 **Aha Moment** - Key insight that makes it click
- 🎯 **Remember This** - Mnemonic or memorable phrase
- ⚠️ **Common Pitfall** - What to avoid
- 🔗 **Code Example** - Link to actual code
- ✅ **Quick Check** - Self-test question

---

## Getting Help

### Recommended Learning Order

**Week 1:**
1. Getting Started → Project Structure → Tech Stack Guide
2. Architecture Overview → Data Flow Guide
3. Frontend Architecture OR Backend Architecture (based on your role)

**Week 2:**
1. Database Architecture → Integration Guide
2. Patterns and Conventions → How-To Guide
3. Code Tours (pick 2-3 relevant tours)

**Week 3:**
1. Testing Guide → Debugging Guide
2. Security Guide OR API Documentation
3. Exercises → First Contributions

### When to Use Which Guide

**I want to...**
- **Get started fast** → [Getting Started](./GETTING_STARTED.md)
- **Understand the big picture** → [Architecture Overview](./ARCHITECTURE_OVERVIEW.md)
- **Find a specific file** → [Project Structure](./PROJECT_STRUCTURE.md)
- **Learn about a technology** → [Tech Stack Guide](./TECH_STACK_GUIDE.md)
- **Trace a request** → [Data Flow Guide](./DATA_FLOW_GUIDE.md)
- **Work on frontend** → [Frontend Architecture](./FRONTEND_ARCHITECTURE.md)
- **Work on backend** → [Backend Architecture](./BACKEND_ARCHITECTURE.md)
- **Work with database** → [Database Architecture](./DATABASE_ARCHITECTURE.md)
- **Add an integration** → [Integration Guide](./INTEGRATION_GUIDE.md)
- **Follow code style** → [Patterns and Conventions](./PATTERNS_AND_CONVENTIONS.md)
- **Add a feature** → [How-To Guide](./HOW_TO_GUIDE.md)
- **Understand a flow** → [Code Tours](./CODE_TOURS.md)
- **Set up my workflow** → [Development Workflow](./DEVELOPMENT_WORKFLOW.md)
- **Write tests** → [Testing Guide](./TESTING_GUIDE.md)
- **Fix a bug** → [Debugging Guide](./DEBUGGING_GUIDE.md)
- **Understand security** → [Security Guide](./SECURITY_GUIDE.md)
- **Use the API** → [API Documentation](./API_DOCUMENTATION.md)
- **Understand the schema** → [Database Schema](./DATABASE_SCHEMA.md)
- **Practice** → [Exercises](./EXERCISES.md)
- **Make first contribution** → [First Contributions](./FIRST_CONTRIBUTIONS.md)
- **Answer a question** → [FAQ](./FAQ.md)

---

## Contributing to Documentation

Found an error? Have a suggestion? Want to add more examples?

1. These docs live in `docs/learning/`
2. Follow the same style and use the symbols above
3. Add code links wherever possible
4. Include examples from the actual codebase
5. Mark any uncertainties with appropriate symbols

---

## Technology Version Notes

As of November 2025, this project uses:

**Up-to-Date:**
- TypeScript 5.9.2 ✅
- Koa 3.0.3 ✅
- Sequelize 6.37.7 ✅
- Socket.io 4.8.1 ✅
- Yjs 13.6.1 ✅
- Jest 29.7.0 ✅

**Intentionally Older (Stable):**
- React 17.0.2 ⚠️ (latest is 19.2.0)
- MobX 4.15.4 ⚠️ (latest is 6.15.0)
- Styled Components 5.3.11 ⚠️ (latest is 6.1.15)
- Node.js 22.x ⚠️ (latest LTS is 24.x)

**Why older versions?**
- Stability and maturity over bleeding edge
- Large migration effort to update React and MobX
- Current versions are well-tested in production
- No security or support concerns

See [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md) for comprehensive analysis.

---

## Next Steps

**If you're brand new:**
→ Start with [Getting Started Guide](./GETTING_STARTED.md)

**If you want the big picture:**
→ Read [Architecture Overview](./ARCHITECTURE_OVERVIEW.md)

**If you want to code:**
→ Try [Hands-On Exercises](./EXERCISES.md)

**If you're ready to contribute:**
→ Follow [First Contributions Guide](./FIRST_CONTRIBUTIONS.md)

---

**Happy Learning!** 🚀

*Last Updated: November 18, 2025*
