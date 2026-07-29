# WordPress Block Editor — Complete Reference

> Source: https://developer.wordpress.org/block-editor/
> WordPress Version: 6.7+

## 1. Overview

The Block Editor (Gutenberg) is the modern WordPress content editing paradigm. It uses a modular **block** system to compose content.

### Key Concepts
- **Block**: Atomic unit of content (paragraph, image, heading, etc.)
- **Block Type**: Blueprint defining a block's behavior (`core/paragraph`)
- **Block Template**: Predefined arrangement of blocks
- **Block Pattern**: Reusable block layouts (stored in DB or theme)
- **Block Theme**: Theme using HTML block templates instead of PHP
- **Global Styles**: Design settings via `theme.json`
- **Full Site Editing (FSE)**: Edit all site parts via block editor

### When to Create Custom Blocks
- Complex UI that shortcodes can't handle
- Structured data with specific editing experience
- Reusable content components across site
- Interactive editor-side functionality

### Block Editor Handbook Sections
| Section | URL | Content |
|---|---|---|
| Getting Started | /block-editor/getting-started/ | Setup, fundamentals, first block tutorial |
| How-to Guides | /block-editor/how-to-guides/ | Data basics, meta boxes, theme support |
| Reference Guides | /block-editor/reference-guides/ | Block API, components, packages |
| Explanations | /block-editor/explanations/ | Architecture, data flow, key concepts |
| Contributor Guide | /block-editor/contributors/ | Contributing code, design, docs |

## 2. Creating a Block

### Quick Start
```bash
npx @wordpress/create-block my-block
cd my-block
npm start
```

### Block Structure
```
my-block/
├── block.json          # Block metadata
├── src/
│   ├── index.js        # Block registration
│   ├── edit.js         # Editor UI
│   ├── save.js         # Frontend output
│   ├── style.scss      # Frontend + editor styles
│   └── editor.scss     # Editor-only styles
└── build/              # Compiled output
```

### block.json
```json
{
    "$schema": "https://schemas.wp.org/trunk/block.json",
    "apiVersion": 3,
    "name": "my-plugin/featured-content",
    "title": "Featured Content",
    "category": "widgets",
    "icon": "star-filled",
    "description": "Display featured content with custom styling",
    "keywords": ["featured", "highlight"],
    "version": "1.0.0",
    "textdomain": "my-plugin",
    "attributes": {
        "title": { "type": "string", "default": "" },
        "showThumbnail": { "type": "boolean", "default": true }
    },
    "supports": {
        "html": false,
        "align": true,
        "color": { "background": true, "text": true }
    },
    "editorScript": "file:./build/index.js",
    "style": "file:./build/style-index.css",
    "editorStyle": "file:./build/index.css"
}
```

### PHP Registration
```php
function myplugin_register_block() {
    register_block_type( __DIR__ . '/build/block.json' );
}
add_action( 'init', 'myplugin_register_block' );
```

## 3. Block APIs

### Block Supports
```json
{
    "supports": {
        "align": true,
        "anchor": true,
        "color": { "background": true, "text": true, "gradients": true },
        "customClassName": true,
        "dimensions": { "minHeight": true },
        "html": false,
        "multiple": true,
        "reusable": true,
        "spacing": { "margin": true, "padding": true },
        "typography": { "fontSize": true, "lineHeight": true }
    }
}
```

### Core Blocks Reference
`core/paragraph`, `core/heading`, `core/image`, `core/gallery`, `core/list`, `core/quote`, `core/code`, `core/table`, `core/separator`, `core/spacer`, `core/columns`, `core/group`, `core/buttons`, `core/media-text`, `core/cover`, `core/embed`, `core/shortcode`, `core/archives`, `core/categories`, `core/latest-posts`, `core/search`, `core/social-links`, `core/navigation`, `core/query`, `core/post-template`, `core/post-title`, `core/post-content`, `core/post-featured-image`, `core/site-logo`, `core/site-title`, `core/site-tagline`

### Data Layer (wp.data)
```javascript
const { select, dispatch, subscribe } = wp.data;

// Current post
const post = select( 'core/editor' ).getCurrentPost();

// Update block
dispatch( 'core/block-editor' ).updateBlockAttributes( clientId, { align: 'wide' } );

// Subscribe to changes
subscribe( () => {
    if ( select( 'core/editor' ).isSavingPost() ) { /* ... */ }
});
```

## 4. Performance
- Use `wp_enqueue_block_style()` for per-block CSS loading in themes
- Avoid heavy dependencies in block code
- Use code splitting for complex blocks
- Use `InnerBlocks` sparingly in deeply nested structures

## 5. Packages Reference
Key npm packages from `@wordpress/`:
- `@wordpress/block-editor` — Block editing components
- `@wordpress/blocks` — Block API functions
- `@wordpress/components` — UI components
- `@wordpress/data` — Data store
- `@wordpress/element` — React abstraction
- `@wordpress/i18n` — Internationalization
- `@wordpress/hooks` — Filter/action system for JS

## 6. AI Reasoning Notes

Building blocks:
1. Start with `@wordpress/create-block` scaffold
2. Define attributes in `block.json`
3. Implement `edit()` for editor UI and `save()` for output
4. Use `useSelect`/`useDispatch` for data access
5. Follow WordPress JS coding standards
6. Test in both editor and frontend contexts
