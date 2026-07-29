# WordPress Plugin Development — Complete Reference

> Source: https://developer.wordpress.org/plugins/
> WordPress Version: 6.7+

## 1. Overview

Plugins add features/functions to WordPress without changing core or theme files. They are self-contained in `wp-content/plugins/`.

### Plugin vs functions.php

| Aspect | Plugin | Theme functions.php |
|---|---|---|
| Location | wp-content/plugins/ | wp-content/themes/{theme}/ |
| Persistence | Survives theme switch | Lost on theme switch |
| Purpose | Single feature | Theme-specific features |
| Header | Required | Not required |

## 2. Plugin File Header

```php
<?php
/**
 * Plugin Name:       My Plugin
 * Plugin URI:        https://example.com/plugins/my-plugin/
 * Description:       Description of the plugin.
 * Version:           1.0.0
 * Requires at least: 6.0
 * Requires PHP:      8.0
 * Author:            Your Name
 * Author URI:        https://example.com
 * License:           GPL v2 or later
 * License URI:       https://www.gnu.org/licenses/gpl-2.0.html
 * Text Domain:       my-plugin
 * Domain Path:       /languages
 * Update URI:        https://example.com/update-info/
 */
```

## 3. Plugin Structure

```
my-plugin/
├── my-plugin.php           # Main file (header + loader)
├── uninstall.php           # Cleanup on delete
├── includes/
│   ├── class-admin.php     # Admin-specific code
│   ├── class-frontend.php  # Frontend code
│   └── class-ajax.php      # AJAX handlers
├── admin/
│   └── views/              # Admin templates
├── public/
│   └── views/              # Public templates
├── languages/              # Translation files
├── assets/
│   ├── css/
│   ├── js/
│   └── images/
├── vendor/                 # Composer deps (if any)
├── composer.json
└── readme.txt              # WordPress.org readme
```

## 4. Activation / Deactivation / Uninstall

```php
// Activation
register_activation_hook( __FILE__, 'myplugin_activate' );
function myplugin_activate(): void {
    if ( ! current_user_can( 'activate_plugins' ) ) return;
    // Set defaults, create tables, schedule cron
    add_option( 'myplugin_version', '1.0.0' );
    if ( ! wp_next_scheduled( 'myplugin_daily' ) ) {
        wp_schedule_event( time(), 'daily', 'myplugin_daily' );
    }
    flush_rewrite_rules();
}

// Deactivation
register_deactivation_hook( __FILE__, 'myplugin_deactivate' );
function myplugin_deactivate(): void {
    wp_clear_scheduled_hook( 'myplugin_daily' );
    flush_rewrite_rules();
}

// Uninstall (uninstall.php in plugin root)
// Or inline:
register_uninstall_hook( __FILE__, 'myplugin_uninstall' );
function myplugin_uninstall(): void {
    if ( ! defined( 'WP_UNINSTALL_PLUGIN' ) ) exit;
    delete_option( 'myplugin_version' );
}
```

## 5. Custom Post Types

```php
function myplugin_register_post_types(): void {
    register_post_type( 'book', array(
        'labels'       => array(
            'name'          => __( 'Books', 'my-plugin' ),
            'singular_name' => __( 'Book', 'my-plugin' ),
            'add_new_item'  => __( 'Add New Book', 'my-plugin' ),
        ),
        'public'       => true,
        'has_archive'  => true,
        'rewrite'      => array( 'slug' => 'books' ),
        'supports'     => array( 'title', 'editor', 'thumbnail', 'excerpt' ),
        'menu_icon'    => 'dashicons-book',
        'show_in_rest' => true,   // REST API + Block Editor
        'capabilities' => array(
            'edit_post'          => 'edit_book',
            'read_post'          => 'read_book',
            'delete_post'        => 'delete_book',
            'edit_posts'         => 'edit_books',
            'publish_posts'      => 'publish_books',
        ),
    ));
}
add_action( 'init', 'myplugin_register_post_types' );
```

## 6. Taxonomies

```php
function myplugin_register_taxonomies(): void {
    register_taxonomy( 'genre', 'book', array(
        'labels'       => array(
            'name'          => __( 'Genres', 'my-plugin' ),
            'singular_name' => __( 'Genre', 'my-plugin' ),
        ),
        'hierarchical' => true,    // Like categories
        'rewrite'      => array( 'slug' => 'genre' ),
        'show_in_rest' => true,
    ));
}
add_action( 'init', 'myplugin_register_taxonomies' );
```

## 7. Hooks — Actions & Filters

```php
// Actions — run code at specific points
add_action( 'init', 'myplugin_init' );
add_action( 'wp_enqueue_scripts', 'myplugin_frontend_assets' );
add_action( 'admin_menu', 'myplugin_admin_page' );
add_action( 'save_post_book', 'myplugin_save_book_meta', 10, 3 );
add_action( 'wp_login', 'myplugin_on_user_login', 10, 2 );

// Filters — modify data
add_filter( 'the_content', 'myplugin_modify_content' );
add_filter( 'excerpt_length', fn() => 30 );
add_filter( 'upload_mimes', 'myplugin_custom_mimes' );
add_filter( 'nav_menu_css_class', 'myplugin_menu_classes', 10, 4 );
```

## 8. Creating Extensible Hooks

```php
// In your plugin: allow others to extend
function myplugin_render_settings(): void {
    // ... your settings HTML ...
    do_action( 'myplugin_after_settings_form' );
}

// Enable data modification
$config = apply_filters( 'myplugin_post_type_args', $default_args );
register_post_type( 'my_cpt', $config );
```

## 9. Security in Plugins

```php
// Prevent direct access
defined( 'ABSPATH' ) || exit;

// Nonce verification
wp_nonce_field( 'myplugin_action', 'myplugin_nonce' );
if ( ! wp_verify_nonce( $_POST['myplugin_nonce'], 'myplugin_action' ) ) {
    wp_die( 'Security check' );
}

// Capability check
if ( ! current_user_can( 'manage_options' ) ) {
    wp_die( 'Unauthorized' );
}

// Sanitize input
$safe   = sanitize_text_field( $_POST['field'] );
$email  = sanitize_email( $_POST['email'] );
$url    = sanitize_url( $_POST['url'] );
$id     = absint( $_POST['id'] );
$html   = wp_kses_post( $_POST['content'] );

// Escape output
echo esc_html( $plain_text );
echo esc_url( $url );
echo esc_attr( $attr_value );
```

## 10. WordPress Plugin Boilerplate Template

```php
<?php
declare( strict_types=1 );
defined( 'ABSPATH' ) || exit;

/**
 * Plugin Name: My Plugin
 * Description: Plugin description
 * Version: 1.0.0
 */

// Define constants
define( 'MYPLUGIN_VERSION', '1.0.0' );
define( 'MYPLUGIN_PATH', plugin_dir_path( __FILE__ ) );
define( 'MYPLUGIN_URL', plugin_dir_url( __FILE__ ) );

// Autoload
require_once MYPLUGIN_PATH . 'includes/class-main.php';

// Initialize
add_action( 'plugins_loaded', function() {
    new MyPlugin\Main();
});
```

## 11. Related Skills

- [Hooks API](skill_view(name='wordpress-documentation', file_path='references/hooks.md'))
- [Database API](skill_view(name='wordpress-documentation', file_path='references/database.md'))
- [Security API](skill_view(name='wordpress-documentation', file_path='references/security.md'))
- [APIs (Options, Settings, Rewrite, Shortcode, etc.)](skill_view(name='wordpress-documentation', file_path='references/apis.md'))
