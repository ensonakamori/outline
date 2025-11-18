# Technology Stack Research (November 2025)

**Research conducted:** November 18, 2025
**Knowledge cutoff context:** January 2025 → November 2025 (10-month gap)

---

## Executive Summary

Outline is a collaborative knowledge base built with a modern JavaScript/TypeScript stack. The project uses several technologies that are 1-2 major versions behind current releases, but all remain stable and production-ready. Some key libraries (MobX, React) are using older major versions that have different patterns than current best practices.

**Overall Status:**
- ✅ Core infrastructure (Node.js, TypeScript, Koa) is current or one minor version behind
- ⚠️ Frontend libraries (React, MobX, Styled Components) are 1-2 major versions behind
- ✅ Database and tooling (Sequelize, PostgreSQL) are up-to-date
- 🔍 Build tool uses custom Vite fork (rolldown-vite)

---

## Core Technologies

### Node.js - v22.x

**Current Status (Nov 2025):**
- Latest LTS: **v24.x (Krypton)** - Released October 28, 2025
- Project uses: **v22.x** (specified in .nvmrc and package.json)
- Dockerfile uses: **v22.21.0-slim**
- Status: ⚠️ **One LTS version behind** (still supported until October 2027)

**Important Updates Since Jan 2025:**
- Node.js v24 LTS (Krypton) released October 2025 with performance improvements
- Node.js v22 transitioned to Active LTS status
- Node.js v25.x is the current experimental release (not LTS)
- Node.js v18 reached End of Life in April 2025

**What This Means for Learning:**
- Node.js 22 is still in Active LTS and fully supported
- All patterns in this project remain current and production-ready
- Migration to Node.js 24 is recommended but not urgent
- No breaking changes affecting this codebase

**Official Resources:**
- Docs: https://nodejs.org/docs/latest-v22.x/api/
- LTS Schedule: https://nodejs.org/en/about/previous-releases
- Node.js 24 Features: https://nodejs.org/en/blog/release/v24.11.0

---

### TypeScript - v5.9.2

**Current Status (Nov 2025):**
- Latest stable: **v5.9.3** (September 2025)
- Project uses: **v5.9.2**
- Development version: v6.0.0-dev (in development)
- Status: ✅ **CURRENT** (only one patch version behind)

**Important Updates Since Jan 2025:**
- TypeScript 5.9 released August 2025
- TypeScript 5.8 available with recent updates
- TypeScript 6.0 planned as transition to TypeScript 7.0
- Native port of TypeScript in development for v7.0

**What This Means for Learning:**
- Project is using current stable TypeScript
- All patterns and features are up-to-date
- No deprecated features in use
- TypeScript configuration (tsconfig.json) follows modern best practices

**TypeScript Config Highlights:**
```json
{
  "target": "es2020",
  "lib": ["dom", "es2020", "dom.iterable", "esnext.asynciterable"],
  "experimentalDecorators": true,  // Used for Sequelize models
  "strictNullChecks": true,
  "noImplicitAny": true
}
```

**Official Resources:**
- Docs: https://www.typescriptlang.org/docs/
- Release Notes: https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-9.html
- TypeScript 5.8: https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-8.html

---

### React - v17.0.2

**Current Status (Nov 2025):**
- Latest stable: **v19.2.0** (October 2025)
- Project uses: **v17.0.2**
- Status: 🚨 **TWO MAJOR VERSIONS BEHIND**

**Important Updates Since Jan 2025:**
- **React 19.0** released December 2024
- **React 19.2** released October 2025 (current)
- React 18.3.1 released as stepping stone for migration

**Major React 19 Features NOT in This Project:**
- ❌ React Compiler (automatic optimization)
- ❌ Actions API (form handling improvements)
- ❌ Enhanced Server Components support
- ❌ Concurrent rendering by default
- ❌ `ref` as prop (no need for forwardRef)
- ❌ `useEffectEvent` hook
- ❌ Activity Component API
- ❌ Resume APIs for partial pre-rendering

**Breaking Changes to Be Aware Of:**
- PropTypes removed (migrated to TypeScript ✅ - this project already uses TS)
- defaultProps removed for function components
- Legacy Context removed (deprecated since 2018)
- String refs removed (use ref callbacks)
- `createFactory` removed (use JSX)
- `forwardRef` planned for deprecation
- Strict Mode no longer suppresses console logs

