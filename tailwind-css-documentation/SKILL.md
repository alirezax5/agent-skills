---
name: "tailwind-css-documentation"
title: "Tailwind CSS v4 — Complete Developer Skill Library"
description: "Production-grade AI Skill Library built from official tailwindcss.com/docs. 20+ reference files covering: Installation, Core Concepts (@theme, @utility, @variant), Layout, Flexbox/Grid, Spacing, Sizing, Typography, Backgrounds, Borders, Effects, Filters, Transforms, Transitions, Animations, Responsive Design, Dark Mode, Customization, Directives & Functions, Plugins, Optimization, Accessibility, Recipes. Tailwind CSS v4+ with @theme paradigm. Cross-index included."
version: "4.0"
tailwind_version: "4.x"
author: "Hermes Agent"
last_updated: "2026-07-30"
category: "software-development"
tags: ["tailwindcss", "css", "utility-framework", "v4", "responsive-design", "dark-mode", "frontend"]
keywords: ["tailwind-v4", "utility-first", "@theme", "@utility", "@variant", "CSS-cascade-layers"]
official_source: "https://tailwindcss.com/docs/"
difficulty: "beginner-to-advanced"
---

# Tailwind CSS v4 — Complete Developer Reference

## Quick Fetch Trick (url-to-content)

```bash
# Tailwind CSS docs are a Next.js SPA — use these URLs for reference:
# https://tailwindcss.com/docs/<topic>
# The key v4 pages:
open https://tailwindcss.com/docs/installation/using-vite
open https://tailwindcss.com/docs/styling-with-utility-classes
open https://tailwindcss.com/docs/responsive-design
open https://tailwindcss.com/docs/dark-mode
open https://tailwindcss.com/docs/theme
open https://tailwindcss.com/docs/functions-and-directives
open https://tailwindcss.com/docs/adding-custom-styles
open https://tailwindcss.com/docs/detecting-classes-in-source-files
open https://tailwindcss.com/docs/upgrade-guide
```

## Reference Files (20+ total)

| # | File | Content | Load Command |
|---|---|---|---|
| — | **SKILL.md** (this file) | Index, cross-refs, architecture | `skill_view(name='tailwind-css-documentation')` |
| — | **references/index.md** | Cross-reference matrix, dependency chains | `file_path='references/index.md'` |
| 1 | **references/installation.md** | Vite, CLI, PostCSS, CDN, Framework guides, Upgrade | `file_path='references/installation.md'` |
| 2 | **references/core-concepts.md** | Utility-first workflow, States, Dark Mode, Theme variables, @theme, @utility, @variant, @custom-variant, @source, @reference, @apply, Cascade Layers | `file_path='references/core-concepts.md'` |
| 3 | **references/layout.md** | Container, Display, Position, Overflow, Z-index, Float, Visibility, Isolation, Object-fit | `file_path='references/layout.md'` |
| 4 | **references/flexbox-grid.md** | Flex, Grid, Gap, Order, Align, Justify, Place items/content | `file_path='references/flexbox-grid.md'` |
| 5 | **references/spacing.md** | Margin, Padding, Space Between | `file_path='references/spacing.md'` |
| 6 | **references/sizing.md** | Width, Height, Min/Max, Size utilities | `file_path='references/sizing.md'` |
| 7 | **references/typography.md** | Font family, size, weight, tracking, leading, text align, decoration, transform | `file_path='references/typography.md'` |
| 8 | **references/backgrounds.md** | Background color, gradient, image, blend modes, clip, origin | `file_path='references/backgrounds.md'` |
| 9 | **references/borders.md** | Border width, radius, color, outline, ring, divide | `file_path='references/borders.md'` |
| 10 | **references/effects.md** | Box shadow, text shadow, opacity, mix-blend, backdrop-filter | `file_path='references/effects.md'` |
| 11 | **references/filters.md** | blur, brightness, contrast, drop-shadow, grayscale, hue-rotate, invert, saturate, sepia | `file_path='references/filters.md'` |
| 12 | **references/transforms.md** | Scale, rotate, translate, skew, transform-origin, perspective | `file_path='references/transforms.md'` |
| 13 | **references/transitions-animations.md** | Transition property, duration, timing, delay, animation, keyframes | `file_path='references/transitions-animations.md'` |
| 14 | **references/responsive.md** | Breakpoints, mobile-first, responsive variants, container queries | `file_path='references/responsive.md'` |
| 15 | **references/dark-mode.md** | Dark mode with class/media selector, custom dark variants | `file_path='references/dark-mode.md'` |
| 16 | **references/accessibility.md** | Screen readers, forced colors, focus-visible, reduced-motion, semantic HTML | `file_path='references/accessibility.md'` |
| 17 | **references/customization.md** | @theme, theme variable namespaces, colors, fonts, spacing, overriding defaults, sharing themes | `file_path='references/customization.md'` |
| 18 | **references/directives.md** | @import, @theme, @utility, @variant, @custom-variant, @source, @reference, @apply, @config, @plugin, @layer | `file_path='references/directives.md'` |
| 19 | **references/plugins.md** | Official plugins, custom plugins via @plugin, third-party | `file_path='references/plugins.md'` |
| 20 | **references/optimization.md** | Content scanning, tree-shaking, production build, bundle size | `file_path='references/optimization.md'` |
| 21 | **references/interactivity.md** | Cursor, pointer-events, resize, scroll, accent-color, carent-color, field-sizing | `file_path='references/interactivity.md'` |
| 22 | **references/debugging-troubleshooting.md** | Unknown class, missing styles, wrong variants, build failures, config issues | `file_path='references/debugging-troubleshooting.md'` |
| 23 | **references/recipes.md** | Navbar, Sidebar, Card, Modal, Form, Table, Dashboard, Hero, Pricing, E-commerce | `file_path='references/recipes.md'` |

## Tailwind CSS v4 Architecture

```mermaid
flowchart LR
    A[CSS Input<br/>@import 'tailwindcss'] --> B[Content Scanner<br/>@source config]
    B --> C[Class Detection<br/>Scans HTML/JS/TSX]
    C --> D[CSS Generation<br/>@theme vars → utilities]
    D --> E[Optimization<br/>Tree-shaking unused]
    E --> F[Final CSS Bundle]
    
    G[@theme<br/>Design Tokens] --> D
    H[@utility<br/>Custom utilities] --> D
    I[@variant<br/>Custom variants] --> D
    J[@plugin<br/>Third-party] --> D
```

## v4 vs v3 Key Differences

| Feature | v3 | v4 |
|---|---|---|
| **Configuration** | `tailwind.config.js` | `@theme` in CSS |
| **Theme system** | JavaScript object | CSS variables + `@theme` |
| **Custom utilities** | `plugin()` in config | `@utility` directive |
| **Custom variants** | `addVariant()` plugin | `@variant` / `@custom-variant` |
| **Cascade layers** | Not used | `@layer theme, base, components, utilities` |
| **Content detection** | `content[]` in config | `@source` directive + auto-detection |
| **Preflight** | Imported separately | `@import 'tailwindcss/preflight'` |
| **Static utilities** | Config-based | Class detection scans source files |
| **Colors** | HEX/RGB by default | OKLCH by default |
| **Arbitrary values** | `[value]` syntax | Same, but more powerful |
| **Plugins** | JS plugins via config | CSS `@plugin` + JS plugins |
