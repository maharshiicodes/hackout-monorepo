# reCarbon

reCarbon is a B2B chemical marketplace that lets manufacturing companies list
the industrial chemicals they can supply and find what they need — matched by
chemistry (CAS number + semantic search) rather than keyword search alone.

This is a monorepo with two independent projects:

```
frontend/          Next.js (App Router) web app
backend/reCarbon/  Node.js + Express + MongoDB API
```

See each project's own README for full setup details:
[`frontend/README.md`](frontend/README.md) ·
[`backend/reCarbon/README.md`](backend/reCarbon/README.md)

## Tech stack

| Layer | Choice |
|---|---|
| Frontend | Next.js 16, React, TypeScript, Tailwind CSS v4, Zustand |
| Backend | Node.js, Express, MongoDB (Mongoose) |
| Search | Pinecone (vector search) + Ollama (embeddings and LLM CAS extraction) |
| Auth | JWT |

## Project structure

```
frontend/           Next.js app (UI, auth, dashboard, feed, profile)
backend/reCarbon/   Express API (models, controllers, routes, services)
  ├── src/
  ├── infra/terraform/   AWS EC2 deployment (Terraform)
  └── Dockerfile
```

## Getting started

Run the backend and frontend in separate terminals — the frontend expects
the backend to already be up.

### 1. Backend (`backend/reCarbon`)

```bash
cd backend/reCarbon
npm install
cp .env.example .env   # then fill in Mongo/Pinecone/Ollama values
npm run dev
```

Runs on `http://localhost:5000` by default. See
[`backend/reCarbon/README.md`](backend/reCarbon/README.md) and
[`backend/reCarbon/API.md`](backend/reCarbon/API.md) for full API docs and
required environment variables (MongoDB, JWT, Pinecone, Ollama).

### 2. Frontend (`frontend`)

```bash
cd frontend
pnpm install
```

Create `frontend/.env.local`:

```bash
NEXT_PUBLIC_API_URL=http://localhost:5000
```

```bash
pnpm dev
```

Runs on `http://localhost:3000`. See
[`frontend/README.md`](frontend/README.md) for project structure and
feature details.

## Deployment

Backend infra (AWS EC2 via Terraform) and a Dockerfile live under
`backend/reCarbon/infra/` and `backend/reCarbon/Dockerfile`.
