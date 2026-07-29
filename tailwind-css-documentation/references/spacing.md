# Tailwind CSS Spacing v4 — Complete Reference

> Source: https://tailwindcss.com/docs/padding + margin + space-between
> Tailwind CSS v4.x

## 1. Overview

Spacing utilities replace margin/padding CSS. Uses `--spacing-*` theme namespace. Default: `--spacing: 0.25rem` (4px base). Each integer N = N × 0.25rem = N × 4px.

## 2. Padding

| Class | CSS | Example |
|---|---|---|
| `p-{n}` | `padding: n × 0.25rem` | `p-4` = 1rem |
| `px-{n}` | `padding-left/right` | `px-4` = 1rem horizontal |
| `py-{n}` | `padding-top/bottom` | `py-4` = 1rem vertical |
| `pt-{n}` / `pr-{n}` / `pb-{n}` / `pl-{n}` | Per-side | `pt-4` |
| `ps-{n}` / `pe-{n}` | Logical (inline-start/end) | `ps-4` |

Values: 0-96 (4px inc), plus `px`, `0.5`, `1.5`, `2.5`, `3.5`. Arbitrary: `p-[17px]`

## 3. Margin

Same pattern: `m-{n}`, `mx-{n}`, `my-{n}`, `mt-*`, `mr-*`, `mb-*`, `ml-*`, `ms-*`, `me-*`

Special: `mx-auto` (centering), `-m-{n}` (negative), `m-px`

## 4. Space Between

```html
<div class="flex flex-col space-y-4">
  <div>Item 1</div>
  <div>Item 2</div>
</div>
```

`space-x-{n}`, `space-y-{n}`, `space-x-reverse`, `space-y-reverse`

**Note**: `gap-*` on flex/grid is preferred over `space-*`.

## 5. Common Patterns

```html
<!-- Card with padding -->
<div class="rounded-xl bg-white p-6 shadow-sm">
  <h3 class="text-lg font-semibold mb-2">Title</h3>
  <p class="text-gray-600">Content</p>
</div>

<!-- Centered container -->
<div class="mx-auto max-w-2xl px-4">

<!-- Button -->
<button class="rounded-lg bg-blue-600 px-4 py-2 text-white">Click</button>

<!-- Section spacing -->
<section class="py-16 md:py-24">
```

## 6. AI Notes

1. `gap-*` > `space-*` for flex/grid (predictable)
2. `mx-auto` for horizontal centering of block elements
3. `py-*` for vertical section spacing
4. Convention: outer container `p-6`, card inner `p-4`

## 7. Cross-Refs

- [Sizing](skill_view(name='tailwind-css-documentation', file_path='references/sizing.md'))
- [Flexbox & Grid](skill_view(name='tailwind-css-documentation', file_path='references/flexbox-grid.md'))
