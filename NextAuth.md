# Auth.js v5 for Next.js App Router

This document shows a production-style Auth.js setup in **JavaScript** for Next.js App Router. It covers:
- `app/auth.js`
- `app/api/auth/[...nextauth]/route.js`
- `proxy.js`
- Google provider
- Credentials provider
- JWT session strategy
- callbacks
- client sign-in/sign-out
- route protection
- server-side auth usage
- session usage in client components
- for-updating-any-field

---

## Folder structure

```txt
app/
  api/
    auth/
      [...nextauth]/
        route.js
  auth.js
  proxy.js
  layout.js
  page.js
components/
  providers.js
  sign-in-form.js
  sign-out-button.js
lib/
  db.js
```

---

## Install

```bash
pnpm add next-auth
```

If you use a database, also install your DB client and hashing library:
```bash
pnpm add bcryptjs
```

If you use Prisma:
```bash
pnpm add mongoose
```

---

## `app/auth.js`

This file holds the main Auth.js configuration and exports `handlers`, `auth`, `signIn`, and `signOut`.

```js
import NextAuth from "next-auth"
import Google from "next-auth/providers/google"
import Credentials from "next-auth/providers/credentials"
import { compare } from "bcryptjs"
import { db } from "@/lib/db"

export const {
  handlers,
  auth,
  signIn,
  signOut
} = NextAuth({
  secret: process.env.AUTH_SECRET,
  trustHost: true,

  session: {
    strategy: "jwt",
    maxAge: 30 * 24 * 60 * 60,
  },

  pages: {
    signIn: "/login",
    error: "/error",
  },

  providers: [
    Google({
      clientId: process.env.AUTH_GOOGLE_ID,
      clientSecret: process.env.AUTH_GOOGLE_SECRET,
    }),

    Credentials({
      name: "Credentials",
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" },
      },

      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) return null

        const user = await db.user.findUnique({
          where: { email: credentials.email },
        })

        if (!user) return null

        const isValid = await compare(credentials.password, user.password)
        if (!isValid) return null

        return {
          id: user.id,
          name: user.name,
          email: user.email,
          image: user.image,
          role: user.role,
        }
      },
    }),
  ],

  callbacks: {
    async signIn({ user, account, profile, email, credentials }) {
      return true
    },

    async jwt({ token, user, account }) {
      if (user) {
        token.id = user.id
        token.name = user.name
        token.email = user.email
        token.picture = user.image
        token.role = user.role
      }

      return token
    },

    async session({ session, token }) {
      if (session.user) {
        session.user.id = token.id
        session.user.name = token.name
        session.user.email = token.email
        session.user.image = token.picture
        session.user.role = token.role
      }

      return session
    },

    async redirect({ url, baseUrl }) {
      if (url.startsWith("/")) return `${baseUrl}${url}`
      if (new URL(url).origin === baseUrl) return url
      return baseUrl
    },
  },
})
```

---

## `app/api/auth/[...nextauth]/route.js`

This file exposes Auth.js handlers to Next.js routes.

```js
import { handlers } from "@/app/auth"

export const { GET, POST } = handlers
```

---

## `app/proxy.js`

This is your route protection file. It runs before matching requests and can redirect unauthenticated users.

```js
export { auth as proxy } from "@/app/auth"

export const config = {
  matcher: [
    "/dashboard/:path*",
    "/profile/:path*",
    "/seller/:path*",
    "/admin/:path*",
  ],
}
```

### What it does
- Protects private routes.
- Runs on matched routes only.
- Uses the built-in `auth` helper from Auth.js.

---

## If you want custom route checks

You can also write logic directly in `proxy.js` instead of re-exporting only.

```js
import { auth } from "@/app/auth"
import { NextResponse } from "next/server"

export default auth((req) => {
  const { nextUrl } = req
  const isLoggedIn = !!req.auth

  const isProtected =
    nextUrl.pathname.startsWith("/dashboard") ||
    nextUrl.pathname.startsWith("/profile") ||
    nextUrl.pathname.startsWith("/seller") ||
    nextUrl.pathname.startsWith("/admin")

  if (isProtected && !isLoggedIn) {
    return NextResponse.redirect(new URL("/login", nextUrl))
  }

  return NextResponse.next()
})

export const config = {
  matcher: [
    "/dashboard/:path*",
    "/profile/:path*",
    "/seller/:path*",
    "/admin/:path*",
  ],
}
```

---

## `components/providers.js`

Wrap your app with the session provider so client components can read session data.

```js
"use client"

import { SessionProvider } from "next-auth/react"

export default function Providers({ children }) {
  return <SessionProvider>{children}</SessionProvider>
}
```

---

## `app/layout.js`

```js
import Providers from "@/components/providers"
import "./globals.css"

export const metadata = {
  title: "My App",
  description: "Auth.js v5 setup",
}

export default function RootLayout({ children }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```

