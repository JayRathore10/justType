# Product Requirements Document (PRD) — justType (v2.1)

**Product Name:** justType  
**Description:** A minimal, zero-login typing practice tool for real-world text—code, terminal commands, numbers, punctuation, and mixed technical sentences.  
**Document Version:** 2.1  
**Status:** Ready for Development (Lean MVP + Full Roadmap)  

---

## 1. Product Philosophy & Core Rules

1. **Zero-Friction Entry:** Opening `justType` lands directly on the active typing interface. No login wall, no onboarding modal, no marketing landing page.
2. **Real-World Muscle Memory:** Every exercise trains realistic character combinations (uppercase, numbers, brackets, operators, paths, syntax) rather than lowercase dictionary words.
3. **Actionable Weakness Tracking:** Instead of only reporting WPM, the app tracks accuracy by character category (e.g., `{}`, `[]`, numbers, symbols) and lets users drill their weakest categories.
4. **Local-First Architecture:** All typing, history, personal bests, and the activity calendar work offline using `IndexedDB` and `localStorage`. Accounts and AI drills are strictly optional enhancements, never prerequisites.

---

## 2. Design System & Anti-AI-Slop Specification

To ensure `justType` looks like a serious developer utility rather than a generic AI-generated template, the frontend **must strictly enforce all 20 design constraints**:

### Visual & Surface Bans
* **No Purple Gradients:** Zero purple, indigo, or multi-stop background gradients. Use flat, solid surfaces (e.g., `#111214` background, `#181A1F` elevated surface).
* **No Gradient Text:** All headings, logos, and metrics use solid, high-contrast ink.
* **No Glows:** Ban neon `box-shadow` glows, drop-shadow halos, and blurred color auras behind elements.
* **No Glossy Blobs:** Ban floating mesh gradients, glassmorphism blobs, and decorative background shapes.
* **No Pill Corners:** Ban `rounded-full` on buttons, tabs, and mode selectors. Use crisp, restrained radii (`2px` to `6px`).
* **No Bouncing Animations:** Ban bobbing elements, bouncing arrows, and spring-bounce transitions. Motion is limited to a smooth, linear caret glide (`60ms–80ms`) and instant state changes.

### Typography & Copy Discipline
* **Not Inter:** Do not use Inter. Pair **IBM Plex Mono** or **JetBrains Mono** (typing area, stats, code) with **IBM Plex Sans** or **Geist** (UI chrome).
* **Say What It Does:** Replace vague taglines with plain, literal copy: *"Practice typing real code, symbols, and numbers."*
* **No Hype Words:** Ban words like *revolutionary, supercharge, unleash, next-gen, seamless, elevate,* or *mastery* across all UI copy and empty states.
* **No Tiny ALL CAPS:** Ban `10px–11px` uppercase tracked-out micro-labels (`uppercase tracking-widest`). Use normal sentence case with clear size/weight contrast.
* **No Emoji in the Nav:** Navigation items (`Test`, `Practice`, `Progress`, `Settings`, `Login`) are plain text only—zero emojis.

### Layout, Hierarchy & Color
* **Strict 4-Point Grid:** All spacing, padding, gaps, and component dimensions must be multiples of `4px` (`4, 8, 12, 16, 24, 32, 48px`).
* **No "Centred Everything":** Do not center-align every block. Use left-aligned editorial layouts for navigation, mode bars, results breakdowns, and progress dashboards; only constrain the typing canvas to a readable max-width (`72ch–80ch`).
* **One Main Button:** Every view has a single primary action (e.g., `Next Test` on the Results screen, `Sync Account` on the Progress prompt)—never competing CTA clusters.
* **One Bright Moment:** Keep 95% of the UI neutral charcoal, off-white, and muted grey, reserving **one** high-contrast accent color (`#EAB308` amber/yellow) for the active caret, active mode indicator, and primary action. (Semantic green/red remain strictly for correct/incorrect keystrokes and the activity calendar.)

