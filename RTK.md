# RTK Query — Guide (Roman Urdu + Code)

> RTK Query Redux Toolkit ka built-in powerful data fetching system hai.  
> Isse:
> - API calls easy ho jaati hain
> - Caching automatic hoti hai
> - Loading/Error handling easy ho jaati hai
> - Optimistic updates possible hote hain
> - Boilerplate bohot kam ho jaata hai

---

## ⚙️ Step 1 — Store Setup

File: `store/store.js`
```js
import { configureStore } from '@reduxjs/toolkit'
import { todosApi } from '@/features/todos/todosApi'

export const store = configureStore({
  reducer: {
    [todosApi.reducerPath]: todosApi.reducer,
  },

  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(todosApi.middleware)
})
```

Roman Urdu Explanation
- reducer  
  Yahan RTK Query ka reducer register hota hai.
- middleware  
  RTK Query ka middleware caching, refetch, invalidation, optimistic updates wagaira handle karta hai.

---

## ⚙️ Step 2 — Provider Setup

File: `app/Providers.jsx`
```jsx
"use client"

import { Provider } from 'react-redux'
import { store } from '@/store/store'

export default function Providers({ children }) {
  return (
    <Provider store={store}>
      {children}
    </Provider>
  )
}
```

Roman Urdu Explanation
- Redux store ko poori app mein available banane ke liye Provider use hota hai.

---

## ⚙️ Step 3 — Layout Setup

File: `app/layout.js`
```jsx
import Providers from './Providers'

export default function Layout({ children }) {
  return (
    <html lang="en">
      <body>
        <Providers>
          {children}
        </Providers>
      </body>
    </html>
  )
}
```

Roman Urdu Explanation
- Next.js App Router mein layout root wrapper hota hai.  
- Yahan hum poori app ko Redux Provider ke andar wrap kar rahe hain.

---

## ⚙️ Step 4 — API Setup

File: `features/todos/todosApi.js`
```js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

export const todosApi = createApi({
  reducerPath: 'todosApi',

  baseQuery: fetchBaseQuery({
    baseUrl: '/api'
  }),

  tagTypes: ['Todo'],

  endpoints: (builder) => ({

    // ───────────────── GET TODOS ─────────────────
    getTodos: builder.query({
      query: () => '/todos',

      providesTags: ['Todo']
    }),

    // ───────────────── ADD TODO ─────────────────
    addTodo: builder.mutation({

      query: (text) => ({
        url: '/todos',
        method: 'POST',

        headers: {
          'Content-Type': 'application/json'
        },

        body: JSON.stringify({ text })
      }),

      async onQueryStarted(text, { dispatch, queryFulfilled }) {

        // Optimistic Update
        const patchResult = dispatch(
          todosApi.util.updateQueryData(
            'getTodos',
            undefined,
            (draft) => {
              draft.push({
                _id: 'temp_id',
                text
              })
            }
          )
        )

        try {

          // Real API Response
          const { data: newTodo } = await queryFulfilled

          // Fake ko real se replace karo
          dispatch(
            todosApi.util.updateQueryData(
              'getTodos',
              undefined,
              (draft) => {

                const index = draft.findIndex(
                  t => t._id === 'temp_id'
                )

                if (index !== -1) {
                  draft[index] = newTodo
                }
              }
            )
          )

        } catch {

          // Error aaye to rollback
          patchResult.undo()
        }
      },

      invalidatesTags: ['Todo']
    }),

    // ───────────────── DELETE TODO ─────────────────
    deleteTodo: builder.mutation({

      query: (id) => ({
        url: '/todos',
        method: 'DELETE',

        headers: {
          'Content-Type': 'application/json'
        },

        body: JSON.stringify({ id })
      }),

      async onQueryStarted(id, { dispatch, queryFulfilled }) {

        const patchResult = dispatch(
          todosApi.util.updateQueryData(
            'getTodos',
            undefined,
            (draft) => {

              const index = draft.findIndex(
                todo => todo._id === id
              )

              if (index !== -1) {
                draft.splice(index, 1)
              }
            }
          )
        )

        try {
          await queryFulfilled
        } catch {
          patchResult.undo()
        }
      },

      invalidatesTags: ['Todo']
    }),

    // ───────────────── UPDATE TODO ─────────────────
    updateTodo: builder.mutation({

      query: ({ id, text }) => ({
        url: '/todos',
        method: 'PUT',

        headers: {
          'Content-Type': 'application/json'
        },

        body: JSON.stringify({ id, text })
      }),

      async onQueryStarted(
        { id, text },
        { dispatch, queryFulfilled }
      ) {

        const patchResult = dispatch(
          todosApi.util.updateQueryData(
            'getTodos',
            undefined,
            (draft) => {

              const todo = draft.find(
                t => t._id === id
              )

              if (todo) {
                todo.text = text
              }
            }
          )
        )

        try {
          await queryFulfilled
        } catch {
          patchResult.undo()
        }
      },

      invalidatesTags: ['Todo']
    })

  })
})

export const {
  useGetTodosQuery,
  useAddTodoMutation,
  useDeleteTodoMutation,
  useUpdateTodoMutation
} = todosApi
```

