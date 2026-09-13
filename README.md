# reCarbon ♻️

**A B2B marketplace that turns captured CO₂ from a waste problem into a tradeable industrial resource.**

 **Live demo:** [hackout26-frontend.vercel.app](https://hackout26-frontend.vercel.app/)
 Built for **Hackout'26**

> Reduce. Reuse. Recycle. **Recarbon.**

---

##  The Idea

Capturing CO₂ is only half the problem — **the other half is finding someone who can use it.**

Cement plants, steel plants, and power plants capture significant quantities of CO₂, but finding a buyer for it is fragmented and manual. Meanwhile, industries doing carbon utilization — synthetic fuels, carbon-based building materials, greenhouses, algae farming — need a reliable, discoverable supply of CO₂ and other industrial chemicals.

**reCarbon connects these two sides**, so one industry's captured carbon can become another industry's raw material.

```
CAPTURE  →  LIST  →  MATCH  →  BID/BUY  →  MOVE  →  UTILIZE
```

### How it works

- **Suppliers** list what they can supply — chemical identity, quantity, purity, physical state, source location, and supply cadence.
- **Buyers** post what they need — material, specs, and delivery location.
- **AI-powered semantic search** lets a buyer describe their need in plain language instead of exact listing terms. Chemical identity is anchored to the **CAS number** (the canonical, unambiguous ID for any chemical, including CO₂), while embeddings capture the semantic context of the query.
- A **personalized feed** surfaces relevant supply/demand automatically, instead of requiring manual searching.
- A **logistics layer** lets transport providers register the pincodes they service, so the platform can identify who can actually move the material between supplier and buyer.

### Who uses it

| Role | What they do |
|---|---|
|  Carbon suppliers / emitters | List captured CO₂ (or other chemicals) available for reuse |
|  Carbon-utilization buyers | Post requirements and discover matching supply |
|  Logistics providers | Register serviceable pincodes to enable movement of material |

**Why start with a general chemical marketplace?** The CAS-number + semantic-matching engine works for *any* chemical — CO₂ included. Building it generally first means the core discovery/matching infrastructure is proven and reusable as the CO₂-specific vertical (pricing, bidding, tracking, carbon-impact reporting) is layered on top.

**Future layer:** using location + logistics data to estimate transport CO₂e and the net carbon benefit of a match — turning reCarbon from a connector into an impact-measurement tool. *(Not yet implemented.)*

---

##  How it's built

A monorepo with an independent frontend and backend.

```
hackout26-frontend/   Next.js (App Router) web app
reCarbon/              Node.js + Express + MongoDB API
```

| Layer | Tech |
|---|---|
| Frontend | Next.js 16, React 19, TypeScript, Tailwind CSS v4, Zustand |
| Backend | Node.js, Express, MongoDB (Mongoose) |
| AI / Search | Pinecone (vector search) · Ollama Cloud (`gpt-oss:120b-cloud` for CAS extraction) · local Ollama (`embeddinggemma` for embeddings) |
| Auth | JWT, bcrypt password hashing |

### What's implemented today

-  Manufacturing-company auth (register/login, JWT), and a separate logistics-company auth flow
-  Create/delete **selling** and **buying** material listings, keyed by CAS number (chemicals are auto-resolved/created — CAS is the single source of truth)
-  **Natural-language search**: query → LLM extracts CAS number → query embedded → Pinecone semantic search filtered by CAS → ranked, matching listings with seller + logistics info
-  Personalized feed of relevant listings, with bookmarking
-  Logistics companies can register/manage serviceable pincodes, surfaced alongside matching seller listings
-  Dashboards for manufacturing companies and logistics companies

### Search pipeline

```
Natural-language query
   ↓  LLM (CAS extraction)
Structured CAS number
   ↓  Local embedding model
Query embedding
   ↓  Pinecone (semantic search, hard-filtered by CAS)
Ranked vector matches
   ↓  MongoDB (hydrate full listing + company + logistics)
Ranked, explainable results
```

The LLM never invents results — it only parses intent. The actual match is always a deterministic database/vector lookup.

---

##  Running it locally

**Backend** (`reCarbon/`)

```bash
cd reCarbon
npm install
cp .env.example .env   # fill in Mongo, JWT, Pinecone, Ollama values
npm run dev            # http://localhost:5000
```

**Frontend** (`hackout26-frontend/`)

```bash
cd hackout26-frontend
pnpm install
echo "NEXT_PUBLIC_API_URL=http://localhost:5000" > .env.local
pnpm dev                # http://localhost:3000
```

Full API reference: [`reCarbon/API.md`](reCarbon/API.md) · [`hackout26-frontend/API.md`](hackout26-frontend/API.md)

---

##  Roadmap

The current build proves the core discovery loop (list → search → match). The fuller product vision — bidding, order/shipment lifecycle, QR-based chain of custody, live tracking, reliability scoring, and carbon-impact analytics — is scoped out in [`reCarbon/FUNCTIONAL_REQUIREMENTS.md`](reCarbon/FUNCTIONAL_REQUIREMENTS.md).
