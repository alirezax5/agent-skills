# Tailwind CSS Dark Mode v4 — Complete Reference

> Source: https://tailwindcss.com/docs/dark-mode
> Tailwind CSS v4.x

## 1. Overview

Dark mode in Tailwind v4 uses the `dark:*` variant. It activates when a parent element has the `dark` class.

```html
<body class="bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100">
  <h1 class="dark:text-blue-300">Dark mode title</h1>
</body>
```

## 2. How it Works

The built-in `dark` variant is defined as:

```css
@variant dark (&:where(.dark, .dark *));
```

This means: apply the utility when any ancestor has class `.dark`.

## 3. Toggle Strategies

### Class Toggle (Recommended)
Most common: toggle the `dark` class on `<html>` using JS:
```js
document.documentElement.classList.toggle('dark')
```

### Media Query (System Preference)
```css
@variant dark (&:where(@media (prefers-color-scheme: dark) { & }));
```

Or combine both:
```css
@custom-variant dark (&:where(.dark, .dark *));
@custom-variant dark-system (@media (prefers-color-scheme: dark) { & });
```

## 4. Dark Mode Patterns

```html
<!-- Card -->
<div class="bg-white dark:bg-gray-800 rounded-xl p-6 shadow-sm
            dark:shadow-gray-900/50">
  <h3 class="text-gray-900 dark:text-white">Title</h3>
  <p class="text-gray-600 dark:text-gray-400">Content</p>
</div>

<!-- Borders -->
<div class="border border-gray-200 dark:border-gray-700">

<!-- Focus ring (visible in both modes) -->
<button class="focus:ring-blue-500 dark:focus:ring-blue-400">

<!-- Gradient hero (dark mode variant) -->
<section class="bg-gradient-to-r from-blue-50 to-purple-50 
               dark:from-gray-900 dark:to-gray-800">
```

## 5. Dark Mode CSS Variables

```css
@theme {
  --color-surface: oklch(1 0 0);         /* white */
  --color-surface-dark: oklch(0.13 0.028 261.692); /* gray-950 */
}
```

Usage preserves semantic naming:
```html
<div class="bg-surface dark:bg-surface-dark">
```

## 6. Image Handling

```html
<picture>
  <source srcset="dark-logo.png" media="(prefers-color-scheme: dark)">
  <img src="light-logo.png" alt="Logo">
</picture>
```

Or with JS toggle:
```html
<img class="dark:hidden" src="light-logo.png" alt="">
<img class="hidden dark:block" src="dark-logo.png" alt="">
```

## 7. Custom Dark Variant

```css
@custom-variant dark {
  &:where(.dark, .dark *) {
    @slot;
  }
}
```

For system-preference only:
```css
@custom-variant dark {
  @media (prefers-color-scheme: dark) {
    @slot;
  }
}
```

## 8. AI Notes

1. Always add `dark:*` variants for every color/background/border
2. Test both modes — ensure contrast in both
3. Use semantic CSS variables for easy dark mode theming
4. Focus rings need visible contrast in both modes
5. Images often need dark-mode variants
6. Shadows should be darker (lower opacity) in dark mode
7. `dark:*` works on any element at any level — no special container needed

## 9. Cross-Refs

- [Accessibility](skill_view(name='tailwind-css-documentation', file_path='references/accessibility.md'))
- [Customization](skill_view(name='tailwind-css-documentation', file_path='references/customization.md'))
- [Core Concepts](skill_view(name='tailwind-css-documentation', file_path='references/core-concepts.md'))
