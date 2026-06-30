# CLAUDE.md — fronted-cueva

Next.js 16 (App Router, `src/`) + TypeScript frontend template for technical
interviews (Cueva Tech context: fintech / prop-trading / EdTech domain). Consumes
the companion `backend-cueva` NestJS API over HTTP. **All code and comments in English.**

## Commands

```bash
npm run dev            # next dev (start backend first + create .env.local)
npm run dev -- -p 3001 # dev on port 3001 to avoid clashing with the backend
npm run build          # next build
npm start              # serve the production build
npm run lint           # eslint (report only)
npm run lint:fix       # eslint --fix (auto-fix)
npm run format         # prettier --write src/
npm run format:check   # prettier check
```

Env: create `.env.local` with `NEXT_PUBLIC_API_URL=http://localhost:3000` before
running `npm run dev`.

## Stack (currently installed)

Next.js 16 · React 19 · TypeScript 5 · Tailwind CSS v4 · Zod v4 ·
**TanStack Query v5** · **shadcn/ui** · ESLint · Prettier.

> **Note on Zod v4:** API is compatible with v3 for common patterns (`z.object`,
> `z.string`, `.safeParse`, `.parse`). Error message internals differ — use
> `result.error.issues` rather than `.errors`.

> **Note on TanStack Query v5:** wrap the app in `<QueryClientProvider>` (see
> `src/lib/query-client.ts`). Import from `@tanstack/react-query`. Use `useQuery`
> for reads and `useMutation` for writes. DevTools available via
> `@tanstack/react-query-devtools` (already installed).

> **shadcn/ui components installed:** `button` · `input` · `label` · `card` ·
> `badge` · `select` · `textarea` · `sonner` (toasts). Add more with
> `npx shadcn@latest add <component>`. Components live in `src/components/ui/`.

**Add as needed during a feature:**
```bash
npx shadcn@latest add <component>   # dialog, table, dropdown-menu, form, tabs…
npm install lucide-react             # icon set (pairs well with shadcn)
npm install framer-motion            # animations
npm install react-hook-form          # only for complex multi-step forms;
                                     # prefer useActionState + Server Actions otherwise
```

## After every code change

Run `npm run lint:fix` to auto-fix style issues, then verify `npm run build` passes
with zero errors before reporting a task as complete.

## Project conventions

- **Folder layout:** `src/app/` (routes), `src/components/<feature>/`,
  `src/hooks/`, `src/lib/`, `src/context/`, `src/types/`. UI primitives from
  shadcn (if added) live in `src/components/ui/`.
- **Server vs Client components:** default to **Server Components**. Add `'use client'`
  only when a component needs interactivity, hooks, or browser APIs. Keep client
  components small and leaf-ward (push state down).
- **Data fetching:** prefer RSC + `async/await` for the initial load. For client-side
  mutations and live data use SWR or TanStack Query — do not hand-roll loading/error
  state where a query hook fits. Never call `fetch` directly inside components;
  route through a typed client in `src/lib/api.ts`.
- **Server Actions** for form mutations: validate with Zod inside the action, treat
  the action like a public endpoint (authenticate + validate every call), and
  `revalidatePath` / `revalidateTag` after writes.
- **Money = integer cents.** Keep cents end-to-end; convert with helpers in
  `src/lib/money.ts` only at display. Never use floats for money.
- **Paginated responses** are `{ data, meta }` with
  `meta = { total, page, limit, totalPages, hasNextPage }` — mirror the backend shape
  in `src/types/api.ts`.
- **Auth:** store JWT in `localStorage` (or httpOnly cookie via middleware for better
  security); own it through a context/hook (`src/context/auth.tsx`). Protected routes
  redirect unauthenticated users to `/login?next=…`.
- **Styling:** use Tailwind utility classes. If a design token system is set up
  (CSS variables in `globals.css`), use those tokens — no one-off hex values in
  components.
- **Caching (Next.js 15/16 model):** `fetch()` is **not** cached by default. Use
  `{ cache: 'force-cache' }` to opt in, or `export const dynamic = 'force-static'`
  for GET route handlers. Always be explicit; never rely on implicit caching.

## TypeScript & React coding guidelines

