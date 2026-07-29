# Tailwind CSS Backgrounds v4 — Complete Reference

> Source: https://tailwindcss.com/docs/background-color + background-image + gradient + blend-mode
> Tailwind CSS v4.x

## 1. Overview

Background utilities: color, gradients, images, blending. Colors use `--color-*` namespace.

## 2. Background Color

`bg-white`, `bg-blue-500`, `bg-red-500/50` (opacity), `bg-[#1da1f2]`

## 3. Gradients

```html
<div class="bg-gradient-to-r from-blue-500 to-purple-600">
<div class="bg-gradient-to-r from-cyan-500 via-blue-500 to-purple-600">
<div class="bg-conic-to-t from-pink-500 via-red-500 to-yellow-500">
<div class="bg-radial-[at_25%_25%] from-white to-zinc-900">
```

Directions: `to-t`, `to-tr`, `to-r`, `to-br`, `to-b`, `to-bl`, `to-l`, `to-tl`

## 4. Background Clip

`bg-clip-border`, `bg-clip-padding`, `bg-clip-content`, `bg-clip-text`

**Gradient text:**
```html
<h1 class="bg-gradient-to-r from-blue-500 to-purple-600 bg-clip-text text-transparent">
  Gradient Text
</h1>
```

## 5. Other Background Properties

**Attachment**: `bg-fixed`, `bg-local`, `bg-scroll`
**Origin**: `bg-origin-border`, `bg-origin-padding`, `bg-origin-content`
**Position**: `bg-bottom`, `bg-center`, `bg-left`, etc.
**Repeat**: `bg-repeat`, `bg-no-repeat`, `bg-repeat-x`, `bg-repeat-y`, `bg-round`, `bg-space`
**Size**: `bg-auto`, `bg-cover`, `bg-contain`

## 6. Blend Modes

`bg-blend-normal`, `multiply`, `screen`, `overlay`, `darken`, `lighten`, `color-dodge`, `color-burn`, `hard-light`, `soft-light`, `difference`, `exclusion`, `hue`, `saturation`, `color`, `luminosity`

## 7. Common Patterns

```html
<!-- Hero with gradient overlay -->
<section class="relative bg-gradient-to-br from-blue-600 to-purple-700 text-white">
  <div class="absolute inset-0 bg-black/40"></div>
  <div class="relative z-10">Content</div>
</section>

<!-- Gradient text -->
<h2 class="bg-gradient-to-r from-amber-500 to-orange-600 bg-clip-text text-transparent">
```

## 8. Cross-Refs

- [Effects](skill_view(name='tailwind-css-documentation', file_path='references/effects-filters-animations.md'))
- [Customization](skill_view(name='tailwind-css-documentation', file_path='references/customization-directives.md'))
