# Design System — Actors & Dependencies

This document maps all actors (systems, tools, teams) that contribute to or consume from the DYN Design System environment. It defines roles, responsibilities, data flow direction, and dependency relationships.

---

## Overview Diagram (Text-Based)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DYN DESIGN SYSTEM ENVIRONMENT                        │
└─────────────────────────────────────────────────────────────────────────────┘

                          ┌──────────────────┐
                          │      FIGMA       │
                          │ (Source of Truth) │
                          └────────┬─────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
         ┌──────────────┐  ┌───────────┐  ┌──────────────────┐
         │  Design System│  │ Storybook │  │  DDC Frontend    │
         │  Page (Docs)  │  │           │  │  (User Canvas)   │
         └──────┬───────┘  └─────┬─────┘  └────────┬─────────┘
                │                 │                  │
                │                 └────────┬─────────┘
                │                          │
                ▼                          ▼
     ┌───────────────────┐     ┌─────────────────────┐
     │  External Consumers│     │  White Label Themes  │
     │  (Marketing, Brand)│     │  (Future)            │
     └───────────────────┘     └─────────────────────┘

         ┌─────────────────────────────────────┐
         │        EXTERNAL TOOLS (Manual)       │
         │  WordPress · Instapages · Others     │
         └─────────────────────────────────────┘
```

---

## Actor Definitions

### 1. Figma — Single Source of Truth (Design)

| Attribute | Detail |
|-----------|--------|
| **Role** | Origin of all design decisions, component definitions, and token values |
| **Owner** | Design Team (Nadine) |
| **Outputs** | Design tokens, component specs, visual guidelines, interaction patterns |
| **Consumers** | Design System Page, Storybook, DDC Frontend |
| **Update Trigger** | Manual design changes by the design team |

**Responsibilities:**
- Define and maintain all visual components
- Manage design tokens (colors, spacing, typography, effects)
- Provide the canonical component structure and variants
- Communicate changes downstream (tokens export, handoff documentation)

---

### 2. Design System Page — Documentation & Communication Hub

| Attribute | Detail |
|-----------|--------|
| **Role** | Central documentation, changelog, usage guidelines, and accessibility reference |
| **Owner** | Design Team + Dev Team (shared) |
| **Inputs** | Figma (design changes), DDC Frontend (implementation status) |
| **Consumers** | Marketing, External stakeholders, Developers, Content teams |
| **Update Trigger** | Any design or component change in Figma or implementation updates |

**Responsibilities:**
- Document component usage, requirements, and accessibility guidelines
- Maintain a visible changelog with descriptions of what changed and why
- Provide a general overview understandable for non-technical stakeholders
- Serve as the external-facing reference for the design system

---

### 3. Storybook — Component Development & Testing

| Attribute | Detail |
|-----------|--------|
| **Role** | Interactive component library for development, testing, and QA |
| **Owner** | Frontend Development Team |
| **Inputs** | Figma (component specs), Design tokens |
| **Outputs** | Implemented components, visual regression tests |
| **Connection** | Directly linked to DDC Frontend codebase |

**Responsibilities:**
- Implement components based on Figma specifications
- Provide interactive documentation for developers
- Enable visual testing and QA of component states
- Serve as the bridge between design intent and production code

---

### 4. DDC Frontend — User-Facing Canvas

| Attribute | Detail |
|-----------|--------|
| **Role** | The production application where end users interact with the design system |
| **Owner** | Frontend Development Team |
| **Inputs** | Storybook (components), Design tokens, Figma (specs) |
| **Outputs** | Live user experience |
| **Connection** | Consumes Storybook components; will consume White Label themes |

**Responsibilities:**
- Render the final user experience using design system components
- Integrate Storybook components into production
- Apply theming (current: DYN brand; future: white label themes)
- Ensure performance and accessibility in production context

---

### 5. White Label Themes — Customer Theming Layer (Future)

| Attribute | Detail |
|-----------|--------|
| **Role** | Theme configurations for customer-specific branding on the DDC platform |
| **Owner** | TBD (likely Product + Design collaboration) |
| **Inputs** | Design token structure from Figma, DDC Frontend theming API |
| **Outputs** | Customer-specific visual configurations |
| **Status** | 🔮 Planned — not yet active |

**Responsibilities (Future):**
- Provide a structured way to override design tokens per customer
- Maintain compatibility with the base design system
- Ensure white label themes don't break component functionality
- Integrate seamlessly into the DDC Frontend theming architecture

**Requirements for Future Integration:**
- Token structure must support theme layering (base → brand override)
- Components must be theme-agnostic (no hardcoded brand values)
- A theme configuration format must be defined (JSON/CSS variables)

---

### 6. External Tools — Manual Consumers (Current State)

| Tool | Purpose | Current Update Method | Future Goal |
|------|---------|----------------------|-------------|
| **WordPress** | Editorial content, SEO-focused sport pages | Manual | Automated or semi-automated sync |
| **Instapages** | Campaign landing pages | Manual | Automated or semi-automated sync |
| **Other tools** | Various marketing/content platforms | Manual | To be evaluated |

| Attribute | Detail |
|-----------|--------|
| **Role** | External platforms that need to reflect design system standards |
| **Owner** | Marketing / Content / Editorial teams |
| **Inputs** | Design guidelines, brand assets, component patterns (manually transferred) |
| **Status** | ⚠️ Currently fully manual — improvement desired but no concrete plan yet |

**Current Pain Points:**
- Every design change requires manual updates across all external tools
- No automated notification when something changes
- Risk of visual inconsistency between DDC and external platforms
- High maintenance effort for the teams managing these tools

**Future Improvement Ideas (to be evaluated):**
- Automated design token export to CSS that external tools can consume
- Notification system when relevant changes occur
- Shared asset library or CDN for common elements
- Style guide export in formats compatible with WordPress/Instapages

---

## Dependency Matrix

| Actor | Depends On | Feeds Into |
|-------|-----------|------------|
| Figma | — (origin) | Design System Page, Storybook, DDC Frontend |
| Design System Page | Figma, DDC Frontend | Marketing, External stakeholders |
| Storybook | Figma | DDC Frontend |
| DDC Frontend | Storybook, Figma | End Users, White Label Themes |
| White Label Themes | DDC Frontend, Figma token structure | DDC Frontend (themed) |
| External Tools | Design System Page (manual) | End Users (WordPress, campaigns) |

---

## Data Flow Direction

```
DESIGN DIRECTION (top-down):
Figma → Tokens/Specs → Storybook → DDC Frontend → Users

