# Technology Stack — justType

**Document Version:** 1.0  
**Architecture:** Decoupled Local-First Monorepo (`client/` Next.js + `server/` Express)  
**Primary Language:** TypeScript (strict mode across frontend and backend)  

---

## 1. Architecture Overview

`justType` uses a **decoupled client-server architecture** where the frontend operates 100% independently for core typing and local progress, while the Express backend handles optional authentication, MongoDB cloud synchronization, and on-demand AI drill generation.

```text
[ Next.js Client (Browser) ]
   ├── Pure TS Typing Engine (Zero-latency, 100% client-side)
   ├── IndexedDB + localStorage (Anonymous history, calendar, settings)
   └── REST API Client (Optional: Auth, /api/sync, /api/generate-drill)
            │
            ▼  HTTPS (JSON + HttpOnly JWT Cookies)
[ Express + TypeScript Server ]
   ├── Zod Validation, Rate Limiting, Helmet, CORS
   ├── MongoDB + Mongoose (Users, TypingTests, DailyActivity)
   └── LLM API Client (Phase 6: Gemini Flash for targeted weak-key drills)
```

---

## 2. Frontend Stack (`client/`)

| Layer | Technology | Purpose & Rationale |
| --- | --- | --- |
| **Framework** | **Next.js (App Router)** | Fast initial paint via Server Components for static shell/navigation; `'use client'` boundary isolated to the typing arena, charts, and local storage. |
| **Language** | **TypeScript (`.tsx`)** | Strict typing for typing engine state machines, telemetry payloads, and IndexedDB schemas. |
| **Styling** | **Tailwind CSS** | Enforces the strict 4-point grid (`4px` multiples) and CSS variable theming (`Warm Retro` default + `Tokyo Night` dark mode) with zero runtime overhead. |
| **Typography** | **`next/font/google`** | Self-hosts `IBM Plex Mono` (`--font-mono`) and `IBM Plex Sans` (`--font-sans`) with zero layout shift (no Inter). |
| **Local Persistence** | **`idb` (IndexedDB) + `localStorage`** | `IndexedDB` stores test history, character-category stats, and cached AI drills; `localStorage` stores UI settings, theme, and recent exercise deduplication queue. |
| **Charts** | **Recharts** | Renders flat, minimal WPM-over-time and accuracy progression charts (no gradient fills or glows). |
| **State Management** | **React Hooks + `useRef`** | Lightweight local state; `useRef` + CSS `transform: translate3d()` drives the caret at 60fps without triggering full React tree re-renders. |
| **Icons** | **Lucide React** *(minimal use)* | Crisp monochrome 16px icons where needed (never inside tinted icon boxes, never in the navbar). |

---

## 3. Backend Stack (`server/`)

| Layer | Technology | Purpose & Rationale |
| --- | --- | --- |
| **Runtime & Framework** | **Node.js + Express.js** | Lightweight, predictable REST API server; cleanly supports future Socket.IO multiplayer races without serverless timeout limits. |
| **Language** | **TypeScript** | Shared type contracts with the frontend for sync payloads, test records, and API responses. |
| **Database** | **MongoDB** | Document model maps naturally to nested `categoryStats` objects and flexible user settings. |
| **ODM** | **Mongoose** | Schema enforcement, compound unique indexes (`userId + clientTestId`, `userId + date`), and aggregation pipelines. |
| **Authentication** | **JWT (`jsonwebtoken`) + `bcrypt`** | Password hashing (`bcrypt` cost ≥ 10) and stateless sessions stored in `HttpOnly`, `Secure`, `SameSite=Lax` cookies. |
| **Validation** | **Zod** | Strict runtime validation for auth inputs, `/api/sync` batch payloads, and `/api/generate-drill` requests. |
| **Security Middleware** | **`helmet`, `cors`, `express-rate-limit`, `cookie-parser`** | Standard HTTP security headers, strict origin allowlisting for the Next.js client, and brute-force/API-abuse protection. |
| **AI Drills (Phase 6)** | **Google Gemini API (`@google/genai`)** | Fast, low-latency generation of targeted weak-symbol drills and custom-topic snippets, isolated behind `/api/generate-drill`. |

---

## 4. Repository Structure