### Naming

- **Classes / Interfaces / Types / Enums / React components**: `UpperCamelCase`.
- **Variables / functions / hooks / methods**: `camelCase`; verb-based names. Hooks
  start with `use`.
- **Module-level constants**: `UPPER_SNAKE_CASE`.
- **Booleans**: start with `is`, `has`, `can`, or `should`.
- **Event handlers**: local handlers start with `handle` (`handleSubmit`); prop
  callbacks start with `on` (`onSelect`).

### Type safety

- All function/component params have explicit types; typed `Promise<T>` returns.
- **Avoid `any`** — use a specific type, `unknown`, or a union.
- Strict equality (`===` / `!==`) only. Wrap magic strings/numbers in named constants.
- **Exported domain/API types live in `src/types/`** — not redeclared inline. Local,
  non-exported prop types may stay co-located with the component.

### Components & functions

- Function components only. One primary component per file.
- Keep render logic declarative; extract non-trivial logic into hooks (`src/hooks/`)
  or pure helpers in `src/lib/`.
- Max 3–4 props; use a props object beyond that. Return early to reduce nesting.
- Memoize (`useMemo`/`useCallback`/`memo`) only where measurably beneficial — not
  by reflex.

### Self-documenting code

- Descriptive names over comments. **Avoid JSDoc.** Keep only short `//` notes for
  non-obvious *why*. No `m`/`p`/underscore member prefixes.
- **No `console.log`** used for temporary debugging in committed code.
- **No dead code** — remove unused exports, imports, and props after any refactor.

### Async & errors

- `async/await` over promise chains. Surface API failures through typed error objects
  (never raw thrown strings). Show errors via toasts or inline field messages.

## Accessibility & performance

- Real `<label>`s, `aria-invalid` + `aria-describedby` on form errors, visible
  `focus-visible` rings, keyboard-operable controls.
- `next/image` for images, `next/font` for fonts (no layout shift).
- Dynamically import heavy or rarely-used code (`next/dynamic`).
- Mobile-first responsive layout.

## Testing

- Component/unit tests with Jest + React Testing Library (add if needed:
  `npm install --save-dev jest @testing-library/react @testing-library/jest-dom`).
- Test pure helpers (`src/lib/`) and key components; keep assertions non-vacuous.

## Commits

Frequent, clear, conventional commit messages: `feat: challenge purchase form`,
`fix: auth redirect loop`, `chore: add eslint rule`. DRY, YAGNI.

## Skills, agents & commands

Invoke these before starting any substantial task — never wait until after writing code.

| When | Use |
|---|---|
| Starting a substantial feature (>~500 LOC / cross-cutting / ambiguous) | `spec-driven-implementation` skill — write PRODUCT.md/TECH.md first |
| Before any creative/feature work — explore intent & design | `superpowers:brainstorming` skill |
| Building, reviewing, or optimizing any React/Next.js UI | `react-nextjs-agent` agent |
| Next.js App Router, RSC, Server Actions, or caching specifics | `next-best-practices` + `vercel-react-best-practices` skills |
| Health-scan an existing React component for issues | `react-doctor` skill |
| Adding or using shadcn/ui components | `shadcn` skill |
| Tailwind tokens, design-system, or utility-class decisions | `tailwind-design-system` skill |
| Visual/UX design quality, layout, accessibility | `frontend-design` / `ui-ux-pro-max` / `web-design-guidelines` skills |
| Animation performance issues | `fixing-motion-performance` skill |
| Writing or healing E2E browser tests | `playwright-cli` skill |
| Advanced TypeScript (generics, conditional types, utility types) | `typescript-advanced-types` skill |
| Domain/business logic that should be test-first | `superpowers:test-driven-development` skill |
| Debugging a non-obvious bug | `superpowers:systematic-debugging` skill |
| Reviewing the current diff for bugs | `/code-review` command |
| Cleanup: reuse / simplify / efficiency on changed code | `/simplify` command |
| Confirm a change works by running the app | `/verify` or `/run` command |
| Need current library/framework docs (Next.js, React, TanStack Query, shadcn…) | `find-docs` skill or `ctx7 docs <library-id>` |