---

## `components/sign-in-form.js`

This is a simple client sign-in form using credentials.

```js
"use client"

import { signIn } from "next-auth/react"

export default function SignInForm() {
  const handleSubmit = async (formData) => {
    const email = formData.get("email")
    const password = formData.get("password")

    await signIn("credentials", {
      email,
      password,
      redirectTo: "/profile",
    })
  }

  return (
    <form action={handleSubmit}>
      <label htmlFor="email">
        Email
        <input id="email" name="email" type="email" />
      </label>

      <label htmlFor="password">
        Password
        <input id="password" name="password" type="password" />
      </label>

      <button type="submit">Sign In</button>
    </form>
  )
}
```

---

## `components/sign-out-button.js`

```js
"use client"

import { signOut } from "next-auth/react"

export default function SignOutButton() {
  return <button onClick={() => signOut({ callbackUrl: "/" })}>Sign Out</button>
}
```

---

## Client session usage

```js
"use client"

import { useSession } from "next-auth/react"

export default function Profile() {
  const { data: session, status } = useSession()

  if (status === "loading") return <p>Loading...</p>
  if (!session) return <p>Not logged in</p>

  return (
    <div>
      <p>{session.user?.name}</p>
      <p>{session.user?.email}</p>
      <p>{session.user?.role}</p>
    </div>
  )
}
```

---

## Server-side session usage

You can use `auth()` in server components or server actions.

```js
import { auth } from "@/app/auth"

export default async function DashboardPage() {
  const session = await auth()

  if (!session) {
    return <p>Unauthorized</p>
  }

  return <div>Welcome {session.user?.name}</div>
}
```

---

## Signup flow

Auth.js does not create signup automatically for Credentials. You create signup separately, then call `signIn("credentials", ...)` after user creation.

```js
"use client"

import { signIn } from "next-auth/react"

export default function SignupForm() {
  const handleSignup = async (formData) => {
    const name = formData.get("name")
    const email = formData.get("email")
    const password = formData.get("password")

    await fetch("/api/signup", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ name, email, password }),
    })

    await signIn("credentials", {
      email,
      password,
      redirectTo: "/dashboard",
    })
  }

  return (
    <form action={handleSignup}>
      <input name="name" placeholder="Name" />
      <input name="email" type="email" placeholder="Email" />
      <input name="password" type="password" placeholder="Password" />
      <button type="submit">Create account</button>
    </form>
  )
}
```

---

## Role-based protection

You can add role checks inside `proxy.js`.

```js
import { auth } from "@/app/auth"
import { NextResponse } from "next/server"

export default auth((req) => {
  const { nextUrl } = req
  const role = req.auth?.user?.role

  if (nextUrl.pathname.startsWith("/admin") && role !== "admin") {
    return NextResponse.redirect(new URL("/unauthorized", nextUrl))
  }

  return NextResponse.next()
})

export const config = {
  matcher: ["/admin/:path*"],
}
```

---

## Environment variables

```env
AUTH_SECRET=your-secret-key
AUTH_GOOGLE_ID=your-google-client-id
AUTH_GOOGLE_SECRET=your-google-client-secret
NEXTAUTH_URL=http://localhost:3000
```
# For Updating any field
```js
const {update}=useSession( );
```
```js
await update({isVerified:Boolean(true)})
```

```js
 async jwt({ token, user, trigger, session }) {
      if (user) {
        token.id = user.id;
        token.name = user.name;
        token.email = user.email;
        token.image = user.image;
        token.role = user.role;
        token.isVerified = Boolean(user.isVerified);
        token.isBlocked = Boolean(user.isBlocked);
      }
      if (trigger === "update" && session) {
        token.isVerified = Boolean(session.isVerified);
        token.isBlocked = Boolean(session.isBoolean);
        return token;
      }
      return token;
    },
```
---

## Common production notes

- Keep auth logic in `app/auth.js`.
- Use `proxy.js` for route protection.
- Use `SessionProvider` only in client provider wrapper.
- Use `auth()` on the server when possible.
- Use JWT strategy when you want stateless session handling.
- Use callbacks when you need custom fields like `id` and `role`.

---

## Roman Urdu summary

- `auth.js` mein Auth.js ka main config hota hai.
- `route.js` handlers expose karta hai.
- `proxy.js` routes protect karta hai.
- `SessionProvider` client components ke liye zaroori hota hai.
- `jwt` aur `session` callbacks custom fields session mein laate hain.
- Google aur Credentials dono support hotay hain.
- Signup alag flow hota hai; login session banata hai.
- Route protection `proxy.js` se ho sakti hai.

---

## Final note

This setup is a strong production baseline. You can extend it later for:
- refresh token handling,
- onboarding steps,
- seller profile completion,
- permissions,
- multi-role dashboards,
- and API route authorization.