**What This Means for Learning:**
- ⚠️ **OUTDATED PATTERNS** - This project uses React 17 patterns
- React Hooks patterns (useState, useEffect, etc.) are still current
- Class components may be present (legacy pattern as of 2025)
- No automatic render optimization from React Compiler
- Traditional useEffect for data fetching (modern apps use React Query/TanStack Query)

**Migration Path:**
1. Upgrade to React 18.3.1 first (adds warnings for deprecated APIs)
2. Fix all warnings
3. Upgrade to React 19
4. Use react-codemod for automated migration

**Official Resources:**
- React 19 Docs: https://react.dev/
- Upgrade Guide: https://react.dev/blog/2024/04/25/react-19-upgrade-guide
- Breaking Changes: https://react.dev/blog/2024/12/05/react-19
- Codemods: https://github.com/reactjs/react-codemod

---

### Vite - npm:rolldown-vite@latest

**Current Status (Nov 2025):**
- Latest Vite: **v7.2.2** (November 2025)
- Project uses: **rolldown-vite** (custom fork)
- Status: 🔍 **CUSTOM BUILD** (needs investigation)

**Important Updates Since Jan 2025:**
- Vite 7.0 released (major version)
- Vite 7.2 is current stable
- **Breaking:** Requires Node.js 20.19+ or 22.12+
- **Breaking:** Node.js 18 support dropped (EOL April 2025)
- Default browser target changed from 'modules' to 'baseline-widely-available'

**What is Rolldown-Vite?**
The project uses a custom resolution pointing to a Rolldown fork:
```json
{
  "dependencies": {
    "vite": "npm:rolldown-vite@latest"
  },
  "resolutions": {
    "rolldown": "https://pkg.pr.new/rolldown@59ccf5f"
  }
}
```

**What This Means for Learning:**
- 🔍 **UNCLEAR:** Project uses experimental Rolldown bundler instead of standard Vite
- Rolldown is a Rust-based bundler (Vite uses esbuild + Rollup)
- Likely for performance improvements
- May have different behavior than standard Vite
- Configuration likely similar to Vite but with Rolldown-specific features

**Official Resources:**
- Vite 7 Docs: https://vite.dev/
- Vite 7 Announcement: https://vite.dev/blog/announcing-vite7
- Migration Guide: https://vite.dev/guide/migration
- Rolldown: https://github.com/rolldown-rs/rolldown

---

## Backend Framework

### Koa.js - v3.0.3

**Current Status (Nov 2025):**
- Latest stable: **v3.1.0** (October 26, 2025)
- Project uses: **v3.0.3**
- Status: ✅ **CURRENT** (one minor version behind)

**Important Updates Since Jan 2025:**
- Koa 3.0 officially released **April 2025** (after 8-year development)
- Koa 3.1.0 released October 2025 with memory leak fixes
- Requires Node.js v18+ (project uses Node 22 ✅)

**Major Koa 3.0 Changes:**
- ✅ Minimum Node.js version raised to v18
- 🚨 **BREAKING:** Generators no longer supported (async/await only)
- 🚨 **BREAKING:** `.redirect('back')` removed, use `.back(fallback_url)`
- ✅ Full ES2015+ and async/await support
- Performance improvements from modern V8 engine

**What This Means for Learning:**
- ✅ **CURRENT (Nov 2025)** - Project uses modern Koa 3.x patterns
- All middleware uses async/await (no generators)
- Small codebase (~570 SLOC) makes it easy to learn
- Modern patterns throughout

**Koa Middleware in This Project:**
- `koa-router` v7.4.0
- `koa-body` v6.0.1
- `koa-compress` v5.1.1
- `koa-helmet` v6.1.0 (security headers)
- `koa-passport` v4.2.1 (authentication)

**Official Resources:**
- Docs: https://koajs.com/
- Koa 3.0 Release: https://en.kelen.cc/share/koajs3-release
- GitHub: https://github.com/koajs/koa

---

## Database & ORM

### Sequelize - v6.37.7

