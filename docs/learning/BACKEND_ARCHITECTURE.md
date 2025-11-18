# Backend Architecture

**Complete guide to Outline's Koa + Sequelize backend.**

**Documentation Date:** November 18, 2025

---

## Overview

Outline's backend uses **Koa 3** (Express alternative) with **Sequelize 6** ORM and follows a **layered architecture** with clear separation of concerns.

✅ **Modern patterns:** Koa 3 and Sequelize 6 are current (Nov 2025)

---

## Architecture Layers

```
HTTP Request
    ↓
1. Middleware (auth, validation, transaction)
    ↓
2. Route Handler (parse request)
    ↓
3. Policy Check (authorization)
    ↓
4. Command Execution (business logic)
    ↓
5. Model Operations (database)
    ↓
6. Event Emission (background jobs)
    ↓
7. Presenter (format response)
    ↓
HTTP Response
```

---

## Koa Middleware

### Middleware Stack

**File:** `server/routes/api/documents/documents.ts`

```typescript
router.post(
  "documents.create",
  auth(),              // 1. Authenticate
  validate(schema),    // 2. Validate input
  transaction(),       // 3. Start DB transaction
  async (ctx) => {     // 4. Route handler
    // Handle request
  }
)
```

### Custom Middleware

```typescript
// Middleware structure
export default function myMiddleware() {
  return async (ctx: APIContext, next: Next) => {
    // Before request
    console.log("Before")
    
    await next()  // Call next middleware/handler
    
    // After request
    console.log("After")
  }
}
```

### Key Middlewares

**Authentication:** `server/middlewares/authentication.ts`
```typescript
export default function auth() {
  return async (ctx, next) => {
    const token = ctx.headers.authorization?.replace("Bearer ", "")
    const payload = verifyJWT(token)
    ctx.state.auth = {
      user: await User.findByPk(payload.userId),
      token: payload
    }
    await next()
  }
}
```

**Validation:** Uses Zod schemas
```typescript
const schema = z.object({
  body: z.object({
    title: z.string(),
    collectionId: z.string().uuid()
  })
})

validate(schema)  // Middleware
```

**Transaction:** Automatic rollback on error
```typescript
export function transaction() {
  return async (ctx, next) => {
    const t = await sequelize.transaction()
    ctx.state.transaction = t
    try {
      await next()
      await t.commit()
    } catch (error) {
      await t.rollback()
      throw error
    }
  }
}
```

---

## Command Pattern

Commands encapsulate business logic in reusable functions.

### Why Commands?

**File:** `server/commands/documentCreator.ts`

```typescript
export default async function documentCreator({
  title, text, collectionId, user, ctx
}: Props): Promise<Document> {
  const { transaction } = ctx.context
  
  // 1. Validate business rules
  const collection = await Collection.findByPk(collectionId, { transaction })
  if (!collection) throw new NotFoundError()
  
  // 2. Create document
  const document = await Document.create({
    title,
    text,
    collectionId,
    teamId: user.teamId,
    userId: user.id
  }, { transaction })
  
  // 3. Create revision
  await Revision.createFromDocument(document, { transaction })
  
  // 4. Emit event
  await Event.create({
    name: "documents.create",
    documentId: document.id
  }, { transaction })
  
  return document
}
```

**Used by:**
- API endpoint: `POST /api/documents.create`
- Document import job
- Template instantiation
- Tests

🎯 **Key Benefit:** Business logic in one place, reused everywhere!

---

## Models (Sequelize)

**File:** `server/models/Document.ts`

```typescript
@Table({ tableName: "documents", paranoid: true })
class Document extends Model {
  @Column(DataType.UUID)
  id: string
  
  @Column
  title: string
  
  @Column(DataType.TEXT)
  text: string
  
  @ForeignKey(() => Collection)
  @Column(DataType.UUID)
  collectionId: string
  
  @BelongsTo(() => Collection)
  collection: Collection
  
  @HasMany(() => Revision)
  revisions: Revision[]
  
  // Lifecycle hooks
  @BeforeCreate
  static async generateUrlId(doc: Document) {
    doc.urlId = generateUrlId(doc.title)
  }
  
  @AfterCreate
  static async createBacklinks(doc: Document) {
    // Create relationship records
  }
}
```

### Key Features

**Decorators:**
- `@Table` - Define table
- `@Column` - Define column
- `@ForeignKey` - Foreign key
- `@BelongsTo` / `@HasMany` - Relationships

**Lifecycle Hooks:**
- `@BeforeCreate` - Before INSERT
- `@AfterCreate` - After INSERT
- `@BeforeSave` - Before UPDATE
- `@BeforeDestroy` - Before DELETE

**Paranoid Mode:**
- Soft deletes (sets `deletedAt`)
- Records not actually deleted
- Can be restored

---

## Policies (Authorization)

**File:** `server/policies/cancan.ts`

```typescript
export function authorize(
  user: User,
  action: string,
  resource: Model
) {
  const abilities = new Abilities()
  const policy = policies[resource.constructor.name]
  
  policy(user, resource, abilities)
  
  if (!abilities.can(action)) {
    throw new AuthorizationError()
  }
}
```

**File:** `server/policies/document.ts`

```typescript
export default function present(
  user: User,
  document: Document,
  abilities: Abilities
) {
  // Everyone in team can read
  if (user.teamId === document.teamId) {
    abilities.add("read")
  }
  
  // Author can update/delete
  if (user.id === document.createdById) {
    abilities.add("update")
    abilities.add("delete")
  }
  
  // Admins can do anything
  if (user.isAdmin) {
    abilities.add("*")
  }
}
```

**Usage:**

