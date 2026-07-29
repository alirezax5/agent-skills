# Tailwind CSS Recipes v4 — Production-Ready Components

> Source: Official tailwindcss.com patterns
> Tailwind CSS v4.x

## 1. Navbar (Responsive)

```html
<nav class="bg-white shadow-sm">
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex justify-between h-16 items-center">
      <a href="/" class="text-xl font-bold">Logo</a>
      <div class="hidden md:flex gap-6">
        <a href="#" class="text-gray-700 hover:text-gray-900">Home</a>
        <a href="#" class="text-gray-700 hover:text-gray-900">About</a>
      </div>
      <button class="md:hidden p-2 rounded-lg hover:bg-gray-100" aria-label="Menu">☰</button>
      <button class="hidden md:inline-flex rounded-lg bg-blue-600 px-4 py-2 text-white text-sm font-medium">Sign Up</button>
    </div>
  </div>
</nav>
```

## 2. Card Grid

```html
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
  <article class="rounded-xl border border-gray-200 bg-white shadow-sm transition-shadow hover:shadow-md overflow-hidden">
    <img class="w-full h-48 object-cover" src="" alt="">
    <div class="p-5">
      <span class="text-xs font-medium text-blue-600 uppercase tracking-wide">Category</span>
      <h3 class="mt-1 text-lg font-semibold line-clamp-2">Card Title</h3>
      <p class="mt-2 text-sm text-gray-600 line-clamp-3">Description</p>
      <a href="#" class="mt-4 inline-flex text-sm font-medium text-blue-600 hover:text-blue-500">Read more →</a>
    </div>
  </article>
</div>
```

## 3. Modal

```html
<div class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/50" role="dialog" aria-modal="true">
  <div class="w-full max-w-md rounded-xl bg-white p-6 shadow-xl">
    <div class="flex items-center justify-between">
      <h2 class="text-lg font-semibold">Title</h2>
      <button class="p-1 rounded-lg hover:bg-gray-100" aria-label="Close">✕</button>
    </div>
    <p class="mt-4 text-gray-600">Content</p>
    <div class="mt-6 flex justify-end gap-3">
      <button class="rounded-lg border border-gray-300 px-4 py-2 text-sm font-medium hover:bg-gray-50">Cancel</button>
      <button class="rounded-lg bg-blue-600 px-4 py-2 text-sm font-medium text-white hover:bg-blue-500">Confirm</button>
    </div>
  </div>
</div>
```

## 4. Dashboard Layout

```html
<div class="grid grid-rows-[auto_1fr] min-h-screen">
  <header class="sticky top-0 z-40 bg-white border-b px-6 py-3">
    <div class="flex items-center justify-between">
      <h1 class="text-xl font-semibold">Dashboard</h1>
    </div>
  </header>
  <div class="flex">
    <aside class="w-64 bg-gray-50 border-r p-4 shrink-0 hidden lg:block">
      <nav class="space-y-1">
        <a href="#" class="flex items-center gap-3 rounded-lg px-3 py-2 text-sm font-medium hover:bg-gray-200">Overview</a>
        <a href="#" class="flex items-center gap-3 rounded-lg px-3 py-2 text-sm font-medium bg-gray-200">Settings</a>
      </nav>
    </aside>
    <main class="flex-1 p-6 overflow-y-auto">
      <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
        <div class="rounded-lg bg-white p-4 shadow-sm border"><p class="text-sm text-gray-500">Users</p><p class="text-2xl font-bold">24K</p></div>
      </div>
    </main>
  </div>
</div>
```

## 5. Pricing

```html
<div class="grid grid-cols-1 md:grid-cols-3 gap-8 max-w-5xl mx-auto px-4">
  <div class="rounded-2xl border p-8 bg-white shadow-sm">
    <h3 class="text-lg font-semibold">Basic</h3>
    <p class="mt-4 text-4xl font-bold">$19<span class="text-base font-normal text-gray-500">/mo</span></p>
    <button class="mt-8 w-full rounded-lg border px-4 py-2 text-sm font-medium hover:bg-gray-50">Get Started</button>
  </div>
  <div class="rounded-2xl border-2 border-blue-500 p-8 bg-white shadow-lg relative">
    <span class="absolute -top-3 left-1/2 -translate-x-1/2 rounded-full bg-blue-600 px-3 py-1 text-xs text-white">Popular</span>
    <h3 class="text-lg font-semibold">Pro</h3>
    <p class="mt-4 text-4xl font-bold">$49<span class="text-base font-normal text-gray-500">/mo</span></p>
    <button class="mt-8 w-full rounded-lg bg-blue-600 px-4 py-2 text-sm font-medium text-white hover:bg-blue-500">Get Started</button>
  </div>
</div>
```

## 6. Form

```html
<form class="max-w-lg mx-auto space-y-6">
  <div>
    <label class="block text-sm font-medium text-gray-700 mb-1">Email</label>
    <input type="email" class="block w-full rounded-lg border border-gray-300 px-3 py-2 text-sm
                 placeholder:text-gray-400 focus:border-blue-500 focus:ring-1 focus:ring-blue-500 focus:outline-none">
  </div>
  <button type="submit" class="w-full rounded-lg bg-blue-600 px-4 py-2.5 text-sm font-semibold text-white
                hover:bg-blue-500 focus-visible:outline-2 focus-visible:outline-blue-600">Submit</button>
</form>
```

## 7. Table

```html
<div class="overflow-x-auto rounded-xl border border-gray-200">
  <table class="min-w-full divide-y divide-gray-200">
    <thead class="bg-gray-50">
      <tr><th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Name</th><th>Status</th></tr>
    </thead>
    <tbody class="bg-white divide-y divide-gray-200">
      <tr class="hover:bg-gray-50">
        <td class="px-6 py-4 text-sm font-medium">John</td>
        <td class="px-6 py-4"><span class="rounded-full bg-green-100 px-2 py-1 text-xs font-medium text-green-700">Active</span></td>
      </tr>
    </tbody>
  </table>
</div>
```

## 8. Toast

```html
<div class="fixed bottom-4 right-4 z-50" role="alert">
  <div class="flex items-center gap-3 rounded-lg bg-gray-900 px-4 py-3 text-white shadow-lg">
    <span class="text-sm">Saved!</span>
    <button class="text-gray-400 hover:text-white">✕</button>
  </div>
</div>
```

## 9. AI Notes

1. Mobile-first — check responsive classes
2. All interactive elements have `focus-visible:` styles
3. Extract to `@utility` or `@layer components` when reused >3x
4. Use `transition-colors` for cheap hover effects

## 10. Cross-Refs

- [Layout](skill_view(name='tailwind-css-documentation', file_path='references/layout.md'))
- [Flexbox & Grid](skill_view(name='tailwind-css-documentation', file_path='references/flexbox-grid.md'))
- [Responsive](skill_view(name='tailwind-css-documentation', file_path='references/responsive.md'))
- [Accessibility](skill_view(name='tailwind-css-documentation', file_path='references/accessibility.md'))
