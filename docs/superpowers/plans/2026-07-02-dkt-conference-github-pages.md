# DKT Conference — GitHub Pages + dk-t.dev Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish the DKT Conference one-pager as static files on GitHub Pages at the custom apex domain `dk-t.dev`, with self-hosted dependencies and a maintainer guide for regular content edits.

**Architecture:** The source is a design-tool export (`.dc.html`) rendered by a client-side React runtime (`support.js`). We copy only the needed files into the repo, self-host React + fonts (removing the CDN blank-page risk), add GitHub Pages plumbing (`.nojekyll`, `CNAME`), rename the entry to `index.html`, then enable Pages + attach the domain via the GitHub API. DNS is configured by the user in Squarespace.

**Tech Stack:** Static HTML/CSS/JS · React 18.3.1 UMD (self-hosted) · GitHub Pages (classic branch-deploy) · `gh` CLI + REST API · Playwright (render verification).

**Source folder (read-only origin):** `/Users/viktor/Downloads/One-Pager Conference`
**Repo (cwd):** `/Users/viktor/Documents/GitHub/DKT/dk-t` — remote `git@github.com:DKT-AI/dk-t.git`, admin confirmed.

**Reference spec:** `docs/superpowers/specs/2026-07-02-dkt-conference-github-pages-design.md`

---

## File Structure

Created/modified in the repo root:

| Path | Responsibility |
|---|---|
| `index.html` | The page (renamed from `DKT Conference.dc.html`); head gets favicon + self-hosted `@font-face`; "Генеральный партнёр"→"Партнёр" |
| `support.js` | dc-runtime; React URLs repointed to `vendor/` |
| `image-slot.js` | image-slot component (verbatim copy) |
| `.image-slots.state.json` | speaker photo, base64 (verbatim copy) |
| `vendor/react.production.min.js`, `vendor/react-dom.production.min.js` | self-hosted React 18.3.1 UMD |
| `assets/dkt-logo.png`, `assets/slurm-logo-white.svg`, `assets/slurm-logo.svg` | images (verbatim copy) |
| `assets/fonts/*.woff2` | self-hosted Hanken Grotesk + JetBrains Mono |
| `.nojekyll` | disables Jekyll so dotfiles are served |
| `CNAME` | `dk-t.dev` |
| `CLAUDE.md` | content-editing guide (source-of-truth map + traps) |
| `DESIGN.md` | DKT design tokens + component patterns |
| `README.md` | repo description |
| `.gitignore` | ignore `.playwright-mcp/`, screenshots |

**Not copied:** `_ds/` (personal brand), `.thumbnail`, `uploads/`.

---

## Task 1: Copy needed source files into the repo

**Files:**
- Create: `index.html`, `support.js`, `image-slot.js`, `.image-slots.state.json`, `assets/dkt-logo.png`, `assets/slurm-logo-white.svg`, `assets/slurm-logo.svg`

- [ ] **Step 1: Copy the five needed files + assets (rename HTML → index.html)**

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
SRC="/Users/viktor/Downloads/One-Pager Conference"
cp "$SRC/DKT Conference.dc.html" index.html
cp "$SRC/support.js" support.js
cp "$SRC/image-slot.js" image-slot.js
cp "$SRC/.image-slots.state.json" .image-slots.state.json
mkdir -p assets
cp "$SRC/assets/dkt-logo.png" "$SRC/assets/slurm-logo-white.svg" "$SRC/assets/slurm-logo.svg" assets/
```

- [ ] **Step 2: Verify the file set (no _ds/uploads/.thumbnail leaked)**

Run:
```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
ls -1 index.html support.js image-slot.js .image-slots.state.json assets/ && echo "---" && ls -d _ds uploads .thumbnail 2>/dev/null || echo "OK: no personal-brand dirs present"
```
Expected: the 4 root files + `assets/` listing (`dkt-logo.png`, `slurm-logo-white.svg`, `slurm-logo.svg`); then `OK: no personal-brand dirs present`.

- [ ] **Step 3: Commit**

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
git add index.html support.js image-slot.js .image-slots.state.json assets/
git commit -m "feat: import DKT Conference page (needed files only)"
```

---

## Task 2: Self-host React (remove unpkg CDN dependency)

