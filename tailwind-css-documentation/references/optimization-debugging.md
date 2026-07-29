# Tailwind CSS Optimization & Debugging v4 — Complete Reference

> Source: https://tailwindcss.com/docs/optimizing-for-production + detecting-classes
> Tailwind CSS v4.x

## 1. Overview

Tailwind v4 auto tree-shakes unused classes by scanning source files. Final CSS contains only used utilities.

## 2. Content Detection

Auto-scans: `./src/`, `./public/`, Vite project root.

Manual: `@source "../node_modules/my-library";`

## 3. Production Build

```bash
# CLI
tailwindcss -i app.css -o dist/styles.css

# Vite
npx vite build

# PostCSS
NODE_ENV=production npx postcss app.css -o dist/styles.css
```

## 4. Optimization Tips

### Use Theme Variables (not arbitrary values)
```css
@theme { --spacing-5: 1.25rem; }
<div class="mt-5 pt-5 gap-5">  <!-- single source, smaller bundle -->
```

### Use @utility for Repeated Patterns
```css
@utility card-shadow {
  box-shadow: var(--shadow-md);
  border-radius: var(--radius-xl);
}
```

### Avoid Dynamic Classes
```html
<!-- ❌ Can't detect -->
<div class={`text-${size}`}>

<!-- ✅ Can detect -->
<div class="text-sm md:text-base lg:text-lg">
```

## 5. Common Issues

| Issue | Cause | Fix |
|---|---|---|
| Class not working | Scanner missed it | Add `@source` |
| Unknown class | Typo or not in theme | Check spelling, define via `@theme` |
| Dark mode not applying | `.dark` missing on parent | Add `class="dark"` to `<html>` |
| Prod missing styles | Dynamic class names | Use static variants |
| v3→v4 migration | Config format changed | Move to `@theme` in CSS |

## 6. Bundle Size Reference

- Minimal site: 2-4KB gzip
- Medium site: 5-15KB
- Large site: 15-40KB
- Design system: 30-60KB

## 7. AI Notes

1. Never use dynamic class names — scanner can't detect
2. Use `@theme` for repeated values
3. Check production bundle in DevTools
4. `@source` for external paths, not your own src
5. v4 tree-shakes better than v3
6. OKLCH compresses better than hex in gzip

## 8. Cross-Refs

- [Installation](skill_view(name='tailwind-css-documentation', file_path='references/installation.md'))
- [Customization](skill_view(name='tailwind-css-documentation', file_path='references/customization-directives.md'))
