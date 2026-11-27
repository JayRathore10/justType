# System Architecture — justType

**Document Version:** 1.0  
**Architecture Style:** Local-First, Decoupled Client–Server Monorepo  
**Frontend:** Next.js (App Router) + TypeScript (`client/`)  
**Backend:** Express.js + TypeScript + MongoDB (`server/`)  

---

## 1. High-Level System Architecture

`justType` is designed around a **strict separation between the real-time typing loop and backend services**. The browser owns 100% of keystroke processing, content generation, metrics calculation, and local persistence. The Express server is an optional layer for cross-device cloud sync (Phase 5) and on-demand AI drill generation (Phase 6).

```text
+-------------------------------------------------------------------+
|                     CLIENT (Next.js App Router)                   |
|                                                                   |
|  [ Server Components ]           [ Client Components ]            |
|  - layout.tsx (Fonts, Theme)     - TypingCanvas (useRef Caret)    |
|  - Static Navbar & Shells        - ModeBar, ResultsView, Charts   |
|                                  - ActivityCalendar (52-week)     |
|                                           │                       |
|                                           ▼                       |
|  [ Pure TS Engine (src/engine/) ]  [ Content (src/content/) ]     |
|  - Keystroke state machine         - Curated JSON snippets        |
|  - WPM / Accuracy / Consistency    - templateGenerator.ts         |
|  - Char-category attribution       - Deduplication queue          |
|  - Auto-indent skip (Code Mode)                                   |
|                                           │                       |
|                                           ▼                       |
|  [ Local Storage Layer (src/storage/) ]                           |
|  - IndexedDB (idb): tests, dailyActivity, cachedDrills            |
|  - localStorage: theme, UI settings, recentExerciseIds            |
+-------------------------------------------------------------------+
                 │
                 │ Optional HTTPS (JSON + HttpOnly JWT Cookie)
                 ▼
+-------------------------------------------------------------------+
|                 SERVER (Express + TypeScript)                     |
|                                                                   |
|  Middleware: helmet -> cors -> rateLimit -> cookieParser -> Zod   |
|                                                                   |
|  [ Routes & Controllers ]                                         |
|  - /api/auth/*          -> Auth Service (bcrypt + JWT)            |
|  - /api/sync            -> Sync Service (Idempotent Upsert/Merge) |
|  - /api/generate-drill  -> AI Drill Service (Phase 6)             |
|          │                                      │                 |
|          ▼                                      ▼                 |
|  [ MongoDB + Mongoose ]                 [ Gemini Flash API ]      |
|  - Users                                - Weak-symbol drills      |
|  - TypingTests (userId + clientTestId)  - Custom topic snippets   |
|  - DailyActivity (userId + date)                                  |
+-------------------------------------------------------------------+
```

---

## 2. Frontend Architecture (`client/`)

### 2.1 Next.js App Router Boundary Strategy
To maximize First Contentful Paint (FCP) while preventing SSR hydration mismatches with browser storage:

* **Server Components (Default):** `src/app/layout.tsx`, navigation chrome, and static page shells render on the server with zero client JS overhead.
* **Client Components (`'use client'`):** Isolated strictly to interactive islands—`TypingCanvas`, `ModeBar`, `ResultsView`, `ActivityCalendar`, and `SettingsForm`.
* **Hydration Safety:** All `IndexedDB` and `localStorage` reads occur inside `useEffect` hooks. Before hydration completes, components render deterministic neutral placeholders so React never throws hydration mismatch warnings.
* **No-Flash Theme Script:** A minimal synchronous `<script>` in `layout.tsx` inspects `localStorage.getItem('justtype-theme')` and sets `data-theme="dark"` on `<html>` before first paint (defaulting to Warm Retro Light Mode).

### 2.2 Zero-Lag Typing Engine Architecture (`src/engine/`)
React's standard state-driven rendering can introduce frame drops on 1,000-character code snippets if every keystroke re-renders the entire character tree. The engine avoids this through three patterns:

1. **Pure TypeScript Core:** All typing logic (character comparison, backspace state rollback, WPM/Raw WPM/Accuracy math, category hit/miss tracking, and auto-indent whitespace skipping) lives in framework-agnostic modules under `src/engine/`.
2. **Caret via `useRef` + GPU Transform:** Caret coordinates are computed from the active character DOM node and applied directly via `caretRef.current.style.transform = translate3d(x, y, 0)` with a `70ms linear` CSS transition—bypassing React reconciliation on cursor movement.
3. **Memoized Rendering:** Target text is split into memoized word/line segments so only the active word/line updates on each keystroke.

### 2.3 Content Engine & Deduplication (`src/content/`)
```text
Request Exercise(mode, difficulty)
   ├── Check local deduplication queue (last N exercise IDs in localStorage)
   ├── If mode == 'programming' -> Pick unseen curated snippet from JSON
   ├── If mode == 'mixed' | 'developer' ->
   │      Hydrate parameterized template (names, dates, IDs, prices, paths)
   └── If mode == 'ai-drill' (Phase 6) ->
          Check IndexedDB cachedDrills -> else call POST /api/generate-drill
          (On network error / offline -> silently fallback to templateGenerator)
```

---

## 3. Local-First Data & Sync Architecture

