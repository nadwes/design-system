# Design Document: White-Label Design System

## Overview

A scaleable, white-label design system built initially for DYN Sports. The system is implemented as a TypeScript-first token library with a three-level token hierarchy, automated WCAG 2.1 AA accessibility validation, a Figma Variables sync pipeline, and a brand registry that supports onboarding new brands without structural changes.

The implementation is a Node.js/TypeScript package that can be consumed by any front-end framework. Tokens are stored as structured JSON/TypeScript objects, validated at build time, and exported in multiple formats (CSS custom properties, JavaScript/TypeScript ESM, and a Figma Variables JSON payload).

---

## Architecture

### High-Level Component Map

```
design-system/
├── src/
│   ├── tokens/
│   │   ├── base/          # Base_Token definitions (primitives)
│   │   ├── semantic/      # Semantic_Token definitions (references Base)
│   │   ├── component/     # Component_Token definitions (references Semantic)
│   │   └── schema.ts      # Token_Set schema (required keys)
│   ├── brands/
│   │   ├── registry.ts    # Brand registry
│   │   ├── dyn-sports/    # DYN Sports Token_Set
│   │   └── default/       # Default/base Token_Set
│   ├── validation/
│   │   ├── contrast.ts    # WCAG contrast ratio calculation
│   │   ├── typography.ts  # WCAG 1.4.12 typography checks
│   │   └── validator.ts   # Orchestrates full validation pipeline
│   ├── figma/
│   │   ├── sync.ts        # Figma Variables sync pipeline
│   │   └── mapping.ts     # Token → Figma Variable name mapping
│   ├── export/
│   │   ├── css.ts         # CSS custom properties exporter
│   │   └── esm.ts         # ESM/TypeScript token exporter
│   └── index.ts           # Public API
├── docs/
│   ├── tokens/            # Auto-generated token documentation
│   └── guides/            # Designer and developer getting-started guides
└── tests/
    ├── unit/
    └── property/
```

### Data Flow

```
Token Definitions (JSON/TS)
        │
        ▼
  Schema Validation ──── fails ──▶ Build Error
        │ passes
        ▼
  Accessibility Validation ──── fails ──▶ Non-compliant report, publish blocked
        │ passes
        ▼
  ┌─────┴──────┐
  │            │
  ▼            ▼
CSS Export   ESM Export   Figma Sync Payload
```

---

## Token Data Model

### TypeScript Type Definitions

```typescript
// A raw primitive value
type BaseTokenValue = string | number;

interface BaseToken {
  id: string;           // e.g. "color.palette.orange.500"
  value: BaseTokenValue;
  category: TokenCategory;
  description?: string;
}

// A semantic token references exactly one base token
interface SemanticToken {
  id: string;           // e.g. "color.action.primary"
  ref: string;          // id of the referenced BaseToken
  category: TokenCategory;
  description?: string;
  accessibilityNote?: string;
}

// A component token references exactly one semantic token
interface ComponentToken {
  id: string;           // e.g. "button.background.default"
  ref: string;          // id of the referenced SemanticToken
  component: string;
  role: string;
  description?: string;
}

type TokenCategory =
  | "color"
  | "typography"
  | "spacing"
  | "border-radius"
  | "border-width"
  | "shadow"
  | "motion";

// A complete token set for one brand
interface TokenSet {
  brandId: string;
  version: string;       // semver
  baseTokens: Record<string, BaseToken>;
  semanticTokens: Record<string, SemanticToken>;
  componentTokens: Record<string, ComponentToken>;
  colorSchemes: {
    light: ColorScheme;
    dark: ColorScheme;
  };
}

interface ColorScheme {
  // Maps semantic color token id → resolved hex value for this scheme
  [semanticTokenId: string]: string;
}
```

### Token Hierarchy Invariants

Every `SemanticToken.ref` must resolve to a key in `TokenSet.baseTokens`.
Every `ComponentToken.ref` must resolve to a key in `TokenSet.semanticTokens`.
These invariants are enforced at schema validation time (build step) and are the basis for correctness properties P1–P3.

---

## Token Schema

The schema defines the complete set of required token IDs that every brand `TokenSet` must satisfy. It is expressed as a TypeScript constant:

```typescript
// src/tokens/schema.ts
export const TOKEN_SCHEMA = {
  required: {
    semantic: [
      // Color roles
      "color.action.primary",
      "color.action.secondary",
      "color.text.default",
      "color.text.subtle",
      "color.text.inverse",
      "color.surface.default",
      "color.surface.raised",
      "color.surface.overlay",
      "color.border.default",
      "color.border.focus",
      "color.state.success",
      "color.state.warning",
      "color.state.error",
      "color.state.info",
      // Typography roles
      "typography.display",
      "typography.heading.xl",
      "typography.heading.lg",
      "typography.heading.md",
      "typography.heading.sm",
      "typography.body.lg",
      "typography.body.md",
      "typography.body.sm",
      "typography.label.lg",
      "typography.label.md",
      "typography.label.sm",
      "typography.caption",
      "typography.overline",
      // Spacing roles
      "spacing.component.padding.sm",
      "spacing.component.padding.md",
      "spacing.component.padding.lg",
      "spacing.layout.gap.sm",
      "spacing.layout.gap.md",
      "spacing.layout.gap.lg",
      "spacing.layout.section",
    ],
    base: [
      // Spacing scale
      "spacing.0", "spacing.1", "spacing.2", "spacing.3",
      "spacing.4", "spacing.5", "spacing.6", "spacing.8",
      "spacing.10", "spacing.12", "spacing.16", "spacing.20", "spacing.24",
      // Border radius
      "radius.none", "radius.sm", "radius.md",
      "radius.lg", "radius.xl", "radius.full",
      // Border width
      "border.width.none", "border.width.sm",
      "border.width.md", "border.width.lg",
      // Shadow
      "shadow.none", "shadow.sm", "shadow.md", "shadow.lg", "shadow.xl",
      // Motion
      "motion.duration.fast", "motion.duration.normal",
      "motion.duration.slow", "motion.duration.none",
      "motion.easing.standard", "motion.easing.decelerate",
      "motion.easing.accelerate",
    ],
  },
} as const;
```

---

## DYN Sports Token Set

### Color Palette (Base Tokens)

| Token ID | Value | Notes |
|---|---|---|
| `color.palette.orange.500` | `#FF5C00` | Primary brand orange |
| `color.palette.orange.400` | `#FF7A33` | |
| `color.palette.orange.600` | `#CC4A00` | |
| `color.palette.navy.900` | `#0D1B2A` | Primary dark |
| `color.palette.navy.800` | `#1A2E42` | |
| `color.palette.neutral.0` | `#FFFFFF` | |
| `color.palette.neutral.50` | `#F8F9FA` | |
| `color.palette.neutral.100` | `#F1F3F5` | |
| `color.palette.neutral.200` | `#E9ECEF` | |
| `color.palette.neutral.500` | `#868E96` | |
| `color.palette.neutral.700` | `#495057` | |
| `color.palette.neutral.900` | `#212529` | |
| `color.palette.green.500` | `#2F9E44` | Success |
| `color.palette.yellow.500` | `#F59F00` | Warning |
| `color.palette.red.500` | `#E03131` | Error |
| `color.palette.blue.500` | `#1971C2` | Info |

### Typography Scale (Base Tokens)

Font-size scale uses a 1.25 modular ratio (Major Third), base 16px = 1rem:

| Token ID | Value (rem) | px equivalent |
|---|---|---|
| `font.size.xs` | `0.64rem` | ~10px |
| `font.size.sm` | `0.8rem` | ~13px |
| `font.size.md` | `1rem` | 16px |
| `font.size.lg` | `1.25rem` | 20px |
| `font.size.xl` | `1.563rem` | 25px |
| `font.size.2xl` | `1.953rem` | 31px |
| `font.size.3xl` | `2.441rem` | 39px |
| `font.size.4xl` | `3.052rem` | 49px |

### Spacing Scale (Base Tokens)

Base unit: 4px = 0.25rem

| Token ID | Value (rem) | px |
|---|---|---|
| `spacing.0` | `0rem` | 0 |
| `spacing.1` | `0.25rem` | 4 |
| `spacing.2` | `0.5rem` | 8 |
| `spacing.3` | `0.75rem` | 12 |
| `spacing.4` | `1rem` | 16 |
| `spacing.5` | `1.25rem` | 20 |
| `spacing.6` | `1.5rem` | 24 |
| `spacing.8` | `2rem` | 32 |
| `spacing.10` | `2.5rem` | 40 |
| `spacing.12` | `3rem` | 48 |
| `spacing.16` | `4rem` | 64 |
| `spacing.20` | `5rem` | 80 |
| `spacing.24` | `6rem` | 96 |