**Files:**
- Create: `vendor/react.production.min.js`, `vendor/react-dom.production.min.js`
- Modify: `support.js:1567-1570`

- [ ] **Step 1: Download the exact React 18.3.1 UMD files unpkg served**

Downloading the identical bytes means the existing SRI hashes in `support.js` stay valid.

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
mkdir -p vendor
curl -fsSL "https://unpkg.com/react@18.3.1/umd/react.production.min.js" -o vendor/react.production.min.js
curl -fsSL "https://unpkg.com/react-dom@18.3.1/umd/react-dom.production.min.js" -o vendor/react-dom.production.min.js
```

- [ ] **Step 2: Verify SRI hashes match what support.js pins**

Run:
```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
echo "react:     sha384-$(openssl dgst -sha384 -binary vendor/react.production.min.js | openssl base64 -A)"
echo "react-dom: sha384-$(openssl dgst -sha384 -binary vendor/react-dom.production.min.js | openssl base64 -A)"
```
Expected — must match `support.js` exactly:
```
react:     sha384-DGyLxAyjq0f9SPpVevD6IgztCFlnMF6oW/XQGmfe+IsZ8TqEiDrcHkMLKI6fiB/Z
react-dom: sha384-gTGxhz21lVGYNMcdJOyq01Edg0jhn/c22nsx0kyqP0TxaV5WVdsSH1fSDUf5YJj1
```
If they differ, STOP — unpkg changed the file; do not proceed (SRI would block loading).

- [ ] **Step 3: Repoint the two React URLs in support.js to the local copies**

Edit `support.js` — change only the two URL string literals (lines ~1567 and ~1569); leave the SRI lines untouched.

Replace:
```js
  var REACT_URL = "https://unpkg.com/react@18.3.1/umd/react.production.min.js";
```
with:
```js
  var REACT_URL = "vendor/react.production.min.js";
```
And replace:
```js
  var REACT_DOM_URL = "https://unpkg.com/react-dom@18.3.1/umd/react-dom.production.min.js";
```
with:
```js
  var REACT_DOM_URL = "vendor/react-dom.production.min.js";
```

- [ ] **Step 4: Verify no unpkg React URLs remain**

Run:
```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
grep -n "unpkg.com/react" support.js || echo "OK: no unpkg React URLs left"
```
Expected: `OK: no unpkg React URLs left`. (A remaining `unpkg.com/@babel` line is fine — Babel only loads for sibling components this page doesn't use.)

- [ ] **Step 5: Commit**

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
git add vendor/ support.js
git commit -m "feat: self-host React 18.3.1, remove unpkg CDN dependency"
```

---

## Task 3: Self-host fonts (Hanken Grotesk + JetBrains Mono)

**Files:**
- Create: `assets/fonts/*.woff2` (9 files), `assets/fonts.css`
- Modify: `index.html` head (replace Google Fonts `<link>`s)

Weights actually used by the page: Hanken Grotesk 400/500/600/700/800/900, JetBrains Mono 400/500/700. Download exactly those (YAGNI).

- [ ] **Step 1: Download the woff2 files from fontsource (jsdelivr)**

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
mkdir -p assets/fonts
BASE="https://cdn.jsdelivr.net/npm/@fontsource"
for w in 400 500 600 700 800 900; do
  curl -fsSL "$BASE/hanken-grotesk@5/files/hanken-grotesk-latin-$w-normal.woff2" -o "assets/fonts/hanken-grotesk-latin-$w-normal.woff2"
done
for w in 400 500 700; do
  curl -fsSL "$BASE/jetbrains-mono@5/files/jetbrains-mono-latin-$w-normal.woff2" -o "assets/fonts/jetbrains-mono-latin-$w-normal.woff2"