```text
justType/
├── client/                            # Next.js (App Router) Frontend
│   ├── src/
│   │   ├── app/
│   │   │   ├── layout.tsx             # IBM Plex fonts + no-flash theme script
│   │   │   ├── globals.css            # Warm Retro & Tokyo Night CSS variables
│   │   │   ├── page.tsx               # Active Typing Test (Home)
│   │   │   ├── practice/page.tsx      # Mode & difficulty selector
│   │   │   ├── progress/page.tsx      # 52-week calendar, streaks, weak-char stats
│   │   │   └── settings/page.tsx      # Local preferences
│   │   ├── components/
│   │   │   ├── layout/                # Plain-text Navbar, PageShell
│   │   │   ├── typing/                # 'use client' — TypingCanvas, Caret, ModeBar
│   │   │   ├── results/               # 'use client' — ResultsView, WpmChart, WeakCharsTable
│   │   │   └── calendar/              # 'use client' — ActivityCalendar, StreakSummary
│   │   ├── engine/                    # Pure TS typing engine, WPM & char-category math
│   │   ├── content/                   # Curated JSON + templateGenerator.ts
│   │   ├── storage/                   # IndexedDB (idb) & localStorage helpers
│   │   ├── services/                  # Typed fetch wrapper for Express API
│   │   └── types/                     # Frontend TypeScript interfaces
│   ├── next.config.ts
│   ├── tailwind.config.ts
│   └── package.json
│
├── server/                            # Express + TypeScript Backend
│   ├── src/
│   │   ├── config/                    # db.ts, env.ts (Zod-validated env vars)
│   │   ├── controllers/               # auth.controller.ts, sync.controller.ts, drill.controller.ts
│   │   ├── middleware/                # auth.middleware.ts, rateLimit.ts, errorHandler.ts
│   │   ├── models/                    # User.ts, TypingTest.ts, DailyActivity.ts
│   │   ├── routes/                    # auth.routes.ts, sync.routes.ts, drill.routes.ts
│   │   ├── services/                  # sync.service.ts, aiDrill.service.ts
│   │   ├── validators/                # Zod schemas
│   │   └── server.ts                  # Express app entry point
│   ├── tsconfig.json
│   └── package.json
│
├── docs/
│   ├── PRD.md
│   ├── SRS.md
│   ├── UI_UX_GUIDELINES.md
│   └── TECH_STACK.md
│
├── docker-compose.yml                 # Local MongoDB + optional app containers
└── README.md
```

---

## 5. Development, Testing & Deployment Tooling

* **Package Manager:** `npm` or `pnpm` (consistent lockfile across `client/` and `server/`).
* **TypeScript Execution (Server):** `tsx` (`tsx watch src/server.ts`) for fast local hot-reloading; `tsc` for production builds.
* **Testing:**
  * **Frontend / Engine:** **Vitest** for pure TypeScript unit tests (WPM math, backspace recalculation, character-category tracking, template generator).
  * **Backend:** **Jest + Supertest** for API integration tests (auth cookies, `/api/sync` idempotency, rate limiting).
* **Linting & Formatting:** ESLint + Prettier.
* **Local Database:** Docker Desktop (`mongo:7` via `docker-compose.yml`) or MongoDB Atlas free tier.
* **Production Hosting:**
  * **Frontend (`client/`):** Vercel (native Next.js App Router support).
  * **Backend (`server/`):** Render, Railway, or Fly.io (persistent Node.js process).
  * **Database:** MongoDB Atlas.

---

## 6. Environment Variables

### `client/.env.local`
```env
NEXT_PUBLIC_API_URL=http://localhost:5000/api
```

### `server/.env`
```env
PORT=5000
NODE_ENV=development
CLIENT_ORIGIN=http://localhost:3000
MONGODB_URI=mongodb://localhost:27017/justtype
JWT_SECRET=replace_with_strong_random_secret
JWT_EXPIRES_IN=7d
GEMINI_API_KEY=replace_with_api_key_for_phase_6
```

---

## 7. Key Technical Decisions

1. **Why Separate Express Backend Instead of Next.js Route Handlers?**  
   Keeps backend concerns cleanly isolated, simplifies rate limiting and Mongoose connection pooling, and leaves a direct upgrade path for **Socket.IO** real-time multiplayer races in V2 without rewriting APIs.
2. **Why `IndexedDB` Over `localStorage` for Test History?**  
   `localStorage` is synchronous (can block the main thread on large JSON parses) and capped at ~5MB. `IndexedDB` is asynchronous, handles thousands of test records and cached drills effortlessly, and supports indexed queries by date and sync status.
3. **Why Pure TypeScript for `src/engine/`?**  
   Decoupling WPM math, character-category attribution, and indentation-skip logic from React components makes the core typing engine trivial to unit-test with Vitest and guarantees zero-lag keystroke processing.