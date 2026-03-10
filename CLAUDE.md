# CLAUDE.md — TürkRadyo (Radyo-Yeni)

## Project Overview

**TürkRadyo** is a zero-dependency, single-file Turkish radio streaming web application. The entire codebase lives in one file: `index.html` (~1018 lines). It is a mobile-first, PWA-capable app written in vanilla JavaScript with embedded CSS and HTML.

- **Version**: 10.0
- **Language**: Turkish UI
- **No build step, no package manager, no frameworks**

---

## Running the Project

There are no build or install steps. Open `index.html` directly in a browser, or serve it with any static HTTP server:

```bash
python -m http.server 8080
# or
npx http-server .
```

No `npm install`, no compilation, no transpilation needed.

---

## File Structure

```
Radyo-Yeni/
├── index.html      # Entire application (HTML + CSS + JS in one file)
└── README.md       # Minimal project title only
```

### `index.html` Layout (by line range)

| Section | Approximate Lines | Contents |
|---------|-------------------|----------|
| `<head>` / meta | 1–13 | PWA meta tags, Google Fonts (`Plus Jakarta Sans`, `Outfit`) |
| CSS styles | 14–494 | Full design system, all component styles |
| HTML structure | 495–850 | App shell, nav, header, pages, modals, audio element |
| JavaScript | 851–1018 | All application logic |

---

## Architecture

### Single-file pattern
All CSS, HTML, and JS are embedded in `index.html`. When editing:
- CSS changes go in the `<style>` block near the top.
- HTML structure changes go in `<body>`.
- JS changes go in the `<script>` block at the bottom.

### State object (`S`)
The global state is a plain object:
```js
S = {
  cur,      // currently selected channel object
  playing,  // boolean
  should,   // boolean — whether audio should be playing
  retries,  // number of reconnect attempts
}
```

### Data persistence (localStorage)
All data is stored client-side in `localStorage` with short keys:

| Key | Contents |
|-----|----------|
| `trch8` | `Array<{id, n, g, u, e, c}>` — channel list |
| `trfv8` | `Array<string>` — favorite channel IDs |
| `trrc8` | `Array<{id, t}>` — recently played (with timestamp) |
| `trint9` | Object — interrupt manager settings |

Channel object fields: `id` (string), `n` (name), `g` (genre), `u` (url), `e` (emoji), `c` (color).

### Key objects / modules

| Name | Purpose |
|------|---------|
| `S` | Global playback state |
| `IM` | InterruptManager — handles phone calls, notifications, background audio |
| `IOS` | iOS recovery — handles network stalls, visibility changes, orientation |

---

## CSS Design System

CSS custom properties are defined on `:root`. Always use these variables — never hardcode colors or spacing.

### Color tokens
```css
--bg, --bg2, --bg3, --bg4     /* background layers (dark) */
--ac                           /* primary accent: #7c6cf0 (purple) */
--ac2                          /* light accent: #b4a9ff */
--ac3                          /* secondary accent: #ff6b9d (pink) */
--ac4                          /* tertiary: #ff9a76 (orange) */
--tx, --tx2, --tx3             /* text: primary / secondary / muted */
--ok, --no, --warn             /* semantic: #3dd68c / #ff5c6c / #ffc857 */
--cd, --cd2, --cd3             /* card backgrounds (rgba white overlays) */
--bd, --bd2                    /* border colors */
--gl, --gl2                    /* glow colors */
```

### Other tokens
```css
--font, --font-display         /* type families */
--radius, --radius-sm, --radius-xs   /* 16px / 12px / 8px */
--shadow, --shadow-glow        /* box-shadow presets */
--transition                   /* cubic-bezier(.22,1,.36,1) */
--st, --sb                     /* safe-area-inset-top / bottom for notch support */
```

---

## Naming Conventions

The codebase uses a minimalist/golf style. Follow these patterns when adding code:

- **DOM element variables**: single letters — `d`, `f`, `h`, `r`, `s`, `x`
- **Private/internal state**: prefixed with `_` — `_curPage`, `_filterGenre`, `_searchQ`, `_sr`
- **Data object keys**: single letters — `n` (name), `u` (url), `g` (genre), `e` (emoji), `c` (color), `t` (timestamp)
- **Abbreviated module names**: `ch` (channels), `fv` (favorites), `rc` (recent)
- **CSS classes**: short, lowercase, hyphenated — `.hdr`, `.abtn`, `.tst`, `.itr`
- **Hidden/active state**: class `h` = hidden, `s` = shown/active