done
```

- [ ] **Step 2: Verify 9 non-empty woff2 files exist**

Run:
```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
find assets/fonts -name '*.woff2' -size +1k | wc -l | tr -d ' '
```
Expected: `9`

- [ ] **Step 3: Create `assets/fonts.css` with @font-face rules**

Create `assets/fonts.css`:
```css
/* Self-hosted DKT fonts — replaces Google Fonts CDN. Paths relative to /index.html. */
@font-face { font-family: 'Hanken Grotesk'; font-style: normal; font-weight: 400; font-display: swap; src: url('assets/fonts/hanken-grotesk-latin-400-normal.woff2') format('woff2'); }
@font-face { font-family: 'Hanken Grotesk'; font-style: normal; font-weight: 500; font-display: swap; src: url('assets/fonts/hanken-grotesk-latin-500-normal.woff2') format('woff2'); }
@font-face { font-family: 'Hanken Grotesk'; font-style: normal; font-weight: 600; font-display: swap; src: url('assets/fonts/hanken-grotesk-latin-600-normal.woff2') format('woff2'); }
@font-face { font-family: 'Hanken Grotesk'; font-style: normal; font-weight: 700; font-display: swap; src: url('assets/fonts/hanken-grotesk-latin-700-normal.woff2') format('woff2'); }
@font-face { font-family: 'Hanken Grotesk'; font-style: normal; font-weight: 800; font-display: swap; src: url('assets/fonts/hanken-grotesk-latin-800-normal.woff2') format('woff2'); }
@font-face { font-family: 'Hanken Grotesk'; font-style: normal; font-weight: 900; font-display: swap; src: url('assets/fonts/hanken-grotesk-latin-900-normal.woff2') format('woff2'); }
@font-face { font-family: 'JetBrains Mono'; font-style: normal; font-weight: 400; font-display: swap; src: url('assets/fonts/jetbrains-mono-latin-400-normal.woff2') format('woff2'); }
@font-face { font-family: 'JetBrains Mono'; font-style: normal; font-weight: 500; font-display: swap; src: url('assets/fonts/jetbrains-mono-latin-500-normal.woff2') format('woff2'); }
@font-face { font-family: 'JetBrains Mono'; font-style: normal; font-weight: 700; font-display: swap; src: url('assets/fonts/jetbrains-mono-latin-700-normal.woff2') format('woff2'); }
```

- [ ] **Step 4: Replace the Google Fonts links in index.html head**

In `index.html`, replace these three lines (inside `<helmet>`, ~lines 11-13):
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Hanken+Grotesk:wght@400;500;600;700;800;900&family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
```
with:
```html
<link rel="stylesheet" href="assets/fonts.css">
```

- [ ] **Step 5: Verify no Google Fonts references remain**

Run:
```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
grep -n "fonts.googleapis.com\|fonts.gstatic.com" index.html || echo "OK: no Google Fonts references left"
```
Expected: `OK: no Google Fonts references left`

- [ ] **Step 6: Commit**

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
git add assets/fonts assets/fonts.css index.html
git commit -m "feat: self-host Hanken Grotesk + JetBrains Mono fonts"
```

---

## Task 4: Content edits (rename partner tier + favicon)

**Files:**
- Modify: `index.html` (partner pill text ~line 224; head favicon)

- [ ] **Step 1: Rename "Генеральный партнёр" → "Партнёр"**

In `index.html`, replace the pill text `Генеральный партнёр` with `Партнёр` (the `<span>` above the Слёрм name, ~line 224).

- [ ] **Step 2: Add favicon link in head**

In `index.html`, add this line immediately after `<link rel="stylesheet" href="assets/fonts.css">` (inside `<helmet>`):
```html
<link rel="icon" href="assets/dkt-logo.png">
```

- [ ] **Step 3: Verify both edits landed**

Run:
```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
grep -c "Генеральный партнёр" index.html | xargs -I{} echo "old tier occurrences (want 0): {}"
grep -c 'rel="icon"' index.html | xargs -I{} echo "favicon links (want 1): {}"
grep -c ">Партнёр<" index.html | xargs -I{} echo "new tier occurrences (want >=1): {}"
```
Expected: old=0, favicon=1, new>=1.

- [ ] **Step 4: Commit**

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
git add index.html
git commit -m "content: rename partner tier to 'Партнёр'; add favicon"
```

---

## Task 5: GitHub Pages plumbing files

**Files:**
- Create: `.nojekyll`, `CNAME`, `.gitignore`

- [ ] **Step 1: Create the three plumbing files**

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
touch .nojekyll
printf 'dk-t.dev\n' > CNAME
printf '.playwright-mcp/\n*.screenshot.png\ndkt-render-test.png\n' > .gitignore
```

- [ ] **Step 2: Verify CNAME content is exactly the apex domain**

Run:
```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
cat CNAME
```
Expected: a single line `dk-t.dev` (no `https://`, no `www`, no trailing slash).