Roman Urdu Explanation
- createApi()  
  Ye RTK Query ka main function hai. Yahan saari API endpoints define hoti hain.
- fetchBaseQuery()  
  Ye axios/fetch ka lightweight wrapper hai.
- tagTypes  
  Caching aur auto refetch ke liye use hota hai.
- providesTags  
  GET request kis tag ko provide karegi.
- invalidatesTags  
  Mutation ke baad kaunsi query refetch hogi.

---

## ⚡ Optimistic Updates Kya Hoti Hain?

- UI ko instantly update karna BEFORE server response aaye.
- User ko fast feel hota hai.

---

## ⚡ Fake ID Problem

Problem:
- Optimistic update mein aap temporary ID use karte ho, e.g. `_id: 'temp_id'`.
- Jab server real MongoDB ID return karta hai, temp ID aur real ID mismatch ho sakta hai.
- Isliye delete/update operations fail lag sakte hain (because they refer to the wrong id).

Solution:
- Jab real API response aaye, optimistic item ko replace karo with the real item:
  - Use `todosApi.util.updateQueryData` and assign `draft[index] = newTodo` to replace the fake todo with the real one.
- This ensures future delete/update actions use the correct `_id`.

---

## ⚙️ Step 5 — Todo Component

File: `components/TodoApp.jsx`
```jsx
"use client"

import { useState } from "react"

import {
  useGetTodosQuery,
  useAddTodoMutation,
  useDeleteTodoMutation,
  useUpdateTodoMutation
} from "@/features/todos/todosApi"

export default function TodoApp() {

  const [text, setText] = useState("")
  const [editId, setEditId] = useState(null)
  const [editText, setEditText] = useState("")

  // Hooks
  const {
    data: todos = [],
    isLoading,
    error
  } = useGetTodosQuery()

  const [addTodo] = useAddTodoMutation()
  const [deleteTodo] = useDeleteTodoMutation()
  const [updateTodo] = useUpdateTodoMutation()

  // Add
  const handleAdd = async () => {

    if (!text.trim()) return

    await addTodo(text)

    setText("")
  }

  // Delete
  const handleDelete = async (id) => {
    await deleteTodo(id)
  }

  // Update
  const handleUpdate = async (id) => {

    if (!editText.trim()) return

    await updateTodo({
      id,
      text: editText
    })

    setEditId(null)
  }

  // Loading
  if (isLoading) {
    return <p>Loading...</p>
  }

  // Error
  if (error) {
    return <p>Error aaya!</p>
  }

  return (
    <div>

      {/* Add Todo */}
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
      />

      <button onClick={handleAdd}>
        Add
      </button>

      {/* Todo List */}
      {todos.map((todo) => (

        <div key={todo._id}>

          {editId === todo._id ? (
            <>
              <input
                value={editText}
                onChange={(e) =>
                  setEditText(e.target.value)
                }
              />

              <button
                onClick={() =>
                  handleUpdate(todo._id)
                }
              >
                Save
              </button>
            </>
          ) : (
            <>
              <p>{todo.text}</p>

              <button
                onClick={() => {
                  setEditId(todo._id)
                  setEditText(todo.text)
                }}
              >
                Edit
              </button>

              <button
                onClick={() =>
                  handleDelete(todo._id)
                }
              >
                Delete
              </button>
            </>
          )}

        </div>
      ))}

    </div>
  )
}
```

Roman Urdu Explanation
- useGetTodosQuery() — Automatic GET request karta hai.
- useAddTodoMutation() — POST request ke liye use hota hai.
- useDeleteTodoMutation() — DELETE request ke liye.
- useUpdateTodoMutation() — PUT/PATCH request ke liye.

---

## ⚡ RTK Query Ke Major Benefits

- Automatic caching
- Auto refetching
- Optimistic updates
- Loading/Error states built-in
- Boilerplate kam
- Redux DevTools support
- Performance fast

---

## 🔥 Final Summary

Fake ID Problem
- Problem: Optimistic update mein `_id: 'temp_id'` aata hai. Lekin MongoDB real ID deta hai, jis se delete/update mismatch ho jaata hai.
- Fix: API response ke baad `draft[index] = newTodo` karke fake ko real se replace karo.

Result
- Smooth UI
- Instant updates
- No delete/update bugs
- Professional RTK Query setup

---

## 🎯 Recommended Improvements (Future)

- Pagination
- Infinite Scroll
- Authentication
- Refresh Tokens
- WebSockets
- Search
- Debouncing
- Separate API slices
- SSR Support
- Persist Redux

---

Done! Ab bas copy paste karo aur project mein use karo 🚀
