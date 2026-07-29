# Tailwind CSS Customization & Directives v4 — Complete Reference

> Source: https://tailwindcss.com/docs/theme + functions-and-directives + adding-custom-styles
> Tailwind CSS v4.x

## 1. Overview

Tailwind v4 is **CSS-first**. All customization happens in your CSS file using directives like `@theme`, `@utility`, `@variant`, and `@apply`. No JavaScript config file needed.

## 2. @theme — Design Tokens

The core of v4 customization. Declares theme variables that generate utility classes.

```css
@theme {
  --color-brand: oklch(0.6 0.2 260);
  --font-heading: "Inter", sans-serif;
  --spacing-13: 3.25rem;
  --breakpoint-3xl: 120rem;
}
```

### Theme Operations

| Operation | Syntax | Effect |
|---|---|---|
| **Extend** | `@theme { --color-brand: red; }` | Adds new utility + CSS variable |
| **Override** | `@theme { --breakpoint-sm: 30rem; }` | Changes existing value |
| **Reset namespace** | `@theme { --color-*: initial; ... }` | Removes all default colors |
| **Clean slate** | `@theme { --*: initial; ... }` | Removes all defaults |
| **Inline** | `@theme inline { --font-sans: var(--font-inter); }` | Resolves at computed-value time |
| **Static** | `@theme static { --color-primary: red; }` | Always generates CSS variable |

### Namespace Reference

| Namespace | Utilities Generated |
|---|---|
| `--color-*` | bg-, text-, border-, outline-, ring-, fill-, stroke-, caret-, accent-, decoration- |
| `--font-*` | font- |
| `--text-*` | text- (size + line-height) |
| `--font-weight-*` | font- |
| `--tracking-*` | tracking- |
| `--leading-*` | leading- |
| `--spacing-*` | p-, m-, w-, h-, gap-, inset-, scroll-m-, scroll-p-, space-, translate- |
| `--radius-*` | rounded- |
| `--shadow-*` | shadow- |
| `--inset-shadow-*` | inset-shadow- |
| `--drop-shadow-*` | drop-shadow- |
| `--blur-*` | blur- |
| `--breakpoint-*` | responsive variants (sm:*, md:*) |
| `--container-*` | max-w-* + @container variants |
| `--animate-*` | animate- + @keyframes |
| `--ease-*` | ease- |
| `--perspective-*` | perspective- |
| `--aspect-*` | aspect- |

### Animation with @keyframes

```css
@theme {
  --animate-slide-in: slide-in 0.3s ease-out;
  @keyframes slide-in {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }
}
```

## 3. @utility — Custom Utilities

```css
@utility text-balance {
  text-wrap: balance;
}

@utility card-hover {
  transition: box-shadow 0.2s, transform 0.2s;
  &:hover {
    box-shadow: var(--shadow-lg);
    transform: translateY(-4px);
  }
}

@utility scrollbar-hidden {
  scrollbar-width: none;
  &::-webkit-scrollbar { display: none; }
}

/* With modifiers (functional utilities) */
@utility tab-* {
  tab-size: var(--tab-size-);
}
```

## 4. @variant / @custom-variant

```css
/* Simple variant */
@variant pointer-coarse {
  @media (pointer: coarse) {
    @slot;
  }
}

/* Composable variant */
@custom-variant dark-landscape {
  .dark & {
    @media (orientation: landscape) {
      @slot;
    }
  }
}
```

## 5. @apply

```css
@layer components {
  .btn-primary {
    @apply inline-flex items-center gap-2 rounded-lg bg-blue-600 px-4 py-2 
           font-semibold text-white shadow-sm hover:bg-blue-500 
           focus-visible:outline-2 focus-visible:outline-offset-2 
           focus-visible:outline-blue-600;
  }
}
```

**When to use**: pattern repeated >3 times. Otherwise, inline utilities.

## 6. @source — Content Detection

```css
@source "../node_modules/my-library";
@source "../pages";
```

