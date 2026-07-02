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