---

## Accessibility Validation

### Contrast Ratio Algorithm

WCAG 2.1 relative luminance formula:

```typescript
function relativeLuminance(hex: string): number {
  const rgb = hexToRgb(hex); // returns { r, g, b } in [0, 255]
  const [r, g, b] = [rgb.r, rgb.g, rgb.b].map((c) => {
    const s = c / 255;
    return s <= 0.03928 ? s / 12.92 : Math.pow((s + 0.055) / 1.055, 2.4);
  });
  return 0.2126 * r + 0.7152 * g + 0.0722 * b;
}

function contrastRatio(fg: string, bg: string): number {
  const L1 = relativeLuminance(fg);
  const L2 = relativeLuminance(bg);
  const lighter = Math.max(L1, L2);
  const darker = Math.min(L1, L2);
  return (lighter + 0.05) / (darker + 0.05);
}
```

### Color Pair Definitions

The validator checks the following semantic token pairs:

| Pair | Foreground Token | Background Token | Required Ratio | Type |
|---|---|---|---|---|
| Default text | `color.text.default` | `color.surface.default` | 4.5:1 | Normal text |
| Subtle text | `color.text.subtle` | `color.surface.default` | 4.5:1 | Normal text |
| Inverse text | `color.text.inverse` | `color.action.primary` | 4.5:1 | Normal text |
| Action primary | `color.action.primary` | `color.surface.default` | 3:1 | UI component |
| Focus border | `color.border.focus` | `color.surface.default` | 3:1 | UI component |
| Success state | `color.state.success` | `color.surface.default` | 3:1 | UI component |
| Warning state | `color.state.warning` | `color.surface.default` | 3:1 | UI component |
| Error state | `color.state.error` | `color.surface.default` | 3:1 | UI component |

### Typography Validation Rules

Per WCAG 2.1 Success Criterion 1.4.12:

| Check | Rule |
|---|---|
| Body font size | `font-size >= 16px` for `typography.body.*` tokens |
| Line height | `line-height >= 1.5 × font-size` for body text |
| Letter spacing | `letter-spacing >= 0.12 × font-size` for body text |

### Validation Report Structure

```typescript
interface ValidationReport {
  brandId: string;
  timestamp: string;
  compliant: boolean;
  colorChecks: ColorCheckResult[];
  typographyChecks: TypographyCheckResult[];
}

interface ColorCheckResult {
  pair: string;           // e.g. "color.text.default on color.surface.default"
  foregroundToken: string;
  backgroundToken: string;
  foregroundValue: string;
  backgroundValue: string;
  actualRatio: number;
  requiredRatio: number;
  passed: boolean;
  scheme: "light" | "dark";
}

interface TypographyCheckResult {
  tokenId: string;
  check: "font-size" | "line-height" | "letter-spacing";
  actualValue: number;
  requiredMinimum: number;
  passed: boolean;
}
```

---

## Brand Registry

```typescript
// src/brands/registry.ts

type BrandStatus = "active" | "pending-validation" | "non-compliant" | "draft";

interface BrandRegistryEntry {
  brandId: string;
  displayName: string;
  tokenSetPath: string;   // relative path to the token set module
  version: string;        // semver of the token set
  status: BrandStatus;
  lastValidated?: string; // ISO timestamp
}

interface BrandRegistry {
  brands: Record<string, BrandRegistryEntry>;
}
```

When a new brand entry is added with `status: "pending-validation"`, the validation pipeline runs automatically and updates the status to `"active"` or `"non-compliant"`.

---

## Figma Variables Sync

### Naming Convention

| Token Level | Figma Variable Name Pattern | Example |
|---|---|---|
| Semantic | `{category}/{semantic-name}` | `color/action/primary` |
| Component | `{category}/{component}/{role}` | `color/button/background-default` |

### Sync Payload Structure

```typescript
interface FigmaVariablesPayload {
  collections: FigmaCollection[];
}

interface FigmaCollection {
  name: string;           // e.g. "Color", "Typography", "Spacing"
  modes: FigmaMode[];
  variables: FigmaVariable[];
}

interface FigmaMode {
  name: string;           // e.g. "DYN Sports / Light", "DYN Sports / Dark"
  brandId: string;
  scheme: "light" | "dark";
}

interface FigmaVariable {
  name: string;           // naming convention above
  tokenId: string;        // source token id for traceability
  valuesByMode: Record<string, string | number>; // modeId → resolved value
}
```