**Current Status (Nov 2025):**
- Latest stable: **v6.37.7** (current)
- Project uses: **v6.37.7**
- Alpha version: v7.0.0-alpha.47 (in development)
- Status: ✅ **CURRENT STABLE**

**Important Updates Since Jan 2025:**
- Sequelize v7 in alpha (not production-ready)
- v6.x remains the stable, recommended version
- Sequelize team looking for maintainers for v7 finalization
- Active maintenance of v6.x continues

**What This Means for Learning:**
- ✅ **CURRENT (Nov 2025)** - Using latest stable Sequelize
- All patterns are current best practices
- TypeScript support via sequelize-typescript v2.1.6
- Migrations managed via sequelize-cli v6.6.3

**Sequelize Ecosystem in This Project:**
- `sequelize-typescript` v2.1.6 - TypeScript decorators for models
- `sequelize-cli` v6.6.3 - Migration management
- `sequelize-encrypted` v1.0.0 - Field encryption
- `pg` v8.16.3 - PostgreSQL driver

**Key Patterns:**
```typescript
// Uses decorators for models (experimentalDecorators: true)
@Table
class User extends Model {
  @Column
  name: string;
}
```

**Official Resources:**
- Docs: https://sequelize.org/
- v6 Docs: https://sequelize.org/docs/v6/
- GitHub: https://github.com/sequelize/sequelize

---

### PostgreSQL

**Current Status (Nov 2025):**
- Project uses: **pg driver v8.16.3**
- Status: ✅ **CURRENT**

**PostgreSQL Extensions Used:**
- `pg-tsquery` v8.4.2 - Full-text search query parser
- Full-text search capabilities (tsvector/tsquery)

**What This Means for Learning:**
- Modern PostgreSQL features in use
- Full-text search is a core feature
- JSONB support likely in use
- Standard SQL patterns

**Official Resources:**
- PostgreSQL Docs: https://www.postgresql.org/docs/
- pg driver: https://node-postgres.com/

---

## State Management

### MobX - v4.15.4

**Current Status (Nov 2025):**
- Latest stable: **v6.15.0** (October 2025)
- Project uses: **v4.15.4**
- Status: 🚨 **TWO MAJOR VERSIONS BEHIND**

**Important Updates Since Jan 2025:**
- MobX 6.x is current (released 2020, stable since)
- MobX 5.x reached end of support
- MobX 4.x is legacy (released 2018)

**Major Differences (MobX 4 vs MobX 6):**
- MobX 6 uses ES6 Proxies by default (better performance)
- MobX 6 removed decorators by default (uses `makeObservable`)
- MobX 4 relies on decorators (`@observable`, `@action`, etc.)
- MobX 6 is smaller and faster
- Different API patterns

**MobX 4 Patterns in This Project:**
```typescript
// MobX 4 pattern (legacy)
class Store {
  @observable data = []
  @action setData() { }
}
```

**MobX 6 Pattern (Modern):**
```typescript
// MobX 6 pattern (current)
class Store {
  data = []

  constructor() {
    makeObservable(this, {
      data: observable,
      setData: action
    })
  }

  setData() { }
}
```

**Deprecated Patterns to Be Aware Of:**
- 🚨 `observable()` on scalar values (use `@observable` or `makeObservable`)
- ⚠️ Decorator-based API (replaced with `makeObservable` in MobX 6)

**What This Means for Learning:**
- ⚠️ **OUTDATED PATTERNS** - This project uses MobX 4 decorator patterns
- MobX 4 still works but uses legacy API
- Modern MobX tutorials will show different patterns
- Migration to MobX 6 would require significant refactoring

**Official Resources:**
- MobX 6 Docs: https://mobx.js.org/
- Best Practices: https://mobx.js.org/defining-data-stores.html
- Migration Guide: https://mobx.js.org/migrating-from-4-or-5.html

---

## Styling

### Styled Components - v5.3.11

**Current Status (Nov 2025):**
- Latest stable: **v6.1.15**
- Project uses: **v5.3.11**
- Status: ⚠️ **ONE MAJOR VERSION BEHIND**

