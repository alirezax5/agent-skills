# Tailwind CSS Responsive Design v4 — Complete Reference

> Source: https://tailwindcss.com/docs/responsive-design + breakpoints
> Tailwind CSS v4.x

## 1. Overview

Tailwind uses **mobile-first** responsive design. Base styles target mobile, variant prefixes target larger screens.

```html
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
```

- Mobile: 1 column
- md (≥48rem): 2 columns
- lg (≥64rem): 4 columns

## 2. Default Breakpoints

| Variant | Min-Width | CSS Media Query |
|---|---|---|
| `sm:*` | 40rem (640px) | `@media (width >= 40rem)` |
| `md:*` | 48rem (768px) | `@media (width >= 48rem)` |
| `lg:*` | 64rem (1024px) | `@media (width >= 64rem)` |
| `xl:*` | 80rem (1280px) | `@media (width >= 80rem)` |
| `2xl:*` | 96rem (1536px) | `@media (width >= 96rem)` |

## 3. Custom Breakpoints

```css
@theme {
  --breakpoint-3xl: 120rem;
  --breakpoint-sm: 30rem;  /* override default */
}
```
Usage: `3xl:grid-cols-6`

## 4. Container Queries

Container-queries use `@` prefix instead of responsive-prefix:

`@sm:*`, `@md:*`, `@lg:*`, `@xl:*`, `@2xl:*`

```html
<div class="@container">  <!-- define container -->
  <div class="grid grid-cols-1 @md:grid-cols-2 @lg:grid-cols-3">
    <!-- Responds to container width, not viewport -->
  </div>
</div>
```

Custom container query breakpoints via `--container-*`:
```css
@theme { --container-3xl: 50rem; }
```

## 5. Responsive Utility Class Combinations

```html
<!-- Text sizes -->
<p class="text-base md:text-lg lg:text-xl">

<!-- Spacing -->
<section class="p-4 md:p-8 lg:p-12">

<!-- Display -->
<div class="hidden md:block">Desktop only</div>
<div class="block md:hidden">Mobile only</div>

<!-- Flex direction -->
<div class="flex-col md:flex-row">

<!-- Width -->
<div class="w-full sm:w-1/2 lg:w-1/3">
```

## 6. Range Breakpoints (between sizes)

```html
<!-- Only between sm and lg (inclusive) -->
<div class="max-sm:hidden lg:hidden">Visible sm-lg</div>
```

## 7. Max-Width Variants

`max-sm:*` (≤40rem), `max-md:*` (≤48rem), `max-lg:*`, `max-xl:*`, `max-2xl:*`

## 8. Common Responsive Patterns

```html
<!-- Mobile hamburger → desktop sidebar -->
<nav class="flex flex-col md:flex-row">
  <button class="md:hidden">☰</button>
  <div class="hidden md:flex md:gap-4">Links</div>
</nav>

<!-- Responsive card grid -->
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">

<!-- Responsive padding -->
<main class="px-4 sm:px-6 lg:px-8 max-w-7xl mx-auto">
```

## 9. AI Notes

1. Always design **mobile-first** — base classes for mobile, then `sm:*`/`md:*`/`lg:*` up
2. `hidden md:block` pattern for "desktop-only" elements
3. `block md:hidden` pattern for "mobile-only" elements
4. Container queries (`@md:*`) are for reusable components (cards, widgets)
5. Viewport queries (`md:*`) control page layout
6. Avoid too many breakpoints — `md` and `lg` cover most needs
7. Use `max-*:` variants sparingly (logic inversion is confusing)

## 10. Cross-Refs

- [Layout](skill_view(name='tailwind-css-documentation', file_path='references/layout.md'))
- [Flexbox & Grid](skill_view(name='tailwind-css-documentation', file_path='references/flexbox-grid.md'))
- [Customization](skill_view(name='tailwind-css-documentation', file_path='references/customization.md'))
