# ReCarbon — Frontend

A B2B marketplace where manufacturing companies list the industrial chemicals
they can supply and find what they need, matched by chemistry (CAS number +
semantic search) rather than keyword search alone.

Built with Next.js (App Router), React, TypeScript, Tailwind CSS v4, and
Zustand. Talks to a separate Node/Express + MongoDB + Pinecone backend
(`backend/reCarbon` in this monorepo) over a REST API.

## Tech stack

| Layer | Choice |
|---|---|
| Framework | Next.js 16 (App Router, Turbopack) |
| Language | TypeScript |
| Styling | Tailwind CSS v4 |
| Client state | Zustand (`persist` middleware for the auth token) |
| HTTP | Axios, via a shared `apiClient` instance |
| Icons | lucide-react |
| Fonts | Geist Sans (body), Space Grotesk (headings) — both via `next/font/google` |

## Features

- **Landing page** (`/`) — public marketing page: hero, feature bento grid,
  "how it works," and a nav that adapts to auth state.
- **Auth** — `/login` and `/register` against the backend's
  `/api/auth/login` and `/api/manufacturing-companies/register`. On login,
  the JWT + company id are persisted via the Zustand auth store
  (`localStorage`-backed).
- **Dashboard** (`/dashboard`) — natural-language search over
  `POST /api/search/selling-materials` (the backend extracts a CAS number
  and ranks sellers semantically); results are clickable and open a company
  details modal.
- **Feed** (`/feed`) — personalized recommendations from
  `GET /api/feed`, based on the logged-in company's own buy/sell interests.
  Previous/Next pagination (the endpoint has no total count, just
  `hasMore`).
- **Profile** (`/profile`) — the logged-in company's details plus its own
  selling and buying listings (from `GET /api/manufacturing-companies/me`),
  each deletable inline.
- **Sell / Buy modals** — forms to publish a selling listing
  (`POST /api/selling-materials`) or a buy request
  (`POST /api/buying-materials`), including a dynamic "add property" UI for
  arbitrary extra attributes beyond purity/quantity/unit, and an Indian
  states dropdown for location.
- **Route protection** — everything under the `(app)` route group
  (`dashboard`, `feed`, `profile`) redirects to `/login` if there's no
  persisted token.

## Project structure

```
app/
  page.tsx                     Landing page ("/")
  login/page.tsx                /login
  register/page.tsx             /register
  (app)/                        Auth-gated route group (shared sidebar layout)
    layout.tsx                  Redirects to /login if not authenticated
    dashboard/page.tsx          /dashboard — search
    feed/page.tsx                /feed — personalized recommendations
    profile/page.tsx            /profile — company details + own listings

  components/
    LoginComponent.tsx           Login form
    RegisterComponent.tsx        Registration form
    Sidebar.tsx                  Nav sidebar for the (app) route group
    LandingNav.tsx                Landing page header (auth-aware)
    SellProductModal.tsx         "Sell product" popup form
    BuyProductModal.tsx          "Buy product" popup form
    DynamicAttributes.tsx        Reusable add/remove key-value rows for both modals
    CompanyDetailsModal.tsx      Seller details popup (Dashboard + Feed)
    InlineDeleteButton.tsx       Inline delete-confirm control (Profile listings)

  store/
    authStore.ts                Zustand store: token, company profile, actions

  lib/
    apiClient.ts                 Axios instance, auto-attaches the Bearer token
    authStorage.ts               Reads the persisted token (shared by apiClient/store)
    indianStates.ts               States + union territories list
    units.ts                      Quantity unit options
```

## Getting started

Requires the backend (`backend/reCarbon`) running separately — see its own
README/`API.md` for setup (MongoDB, Pinecone, Ollama).

```bash
pnpm install
```

Create `.env.local`:

```bash
NEXT_PUBLIC_API_URL=http://localhost:5000
```

Run the dev server:

```bash
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000).

## Environment variables

| Variable | Description |
|---|---|
| `NEXT_PUBLIC_API_URL` | Base URL of the backend API (e.g. `http://localhost:5000`) |

## Auth flow

1. `/login` posts credentials to the backend, gets back `{ token, companyId }`,
   and saves them via `useAuthStore().setToken()`.
2. `apiClient` (an Axios instance) reads the persisted token on every request
   and attaches `Authorization: Bearer <token>` — endpoints that don't need
   auth just ignore the header.
3. The `(app)` layout waits for the Zustand store to hydrate from
   `localStorage`, then redirects to `/login` if there's no token.
4. `/profile` calls `GET /api/manufacturing-companies/me` and caches the
   result in the store, so navigating away and back doesn't refetch.

## Scripts

| Command | Description |
|---|---|
| `pnpm dev` | Start the dev server (Turbopack) |
| `pnpm build` | Production build |
| `pnpm start` | Run the production build |
| `pnpm lint` | ESLint |
