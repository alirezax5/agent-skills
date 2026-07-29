# Tailwind CSS Layout v4 — Complete Reference

> Source: https://tailwindcss.com/docs/display, position, overflow, z-index, float, visibility, object-fit
> Tailwind CSS v4.x

## 1. Overview

Layout utilities for positioning, display, and overflow behaviors. All are mobile-first responsive.

## 2. Display

| Class | CSS |
|---|---|
| `block` | `display: block` |
| `inline-block` | `display: inline-block` |
| `inline` | `display: inline` |
| `flex` | `display: flex` |
| `inline-flex` | `display: inline-flex` |
| `grid` | `display: grid` |
| `inline-grid` | `display: inline-grid` |
| `table` / `table-row` / `table-cell` | `display: table/row/cell` |
| `hidden` | `display: none` |
| `contents` | `display: contents` |
| `flow-root` | `display: flow-root` |
| `list-item` | `display: list-item` |

Responsive: `md:flex`, `lg:grid`, `sm:hidden`

## 3. Container

```html
<div class="container mx-auto px-4">
```

Sets `max-width` matching the current breakpoint. Always pair with `mx-auto` and `px-*`.

## 4. Position

| Class | CSS |
|---|---|
| `static` | `position: static` |
| `fixed` | `position: fixed` |
| `absolute` | `position: absolute` |
| `relative` | `position: relative` |
| `sticky` | `position: sticky` |

### Top/Right/Bottom/Left

`inset-0` (all sides), `inset-x-0` (left+right), `inset-y-0` (top+bottom), `top-0`, `right-0`, `bottom-0`, `left-0`, `top-1/2`, `left-1/2`, `top-full`, `top-auto`, `top-[17px]`

**Centering patterns:**
```html
<!-- Flex centering -->
<div class="absolute inset-0 flex items-center justify-center">

<!-- Transform centering -->
<div class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2">
```

## 5. Overflow

`overflow-auto`, `overflow-hidden`, `overflow-clip`, `overflow-visible`, `overflow-scroll`; plus `overflow-x-*` and `overflow-y-*`.

## 6. Z-index

`z-0` through `z-50`, plus `z-auto`, `z-[-1]` (arbitrary).

## 7. Float & Clear

`float-start`, `float-end`, `float-right`, `float-left`, `float-none`
`clear-start`, `clear-end`, `clear-right`, `clear-left`, `clear-both`, `clear-none`

## 8. Visibility

`visible`, `invisible`, `collapse`

## 9. Box Sizing

`box-border` (default), `box-content`

## 10. Object Fit & Position

`object-contain`, `object-cover`, `object-fill`, `object-none`, `object-scale-down`
Position: `object-bottom`, `object-center`, `object-left`, etc.

## 11. Common Patterns

```html
<!-- Sticky header -->
<header class="sticky top-0 z-50 bg-white shadow-sm">

<!-- Full-height sidebar layout -->
<div class="flex h-screen">
  <aside class="w-64 overflow-y-auto shrink-0">Sidebar</aside>
  <main class="flex-1 overflow-y-auto p-6">Content</main>
</div>

<!-- Cover section -->
<section class="relative min-h-screen flex items-center justify-center overflow-hidden">
  <img class="absolute inset-0 w-full h-full object-cover" src="..." alt="">
  <div class="relative z-10 text-center text-white">Content</div>
</section>
```

## 12. AI Reasoning Notes

1. `flex` for 1D, `grid` for 2D
2. `hidden` removes from flow; `invisible` keeps space
3. `container mx-auto px-4` is the standard responsive container
4. `sticky top-0` for headers
5. `overflow-auto` on scrollable container, not body
6. `object-cover` for backgrounds, `object-contain` for product images
7. `shrink-0` on fixed-width sidebars prevents collapse

## 13. Cross-References

- [Flexbox & Grid](skill_view(name='tailwind-css-documentation', file_path='references/flexbox-grid.md'))
- [Spacing](skill_view(name='tailwind-css-documentation', file_path='references/spacing.md'))
- [Sizing](skill_view(name='tailwind-css-documentation', file_path='references/sizing.md'))
- [Responsive](skill_view(name='tailwind-css-documentation', file_path='references/responsive.md'))
