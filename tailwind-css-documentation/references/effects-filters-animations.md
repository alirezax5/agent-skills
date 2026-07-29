# Tailwind CSS Effects, Filters, Transforms & Animations v4 — Complete Reference

> Source: https://tailwindcss.com/docs/box-shadow + filter + backdrop-filter + scale + rotate + translate + animation + transition
> Tailwind CSS v4.x

## 1. Box Shadow

```html
<div class="shadow-sm">...</div>
<div class="shadow-md">...</div>
<div class="shadow-lg">...</div>
<div class="shadow-xl">...</div>
<div class="shadow-2xl">...</div>
<div class="shadow-inner">...</div>
<div class="shadow-none">...</div>
<div class="shadow-[0_4px_12px_rgba(0,0,0,0.1)]">  <!-- custom -->
```

Inset shadows: `inset-shadow-xs`, `inset-shadow-sm`, `inset-shadow-md` (new in v4)

## 2. Text Shadow (new in v4)

`text-shadow-2xs`, `text-shadow-xs`, `text-shadow-sm`, `text-shadow-md`, `text-shadow-lg`

## 3. Opacity

`opacity-0` to `opacity-100` (step 5: 0, 5, 10, 15, ..., 100)

## 4. Mix Blend Mode

`mix-blend-normal`, `multiply`, `screen`, `overlay`, `darken`, `lighten`, `color-dodge`, `color-burn`, `hard-light`, `soft-light`, `difference`, `exclusion`, `hue`, `saturation`, `color`, `luminosity`, `plus-darker`, `plus-lighter`

## 5. Filters (Individual)

| Class | CSS |
|---|---|
| `blur-sm` | `filter: blur(8px)` |
| `blur-md` / `blur-lg` / `blur-xl` / `blur-2xl` / `blur-3xl` | Progressive |
| `blur-none` | `blur(0)` |
| `brightness-50` through `brightness-200` | `brightness(N%)` |
| `contrast-50` through `contrast-200` | `contrast(N%)` |
| `drop-shadow-sm` through `drop-shadow-2xl` | `drop-shadow(...)` |
| `grayscale` | `grayscale(100%)` |
| `grayscale-0` | `grayscale(0)` |
| `hue-rotate-0` / `hue-rotate-15` / `hue-rotate-30` / `hue-rotate-60` / `hue-rotate-90` / `hue-rotate-180` | hue-rotate |
| `invert` / `invert-0` | `invert(100%)`/`invert(0)` |
| `saturate-0` through `saturate-200` | `saturate(N%)` |
| `sepia` / `sepia-0` | `sepia(100%)`/`sepia(0)` |

## 6. Filter Preset

```html
<div class="filter blur-sm brightness-110">...</div>
<div class="filter-none">...</div>  <!-- reset -->
```

## 7. Backdrop Filters

Same as filters but with `backdrop-` prefix: `backdrop-blur-sm`, `backdrop-brightness-110`, etc.

```html
<!-- Glass morphism -->
<div class="bg-white/70 backdrop-blur-md rounded-xl">
```

## 8. Transforms

### Scale

| Class | CSS |
|---|---|
| `scale-50` through `scale-150` (increments of 25) | `scale(N%)` |
| `scale-x-50` / `scale-y-50` | Per-axis |
| `scale-3d` | 3D scale |

### Rotate

`rotate-0`, `rotate-45`, `rotate-90`, `rotate-180`, `rotate-[-45deg]` (arbitrary)

### Translate

| Class | CSS |
|---|---|
| `translate-x-0` through `translate-x-96` | `translateX(n × 0.25rem)` |
| `-translate-x-4` | Negative |
| `translate-x-1/2` / `translate-x-full` | Percentage |
| `-translate-x-1/2` | Centering |
| `translate-y-*` | Same pattern |

### Skew

`skew-x-1` through `skew-x-12`, `skew-y-*`, `-skew-x-3` (negative)

### Transform Origin

