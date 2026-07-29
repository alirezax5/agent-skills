# Tailwind CSS Sizing v4 — Complete Reference

> Source: https://tailwindcss.com/docs/width + height + min-width + max-width + min-height + max-height + size
> Tailwind CSS v4.x

## 1. Overview

Sizing utilities control width, height, and dimension constraints. Driven by `--spacing-*` (each = 0.25rem).

## 2. Width

`w-0` to `w-96`, `w-px`, `w-auto`, `w-1/2`, `w-1/3`, `w-2/3`, `w-1/4`, `w-3/4`, `w-1/5` through `w-4/5`, `w-1/6`, `w-5/6`, `w-full`, `w-screen`, `w-svw`, `w-lvw`, `w-dvw`, `w-min`, `w-max`, `w-fit`

Arbitrary: `w-[17px]`, `w-[calc(100%-2rem)]`

## 3. Height

Same pattern: `h-0` to `h-96`, `h-px`, `h-auto`, `h-1/2`, `h-full`, `h-screen`, `h-svh`, `h-lvh`, `h-dvh`, `h-min`, `h-max`, `h-fit`

## 4. Min-Width

`min-w-0` (overflow fix!), `min-w-full`, `min-w-min`, `min-w-max`, `min-w-fit`

## 5. Max-Width

`max-w-0`/`max-w-none`/`max-w-xs`(20rem)/`max-w-sm`(24rem)/`max-w-md`(28rem)/`max-w-lg`(32rem)/`max-w-xl`(36rem)/`max-w-2xl`(42rem)/`max-w-3xl`(48rem)/`max-w-4xl`(56rem)/`max-w-5xl`(64rem)/`max-w-6xl`(72rem)/`max-w-7xl`(80rem)/`max-w-full`/`max-w-min`/`max-w-max`/`max-w-fit`/`max-w-prose`(65ch)

## 6. Min-Height

`min-h-0`, `min-h-full`, `min-h-screen`, `min-h-svh`, `min-h-dvh`

## 7. Max-Height

`max-h-0` to `max-h-96`, `max-h-none`, `max-h-full`, `max-h-screen`

## 8. Size (Logical)

`size-{n}` — both width and height: `size-16` = 64×64

## 9. Common Patterns

```html
<section class="min-h-screen flex items-center">        <!-- Full viewport hero -->
<article class="mx-auto max-w-prose">                    <!-- Readable content -->
<aside class="w-64 shrink-0">                            <!-- Fixed sidebar -->
<main class="min-w-0 flex-1">                            <!-- Overflow-safe content -->
<img class="size-16 rounded-full" src="" alt="">         <!-- Avatar -->
```

## 10. AI Notes

1. `min-w-0` fixes flex overflow shrink bug
2. `min-h-screen` for full-page sections
3. `w-full + max-w-{size} + mx-auto` for constrained containers
4. `max-w-prose` (65ch) for optimal reading
5. `size-*` great for avatars/icons

## 11. Cross-Refs

- [Spacing](skill_view(name='tailwind-css-documentation', file_path='references/spacing.md'))
- [Layout](skill_view(name='tailwind-css-documentation', file_path='references/layout.md'))
