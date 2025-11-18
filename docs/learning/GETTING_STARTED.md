# Getting Started with Outline

**Welcome!** This guide will help you set up Outline for local development in under 30 minutes.

**Documentation Date:** November 18, 2025
**Tech Stack:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Quick Start](#quick-start)
3. [Detailed Setup](#detailed-setup)
4. [Configuration](#configuration)
5. [Running the Application](#running-the-application)
6. [Verification](#verification)
7. [Troubleshooting](#troubleshooting)
8. [Next Steps](#next-steps)

---

## Prerequisites

### Required Software

**Node.js 22.x** (✅ Current for this project)
```bash
# Check your Node version
node --version  # Should be v22.x

# Install Node 22 via nvm (recommended)
nvm install 22
nvm use 22
```

⚠️ **Note:** This project requires Node.js 22. The latest LTS is Node 24 (Nov 2025), but Outline is tested with Node 22.

**Yarn 1.22+** (Package manager)
```bash
# Check yarn version
yarn --version  # Should be 1.22.x

# Install yarn if needed
npm install -g yarn
```

**PostgreSQL 12+** (Database)
```bash
# Check if PostgreSQL is installed
psql --version

# macOS
brew install postgresql@14
brew services start postgresql@14

# Ubuntu/Debian
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql

# Windows
# Download from https://www.postgresql.org/download/windows/
```

**Redis 6+** (Caching & job queue)
```bash
# Check if Redis is installed
redis-cli --version

# macOS
brew install redis
brew services start redis

# Ubuntu/Debian
sudo apt install redis-server
sudo systemctl start redis-server

# Windows
# Download from https://github.com/microsoftarchive/redis/releases
```

**Git** (Version control)
```bash
git --version
```

### Optional (Docker Alternative)

If you prefer Docker, you can use the provided `docker-compose.yml`:
```bash
# Install Docker Desktop
# Then run PostgreSQL and Redis via Docker
docker-compose up redis postgres -d
```

This will start Redis on `localhost:6379` and PostgreSQL on `localhost:5432`.

---

## Quick Start

**For the impatient (5 minutes to running):**

```bash
# 1. Clone the repository
git clone https://github.com/outline/outline.git
cd outline

# 2. Install dependencies
yarn install

# 3. Start PostgreSQL and Redis (via Docker)
docker-compose up -d redis postgres

# 4. Copy environment variables
cp .env.sample .env

# 5. Update .env with minimal config
# Edit these values in .env:
#   SECRET_KEY=your_random_32_byte_hex        # Generate: openssl rand -hex 32
#   UTILS_SECRET=another_random_key           # Generate: openssl rand -hex 32
#   DATABASE_URL=postgres://user:pass@localhost:5432/outline
#   REDIS_URL=redis://localhost:6379
#   URL=http://localhost:3000
#   PORT=3000

# 6. Create and migrate database
yarn db:create
yarn db:migrate

# 7. Build the application
yarn build

# 8. Start development server
yarn dev:watch

# 9. Open http://localhost:3000
```

Done! Jump to [Verification](#verification) to confirm everything works.

---

## Detailed Setup

### Step 1: Clone the Repository

```bash
git clone https://github.com/outline/outline.git
cd outline
```

### Step 2: Install Dependencies

Outline uses Yarn workspaces for the monorepo:

```bash
# Install all dependencies (frontend + backend)
yarn install
```

This will:
- Install ~200+ npm packages
- Apply patches via `patch-package`
- Set up husky git hooks
- Take 2-5 minutes depending on your connection

**If you see warnings:**
- Peer dependency warnings are normal
- Deprecated package warnings can be ignored

### Step 3: Set Up Database and Redis

#### Option A: Docker (Recommended)

```bash
# Start PostgreSQL and Redis containers
docker-compose up -d redis postgres

# Verify they're running
docker-compose ps
```

**Docker services:**
- PostgreSQL: `localhost:5432` (user: `user`, password: `pass`, database: `outline`)
- Redis: `localhost:6379`

#### Option B: Local Installation

**PostgreSQL:**
```bash
# macOS
brew services start postgresql@14

# Create database and user
psql postgres
CREATE DATABASE outline;
CREATE USER user WITH PASSWORD 'pass';
GRANT ALL PRIVILEGES ON DATABASE outline TO user;
\q
```

**Redis:**
```bash
# macOS
brew services start redis

# Verify Redis is running
redis-cli ping  # Should return "PONG"
```

### Step 4: Configure Environment Variables

```bash
# Copy the sample environment file
cp .env.sample .env
```

**Edit `.env` with required values:**

```bash
# Generate secrets (run these in your terminal)
openssl rand -hex 32  # Copy output for SECRET_KEY
openssl rand -hex 32  # Copy output for UTILS_SECRET
```

**Minimal `.env` configuration:**
```env
# Basic
NODE_ENV=development
URL=http://localhost:3000
PORT=3000

# Secrets (use values from openssl commands above)
SECRET_KEY=<your_generated_secret_key>
UTILS_SECRET=<your_generated_utils_secret>

# Database
DATABASE_URL=postgres://user:pass@localhost:5432/outline
PGSSLMODE=disable  # Only for local development

# Redis
REDIS_URL=redis://localhost:6379

# File Storage (local for development)
FILE_STORAGE=local
FILE_STORAGE_LOCAL_ROOT_DIR=./data

# Language
DEFAULT_LANGUAGE=en_US
```

**Authentication (choose one):**

For local development, you need at least one authentication provider:

**Option 1: Email Authentication (Simplest)**
```env
# Add to .env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-app-password
SMTP_FROM_EMAIL=your-email@gmail.com
SMTP_SECURE=false
```

**Option 2: Google OAuth (Requires setup)**
```env
GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-secret
```

**Option 3: Slack OAuth (Requires setup)**
```env
SLACK_CLIENT_ID=your-client-id
SLACK_CLIENT_SECRET=your-secret
```

💡 **For quick setup, use email authentication or skip auth setup and configure later.**

### Step 5: Database Setup

```bash
# Create the database (if not created manually)
yarn db:create

# Run migrations (creates all tables)
yarn db:migrate
```

**Expected output:**
```
== 20160101000000-initial: migrating =======
== 20160101000000-initial: migrated (0.123s)
== 20160102000000-add-teams: migrating =======
... (200+ migrations)
```

**Verify database:**
```bash
# Connect to database
psql postgres://user:pass@localhost:5432/outline

# List tables
\dt
# You should see: users, teams, documents, collections, etc.

\q
```

### Step 6: Build the Application

```bash
# Build frontend and backend
yarn build
```

This will:
1. Build frontend with Vite → `build/app/`
2. Extract i18n translations → `build/shared/i18n/`
3. Compile backend with Babel → `build/server/`

**Build time:** 30-60 seconds

**Expected output structure:**
```
build/
├── app/              # Frontend bundle
│   ├── index.html
│   ├── assets/
│   └── ...
├── server/           # Backend compiled
│   ├── index.js
│   ├── routes/
│   └── ...
└── shared/
    └── i18n/
```

---

## Running the Application

### Development Mode (Recommended)

**Watch mode (hot reload for both frontend and backend):**
```bash
yarn dev:watch
```

This starts:
- Frontend dev server (Vite) on `http://localhost:3001` (proxied)
- Backend server on `http://localhost:3000`
- Hot module replacement (HMR) for instant updates

**Backend only:**
```bash
yarn dev:backend
```

**Frontend only:**
```bash
yarn vite:dev
```

### Production Mode

```bash
# Build first
yarn build

# Start production server
yarn start
```

**Multi-process mode:**
```bash
# Set WEB_CONCURRENCY in .env
WEB_CONCURRENCY=4 yarn start
```

---

## Verification

### Check Everything is Running

**1. Open your browser:**
```
http://localhost:3000
```

You should see the Outline login page.

**2. Check backend health:**
```bash
curl http://localhost:3000/_health
# Should return: OK
```

**3. Check services:**
```bash
# Check if processes are running
ps aux | grep node

# Check logs
# Look for:
# - "Listening on http://localhost:3000" ✅
# - Database connection established ✅
# - Redis connection established ✅
```

**4. Create test account:**

If using email auth:
1. Go to `http://localhost:3000`
2. Click "Continue with Email"
3. Enter your email
4. Check terminal for magic link (in development, links are logged to console)
5. Click the link to sign in

**5. Verify features:**
- ✅ Can create a document
- ✅ Can edit with rich text editor
- ✅ Can create a collection
- ✅ Can search documents
- ✅ Can upload images

---

## Troubleshooting

### Common Issues

#### 1. Port 3000 already in use

**Error:** `EADDRINUSE: address already in use :::3000`

**Solution:**
```bash
# Find process using port 3000
lsof -i :3000

# Kill it
kill -9 <PID>

# Or use a different port in .env
PORT=3001
```

#### 2. Database connection failed

**Error:** `ECONNREFUSED 127.0.0.1:5432`

**Solution:**
```bash
# Check PostgreSQL is running
# macOS
brew services list | grep postgresql

# Ubuntu
sudo systemctl status postgresql

# Start it if stopped
brew services start postgresql@14  # macOS
sudo systemctl start postgresql    # Ubuntu

# Check with Docker
docker-compose ps
docker-compose up -d postgres
```

#### 3. Redis connection failed

**Error:** `ECONNREFUSED 127.0.0.1:6379`

**Solution:**
```bash
# Check Redis is running
redis-cli ping  # Should return "PONG"

# Start Redis
brew services start redis  # macOS
sudo systemctl start redis-server  # Ubuntu

# Or via Docker
docker-compose up -d redis
```

#### 4. Migration errors

**Error:** `Relation already exists` or `Migration failed`

**Solution:**
```bash
# Drop and recreate database
yarn db:reset

# Or manually
psql postgres
DROP DATABASE outline;
CREATE DATABASE outline;
GRANT ALL PRIVILEGES ON DATABASE outline TO user;
\q

# Then run migrations
yarn db:migrate
```

#### 5. Build errors

**Error:** `Cannot find module` or TypeScript errors

**Solution:**
```bash
# Clean build artifacts
yarn clean

# Reinstall dependencies
rm -rf node_modules
yarn install

# Rebuild
yarn build
```

#### 6. Node version mismatch

**Error:** `The engine "node" is incompatible`

**Solution:**
```bash
# Use Node 22
nvm install 22
nvm use 22

# Or if using system Node
# Download from https://nodejs.org/download/release/latest-v22.x/
```

#### 7. Out of memory during build

**Error:** `JavaScript heap out of memory`

**Solution:**
```bash
# Increase Node memory limit
export NODE_OPTIONS="--max-old-space-size=4096"

# Then rebuild
yarn build
```

### Development Issues

#### Hot reload not working

```bash
# Restart the dev server
# Press Ctrl+C to stop
yarn dev:watch
```

#### TypeScript errors in IDE

```bash
# Restart TypeScript server in VSCode
# Cmd+Shift+P → "TypeScript: Restart TS Server"

# Or check tsconfig.json is being recognized
```

#### Missing environment variables

```bash
# Check .env file exists
ls -la .env

# Validate required variables are set
grep SECRET_KEY .env
grep DATABASE_URL .env
grep REDIS_URL .env
```

---

## Development Scripts Reference

```bash
# Development
yarn dev:watch              # Full stack with hot reload ⭐ Most used
yarn dev:backend            # Backend only with reload
yarn vite:dev              # Frontend only with HMR

# Building
yarn build                 # Full production build
yarn build:server          # Backend only
yarn vite:build           # Frontend only
yarn clean                # Clean build artifacts

# Database
yarn db:create            # Create database
yarn db:migrate           # Run migrations
yarn db:rollback          # Rollback last migration
yarn db:reset             # Drop, create, and migrate
yarn db:create-migration  # Create new migration

# Testing
yarn test                 # Run all tests
yarn test:app            # Frontend tests
yarn test:server         # Backend tests
yarn test:shared         # Shared code tests

# Code Quality
yarn lint                # Lint all code (oxlint)
yarn lint:changed        # Lint only changed files
yarn format              # Format with Prettier
yarn format:check        # Check formatting

# Other
yarn upgrade             # Update dependencies
yarn install-local-ssl   # Install local SSL certificate
```

---

## File Structure Quick Reference

```
outline/
├── .env                    # Your local config (don't commit!)
├── .env.sample            # Template for .env
├── package.json           # Dependencies and scripts
├── docker-compose.yml     # Docker services
├── vite.config.ts         # Frontend build config
├── build.js               # Backend build script
├── tsconfig.json          # TypeScript config
├── .jestconfig.json       # Test config
│
├── app/                   # 🎨 Frontend (React)
│   ├── index.tsx          # Entry point
│   ├── components/        # UI components
│   ├── scenes/            # Pages
│   ├── stores/            # MobX stores
│   └── models/            # Data models
│
├── server/                # ⚙️ Backend (Koa)
│   ├── index.ts           # Entry point
│   ├── routes/            # API routes
│   ├── models/            # Database models
│   ├── commands/          # Business logic
│   ├── queues/            # Background jobs
│   └── migrations/        # Database migrations
│
├── shared/                # 🔄 Shared code
│   ├── editor/            # ProseMirror editor
│   ├── types.ts           # TypeScript types
│   └── utils/             # Utilities
│
├── plugins/               # 🔌 Plugins
│   ├── slack/             # Slack integration
│   ├── google/            # Google OAuth
│   └── ...                # More plugins
│
├── build/                 # 📦 Compiled output (generated)
├── data/                  # 💾 Local file storage (generated)
└── docs/                  # 📚 Documentation
    └── learning/          # ← You are here!
```

---

## Next Steps

Now that Outline is running locally, continue your learning:

1. **Understand the Architecture**
   → [Architecture Overview](./ARCHITECTURE_OVERVIEW.md)

2. **Explore the Codebase**
   → [Project Structure](./PROJECT_STRUCTURE.md)

3. **Learn the Technologies**
   → [Tech Stack Guide](./TECH_STACK_GUIDE.md)

4. **Trace a Request**
   → [Data Flow Guide](./DATA_FLOW_GUIDE.md)

5. **Dive into Frontend**
   → [Frontend Architecture](./FRONTEND_ARCHITECTURE.md)

6. **Dive into Backend**
   → [Backend Architecture](./BACKEND_ARCHITECTURE.md)

---

## Pro Tips

### 🎯 Productivity Boosters

**1. Use Tmux/iTerm2 splits for development:**
```
┌─────────────┬─────────────┐
│   Backend   │  Frontend   │
│ dev:backend │  vite:dev   │
├─────────────┴─────────────┤
│       Browser/Tests       │
└───────────────────────────┘
```

**2. VSCode recommended extensions:**
- ESLint
- Prettier
- TypeScript
- styled-components
- Jest
- GitLens
- Thunder Client (API testing)

**3. Browser extensions:**
- React Developer Tools
- MobX DevTools (search for "mobx-devtools-mst" for MobX 4)
- Redux DevTools (for debugging state)

**4. Database tools:**
- TablePlus (macOS)
- DBeaver (cross-platform)
- pgAdmin (official PostgreSQL GUI)
- Or VS Code PostgreSQL extension

**5. API testing:**
- Thunder Client (VS Code extension)
- Postman
- Insomnia
- Or just `curl`

### 🧪 Testing as You Code

```bash
# Run specific test file in watch mode
yarn test server/models/Document.test.ts --watch

# Run tests matching a pattern
yarn test --testNamePattern="should create document"

# Run all tests once
make test
```

### 🔍 Debugging

**Backend debugging with Node inspector:**
```bash
# Already enabled in dev mode
yarn dev:backend

# Then in Chrome: chrome://inspect
# Or in VSCode: F5 (with launch config)
```

**Frontend debugging:**
- Open React DevTools in browser
- Use browser debugger (`debugger` statement)
- Console.log is your friend (it's okay!)

### 📊 Monitoring in Development

Watch the console output for:
```
[database] → Database queries
[http] → HTTP requests (enable with DEBUG=http)
[email] → Email logs (magic links appear here)
[jobs] → Background job processing
```

---

## Common First-Time Developer Questions

**Q: Do I need to rebuild after every change?**
A: No! `yarn dev:watch` has hot reload. Changes appear automatically.

**Q: How do I add test data?**
A: Create documents/collections through the UI, or write a seed script.

**Q: Can I use different Node/PostgreSQL/Redis versions?**
A: Stick to the specified versions to avoid compatibility issues.

**Q: Where are uploaded files stored?**
A: In `./data/` when using `FILE_STORAGE=local`.

**Q: How do I reset everything?**
A: `yarn db:reset` (database) + `rm -rf data/` (files) + `yarn clean` (build)

**Q: Why am I not seeing logs?**
A: Set `DEBUG=*` and `LOG_LEVEL=debug` in `.env` for verbose logging.

---

**You're all set!** Continue to [Architecture Overview](./ARCHITECTURE_OVERVIEW.md) to understand how Outline works.

*Last Updated: November 18, 2025*
