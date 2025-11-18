# Data Flow Guide

**Complete journey of data through Outline - from browser click to database and back.**

**Documentation Date:** November 18, 2025

---

## Table of Contents

1. [Overview](#overview)
2. [Creating a Document - Complete Flow](#creating-a-document---complete-flow)
3. [API Request Lifecycle](#api-request-lifecycle)
4. [Real-Time Collaboration Flow](#real-time-collaboration-flow)
5. [Background Job Flow](#background-job-flow)
6. [Authentication Flow](#authentication-flow)
7. [WebSocket Notification Flow](#websocket-notification-flow)

---

## Overview

Understanding data flow is crucial for full-stack development. This guide traces actual data through Outline's architecture with **real code examples**.

🧠 **Mental Model:** Think of data flow like a restaurant:
- **Frontend (React)** = Customer ordering food
- **API Routes** = Waiter taking the order
- **Commands** = Chef preparing the food
- **Models** = Recipe and ingredients
- **Database** = Pantry storing ingredients
- **Events** = Bell ringing when food is ready
- **Background Jobs** = Dishwashing happening in parallel

---

## Creating a Document - Complete Flow

Let's trace what happens when you click "New Document" in Outline, following the actual code.

### Step 1: User Click (Frontend)

User clicks "New Document" button

### Step 2: MobX Store Action

**File:** `app/stores/DocumentsStore.ts`

```typescript
class DocumentsStore {
  @action
  async create(params: DocumentParams) {
    // 1. Call the API
    const response = await client.post("/api/documents.create", params)

    // 2. Create model and add to store
    const document = new Document(response.data, this)
    this.add(document)

    return document
  }
}
```

🌉 **Bridge from React:** MobX stores = Redux (action creators + reducers), but simpler!

### Step 3: HTTP Request

```http
POST /api/documents.create HTTP/1.1
Authorization: Bearer <jwt-token>
Content-Type: application/json

{
  "title": "Untitled",
  "collectionId": "abc123"
}
```

### Step 4: Middleware Stack

**File:** `server/routes/api/documents/documents.ts`

```typescript
router.post(
  "documents.create",
  auth(),              // ← Who are you?
  validate(schema),    // ← Is your data valid?
  transaction(),       // ← Start DB transaction
  async (ctx) => {
    // Route handler
  }
)
```

Middleware runs **top to bottom** like a waterfall:
```
Request → auth() → validate() → transaction() → Handler → Response
```

### Step 5: Authorization Check

```typescript
// Check permission
authorize(user, "createDocument", collection)
```

**File:** `server/policies/collection.ts`

Policies check: "Can this user do this action?"

### Step 6: Command Execution

**File:** `server/commands/documentCreator.ts`

```typescript
export default async function documentCreator({
  title, text, collectionId, user, ctx
}: Props): Promise<Document> {
  // 1. Create the document
  const document = await Document.create({
    title: title || "",
    text: text || "",
    collectionId,
    teamId: user.teamId,
    userId: user.id
  }, { transaction })

  // 2. Create initial revision
  await Revision.createFromDocument(document, { transaction })

  // 3. Emit event for background processing
  await Event.create({
    name: "documents.create",
    documentId: document.id
  }, { transaction })

  return document
}
```

🎯 **Command Pattern Benefits:**
- **Reusability:** Used by API, imports, tests
- **Testability:** Test in isolation
- **Consistency:** Same logic everywhere

### Step 7: Event & Background Jobs

```typescript
// Event gets picked up by processor
await Event.create({ name: "documents.create", ... })
```

**Triggers:**
- Webhook notifications
- Email notifications
- Search index update

### Step 8: Presenter (Format Response)

**File:** `server/presenters/document.ts`

```typescript
export default function presentDocument(document, options) {
  return {
    id: document.id,
    title: document.title,
    createdBy: presentUser(document.createdBy),
    permissions: {
      update: can(options?.user, "update", document),
      delete: can(options?.user, "delete", document)
    }
  }
}
```

⚠️ **Never** send raw database models to frontend!

### Step 9: Frontend Updates

```typescript
// MobX automatically triggers React re-render
this.documents.set(document.id, document)
```

💡 **Aha Moment:** With MobX, components auto-update when data changes!

---

## Complete Flow Diagram

```
Frontend: User Click
    ↓
Store.create() → HTTP POST
    ↓
Backend: Middleware Stack
    ↓
auth() → validate() → transaction()
    ↓
Route Handler: authorize()
    ↓
Command: documentCreator()
    ↓
Database: Document.create() + Revision.create() + Event.create()
    ↓
Presenter: Format response
    ↓
Frontend: Store updates → React re-renders
    ↓
Background: Jobs process event (webhooks, emails, search)
```

---

## API Request Lifecycle (Generic)

Every API request follows:

1. **Authentication** - Verify JWT, load user
2. **Validation** - Check request schema (Zod)
3. **Authorization** - Check permissions (policies)
4. **Business Logic** - Execute command
5. **Response** - Format with presenter

---

## Real-Time Collaboration Flow

Multiple users editing same document:

```
User A types "Hello"          User B types "World"
    ↓                              ↓
Y.js operation (CRDT)         Y.js operation
    ↓                              ↓
WebSocket to Hocuspocus       WebSocket
    ↓                              ↓
    ←──── Server merges ────→
    ↓                              ↓
Both see "HelloWorld" (automatic merge!)
```

**File:** `server/services/collaboration.ts`

Uses **Y.js** (CRDT) - Like Git for real-time documents!

---

## Background Job Flow

```
Event Created → Processor → Queue Jobs → Worker Executes
```

**Example:** Document Import

1. User uploads file → Queue job
2. Worker processes → Parse file
3. Create documents → Use documentCreator command
4. Update progress → Frontend polls status

---

## Authentication Flow (OAuth)

```
User → Click "Login with Google"
  ↓
Redirect to Google
  ↓
Google callback with code
  ↓
Exchange code for access token
  ↓
Get user profile
  ↓
Create/update user in DB
  ↓
Create JWT token
  ↓
Set httpOnly cookie
  ↓
Redirect to app
```

**File:** `server/routes/auth/providers/google.ts`

---

## WebSocket Notification Flow

```
User updates document
  ↓
Event.create("documents.update")
  ↓
WebsocketsProcessor
  ↓
socket.io.broadcast()
  ↓
All clients receive notification
  ↓
Frontend shows toast: "Doc updated by X"
```

**File:** `server/queues/processors/WebsocketsProcessor.ts`

---

## Summary

**Key Patterns:**

1. **Middleware Stack** - Layered request processing
2. **Command Pattern** - Reusable business logic
3. **Policy-Based Auth** - Centralized permissions
4. **Event-Driven** - Async background processing
5. **Presenter Pattern** - Safe data formatting
6. **CRDT (Y.js)** - Conflict-free collaboration
7. **MobX Reactivity** - Auto UI updates

🎯 **Remember:** Data flows **one direction**:
```
User → Store → API → Middleware → Command → DB → Events → Jobs
  ↑                                              ↓
  └──────────── Response ← Presenter ───────────┘
```

---

## Next Steps

- [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) - React + MobX deep dive
- [Backend Architecture](./BACKEND_ARCHITECTURE.md) - Koa + Commands deep dive
- [Code Tours](./CODE_TOURS.md) - Follow specific journeys
- [How-To Guide](./HOW_TO_GUIDE.md) - Build features

**Practice:** Trace the flow for **deleting a document**!

*Last Updated: November 18, 2025*
