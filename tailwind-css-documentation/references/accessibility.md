# Tailwind CSS Accessibility v4 — Complete Reference

> Source: https://tailwindcss.com/docs/accessibility
> Tailwind CSS v4.x

## 1. Overview

Tailwind provides utilities and variants for building accessible interfaces.

## 2. Focus Management

**Focus-Visible** (keyboard only):
```html
<button class="focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-500">
```

**Focus-Within** (parent on focus):
```html
<div class="focus-within:ring-2 focus-within:ring-blue-500 rounded-lg">
  <input class="outline-none p-2">
</div>
```

## 3. Reduced Motion

```html
<div class="motion-safe:animate-spin motion-reduce:transition-none">
```

`motion-safe:*` — only when user allows motion
`motion-reduce:*` — when prefers-reduced-motion: reduce

## 4. Screen Reader

```html
<span class="sr-only">Visible only to screen readers</span>
<span class="not-sr-only">Visible to everyone</span>
```

## 5. Semantic HTML

✅ Use semantic elements + Tailwind:
```html
<nav aria-label="Main navigation" class="flex gap-4">
<button aria-label="Close" class="...">
<main role="main">
```

❌ Avoid div soup with `role="..."`

## 6. Color & Contrast

- Minimum 4.5:1 contrast ratio for normal text
- Don't rely solely on color to convey info
- Test with both light and dark modes
- `text-transparent` + `bg-clip-text` needs color fallback

## 7. AI Notes

1. Always `focus-visible:` for focus styles (not `focus:`)
2. Never `focus:outline-none` without visible alternative
3. `sr-only` for screen-reader labels
4. `motion-safe:` for animations, `motion-reduce:` for fallbacks
5. Use `<button>` not `<div role="button">`

## 8. Cross-Refs

- [Dark Mode](skill_view(name='tailwind-css-documentation', file_path='references/dark-mode.md'))
- [Effects & Animations](skill_view(name='tailwind-css-documentation', file_path='references/effects-filters-animations.md'))