---

## UI Navigation

The app has 4 pages controlled by a bottom tab bar:

| Tab | Page ID | Contents |
|-----|---------|----------|
| Favorites | `F` | Channels marked as favorites |
| All Channels | `A` | Complete channel list |
| Recent | `R` | Recently played channels |
| Settings | `S` | Data import/export, interrupt config, about |

Page switching is done by toggling a CSS class on `.page-*` containers; do not use `display:none/block` directly.

---

## External APIs

### Radio Browser API
- Base URLs (tried in parallel via `Promise.any()`): `de1`, `nl1`, `at1`, `de2` mirrors at `radio-browser.info`
- Timeout per request: 6 seconds
- Endpoints used:
  - `GET /json/stations/search?name=<query>&hidebroken=true&order=clickcount&reverse=true`
  - `GET /json/stations/search?tag=<genre>&hidebroken=true&order=clickcount&reverse=true`
- Turkish stations are fetched first with `countrycode=TR`, then global results are appended and deduplicated.

### Web Platform APIs used
- **Web Audio API** — audio context management
- **Media Session API** — lock screen controls and artwork
- **File API** — JSON import/export
- **localStorage** — all persistence

---

## Key Behaviors

### Audio retry logic
On stream failure, the player retries up to 4 times with increasing delays. Managed in `S.retries`.

### Interrupt handling (`IM`)
Detects and responds to:
- Phone calls — pauses playback
- Notification sounds — optionally ducks volume
- App backgrounding — optionally reduces volume
Configurable via the Settings page and persisted in `trint9`.

### iOS quirks (`IOS`)
Handles:
- Network stall recovery on iOS
- `visibilitychange` and `focus`/`blur` events
- `deviceorientationchange` resume logic

### Toast notifications
Shown via a helper function; auto-dismiss after 2.6s. Use the existing toast element (`.tst`), do not create new notification elements.

### Confirm dialogs
Destructive actions (delete, clear history) must use the existing confirm dialog pattern — do not use `window.confirm()`.

---

## Security Practices

- Always use `textContent` (not `innerHTML`) when inserting user-generated or external data into the DOM.
- URLs must pass the `isUrl()` validator (checks for `http://` or `https://` protocol) before use.
- No `eval()` or dynamic `Function()` construction.
- Sanitize all data loaded from localStorage before rendering.

---

## Animations & Transitions

- Prefer CSS animations/transitions over JS-driven animation.
- Use `--transition` (`cubic-bezier(.22,1,.36,1)`) for all interactive transitions.
- Interactive transitions: 200–400ms.
- Ambient/ambient-background animations: 8–20s duration.
- Use staggered `animation-delay` via `:nth-child()` for list card appearances.
- Glassmorphism: `backdrop-filter: blur(Xpx) saturate(Y%)` — match existing values.

---

## Development Workflow

### Branching
- Main branch: `master`
- Feature branches follow the pattern: `claude/<task-id>`

### Making changes
Since the entire app is one file, all edits are to `index.html`. Keep the three sections (CSS, HTML, JS) in their respective areas within the file.

### No linter / formatter
There is no ESLint, Prettier, or other tooling. Follow the existing minified/compact style for CSS and match the surrounding code style for JS.

### Testing
Manual browser testing only. No test framework is configured. Test:
1. Open `index.html` in Chrome/Safari (mobile emulation recommended)
2. Verify playback, favorites, search, and settings flows
3. Check iOS behavior if relevant

---

## Common Pitfalls

- **Do not add a build step** — the project is intentionally dependency-free.
- **Do not split into multiple files** — the single-file architecture is deliberate.
- **Do not use `innerHTML` with external data** — use `textContent` or sanitize first.
- **Do not hardcode colors** — always use CSS custom properties.
- **Do not use `window.confirm()`** — use the existing confirm dialog component.
- **Match the minified CSS style** — do not expand/pretty-print existing CSS rules.
- **Respect safe-area insets** — use `--st` / `--sb` vars for top/bottom spacing in fixed elements.
