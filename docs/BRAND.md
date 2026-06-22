# Brand Guide

A small, portable visual system for project READMEs — a denim‑blue + warm‑accent
identity on a dark ink panel, with an SVG horizontal logo and a matching square
icon. Copy this file into any repo and adapt the **project‑specific** bits
(wordmark, subtitle, icon glyph); keep the **system** bits (palette, panel,
typography, layout) the same so everything looks like one family.

The anchor color is **PANTONE 17‑4139 TCX "Niagara"** (a denim/teal blue,
≈ `#5B89A6`). The system deliberately avoids the teal‑green / purple‑gradient
"AI" look — the warmth comes from an earthy complementary accent, not a neon.

---

## 1. Color palette

| Role | Token | Hex | Notes |
|------|-------|-----|-------|
| **Primary — Niagara** | `niagara` | `#5B89A6` | Brand anchor (PANTONE 17‑4139 TCX). Used as the base of the node/primary gradient. |
| Primary gradient (top→bottom) | `primary-grad` | `#6E9DBA → #4E7C97` | Fills primary shapes. |
| Primary line / mesh | `primary-line` | `#4E7C97` | Connectors, hairlines. |
| Subtitle / mid blue | `mid` | `#7FA6BC` | Letter‑spaced subtitles, secondary text. |
| **Accent — Terracotta** | `accent` | `#C4683F` | The one warm pop. Wordmark accent word + rule + the "highlighted" shape. |
| Accent gradient (top→bottom) | `accent-grad` | `#C4683F → #974124` | Warm‑to‑brick; the highlighted shape. |
| Ink panel (top→bottom) | `ink` | `#1A2C39 → #0F1C25` | The banner background; gives a premium, theme‑independent surface. |
| Panel border | `border` | `#2C4C61` | 1.5–2px stroke that defines the panel edge. |
| Text — off‑white | `text` | `#EAF2F7` | Wordmark, light shapes. |
| Warm white | `warm-white` | `#FBEEE3` | A "core" dot sitting on the accent shape (subtle warmth tie‑in). |

**Accent variations.** Keep Niagara + the dark ink constant; vary only the warm
accent if a project wants its own flavor (all are warm complements of Niagara,
so they harmonize):

| Variant | Gradient | Flat |
|---------|----------|------|
| **Terracotta** (default) | `#C4683F → #974124` | `#C4683F` |
| Copper | `#DB9560 → #BE6C3A` | `#D2814E` |
| Amber / brass | `#CDA052 → #9C6F2E` | `#C5953F` |

> Rule of thumb: the accent should be a **warm hue (≈ 20–45°)** so it complements
> Niagara's ≈ 205°. Avoid greens and violets — that's where the "AI" cliché lives.

---

## 2. The panel

Every banner/icon sits on a self‑contained dark panel so it renders identically
in GitHub light **and** dark mode (no `prefers-color-scheme` gymnastics):

- Background: vertical linear gradient `ink` (`#1A2C39 → #0F1C25`).
- Border: `border` (`#2C4C61`), `stroke-width` 1.5 (banner) / 2 (icon).
- Corner radius: `28` on the banner panel, `48` on the square icon.

---

## 3. Typography

The wordmark is a **bold, uppercase** sans with one accent word:

- Font stack (no embedded fonts needed):
  `'Segoe UI','SF Pro Display',system-ui,-apple-system,'Helvetica Neue',Arial,sans-serif`
- Wordmark: `font-size` ~66, `font-weight` 800, `letter-spacing` 0.5, fill `text`.
  The **last token** (e.g. a model/series name) is filled with `accent`.
- Accent rule: a thin (`height` 3, `rx` 1.5) `accent` rectangle under the
  wordmark at ~0.8 opacity, separating it from the subtitle.
- Subtitle: `font-size` ~19, `font-weight` 600, `letter-spacing` ~4.4, fill `mid`,
  in `UPPERCASE · WITH · MIDDOT · SEPARATORS`.

> SVG `<text>` collapses leading whitespace, so put a gap **before** the accent
> tspan with `dx="20"`, not a literal space.

---

## 4. Logo lockup & README layout

Match this order at the top of every README (the same as the reach/sisyphus
repos):

```markdown
# Project Name

![Project Name](docs/assets/<project>/logos/<project>_logo_horizontal.svg)

[![CI](...)](...) [![License](...)](...) [![…](...)](...)

One‑line description of the project.
```

**Asset paths / naming** (keep this convention across repos):

```
docs/assets/<project>/
├── logos/<project>_logo_horizontal.svg   # banner used in the README
└── icons/<project>_icon.svg              # square mark (favicon/social)
```

Lockup geometry: **icon on the left, wordmark + subtitle to its right**, both
vertically centered on the panel. Balance the side margins (~64–85px each).

---

## 5. Iconography (per‑project)

The **glyph is bespoke per project** — it should say something about what the
project *is*. Keep it inside the same dark panel, use `primary-grad` for the
"normal" elements and `accent-grad` for the one "special" element, and add small
`text`/`warm-white` core dots for life.

- This repo (a 4‑node cluster) → a **2×2 node grid**, control‑plane node in
  terracotta, workers in Niagara, joined by a faint interconnect mesh.
- A remote‑access tool → an antenna / signal arc reaching a node.
- A workspace/build tool → stacked layers or a boulder‑on‑incline silhouette.

Keep it geometric and legible at 32px. One accent element only.

---

## 6. Reusable SVG templates