`origin-center`, `origin-top`, `origin-top-right`, `origin-right`, `origin-bottom-right`, `origin-bottom`, `origin-bottom-left`, `origin-left`, `origin-top-left`

### Perspective

`perspective-dramatic`(100px), `perspective-near`(300px), `perspective-normal`(500px), `perspective-midrange`(800px), `perspective-distant`(1200px)

### Transform Style

`transform-3d`, `transform-flat`, `transform-content`

## 9. Transitions

```html
<button class="transition-colors duration-200 ease-in-out hover:bg-blue-600">
```

| Property | Classes |
|---|---|
| **Property** | `transition-none`, `transition-all`, `transition-colors`, `transition-opacity`, `transition-shadow`, `transition-transform`, `transition-[property]` |
| **Duration** | `duration-75`, `duration-100`, `duration-150`, `duration-200`, `duration-300`, `duration-500`, `duration-700`, `duration-1000` |
| **Timing** | `ease-linear`, `ease-in`, `ease-out`, `ease-in-out` (plus custom via `--ease-*`) |
| **Delay** | `delay-75`, `delay-100`, `delay-150`, `delay-200`, `delay-300`, `delay-500` |
| **Behavior** | `transition-behavior-discrete`, `transition-behavior-normal` |

## 10. Animations

| Class | Animation |
|---|---|
| `animate-none` | No animation |
| `animate-spin` | 360° spin (1s linear infinite) |
| `animate-ping` | Scale + fade (1s cubic-bezier infinite) |
| `animate-pulse` | Opacity fade (2s ease-in-out infinite) |
| `animate-bounce` | Bounce (1s infinite) |

Custom animation via `@theme`:
```css
@theme {
  --animate-fade-in: fade-in 0.3s ease-out;
  @keyframes fade-in {
    from { opacity: 0; transform: scale(0.95); }
    to { opacity: 1; transform: scale(1); }
  }
}
```
Usage: `class="animate-fade-in"`

## 11. Using `transition-*` Standard Properties

- `transition-colors`: color, background-color, border-color, text-decoration-color, outline-color, fill, stroke
- `transition-all`: all animatable properties
- `transition-[property]`: arbitrary

## 12. Common Patterns

```html
<!-- Button hover -->
<button class="transition-all duration-200 hover:scale-105 hover:shadow-lg active:scale-95">

<!-- Card hover lift -->
<div class="transition-shadow duration-300 hover:shadow-xl">

<!-- Glass morphism -->
<div class="bg-white/70 backdrop-blur-md rounded-xl border border-white/20 shadow-lg">

<!-- Hover image zoom -->
<div class="overflow-hidden rounded-lg">
  <img class="transition-transform duration-500 hover:scale-110" src="..." alt="">
</div>

<!-- Fade in on hover (show overlay) -->
<div class="group relative">
  <div class="absolute inset-0 bg-black/50 opacity-0 group-hover:opacity-100 transition-opacity">
    Overlay
  </div>
  <img src="..." alt="">
</div>

<!-- Loading spinner -->
<div class="animate-spin rounded-full size-8 border-4 border-blue-500 border-t-transparent">
</div>
```

## 13. AI Notes

1. `transition-colors` for button/text hover effects (cheaper than `transition-all`)
2. `transition-transform` + `hover:scale-105` for interactive cards
3. `group-hover:` for parent-child hover interactions
4. `backdrop-blur-*` for glass morphism (requires `bg-white/70` for effect)
5. `active:scale-95` for button press feedback
6. Custom animations are defined via `--animate-*` + `@keyframes` inside `@theme`
7. `drop-shadow-*` respects image transparency (unlike `shadow-*`)
8. Origin: `hover:scale-110 origin-center` to scale from center

## 14. Cross-Refs

- [Backgrounds](skill_view(name='tailwind-css-documentation', file_path='references/backgrounds.md'))
- [Borders](skill_view(name='tailwind-css-documentation', file_path='references/borders.md'))
- [Accessibility](skill_view(name='tailwind-css-documentation', file_path='references/accessibility.md')) (motion-safe/reduce)
