# Software Requirements Specification (SRS) — justType

**Document Version:** 1.0  
**Project Name:** justType  
**Architecture:** Local-First Next.js (App Router) Web Application  
**Status:** Approved for Development  

---

## 1. Introduction

### 1.1 Purpose
This Software Requirements Specification (SRS) defines the functional, non-functional, data, and interface requirements for **justType**, a real-world typing practice web application. It serves as the technical contract for frontend, backend, and QA implementation.

### 1.2 Scope
`justType` enables users to practice typing realistic content—mixed alphanumeric/symbol sentences, programming code, and developer CLI commands—without requiring authentication. Core typing, statistics, character-category weakness tracking, and a 52-week activity calendar operate 100% client-side using `IndexedDB` and `localStorage`. Optional account authentication, MongoDB cloud synchronization, and on-demand AI drill generation are provided via Next.js Route Handlers.

### 1.3 Definitions & Acronyms
* **WPM (Words Per Minute):** `(Correct Characters / 5) / Elapsed Minutes`
* **Raw WPM:** `(Total Characters Typed / 5) / Elapsed Minutes`
* **Accuracy:** `(Correct Characters / Total Characters Typed) * 100`
* **SSR:** Server-Side Rendering (Next.js App Router)
* **IDB:** Browser `IndexedDB` (accessed via the `idb` wrapper)
* **UUID:** Universally Unique Identifier (`crypto.randomUUID()`)

---

## 2. Overall Description

### 2.1 Product Perspective
`justType` is a standalone, local-first web application. The typing engine has zero runtime dependency on backend APIs or external LLM services. Network requests are strictly isolated to optional authentication, cloud sync (Phase 5), and explicit user-triggered AI drills (Phase 6).

### 2.2 User Classes
1. **Anonymous User (Default):** Accesses all typing modes, settings, local history, weak-character analytics, and the local activity calendar immediately upon page load.
2. **Authenticated User (Optional):** Retains full offline capability while synchronizing test records, daily activity, and settings across devices via MongoDB.

### 2.3 Operating Environment
* **Client:** Modern desktop and mobile browsers (Chrome, Firefox, Safari, Edge) supporting ES2022, `IndexedDB`, `localStorage`, and CSS Custom Properties.
* **Framework:** Next.js (App Router), React, TypeScript (`.tsx`), Tailwind CSS.
* **Server (Phase 5–6):** Node.js runtime hosting Next.js Route Handlers, MongoDB (Mongoose), and an external LLM API (e.g., Gemini Flash).

### 2.4 Design & Implementation Constraints
* **Strict TypeScript:** All source files must use TypeScript (`.ts` / `.tsx`) with strict mode enabled.
* **Zero Hydration Mismatches:** Browser-only storage (`IndexedDB`, `localStorage`) must be read inside `useEffect` hooks; SSR must render deterministic neutral markup.
* **Anti-AI-Slop UI Compliance:** All views must adhere strictly to `UI_UX_GUIDELINES.md` (flat solid surfaces, strict 4-point grid, `IBM Plex Mono` + `IBM Plex Sans`, no Inter, no gradients, no glows, no `rounded-full` pills, no emojis in navigation).

---

## 3. Functional Requirements

### 3.1 Typing Engine (`FR-ENG`)
* **FR-ENG-01 (Immediate Entry):** Loading `/` shall render an active, ready-to-type exercise without modals, login walls, or onboarding overlays.
* **FR-ENG-02 (Timer Trigger):** The test timer shall begin on the first valid printable keystroke, not on page load.
* **FR-ENG-03 (Test Configurations):** The engine shall support timed durations (`15s`, `30s`, `60s`, `120s`) and `Snippet Completion` mode (test ends when the final character is typed).
* **FR-ENG-04 (Character State Tracking):** Every character in the target text shall transition between `untyped`, `correct`, and `incorrect`. Extra typed characters beyond a word boundary shall be tracked as `extra`.
* **FR-ENG-05 (Backspace Handling):** When enabled (default), `Backspace` shall revert the previous character state and accurately recalculate live accuracy and character counts.
* **FR-ENG-06 (Code Indentation UX):** In Programming Mode, pressing `Enter` at the end of a line shall automatically advance the cursor past leading indentation whitespace on the next line.
* **FR-ENG-07 (Keyboard Shortcuts):**
  * `Esc`: Close open modals/overlays, or restart the active test when focus is in the typing arena.
  * `Ctrl + Enter` / `Cmd + Enter`: Restart the active test immediately (avoids conflicting with `Tab` in code modes).

### 3.2 Typing Modes & Content Engine (`FR-CNT`)
* **FR-CNT-01 (V1 Modes):** The system shall support three primary modes in V1, each with four difficulty levels (`easy`, `medium`, `hard`, `expert`):
  1. **Mixed Text (Default):** Meaningful English sentences combining uppercase, lowercase, numbers, punctuation, brackets, and symbols.
  2. **Programming:** Syntactically valid snippets in JavaScript/TypeScript, Python, and C++ preserving exact newlines and indentation.
  3. **Developer / Terminal:** Realistic Git commands, shell pipelines, file paths, environment variables, and JSON snippets.