The sync script (`src/figma/sync.ts`) generates this payload and can be consumed by the Figma REST API or the Figma Tokens plugin.

---

## Export Formats

### CSS Custom Properties

```css
/* Generated output example */
:root {
  --color-action-primary: #FF5C00;
  --color-text-default: #212529;
  --spacing-4: 1rem;
  /* ... */
}

[data-brand="dyn-sports"][data-scheme="dark"] {
  --color-action-primary: #FF7A33;
  /* ... */
}
```

### ESM/TypeScript

```typescript
// Generated output example
export const tokens = {
  color: {
    action: { primary: "var(--color-action-primary)" },
    text: { default: "var(--color-text-default)" },
  },
  spacing: { 4: "var(--spacing-4)" },
} as const;
```

---

## Correctness Properties

The following universal properties must hold for any valid token set and are used to drive property-based tests.

**Property 1: Semantic token reference integrity**
For every `SemanticToken` in a `TokenSet`, its `ref` field must resolve to an existing `BaseToken` id within the same `TokenSet`.
- Validates: Requirements 1.2

**Property 2: Component token reference integrity**
For every `ComponentToken` in a `TokenSet`, its `ref` field must resolve to an existing `SemanticToken` id within the same `TokenSet`.
- Validates: Requirements 1.3

**Property 3: Base token update propagation**
After updating a `BaseToken` value, all `SemanticTokens` and `ComponentTokens` that transitively reference it must resolve to the new value when their resolved value is computed.
- Validates: Requirements 1.4

**Property 4: Token set schema completeness**
For any `TokenSet` that passes schema validation, every token id listed in `TOKEN_SCHEMA.required.semantic` and `TOKEN_SCHEMA.required.base` must be present in the token set.
- Validates: Requirements 2.2, 2.3

**Property 5: Token set extension correctness**
For any `TokenSet` B that extends default `TokenSet` A with override set O, the resolved value of any token in B equals the value in O if the token is in O, otherwise equals the value in A.
- Validates: Requirements 2.4

**Property 6: Token set diff completeness**
For any two `TokenSets` A and B, `diff(A, B)` returns exactly the set of token ids present in A but not B, plus the set present in B but not A (symmetric difference).
- Validates: Requirements 2.5, 9.5

**Property 7: Normal text contrast compliance**
For any compliant `TokenSet`, every text-on-background color pair designated as normal text must have a contrast ratio ≥ 4.5, for both light and dark schemes.
- Validates: Requirements 3.3, 8.1

**Property 8: Large text and UI component contrast compliance**
For any compliant `TokenSet`, every color pair designated as large text or UI component must have a contrast ratio ≥ 3.0, for both light and dark schemes.
- Validates: Requirements 3.4, 3.5, 8.1

**Property 9: Typography spacing compliance**
For any compliant `TokenSet`, every body text typography token must satisfy: `font-size >= 16px`, `line-height >= 1.5 × font-size`, and `letter-spacing >= 0.12 × font-size`.
- Validates: Requirements 4.3, 4.4, 4.5, 8.2

**Property 10: Spacing scale base unit**
For every spacing base token with step index N, its value in rem equals `N × 0.25rem`.
- Validates: Requirements 5.2

**Property 11: Figma variable naming convention**
For any token exported to Figma, its variable name matches the pattern `{category}/{semantic-name}` for semantic tokens and `{category}/{component}/{role}` for component tokens.
- Validates: Requirements 7.4

**Property 12: Schema update gap detection**
For any schema update that adds a new required token id T, `findMissingTokens(registry, newSchema)` returns every brand id whose token set does not contain T.
- Validates: Requirements 9.5

---

## Grid System & Layout

### Design Principles

The layout system has two distinct modes that must never be confused:

- **Full-bleed** — element stretches edge-to-edge at any viewport width (hero backgrounds, video players, section backgrounds)
- **Content-contained** — content sits inside a max-width wrapper, centered, with horizontal gutters

The max content width uses **Tailwind's default `2xl` breakpoint: 1536px** (`max-w-screen-2xl`). No custom config needed. Beyond this, content stops growing and remains centered. This prevents the extreme scaling currently visible on wide monitors.

---

### Breakpoints (Tailwind defaults — no custom config required)

