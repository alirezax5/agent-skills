# Tailwind CSS Borders & Ring v4 — Complete Reference

> Source: https://tailwindcss.com/docs/border-width + border-radius + border-color + outline + ring + divide
> Tailwind CSS v4.x

## 1. Overview

Border utilities for width, radius, color, style, and divide. Ring for focus indicators.

## 2. Border Width

`border` (1px), `border-0`, `border-2`, `border-4`, `border-8`, `border-t-2`, `border-b-0`, `border-x-2`, `border-y-2`

## 3. Border Color

`border-gray-200`, `border-blue-500`, `border-red-500/50`, `border-[#1da1f2]`

## 4. Border Style

`border-solid` (default), `dashed`, `dotted`, `double`, `hidden`, `none`

## 5. Border Radius

`rounded-none`, `rounded-xs`(2px), `rounded-sm`(4px), `rounded`(4px), `rounded-md`(6px), `rounded-lg`(8px), `rounded-xl`(12px), `rounded-2xl`(16px), `rounded-3xl`(24px), `rounded-4xl`(32px), `rounded-full`(pill)

Per-corner: `rounded-t-*`, `rounded-r-*`, `rounded-b-*`, `rounded-l-*`, `rounded-tl-*`, `rounded-tr-*`, `rounded-br-*`, `rounded-bl-*`

## 6. Outline

`outline-0`, `outline-1`, `outline-2`, `outline-4`, `outline-8`
`outline-blue-500`, `outline-dashed`, `outline-offset-2`

## 7. Ring (Focus)

```html
<button class="ring-2 ring-blue-500 ring-offset-2 ring-offset-gray-100">
<button class="focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2">
```

`ring-0`, `ring-1`, `ring-2`, `ring-4`, `ring-8`. Color: `ring-{color}`. Offset: `ring-offset-0` through `ring-offset-8`. Inset: `ring ring-inset ring-blue-500/30`

## 8. Divide (Between Children)

```html
<div class="divide-y divide-gray-200">
  <div class="py-3">Item 1</div>
  <div class="py-3">Item 2</div>
</div>
```

`divide-x-{n}`, `divide-y-{n}`, `divide-x-reverse`, `divide-{color}`, `divide-{style}`

## 9. Common Patterns

```html
<div class="rounded-xl border border-gray-200 bg-white p-6 shadow-sm">  <!-- Card -->
<button class="rounded-lg border-2 border-blue-600 px-4 py-2 text-blue-600">  <!-- Outline button -->
<input class="rounded-md border border-gray-300 px-3 py-2 focus:border-blue-500 focus:ring-1">  <!-- Input -->
<img class="size-12 rounded-full ring-2 ring-white" src="" alt="">  <!-- Avatar -->
<ul class="divide-y divide-gray-100">  <!-- List -->
```

## 10. AI Notes

1. `rounded-lg` for cards, `rounded-xl` for larger cards
2. Ring > outline for focus (no layout shift)
3. `focus-visible:` for keyboard focus
4. `divide-y` cleaner than manual border-bottom per child
5. `rounded-full` + `ring-2` + `ring-white` for avatar stacks
6. Ring offset = element background color

## 11. Cross-Refs

- [Effects](skill_view(name='tailwind-css-documentation', file_path='references/effects-filters-animations.md'))
- [Interactivity](skill_view(name='tailwind-css-documentation', file_path='references/interactivity.md'))