- [ ] **Step 3: Commit**

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
git add .nojekyll CNAME .gitignore
git commit -m "chore: add GitHub Pages plumbing (.nojekyll, CNAME, .gitignore)"
```

---

## Task 6: DESIGN.md — design tokens + patterns

**Files:**
- Create: `DESIGN.md`

- [ ] **Step 1: Write DESIGN.md**

Create `DESIGN.md`:
```markdown
# DKT Conference — Design System

Tokens and patterns for **this site**. Extracted from `index.html` (source of truth).
This is the DKT conference brand — NOT Viktor's personal "Deep Signal" system.
Any content edit must follow these values so the page stays on-brand.

## Colors

- Background gradient: `#0B547D → #073a54 → #062f44`; base text `#EAF6FF`
- Accent cyan `#43E1FD` — **swappable via theme switcher**:
  Бирюзовый `#43E1FD` / Малиновый `#E85872` / Жёлтый `#FED74E`
- Yellow CTA `#FEEB35` (text on it `#08304a`); secondary button raspberry `#E85872`
- Secondary text: `#BCDAEC`, `#90B9D0`, `#B9D9EC`

## Fonts (self-hosted in assets/fonts/)

- **Hanken Grotesk** — headings/body, weights 400–900
- **JetBrains Mono** — labels, time, nav; uppercase + letter-spacing

## Component patterns (what makes it look "DKT")

- **Overline label:** JetBrains Mono, ~13px, `letter-spacing: 0.2em` (0.18em in hero),
  `text-transform: uppercase`, color `var(--accent-soft)`.
- **Pill buttons `border-radius: 999px`:** primary = yellow `#FEEB35` bg / `#08304a`
  text / big shadow; secondary = raspberry `#E85872` / white text.
  Hover: `transform: translateY(-1..-2px)` + `filter: brightness(1.05)`.
- **Card:** `background: rgba(255,255,255,0.04)`, `border: 1px solid rgba(255,255,255,0.09)`,
  `border-radius: 22px`, `padding: 34px`; hover glow
  `box-shadow: 0 0 34px color-mix(... accent 16% ...)` + `translateY(-3px)`.
- **Status pill with dot:** inline-flex, dot `7px`/`border-radius:50%`, mono text,
  translucent accent bg + border.
- **Accent contract:** use `var(--accent, #43E1FD)` / `var(--accent-soft, …)` — do
  NOT hardcode cyan, or the theme switcher breaks. Deliberate hardcodes: yellow/raspberry
  decorative accents and the 2nd speaker card.
- **Rhythm:** sections `padding: 80px 0`, `max-width: 1180px`, side `padding: 0 32px`.
- **Radii:** `999px` pills, `50%` avatars/dots, `14/16/18/22px` cards.

## Tone

"A gathering of community friends" — warm, not corporate. Footer: «сделано с теплом
для комьюнити».
```

- [ ] **Step 2: Commit**

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
git add DESIGN.md
git commit -m "docs: add DKT design tokens + component patterns (DESIGN.md)"
```

---

## Task 7: CLAUDE.md — content-editing guide

**Files:**
- Create: `CLAUDE.md`

- [ ] **Step 1: Write CLAUDE.md**

