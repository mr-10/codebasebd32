# Next.js / Vercel Build Error Fix: Parallel Pages Resolving to the Same Path

## Error Summary

During deployment on Vercel or local build (`npm run build`), Next.js / Turbopack produces the following error:

```text
Error: Turbopack build failed with 1 error:
./app/login
Error: You cannot have two parallel pages that resolve to the same path. Please check /(auth)...
```

---

## Root Cause Analysis

In the Next.js **App Router**, folder names enclosed in parentheses (e.g., `(auth)`, `(main)`, `(dashboard)`) are **Route Groups**. Route groups allow you to organize files and layouts without affecting the URL path structure.

Because route groups do not add a prefix to the URL, any `page.tsx` (or `.jsx`, `.js`, `.ts`) inside a route group resolves directly to the relative subfolder path.

The error occurs when you have **two or more `page` files that resolve to the exact same URL path**.

For example, both of the following files map to `/login`:
1. `app/login/page.tsx` $\rightarrow$ resolves to `/login`
2. `app/(auth)/login/page.tsx` $\rightarrow$ resolves to `/login`

Because Next.js cannot determine which page component to serve at `/login`, the build fails.

---

## How to Fix

### Step 1: Locate Duplicate Login Pages
Check your `app` directory for multiple `login` page definitions:
- Check root level: `app/login/page.tsx`
- Check route groups:
  - `app/(auth)/login/page.tsx`
  - `app/(main)/login/page.tsx`
  - `app/(public)/login/page.tsx`

---

### Step 2: Choose One of the Following Solutions

#### Solution A: Delete the Duplicate File (Recommended)
If you intended to move `login` into the `(auth)` group, delete the old `app/login` directory:

```bash
# Keep app/(auth)/login/page.tsx and delete app/login:
rm -rf app/login
```

Or if you prefer keeping `app/login/page.tsx`, delete the `(auth)/login` route:

```bash
rm -rf "app/(auth)/login"
```

---

#### Solution B: Move/Rename One of the Routes
If both pages serve different purposes, rename one of them to have a unique URL path (e.g., `/admin-login` or `/signin`):

- `app/(auth)/login/page.tsx` $\rightarrow$ `/login`
- `app/(auth)/admin-login/page.tsx` $\rightarrow$ `/admin-login`

---

### Step 3: Verify the Fix Locally
Run the build command locally before re-deploying to Vercel:

```bash
npm run build
```

If the build succeeds without route conflict errors, commit and push your changes to trigger a new Vercel deployment.
