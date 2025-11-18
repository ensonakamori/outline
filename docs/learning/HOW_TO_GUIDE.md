# How-To Guide

**Practical step-by-step tutorials for common development tasks.**

**Documentation Date:** November 18, 2025

---

## How to Add a New API Endpoint

### Step 1: Create Route File

**File:** `server/routes/api/myresource/myresource.ts`

```typescript
import Router from "koa-router"
import auth from "@server/middlewares/authentication"
import validate from "@server/middlewares/validate"
import { MyResource } from "@server/models"
import * as T from "./schema"

const router = new Router()

router.post(
  "myresource.create",
  auth(),
  validate(T.MyResourceCreateSchema),
  async (ctx: APIContext<T.MyResourceCreateReq>) => {
    const { auth } = ctx.state
    const { name, data } = ctx.input.body
    
    const resource = await MyResource.create({
      name,
      data,
      teamId: auth.user.teamId,
      userId: auth.user.id
    })
    
    ctx.body = {
      data: presentMyResource(resource)
    }
  }
)

export default router
```

### Step 2: Create Schema

**File:** `server/routes/api/myresource/schema.ts`

```typescript
import { z } from "zod"

export const MyResourceCreateSchema = z.object({
  body: z.object({
    name: z.string().min(1).max(100),
    data: z.object({}).optional()
  })
})

export type MyResourceCreateReq = z.infer<typeof MyResourceCreateSchema>
```

### Step 3: Register Route

**File:** `server/routes/api/index.ts`

```typescript
import myresource from "./myresource"

router.use("/", myresource.routes())
```

### Step 4: Create Presenter

**File:** `server/presenters/myresource.ts`

```typescript
export default function presentMyResource(resource: MyResource) {
  return {
    id: resource.id,
    name: resource.name,
    data: resource.data,
    createdAt: resource.createdAt
  }
}
```

### Step 5: Test

**File:** `server/routes/api/myresource/myresource.test.ts`

```typescript
import { buildUser } from "@server/test/factories"
import { getTestServer } from "@server/test/support"

describe("myresource.create", () => {
  const server = getTestServer()
  
  it("should create resource", async () => {
    const user = await buildUser()
    
    const res = await server.post("/api/myresource.create", {
      body: { name: "Test" },
      headers: { authorization: `Bearer ${user.getJwtToken()}` }
    })
    
    expect(res.status).toBe(200)
    expect(res.body.data.name).toBe("Test")
  })
})
```

---

## How to Add a Database Model

### Step 1: Create Migration

```bash
yarn db:create-migration --name add-myresource
```

**File:** `server/migrations/YYYYMMDDHHMMSS-add-myresource.js`

```javascript
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable("myresources", {
      id: {
        type: Sequelize.UUID,
        primaryKey: true,
        defaultValue: Sequelize.UUIDV4
      },
      name: {
        type: Sequelize.STRING,
        allowNull: false
      },
      data: {
        type: Sequelize.JSONB
      },
      teamId: {
        type: Sequelize.UUID,
        allowNull: false,
        references: {
          model: "teams",
          key: "id"
        }
      },
      userId: {
        type: Sequelize.UUID,
        references: {
          model: "users",
          key: "id"
        }
      },
      createdAt: {
        type: Sequelize.DATE,
        allowNull: false
      },
      updatedAt: {
        type: Sequelize.DATE,
        allowNull: false
      },
      deletedAt: {
        type: Sequelize.DATE
      }
    })
    
    await queryInterface.addIndex("myresources", ["teamId"])
  },
  
  down: async (queryInterface) => {
    await queryInterface.dropTable("myresources")
  }
}
```

### Step 2: Create Model

**File:** `server/models/MyResource.ts`

```typescript
import {
  Table,
  Column,
  ForeignKey,
  BelongsTo,
  DataType
} from "sequelize-typescript"
import Model from "./base/Model"
import Team from "./Team"
import User from "./User"

@Table({
  tableName: "myresources",
  paranoid: true  // Soft deletes
})
export default class MyResource extends Model {
  @Column(DataType.UUID)
  id: string
  
  @Column
  name: string
  
  @Column(DataType.JSONB)
  data: object
  
  @ForeignKey(() => Team)
  @Column(DataType.UUID)
  teamId: string
  
  @BelongsTo(() => Team)
  team: Team
  
  @ForeignKey(() => User)
  @Column(DataType.UUID)
  userId: string
  
  @BelongsTo(() => User)
  user: User
}
```

### Step 3: Run Migration

```bash
yarn db:migrate
```

---

## How to Add a React Component

### Step 1: Create Component

**File:** `app/components/MyComponent/MyComponent.tsx`

```typescript
import { observer } from "mobx-react"
import * as React from "react"
import styled from "styled-components"
import { useStores } from "~/hooks/useStores"

type Props = {
  documentId: string
}

const MyComponent = ({ documentId }: Props) => {
  const { documents } = useStores()
  const document = documents.get(documentId)
  
  if (!document) return null
  
  return (
    <Container>
      <Title>{document.title}</Title>
      <Text>{document.text}</Text>
    </Container>
  )
}

const Container = styled.div`
  padding: 16px;
  background: ${props => props.theme.background};