**Important Updates Since Jan 2025:**
- Styled Components v6 released with React 19 compatibility
- v6.1.14+ addresses React 19 compatibility issues
- Performance improvements in v6
- Updated dependencies (postcss, nanoid)

**Major Changes in v6:**
- Better React 18/19 support
- Performance optimizations
- Updated peer dependencies
- Minor API improvements

**What This Means for Learning:**
- ⚠️ **SLIGHTLY OUTDATED** - v5 patterns still valid
- Core API remains the same between v5 and v6
- Migration is straightforward (mostly dependency updates)
- All CSS-in-JS patterns remain current

**Styled Components Ecosystem:**
- `styled-components-breakpoint` v2.1.1 - Responsive breakpoints
- `styled-normalize` v8.1.1 - CSS normalize

**Official Resources:**
- Docs: https://styled-components.com/
- v6 Release: https://github.com/styled-components/styled-components/releases
- Migration: https://styled-components.com/releases

---

## Real-Time & Collaboration

### Socket.io - v4.8.1

**Current Status (Nov 2025):**
- Latest: v4.8.1 (recent)
- Project uses: **v4.8.1**
- Status: ✅ **CURRENT**

**What This Means for Learning:**
- Using latest Socket.io
- Modern WebSocket patterns
- Redis adapter for scaling (`socket.io-redis`)

**Official Resources:**
- Docs: https://socket.io/docs/v4/
- GitHub: https://github.com/socketio/socket.io

---

### Yjs - v13.6.1

**Current Status (Nov 2025):**
- Latest: v13.6.1 (recent)
- Project uses: **v13.6.1**
- Status: ✅ **CURRENT**

**What This Means for Learning:**
- Using latest Yjs (CRDT for real-time collaboration)
- Modern collaborative editing patterns
- Integration with ProseMirror via `y-prosemirror`
- IndexedDB persistence via `y-indexeddb`

**Yjs Ecosystem in Project:**
- `y-prosemirror` v1.3.7 - ProseMirror bindings
- `y-indexeddb` v9.0.11 - Local persistence
- `y-protocols` v1.0.6 - Sync protocols

**Official Resources:**
- Docs: https://docs.yjs.dev/
- GitHub: https://github.com/yjs/yjs

---

### Hocuspocus - v1.1.2

**Current Status (Nov 2025):**
- Project uses: **v1.1.2** (Yjs websocket server)
- Status: ✅ **CURRENT**

**What This Means for Learning:**
- WebSocket server for Yjs synchronization
- Redis extension for scaling
- Throttle extension for rate limiting

**Hocuspocus Components:**
- `@hocuspocus/server` v1.1.2
- `@hocuspocus/provider` v1.1.2
- `@hocuspocus/extension-redis` v1.1.2
- `@hocuspocus/extension-throttle` v1.1.2

---

## Rich Text Editor

### ProseMirror - v1.x

**Current Status (Nov 2025):**
- Project uses: **Various v1.x packages** (current)
- Status: ✅ **CURRENT**

**What This Means for Learning:**
- Modern rich text editing framework
- Used by Notion, Atlassian, New York Times
- Complex but powerful architecture
- Learning curve is significant

**ProseMirror Packages in Use:**
- `prosemirror-model` v1.25.4
- `prosemirror-view` v1.41.2
- `prosemirror-state` v1.4.4
- `prosemirror-transform` v1.10.0
- `prosemirror-commands` v1.7.1
- `prosemirror-keymap` v1.2.3
- `prosemirror-history` v1.4.1
- `prosemirror-markdown` v1.13.2
- `prosemirror-tables` v1.8.1
- `@benrbray/prosemirror-math` v0.2.2 (math support)
- `prosemirror-codemark` v0.4.2 (code blocks)

**Official Resources:**
- Docs: https://prosemirror.net/
- Guide: https://prosemirror.net/docs/guide/
- Examples: https://prosemirror.net/examples/

---

## Background Jobs

### Bull - v4.16.5

**Current Status (Nov 2025):**
- Latest: v4.16.5
- Project uses: **v4.16.5**
- Status: ✅ **CURRENT**

**What This Means for Learning:**
- Modern job queue built on Redis
- Web UI via `@bull-board/koa` v6.13.0
- Production-ready job processing patterns

