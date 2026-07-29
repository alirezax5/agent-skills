# WordPress User Admin — Dashboard & Settings Reference

> Source: https://wordpress.org/documentation/support-guides/ + https://wordpress.org/documentation/article/administration-screens/
> WordPress Version: 6.7+

## 1. Dashboard Structure

```
Dashboard
├── Home              # At a Glance, Activity, Quick Draft, Site Health, WP Events
├── Updates           # WordPress/plugin/theme/translation updates

Posts
├── All Posts         # List table, bulk edit, quick edit, search, filter
├── Add New           # Block Editor (Gutenberg)
├── Categories        # Hierarchical taxonomy management
├── Tags              # Non-hierarchical taxonomy

Media
├── Library           # Grid/List view of all uploaded files
├── Add New           # Upload interface (drag-drop)

Pages
├── All Pages         # List of static pages
├── Add New           # Block Editor for pages

Comments              # Approve/unapprove/spam/trash

Appearance
├── Themes            # Activate, install, preview themes
├── Customize         # Classic theme customizer (live preview)
├── Editor            # Site Editor (FSE for block themes)
├── Widgets           # Widget management or Block Widgets Editor
├── Menus             # Navigation menu builder
├── Header            # Custom header image (classic themes)
├── Background        # Custom background (classic themes)
├── Theme File Editor # Direct PHP editing via WP-Admin

Plugins
├── Installed Plugins # List, activate, deactivate
├── Add New           # Browse/install from WP repo
├── Plugin File Editor

Users
├── All Users         # List, add, edit, delete users
├── Add New           # Create new user
├── Your Profile      # Toolbar toggle, password, personal options, color scheme

Tools
├── Available Tools   # Shortcuts to import/export
├── Import            # From Blogger, WordPress, Tumblr, etc.
├── Export            # Content as WXR (XML)
├── Site Health       # Info + troubleshooting tabs
├── Export Personal Data
├── Erase Personal Data

Settings
├── General           # Site title, tagline, URL, language, timezone
├── Writing           # Default category, post via email, update services
├── Reading           # Front page, blog pages, search visibility, RSS
├── Discussion        # Comment settings, avatars, pingbacks
├── Media             # Thumbnail/medium/large image sizes
├── Permalinks        # URL structure configuration
├── Privacy           # Privacy policy page selection
```

## 2. Key Settings Reference

### General Settings (Options API)
```php
get_option( 'blogname' );        // Site title
get_option( 'blogdescription' ); // Tagline
get_option( 'siteurl' );         // WordPress address (URL)
get_option( 'home' );            // Site address (URL)
get_option( 'admin_email' );     // Admin email
get_option( 'timezone_string' ); // Timezone
get_option( 'date_format' );     // Date format ('F j, Y')
get_option( 'time_format' );     // Time format ('g:i a')
get_option( 'start_of_week' );   // 0=Sunday, 1=Monday
```

### Reading
```php
get_option( 'show_on_front' );        // 'posts' or 'page'
get_option( 'page_on_front' );        // Front page ID
get_option( 'page_for_posts' );       // Posts page ID
get_option( 'posts_per_page' );       // Blog pages show at most
get_option( 'blog_public' );          // 1=visible, 0=discourage search
```

### Discussion
```php
get_option( 'default_comment_status' ); // 'open' or 'closed'
get_option( 'default_ping_status' );    // 'open' or 'closed'
get_option( 'comment_moderation' );     // Moderate all comments
get_option( 'thread_comments' );        // Enable threaded (nested) comments
get_option( 'thread_comments_depth' );  // Levels: 1-10
get_option( 'comments_per_page' );      // Comments per page
get_option( 'require_name_email' );     // Name+email required
get_option( 'show_avatars' );           // Show avatars
```

### Media
```php
get_option( 'thumbnail_size_w' );  // 150
get_option( 'thumbnail_size_h' );  // 150
get_option( 'thumbnail_crop' );    // 1=crop to exact
get_option( 'medium_size_w' );     // 300
get_option( 'medium_size_h' );     // 300
get_option( 'large_size_w' );      // 1024
get_option( 'large_size_h' );      // 1024
```

### Permalinks
```php
get_option( 'permalink_structure' ); // '/%postname%/'
```

## 3. Admin Bar & Screen Options

- **Toolbar** (formerly Admin Bar): Quick access links, visible when logged in
- Toggle in Users → Your Profile → "Show Toolbar when viewing site"
- **Screen Options**: Each screen has configurable panels (click the tab at top-right)
- **Help**: Contextual help for the current screen (tab at top-right)

## 4. Site Health

Located at Tools → Site Health:

- **Status tab**: Critical issues, recommendations (plugin, theme, server checks)
- **Info tab**: WordPress, Active Theme, Active Plugins, Media Handling, Server, Database, etc.

Uses `WP_Site_Health` class. Developers can add tests:
```php
add_filter( 'site_status_tests', function( $tests ) {
    $tests['direct']['my_custom_test'] = array(
        'label' => 'Custom Test',
        'test'  => 'my_health_test_callback',
    );
    return $tests;
});
```
