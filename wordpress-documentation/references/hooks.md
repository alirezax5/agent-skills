# WordPress Hooks API — Complete Reference

> Source: https://developer.wordpress.org/apis/hooks/ + Plugin Handbook Hooks chapter
> WordPress Version: 6.7+

## 1. Overview

**Hooks** are WordPress's plugin/theme extension system. They allow code to interact/modify other code at predefined spots without editing core files.

### Two Types

| Type | Purpose | Function |
|---|---|---|
| **Action** | Add/remove functionality at specific points | `do_action()` to create, `add_action()` to hook |
| **Filter** | Modify content before output | `apply_filters()` to create, `add_filter()` to hook |

### Why They Exist
- Allow plugins/themes to extend WP without modifying core
- Enable modular, decoupled architecture
- Provide backward-compatible extension points

### When to Use
- **Actions**: When you need to run code at a specific moment (save post, send email, redirect)
- **Filters**: When you need to modify data (content, title, excerpt, query)

### Common Mistakes
- ❌ Echoing inside a filter callback (filters must return values)
- ❌ Using `exit`/`die` inside hooks without consideration
- ❌ Not prefixing custom hook names (causing collisions)
- ❌ Confusing actions with filters

---

## 2. Architecture

```mermaid
graph LR
    A[WP Core/Plugin Code] --> B{do_action() or apply_filters()?}
    B -->|Action| C[Execute all hooked callbacks]
    B -->|Filter| D[Pass value through all callbacks]
    C --> E[Return control to caller]
    D --> F[Return modified value]
```

### Execution Order
1. WordPress loads plugins → registers hooks via `add_action()`/`add_filter()`
2. During execution, `do_action('hook_name')` is reached
3. All callbacks registered on that hook run in priority order (default 10)
4. Callbacks with same priority run in registration order

---

## 3. API Reference

### Core Functions

| Function | Purpose | Parameters |
|---|---|---|
| `add_action( $hook, $callback, $priority, $accepted_args )` | Register action callback | hook: string, callback: callable, priority: int (10), accepted_args: int (1) |
| `add_filter( $hook, $callback, $priority, $accepted_args )` | Register filter callback | Same as above |
| `do_action( $hook, ...$args )` | Execute action callbacks | hook: string, args: mixed (variadic) |
| `apply_filters( $hook, $value, ...$args )` | Execute filter callbacks | hook: string, value: mixed (modified), args: mixed |
| `remove_action( $hook, $callback, $priority )` | Remove action callback | Must match add_action() args exactly |
| `remove_filter( $hook, $callback, $priority )` | Remove filter callback | Same |
| `has_action( $hook, $callback )` | Check if action is registered | callback: optional |
| `has_filter( $hook, $callback )` | Check if filter is registered | callback: optional |
| `did_action( $hook )` | Check if action has been run | Returns int (number of times) |
| `current_action()` | Get current action name | Useful inside callbacks |
| `current_filter()` | Get current filter name | Useful inside callbacks |
| `doing_action( $hook )` | Check if action is currently being processed | |
| `doing_filter( $hook )` | Check if filter is currently being processed | |

### Important WordPress Hooks (Execution Order)

```
1. muplugins_loaded        → Must-Use plugins loaded
2. registered_taxonomy     → Taxonomy registered
3. registered_post_type    → Post type registered
4. plugins_loaded          → All active plugins loaded ← KEY
5. setup_theme             → Theme loaded
6. after_setup_theme       → Theme setup complete ← KEY for theme
7. init                    → WP initialized ← KEY for CPTs/taxonomies
8. wp_loaded               → WP fully loaded
9. wp_head                 → <head> output
10. wp_enqueue_scripts     → Enqueue styles/scripts ← KEY
11. template_redirect      → Before template determined
12. wp_footer              → Footer output
13. shutdown               → Script execution ends
```

---

## 4. Examples

### Simple Action
```php
// Send email when a post is published
add_action( 'publish_post', function( $post_id ) {
    $admin_email = get_option( 'admin_email' );
    wp_mail( $admin_email, 'Post Published', "Post #{$post_id} was published." );
});
```

### Simple Filter
```php
// Change excerpt length
add_filter( 'excerpt_length', function( $length ) {
    return 40; // Default is 55
});
```

