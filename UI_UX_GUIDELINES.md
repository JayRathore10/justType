# UI/UX Design Guidelines — justType

**Document Version:** 1.0  
**Stack Target:** Next.js (App Router), Tailwind CSS, `next/font/google`  
**Default Theme:** Light Mode — Warm Retro (`#FFEEB3`)  
**Alternate Theme:** Dark Mode — Tokyo Night (`#1A1B26`)  

---

## 1. Core Design Philosophy

`justType` is a tactile, distraction-free developer utility—not a SaaS marketing template. Every pixel serves one goal: letting the user open the page, start typing immediately, and read their telemetry without visual noise.

To keep the interface crisp, intentional, and free of generic AI-generated UI patterns, every component must pass the **20 Anti-AI-Slop Rules** below.

---

## 2. The 20 Anti-AI-Slop Rules (Strict Enforcement)

### Surface, Color & Shape
1. **No Purple Gradients:** Zero purple, indigo, or multi-stop gradients anywhere. All backgrounds are flat, solid hex colors.
2. **No Gradient Text:** Headings, logos, WPM counters, and links use solid ink (`--text-main` or `--accent`).
3. **No Glows:** Ban neon `box-shadow` halos, colored drop shadows, and blurred auras around buttons, carets, or containers.
4. **No Glossy Blobs:** Ban floating mesh gradients, glassmorphism overlays, and decorative background shapes.
5. **No Pill Corners:** Ban `rounded-full` on buttons, mode selectors, and tags. Use crisp, restrained corners (`2px` to `6px`, default `4px`). *(Note: Toolbar controls and mode selectors are styled as flat segmented buttons, never rounded pills.)*
6. **One Bright Moment:** Reserve the primary accent (`#F47321` in Light Mode; `#7AA2F7` / `#E0AF68` in Dark Mode) strictly for the active caret, active mode indicator, and primary action button. Semantic green/red are used only for typing feedback and the activity calendar.

### Typography & Copy
7. **Not Inter:** Never load or fall back to Inter. Use **IBM Plex Mono** (typing arena, code, metrics, shortcuts) paired with **IBM Plex Sans** (navigation, labels, settings).
8. **Say What It Does:** Use plain, literal copy (e.g., *"Practice typing real code, symbols, and numbers"*). No abstract slogans.
9. **No Hype Words:** Ban *revolutionary, supercharge, unleash, next-gen, seamless, elevate, mastery,* and *game-changing* across all UI text and empty states.
10. **No Tiny ALL CAPS:** Ban `10px–11px` uppercase tracked-out micro-labels (`uppercase tracking-widest`). Use sentence case with clear size and weight hierarchy.
11. **No Emoji in the Nav:** Navigation items (`Test`, `Practice`, `Progress`, `Settings`, `Login`) are plain text only.

### Layout, Hierarchy & Components
12. **Strict 4-Point Grid:** All margins, paddings, gaps, heights, and container widths must be multiples of `4px` (`4, 8, 12, 16, 20, 24, 32, 48, 64px`).
13. **No "Centred Everything":** Do not center-align every section. Use left-aligned editorial layouts for navigation, mode toolbars, results tables, and progress dashboards. Constrain the typing arena to a readable width (`72ch–80ch`), aligned cleanly with the page container.
14. **One Main Button:** Every screen has a single primary CTA button (e.g., `Next Test` on Results, `Sync Account` on Progress). Secondary actions use plain text links or subtle bordered buttons.
15. **No Pill Above Headline:** Never place eyebrow badges (e.g., `v1.0 • Free`) above page titles.
16. **No Fake Reviews:** Zero fabricated testimonials, user quotes, or star ratings.
17. **No Stat Banners:** Zero vanity banners (e.g., *"100,000+ tests completed"*).
18. **No Icon Boxes:** Never place icons inside tinted rounded-square boxes next to headings or settings rows.
19. **Cards Must Say Something:** Never use empty or decorative cards. Every container using `--bg-container` or `--bg-secondary` must display live functional data (the typing arena, a WPM chart, a weak-character table, or the contribution calendar).
20. **No Bouncing Animations:** Ban bobbing elements, bouncing arrows, and spring-overshoot transitions. Motion is limited to a linear `60ms–80ms` caret glide and instant state transitions.

---

## 3. Color System & CSS Variables

