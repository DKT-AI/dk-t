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
`DESIGN.md`.
