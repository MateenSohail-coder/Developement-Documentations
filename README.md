# Redux Toolkit (RTK) — Complete Documentation
> **Mix: Roman Urdu + English** | MERN Stack / Next.js ke liye

---

## 📚 Table of Contents

1. [Redux Kya Hai?](#1-redux-kya-hai)
2. [Store](#2-store)
3. [createSlice + Reducers](#3-createslice--reducers)
4. [useSelector](#4-useselector)
5. [useDispatch](#5-usedispatch)
6. [createAsyncThunk](#6-createasyncthunk)
7. [extraReducers](#7-extrareducers)
8. [RTK Query](#8-rtk-query)
9. [Middleware](#9-middleware)
10. [Immer](#10-immer)
11. [Redux DevTools](#11-redux-devtools)
12. [Selectors / Reselect](#12-selectors--reselect)
13. [Code Splitting](#13-code-splitting)
14. [Project File Structure](#14-project-file-structure)
15. [Quick Reference Cheatsheet](#15-quick-reference-cheatsheet)

---

## 1. Redux Kya Hai?

> **Redux = Poori app ka ek centralized state manager**

### Bina Redux (useContext wala tarika):
```jsx
const UserContext = createContext()
const CartContext = createContext()
const ThemeContext = createContext()
// 😩 Alag alag contexts, bikhra hua data
```

### Redux ke saath:
```jsx
const store = configureStore({
  reducer: {
    user: userReducer,
    cart: cartReducer,
    theme: themeReducer,
  }
})
// ✅ Sab ek jagah — ek store
```

### Flow:
```
Component
   ↓ dispatch(action)
Reducer (rules)
   ↓ state update
Store (data)
   ↓ useSelector
Component (re-render)
```

---

## 2. Store

> **Store = School ka office — sab ka record yahan hai**

### Setup:
```jsx
// store.js
import { configureStore } from '@reduxjs/toolkit'
import todoReducer from './features/todoSlice'
import userReducer from './features/userSlice'

export const store = configureStore({
  reducer: {
    todos: todoReducer,   // state.todos
    user: userReducer,    // state.user
  }
})
```

### Next.js mein Provider wrap karo:
```jsx
// app/Providers.jsx
"use client"

import { Provider } from 'react-redux'
import { store } from './store'

export default function Providers({ children }) {
  return (
    <Provider store={store}>
      {children}
    </Provider>
  )
}

// app/layout.js
import Providers from './Providers'

export default function Layout({ children }) {
  return (
    <html>
      <body>
        <Providers>
          {children}
        </Providers>
      </body>
    </html>
  )
}
```

> **Important:** Next.js mein `"use client"` Providers pe zaroor lagao!

---

## 3. createSlice + Reducers

> **createSlice = State + Rules ek jagah**
> **Reducer = Babu jo jaanta hai kaise data change karna hai**

```jsx
// features/todoSlice.js
import { createSlice } from '@reduxjs/toolkit'

const todoSlice = createSlice({
  name: 'todos',                          // slice ka naam

  initialState: {                         // shuru ki value
    list: [],
    loading: false,
    error: null
  },

  reducers: {
    // ADD
    addTodo: (state, action) => {
      state.list.push({
        id: Date.now(),
        text: action.payload,             // jo tune dispatch mein bheja
        done: false
      })
    },

    // REMOVE
    removeTodo: (state, action) => {
      state.list = state.list.filter(
        todo => todo.id !== action.payload // action.payload = id
      )
    },

    // UPDATE
    updateTodo: (state, action) => {
      const todo = state.list.find(
        todo => todo.id === action.payload.id
      )
      if (todo) {
        todo.text = action.payload.text   // sirf text badlo
      }
    },

    // TOGGLE
    toggleTodo: (state, action) => {
      const todo = state.list.find(
        todo => todo.id === action.payload
      )
      if (todo) {
        todo.done = !todo.done
      }
    },

    // LOCAL utility
    clearError: (state) => {
      state.error = null
    }
  }
})

// Actions export karo (useDispatch ke liye)
export const { addTodo, removeTodo, updateTodo, toggleTodo, clearError } = todoSlice.actions

// Reducer export karo (store ke liye)
export default todoSlice.reducer
```

### action.payload kya hota hai?
```jsx
dispatch(addTodo("Namaz padhni hai"))
//              ↑ ye action.payload hai

dispatch(removeTodo(123))
//                  ↑ ye action.payload hai

dispatch(updateTodo({ id: 123, text: "Naya text" }))
//                   ↑ ye action.payload hai (object)
```

---

## 4. useSelector

> **useSelector = Store se sirf apna kaam ka data lo**

```jsx
"use client"
import { useSelector } from 'react-redux'

function MyComponent() {
  // Poori list lo
  const todos = useSelector((state) => state.todos.list)

  // Sirf loading lo
  const loading = useSelector((state) => state.todos.loading)

  // Sirf error lo
  const error = useSelector((state) => state.todos.error)

  // Filter karke lo (sirf done wale)
  const doneTodos = useSelector(
    (state) => state.todos.list.filter(t => t.done)
  )

  return <div>{todos.length} todos hain</div>
}
```

> **Tip:** Sirf wo data lo jo component ko chahiye — poora state mat lo!

---

## 5. useDispatch

> **useDispatch = Store ko order dene wala haath**

```jsx
"use client"
import { useDispatch } from 'react-redux'
import { addTodo, removeTodo, updateTodo } from './todoSlice'

function TodoApp() {
  const dispatch = useDispatch()

  // ADD
  const handleAdd = () => {
    dispatch(addTodo("Kuch kaam karo"))
  }

  // REMOVE
  const handleRemove = (id) => {
    dispatch(removeTodo(id))
  }

  // UPDATE
  const handleUpdate = (id, newText) => {
    dispatch(updateTodo({ id, text: newText }))
  }

  return (
    <div>
      <button onClick={handleAdd}>Add</button>
      <button onClick={() => handleRemove(1)}>Remove</button>
      <button onClick={() => handleUpdate(1, "Updated!")}>Update</button>
    </div>
  )
}
```

---

## 6. createAsyncThunk

> **createAsyncThunk = Async kaam karne ka Redux ka tarika (API calls, DB)**

### Kab Use Karo?
```
Local state change  → Normal reducer (reducers:{})
API call + DB save  → createAsyncThunk
```

### Syntax:
```jsx
import { createAsyncThunk } from '@reduxjs/toolkit'

export const addTodoAsync = createAsyncThunk(
  'todos/addTodoAsync',       // unique name
  async (text) => {           // text = dispatch mein jo bheja
    const res = await fetch('/api/todos', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ text })
    })
    return res.json()         // ye fulfilled ka action.payload banega
  }
)
```

### Teen Auto States:
```
dispatch(addTodoAsync("text"))
         ↓
  pending   → API call shuru ⏳
         ↓
  fulfilled → Success ✅ (return wala data milta hai)
         ↓
  rejected  → Error ❌
```

### Complete Example:
```jsx
// features/todoThunks.js
import { createAsyncThunk } from '@reduxjs/toolkit'

// GET all todos
export const fetchTodos = createAsyncThunk(
  'todos/fetchTodos',
  async () => {
    const res = await fetch('/api/todos')
    return res.json()
  }
)

// POST new todo
export const addTodoAsync = createAsyncThunk(
  'todos/addTodoAsync',
  async (text) => {
    const res = await fetch('/api/todos', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ text })
    })
    return res.json()
  }
)

// DELETE todo
export const removeTodoAsync = createAsyncThunk(
  'todos/removeTodoAsync',
  async (id) => {
    await fetch('/api/todos', {
      method: 'DELETE',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ id })
    })
    return id   // id wapas bhejo — list se hatane ke liye
  }
)
```

### Error Handling:
```jsx
export const addTodoAsync = createAsyncThunk(
  'todos/addTodoAsync',
  async (text, { rejectWithValue }) => {  // rejectWithValue = custom error
    try {
      const res = await fetch('/api/todos', {
        method: 'POST',
        body: JSON.stringify({ text })
      })
      if (!res.ok) throw new Error('Server error')
      return res.json()
    } catch (error) {
      return rejectWithValue(error.message) // rejected mein milega
    }
  }
)
```

---

## 7. extraReducers

> **extraReducers = AsyncThunk ke teen states handle karne ki jagah**

```jsx
import { createSlice } from '@reduxjs/toolkit'
import { fetchTodos, addTodoAsync, removeTodoAsync } from './todoThunks'

const todoSlice = createSlice({
  name: 'todos',
  initialState: {
    list: [],
    loading: false,
    error: null
  },

  reducers: {
    clearError: (state) => { state.error = null }  // local utility
  },

  extraReducers: (builder) => {

    // ── FETCH ──────────────────────────────────
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.loading = true
        state.error = null
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.loading = false
        state.list = action.payload       // DB se aaye sab todos
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.loading = false
        state.error = action.payload || "Fetch nahi hua!"
      })

    // ── ADD ────────────────────────────────────
    builder
      .addCase(addTodoAsync.pending, (state) => {
        state.loading = true
      })
      .addCase(addTodoAsync.fulfilled, (state, action) => {
        state.loading = false
        state.list.push(action.payload)   // naya todo list mein
      })
      .addCase(addTodoAsync.rejected, (state, action) => {
        state.loading = false
        state.error = action.payload || "Add nahi hua!"
      })

    // ── REMOVE ─────────────────────────────────
    builder
      .addCase(removeTodoAsync.fulfilled, (state, action) => {
        state.list = state.list.filter(
          todo => todo._id !== action.payload
        )
      })
  }
})

export const { clearError } = todoSlice.actions
export default todoSlice.reducer
```

---

## 8. RTK Query

> **RTK Query = createAsyncThunk ka upgraded version**
> **Caching, refetching, loading states — sab automatic!**

### createAsyncThunk vs RTK Query:
```
createAsyncThunk          RTK Query
──────────────────────    ──────────────────────
Manual loading state      Automatic ✅
Manual caching            Automatic ✅
Zyada boilerplate         Kam code ✅
Full control              Convention-based ✅
```

### Setup:
```jsx
// features/todosApi.js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

export const todosApi = createApi({
  reducerPath: 'todosApi',              // store mein kahan save ho
  baseQuery: fetchBaseQuery({
    baseUrl: '/api'                     // base URL
  }),

  endpoints: (builder) => ({

    // GET — sab todos
    getTodos: builder.query({
      query: () => '/todos',
      providesTags: ['Todo']            // cache tag
    }),

    // POST — naya todo
    addTodo: builder.mutation({
      query: (text) => ({
        url: '/todos',
        method: 'POST',
        body: { text }
      }),
      invalidatesTags: ['Todo']         // cache clear karo
    }),

    // DELETE — todo hatao
    deleteTodo: builder.mutation({
      query: (id) => ({
        url: '/todos',
        method: 'DELETE',
        body: { id }
      }),
      invalidatesTags: ['Todo']
    }),
  })
})

// Auto-generated hooks
export const {
  useGetTodosQuery,
  useAddTodoMutation,
  useDeleteTodoMutation
} = todosApi
```

### Store mein add karo:
```jsx
// store.js
import { configureStore } from '@reduxjs/toolkit'
import { todosApi } from './features/todosApi'

export const store = configureStore({
  reducer: {
    [todosApi.reducerPath]: todosApi.reducer,  // RTK Query ka reducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(todosApi.middleware)  // zaruri hai!
})
```

### Component mein use karo:
```jsx
"use client"
import {
  useGetTodosQuery,
  useAddTodoMutation,
  useDeleteTodoMutation
} from './features/todosApi'

function TodoApp() {
  // GET — auto fetch, auto loading, auto caching
  const { data: todos, isLoading, error } = useGetTodosQuery()

  // MUTATIONS
  const [addTodo] = useAddTodoMutation()
  const [deleteTodo] = useDeleteTodoMutation()

  const handleAdd = async () => {
    await addTodo("Naya todo")     // call karo, baki RTK handle karega
  }

  if (isLoading) return <p>Loading...</p>
  if (error) return <p>Error aaya!</p>

  return (
    <div>
      <button onClick={handleAdd}>Add Todo</button>
      {todos?.map(todo => (
        <div key={todo._id}>
          {todo.text}
          <button onClick={() => deleteTodo(todo._id)}>Delete</button>
        </div>
      ))}
    </div>
  )
}
```

---

## 9. Middleware

> **Middleware = Har action ke beech mein kuch karna**
> Jaise: Logging, Authentication check, Error handling

```
dispatch(action)
      ↓
  Middleware 1   ← yahan kuch karo
      ↓
  Middleware 2   ← yahan bhi kuch karo
      ↓
  Reducer        ← phir reducer chalega
```

### Custom Middleware Example:
```jsx
// middleware/logger.js
const loggerMiddleware = (store) => (next) => (action) => {
  console.log('Action bheja:', action.type)
  console.log('Pehle state:', store.getState())

  const result = next(action)   // agle middleware ya reducer pe bhejo

  console.log('Baad ki state:', store.getState())
  return result
}

// store.js mein add karo
import { configureStore } from '@reduxjs/toolkit'
import { loggerMiddleware } from './middleware/logger'

export const store = configureStore({
  reducer: { todos: todoReducer },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(loggerMiddleware)
})
```

### RTK Query ka Middleware (zaruri hai):
```jsx
middleware: (getDefaultMiddleware) =>
  getDefaultMiddleware().concat(todosApi.middleware)
```

---

## 10. Immer

> **Immer = RTK andar se use karta hai — directly state mutate karne deta hai**
> Tu pehle se use kar raha hai — pata nahi tha! 😄

### Bina Immer (purana React tarika):
```jsx
// Naya array banana padta tha — original ko touch nahi kar sakte the
addTodo: (state, action) => {
  return { ...state, list: [...state.list, action.payload] }
}
```

### Immer ke saath (RTK ka tarika):
```jsx
// Seedha mutate karo — Immer andar se handle karta hai
addTodo: (state, action) => {
  state.list.push(action.payload)   // ← seedha push! Immer magic ✨
}

removeTodo: (state, action) => {
  state.list = state.list.filter(   // ← seedha assign!
    todo => todo.id !== action.payload
  )
}
```

> **Note:** Immer sirf RTK reducers ke andar kaam karta hai — bahar nahi!

---

## 11. Redux DevTools

> **DevTools = Browser mein Redux debug karne ka tool**

### Install:
```
Chrome/Firefox mein extension install karo:
"Redux DevTools Extension"
```

### RTK ke saath koi extra setup nahi:
```jsx
// configureStore automatically DevTools enable karta hai
export const store = configureStore({
  reducer: { todos: todoReducer }
  // DevTools automatic hai! ✅
})
```

### DevTools mein kya milta hai:
```
Action History    → Har action ka record
State Diff        → Pehle/baad ki state
Time Travel       → Purani state pe wapas jao
Action Replay     → Actions dobara chalao
```

---

## 12. Selectors / Reselect

> **Selectors = useSelector ko organized aur optimized rakhna**

### Basic Selectors (Alag File Mein):
```jsx
// features/todoSelectors.js

// Simple selectors
export const selectTodos = (state) => state.todos.list
export const selectLoading = (state) => state.todos.loading
export const selectError = (state) => state.todos.error

// Component mein:
const todos = useSelector(selectTodos)       // clean!
const loading = useSelector(selectLoading)
```

### Reselect — Memoized Selectors:
```jsx
// Jab heavy computation ho tab use karo
import { createSelector } from '@reduxjs/toolkit'

const selectTodos = (state) => state.todos.list

// Ye sirf tab recalculate hoga jab todos change hoga
export const selectDoneTodos = createSelector(
  [selectTodos],
  (todos) => todos.filter(todo => todo.done)    // ← heavy logic yahan
)

export const selectPendingTodos = createSelector(
  [selectTodos],
  (todos) => todos.filter(todo => !todo.done)
)

export const selectTodoCount = createSelector(
  [selectTodos],
  (todos) => todos.length
)

// Component mein:
const doneTodos = useSelector(selectDoneTodos)
const count = useSelector(selectTodoCount)
```

> **Kab use karo Reselect?** Jab selector mein filter/map/sort ho — performance ke liye!

---

## 13. Code Splitting

> **Code Splitting = Reducers ko lazily load karna — badi apps ke liye**

### Normal (Sab ek baar load):
```jsx
// store.js
import todoReducer from './todoSlice'
import userReducer from './userSlice'
import adminReducer from './adminSlice'   // ← admin page pe bhi load hoga

export const store = configureStore({
  reducer: { todos: todoReducer, user: userReducer, admin: adminReducer }
})
```

### Code Splitting (Zarurat pe load):
```jsx
// store.js
import { configureStore, combineReducers } from '@reduxjs/toolkit'
import todoReducer from './todoSlice'
import userReducer from './userSlice'

export const store = configureStore({
  reducer: {
    todos: todoReducer,
    user: userReducer,
    // admin wala yahan nahi — page pe jaane pe load hoga
  }
})

// Admin page component mein:
import { useEffect } from 'react'
import { store } from '@/store'
import adminReducer from '@/features/adminSlice'

export default function AdminPage() {
  useEffect(() => {
    // Sirf is page pe aane ke baad load hoga
    store.injectReducer('admin', adminReducer)
  }, [])

  return <div>Admin Panel</div>
}
```

> **Kab zarurat hai?** Jab app bohot badi ho — 10+ slices. Chhoti apps mein zarurat nahi!

---

## 14. Project File Structure

### Chhoti App:
```
src/
├── app/
│   ├── layout.js
│   ├── page.js
│   └── Providers.jsx       ← "use client" + Provider
├── store.js                ← configureStore
└── features/
    └── todoSlice.js        ← slice + thunks sab ek jagah
```

### Badi App (Recommended):
```
src/
├── app/
│   ├── layout.js
│   ├── page.js
│   └── Providers.jsx
├── store.js
├── middleware/
│   └── logger.js
└── features/
    ├── todo/
    │   ├── todoSlice.js        ← slice + extraReducers
    │   ├── todoThunks.js       ← asyncThunks
    │   ├── todoSelectors.js    ← selectors
    │   └── todosApi.js         ← RTK Query (agar use kar rahe ho)
    └── user/
        ├── userSlice.js
        ├── userThunks.js
        └── userSelectors.js
```

---

## 15. Quick Reference Cheatsheet

```
CONCEPT           KAB USE KARO                  CODE
──────────────────────────────────────────────────────────────────
store           → Ek baar setup              configureStore({reducer:{}})
createSlice     → State + rules              createSlice({name, initialState, reducers})
reducers        → Local/sync kaam            addTodo: (state, action) => {}
useSelector     → Data padhna                useSelector(state => state.todos.list)
useDispatch     → Action bhejana             dispatch(addTodo("text"))
asyncThunk      → API calls                  createAsyncThunk('name', async()=>{})
extraReducers   → Async handle karna         builder.addCase(thunk.fulfilled, ...)
RTK Query       → Data fetching (advanced)   createApi({endpoints:{}})
Middleware      → Actions intercept karna    store => next => action => {}
Immer           → Direct mutation (built-in) state.list.push(item)
DevTools        → Debugging                  Browser extension
Selectors       → Organized state access     (state) => state.slice.data
Reselect        → Memoized computation       createSelector([input], output)
Code Splitting  → Lazy reducers (big apps)   store.injectReducer()
```

---

## Complete Flow — Ek Nazar Mein

```
                    NEXT.JS APP
┌─────────────────────────────────────────────────┐
│                                                 │
│   Component                                     │
│      │                                          │
│   useDispatch ──→ dispatch(addTodoAsync(text))  │
│                          │                      │
│                   createAsyncThunk              │
│                          │                      │
│                   fetch('/api/todos')           │
│                          │                      │
└──────────────────────────┼──────────────────────┘
                           │ HTTP Request
┌──────────────────────────┼──────────────────────┐
│         NEXT.JS API      │                      │
│                          ↓                      │
│              app/api/todos/route.js             │
│                          │                      │
│                     MongoDB save               │
│                          │                      │
│                    Response wapas              │
└──────────────────────────┼──────────────────────┘
                           │ Data return
┌──────────────────────────┼──────────────────────┐
│                          ↓                      │
│             extraReducers.fulfilled             │
│                          │                      │
│                  state.list.push()              │
│                          │                      │
│             STORE updated ✅                    │
│                          │                      │
│   useSelector ←──────────┘                      │
│      │                                          │
│   Component re-render 🎉                        │
└─────────────────────────────────────────────────┘
```

---

*Documentation by: *Abdul Mateen* — Redux Toolkit Seekhte Seekhte* 😄
*Reference: Redux Toolkit Official Docs — redux-toolkit.js.org*