DOCUMENTATION DIRECTION (lateral):
Figma → Design System Page → Marketing / External / Dev Teams

THEMING DIRECTION (future, layered):
Figma Base Tokens → White Label Override → DDC Frontend Themed Output

EXTERNAL TOOLS (currently disconnected):
Design System Page → (manual copy) → WordPress / Instapages / Others
```

---

## Communication Flow

| Change Origin | Who Needs to Know | How (Current) | How (Ideal Future) |
|---------------|-------------------|---------------|---------------------|
| Figma component update | Dev, Storybook, Docs | Manual handoff | Automated token sync + notification |
| New component in Figma | Dev, Storybook, Docs, Marketing | Manual handoff | Automated pipeline |
| Token value change | All actors | Manual export | Automated token distribution |
| DDC implementation done | Docs, Marketing | Manual update | Auto-status in Design System Page |
| External tool needs update | WordPress/Instapage editors | Slack/Email | Automated notification or sync |

---

## Priority & Maturity Levels

| Actor | Priority | Maturity | Next Step |
|-------|----------|----------|-----------|
| Figma | 🔴 Critical | ✅ Active | Maintain as source of truth |
| Design System Page | 🔴 Critical | 🟡 In Progress | Build out changelog + component docs |
| Storybook | 🔴 Critical | 🟡 In Progress | Align with Figma specs |
| DDC Frontend | 🔴 Critical | 🟡 In Progress | Consume Storybook components |
| White Label Themes | 🟡 Medium | ⬜ Planned | Define token layering architecture |
| External Tools | 🟠 Important | 🔴 Manual/Fragile | Evaluate automation options |

---

## Open Questions for Future Planning

1. **Token distribution format** — What format should exported tokens use so all consumers (Storybook, DDC, external tools) can consume them?
2. **White Label architecture** — How deep should theme overrides go? (Colors only? Typography? Spacing? Component variants?)
3. **External tool automation** — Is a shared CSS/token CDN feasible for WordPress and Instapages?
4. **Notification system** — Should there be an automated alert when Figma changes affect external tools?
5. **Versioning** — Should the design system have formal version numbers that external tools can pin to?

---

*Last updated: 2025-05-21*
*Status: Living document — will evolve as the system matures*
