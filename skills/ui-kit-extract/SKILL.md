---
name: ui-kit-extract
version: "2.0.0"
description: >
  Extract UI components from React/Next.js projects into @stsgs/ui library.
  Works with: local project, single GitHub repo, or multiple repos.
  Finds hardcoded patterns, generalizes domain logic, parameterizes colors,
  creates reusable components. Does NOT modify source projects.
  Activate when user wants to:
  - Extract components to library
  - Scan project for UI patterns
  - Build component library from existing projects
  - Or mentions: "extract components", "build ui kit", "извлечь компоненты"
---

# UI Kit Extract

## Overview

Extracts reusable UI components from existing projects into @stsgs/ui.

**Key principle:** Source projects remain untouched. Components are copied, generalized, and enhanced.

**Target library:** `@stsgs/ui` (npm package, 155+ components, shadcn/ui compatible)

---

## Known Extraction Sources

| Source | Status | What was extracted |
|--------|--------|--------------------|
| Ormuz-monitor | DONE | 33+ scifi-features (~230 files) |
| Component-Browser | DONE | CompareModal, StatsDashboard, useKeyboardShortcuts |
| Z.Code-Guide | DONE | pipeline-stepper, version-history, section-header, etc. |
| CHROMEDNA | DONE | GlowButton, TradeButton, SessionBadge |
| Code-Realm | PARTIAL | tokens, presets, layout patterns available for extraction |

When adding a new source, add it to this table.

---

## Usage Modes

```
# Local project (current directory)
"извлеки компоненты в ui-kit"

# Single GitHub repo
"извлеки компоненты из https://github.com/user/project"

# Multiple repos
"извлеки компоненты из этих репо: url1, url2, url3"
```

---

## Workflow

### Phase 1: Scan

Scan for hardcoded UI patterns:

```bash
# Status dots / indicators
rg "w-[0-9.]+ h-[0-9.]+ rounded-full bg-" src/

# Buttons with colors
rg "bg-amber-|bg-green-|bg-red-|bg-blue-" src/ | rg "button|Button|className.*px-"

# Badges / tags
rg "px-[0-9.]+ py-[0-9.]+ rounded-full|rounded-lg" src/ | rg "bg-"

# Cards / panels
rg "rounded-lg border.*bg-|glass-card|glass-panel" src/

# Animations / effects
rg "animate-pulse|animate-spin|glow-|shadow-\[" src/

# Layout patterns (grids, flex, sections)
rg "grid-template|gap-[0-9]|minmax\(" src/
```

### Phase 2: Group Patterns

Group found patterns by type:

| Category | Patterns to Find |
|----------|------------------|
| StatusDots | `w-2 h-2 rounded-full bg-green-500` |
| Buttons | `px-4 py-2 rounded bg-amber-500` |
| Badges | `px-2 py-1 rounded-full text-green-400` |
| Cards | `p-4 rounded-lg border bg-gray-900` |
| Dividers | `border-t border-gray-700` |
| Inputs | `px-3 py-2 rounded border bg-gray-800` |
| Layouts | `grid grid-cols-* gap-*` sections |
| Charts | SVG-based sparklines, bars, gauges |
| Timelines | Vertical/horizontal lists with connectors |

### Phase 3: Interactive Selection

For each pattern found:

```
=== [1/5] GlowButton ===
Source: 3 files, 7 occurrences

Pattern: bg-amber-500/15 text-amber-400 border-amber-500/30

Files:
  Header.tsx:198, 222
  TradeSimulation.tsx:212
  LeftPanel.tsx:234

Detected color: amber

Actions:
  [Y] Extract with parameterized colors
  [N] Skip this pattern
  [A] Extract all remaining without asking
  [Q] Quit

Choice: _
```

### Phase 4: Parameterize Colors

When extracting, replace hardcoded colors with variants:

**Before (hardcoded):**
```tsx
<button className="bg-amber-500/15 text-amber-400 border-amber-500/30">
```

**After (parameterized):**
```tsx
export interface GlowButtonProps {
  variant?: 'amber' | 'green' | 'red' | 'blue'
}

const variantStyles = {
  amber: 'bg-amber-500/15 text-amber-400 border-amber-500/30',
  green: 'bg-green-500/15 text-green-400 border-green-500/30',
  red: 'bg-red-500/15 text-red-400 border-red-500/30',
  blue: 'bg-blue-500/15 text-blue-400 border-blue-500/30',
}
```

### Phase 5: Create Component

Generate component following @stsgs/ui conventions:

```tsx
/**
 * GlowButton - Button with animated glow effect
 * Extracted from: CHROMEDNA
 *
 * @example
 * <GlowButton variant="amber">Click me</GlowButton>
 */

'use client'

import { forwardRef } from 'react'
import { cn } from '@/lib/utils'

export type GlowVariant = 'amber' | 'green' | 'red' | 'blue'

export interface GlowButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: GlowVariant
  size?: 'sm' | 'md' | 'lg'
  active?: boolean
}

const variantStyles: Record<GlowVariant, string> = {
  amber: 'bg-amber-500/15 text-amber-400 border-amber-500/30',
  green: 'bg-green-500/15 text-green-400 border-green-500/30',
  red: 'bg-red-500/15 text-red-400 border-red-500/30',
  blue: 'bg-blue-500/15 text-blue-400 border-blue-500/30',
}

export const GlowButton = forwardRef<HTMLButtonElement, GlowButtonProps>(
  ({ variant = 'amber', size = 'md', active, className, ...props }, ref) => {
    return (
      <button
        ref={ref}
        data-slot="glow-button"
        className={cn(
          'inline-flex items-center justify-center rounded-md font-medium',
          'transition-all duration-200',
          variantStyles[variant],
          className
        )}
        {...props}
      />
    )
  }
)

GlowButton.displayName = 'GlowButton'
```