Create `CLAUDE.md`:
```markdown
# DKT Conference site — editing guide

Static one-pager published to GitHub Pages at **https://dk-t.dev**. Source of truth
for design: `DESIGN.md`. This page is a **design-tool export** rendered by a
client-side React runtime (`support.js`), NOT plain HTML — text edits are safe,
structural edits need care. **After ANY edit: render locally and eyeball it before commit** (see bottom).

## How it renders

`index.html` contains `<x-dc>` template markup with `<sc-if>` blocks and `{{ vars }}`.
`support.js` loads self-hosted React (`vendor/`) and renders it. Fonts are self-hosted
(`assets/fonts/`). The speaker photo lives base64-encoded in `.image-slots.state.json`.
`.nojekyll` MUST stay (Jekyll would hide the dot-prefixed JSON → speaker photo breaks).

## Single source of truth — dates & URLs are DUPLICATED

Change EVERY occurrence or the page will disagree with itself:

| Fact | Where | Note |
|---|---|---|
| Countdown target | `renderVals()` in bottom `<script>`: `'2026-08-15T13:00:00+03:00'` | the ONLY value that moves the counter |
| Human date "15 августа 2026 · 13:00" | hero-center, hero-split, footer | cosmetic — must match the target above |
| Registration form `forms.gle/...` | header CTA, hero-center CTA, hero-split CTA | 3 places |
| Telegram `t.me/DevOpsKitchenTalks` | "Стать спикером", "Стать партнёром", footer | 4 places |
| Слёрм logo/link `slurm.io` | 2 links + description + tier pill | partner block |
| Podcast/social | Apple / Яндекс / Google Podcasts, YouTube, Instagram (footer) | Google Podcasts is discontinued |
| Hero subtitle | hero-center AND hero-split | edit both |
| Copyright year | footer | |

(Line numbers drift as you edit — search by the strings above, don't trust fixed lines.)

## Common edits

### Add a speaker (SPEAKERS section)
- The two example cards are DIFFERENT: card 1 (Alexander Dovnar) uses `<image-slot>`;
  card 2 ("Загадочный гость") is a hardcoded placeholder. **Copy the Dovnar card.**
- Give the new `<image-slot>` a UNIQUE `id` (`dkt-speaker-2`, …).
- **Photo:** put the file in `assets/` and set `src="assets/name.jpg"` on the
  `<image-slot>`. Drag-and-drop only works in the original design tool (needs
  `window.omelette`, absent on GitHub Pages) — you cannot drag from the repo.
- The photo in `.image-slots.state.json` overrides `src`. To switch slot 1 to a file,
  delete its key from that JSON.

### Edit the agenda (PROGRAM section)
- Each row is a time slot + label. Copy an existing row, change time and text.
  Keep the mono-font time style (see DESIGN.md overline pattern).

### Add a partner (PARTNERS section)
- Add the logo to `assets/`, copy the Слёрм block, update logo `src`, link, name,
  description, and the tier pill text.

### Change accent color / hero layout
- These come from `data-props` at the bottom (HTML-entity-encoded JSON — fragile).
  Prefer changing the accent via the `?? 'Бирюзовый'` fallback in `renderVals()`.
  Both hero variants (center / two-column) exist in the HTML; which shows is decided
  by `heroVariant` — leave the `<sc-if>` blocks alone.

## Verify before commit (REQUIRED)

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
python3 -m http.server 8765 --bind 127.0.0.1 &
# open http://127.0.0.1:8765/  — confirm the section you changed renders, no console errors
# then: kill the server
```

## Deploy

Push to `main` → GitHub Pages auto-deploys (classic branch-deploy). Never push a
`main` without the `CNAME` file (it would unset the custom domain). Design system:
`DESIGN.md`. Full deploy history/spec: `docs/superpowers/`.
```

- [ ] **Step 2: Commit**

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
git add CLAUDE.md
git commit -m "docs: add content-editing guide (CLAUDE.md)"
```

---

## Task 8: README.md

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write README.md**

Create `README.md`:
```markdown
# DKT Conference

Landing page for the DevOps Kitchen Talks conference — **https://dk-t.dev**
(15 августа 2026 · оффлайн + онлайн).

Static site on GitHub Pages. Self-hosted React + fonts; no external CDN at runtime.

## Local preview

```bash
python3 -m http.server 8765 --bind 127.0.0.1
# open http://127.0.0.1:8765/
```

## Editing content

See [`CLAUDE.md`](./CLAUDE.md) for the editing guide (speakers, agenda, partners,
dates) and [`DESIGN.md`](./DESIGN.md) for the design system. Push to `main` to deploy.
```

- [ ] **Step 2: Commit**

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
git add README.md
git commit -m "docs: add README"
```

---

## Task 9: Local render verification (Playwright)

**Files:** none (verification only)

- [ ] **Step 1: Serve the repo locally**

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
python3 -m http.server 8765 --bind 127.0.0.1 > /tmp/dkt-verify.log 2>&1 &
sleep 1.5
curl -sI http://127.0.0.1:8765/ | head -1
```
Expected: `HTTP/1.0 200 OK`