Tailwind auto-scans `./src/`, `./public/`, and Vite root. Use `@source` for external paths.

## 7. @reference

Import a CSS file for its variables/utilities without duplicating its output:
```css
@reference "../components/button.css";
```

## 8. @layer

```css
@layer base {
  h1 { @apply text-4xl font-bold; }
  a  { @apply text-blue-600 underline; }
}

@layer components {
  .card { @apply rounded-xl bg-white p-6 shadow-md; }
}
```

Cascade order: `theme` → `base` → `components` → `utilities`

## 9. @import

```css
/* Standard v4 import */
@import "tailwindcss";

/* Named layer imports */
@import "tailwindcss/theme.css" layer(theme);
@import "tailwindcss/preflight.css" layer(base);
@import "tailwindcss/utilities.css" layer(utilities);
```

## 10. Preflight (CSS Reset)

Included automatically. Disabled via:
```css
@import "tailwindcss";
@import "tailwindcss/theme.css" layer(theme);
/* Skip preflight import */
@import "tailwindcss/utilities.css" layer(utilities);
```

## 11. Adding Custom Styles

### Option 1: @layer
```css
@layer base {
  h1 { @apply text-4xl font-bold; }
}
```

### Option 2: Direct CSS (outside layers)
```css
.fancy-text {
  background: linear-gradient(45deg, #f06, #9f6);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
```

### Option 3: Inline arbitrary variants
```css
@custom-variant supports-grid {
  @supports (display: grid) {
    @slot;
  }
}
```

## 12. Sharing Themes Across Projects

```css
/* brand-theme.css */
@theme {
  --color-primary: oklch(0.6 0.2 260);
  --color-secondary: oklch(0.7 0.15 150);
}

/* app.css */
@import "tailwindcss";
@import "../shared/brand-theme.css";
```

Publishable to NPM as standalone CSS package.

## 13. Configuration Example (Complete)

```css
@import "tailwindcss";

@theme {
  --color-brand: oklch(0.6 0.2 260);
  --color-brand-dark: oklch(0.5 0.15 260);
  --font-heading: "Inter", sans-serif;
  --breakpoint-xs: 30rem;
  --animate-fade-in: fade-in 0.3s ease-out;
  @keyframes fade-in {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }
}

@utility card {
  border-radius: var(--radius-xl);
  padding: var(--spacing-6);
  background: var(--color-white);
  box-shadow: var(--shadow-md);
}

@custom-variant supports-grid {
  @supports (display: grid) {
    @slot;
  }
}

@layer base {
  body {
    @apply text-gray-900 antialiased;
    font-family: var(--font-heading);
  }
}

@layer components {
  .btn {
    @apply inline-flex items-center justify-center gap-2 rounded-lg 
           px-4 py-2 text-sm font-medium transition-colors duration-150 
           focus-visible:outline-2 focus-visible:outline-offset-2;
  }
  .btn-primary {
    @apply btn bg-blue-600 text-white hover:bg-blue-500 
           focus-visible:outline-blue-600;
  }
}
```

## 14. AI Notes

1. v4 is CSS-first — think in `@theme`, `@utility`, `@variant` not JS
2. `@theme { --color-*: initial; }` to remove defaults, then redefine
3. `@apply` in `@layer components` — never in `@layer utilities`
4. Use `@custom-variant` for reusable, composable variant logic
5. Use `@theme inline { ... }` when referencing other CSS variables
6. Share theme via `@import` of standalone CSS files
7. `@utility` automatically gets variant support (hover:, md:, dark:)
8. Animation keyframes go inside `@theme` to be bundled with the animate utility

## 15. Cross-Refs

- [Core Concepts](skill_view(name='tailwind-css-documentation', file_path='references/core-concepts.md'))
- [Installation](skill_view(name='tailwind-css-documentation', file_path='references/installation.md'))
- [Plugins](skill_view(name='tailwind-css-documentation', file_path='references/plugins.md'))
- [Optimization](skill_view(name='tailwind-css-documentation', file_path='references/optimization.md'))