### Phase 6: Add to Library

1. Create component file in appropriate category (see Component Categories below)
2. Update barrel export (index.ts) for the category
3. Verify TypeScript compiles: `npx tsc --noEmit`
4. Git commit and push

---

## Color Parameterization Rules

| Original Color | Variants to Add |
|----------------|-----------------|
| amber, yellow | amber, green, red, blue |
| green, emerald | green, amber, red, blue |
| red, rose | red, amber, green, blue |
| blue, indigo | blue, amber, green, red |
| purple, violet | purple, blue, amber, green |
| gray, slate | gray (only) |

Default variant = the original color from source project.

---

## Component Categories

Categories match @stsgs/ui package structure:

### ui/ (Level 1 - Primitives)
Base shadcn/ui compatible components.
- StatusDot, GlowDot, Badge, Tag
- Button, IconButton, GlowButton
- Divider, AnimatedDivider
- MetricValue, DataLabel
- ProgressBar, LoadingSpinner
- Accordion, Alert, Avatar, Calendar, Card, Dialog, etc.

### sections/ (Level 2 - Page Compositions)
Page-level layout sections.
- HeroSection, CTASection, PricingGrid
- FeatureGrid, TestimonialSection, FAQSection
- StatsSection, TeamGrid, ContactForm
- Footer, Header, Navigation

### features/ (Level 3 - Interactive Components)
Complex interactive components with state.
- ScifiButtonGroup, ScifiTabbedView
- ScifiTimeline, ScifiScenarioCards
- ScifiLiveFeed, ScifiTickerBar
- PipelineStepper, VersionHistory
- KeyboardShortcutsGrid, MobilePageHeader
- IconSidebarNav, CompareModal, StatsDashboard

When extracting, choose the category that best fits the component's complexity and purpose.

---

## Anti-Monolith Rules (MANDATORY)

Every extracted component MUST comply:

| Rule | Limit |
|------|-------|
| Max lines per file | 150 lines |
| Max useState calls | 3 per component |
| forwardRef | Required on main export |
| cn() | Required for className merging |
| data-slot | Required on root element |
| JSDoc @example | Required on main export |
| Domain references | Zero (generalize all) |
| Russian text | Replace with English |
| Zero fetch/API calls | Pure rendering only |

If a component exceeds 150 lines, split into:
- Main component (component-name.tsx)
- Sub-components (component-name-part.tsx)
- Types (component-name-types.ts)

Example: ScifiCorrelationDashboard (5 files):
- correlation-dashboard.tsx (main, 98 lines)
- correlation-row.tsx (sub-component, 65 lines)
- asset-row.tsx (sub-component, 42 lines)
- correlation-sparkline.tsx (sub-component, 38 lines)
- correlation-types.ts (types, 28 lines)

---

## Output Format

After extraction, show summary:

```
=== EXTRACTION COMPLETE ===

Extracted: 3 components
  GlowButton    @stsgs/ui/ui/glow-button.tsx
  TradeButton   @stsgs/ui/features/trade-button/trade-button.tsx
  SessionBadge  @stsgs/ui/ui/session-badge.tsx

Skipped: 2 patterns
  CustomPattern (user skipped)
  AnotherPattern (user skipped)

Changes in @stsgs/ui:
  +3 new files
  +2 modified index files

[Push to GitHub] [View changes] [Done]
```

---

## Important Rules

1. **NEVER modify source projects** - only read and extract
2. **Always parameterize colors** - use variant prop
3. **Keep components under 150 lines** - split if needed
4. **Use forwardRef** - for ref forwarding
5. **Use cn() from @/lib/utils** - not clsx directly
6. **Add data-slot** - on root element of every component
7. **Export types** - Props interfaces
8. **Add JSDoc** - with @example on main export
9. **Generalize domain terms** - no "oil", "military", "trading" in generic components
10. **Verify TypeScript compiles** - before committing

---

## Checklist Before Completion

- [ ] Scanned all specified projects
- [ ] Showed interactive selection for each pattern
- [ ] Parameterized all colors to variants
- [ ] Created component files in correct category
- [ ] All files pass anti-monolith rules (<=150 lines, <=3 useState)
- [ ] All components use forwardRef, cn(), data-slot
- [ ] JSDoc with @example on every main export
- [ ] Zero domain-specific references (generalized)
- [ ] Updated barrel exports (index.ts)
- [ ] Verified TypeScript compiles (npx tsc --noEmit)
- [ ] Git committed with descriptive message
- [ ] Pushed to GitHub repository

---

**Document complies with No-Unicode Policy v2.1**