- [ ] **Step 2: Navigate + capture console errors + screenshot (Playwright MCP)**

Use Playwright MCP tools:
1. `browser_navigate` → `http://127.0.0.1:8765/`
2. `browser_wait_for` text `DKT`
3. `browser_console_messages` level `error` all=true
4. `browser_take_screenshot` fullPage=true

Expected: page renders (hero, live countdown, speakers with photo, agenda, partner
pill reads **Партнёр**, footer). Console: **no errors** (the earlier `favicon.ico` 404
is gone; no failed `unpkg`/`googleapis` requests — all local origin).

- [ ] **Step 3: Confirm no external runtime requests**

Verify via Playwright `browser_network_requests` that React and fonts loaded from
`127.0.0.1:8765/vendor/` and `127.0.0.1:8765/assets/fonts/` — NOT from `unpkg.com`
or `fonts.googleapis.com`.

- [ ] **Step 4: Stop the server**

```bash
pkill -f "http.server 8765" || true
```
If Step 2 or 3 shows errors or external requests, fix the offending task before continuing. No commit (verification only).

---

## Task 10: Push to GitHub

**Files:** none (git push)

- [ ] **Step 1: Confirm clean tree + review log**

Run:
```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
git status --short && echo "---" && git log --oneline
```
Expected: clean working tree; commits from Tasks 1-8 + the earlier spec commits.

- [ ] **Step 2: Push main to origin**

```bash
cd /Users/viktor/Documents/GitHub/DKT/dk-t
git push -u origin main
```
Expected: branch `main` pushed to `git@github.com:DKT-AI/dk-t.git`.

- [ ] **Step 3: Verify CNAME + .nojekyll are on the remote**

Run:
```bash
gh api repos/DKT-AI/dk-t/contents/CNAME --jq '.content' | base64 -d
gh api repos/DKT-AI/dk-t/contents/.nojekyll --jq '.name'
```
Expected: `dk-t.dev` then `.nojekyll`.

---

## Task 11: Enable GitHub Pages (classic branch-deploy)

**Files:** none (API)

- [ ] **Step 1: Enable Pages from main / root**

```bash
echo '{"source":{"branch":"main","path":"/"},"build_type":"legacy"}' \
  | gh api -X POST repos/DKT-AI/dk-t/pages --input -
```
Expected: JSON with `"status"` and a `"url"`. If it returns 409 "already exists", that's fine — Pages is already enabled. (Using `--input -` with a JSON body is the reliable way to send the nested `source` object; `-f 'source[branch]=…'` bracket syntax is unreliable.)

- [ ] **Step 2: Verify Pages is building**

Run:
```bash
gh api repos/DKT-AI/dk-t/pages --jq '{status:.status, html_url:.html_url, source:.source}'
```
Expected: `source` = branch `main` path `/`; `status` eventually `built` (may be `building` at first — re-run after ~1 min).

- [ ] **Step 3: Confirm the default Pages URL serves the page (pre-domain)**

Run (after status is `built`):
```bash
curl -sI "https://dkt-ai.github.io/dk-t/" | head -1
```
Expected: `HTTP/2 200`. (Assets use relative paths, so they resolve under `/dk-t/` too.)

---

## Task 12: Attach custom domain via API

**Files:** none (API)

- [ ] **Step 1: Set the custom domain (idempotent — CNAME already committed)**

```bash
echo '{"cname":"dk-t.dev","source":{"branch":"main","path":"/"}}' \
  | gh api -X PUT repos/DKT-AI/dk-t/pages --input -
```
Expected: HTTP 204 (no body) or 200. This confirms the already-committed `CNAME`.

- [ ] **Step 2: Verify the domain is registered on Pages**

Run:
```bash
gh api repos/DKT-AI/dk-t/pages --jq '{cname:.cname, https_enforced:.https_enforced, cert:.https_certificate.state}'
```
Expected: `cname` = `dk-t.dev`. `cert.state` will be `null`/`new`/`authorization_created` until DNS resolves (Task 13). Do NOT enable HTTPS enforcement yet.

- [ ] **Step 3: Print the domain-verification TXT record for the user**