Copy, then change the three marked spots: **PROJECT WORDMARK**, the **accent word
fill stays `#C4683F`**, the **subtitle**, and (optionally) the **accent hex** if
using a variant. Replace the icon group with your project glyph.

### Horizontal banner — `docs/assets/<project>/logos/<project>_logo_horizontal.svg`

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 800 240" width="800" height="240" role="img" aria-label="PROJECT NAME">
  <defs>
    <linearGradient id="bg" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0" stop-color="#1A2C39"/><stop offset="1" stop-color="#0F1C25"/>
    </linearGradient>
    <linearGradient id="node" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0" stop-color="#6E9DBA"/><stop offset="1" stop-color="#4E7C97"/>
    </linearGradient>
    <linearGradient id="cp" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0" stop-color="#C4683F"/><stop offset="1" stop-color="#974124"/>
    </linearGradient>
  </defs>

  <rect x="1" y="1" width="798" height="238" rx="28" fill="url(#bg)" stroke="#2C4C61" stroke-width="1.5"/>

  <!-- ICON: replace this group with your project glyph (use url(#node) / url(#cp)) -->
  <g transform="translate(80,44)">
    <g stroke="#4E7C97" fill="none" stroke-linecap="round">
      <g opacity="0.5" stroke-width="3">
        <line x1="30" y1="30" x2="114" y2="30"/><line x1="30" y1="114" x2="114" y2="114"/>
        <line x1="30" y1="30" x2="30" y2="114"/><line x1="114" y1="30" x2="114" y2="114"/>
      </g>
      <g opacity="0.26" stroke-width="2">
        <line x1="30" y1="30" x2="114" y2="114"/><line x1="114" y1="30" x2="30" y2="114"/>
      </g>
    </g>
    <g stroke="#EAF2F7" stroke-opacity="0.10" stroke-width="1">
      <rect x="0" y="0" width="60" height="60" rx="13" fill="url(#cp)"/>
      <rect x="84" y="0" width="60" height="60" rx="13" fill="url(#node)"/>
      <rect x="0" y="84" width="60" height="60" rx="13" fill="url(#node)"/>
      <rect x="84" y="84" width="60" height="60" rx="13" fill="url(#node)"/>
    </g>
    <g>
      <circle cx="30" cy="30" r="6" fill="#FBEEE3" opacity="0.92"/>
      <circle cx="114" cy="30" r="6" fill="#EAF2F7" opacity="0.85"/>
      <circle cx="30" cy="114" r="6" fill="#EAF2F7" opacity="0.85"/>
      <circle cx="114" cy="114" r="6" fill="#EAF2F7" opacity="0.85"/>
    </g>
  </g>

  <!-- WORDMARK: change text + subtitle; keep accent tspan fill #C4683F.
       Match the rule's width to your wordmark; preview and nudge. -->
  <g font-family="'Segoe UI','SF Pro Display',system-ui,-apple-system,'Helvetica Neue',Arial,sans-serif">
    <text x="316" y="116" font-size="66" font-weight="800" letter-spacing="0.5" fill="#EAF2F7">ACME<tspan fill="#C4683F" dx="20">OPS</tspan></text>
    <rect x="318" y="134" width="318" height="3" rx="1.5" fill="#C4683F" opacity="0.8"/>
    <text x="320" y="168" font-size="19" font-weight="600" letter-spacing="4.4" fill="#7FA6BC">SHORT · STACK · SUBTITLE</text>
  </g>
</svg>
```

### Square icon — `docs/assets/<project>/icons/<project>_icon.svg`

Same `defs`, a `240×240` panel with `rx="48"`, and just the centered glyph
(no wordmark). See `docs/assets/turing-cluster/icons/turing_cluster_icon.svg`
in this repo for a worked example.

---

## 7. Badges

Use shields.io flat badges in this order under the logo: **CI status → license →
project‑specific (version/compat) → release**. Conventions used here:

- Live "current version" from a JSON manifest (no hardcoded string to go stale):
  `https://img.shields.io/badge/dynamic/json?url=<raw manifest URL>&query=$.path.to.version&label=Label&color=blue`
- Release by highest tag (robust to out‑of‑order tags):
  `https://img.shields.io/github/v/release/<owner>/<repo>?sort=semver&color=blue`
- Prefer `&color=blue` to sit in the Niagara family; reserve other colors for
  semantic states (green = pass, yellow = license, etc.).

---

## 8. Tooling & checks

- **Preview before commit.** Render SVG → PNG and look at it:
  `rsvg-convert -z 2 -b '#ffffff' logo.svg -o /tmp/logo.png`
  (also try `-b '#0d1117'` for GitHub dark). `inkscape`, `resvg`, and
  `cairosvg` work too.
- **Validate** it's well‑formed: `python3 -c "import xml.dom.minidom as m; m.parse('logo.svg')"`.
- Keep text as `<text>` (not paths) so wordmarks stay editable — accept that the
  exact font depends on the viewer's system; the bold sans stack renders cleanly
  on all common platforms.

### New‑project checklist

1. Copy the banner template → `docs/assets/<project>/logos/<project>_logo_horizontal.svg`.
2. Swap the wordmark + subtitle (keep the accent tspan); pick an accent variant if desired.
3. Replace the icon group with a project glyph; make a square `…_icon.svg` to match.
4. Preview with `rsvg-convert`; nudge wordmark `x` and the rule `width`.
5. Add to the README as `# Title` → logo → badges → description.

---

*Palette anchored on PANTONE 17‑4139 TCX (Niagara). Reference implementation:
the banner/icon under `docs/assets/turing-cluster/`.*