* **FR-CNT-02 (Template Generator):** Mixed Text and Developer modes shall use parameterized templates hydrated with randomized realistic pools (names, dates, timestamps, IDs, prices, IPs, paths) to prevent repetitive tests.
* **FR-CNT-03 (Deduplication):** The client shall maintain a rolling queue of recently completed exercise IDs in `localStorage` and prioritize unseen exercises.

### 3.3 Metrics & Weak-Character Analytics (`FR-MET`)
* **FR-MET-01 (Core Metrics):** Upon test completion, the system shall calculate and display WPM, Raw WPM, Accuracy, Consistency, Correct/Incorrect/Extra/Missed character counts, and a WPM-over-time series.
* **FR-MET-02 (Character Category Tracking):** The engine shall record hit/miss counts across 9 categories: `uppercase`, `lowercase`, `numbers`, `parentheses ()`, `squareBrackets []`, `curlyBraces {}`, `operators +-=*/<>`, `punctuation ,.:;'"`, and `symbols @#$%^&`.
* **FR-MET-03 (Weakest-Category Surface):** The Results view shall display the user's 3 lowest-accuracy categories from the test alongside a single primary action (`Next Test`) and secondary links (`Retry`, `Drill weak keys`).

### 3.4 Activity Calendar & Streaks (`FR-CAL`)
* **FR-CAL-01 (Daily Activity Aggregation):** Completing a valid typing test shall increment the user's daily record grouped by their **local calendar date (`YYYY-MM-DD`)**. Visiting the site without completing a test shall not record activity.
* **FR-CAL-02 (Intensity Levels):** Each day tile on the 52-week contribution grid shall map to one of five intensity levels based on **total correct characters typed that day**:
  * `Level 0`: `0`
  * `Level 1`: `1 – 499`
  * `Level 2`: `500 – 1,499`
  * `Level 3`: `1,500 – 2,999`
  * `Level 4`: `3,000+`
* **FR-CAL-03 (Calendar Tooltips):** Hovering or focusing a date tile shall display the formatted date, tests completed, correct characters, total practice time, average WPM, and average accuracy.
* **FR-CAL-04 (Streaks):** The system shall compute Current Streak, Longest Streak, and Total Active Days using the same local-day boundary.

### 3.5 Local Persistence & Cloud Sync (`FR-SYN`)
* **FR-SYN-01 (Local Storage):** Test records, character statistics, and daily aggregates shall persist in `IndexedDB`. UI settings (theme, font size, caret style, backspace toggle) shall persist in `localStorage`.
* **FR-SYN-02 (Optional Auth — Phase 5):** Users may register or log in via `/api/auth/*`. Authentication shall use `bcrypt` password hashing and `HttpOnly`, `Secure`, `SameSite=Lax` JWT cookies.
* **FR-SYN-03 (Idempotent Merge — Phase 5):** Upon login/signup, the client shall POST unsynced local test records to `/api/sync`. The server shall deduplicate by `(userId, clientTestId)`, merge records without overwriting cloud history, and recalculate `DailyActivity` totals.

### 3.6 Adaptive AI Drills (`FR-AI` — Phase 6 Optional)
* **FR-AI-01 (On-Demand Generation):** When explicitly triggered by the user (`Drill weak keys` or `Custom topic`), the client shall call `POST /api/generate-drill` with `{ mode, difficulty, weakCategories, topic }`.
* **FR-AI-02 (Sanitization & Caching):** The server shall strip markdown fences and return plain, ready-to-type text. Generated drills shall be cached in `IndexedDB` for offline replay.
* **FR-AI-03 (Graceful Fallback):** If the user is offline, unauthenticated (if gated), or rate-limited, the client shall silently fall back to the local `templateGenerator.ts`.

---

## 4. Data Models & Interfaces

### 4.1 Client-Side `IndexedDB` Object Stores
* **`tests` store** (KeyPath: `id` [UUID], Index: `completedAt`, `synced`)
  ```ts
  interface TypingTestRecord {
    id: string;                 // Client-generated UUID
    mode: 'mixed' | 'programming' | 'developer';
    difficulty: 'easy' | 'medium' | 'hard' | 'expert';
    durationSec: number;
    wpm: number;
    rawWpm: number;
    accuracy: number;
    consistency: number;
    correctChars: number;
    incorrectChars: number;
    extraChars: number;
    missedChars: number;
    categoryStats: Record<CharCategory, { hits: number; misses: number }>;
    localDate: string;          // YYYY-MM-DD in user's local timezone
    completedAt: string;        // ISO-8601 UTC timestamp
    synced: boolean;
  }
  ```
* **`dailyActivity` store** (KeyPath: `date` [`YYYY-MM-DD`])
* **`cachedDrills` store** (KeyPath: `id`, stores Phase 6 AI-generated exercises)

