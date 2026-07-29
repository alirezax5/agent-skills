# Tailwind CSS Installation v4 — Complete Reference

> Source: https://tailwindcss.com/docs/installation/using-vite
> Tailwind CSS v4.x

## 1. Overview

Tailwind v4 is a CSS-first configuration framework. No JavaScript config file needed — everything is done via CSS directives like `@theme`, `@utility`, and `@variant`.

### Installation Methods

| Method | Best For |
|---|---|
| **Vite** | Modern JS frameworks (React, Vue, Svelte, Solid, Laravel) |
| **PostCSS** | Custom PostCSS pipelines (Next.js, Nuxt) |
| **CLI** | Static sites, plain HTML |
| **CDN** | Prototyping, simple pages |
| **Play CDN** | Playgrounds, demos |

## 2. Vite Installation (Recommended)

```bash
npm install tailwindcss @tailwindcss/vite
```

**vite.config.js:**
```js
import { defineConfig } from 'vite'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [tailwindcss()],
})
```

**app.css:**
```css
@import "tailwindcss";
```

**index.html / src (any file Tailwind scans):**
```html
<h1 class="text-3xl font-bold underline">Hello World!</h1>
```

Run: `npx vite` or `npm run dev`

## 3. PostCSS Installation

```bash
npm install tailwindcss @tailwindcss/postcss
```

**postcss.config.mjs:**
```js
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
}
```

**app.css:**
```css
@import "tailwindcss";
```

## 4. CLI Installation (No Build Step)

```bash
npm install tailwindcss @tailwindcss/cli
npx tailwindcss -i app.css -o dist/styles.css
```

**package.json:**
```json
{
  "scripts": {
    "dev": "tailwindcss -i app.css -o dist/styles.css --watch",
    "build": "tailwindcss -i app.css -o dist/styles.css"
  }
}
```

Watch mode: `npx tailwindcss -i app.css -o dist/styles.css --watch`

## 5. CDN (Prototyping Only)

```html
<link rel="stylesheet" href="https://unpkg.com/tailwindcss@4">
```

Not suitable for production (no tree-shaking, no custom theme).

## 6. Framework-Specific Guides

| Framework | Setup |
|---|---|
| **Laravel** | Vite — `npm install tailwindcss @tailwindcss/vite` |
| **Next.js** | PostCSS (built-in support) |
| **Nuxt** | PostCSS or `@nuxtjs/tailwindcss` module |
| **Remix** | PostCSS (official template) |
| **SvelteKit** | Vite (built-in) |
| **SolidJS** | Vite (built-in) |
| **Astro** | PostCSS or Vite integration |
| **WordPress** | CLI (enqueue dist/styles.css) or CDN |

## 7. Editor Setup

Install **Tailwind CSS IntelliSense** VS Code extension for autocomplete, hover preview, and linting.

## 8. PostCSS/CSS Plugin Usage

```css
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.6 0.2 260);
}

@utility card {
  border-radius: var(--radius-xl);
  padding: var(--spacing-6);
  box-shadow: var(--shadow-md);
}

@variant pointer-coarse {
  @media (pointer: coarse) {
    @slot;
  }
}
```

## 9. Compatability

- **Browsers**: Chrome 110+, Firefox 120+, Safari 16.5+, Edge 110+
- **Requires**: CSS cascade layers (`@layer`) support
- **IE11**: NOT supported

## 10. Upgrade from v3 to v4

| v3 pattern | v4 equivalent |
|---|---|
| `tailwind.config.js` | `@theme` directive in CSS |
| `theme.extend` in config | `@theme { --key: value }` |
| `plugin(function({ addUtilities }) {...})` | `@utility` directive |
| `addVariant()` | `@custom-variant` |
| `@tailwind base/components/utilities` | `@import "tailwindcss"` + `@layer` |
| `content: ['./src/**/*.html']` | `@source` or auto-detection |

## 11. AI Reasoning Notes

1. Use **Vite** for new projects — it's the recommended path
2. CDN is **prototyping-only** — never use in production
3. `@import "tailwindcss"` replaces `@tailwind base/components/utilities` from v3
4. No `tailwind.config.js` needed in v4 — use `@theme` in CSS
5. CLI needs `--watch` for dev and `--minify` for production

## 12. Cross-References

- [Core Concepts](skill_view(name='tailwind-css-documentation', file_path='references/core-concepts.md'))
- [Directives](skill_view(name='tailwind-css-documentation', file_path='references/directives.md'))
- [Customization](skill_view(name='tailwind-css-documentation', file_path='references/customization.md'))