### Content & Component Bans
* **No Pill Above Headline:** Never place eyebrow badges (e.g., `v1.0 • Free Forever`) above headings.
* **No Fake Reviews:** Zero fabricated testimonials, user quotes, or star ratings.
* **No Stat Banners:** Zero vanity banners (e.g., *"50,000+ tests completed"*).
* **No Icon Boxes:** Never wrap icons in tinted rounded-square boxes next to feature titles or settings.
* **Cards Must Say Something:** Avoid decorative cards. Any bordered container must display live, functional data (e.g., a WPM chart, a weak-character table, or the contribution calendar)—never filler marketing text.

---

## 3. MVP Scope (V1) vs. Post-MVP (V2)

To ship fast without content burnout, V1 focuses on the core engine, 3 high-impact modes, and local progress.

| Feature Area | V1 (Lean MVP) | V2 (Expansion) |
|---|---|---|
| **Typing Modes** | 1. **Mixed Text** (Default)<br>2. **Programming** (JS/TS, Python, C++)<br>3. **Developer / Terminal** (Git, CLI, paths, JSON) | Normal Text, Messages & Emails, Data & Syntax (SQL/YAML/HTML), Special Characters drill, additional languages (Go, Rust, Java, C#) |
| **Difficulty Levels** | Easy, Medium, Hard, Expert (across all 3 modes) | Adaptive difficulty based on user's weakest characters |
| **Content Engine** | Curated JSON library + **Template Generator** (names, dates, IDs, prices, paths) | Custom user text import, **Adaptive AI Drills** (Phase 6) |
| **Persistence** | `IndexedDB` (test history, char stats) + `localStorage` (settings, theme) | MongoDB + Auth + Local-to-Cloud Sync |
| **Analytics** | WPM, Raw WPM, Accuracy, Consistency, **Character Category Breakdown**, **Activity Calendar**, Streaks | Inter-keystroke latency diagnostics, Public Leaderboards, Multiplayer Races (Socket.IO) |
| **Themes** | Dark (default) + Light | Midnight, Nord, Dracula, Monokai |

---

## 4. Core Functional Specifications

### 4.1 Typing Engine (Zero-Lag Architecture)
* **Test Durations:** `15s`, `30s`, `60s`, `120s`, plus `Snippet Completion` mode for Programming/Developer exercises.
* **Start Trigger:** Timer starts on the first valid printable keystroke.
* **Performance Requirement:** Keystroke handling must not trigger full-tree React re-renders. Track active index and character states with minimal state updates and drive caret positioning via `useRef` + CSS `transform: translate3d()`.
* **Code Indentation UX:** In Programming Mode, pressing `Enter` automatically skips leading indentation whitespace on the next line so users aren't forced to tap `Space` 4–8 times per line.
* **Keyboard Shortcuts:**
  * `Esc` — Close modal/settings overlay, or restart test when focus is in the typing area.
  * `Ctrl + Enter` (or `Cmd + Enter`) — Restart test (avoids conflicting with `Tab` in code modes).
  * `Backspace` — Configurable (enabled by default); accurately recalculates character stats.

### 4.2 Content Engine & Template Generator
To prevent repetition without manually writing thousands of sentences:
* **Curated Snippets:** Stored in structured JSON by `mode`, `difficulty`, and `tags`.
* **Template Generator (Mixed Text & Developer Modes):** Uses parameterized templates hydrated with realistic pools (names, ISO dates, timestamps, order IDs, prices, HTTP status codes, file paths).
  * *Example Template:* `Warning: {user} failed login ({n}/5 attempts) from {ip}; retry at {time}.`
* **Deduplication:** Maintain a rolling queue of recently seen exercise IDs in `localStorage` so users never see repeats back-to-back.

### 4.3 Metrics & Weak-Character Tracking
* **Standard Formulas:**
  * `WPM = (Correct Characters / 5) / Minutes`
  * `Raw WPM = (Total Characters Typed / 5) / Minutes`
  * `Accuracy = (Correct Characters / Total Characters Typed) * 100`
* **Category Accuracy Breakdown:** Track hit/miss ratios for:
  * Uppercase, Lowercase, Numbers, Parentheses `()`, Square Brackets `[]`, Curly Braces `{}`, Operators `+-=*/<>`, Punctuation `,.:;'"`, and Symbols `@#$%^&`.
* **Actionable Results Screen:** Displays final WPM, Accuracy, WPM-over-time line chart, and a left-aligned table of the user's **3 weakest character categories** with a single primary button (`Next Test`) and a secondary link (`Drill Weak Symbols`).

### 4.4 Activity Calendar & Streak System
* **Visual Grid:** GitHub-style 52-week contribution grid using the user's local timezone.
* **Metric:** Driven strictly by **total correct characters typed per day** (visiting the site does not count).
  * `Level 0`: `0` (`#161B22`)
  * `Level 1`: `1–499` (`#0E4429`)
  * `Level 2`: `500–1,499` (`#006D32`)
  * `Level 3`: `1,500–2,999` (`#26A641`)
  * `Level 4`: `3,000+` (`#39D353`)
* **Tooltip:** Displays date, tests completed, correct characters, time practiced, average WPM, and average accuracy.
* **Streaks:** Current streak, longest streak, and total active days calculated from the same local-day boundary.

### 4.5 Anonymous-First & Cloud Sync (Phase 5)
* Every completed test gets a client-generated `UUID` and ISO timestamp.
* When an anonymous user signs up or logs in, the client uploads local `IndexedDB` records to `/api/sync`.
* The backend deduplicates by `testId` (unique index on `userId + testId`) and recalculates `DailyActivity` aggregates (unique index on `userId + date`) so zero progress is lost or double-counted.

### 4.6 Adaptive AI Drills (Phase 6 — Optional)
AI is used strictly as an **on-demand, targeted drill generator**—never in the core typing loop and never as a chatbot.
* **Weak-Symbol Drill Generator:** Sends the user's top 3 weakest character categories (e.g., `{}`, `[]`, `=>`, numbers) to a Next.js Route Handler (`/api/generate-drill`) to generate realistic, meaningful code or Mixed Text exercises heavily weighted toward those exact characters.
* **On-Demand Stack & Topic Snippets:** Allows users to enter a short topic (e.g., *"Next.js middleware with JWT"*, *"Docker CLI commands"*, *"PostgreSQL joins"*) to generate a syntactically valid 60-second snippet.
* **One-Line Mistake Diagnostics:** Tracks inter-keystroke latency alongside errors. After 5–10 tests, outputs one plain, concrete observation (e.g., *"You pause 340ms before `{` and frequently mistype `->` as `=>`"*).
* **Strict Anti-Slop AI Rules:**
  * No chat sidebars, no sparkle icons, no *"AI-Powered"* badges. Label controls plainly (`Drill weak keys`, `Custom topic`).
  * Zero latency in standard tests; AI only runs when explicitly triggered.
  * Cache generated drills in `IndexedDB` for offline replay.
  * Silently fall back to local `templateGenerator.ts` if offline or rate-limited.

---

## 5. Tech Stack & Project Structure (Next.js App Router)

**Frontend:** Next.js (App Router), React, TypeScript (`.tsx`), Tailwind CSS, `next/font/google` (`IBM_Plex_Mono` + `IBM_Plex_Sans`), Recharts, `idb` (lightweight IndexedDB wrapper).  
**Backend (Phase 5–6):** Next.js Route Handlers (`src/app/api/`) with MongoDB, Mongoose, JWT (HttpOnly cookies), bcrypt, Zod validation, and a fast LLM API (e.g., Gemini Flash) for Phase 6 drills—or a separate Node.js + Express + TypeScript server if adding Socket.IO multiplayer later.

```text
justType/
├── src/
│   ├── app/
│   │   ├── layout.tsx                 # Root shell, IBM Plex fonts, theme init script
│   │   ├── globals.css                # 4-point grid tokens & anti-slop base styles
│   │   ├── page.tsx                   # Home / Active Typing Test
│   │   ├── practice/page.tsx          # Mode & difficulty selector
│   │   ├── progress/page.tsx          # Activity calendar, streaks, weak-char stats
│   │   ├── settings/page.tsx          # Local preferences
│   │   └── api/
│   │       ├── auth/                  # (Phase 5) Login, signup, session
│   │       ├── sync/route.ts          # (Phase 5) Local-to-cloud idempotent merge
│   │       └── generate-drill/route.ts # (Phase 6) Adaptive weak-key & topic drills
│   │
│   ├── components/
│   │   ├── layout/                    # Navbar (plain text, no emojis)
│   │   ├── typing/                    # 'use client' — TypingCanvas, Caret, ModeBar
│   │   ├── results/                   # 'use client' — ResultsView, WpmChart, WeakCharsTable
│   │   └── calendar/                  # 'use client' — ActivityCalendar, StreakSummary
│   │
│   ├── engine/                        # Pure TS typing engine, WPM & char-category math
│   ├── content/                       # Curated JSON + templateGenerator.ts
│   ├── storage/                       # 'use client' — IndexedDB & localStorage layer
│   └── types/                         # Shared TypeScript interfaces
│
├── public/
├── next.config.ts
├── tailwind.config.ts
└── package.json
```

### Next.js-Specific Architecture Rules
* **Client/Server Boundary:** Keep `layout.tsx`, navigation, and initial exercise data as Server Components for instant first paint. Mark interactive typing components (`TypingCanvas`, `ModeBar`), charts, and `IndexedDB` hooks with `'use client'`.
* **Zero Hydration Mismatch:** Load anonymous history, streaks, and settings from `IndexedDB`/`localStorage` inside `useEffect` (with a clean neutral state on first render) so SSR never throws hydration errors.
* **No Flash of Wrong Theme:** Inject a small inline `<script>` in `layout.tsx` to read the saved theme from `localStorage` before paint so Dark Mode doesn't flash white on refresh.
* **Font Loading:** Use `next/font/google` to load `IBM_Plex_Mono` and `IBM_Plex_Sans` with CSS variables (`--font-mono`, `--font-sans`) for zero layout shift.

---

## 6. Development Phases

1. **Phase 1 — Core Engine & UI Shell:** Initialize Next.js (`npx create-next-app@latest justtype --ts --tailwind --eslint --app --src-dir`), configure `next/font/google` (`IBM_Plex_Mono`, `IBM_Plex_Sans`), set up strict 4px spacing tokens in `globals.css`, and build the zero-lag `'use client'` `TypingCanvas`, timer, WPM/Accuracy calculator, and Results view.
2. **Phase 2 — Modes & Content Engine:** Implement Mixed Text (with template generator), Programming Mode (with auto-indent skip), and Developer Mode across all 4 difficulties.
3. **Phase 3 — Local Progress & Calendar:** Wire up `IndexedDB` storage, weak-character category analytics, 52-week contribution calendar, and streak tracking.
4. **Phase 4 — Polish & Accessibility:** Light/Dark themes, mobile keyboard support, reduced-motion support, and performance profiling on 500+ character code snippets.
5. **Phase 5 — Optional Auth & Cloud Sync:** MongoDB integration, JWT auth, and idempotent local-to-cloud merge via `/api/sync`.
6. **Phase 6 — Adaptive AI Drills:** Build `/api/generate-drill` for targeted weak-symbol drills and custom topic snippets, add `IndexedDB` caching, and implement graceful offline fallback.

---

## 7. Acceptance Criteria (V1 Launch)

* Opening the app lets the user type immediately with zero login prompts or marketing fluff.
* All 20 anti-AI-slop rules pass visual review (no gradients, no glows, no Inter, no emojis in nav, strict 4px spacing, one bright accent).
* Mixed Text, Programming, and Developer modes work across Easy, Medium, Hard, and Expert.
* Programming mode preserves newlines and skips leading indentation cleanly on `Enter`.
* WPM, Accuracy, and Character Category breakdowns calculate accurately, including backspace corrections.
* Refreshing the page preserves test history, settings, streaks, and the contribution calendar locally with zero Next.js hydration errors.
* Typing more correct characters on the same day visibly darkens/brightens that day's green tile on the calendar.
* Core typing and local progress work 100% offline with zero dependency on external AI APIs.