# Tailwind CSS Interactivity & Miscellaneous v4 — Complete Reference

> Source: https://tailwindcss.com/docs/cursor + pointer-events + resize + scroll + accent-color + caret-color + etc.
> Tailwind CSS v4.x

## 1. Cursor

`cursor-auto`, `cursor-default`, `cursor-pointer`, `cursor-wait`, `cursor-text`, `cursor-move`, `cursor-help`, `cursor-not-allowed`, `cursor-none`, `cursor-context-menu`, `cursor-progress`, `cursor-cell`, `cursor-crosshair`, `cursor-vertical-text`, `cursor-alias`, `cursor-copy`, `cursor-no-drop`, `cursor-grab`, `cursor-grabbing`, `cursor-zoom-in`, `cursor-zoom-out`

## 2. Pointer Events

`pointer-events-none`, `pointer-events-auto`

## 3. Resize

`resize-none`, `resize` (both), `resize-x`, `resize-y`

## 4. Scroll Behavior

`scroll-auto`, `scroll-smooth`

## 5. Scrollbar (new in v4)

`scrollbar-color-auto`, `scrollbar-width-auto`, `scrollbar-width-thin`, `scrollbar-width-none`
`scrollbar-gutter-auto`, `scrollbar-gutter-stable`, `scrollbar-gutter-stable-both-edges`

## 6. Scroll Margin / Padding

`scroll-m-{n}`, `scroll-mt-*`, `scroll-mb-*`, `scroll-ml-*`, `scroll-mr-*`
`scroll-p-{n}`, `scroll-pt-*`, etc.

## 7. Scroll Snap

**Type**: `snap-none`, `snap-x`, `snap-y`, `snap-both`, `snap-mandatory`, `snap-proximity`
**Align**: `snap-start`, `snap-end`, `snap-center`, `snap-align-none`
**Stop**: `snap-normal`, `snap-always`

## 8. Touch Action

`touch-auto`, `touch-none`, `touch-pan-x`, `touch-pan-y`, `touch-pan-left`, `touch-pan-right`, `touch-pan-up`, `touch-pan-down`, `touch-pinch-zoom`, `touch-manipulation`

## 9. Accent Color

`accent-red-500`, `accent-blue-600`, etc. (for checkbox, radio, range inputs)

## 10. Caret Color

`caret-red-500`, `caret-blue-600` (text input cursor color)

## 11. Appearance

`appearance-none`, `appearance-auto`

## 12. Field Sizing (new in v4)

`field-sizing-content`, `field-sizing-fixed`

## 13. User Select

`select-none`, `select-text`, `select-all`, `select-auto`

## 14. Will Change

`will-change-auto`, `will-change-scroll`, `will-change-contents`, `will-change-transform`

## 15. Color Scheme

`color-scheme-light`, `color-scheme-dark`, `color-scheme-normal` (native scrollbar coloring)

## 16. SVG

**Fill**: `fill-none`, `fill-current`, `fill-red-500`, `fill-[#1da1f2]`
**Stroke**: `stroke-none`, `stroke-current`, `stroke-red-500`
**Stroke Width**: `stroke-0` through `stroke-2` plus arbitrary

## 17. Common Patterns

```html
<button class="cursor-pointer disabled:cursor-not-allowed disabled:opacity-50">  <!-- Button states -->
<div class="pointer-events-none opacity-50">                                       <!-- Disabled overlay -->
<textarea class="resize-none">                                                     <!-- Fixed textarea -->
<div class="snap-x snap-mandatory overflow-x-auto">                                 <!-- Horizontal snap scroll -->
<input type="checkbox" class="accent-blue-600 size-5">                             <!-- Styled checkbox -->
```

## 18. Cross-Refs

- [Accessibility](skill_view(name='tailwind-css-documentation', file_path='references/accessibility.md'))
- [Borders](skill_view(name='tailwind-css-documentation', file_path='references/borders.md'))
