---
description: Ruleset for Next.js and React development. Strictly enforces App Router paradigms, React Server Components (RSC) by default, Server Actions for mutations, and strict TypeScript.
globs: "**/*.{ts,tsx}"
---

# Next.js & React Development Rules

**Core Goal:** Generate modern Next.js code using the App Router. Maximize performance by keeping code on the server by default and pushing client-side interactivity to the leaves of the component tree.

## 1. Core Principles & Architecture
1. **App Router Only:** All routing and layouts MUST use the `app/` directory conventions (`page.tsx`, `layout.tsx`, `loading.tsx`, etc.). The `pages/` directory is strictly forbidden.
2. **TypeScript Strictness:** All components, functions, and Server Actions MUST have strictly typed inputs and outputs using interfaces or types. The use of `any` is strictly prohibited.
3. **Tailwind CSS:** All styling must be done using Tailwind CSS utility classes. Avoid custom CSS files or inline styles unless absolutely necessary for dynamic calculations.

## 2. Server vs. Client Components (The Boundary)
By default, every component is a React Server Component (RSC).

- **When to use `"use client"`:** ONLY add the `"use client"` directive at the top of a file if the component explicitly requires:
    - React hooks (`useState`, `useEffect`, `useRef`, `useReducer`, `useActionState`).
    - Browser APIs (e.g., `window`, `document`, `localStorage`).
    - Event listeners (`onClick`, `onChange`).
- **Push the Boundary Down:** Keep Client Components as small as possible and push them to the leaves of the component tree. 
    - ❌ **WRONG:** Making an entire `page.tsx` a Client Component just because it contains one interactive button.
    - ✅ **CORRECT:** Extracting the button into a separate `InteractiveButton.tsx` (with `"use client"`) and importing it into the Server Component `page.tsx`.



## 3. Data Fetching & Mutations
Vercel best practices dictate abandoning traditional API routes for standard operations in favor of Server Actions.

- **Data Fetching (READ):** Fetch data directly inside Server Components using async/await. Do not use `useEffect` for data fetching.
    - ✅ **CORRECT:** `export default async function Page() { const data = await db.query(); return <div>{data}</div>; }`
- **Mutations (CREATE/UPDATE/DELETE):** ALWAYS use **Server Actions** for data mutations (e.g., form submissions).
    - Server Actions MUST reside in a separate file (e.g., `actions.ts`) with the `"use server"` directive at the top.
    - Use `useActionState` (or `useFormState` in older React versions) and `useFormStatus` inside Client Components to handle loading states and responses from Server Actions.
    - ❌ **WRONG:** Creating a file in `app/api/route.ts` just to handle a basic form submission.

## 4. UI/UX and Performance
1. **Suspense & Streaming:** Wrap slow asynchronous data fetching components in `<Suspense fallback={<Skeleton />}>` to stream the UI to the user instantly.
2. **Next.js Native Components:** - ALWAYS use `next/image` (`<Image />`) instead of standard `<img>` tags.
    - ALWAYS use `next/link` (`<Link />`) instead of standard `<a>` tags for internal navigation.

## 5. Directory Structure Reference
```text
app/
├── (routes)/           # Route groups for logical separation (e.g., (auth), (dashboard))
│   └── page.tsx        # Server Component by default
├── actions/            # Reusable Server Actions ("use server")
├── components/         # UI Components
│   ├── server/         # Pure UI or async data components
│   └── client/         # Interactive components ("use client" here only)
└── lib/                # Utilities, database configs, types
```

---

### ✅ LLM Pre-Generation Checklist

Before finalizing any Next.js / React code, verify:

1. Does `"use client"` appear on components that do NOT use hooks or browser APIs? Remove it.
2. Are mutations using Server Actions, not `app/api/route.ts`?
3. Are Server Actions in a separate file (e.g., `actions/[feature].ts`) with `"use server"` at the top?
4. Is `any` used anywhere? Remove it — use proper TypeScript types.
5. Are `<img>` tags present? Replace with `<Image />` from `next/image`.
6. Are `<a>` tags used for internal navigation? Replace with `<Link />` from `next/link`.
7. Are components that fetch slow async data wrapped in `<Suspense fallback={...}>`?
8. Is any Client Component larger than necessary? Can the `"use client"` boundary be pushed further down the tree?