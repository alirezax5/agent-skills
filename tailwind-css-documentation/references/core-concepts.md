# Tailwind CSS Core Concepts v4 — Complete Reference

> Source: https://tailwindcss.com/docs/styling-with-utility-classes + responsive-design + dark-mode + theme
> Tailwind CSS v4.x

## 1. Overview

Tailwind CSS is a **utility-first** CSS framework. Instead of writing custom CSS, you compose designs using small, single-purpose utility classes directly in your HTML.

### Utility-First Workflow

**Traditional CSS:**
```css
.card { background: white; padding: 24px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
```
```html
<div class="card">...</div>
```

**Tailwind:**
```html
<div class="rounded-xl bg-white p-6 shadow-md">...</div>
```

## 2. Cascade Layers Architecture

```css
@layer theme, base, components, utilities;
@import "tailwindcss/theme.css" layer(theme);
@import "tailwindcss/preflight.css" layer(base);
@import "tailwindcss/utilities.css" layer(utilities);
```

| Layer | Content | Override |
|---|---|---|
| **theme** | CSS variables (design tokens) | `@theme` |
| **base** | Preflight (CSS reset) | `@layer base` |
| **components** | Component styles | `@layer components` |
| **utilities** | Utility classes | Lowest priority |

## 3. @theme — Design Tokens

```css
@theme {
  --color-mint-500: oklch(0.72 0.11 178);
  --font-display: "Inter", sans-serif;
  --breakpoint-3xl: 120rem;
  --spacing-18: 4.5rem;
}
```

### Behavioral Namespaces

| Namespace | What it generates | Example |
|---|---|---|
| `--color-*` | bg-, text-, border-, fill-, stroke- | `bg-mint-500` |
| `--font-*` | font- family | `font-display` |
| `--text-*` | Font size + line height | `text-base` |
| `--font-weight-*` | Font weight | `font-bold` |
| `--tracking-*` | Letter spacing | `tracking-tight` |
| `--leading-*` | Line height | `leading-relaxed` |
| `--spacing-*` | p-, m-, w-, h-, gap- | `p-18` |
| `--radius-*` | rounded- | `rounded-xl` |
| `--shadow-*` | shadow- | `shadow-md` |
| `--breakpoint-*` | Responsive variants | `3xl:*` |
| `--animate-*` | animate- + keyframes | `animate-spin` |
| `--blur-*` | blur- | `blur-md` |
| `--ease-*` | Transition timing | `ease-out` |
| `--perspective-*` | perspective- | `perspective-near` |
| `--aspect-*` | aspect- | `aspect-video` |
| `--container-*` | max-w- + @container | `max-w-md` |

### Theme Operations

```css
/* Extend */
@theme { --color-brand: #ff69b4; }

/* Override */
@theme { --breakpoint-sm: 30rem; }

/* Reset namespace */
@theme { --color-*: initial; --color-white: #fff; }

/* Clean slate */
@theme { --*: initial; }
```

### inline option — resolves at computed-value time:
```css
@theme inline { --font-sans: var(--font-inter); }
```

### static option — always generates CSS variable:
```css
@theme static { --color-primary: var(--color-red-500); }
```

## 4. @utility — Custom Utilities

```css
@utility card {
  border-radius: var(--radius-xl);
  padding: var(--spacing-6);
  box-shadow: var(--shadow-md);
  background-color: var(--color-white);
}

@utility tab-* {
  tab-size: var(--tab-size-);
}

@utility text-balance {
  text-wrap: balance;
}
```

Usage:
```html
<div class="card hover:card md:card">...</div>
```

## 5. @variant / @custom-variant

### Built-in Variants

