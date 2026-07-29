# Tailwind CSS Typography v4 — Complete Reference

> Source: https://tailwindcss.com/docs/font-family + font-size + font-weight + text-align + etc.
> Tailwind CSS v4.x

## 1. Overview

Typography utilities: font family, size, weight, line-height (leading), letter-spacing (tracking), text alignment, decoration, transform, overflow, wrapping.

Driven by: `--font-*`, `--text-*`, `--font-weight-*`, `--tracking-*`, `--leading-*`.

## 2. Font Family

`font-sans`, `font-serif`, `font-mono`. Custom: `@theme { --font-display: "Inter", sans-serif; }` → `font-display`

## 3. Font Size

`text-xs` (12px), `text-sm` (14px), `text-base` (16px), `text-lg` (18px), `text-xl` (20px), `text-2xl` (24px) through `text-9xl` (128px)

Arbitrary: `text-[1.25rem]`, `text-[clamp(1rem,3vw,2rem)]`

## 4. Font Weight

`font-thin`(100), `font-extralight`(200), `font-light`(300), `font-normal`(400), `font-medium`(500), `font-semibold`(600), `font-bold`(700), `font-extrabold`(800), `font-black`(900)

## 5. Line Height (Leading)

`leading-3`(12px) through `leading-10`(40px), plus `leading-none`(1), `leading-tight`(1.25), `leading-snug`(1.375), `leading-normal`(1.5), `leading-relaxed`(1.625), `leading-loose`(2)

## 6. Letter Spacing (Tracking)

`tracking-tighter`(-.05em), `tracking-tight`(-.025em), `tracking-normal`, `tracking-wide`(.025em), `tracking-wider`(.05em), `tracking-widest`(.1em)

## 7. Text Alignment: `text-left`, `text-center`, `text-right`, `text-justify`, `text-start`, `text-end`
## 8. Text Decoration: `underline`, `overline`, `line-through`, `no-underline`
## 9. Color: `decoration-red-500`, `decoration-[#1da1f2]`
## 10. Style: `decoration-solid`, `double`, `dotted`, `dashed`, `wavy`
## 11. Thickness: `decoration-0` through `decoration-8`
## 12. Offset: `underline-offset-auto`, `underline-offset-0` through `underline-offset-8`
## 13. Transform: `uppercase`, `lowercase`, `capitalize`, `normal-case`

## 14. Text Overflow & Wrap

`truncate` (ellipsis+nowrap+overflow), `text-ellipsis`, `text-clip`, `text-wrap`, `text-nowrap`, `text-balance`, `text-pretty`
`break-normal`, `break-words`, `break-all`
`whitespace-normal/nowrap/pre/pre-line/pre-wrap`

## 15. Line Clamp

`line-clamp-1` through `line-clamp-6`, `line-clamp-none`

## 16. Common Patterns

```html
<!-- Article -->
<article class="prose prose-lg mx-auto max-w-prose">
  <h1 class="text-4xl font-bold tracking-tight">Title</h1>
  <p class="text-lg leading-relaxed text-gray-700">Body...</p>
</article>

<!-- Card -->
<h3 class="text-xl font-semibold line-clamp-2">Title that wraps</h3>
<p class="text-sm text-gray-500 mt-1 line-clamp-3">Excerpt</p>

<!-- Hero -->
<h1 class="text-5xl md:text-7xl font-extrabold tracking-tight text-balance">
```

## 17. AI Notes

1. `text-base` = body, `text-sm` = secondary, `text-xs` = labels
2. `tracking-tight` for large headings
3. `leading-relaxed` for article body
4. `truncate` and `line-clamp-*` for card excerpts
5. `text-balance` / `text-pretty` avoid orphan words in headings
6. Install `@tailwindcss/typography` for `prose` class

## 18. Cross-Refs

- [Spacing](skill_view(name='tailwind-css-documentation', file_path='references/spacing.md'))
- [Customization](skill_view(name='tailwind-css-documentation', file_path='references/customization.md'))
- [Responsive](skill_view(name='tailwind-css-documentation', file_path='references/responsive.md'))