| Name | Token | Min width | Tailwind prefix | Primary use |
|---|---|---|---|---|
| Mobile | `bp.mobile` | 0px | _(default)_ | Single column, full-width stacks |
| Tablet | `bp.tablet` | 768px | `md:` | 2-column layouts, side-by-side content |
| Desktop | `bp.desktop` | 1280px | `xl:` | Full multi-column grid, sidebar layouts |
| Wide | `bp.wide` | 1536px | `2xl:` | Max-width cap — content freezes, no further scaling |

> All breakpoints are Tailwind defaults. No `tailwind.config.js` changes needed.
> Tailwind's `lg:` (1024px) is available for intermediate adjustments but is not a primary layout breakpoint in the DYN grid.

---

### Content Wrapper

The wrapper is the single source of truth for horizontal content boundaries. It must be used on every section that contains readable content.

```
┌─────────────────────────────────────────────────────────────────┐  ← viewport (any width)
│  gutter │◄──────────── max 1536px content ────────────────►│ gutter  │
└─────────────────────────────────────────────────────────────────┘
```

#### Wrapper token values

| Property | Mobile | Tablet (md) | Desktop (xl) | Wide (2xl) |
|---|---|---|---|---|
| `layout.wrapper.max-width` | 100% | 100% | 100% | 1536px (`max-w-screen-2xl`) |
| `layout.wrapper.padding-x` | 16px | 24px | 48px | 48px |

#### Tailwind implementation

```html
<!-- Standard content wrapper — no custom config needed -->
<div class="mx-auto w-full max-w-screen-2xl px-4 md:px-6 xl:px-12">
  <!-- content -->
</div>
```

#### CSS custom property

```css
:root {
  --layout-wrapper-max-width: 1536px; /* Tailwind screen-2xl default */
  --layout-wrapper-padding-x-mobile: 16px;
  --layout-wrapper-padding-x-tablet: 24px;
  --layout-wrapper-padding-x-desktop: 48px;
}
```

---

### Column Grid

The grid uses a 12-column system at desktop, collapsing to fewer columns at smaller breakpoints.

| Breakpoint | Columns | Gutter | Margin (= wrapper padding-x) |
|---|---|---|---|
| Mobile | 4 | 16px | 16px |
| Tablet | 8 | 16px | 24px |
| Desktop | 12 | 24px | 48px |
| Wide | 12 | 24px | 48px (content capped at 1536px) |

#### Common column spans

| Use case | Mobile | Tablet | Desktop |
|---|---|---|---|
| Full width | 4 col | 8 col | 12 col |
| Main content + sidebar | 4 col | 5 col | 8 col |
| Sidebar | 4 col | 3 col | 4 col |
| Half-half split | 4 col | 4 col | 6 col |
| Card (3-up) | 4 col | 4 col | 4 col |
| Card (4-up) | 4 col | 4 col | 3 col |

---

### Full-Bleed vs Contained Layouts

Some elements intentionally break out of the wrapper. The rule is:

| Element type | Layout mode | Notes |
|---|---|---|
| Hero background / video | Full-bleed | `w-full`, no max-width |
| Hero text content | Contained | Sits inside wrapper on top of full-bleed bg |
| Section background color | Full-bleed | Background spans full width |
| Section text/cards | Contained | Content inside wrapper |
| Inline video player | Contained | Respects column grid |
| Full-width editorial video | Full-bleed up to 1920px | See media aspect ratios below |
| Navigation bar | Full-bleed background | Content inside wrapper |

#### Pattern: full-bleed section with contained content

```html
<section class="w-full bg-[#00141E]">
  <div class="mx-auto w-full max-w-content px-4 md:px-6 xl:px-12">
    <!-- content here -->
  </div>
</section>
```

#### Pattern: full-bleed hero video with contained text overlay

```html
<section class="relative w-full">
  <!-- Full-bleed video layer -->
  <video class="absolute inset-0 w-full h-full object-cover -z-10" ...></video>
  <!-- Contained text -->
  <div class="mx-auto w-full max-w-content px-4 md:px-6 xl:px-12 py-24">
    <h1>...</h1>
  </div>
</section>
```

---

### Media Aspect Ratios

All video and image media must use a defined aspect ratio token to ensure correct resolution requests and prevent layout shift (CLS).

#### Aspect ratio tokens

