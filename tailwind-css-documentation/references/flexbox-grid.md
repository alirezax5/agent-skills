# Tailwind CSS Flexbox & Grid v4 — Complete Reference

> Source: https://tailwindcss.com/docs/flex + grid + gap + align + justify
> Tailwind CSS v4.x

## 1. Overview

Flexbox (1D) handles rows OR columns. Grid (2D) handles rows AND columns simultaneously.

### Decision: Flex vs Grid
- Need a row or column? → **Flexbox**
- Need wrapping items? → **Flexbox** with `flex-wrap`
- Need alignment on both axes? → **Grid**
- Need explicit placement? → **Grid**
- Equal-height columns? → **Either** (Grid is cleaner)
- Reverse order? → **Flexbox** (`flex-row-reverse`)

## 2. Flexbox

### Container
```html
<div class="flex">...</div>          <!-- row (default) -->
<div class="flex-col">...</div>      <!-- column -->
<div class="flex-wrap">...</div>     <!-- wrap -->
```

### Direction: `flex-row`, `flex-row-reverse`, `flex-col`, `flex-col-reverse`
### Wrap: `flex-wrap`, `flex-wrap-reverse`, `flex-nowrap`

### Children

| Class | CSS | Effect |
|---|---|---|
| `flex-1` | `flex: 1 1 0%` | Equal space (grow + shrink) |
| `flex-auto` | `flex: 1 1 auto` | Grow based on content |
| `flex-initial` | `flex: 0 1 auto` | Default |
| `flex-none` | `flex: none` | Fixed size |
| `grow` / `grow-0` | `flex-grow: 1/0` | Grow toggle |
| `shrink` / `shrink-0` | `flex-shrink: 1/0` | Shrink toggle |
| `basis-64` | `flex-basis: 16rem` | Initial size |

### Gap: `gap-4`, `gap-x-4`, `gap-y-4`
### Order: `order-first`, `order-last`, `order-none`, `order-1` to `order-12`

### Alignment (Main Axis)

| Class | Behavior |
|---|---|
| `justify-start` | Start |
| `justify-end` | End |
| `justify-center` | Center |
| `justify-between` | Space between |
| `justify-around` | Space around |
| `justify-evenly` | Evenly |

### Alignment (Cross Axis)

| Class | Behavior |
|---|---|
| `items-start` | Cross-start |
| `items-end` | Cross-end |
| `items-center` | Cross-center |
| `items-baseline` | Baseline |
| `items-stretch` | Stretch (default) |

### Align Self (per child)

`self-auto`, `self-start`, `self-end`, `self-center`, `self-stretch`, `self-baseline`

## 3. Grid

```html
<div class="grid grid-cols-3 gap-4">...</div>
```

### Template Columns/Rows

`grid-cols-1` through `grid-cols-12`; `grid-cols-none`, `grid-cols-subgrid`
`grid-rows-1` through `grid-rows-6`; `grid-rows-none`, `grid-rows-subgrid`

### Auto Flow

`grid-flow-row` (default), `grid-flow-col`, `grid-flow-row-dense`, `grid-flow-col-dense`

### Span & Placement

`col-span-2`, `col-span-full`, `col-start-2`, `col-end-4`
`row-span-2`, `row-span-full`, `row-start-2`, `row-end-4`

### Auto Sizing

`auto-cols-auto/min/max/fr`, `auto-rows-auto/min/max/fr`

## 4. Alignment Matrix

| Property | start | end | center | stretch |
|---|---|---|---|---|
| **justify-content** | `justify-start` | `justify-end` | `justify-center` | `justify-stretch` |
| **align-items** | `items-start` | `items-end` | `items-center` | `items-stretch` |
| **align-self** | `self-start` | `self-end` | `self-center` | `self-stretch` |
| **justify-items** | `justify-items-start` | `justify-items-end` | `justify-items-center` | `justify-items-stretch` |
| **justify-self** | `justify-self-auto` | `justify-self-end` | `justify-self-center` | `justify-self-stretch` |
| **place-content** | `place-content-start` | `place-content-end` | `place-content-center` | `place-content-stretch` |
| **place-items** | `place-items-start` | `place-items-end` | `place-items-center` | `place-items-stretch` |
| **place-self** | `place-self-auto` | `place-self-end` | `place-self-center` | `place-self-stretch` |

## 5. Common Patterns

### Holy Grail Layout
```html
<div class="grid grid-rows-[auto_1fr_auto] min-h-screen">
  <header>Header</header>
  <div class="grid grid-cols-[250px_1fr_250px]">
    <nav>Left</nav>
    <main>Content</main>
    <aside>Right</aside>
  </div>
  <footer>Footer</footer>
</div>
```

### Auto-fill Card Grid
```html
<div class="grid grid-cols-[repeat(auto-fill,minmax(250px,1fr))] gap-6">
```

### Sidebar + Content
```html
<div class="flex gap-8">
  <aside class="w-64 shrink-0">Sidebar</aside>
  <main class="flex-1 min-w-0">Content</main>
</div>
```

## 6. AI Reasoning Notes

1. `gap-*` instead of margin on children for spacing
2. `flex-1` is most common — takes remaining space
3. `min-w-0` fixes flex children not shrinking below content
4. `shrink-0` prevents fixed elements from collapsing
5. `grid-cols-[repeat(auto-fill,minmax(250px,1fr))]` for responsive grids
6. `flex-col md:flex-row` for mobile-first responsive layouts
7. Sticky footer: `grid-rows-[auto_1fr_auto]` + `min-h-screen`

## 7. Cross-References

- [Layout](skill_view(name='tailwind-css-documentation', file_path='references/layout.md'))
- [Spacing](skill_view(name='tailwind-css-documentation', file_path='references/spacing.md'))
- [Sizing](skill_view(name='tailwind-css-documentation', file_path='references/sizing.md'))
- [Responsive](skill_view(name='tailwind-css-documentation', file_path='references/responsive.md'))
