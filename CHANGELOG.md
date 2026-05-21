# Changelog — DYN Sports Design System

All changes to this design system are documented here.
Format: `v{major}.{minor}` — major = breaking or structural change, minor = addition or update.

---

## v1.7 — 2026-05-21

### Added — Layout / Device Landscape
- **TV / Smart TV category added** to Device Landscape grid.
  - Priority resolutions based on webtracking playtime share data (April 2026):
    1. 1920×1080 — Samsung Tizen / Fire TV / Sky STB (FHD, dominant output)
    2. 3840×2160 — Samsung 4K / Fire TV 4K (UHD)
    3. 1280×720 — Fire TV Stick Lite / budget Smart TVs (HD)
  - Top 3 TV platforms by watch time: Samsung Tizen (16.1%), Amazon Fire TV (15.4%), Sky STB (11.6%)
- **Device landscape grid updated** from 3-column to 4-column layout to accommodate TV category.
  - Added intermediate breakpoint at 1400px (2-column) for better responsiveness.
- Impact: Design / Dev

---

## v1.6 — 2026-05-06

### Added — Foundations / Config Layer
- **Introduced `config/` layer** — new folder for static design system configuration data, separate from Figma token files and UI logic.
- **`config/layout.device-landscape.json`** — structured device landscape data model under `layout.deviceLandscape`.
  - Three sections: `mobile`, `desktop`, `tablet`
  - Each section includes `lastEvaluated`, `nextReview`, `dataSource`, `priorityResolutions[]`, and `rawData[]`
  - Priority resolutions sourced from `Webtracking_Devices_042026.csv` (April 2026 analytics)
  - Mobile: `390x844` (iPhone 14/15), `393x852` (iPhone 16), `360x800` (Android mid-range)
  - Desktop: `1920x1080` (Windows FHD), `1440x900` (Mac older Retina), `2560x1600` (MacBook Pro M-series)
  - Tablet: `820x1180` (iPad), `800x1280` (Android/Amazon Fire)
- **`config/README.md`** — documents the config layer purpose, file structure, field definitions, and update rules.

**Impact:** Design · Dev

---

## v1.5 — 2026-05-06

### Changed — Documentation / HTML Page
- **Improved page spacing, content alignment, and responsive layout consistency.**
- Unified padding system across all four tabs using `.panel-main` + `.panel-content` containers.
- Content capped at 1200px readable width, centered within the available area after the sidebar.
- Consistent top offset (104px) below fixed header + tab bar on all panels.
- Horizontal padding scales with breakpoints: 16px mobile → 32px tablet → 48px desktop.
- Foundations hero remains full-bleed; all other content is contained and breathable.
- Removed inline `style="padding:..."` overrides from panel-main divs — spacing now fully CSS-driven.

**Impact:** Design

---

## v1.4 — 2026-05-06

### Changed — Documentation / HTML Page
- **Tab architecture refactored.** Separated global layout from per-tab content and contextual side navigation.
- Each tab now has its own independent panel (`latestUpdatesPanel`, `foundationsPanel`, `coreComponentsPanel`, `compositeComponentsPanel`) — only one visible at a time.
- Each panel has its own contextual sidebar with tab-specific navigation items:
  - **Latest Updates**: week periods only (Week 19, 18, 17 | 2026)
  - **Foundations**: foundation sections (Color Primitives → Grid System)
  - **Core Components**: component sections (Buttons, Inputs, Checkboxes, Radios, Toggles, Badges)
  - **Composite Components**: composite sections (Pricing Cards, Forms, Headers, Steppers, Cards, Navigation Patterns)
- Pricing Cards moved from Foundations to Composite Components.
- Removed global sidebar and `<main>` wrapper — replaced by per-panel layout shells.
- JavaScript tab switcher updated to show/hide per-panel sidebars independently.
- **Improved page spacing, content alignment, and responsive layout consistency.** Unified padding system across all tabs. Content now properly centered with consistent breathable spacing. Sidebar offset and horizontal padding follow the design system scale (16px mobile, 32px tablet, 48px desktop).

**Impact:** Design · Dev

---


### Fixed
- HTML overview file recreated after accidental deletion. Structure restored from available project context (Figma token JSON files, spec documents, README, and CHANGELOG).

---

## v1.2 — 2026-05-06

### Changed
- Dark/light mode toggle icon logic corrected: ☀ shows in dark mode (switch to light), ☾ shows in light mode (switch to dark)

---

## v1.1 — 2026-05-06

### Added
- Grid system & layout section added to design document and HTML preview
  - Breakpoints: Mobile (0px), Tablet (768px / `md:`), Desktop (1280px / `xl:`), Wide (1536px / `2xl:`)
  - Content wrapper: `max-w-screen-2xl` — Tailwind default, no custom config needed
  - Full-bleed vs contained layout patterns with code examples
  - Media aspect ratio tokens: 16:9 (hero, player, thumbnail), 21:9 (cinematic), 3:2 (editorial), 1:1 (square, split-screen)
  - Minimum source resolutions documented for all media tokens
  - Layout token summary table added to design.md

---

## v1.0 — 2026-05-06

### Added
- Initial design system setup for DYN Sports (Dark Mode)
- Figma token files exported and committed:
  - `primitives/DYN.tokens.json` — color palette, font sizes, spacing, weights
  - `colors-global/Dark Mode.tokens.json` — semantic color roles
  - `colors-component-tokens/Mode 1.tokens.json` — component-level tokens (pricing cards)
  - `effects-global/DYN - Dark Mode.tokens.json` — overlay and glass effects
  - `spacings-global/DYN.tokens.json` — component spacing (header, card, navigation)
  - `stroke-global/DYN.tokens.json` — border widths and corner radius
  - `typos-global/DYN.tokens.json` — typography styles (h1–h7, p1–p3, interaction)
- Spec documents:
  - `requirements.md` — 10 requirement areas covering token architecture, accessibility, white-label scalability
  - `design.md` — technical design with token hierarchy, WCAG validation, Figma sync pipeline
  - `tasks.md` — 16 implementation tasks with checkpoints
- HTML preview (`design-system-preview.html`):
  - Color primitives: Cool Neutrals, Blue Shades, Classic Neutrals, Green, Red, Yellow, Orange
  - Global color roles: Brand, Surface, Text, Status, Divider (Dark Mode)
  - Sport-specific colors: Handball, Basketball, Volleyball, Table Tennis, Hockey
  - Typography scale: heading-h1 through heading-h7, copy-p1/p2/p3, interaction
  - Spacing scale: 0–64px
  - Stroke & border radius tokens
  - Effects & overlays (backdrop levels, glass stroke)
  - WCAG accessibility contrast check panel (live JS calculation)
  - Token hierarchy diagram (Primitive → Global → Component)
  - Pricing card component tokens (Standard, Deal, Voucher)
  - Dark/light mode toggle
  - Fixed sidebar navigation with scroll-aware active states