| Variant | CSS | Usage |
|---|---|---|
| `hover:*` | `:hover` | `hover:bg-blue-500` |
| `focus:*` | `:focus` | `focus:outline-2` |
| `active:*` | `:active` | `active:scale-95` |
| `disabled:*` | `:disabled` | `disabled:opacity-50` |
| `visited:*` | `:visited` | `visited:text-purple-600` |
| `group-hover:*` | Parent hover | `group-hover:opacity-100` |
| `group-focus:*` | Parent focus | `group-focus:block` |
| `peer-hover:*` | Sibling hover | `peer-hover:opacity-100` |
| `focus-visible:*` | `:focus-visible` | `focus-visible:ring-2` |
| `focus-within:*` | `:focus-within` | `focus-within:ring` |
| `motion-safe:*` | prefers-reduced-motion: no-preference | `motion-safe:animate-spin` |
| `motion-reduce:*` | prefers-reduced-motion: reduce | `motion-reduce:transition-none` |
| `dark:*` | `.dark` selector | `dark:bg-gray-900` |
| `print:*` | `@media print` | `print:hidden` |
| `portrait:*` / `landscape:*` | Orientation | `portrait:flex-col` |
| `rtl:*` / `ltr:*` | `[dir="rtl"]` / `[dir="ltr"]` | `rtl:text-right` |
| `open:*` | `[open]` | `open:bg-gray-50` |
| `inert:*` | `:inert` | `inert:opacity-50` |
| `starting:*` | `@starting-style` | `starting:opacity-0` |
| `selection:*` | `::selection` | `selection:bg-yellow-200` |
| `first-letter:*` | `::first-letter` | `first-letter:text-4xl` |
| `placeholder:*` | `::placeholder` | `placeholder:text-gray-400` |
| `file:*` | `::file-selector-button` | `file:bg-blue-500` |
| `backdrop:*` | `::backdrop` | `backdrop:bg-black/50` |
| `before:*` / `after:*` | `::before`/`::after` | `before:content-['']` |
| `first:*` / `last:*` | `:first-child` / `:last-child` | `first:rounded-t` |
| `odd:*` / `even:*` | `:nth-child(odd/even)` | `odd:bg-gray-50` |
| `only:*` | `:only-child` | `only:block` |
| `not-*:` | `:not()` | `not-last:border-b` |

### Custom Variants

```css
@variant pointer-coarse {
  @media (pointer: coarse) { @slot; }
}

@custom-variant dark-landscape {
  .dark & {
    @media (orientation: landscape) { @slot; }
  }
}
```

## 6. @source & @reference

```css
@source "../node_modules/my-library";
@reference "../components/button.css";
```

`@source` adds paths for class scanning. `@reference` imports variables/utilities without duplicating output.

## 7. @apply

```css
.btn-primary {
  @apply inline-flex items-center gap-2 rounded-lg bg-blue-600 px-4 py-2 
         font-semibold text-white shadow-sm hover:bg-blue-500 
         focus-visible:outline-2 focus-visible:outline-blue-600;
}
```

Best practice: use in `@layer components`, only when pattern repeats >3 times.

## 8. Responsive Design (Mobile-First)

```html
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
```

Default breakpoints: `sm: 40rem`, `md: 48rem`, `lg: 64rem`, `xl: 80rem`, `2xl: 96rem`
Container queries: `@sm:*`, `@md:*`, `@lg:*`, `@xl:*`, `@2xl:*`

## 9. Dark Mode

```html
<body class="dark:bg-gray-900 dark:text-white">
  <h1 class="dark:text-blue-300">Hello</h1>
</body>
```

Activates when parent has `dark` class (or `@media (prefers-color-scheme: dark)`).

## 10. Arbitrary Values & Properties

```html
<div class="mt-[17px] bg-[#1da1f2] text-[clamp(1rem,5vw,2rem)]">
<div class="[color:red] [scrollbar-width:thin]">
```

Type annotations: `color:`, `length:`, `number:`, `percentage:`, `url:`, `position:`

## 11. AI Reasoning Notes

1. Utility-first first — extract to `@apply` only when repeating >3 times
2. Use `@theme` for all design tokens, never JS config
3. Mobile-first: base styles for mobile, `md:*` for larger
4. `@slot` in `@variant` is the insertion point for the utility class
5. Arbitrary values for one-offs; `@theme` for reuse
6. v4 has **no IE11** support

## 12. Cross-References

- [Installation](skill_view(name='tailwind-css-documentation', file_path='references/installation.md'))
- [Directives](skill_view(name='tailwind-css-documentation', file_path='references/directives.md'))
- [Customization](skill_view(name='tailwind-css-documentation', file_path='references/customization.md'))
- [Responsive](skill_view(name='tailwind-css-documentation', file_path='references/responsive.md'))