### 3.1 Client Storage Schema
* **`IndexedDB` (`justtype-db` via `idb`):**
  * `tests` store: KeyPath `id` (client-generated `UUID`), indexed by `completedAt` and `synced` (`0` or `1`).
  * `dailyActivity` store: KeyPath `date` (`YYYY-MM-DD` in the user's local timezone). Stores `testsCompleted`, `correctChars`, `totalChars`, `typingTimeSec`, and `activityLevel` (`0–4`).
  * `cachedDrills` store: KeyPath `id`. Caches Phase 6 AI-generated drills for offline reuse.
* **`localStorage`:**
  * `justtype-theme`: `'light' | 'dark'`
  * `justtype-settings`: JSON object (`fontSize`, `backspaceEnabled`, `caretStyle`, `showLiveStats`).
  * `justtype-recent-ids`: Rolling array of recently completed exercise IDs.

### 3.2 Idempotent Local-to-Cloud Sync Flow (Phase 5)
When an anonymous user signs up, logs in, or completes a test while authenticated:

1. **Client Collects Delta:** Queries `IndexedDB` `tests` store where `synced === false`.
2. **Batch Upload:** Sends `POST /api/sync` with `{ tests: unsyncedTests, settings }`.
3. **Server Deduplication:** Express validates the payload with **Zod**, then performs a `bulkWrite` upsert against MongoDB `TypingTest` using the compound unique index `{ userId: 1, clientTestId: 1 }`. Duplicate submissions are safely ignored.
4. **Daily Aggregate Recalculation:** For every affected `localDate` in the batch, the server recomputes the `DailyActivity` document from `TypingTest` records and upserts it using `{ userId: 1, date: 1 }`.
5. **Client Acknowledgment:** Server returns the merged `DailyActivity[]` and synced `clientTestId[]`. Client marks those local records `synced = true` in `IndexedDB` and updates the local calendar.

---

## 4. Backend Architecture (`server/`)

The Express backend follows a strict layered architecture: **Routes → Middleware → Controllers → Services → Models**.

### 4.1 Request Pipeline
1. **Security & Parsing:** `helmet()` → `cors({ origin: CLIENT_ORIGIN, credentials: true })` → `express.json({ limit: '1mb' })` → `cookieParser()`.
2. **Rate Limiting:**
   * Auth routes (`/api/auth/*`): Strict IP rate limit (e.g., 20 requests / 15 min).
   * AI Drill route (`/api/generate-drill`): Per-user/IP rate limit (e.g., 15 requests / hour) to control LLM API costs.
3. **Validation:** Zod middleware validates `req.body` before hitting controllers, returning structured `400 Bad Request` responses on failure.
4. **Authentication:** `authMiddleware` verifies the `HttpOnly` JWT cookie and attaches `req.userId`.
5. **Centralized Error Handling:** All async errors forward to `errorHandler.ts`, which logs internal details server-side and returns sanitized JSON errors to the client.

### 4.2 Database Indexing Strategy (MongoDB)
| Collection | Index | Type | Purpose |
| --- | --- | --- | --- |
| `users` | `{ email: 1 }`, `{ username: 1 }` | Unique | Fast login lookup and duplicate prevention |
| `typingtests` | `{ userId: 1, clientTestId: 1 }` | Compound Unique | Guarantees idempotent local-to-cloud sync |
| `typingtests` | `{ userId: 1, completedAt: -1 }` | Compound | Fast paginated test history queries |
| `dailyactivities` | `{ userId: 1, date: 1 }` | Compound Unique | 1-to-1 daily calendar grid lookup per user |

---

## 5. Adaptive AI Drill Pipeline (Phase 6 — Optional)

AI integration is strictly isolated from the core typing loop:

```text
User clicks "Drill weak keys" or submits "Custom topic"
   │
   ▼
POST /api/generate-drill { mode, difficulty, weakCategories, topic }
   │
   ▼
Express Server (aiDrill.service.ts)
   ├── Validate input via Zod + enforce rate limit
   ├── Construct strict system prompt (plain text only, no explanations, exact target length)
   ├── Call Gemini Flash API with timeout (5s)
   ├── Sanitize output (strip markdown fences, normalize line endings)
   └── Return { id, mode, difficulty, text }
   │
   ▼
Client caches drill in IndexedDB (cachedDrills) and loads into TypingCanvas
```

* **Failure Mode:** If the LLM API times out, returns invalid output, or the user is offline, the client catches the error and immediately generates a local exercise via `templateGenerator.ts` without blocking the user.

---

## 6. Deployment & Runtime Topology

* **Frontend (`client/`):** Deployed on **Vercel** (automatic preview deployments, Edge CDN for static assets and fonts).
* **Backend (`server/`):** Deployed as a persistent Node.js service on **Render / Railway / Fly.io** (supports future Socket.IO multiplayer upgrades without serverless cold starts).
* **Database:** **MongoDB Atlas** (TLS-encrypted connection via `MONGODB_URI`).
* **Local Development:** `docker-compose.yml` spins up `mongo:7` locally; `client/` runs on `http://localhost:3000` (`next dev`) and `server/` runs on `http://localhost:5000` (`tsx watch src/server.ts`).