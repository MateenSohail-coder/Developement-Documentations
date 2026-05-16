# Next.js Configuration Files - Short Guide

## 1. .gitignore

### Purpose
GitHub par unnecessary ya secret files upload hone se rokta hai.

### Example
```bash
node_modules
.next
.env
```

### Use
- API keys safe
- repo clean
- unnecessary files ignore

---

## 2. postcss.config.mjs

### Purpose
CSS ko process aur optimize karta hai.

### Example
```js
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
    autoprefixer: {},
  },
};

export default config;
```

### Use
- Tailwind CSS
- browser compatibility
- CSS optimization

---

## 3. next.config.mjs

### Purpose
Next.js ki main settings file.

### Example
```js
const nextConfig = {
  reactStrictMode: true,
};

export default nextConfig;
```

### Use
- image optimization
- redirects
- environment variables
- performance settings

---

## 4. jsconfig.json

### Purpose
Clean imports aur VS Code support.

### Example
```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

### Use
```js
import Button from "@/components/Button";
```

---

## 5. eslint.config.mjs

### Purpose
Code errors aur bad practices detect karta hai.

### Example
```js
export default [
  {
    rules: {
      "no-console": "warn",
    },
  },
];
```

### Use
- clean code
- bug reduction
- better team collaboration

---

# Final Summary

| File | Kaam |
|---|---|
| .gitignore | unwanted files ignore |
| postcss.config.mjs | CSS optimize |
| next.config.mjs | Next.js settings |
| jsconfig.json | short imports |
| eslint.config.mjs | code checking |