```typescript
// In route handler
authorize(user, "update", document)

// Or check without throwing
if (can(user, "delete", document)) {
  // Show delete button
}
```

---

## Presenters (Response Formatting)

**File:** `server/presenters/document.ts`

```typescript
export default function presentDocument(
  document: Document,
  options?: { user?: User }
) {
  return {
    id: document.id,
    title: document.title,
    text: document.text,
    url: document.url,
    createdAt: document.createdAt,
    updatedAt: document.updatedAt,
    
    // Transform relationships
    createdBy: presentUser(document.createdBy),
    collection: presentCollection(document.collection),
    
    // Add permissions
    permissions: {
      read: true,
      update: can(options?.user, "update", document),
      delete: can(options?.user, "delete", document)
    }
  }
}
```

⚠️ **Security:** Never send raw models to frontend!
- Hide sensitive fields
- Add computed fields
- Include permissions

---

## Event-Driven Architecture

### Event Flow

```
Model Operation (save, delete)
    ↓
Event.create()
    ↓
EventsProcessor detects event
    ↓
Queue background jobs
    ↓
Worker processes jobs
```

**Creating events:**

```typescript
// In command
await Event.create({
  name: "documents.create",
  documentId: document.id,
  collectionId: document.collectionId,
  teamId: user.teamId,
  actorId: user.id,
  ip: ctx.ip
}, { transaction })
```

**Processing events:**

**File:** `server/queues/processors/DocumentsProcessor.ts`

```typescript
export default class DocumentsProcessor {
  static async handleEvent(event: Event) {
    switch (event.name) {
      case "documents.create":
        // Send webhooks
        await WebhookQueue.add({ eventId: event.id })
        
        // Send email notifications
        await EmailQueue.add({
          type: "documentCreated",
          documentId: event.documentId
        })
        
        // Update search index
        await SearchQueue.add({
          action: "index",
          documentId: event.documentId
        })
        break
    }
  }
}
```

---

## Background Jobs (Bull)

**File:** `server/queues/tasks/DocumentImportTask.ts`

```typescript
export default class DocumentImportTask extends Task<Props> {
  async perform({ fileId, teamId }: Props) {
    // 1. Load file
    const file = await FileStorage.get(fileId)
    
    // 2. Parse file
    const parsed = await parseMarkdown(file)
    
    // 3. Create documents (reuse command!)
    for (const page of parsed.pages) {
      await documentCreator({
        title: page.title,
        text: page.content,
        collectionId: parsed.collectionId,
        user: await User.findByPk(userId)
      })
      
      // Update progress
      this.progress = (i / pages.length) * 100
    }
    
    return { documentsCreated: pages.length }
  }
}
```

**Scheduling:**

```typescript
// Queue a job
const job = await DocumentImportTask.schedule({
  fileId: "file-123",
  teamId: "team-456"
})

// Check status
const status = await DocumentImportTask.getStatus(job.id)
```

---

## Multi-Service Architecture

Outline runs multiple services:

**1. Web** - Serves API and static files
**2. Worker** - Background job processing
**3. Collaboration** - Real-time editing (Yjs)
**4. WebSockets** - Live notifications
**5. Cron** - Scheduled tasks

All share code (models, commands) but run separately.

**File:** `server/index.ts`

```typescript
const services = env.SERVICES.split(",")

if (services.includes("web")) {
  startWebService()
}

if (services.includes("worker")) {
  startWorkerService()
}

if (services.includes("collaboration")) {
  startCollaborationService()
}
```

---

## Error Handling

**Custom errors:**

**File:** `server/errors.ts`

```typescript
class ValidationError extends Error {
  statusCode = 400
}

class AuthorizationError extends Error {
  statusCode = 403
}

class NotFoundError extends Error {
  statusCode = 404
}
```

**Global error handler:**

```typescript
app.use(async (ctx, next) => {
  try {
    await next()
  } catch (error) {
    ctx.status = error.statusCode || 500
    ctx.body = {
      error: error.message,
      code: error.code
    }
  }
})
```

---

## Database Queries

### Basic Queries

```typescript
// Find by primary key
const doc = await Document.findByPk(id)

// Find one
const doc = await Document.findOne({
  where: { title: "My Doc" }
})

// Find all
const docs = await Document.findAll({
  where: {
    teamId: user.teamId,
    deletedAt: null
  },
  order: [["createdAt", "DESC"]],
  limit: 10
})
```

### Complex Queries

```typescript
// With relationships
const doc = await Document.findByPk(id, {
  include: [
    { model: User, as: "createdBy" },
    { model: Collection }
  ]
})

// With conditions
const docs = await Document.findAll({
  where: {
    [Op.or]: [
      { title: { [Op.like]: `%${query}%` } },
      { text: { [Op.like]: `%${query}%` } }
    ]
  }
})
```

---

## Testing

**File:** `server/routes/api/documents/documents.test.ts`

```typescript
describe("documents.create", () => {
  it("should create document", async () => {
    const user = await buildUser()
    const collection = await buildCollection({ teamId: user.teamId })
    
    const res = await server.post("/api/documents.create", {
      body: {
        title: "Test Doc",
        collectionId: collection.id
      },
      headers: {
        authorization: `Bearer ${user.getJwtToken()}`
      }
    })
    
    expect(res.status).toBe(200)
    expect(res.body.data.title).toBe("Test Doc")
  })
})
```

---

## Next Steps

- [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) - Client-side patterns
- [Database Architecture](./DATABASE_ARCHITECTURE.md) - Data models
- [How-To Guide](./HOW_TO_GUIDE.md) - Build features

*Last Updated: November 18, 2025*