Run:
```bash
echo "In GitHub → repo/org Settings → Pages → 'Verify domains', add the shown TXT record:"
echo "  Host:  _github-pages-challenge-DKT-AI"
echo "  (value is displayed in the GitHub UI — copy it into Squarespace DNS)"
```
This TXT is optional-but-recommended (takeover protection); include it in the DNS handoff.

---

## Task 13: DNS handoff to user (Squarespace) + verification loop

**Files:** none (user action + polling)

- [ ] **Step 1: Present the DNS records for the user to enter in Squarespace**

Tell the user to add, at Squarespace Domains → `dk-t.dev` → DNS Settings, and to
**delete any default Squarespace `@` A/ALIAS/ANAME/parking records first**:

- 4× **A**, host `@`: `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
- 4× **AAAA**, host `@`: `2606:50c0:8000::153`, `2606:50c0:8001::153`, `2606:50c0:8002::153`, `2606:50c0:8003::153`
- 1× **CNAME**, host `www` → `dkt-ai.github.io`
- 1× **TXT** (verification), host `_github-pages-challenge-DKT-AI` → value from GitHub UI (Task 12 Step 3)

Wait for the user to confirm the records are saved.

- [ ] **Step 2: Poll DNS propagation**

Run (repeat until it shows GitHub IPs):
```bash
dig +short dk-t.dev A
dig +short www.dk-t.dev CNAME
```
Expected: the four `185.199.10x.153` IPs; `www` → `dkt-ai.github.io.`

- [ ] **Step 3: Poll GitHub certificate issuance**

Run (repeat until `cert.state` = `approved`):
```bash
gh api repos/DKT-AI/dk-t/pages --jq '{cname:.cname, cert:.https_certificate.state, https_enforced:.https_enforced}'
```
Expected: eventually `cert` = `approved`. Can take minutes to a few hours.

- [ ] **Step 4: Enable HTTPS enforcement (only after cert = approved)**

```bash
echo '{"https_enforced":true,"cname":"dk-t.dev","source":{"branch":"main","path":"/"}}' \
  | gh api -X PUT repos/DKT-AI/dk-t/pages --input -
```
Expected: HTTP 204/200.

---

## Task 14: Final production verification

**Files:** none (verification)

- [ ] **Step 1: Verify HTTPS + redirects via curl**

Run:
```bash
curl -sI "https://dk-t.dev/" | head -1
curl -sI "http://dk-t.dev/" | grep -i "^location:" || echo "(http→https handled by HSTS)"
curl -sI "https://www.dk-t.dev/" | grep -iE "^(HTTP|location:)"
```
Expected: `https://dk-t.dev/` → `HTTP/2 200`; `www` → 301 redirect to apex (may lag until both domains verify).

- [ ] **Step 2: Final browser render on the live domain (Playwright MCP)**

1. `browser_navigate` → `https://dk-t.dev/`
2. `browser_wait_for` text `DKT`
3. `browser_console_messages` level `error` all=true → expect none
4. `browser_take_screenshot` fullPage=true

Expected: full page renders over valid TLS; countdown live; partner pill reads
**Партнёр**; no console errors; no external CDN requests.

- [ ] **Step 3: Confirm success criteria (from spec)**

Check each: (1) `dk-t.dev` valid TLS + HTTPS enforced; (2) full render no errors;
(3) React+fonts from own origin; (4) `www`→apex redirect; (5) `CLAUDE.md`+`DESIGN.md`
present; (6) `_ds/` not in repo (`gh api repos/DKT-AI/dk-t/contents/_ds` → 404 expected).

```bash
gh api repos/DKT-AI/dk-t/contents/_ds 2>&1 | grep -q "Not Found" && echo "OK: _ds not published" || echo "WARN: _ds present — remove it"
```

---

## Notes for the executor

- **Manual gate at Task 13:** requires the user to log into Squarespace. Pause and hand off the DNS records; don't attempt to automate Squarespace.
- **Async waits (Tasks 11-13):** Pages build, DNS propagation, and cert issuance are not instant. Poll; don't treat "not ready yet" as failure.
- **RTK/hook note:** `git`/`gh`/`curl` commands may be transparently proxied by the user's rtk hook — this is expected and fine.
- **Never** push a `main` lacking `CNAME` after Task 12, or the custom domain silently unsets.
