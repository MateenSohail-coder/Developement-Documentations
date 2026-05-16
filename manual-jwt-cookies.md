# JWT + Cookies Manual Setup Guide 🍪🔐

Ye documentation tumhare JWT aur Cookies authentication setup ko explain karti hai using `jose` package.

---

# Install Package

```bash
npm install jose
```

---

# Full Code

```javascript
import { jwtVerify, SignJWT } from "jose";

const secret = new TextEncoder().encode(process.env.JWT_SECRET);

// =======================
// SIGN TOKEN
// =======================

export async function signtoken(payload) {
  return await new SignJWT(payload)
    .setProtectedHeader({ alg: "HS256" })
    .setExpirationTime("10d")
    .sign(secret);
}

// =======================
// VERIFY TOKEN
// =======================

export async function verifytoken(token) {
  const { payload } = await jwtVerify(token, secret);
  return payload;
}

// =======================
// SET COOKIE
// =======================

export async function setcookies(token, response) {
  response.cookies.set("practice", token, {
    httpOnly: true,
    secure: false,
    sameSite: "lax",
    maxAge: 60 * 60 * 24 * 10,
    path: "/",
  });
}

// =======================
// GET TOKEN FROM COOKIE
// =======================

export async function gettoken(req) {
  return req.cookies.get("practice")?.value || null;
}
```

---

# JWT Kya Hota Hai? 🤔

JWT ka full form:

**JSON Web Token**

Ye ek secure token hota hai jo user ki identity ko store karta hai.

Mostly authentication systems me use hota hai.

---

# Authentication Flow 🔥

```text
1. User Login
        ↓
2. JWT Generate
        ↓
3. Cookie Me Save
        ↓
4. Har Request Me Verify
        ↓
5. User Authorized
```

---

# Secret Key

```javascript
const secret = new TextEncoder().encode(process.env.JWT_SECRET);
```

Ye line JWT secret ko encode karti hai.

## `.env` File

```env
JWT_SECRET=mysecretkey123
```

---

# Sign Token

```javascript
signtoken(payload)
```

Ye function JWT token create karta hai.

## Features

- User data store karta hai
- Secure token generate karta hai
- Expiry set karta hai
- Authentication handle karta hai

## Example

```javascript
const token = await signtoken({
  id: 1,
  email: "mateen@gmail.com",
});
```

---

# Protected Header

```javascript
.setProtectedHeader({ alg: "HS256" })
```

Ye define karta hai token kis algorithm se sign hoga.

## HS256

- Secure
- Fast
- Most common JWT algorithm

---

# Token Expiry

```javascript
.setExpirationTime("10d")
```

Token 10 days baad expire ho jayega.

## Examples

```javascript
"1h"   // 1 hour
"7d"   // 7 days
"30m"  // 30 minutes
```

---

# Verify Token

```javascript
verifytoken(token)
```

Ye token verify karta hai.

## Verify Kya Karta Hai?

- Token valid hai ya nahi
- Secret correct hai ya nahi
- Token expire hua ya nahi

Agar sab sahi ho to payload return hota hai.

---

# Cookies 🍪

Cookies browser me small data store karti hain.

Authentication systems me JWT mostly cookies me save hota hai.

---

# Set Cookies

```javascript
setcookies(token, response)
```

Ye browser me cookie save karta hai.

---

# Cookie Options

## httpOnly

```javascript
httpOnly: true
```

Browser JavaScript cookie access nahi kar sakta.

### Benefit

- XSS attacks se protection

---

## secure

```javascript
secure: false
```

Production me:

```javascript
secure: true
```

Use karo taake sirf HTTPS pe chale.

---

## sameSite

```javascript
sameSite: "lax"
```

CSRF attacks se protection deta hai.

---

## maxAge

```javascript
maxAge: 60 * 60 * 24 * 10
```

Cookie 10 days tak rahegi.

---

## path

```javascript
path: "/"
```

Pure website me accessible hogi.

---

# Get Token

```javascript
gettoken(req)
```

Browser se token read karta hai.

## Return

```javascript
return token
```

Agar token exist kare warna:

```javascript
null
```

---

# JWT Features 🚀

- Stateless Authentication
- Fast
- Secure
- Scalable
- No Session Database
- Easy API Authentication

---

# Cookies Features 🍪

- Browser me auto save hoti hain
- Har request ke sath auto jati hain
- Session management easy hota hai
- Secure authentication

---

# Security Tips 🔐

## Production Me

```javascript
secure: true
```

## Strong Secret Use Karo

```env
JWT_SECRET=superstrongsecretkey
```

## Short Expiry Better Hai

```javascript
.setExpirationTime("1d")
```

---

# Common Use Cases

✅ Login System  
✅ Protected Routes  
✅ Admin Panel  
✅ Middleware Auth  
✅ API Authentication  
✅ User Sessions

---

# Example Login

```javascript
const token = await signtoken({
  id: user.id,
  email: user.email,
});

await setcookies(token, response);
```

---

# Example Protected Route

```javascript
const token = await gettoken(req);

if (!token) {
  return "Unauthorized";
}

const user = await verifytoken(token);
```

---

# Final Summary 🧠

JWT user authentication handle karta hai.

Cookies token ko browser me securely store karti hain.

Dono milke modern authentication system banate hain.

---

# Tech Stack

- JWT
- JOSE Package
- Cookies
- Node.js
- Next.js
- Express.js

---

# Author Notes 😎

Ye approach modern full-stack applications me bohat zyada use hoti hai.

Simple + Secure + Professional Authentication Setup 🚀
