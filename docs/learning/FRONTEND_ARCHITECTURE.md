# Frontend Architecture

**Complete guide to Outline's React + MobX 4 frontend.**

**Documentation Date:** November 18, 2025

---

## Overview

Outline's frontend uses **React 17** with **MobX 4** for state management. This guide assumes you know React and teaches you MobX patterns and how they work together.

⚠️ **Note:** Uses React 17 and MobX 4 (older versions) - patterns differ from latest tutorials.

---

## Architecture Overview

```
React Components (UI)
       ↓
  Observer HOC (MobX)
       ↓
  Stores (State Management)
       ↓
  Models (Data Entities)
       ↓
  API Client
       ↓
  Backend API
```

---

## MobX 4 - State Management

###

 Core Concepts

**Three decorators you need to know:**

```typescript
class DocumentsStore {
  @observable documents = new Map()    // Tracked state
  
  @action add(doc) {                   // Modify state
    this.documents.set(doc.id, doc)
  }
  
  @computed get sorted() {             // Derived state
    return Array.from(this.documents.values()).sort()
  }
}
```

🌉 **Bridge from React:**
- `@observable` = `useState`
- `@action` = setState function
- `@computed` = `useMemo`
- `observer` HOC = Subscribe to changes

### RootStore Pattern

**File:** `app/stores/RootStore.ts`

```typescript
class RootStore {
  documents: DocumentsStore
  collections: CollectionsStore
  auth: AuthStore
  ui: UiStore
  // ... 30+ stores
  
  constructor() {
    this.documents = new DocumentsStore(this)
    this.collections = new CollectionsStore(this)
    // All stores have access to rootStore
  }
}
```

**Usage in components:**

```typescript
const Document = observer(() => {
  const { documents, auth } = useStores()  // Get stores from context
  
  const doc = documents.get(docId)
  
  // Component re-renders when doc changes!
  return <h1>{doc.title}</h1>
})
```

---

## Models

**File:** `app/models/Document.ts`

```typescript
class Document extends ArchivableModel {
  @observable isSaving = false
  
  @Field
  @observable
  id: string
  
  @Field
  @observable
  title: string
  
  @Field
  @observable
  text: string
  
  @Relation(() => Collection)
  collection?: Collection
  
  @action
  async save() {
    this.isSaving = true
    const response = await client.post("/api/documents.update", {
      id: this.id,
      title: this.title,
      text: this.text
    })
    this.updateFromJson(response.data)
    this.isSaving = false
  }
}
```

**Key features:**
- `@Field` - Marks fields for serialization
- `@Relation` - Defines relationships
- Methods for CRUD operations

---

## Component Patterns

### Observer Pattern

```typescript
import { observer } from "mobx-react"

const DocumentRow = observer(({ documentId }) => {
  const { documents } = useStores()
  const doc = documents.get(documentId)
  
  // Auto re-renders when doc.title or doc.updatedAt changes
  return (
    <div>
      <h3>{doc.title}</h3>
      <time>{doc.updatedAt}</time>
    </div>
  )
})
```

💡 **Aha Moment:** `observer` makes components reactive - they auto-update when observables they use change!

### Accessing Stores

```typescript
// Via hook (modern)
const { documents, auth } = useStores()

// Via inject (legacy, still used in codebase)
@inject("documents", "auth")
@observer
class DocumentList extends React.Component {
  render() {
    const { documents } = this.props
    // ...
  }
}
```

---

## Data Flow

```
User clicks button
  ↓
Component calls store method
  ↓
Store makes API call
  ↓
Store updates @observable
  ↓
Observer components re-render automatically
```

**Example:**

```typescript
// Component
const CreateButton = observer(() => {
  const { documents } = useStores()
  
  const handleClick = async () => {
    const doc = await documents.create({ title: "New Doc" })
    history.push(doc.url)
  }
  
  return <button onClick={handleClick}>New Document</button>
})

// Store
class DocumentsStore {
  @action
  async create(params) {
    const response = await client.post("/api/documents.create", params)
    const doc = new Document(response.data, this)
    this.add(doc)  // Triggers re-render!
    return doc
  }
}
```

---

## Routing (React Router v5)

**File:** `app/routes/index.tsx`

⚠️ **Uses React Router v5** (not v6 - different API!)

```typescript
<Switch>
  <Route path="/doc/:id" component={DocumentScene} />
  <Route path="/collection/:id" component={CollectionScene} />
  <Route path="/settings" component={SettingsScene} />
</Switch>
```

**In components:**

```typescript
import { useParams, useHistory } from "react-router-dom"

const DocumentScene = () => {
  const { id } = useParams()  // Get route params
  const history = useHistory()  // Navigate programmatically
  
  const handleDelete = () => {
    history.push("/home")
  }
}
```

---

## Editor Integration (ProseMirror)

**File:** `app/components/Editor/Editor.tsx`

```typescript
import { EditorView } from "prosemirror-view"
import { EditorState } from "prosemirror-state"

class Editor extends React.Component {
  view: EditorView
  
  componentDidMount() {
    this.view = new EditorView(this.ref, {
      state: EditorState.create({
        doc: this.props.document.data,
        plugins: [...]
      }),
      dispatchTransaction: (transaction) => {
        const newState = this.view.state.apply(transaction)
        this.view.updateState(newState)
        
        // Sync to backend
        this.props.document.updateText(newState.doc)
      }
    })
  }
}
```

---

## Styling (Styled Components v5)

```typescript
import styled from "styled-components"

const Button = styled.button<{ primary?: boolean }>`
  background: ${props => props.primary ? props.theme.primary : "white"};
  padding: 10px 20px;
  border-radius: 4px;
`

const DocumentTitle = styled.h1`
  font-size: 2em;
  color: ${props => props.theme.text};
`
```

**Theme access:**

```typescript
import { useTheme } from "styled-components"

const Component = () => {
  const theme = useTheme()
  return <div style={{ color: theme.primary }}>Text</div>
}
```

---

## Common Patterns

### Loading States

```typescript
const DocumentList = observer(() => {
  const { documents } = useStores()
  
  React.useEffect(() => {
    documents.fetchPage()  // Load data
  }, [])
  
  if (documents.isFetching) return <LoadingIndicator />
  if (documents.isLoaded) return <List />
})
```

### Optimistic Updates

```typescript
@action
async star(document) {
  // Update UI immediately
  document.isStarred = true
  
  try {
    await client.post("/api/documents.star", { id: document.id })
  } catch (error) {
    // Rollback on error
    document.isStarred = false
    toast.error("Failed to star")
  }
}
```

---

## Next Steps

- [Backend Architecture](./BACKEND_ARCHITECTURE.md) - Server-side patterns
- [Data Flow Guide](./DATA_FLOW_GUIDE.md) - Complete request lifecycle
- [How-To Guide](./HOW_TO_GUIDE.md) - Practical examples

*Last Updated: November 18, 2025*
