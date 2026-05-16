# MongoDB Connection Setup (Production Ready)

## File: `lib/mongodb.js`

```javascript
import mongoose from "mongoose";

const MONGODB_URI =
  process.env.MONGODB_URI || "mongodb://127.0.0.1:27017/practice";

if (!MONGODB_URI) {
  throw new Error("Please define MONGODB_URI in environment variables");
}

let cached = global.mongoose;

if (!cached) {
  cached = global.mongoose = {
    conn: null,
    promise: null,
  };
}

export async function connectDB() {
  if (cached.conn) {
    return cached.conn;
  }

  if (!cached.promise) {
    cached.promise = mongoose.connect(MONGODB_URI, {
      dbName: "practice",
    });
  }

  cached.conn = await cached.promise;

  return cached.conn;
}
```

---

# Why This Approach?

## 1. Prevents Multiple Connections
In production and serverless environments like Next.js, hot reloads can create multiple MongoDB connections.

This caching pattern prevents that issue.

---

## 2. Cleaner Structure
Recommended folder structure:

```bash
src/
 ├── lib/
 │    └── mongodb.js
 ├── models/
 ├── app/
```

---

## 3. Usage Example

```javascript
import { connectDB } from "@/lib/mongodb";

export async function GET() {
  await connectDB();

  // your database logic here
}
```

---

## 4. Extra Tip

Optional:

```javascript
mongoose.set("strictQuery", true);
```

Useful in many production apps.

---

# Recommended Stack

- Node.js
- MongoDB
- Mongoose
- Next.js