Both themes are implemented via CSS custom properties on `:root` (Light Mode default) and `[data-theme="dark"]` (Tokyo Night).

### Light Mode — Warm Retro (Default)

| UI Element / Role | Hex Code | CSS Variable | Usage in App |
| --- | --- | --- | --- |
| **Website Background** | `#FFEEB3` | `--bg-main` | Main page canvas |
| **Typing Test Container** | `#B4D7CD` | `--bg-container` | Primary typing arena & swatch panel |
| **Secondary Cards** | `#8EA8C3` | `--bg-secondary` | Toolbar segments, telemetry stat containers & footer shortcut hints |
| **Buttons / CTA & Caret** | `#F47321` | `--accent`, `--caret` | Active mode state, primary CTA, focus ring & typing cursor |
| **Main Text** | `#29343B` | `--text-main` | Headings, active metric values & primary copy |
| **Muted Text** | `#64717A` | `--text-muted` | Untyped characters, labels & keyboard shortcut hints |
| **Correct Typing** | `#387D65` | `--correct` | Correctly typed characters |
| **Incorrect Typing** | `#C94C42` | `--incorrect` | Mistyped & extra overflow characters |

### Dark Mode — Tokyo Night

| UI Element / Role | Hex Code | CSS Variable | Usage in App |
| --- | --- | --- | --- |
| **Main Background** | `#1A1B26` | `--bg-main` | Main page canvas |
| **Typing Test Container** | `#24283B` | `--bg-container` | Primary typing arena & swatch panel |
| **Secondary Cards** | `#1F2335` | `--bg-secondary` | Toolbar segments, telemetry stat containers & footer shortcut hints |
| **Primary Accent / CTA** | `#7AA2F7` | `--accent` | Active mode state, primary CTA & focus highlights |
| **Cursor / Caret** | `#E0AF68` | `--caret` | Typing caret |
| **Primary Text** | `#C0CAF5` | `--text-main` | Headings, active metric values & primary copy |
| **Muted Text** | `#565F89` | `--text-muted` | Untyped characters, labels & keyboard shortcut hints |
| **Correct Characters** | `#9ECE6A` | `--correct` | Correctly typed characters |
| **Incorrect Characters** | `#F7768E` | `--incorrect` | Mistyped & extra overflow characters |

### CSS Token Implementation (`src/app/globals.css`)

```css
:root {
  --bg-main: #FFEEB3;
  --bg-container: #B4D7CD;
  --bg-secondary: #8EA8C3;
  --accent: #F47321;
  --caret: #F47321;
  --text-main: #29343B;
  --text-muted: #64717A;
  --correct: #387D65;
  --incorrect: #C94C42;
  --radius-sm: 2px;
  --radius-md: 4px;
}

[data-theme="dark"] {
  --bg-main: #1A1B26;
  --bg-container: #24283B;
  --bg-secondary: #1F2335;
  --accent: #7AA2F7;
  --caret: #E0AF68;
  --text-main: #C0CAF5;
  --text-muted: #565F89;
  --correct: #9ECE6A;
  --incorrect: #F7768E;
}

body {
  background-color: var(--bg-main);
  color: var(--text-main);
  font-family: var(--font-sans), system-ui, sans-serif;
}
```

---

## 4. Typography

Load fonts via `next/font/google` in `src/app/layout.tsx` to guarantee zero layout shift:

* **Monospace (`--font-mono`):** `IBM Plex Mono` (weights `400`, `500`, `600`) — used for the typing arena, code snippets, live WPM/accuracy readouts, and keyboard shortcut hints.
* **UI Sans (`--font-sans`):** `IBM Plex Sans` (weights `400`, `500`, `600`) — used for navigation, mode labels, settings, and headings.

### Type Scale (4-Point Aligned)
* **Page Heading:** `24px` (`1.5rem`), weight `600`, line-height `32px`, sentence case.
* **Typing Arena Text:** `20px` (`1.25rem`) default (user-configurable `16px–28px`), `IBM Plex Mono`, line-height `32px` (`1.6`).
* **Primary Metric (WPM / Accuracy):** `32px` (`2rem`), `IBM Plex Mono`, weight `600`, `tabular-nums`.
* **UI Body / Controls:** `14px` (`0.875rem`), weight `400`–`500`, line-height `20px`.
* **Secondary / Hints:** `12px` (`0.75rem`), sentence case, `--text-muted`.

---

## 5. Layout & 4-Point Spacing System

