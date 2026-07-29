# WordPress REST API — Complete Reference

> Source: https://developer.wordpress.org/rest-api/
> WordPress Version: 6.7+

## 1. Overview

The WordPress REST API provides a structured JSON interface for interacting with WordPress content externally. It powers the Block Editor and enables headless WordPress architectures.

### Key Concepts
- **Routes**: URLs representing API endpoints (e.g., `/wp/v2/posts`)
- **Endpoints**: Individual functions on a route (GET, POST, DELETE)
- **Schema**: Defines available data and formats
- **Authentication**: Proves identity for protected endpoints
- **Response**: JSON object returned by the API

### Default Routes
| Route | Endpoints | Description |
|---|---|---|
| `/wp/v2/posts` | GET, POST | Blog posts |
| `/wp/v2/pages` | GET, POST | Pages |
| `/wp/v2/users` | GET, POST | Users |
| `/wp/v2/categories` | GET, POST | Categories |
| `/wp/v2/tags` | GET, POST | Tags |
| `/wp/v2/media` | GET, POST | Media items |
| `/wp/v2/types` | GET | Post types |
| `/wp/v2/statuses` | GET | Post statuses |
| `/wp/v2/settings` | GET, PUT | Site settings |
| `/wp/v2/themes` | GET | Themes |
| `/wp/v2/plugins` | GET, POST, PUT, DELETE | Plugins |
| `/wp/v2/block-types` | GET | Block types |
| `/wp/v2/block-directory/search` | GET | Search blocks |
| `/wp/v2/search` | GET | Search across types |
| `/wp/v2/pattern-directory/patterns` | GET | Block patterns |

## 2. Using the REST API

### Authentication Methods

| Method | Use Case |
|---|---|
| **Cookie/Nonce** | Plugin/theme AJAX (logged-in users) |
| **Application Password** | External apps, CLI tools (since WP 5.6) |
| **OAuth** | Third-party apps (via plugin) |

### Basic Fetch Examples

```javascript
// GET posts (public — no auth needed)
fetch('https://example.com/wp-json/wp/v2/posts?per_page=10')
    .then(res => res.json())
    .then(posts => console.log(posts));

// POST with application password
const response = await fetch('https://example.com/wp-json/wp/v2/posts', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Basic ' + btoa('username:app_password')
    },
    body: JSON.stringify({
        title: 'API Created Post',
        content: 'Hello from REST API',
        status: 'publish'
    })
});
```

### PHP Client Example (Internal)
```php
// Using wp_remote_get()
$response = wp_remote_get( rest_url( 'wp/v2/posts/1' ) );
$body = wp_remote_retrieve_body( $response );
$data = json_decode( $body );

// Using wp_remote_post()
wp_remote_post( rest_url( 'wp/v2/posts' ), array(
    'headers' => array(
        'Content-Type' => 'application/json',
        'X-WP-Nonce'   => wp_create_nonce( 'wp_rest' ),
    ),
    'body' => json_encode( array(
        'title'  => 'New Post',
        'status' => 'draft',
    ) ),
));
```

## 3. Custom Routes

```php
// Register custom route
add_action( 'rest_api_init', function() {
    register_rest_route( 'myplugin/v1', '/items', array(
        'methods'             => 'GET',
        'callback'            => 'myplugin_get_items',
        'permission_callback' => function() {
            return current_user_can( 'read' );
        },
        'args' => array(
            'page' => array(
                'required'          => false,
                'default'           => 1,
                'sanitize_callback' => 'absint',
                'validate_callback' => function( $param ) {
                    return is_numeric( $param ) && $param > 0;
                },
            ),
        ),
    ));
});

function myplugin_get_items( WP_REST_Request $request ) {
    $page = $request->get_param( 'page' );
    $items = get_posts( array(
        'posts_per_page' => 10,
        'paged'          => $page,
    ) );

    // Return WP_REST_Response
    return new WP_REST_Response( $items, 200 );
}
```

### Extending Existing Routes
```php
// Add field to existing post response
add_action( 'rest_api_init', function() {
    register_rest_field( 'post', 'featured_meta', array(
        'get_callback' => function( $post ) {
            return get_post_meta( $post['id'], 'featured', true );
        },
        'update_callback' => function( $value, $post ) {
            update_post_meta( $post->ID, 'featured', $value );
        },
        'schema' => array(
            'type'        => 'string',
            'description' => 'Featured post meta',
            'context'     => array( 'view', 'edit' ),
        ),
    ));
});
```

## 4. Security

- Always set `permission_callback` (required since WP 5.5)
- Use nonce for internal requests: `wp_create_nonce('wp_rest')`
- Validate and sanitize all parameters
- Rate limit if exposing to anonymous users
- Use `current_user_can()` for permission checks

## 5. Performance

- Use `?_embed` to include related data in one request
- Paginate with `per_page` and `page` parameters
- Use ETag/Last-Modified headers for caching
- Consider `register_rest_route()` with caching plugins

## 6. AI Reasoning Notes

1. Internal AJAX: use nonce auth with `apiFetch` (WordPress package)
2. External apps: use Application Passwords
3. Always include `permission_callback` (required since 5.5)
4. Use `register_rest_field()` to extend existing routes
5. Use `register_rest_route()` for new custom endpoints
6. Validate all params with `sanitize_callback` and `validate_callback`