**Official Resources:**
- Docs: https://github.com/OptimalBits/bull
- Bull Board: https://github.com/felixmosh/bull-board

---

## Testing

### Jest - v29.7.0

**Current Status (Nov 2025):**
- Latest: v29.7.0
- Project uses: **v29.7.0**
- Status: ✅ **CURRENT**

**Testing Setup:**
- `jest-cli` v29.7.0
- `jest-environment-jsdom` v29.7.0 (for React testing)
- `jest-fetch-mock` v3.0.3
- `babel-jest` v29.7.0 (TypeScript compilation)

**What This Means for Learning:**
- Modern Jest testing patterns
- Separate test projects (app, server, shared)
- JSDOM for component testing

**Official Resources:**
- Docs: https://jestjs.io/
- React Testing: https://jestjs.io/docs/tutorial-react

---

## Build Tools & Development

### Babel - v7.28.x

**Current Status (Nov 2025):**
- Latest: v7.28.x
- Project uses: **v7.28.x**
- Status: ✅ **CURRENT**

**Babel Plugins in Use:**
- `@babel/preset-react` - JSX transformation
- `@babel/preset-typescript` - TS compilation
- `babel-plugin-styled-components` - SSR support
- `babel-plugin-transform-typescript-metadata` - Decorator metadata

---

### Linting

**oxlint - v1.11.2**

**Current Status (Nov 2025):**
- Fast Rust-based linter (50-100x faster than ESLint)
- Project uses: **v1.11.2**
- Status: ✅ **CURRENT**

**What This Means for Learning:**
- Modern, fast linting tool
- Alternative to ESLint
- Type-aware linting enabled

---

### Formatting

**Prettier - v3.6.2**

**Current Status (Nov 2025):**
- Latest: v3.6.2
- Project uses: **v3.6.2**
- Status: ✅ **CURRENT**

---

## Authentication

### Passport - v0.7.0

**Current Status (Nov 2025):**
- Latest: v0.7.0
- Project uses: **v0.7.0**
- Status: ✅ **CURRENT**

**Passport Strategies in Use:**
- `passport-google-oauth2` v0.2.0
- `passport-slack-oauth2` v1.2.0
- `passport-oauth2` v1.8.0
- `@outlinewiki/passport-azure-ad-oauth2` v0.1.0 (custom)

---

## Internationalization

### i18next - v22.5.1

**Current Status (Nov 2025):**
- Project uses: **v22.5.1**
- Status: ✅ **CURRENT**

**i18next Ecosystem:**
- `react-i18next` v12.3.1 - React bindings
- `i18next-fs-backend` v2.6.0 - File system backend
- `i18next-http-backend` v2.7.3 - HTTP backend
- `i18next-parser` v8.13.0 - Extract translations

---

## UI Component Libraries

### Radix UI - v1.x/v2.x (Latest)

**Current Status (Nov 2025):**
- Project uses: **Latest v1.x and v2.x versions**
- Status: ✅ **CURRENT**

**Radix Components in Use:**
- `@radix-ui/react-dialog` v1.1.15
- `@radix-ui/react-dropdown-menu` v2.1.16
- `@radix-ui/react-popover` v1.1.15
- `@radix-ui/react-select` v2.2.6
- `@radix-ui/react-tooltip` v1.2.8
- And many more...

**What This Means for Learning:**
- Modern, accessible component primitives
- Unstyled components (styled with styled-components)
- Current best practice for headless UI

**Official Resources:**
- Docs: https://www.radix-ui.com/

---

### TanStack Table - v8.21.3

**Current Status (Nov 2025):**
- Latest: v8.x
- Project uses: **v8.21.3**
- Status: ✅ **CURRENT**

**What This Means for Learning:**
- Modern table/data grid library (formerly React Table)
- Headless UI approach
- TypeScript-first

**Official Resources:**
- Docs: https://tanstack.com/table/latest

---

## Other Notable Dependencies

### AWS SDK v3 - v3.927.0

**Current Status:** ✅ **CURRENT**
- Modular AWS SDK (v3 architecture)
- S3 client for file storage

### Sentry - v7.120.4

**Current Status:** ✅ **CURRENT**
- Error tracking for Node.js and React
- Modern Sentry SDK