* **Max Container Width:** `960px` (`60rem`), horizontal padding `24px` on desktop, `16px` on mobile.
* **Alignment:** Left-aligned editorial structure. The navbar, mode toolbar, typing arena, and footer hints share a single left vertical axis.
* **Spacing Scale:** Strictly multiples of `4px`:
  * Tight element gaps (between inline controls): `8px` (`gap-2`)
  * Component internal padding: `16px` or `24px` (`p-4` / `p-6`)
  * Section vertical rhythm: `24px` or `32px` (`space-y-6` / `space-y-8`)

---

## 6. Component Specifications

### 6.1 Navigation Bar
* Left: Plain text wordmark `justType` (`IBM Plex Mono`, `18px`, weight `600`) + left-aligned links (`Test`, `Practice`, `Progress`).
* Right: `Settings`, Theme toggle (`Light` / `Dark` plain text or crisp 16px monochrome icon without a box), and optional `Login`.
* Zero emojis, zero background blur, zero sticky drop shadows.

### 6.2 Mode & Difficulty Toolbar
* Flat container using `--bg-secondary` with `4px` border-radius (`rounded-[4px]`) and `4px` internal padding.
* Segmented rectangular buttons (`rounded-[2px]`) for Mode (`Mixed`, `Code`, `Terminal`), Difficulty (`Easy`, `Medium`, `Hard`, `Expert`), and Duration (`15s`, `30s`, `60s`, `120s`).
* Active state: solid `--accent` background with high-contrast text (or crisp `2px` bottom border in `--accent` with `--text-main` text). Never `rounded-full`.

### 6.3 Typing Arena (`TypingCanvas`)
* Container: `--bg-container`, `24px` padding (`p-6`), `4px` border-radius (`rounded-[4px]`).
* Character States:
  * **Untyped:** `--text-muted`
  * **Correct:** `--correct`
  * **Incorrect:** `--incorrect` + subtle `1px` bottom underline (`text-decoration: underline`) so errors remain distinguishable for colorblind users.
  * **Extra overflow characters:** `--incorrect` at `80%` opacity.
* Caret: `2px` wide vertical bar in `--caret`, positioned via `transform: translate3d()` with `transition: transform 70ms linear`. No glow, no bounce.
* Code Mode: Preserves newlines and monospace indentation; pressing `Enter` skips leading whitespace cleanly.

### 6.4 Results Screen
* Left-aligned summary grid using `--bg-secondary` containers (`rounded-[4px]`, `p-4`): WPM, Raw WPM, Accuracy, Consistency.
* WPM-over-time line chart (flat lines, no gradient area fills, no glowing dots).
* **Weakest Characters Table:** Left-aligned table listing the user's 3 lowest-accuracy categories (e.g., `Curly Braces {} — 74%`).
* **Actions:** One primary button (`Next Test` in `--accent`, `rounded-[4px]`) alongside plain secondary links (`Retry`, `Drill weak keys`).

### 6.5 Activity Calendar (52-Week Grid)
* Rendered inside a `--bg-container` panel (`p-6`, `rounded-[4px]`).
* Individual day tiles: `12px × 12px` with `2px` border-radius and `4px` gap.
* Hover tooltip: Flat `--bg-secondary` surface, `1px solid` muted border, `4px` radius, showing date, tests completed, correct characters, time practiced, and average WPM.

---

## 7. Next.js Theme Initialization (No-Flash Rule)

Because Light Mode (`Warm Retro`) is the default, inject a minimal synchronous script in `src/app/layout.tsx` before React hydrates so users who selected `Tokyo Night` never experience a white/yellow flash on page reload:

```tsx
<script
  dangerouslySetInnerHTML={{
    __html: `
      (function() {
        const saved = localStorage.getItem('justtype-theme');
        if (saved === 'dark') {
          document.documentElement.setAttribute('data-theme', 'dark');
        }
      })();
    `,
  }}
/>
```

---

## 8. Accessibility & Reduced Motion

* **Focus States:** Visible `2px solid var(--accent)` outline with `2px` offset on keyboard navigation (`:focus-visible`). Never remove focus rings without replacing them.
* **Color Independence:** Typing errors pair `--incorrect` with an underline; active states pair color with font-weight or border changes.
* **Reduced Motion:** Respect `@media (prefers-reduced-motion: reduce)` by setting caret transition duration to `0ms`.