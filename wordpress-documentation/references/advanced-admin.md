# WordPress Advanced Administration — Debugging, Server, Performance

> Source: https://developer.wordpress.org/advanced-administration/
> WordPress Version: 6.7+

## 1. Debugging

### Enable Debug Mode (wp-config.php)
```php
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true );        // Log to /wp-content/debug.log
define( 'WP_DEBUG_DISPLAY', false );   // Don't show on screen (prod)
define( 'SCRIPT_DEBUG', true );        // Use unminified JS/CSS
define( 'SAVEQUERIES', true );         // Log DB queries ($wpdb->queries)
```

### Debug.Log Location
`/wp-content/debug.log` — check with `tail -f`

### Common Debug Techniques
1. Enable WP_DEBUG
2. Check `debug.log`
3. Disable plugins one-by-one
4. Switch to default theme
5. Increase memory: `define('WP_MEMORY_LIMIT', '256M');`
6. Use Query Monitor plugin
7. Check PHP error logs (server-level)

### Recovery Mode
If a fatal error occurs, WordPress enters Recovery Mode. An admin can:
1. Click the link in the admin email
2. Access the site temporarily to fix plugins/themes
3. The broken plugin/theme is paused

## 2. Server Configuration

### File Permissions
```bash
# Recommended
find /path/to/wp -type d -exec chmod 755 {} \;
find /path/to/wp -type f -exec chmod 644 {} \;
# wp-config.php — more restrictive
chmod 600 wp-config.php
```

### .htaccess (Apache)
```apache
# BEGIN WordPress
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
# END WordPress
```

### Nginx Rules
```nginx
location / {
    try_files $uri $uri/ /index.php?$args;
}

location ~ \.php$ {
    fastcgi_pass unix:/var/run/php-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}

# Block wp-config.php access
location ~* /wp-config.php { deny all; }

# Block uploads directory PHP execution
location ~* /uploads/.*\.php$ { deny all; }
```

### PHP Requirements (WP 6.7+)
- PHP 8.0+ (recommended: PHP 8.2+)
- MySQL 8.0+ or MariaDB 10.5+
- HTTPS support
- Memory limit: 128MB+ (recommended: 256MB)

## 3. Performance Optimization

### Caching Layers
1. **Page Cache** (static HTML) — WP Rocket, W3 Total Cache, LiteSpeed Cache
2. **Object Cache** — Redis, Memcached via `wp-config.php`:
   ```php
   define( 'WP_CACHE', true );
   ```
3. **OPcache** — PHP bytecode cache (enable in php.ini)
4. **CDN** — Static assets (Cloudflare, etc.)

### Database Optimization
```sql
-- Run periodically
OPTIMIZE TABLE wp_posts;
OPTIMIZE TABLE wp_postmeta;
OPTIMIZE TABLE wp_options;
```

Or use WP-CLI: `wp db optimize`

### Image Optimization
- Use WebP format (WordPress 5.8+ supports WebP)
- Set appropriate thumbnail sizes in Settings → Media
- Use a CDN for images
- Lazy load images (WordPress 5.9+ has built-in loading="lazy")

### Query Optimization
```php
// When pagination isn't needed:
new WP_Query( array(
    'no_found_rows'          => true,
    'update_post_meta_cache' => false,
    'update_post_term_cache' => false,
) );

// Only get IDs when that's all you need:
new WP_Query( array( 'fields' => 'ids' ) );
```

### Transients for Expensive Operations
```php
function get_expensive_data() {
    $cache = get_transient( 'myplugin_expensive_data' );
    if ( false === $cache ) {
        $cache = expensive_query();
        set_transient( 'myplugin_expensive_data', $cache, HOUR_IN_SECONDS );
    }
    return $cache;
}
```

## 4. Server Migrations Checklist

- [ ] Export DB: `wp db export backup.sql`
- [ ] Copy files via rsync or SCP
- [ ] Import DB on new server
- [ ] Search-replace URLs: `wp search-replace 'old.com' 'new.com' --all-tables`
- [ ] Flush rewrite rules: `wp rewrite flush`
- [ ] Regenerate .htaccess
- [ ] Update `wp-config.php` with new DB credentials
- [ ] Test all pages, forms, and search
- [ ] Set up caching and CDN

## 5. Troubleshooting

| Symptom | Common Cause | Fix |
|---|---|---|
| White screen of death | PHP fatal error | Enable WP_DEBUG, check debug.log |
| 500 Internal Server Error | .htaccess or plugin conflict | Rename .htaccess, disable plugins |
| Cannot install plugins | File permissions | chmod -R 755 wp-content |
| Connection timed out | Low PHP memory | Increase WP_MEMORY_LIMIT |
| 404 on all pages except home | Permalinks broken | Settings → Permalinks → Save |
| "Briefly unavailable" | WP Recovery Mode | Check email, click recovery link |
| Slow admin | Too many plugins | Audit plugins, enable object cache |
| Email not sending | Server mail misconfigured | Use SMTP plugin (WP Mail SMTP) |

## 6. Related Skills

- [Security API](skill_view(name='wordpress-documentation', file_path='references/security.md'))
- [Database API](skill_view(name='wordpress-documentation', file_path='references/database.md'))