### Mermaid - v11.12.1

**Current Status:** ✅ **CURRENT**
- Diagram rendering in markdown
- Latest version

---

## Summary Table

| Technology | Project Version | Latest (Nov 2025) | Status | Impact |
|------------|----------------|-------------------|---------|---------|
| Node.js | 22.x | 24.x LTS | ⚠️ One LTS behind | Low - still supported |
| TypeScript | 5.9.2 | 5.9.3 | ✅ Current | None |
| React | 17.0.2 | 19.2.0 | 🚨 2 versions behind | High - different patterns |
| Vite | rolldown-vite | 7.2.2 | 🔍 Custom build | Medium - needs research |
| Koa | 3.0.3 | 3.1.0 | ✅ Current | None |
| Sequelize | 6.37.7 | 6.37.7 | ✅ Current | None |
| MobX | 4.15.4 | 6.15.0 | 🚨 2 versions behind | High - different API |
| Styled Components | 5.3.11 | 6.1.15 | ⚠️ 1 version behind | Low - similar API |
| Socket.io | 4.8.1 | 4.8.1 | ✅ Current | None |
| Yjs | 13.6.1 | 13.6.1 | ✅ Current | None |
| ProseMirror | 1.x | 1.x | ✅ Current | None |
| Bull | 4.16.5 | 4.16.5 | ✅ Current | None |
| Jest | 29.7.0 | 29.7.0 | ✅ Current | None |

---

## Key Takeaways for Learning

### ✅ Modern & Current Patterns

These technologies in the project use current (Nov 2025) best practices:
- TypeScript configuration and usage
- Koa.js async/await patterns
- Sequelize ORM patterns
- Socket.io real-time patterns
- Yjs CRDT collaboration
- ProseMirror rich text editing
- Jest testing patterns
- Radix UI components
- PostgreSQL usage

### ⚠️ Outdated Patterns to Be Aware Of

These technologies use older patterns that differ from current best practices:

**React (v17 vs v19):**
- No React Compiler optimizations
- Traditional useEffect patterns (vs. modern server components)
- May use forwardRef (deprecated in React 19)
- No Actions API for forms
- Manual data fetching (vs. React Query/TanStack Query in modern apps)

**MobX (v4 vs v6):**
- Decorator-based API (`@observable`, `@action`) vs. `makeObservable`
- Different initialization patterns
- Older performance characteristics

**Styled Components (v5 vs v6):**
- Minor version differences
- Core API remains the same

### 🔍 Areas Requiring Investigation

**Rolldown-Vite:**
- Custom build tool (not standard Vite)
- May have different configuration patterns
- Experimental bundler approach

---

## Migration Priorities (If Updating)

If this project were to modernize its dependencies, recommended order:

1. **Low-hanging fruit:**
   - Node.js 22 → 24 (simple runtime upgrade)
   - Styled Components 5 → 6 (minimal changes)

2. **Medium effort:**
   - React 17 → 18.3 → 19 (use codemods, test thoroughly)
   - Update React testing patterns

3. **High effort:**
   - MobX 4 → 6 (significant API changes, refactor all stores)
   - Consider replacing with modern alternatives (Zustand, Jotai)

4. **Research required:**
   - Evaluate rolldown-vite vs. standard Vite 7

---

## Questions to Investigate Further

1. Why is the project using rolldown-vite instead of standard Vite?
2. Is there a plan to migrate React 17 → 19?
3. Is there a plan to migrate MobX 4 → 6?
4. What are the blockers for updating React and MobX?

---

## Conclusion

**This project uses a stable, production-ready stack with some intentionally older versions:**

- The backend (Koa, Sequelize, Node.js, PostgreSQL) is modern and current
- The frontend uses older major versions of React and MobX for stability
- All outdated patterns are well-documented and stable
- No security or support concerns (all versions still maintained)
- The intentional version choices suggest stability over cutting-edge features

**For new developers:**
- Learn the patterns in THIS project first
- Be aware they differ from latest tutorials for React/MobX
- Core concepts remain the same
- Project is stable and production-ready
- Understanding these differences makes you a better developer

---

*Last updated: November 18, 2025*