### 4.2 Server-Side MongoDB Schemas (Phase 5)
* **`User`:** `_id`, `username` (unique), `email` (unique), `passwordHash`, `settings`, `createdAt`, `updatedAt`.
* **`TypingTest`:** `_id`, `userId` (ref `User`), `clientTestId` (string), plus all telemetry fields from `TypingTestRecord`. **Compound Unique Index:** `{ userId: 1, clientTestId: 1 }`.
* **`DailyActivity`:** `_id`, `userId` (ref `User`), `date` (`YYYY-MM-DD`), `testsCompleted`, `correctChars`, `totalChars`, `typingTimeSec`, `activityLevel`. **Compound Unique Index:** `{ userId: 1, date: 1 }`.

### 4.3 REST API Endpoints (Next.js Route Handlers)
| Method | Endpoint | Phase | Purpose |
| --- | --- | --- | --- |
| `POST` | `/api/auth/signup` | Phase 5 | Create account, validate with Zod, set HttpOnly JWT cookie |
| `POST` | `/api/auth/login` | Phase 5 | Authenticate user, set HttpOnly JWT cookie |
| `POST` | `/api/auth/logout` | Phase 5 | Clear JWT cookie |
| `POST` | `/api/sync` | Phase 5 | Idempotent upload/merge of local `TypingTestRecord[]` and return updated cloud aggregates |
| `POST` | `/api/generate-drill` | Phase 6 | Generate targeted weak-symbol or custom-topic typing snippet |

---

## 5. External Interface Requirements (UI/UX)

* **Default Theme (Light — Warm Retro):** Background `#FFEEB3`, Typing Container `#B4D7CD`, Secondary `#8EA8C3`, Accent/Caret `#F47321`, Text `#29343B`, Muted `#64717A`, Correct `#387D65`, Incorrect `#C94C42`.
* **Alternate Theme (Dark — Tokyo Night):** Background `#1A1B26`, Typing Container `#24283B`, Secondary `#1F2335`, Accent `#7AA2F7`, Caret `#E0AF68`, Text `#C0CAF5`, Muted `#565F89`, Correct `#9ECE6A`, Incorrect `#F7768E`.
* **No-Flash Theme Script:** A synchronous inline script in `src/app/layout.tsx` shall apply `data-theme="dark"` before first paint if saved in `localStorage`.
* **Typography:** `IBM Plex Mono` (`--font-mono`) and `IBM Plex Sans` (`--font-sans`) loaded via `next/font/google`.

---

## 6. Non-Functional Requirements

### 6.1 Performance (`NFR-PERF`)
* **Keystroke Latency:** Input-to-paint latency shall remain `< 16ms` (60fps) on snippets up to 1,000 characters. Caret movement shall use `useRef` + CSS `transform: translate3d()` with `70ms linear` transition rather than full React tree re-renders.
* **Initial Load:** First Contentful Paint (FCP) on `/` shall be `< 1.2s` on standard broadband via Next.js Server Components for static chrome.

### 6.2 Reliability & Offline Support (`NFR-REL`)
* All V1 features (typing, stats, history, calendar, settings) shall function with zero network connectivity after initial page load.
* No completed test data shall be lost on browser refresh or during anonymous-to-authenticated migration.

### 6.3 Security (`NFR-SEC`)
* All API payloads shall be validated with **Zod** schemas.
* Authentication and `/api/generate-drill` endpoints shall enforce IP/user rate limiting.
* Passwords shall be hashed with `bcrypt` (cost factor ≥ 10); secrets and API keys shall reside strictly in server-side environment variables.

### 6.4 Accessibility (`NFR-A11Y`)
* Typing errors shall never rely solely on color; incorrect characters shall also display a `1px` underline.
* All interactive controls shall maintain visible `:focus-visible` outlines (`2px solid var(--accent)`).
* `@media (prefers-reduced-motion: reduce)` shall set caret transitions to `0ms`.

---

## 7. Verification & Acceptance Criteria

1. **Zero-Login Test:** Opening `/` in an incognito window allows immediate typing and saves results to `IndexedDB` across page reloads.
2. **Engine Accuracy:** Unit tests (Jest/Vitest) verify WPM, Raw WPM, Accuracy, backspace recalculation, and character-category attribution across edge cases.
3. **Code Mode Indentation:** Pressing `Enter` on a multiline Python/JS snippet skips leading spaces on the next line without marking them incorrect.
4. **Calendar Reactivity:** Completing tests totaling 500+ and 1,500+ correct characters on the same local day transitions the tile from Level 1 → Level 2 → Level 3.
5. **Sync Idempotency:** Submitting the same `clientTestId` twice to `/api/sync` results in exactly one MongoDB document and no double-counted daily characters.
6. **UI Compliance:** Visual audit confirms zero violations of the 20 Anti-AI-Slop rules and exact adherence to both Warm Retro and Tokyo Night palettes.