`

const Title = styled.h2`
  font-size: 24px;
  color: ${props => props.theme.text};
`

const Text = styled.p`
  color: ${props => props.theme.textSecondary};
`

export default observer(MyComponent)
```

### Step 2: Export from Index

**File:** `app/components/index.ts`

```typescript
export { default as MyComponent } from "./MyComponent"
```

### Step 3: Use in Scene

```typescript
import { MyComponent } from "~/components"

const DocumentScene = () => {
  return <MyComponent documentId={docId} />
}
```

---

## How to Add a MobX Store

### Step 1: Create Store

**File:** `app/stores/MyResourcesStore.ts`

```typescript
import { action, runInAction } from "mobx"
import MyResource from "~/models/MyResource"
import RootStore from "./RootStore"
import Store from "./base/Store"

export default class MyResourcesStore extends Store<MyResource> {
  constructor(rootStore: RootStore) {
    super(rootStore, MyResource)
  }
  
  @action
  async fetchAll() {
    this.isFetching = true
    
    try {
      const res = await this.client.post("/api/myresources.list")
      
      runInAction("MyResourcesStore#fetchAll", () => {
        res.data.forEach(data => {
          this.add(new MyResource(data, this))
        })
        this.isLoaded = true
      })
    } finally {
      this.isFetching = false
    }
  }
}
```

### Step 2: Register in RootStore

**File:** `app/stores/RootStore.ts`

```typescript
import MyResourcesStore from "./MyResourcesStore"

export default class RootStore {
  myResources: MyResourcesStore
  
  constructor() {
    this.myResources = new MyResourcesStore(this)
  }
}
```

### Step 3: Use in Components

```typescript
const Component = observer(() => {
  const { myResources } = useStores()
  
  useEffect(() => {
    myResources.fetchAll()
  }, [])
  
  const resources = Array.from(myResources.orderedData)
  
  return (
    <div>
      {resources.map(r => <div key={r.id}>{r.name}</div>)}
    </div>
  )
})
```

---

## How to Add a Command

**File:** `server/commands/myResourceCreator.ts`

```typescript
import { MyResource, Event } from "@server/models"
import { APIContext } from "@server/types"
import User from "@server/models/User"

type Props = {
  name: string
  data: object
  user: User
  ctx: APIContext
  ip?: string
}

export default async function myResourceCreator({
  name,
  data,
  user,
  ctx,
  ip
}: Props): Promise<MyResource> {
  const { transaction } = ctx.state
  
  // Create resource
  const resource = await MyResource.create({
    name,
    data,
    teamId: user.teamId,
    userId: user.id
  }, { transaction })
  
  // Emit event
  await Event.create({
    name: "myresources.create",
    modelId: resource.id,
    teamId: user.teamId,
    actorId: user.id,
    ip
  }, { transaction })
  
  return resource
}
```

**Use in route:**

```typescript
import myResourceCreator from "@server/commands/myResourceCreator"

router.post("myresource.create", async (ctx) => {
  const resource = await myResourceCreator({
    ...ctx.input.body,
    user: ctx.state.auth.user,
    ctx
  })
  
  ctx.body = { data: presentMyResource(resource) }
})
```

---

## How to Add Tests

### Backend Test

```typescript
import { buildUser, buildMyResource } from "@server/test/factories"
import { getTestServer } from "@server/test/support"

describe("#myMethod", () => {
  it("should work correctly", async () => {
    const user = await buildUser()
    const resource = await buildMyResource({ userId: user.id })
    
    const result = resource.myMethod()
    
    expect(result).toBe("expected")
  })
})
```

### Frontend Test

```typescript
import { render, screen } from "@testing-library/react"
import MyComponent from "./MyComponent"

describe("MyComponent", () => {
  it("should render title", () => {
    render(<MyComponent title="Test" />)
    
    expect(screen.getByText("Test")).toBeInTheDocument()
  })
})
```

---

## How to Debug

### Backend

```bash
# Enable debug logs
DEBUG=* yarn dev:backend

# Specific categories
DEBUG=database,http yarn dev:backend

# Use Node inspector
yarn dev:backend
# Then open chrome://inspect
```

### Frontend

```typescript
// Access stores in console
window.stores

// Debug MobX
import { trace } from "mobx"

@computed get something() {
  trace()  // Logs dependencies
  return this.data
}
```

---

## How to Add Background Job

**File:** `server/queues/tasks/MyTask.ts`

```typescript
import BaseTask from "./BaseTask"

type Props = {
  resourceId: string
}

export default class MyTask extends BaseTask<Props> {
  async perform({ resourceId }: Props) {
    const resource = await MyResource.findByPk(resourceId)
    
    // Do work
    await processResource(resource)
    
    return { processed: true }
  }
}
```

**Schedule:**

```typescript
await MyTask.schedule({ resourceId: "123" })
```

---

*Last Updated: November 18, 2025*
