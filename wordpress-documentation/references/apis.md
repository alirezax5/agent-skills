# WordPress APIs — Complete Reference

> Source: https://developer.wordpress.org/apis/
> WordPress Version: 6.7+

## 1. Options API

Simple key-value persistent storage in `wp_options` table.

```php
// Create/Update
update_option( 'myplugin_key', $value, false );  // $autoload = false
add_option( 'myplugin_key', $default_value );    // Only adds if doesn't exist

// Read
$value = get_option( 'myplugin_key', 'default' );

// Delete
delete_option( 'myplugin_key' );

// Network-wide (multisite)
get_site_option( 'myplugin_key' );
update_site_option( 'myplugin_key', $value );
delete_site_option( 'myplugin_key' );

// Autoload behavior
// true = loaded on every page (use sparingly)
// false = lazy loaded only when requested
```

## 2. Settings API

Standard way to register settings, sections, and fields for the admin UI.

```php
// Register setting and render callback
add_action( 'admin_init', function() {
    // Register a setting
    register_setting( 'myplugin_settings', 'myplugin_option' );

    // Add section
    add_settings_section(
        'myplugin_main_section',
        'Main Settings',
        function() { echo '<p>Main settings description</p>'; },
        'myplugin'
    );

    // Add field
    add_settings_field(
        'myplugin_text_field',
        'Text Input',
        function() {
            $value = get_option( 'myplugin_option', '' );
            echo '<input type="text" name="myplugin_option" value="' . esc_attr( $value ) . '" />';
        },
        'myplugin',
        'myplugin_main_section'
    );
});

// Render settings page
function myplugin_settings_page() {
    ?>
    <div class="wrap">
        <h1><?php esc_html_e( 'My Plugin Settings', 'myplugin' ); ?></h1>
        <form action="options.php" method="post">
            <?php
            settings_fields( 'myplugin_settings' );
            do_settings_sections( 'myplugin' );
            submit_button();
            ?>
        </form>
    </div>
    <?php
}

// Register admin menu page
add_action( 'admin_menu', function() {
    add_options_page(
        'My Plugin',
        'My Plugin',
        'manage_options',
        'myplugin',
        'myplugin_settings_page'
    );
});
```

## 3. Rewrite API

Add custom URL structures.

```php
add_action( 'init', function() {
    // Add rewrite tag
    add_rewrite_tag( '%product_id%', '([0-9]+)' );

    // Add rewrite rule
    add_rewrite_rule(
        'products/([0-9]+)/?$',
        'index.php?product_id=$matches[1]',
        'top'
    );
});

// Always flush rewrite rules after registration (one-time)
// Go to Settings → Permalinks or call:
// flush_rewrite_rules();
```

**Key Functions:**
- `add_rewrite_tag( $name, $regex )` — Register new query var
- `add_rewrite_rule( $regex, $query, $position )` — Add rewrite rule
- `add_rewrite_endpoint( $name, $places )` — Add endpoint (like `/trackback/`)
- `flush_rewrite_rules( $hard )` — Flush rules (expensive, do sparingly)
- `add_permastruct( $name, $structure, $args )` — Add permalink structure

## 4. Shortcode API

```php
// Enclosing shortcode: [my_shortcode]content[/my_shortcode]
// Self-closing:       [my_shortcode param="value"]

function myplugin_shortcode( $atts, $content = null, $tag = '' ) {
    // Normalize attributes with defaults
    $atts = shortcode_atts( array(
        'title' => 'Default Title',
        'count' => 5,
    ), $atts, $tag );

    // Security: escape output
    $title = esc_html( $atts['title'] );

    // Enclosing content
    $body = $content ? do_shortcode( $content ) : '';

    ob_start();
    ?>
    <div class="my-shortcode">
        <h3><?php echo $title; ?></h3>
        <?php echo wp_kses_post( $body ); ?>
    </div>
    <?php
    return ob_get_clean();
}
add_shortcode( 'my_shortcode', 'myplugin_shortcode' );

// Remove shortcode
remove_shortcode( 'my_shortcode' );

// Check if shortcode exists
shortcode_exists( 'gallery' );
```

**Best Practices:**
- Always use `shortcode_atts()` for defaults
- Return output, don't echo it
- Escape all output
- Use `ob_start()/ob_get_clean()` for complex HTML
- Call `do_shortcode()` on enclosed content if nested shortcodes needed

## 5. Transients API

Temporary caching with expiration.

