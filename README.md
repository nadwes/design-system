# DYN Sports — Design System

A scaleable, white-label design system built initially for DYN Sports, with architecture to support multiple brands.

## What's in this repo

| File / Folder | Description |
|---|---|
| `design-system-preview.html` | Visual token reference — open in any browser, no build step needed |
| `.kiro/specs/design-system/requirements.md` | Full requirements document (10 requirement areas) |
| `.kiro/specs/design-system/design.md` | Technical design document — token architecture, grid system, media ratios |
| `.kiro/specs/design-system/tasks.md` | Implementation task list |
| `Figma Design Tokens Nadine/` | Exported Figma token JSON files (primitives, colors, typography, spacing, stroke, effects) |

## Quick start — preview

Just open `design-system-preview.html` in your browser. No server, no dependencies.

The preview covers:
- Color primitives & semantic roles (Dark Mode)
- Sport-specific colors (Handball, Basketball, Volleyball, Table Tennis, Hockey)
- Typography scale (Bebas Neue / Oswald / Titillium Web)
- Spacing scale
- Stroke & border radius tokens
- Effects & overlays
- Grid system & layout (breakpoints, content wrapper, media aspect ratios)
- WCAG accessibility contrast checks
- Token hierarchy (Primitive → Global → Component)
- Pricing card component tokens

## Token structure

Tokens are organized in three layers:

```
Primitive  →  Global (Dark Mode)  →  Component
blue500       brand/primary/regular   card/pricing/standard/action-button
#006299       #006299                 #006299
```

## Fonts

The DYN design system uses three typefaces:
- **Bebas Neue** — headings (h1–h3, h5, h7), interaction/buttons
- **Oswald** — display headings (h4, h6)
- **Titillium Web** — body copy (p1, p2, p3)

## Grid system

| Breakpoint | Tailwind prefix | Min width | Columns | Wrapper padding |
|---|---|---|---|---|
| Mobile | _(default)_ | 0px | 4 | 16px |
| Tablet | `md:` | 768px | 8 | 24px |
| Desktop | `xl:` | 1280px | 12 | 48px |
| Wide | `2xl:` | 1536px | 12 | 48px (content frozen) |

Content wrapper (no custom Tailwind config needed):
```html
<div class="mx-auto w-full max-w-screen-2xl px-4 md:px-6 xl:px-12">
  <!-- content -->
</div>
```

## Media aspect ratios

| Token | Ratio | Min source resolution | Use case |
|---|---|---|---|
| `media.ratio.hero` | 16:9 | 1536 × 864px | Full-bleed hero video |
| `media.ratio.hero-wide` | 21:9 | 1536 × 659px | Cinematic hero |
| `media.ratio.player` | 16:9 | 1920 × 1080px | Inline video player |
| `media.ratio.thumbnail` | 16:9 | 960 × 540px | Match/event cards |
| `media.ratio.thumbnail-square` | 1:1 | 480 × 480px | Avatars, badges |
| `media.ratio.editorial` | 3:2 | 1200 × 800px | Article headers |
| `media.ratio.split-screen` | 1:1 / 50svw×100svh | 960 × 960px | Split-screen hero |

## Figma token files

Exported from Figma using the Tokens Studio plugin:

| File | Contents |
|---|---|
| `primitives/DYN.tokens.json` | Raw color palette, font sizes, spacing scale, font weights |
| `colors-global/Dark Mode.tokens.json` | Semantic color roles for dark mode |
| `colors-component-tokens/Mode 1.tokens.json` | Component-level color tokens (pricing cards, etc.) |
| `effects-global/DYN - Dark Mode.tokens.json` | Overlay and glass effect tokens |
| `spacings-global/DYN.tokens.json` | Component spacing tokens (header, card, navigation) |
| `stroke-global/DYN.tokens.json` | Border widths and corner radius tokens |
| `typos-global/DYN.tokens.json` | Typography style tokens (h1–h7, p1–p3, interaction) |

## Status

This design system is in active development. The token definitions are complete for the DYN Dark Mode. Implementation tasks are tracked in `.kiro/specs/design-system/tasks.md`.