### Creating Custom Hooks (for extensibility)
```php
// In your plugin: create action
function myplugin_render_settings_page() {
    // ... form HTML ...
    do_action( 'myplugin_after_settings_form' );
}

// Another plugin hooks into it
add_action( 'myplugin_after_settings_form', function() {
    echo '<p>Additional settings from add-on</p>';
});
```

### Filter with Arguments
```php
// Apply filter to allow modification
$title = apply_filters( 'myplugin_item_title', $default_title, $item_id, $context );

// Another plugin modifies it
add_filter( 'myplugin_item_title', function( $title, $item_id, $context ) {
    if ( 'featured' === $context && $item_id > 100 ) {
        $title = '★ ' . $title;
    }
    return $title;
}, 10, 3 ); // priority 10, accepts 3 args
```

### Remove a Hook
```php
// Remove a script enqueued by another plugin
add_action( 'wp_enqueue_scripts', function() {
    remove_action( 'wp_enqueue_scripts', 'other_plugin_enqueue_script', 10 );
}, 11 ); // Must run after the original add_action
```

---

## 5. Common Patterns

### Conditional Hook Removal
```php
add_action( 'init', function() {
    if ( is_admin() ) {
        remove_action( 'admin_notices', 'some_plugin_notice' );
    }
});
```

### Namespace Your Hooks
```php
// Always prefix hook names to avoid collisions
// ✅ Good: 'myplugin_save_settings'
// ❌ Bad: 'save_settings'
```

### Using Anonymous Functions (Closures)
```php
add_filter( 'the_content', function( $content ) {
    if ( ! is_single() ) return $content;
    return $content . '<p>Read more...</p>';
});
```

---

## 6. Anti-patterns

### ❌ Echoing inside Filters
```php
// WRONG
add_filter( 'the_title', function( $title ) {
    echo '<strong>' . $title . '</strong>'; // Echo = output at wrong position!
    return $title;
});

// CORRECT
add_filter( 'the_title', function( $title ) {
    return '<strong>' . $title . '</strong>';
});
```

### ❌ Forgetting to Return in Filters
```php
// WRONG — returns null, destroying the value
add_filter( 'excerpt_length', function( $length ) {
    // forgot return
});

// CORRECT
add_filter( 'excerpt_length', function( $length ) {
    return 30;
});
```

### ❌ Direct DB Queries Instead of Hooks
```php
// WRONG — don't modify core files
// Instead, use hooks:
add_filter( 'posts_where', function( $where, $query ) {
    if ( ! is_admin() && $query->is_main_query() ) {
        $where .= ' AND post_date > "2024-01-01"';
    }
    return $where;
}, 10, 2 );
```

---

## 7. Performance

- Hook callbacks run on every page load — keep them lightweight
- Use `remove_action()` sparingly (adds overhead)
- For expensive operations inside hooks, use `wp_schedule_single_event()` (WP Cron)
- Use conditional checks (`is_admin()`, `is_single()`) to skip unnecessary callback execution

## 8. Security

- Always escape output inside callbacks (`esc_html`, `esc_url`, etc.)
- Sanitize input before using it in hooks
- Check capabilities before executing privileged actions:
```php
add_action( 'save_post', function( $post_id ) {
    if ( ! current_user_can( 'edit_post', $post_id ) ) return;
    // ... safe to proceed
});
```

## 9. Testing Hooks
- Use `did_action()` to verify an action was triggered
- Use `has_filter()` to verify filters are registered
- Mock `apply_filters()` return values in unit tests
- WordPress PHPUnit test suite provides `tests_add_filter()`

## 10. Troubleshooting

| Problem | Likely Cause | Solution |
|---|---|---|
| Hook not running | Wrong hook name | Check with `has_action($hook)` |
| Wrong execution order | Priority collision | Set explicit priority (5, 9, 11 instead of 10) |
| Filter not changing value | Return forgotten or too late | Ensure filter `return`s the modified value |
| Infinite loop | Hook calling itself | Add recursion guard or remove temporarily |

## 11. AI Reasoning Notes

When solving problems with hooks:
1. **Identify the right hook** first — use the execution order table above
2. For **adding** content: use Actions
3. For **modifying** data: use Filters
4. Always **prefix** custom hook names (plugin/theme name)
5. Filters **must return** the value; Actions just run code
6. Check `did_action()` for conditional execution
7. Use priority parameter (default 10) to control callback order