```php
// Set transient (expire in 12 hours)
set_transient( 'myplugin_data', $expensive_data, 12 * HOUR_IN_SECONDS );

// Get transient
$data = get_transient( 'myplugin_data' );

// Delete
delete_transient( 'myplugin_data' );

// Network-wide
set_site_transient( 'myplugin_data', $value, DAY_IN_SECONDS );
get_site_transient( 'myplugin_data' );

// Pattern with fallback
if ( false === ( $data = get_transient( 'myplugin_data' ) ) ) {
    $data = expensive_operation();
    set_transient( 'myplugin_data', $data, HOUR_IN_SECONDS );
}

// Time constants (since WP 3.5)
MINUTE_IN_SECONDS  // 60
HOUR_IN_SECONDS    // 3600
DAY_IN_SECONDS     // 86400
WEEK_IN_SECONDS    // 604800
MONTH_IN_SECONDS   // 2592000
YEAR_IN_SECONDS    // 31536000
```

## 6. HTTP API

Making external HTTP requests.

```php
// GET request
$response = wp_remote_get( 'https://api.example.com/data', array(
    'timeout'     => 30,
    'headers'     => array( 'Accept' => 'application/json' ),
    'redirection' => 5,
) );

// POST request
$response = wp_remote_post( 'https://api.example.com/submit', array(
    'body' => array(
        'name'  => 'John',
        'email' => 'john@example.com',
    ),
) );

// Handle response
if ( is_wp_error( $response ) ) {
    $error_message = $response->get_error_message();
    // Handle error
} else {
    $body = wp_remote_retrieve_body( $response );
    $headers = wp_remote_retrieve_headers( $response );
    $code = wp_remote_retrieve_response_code( $response );
    $data = json_decode( $body, true );
}
```

## 7. Cron API

Schedule recurring or one-time tasks.

```php
// Schedule a recurring event (on plugin activation)
register_activation_hook( __FILE__, function() {
    if ( ! wp_next_scheduled( 'myplugin_daily_cleanup' ) ) {
        wp_schedule_event( time(), 'daily', 'myplugin_daily_cleanup' );
    }
});

// Hook the event
add_action( 'myplugin_daily_cleanup', 'myplugin_cleanup_function' );

// Clear on deactivation
register_deactivation_hook( __FILE__, function() {
    wp_clear_scheduled_hook( 'myplugin_daily_cleanup' );
});

// One-time event (schedule)
wp_schedule_single_event( time() + 300, 'myplugin_delayed_task' );

// Custom schedule
add_filter( 'cron_schedules', function( $schedules ) {
    $schedules['every_minute'] = array(
        'interval' => 60,
        'display'  => 'Every Minute',
    );
    return $schedules;
});
```

## 8. Widgets API

```php
// Register widget
class MyPlugin_Widget extends WP_Widget {
    public function __construct() {
        parent::__construct(
            'myplugin_widget',
            'My Plugin Widget',
            array( 'description' => 'Displays custom content' )
        );
    }

    public function widget( $args, $instance ) {
        echo $args['before_widget'];
        if ( ! empty( $instance['title'] ) ) {
            echo $args['before_title'] . esc_html( $instance['title'] ) . $args['after_title'];
        }
        echo '<p>' . esc_html( $instance['text'] ) . '</p>';
        echo $args['after_widget'];
    }

    public function form( $instance ) {
        $title = ! empty( $instance['title'] ) ? $instance['title'] : '';
        ?>
        <p>
            <label for="<?php echo esc_attr( $this->get_field_id( 'title' ) ); ?>">Title:</label>
            <input class="widefat" id="<?php echo esc_attr( $this->get_field_id( 'title' ) ); ?>"
                   name="<?php echo esc_attr( $this->get_field_name( 'title' ) ); ?>"
                   type="text" value="<?php echo esc_attr( $title ); ?>">
        </p>
        <?php
    }

    public function update( $new_instance, $old_instance ) {
        $instance = array();
        $instance['title'] = sanitize_text_field( $new_instance['title'] );
        return $instance;
    }
}

// Register widget
add_action( 'widgets_init', function() {
    register_widget( 'MyPlugin_Widget' );
});
```

## 9. Filesystem API

```php
// Get WP Filesystem instance
function myplugin_write_file( $file_path, $content ) {
    global $wp_filesystem;

    if ( ! function_exists( 'WP_Filesystem' ) ) {
        require_once ABSPATH . 'wp-admin/includes/file.php';
    }

    WP_Filesystem();

    if ( ! $wp_filesystem->exists( dirname( $file_path ) ) ) {
        $wp_filesystem->mkdir( dirname( $file_path ) );
    }

    $wp_filesystem->put_contents( $file_path, $content, FS_CHMOD_FILE );
}
```

## 10. Related Skills
- [Hooks API](skill_view(name='wordpress-documentation', file_path='references/hooks.md'))
- [Database API](skill_view(name='wordpress-documentation', file_path='references/database.md'))
- [Security API](skill_view(name='wordpress-documentation', file_path='references/security.md'))