| Token | Ratio | Pixel example at 1920px | Use case |
|---|---|---|---|
| `media.ratio.hero` | 16:9 | 1536 × 864px | Full-bleed hero video/image |
| `media.ratio.hero-wide` | 21:9 | 1536 × 659px | Cinematic hero on wide screens |
| `media.ratio.player` | 16:9 | — | Inline video player (always 16:9) |
| `media.ratio.thumbnail` | 16:9 | 480 × 270px | Event/match card thumbnails |
| `media.ratio.thumbnail-square` | 1:1 | 480 × 480px | Sport icon, avatar, team badge |
| `media.ratio.editorial` | 3:2 | 1200 × 800px | Editorial/article header image |
| `media.ratio.split-screen` | 1:1 | — | 50/50 split layout (video fills half viewport height) |

#### Why 16:9 for hero and player

Broadcast sports content is produced in 16:9. Using any other ratio for the player would require letterboxing or cropping, degrading quality. The hero background video should also be 16:9 so the source file is never upscaled.

#### Required source resolutions

| Token | Minimum source resolution | Notes |
|---|---|---|
| `media.ratio.hero` | 1536 × 864px | Provide 2× (3072 × 1728) for retina |
| `media.ratio.hero-wide` | 1536 × 659px | Crop from 16:9 master |
| `media.ratio.player` | 1920 × 1080px (HD) or 3840 × 2160px (4K) | Streaming adaptive bitrate |
| `media.ratio.thumbnail` | 960 × 540px | 2× for retina card grids |
| `media.ratio.thumbnail-square` | 480 × 480px | 2× = 960 × 960px |
| `media.ratio.editorial` | 1200 × 800px | 2× = 2400 × 1600px |
| `media.ratio.split-screen` | 960 × 960px | Fills 50svw, height = 100svh |

#### Tailwind aspect ratio classes

```html
<!-- Hero video -->
<div class="aspect-video w-full">          <!-- 16:9 -->
  <video class="w-full h-full object-cover" ...></video>
</div>

<!-- Cinematic hero (custom) -->
<div class="aspect-[21/9] w-full">
  <video class="w-full h-full object-cover" ...></video>
</div>

<!-- Match thumbnail card -->
<div class="aspect-video w-full overflow-hidden rounded-md">
  <img class="w-full h-full object-cover" ...>
</div>

<!-- Split-screen (current pattern in codebase) -->
<div class="w-full lg:w-[50svw] lg:h-screen">
  <video class="w-full h-full object-cover" ...></video>
</div>
```

#### CSS custom properties

```css
:root {
  --media-ratio-hero: 16 / 9;
  --media-ratio-hero-wide: 21 / 9;
  --media-ratio-player: 16 / 9;
  --media-ratio-thumbnail: 16 / 9;
  --media-ratio-thumbnail-square: 1 / 1;
  --media-ratio-editorial: 3 / 2;
  --media-ratio-split-screen: 1 / 1;
}
```

---

### Layout Token Summary

All layout tokens belong to the `layout` category in the token schema.

| Token ID | Value | Notes |
|---|---|---|
| `layout.wrapper.max-width` | `1536px` | Tailwind `max-w-screen-2xl` default — no custom config needed |
| `layout.wrapper.padding-x.mobile` | `16px` | spacing-16 |
| `layout.wrapper.padding-x.tablet` | `24px` | spacing-24 |
| `layout.wrapper.padding-x.desktop` | `48px` | spacing-48 |
| `layout.grid.columns.mobile` | `4` | |
| `layout.grid.columns.tablet` | `8` | |
| `layout.grid.columns.desktop` | `12` | |
| `layout.grid.gutter.mobile` | `16px` | spacing-16 |
| `layout.grid.gutter.tablet` | `16px` | spacing-16 |
| `layout.grid.gutter.desktop` | `24px` | spacing-24 |
| `layout.breakpoint.tablet` | `768px` | Tailwind `md:` |
| `layout.breakpoint.desktop` | `1280px` | Tailwind `xl:` |
| `layout.breakpoint.wide` | `1536px` | Tailwind `2xl:` default |
| `media.ratio.hero` | `16/9` | |
| `media.ratio.hero-wide` | `21/9` | |
| `media.ratio.player` | `16/9` | |
| `media.ratio.thumbnail` | `16/9` | |
| `media.ratio.thumbnail-square` | `1/1` | |
| `media.ratio.editorial` | `3/2` | |
| `media.ratio.split-screen` | `1/1` | |
