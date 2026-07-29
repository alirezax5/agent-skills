# WordPress Theme Development — Complete Reference

> Source: https://developer.wordpress.org/themes/
> WordPress Version: 6.7+

## 1. Overview

WordPress themes control the visual presentation of content. Two paradigms exist:

| Type | Files | Key Feature |
|---|---|---|
| **Classic Theme** | PHP files (template tags) | Traditional, full PHP control |
| **Block Theme** | HTML files (block markup) | Full Site Editing, theme.json |

### Required Files
1. **style.css** — Theme header (REQUIRED for all themes)
2. **index.php** (classic) or **index.html** (block) — Catch-all fallback

## 2. style.css Header
```css
/*
Theme Name: My Theme
Theme URI: https://example.com
Author: Your Name
Author URI: https://example.com
Description: Theme description
Version: 1.0.0
Requires at least: 6.0
Tested up to: 6.7
Requires PHP: 8.0
License: GPL v2 or later
Text Domain: my-theme
Tags: blog, one-column, custom-logo, featured-images
*/
```

## 3. Template Hierarchy (Classic Themes)

```
FRONT: front-page.php → home.php → page.php → singular.php → index.php
POST:  single-{post-type}-{slug}.php → single-{post-type}.php → single.php → singular.php → index.php
PAGE:  {custom-template}.php → page-{slug}.php → page-{id}.php → page.php → singular.php → index.php
CAT:   category-{slug}.php → category-{id}.php → category.php → archive.php → index.php
TAG:   tag-{slug}.php → tag-{id}.php → tag.php → archive.php → index.php
TAX:   taxonomy-{tax}-{term}.php → taxonomy-{tax}.php → taxonomy.php → archive.php → index.php
CPT:   archive-{post-type}.php → archive.php → index.php
AUTH:  author-{nicename}.php → author-{id}.php → author.php → archive.php → index.php
DATE:  date.php → archive.php → index.php
SEARCH: search.php → index.php
404:    404.php → index.php
```

## 4. Classic Theme Template Tags

```php
<?php get_header(); ?>
<?php get_footer(); ?>
<?php get_sidebar(); ?>
<?php get_template_part( 'loop' ); ?>
<?php get_search_form(); ?>

// The Loop
<?php if ( have_posts() ) : while ( have_posts() ) : the_post(); ?>
    <h2><?php the_title(); ?></h2>
    <?php the_content(); ?>
    <?php the_post_thumbnail( 'medium' ); ?>
    <?php the_category( ', ' ); ?>
    <?php the_tags(); ?>
    <?php the_meta(); ?>
    <?php edit_post_link(); ?>
<?php endwhile; endif; ?>
```

## 5. Block Theme Templates (HTML)

```html
<!-- wp:template-part {"slug":"header","area":"header"} /-->

<!-- wp:group {"tagName":"main"} -->
<main class="wp-block-group">
    <!-- wp:post-title /-->
    <!-- wp:post-content /-->
</main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer","area":"footer"} /-->
```

Parts in `/parts/header.html`, `/parts/footer.html`.

## 6. functions.php — Theme Setup

```php
<?php
declare( strict_types=1 );

function mytheme_setup(): void {
    // Translations
    load_theme_textdomain( 'mytheme', get_template_directory() . '/languages' );

    // Features
    add_theme_support( 'automatic-feed-links' );
    add_theme_support( 'post-thumbnails' );
    add_theme_support( 'title-tag' );
    add_theme_support( 'custom-logo', array(
        'height'      => 100,
        'width'       => 400,
        'flex-height' => true,
        'flex-width'  => true,
    ) );
    add_theme_support( 'html5', array(
        'search-form', 'comment-form', 'comment-list',
        'gallery', 'caption', 'style', 'script',
    ) );
    add_theme_support( 'responsive-embeds' );

    // Menus
    register_nav_menus( array(
        'primary' => __( 'Primary Menu', 'mytheme' ),
        'footer'  => __( 'Footer Menu', 'mytheme' ),
    ) );
}
add_action( 'after_setup_theme', 'mytheme_setup' );

// Content width
function mytheme_content_width(): void {
    $GLOBALS['content_width'] = apply_filters( 'mytheme_content_width', 1200 );
}
add_action( 'after_setup_theme', 'mytheme_content_width', 0 );

// Enqueue assets
function mytheme_scripts(): void {
    wp_enqueue_style( 'mytheme-style', get_stylesheet_uri(), array(), wp_get_theme()->get( 'Version' ) );
    wp_enqueue_script( 'mytheme-script', get_template_directory_uri() . '/js/script.js',
        array( 'jquery' ), '1.0', array( 'strategy' => 'defer' ) );

    if ( is_singular() && comments_open() && get_option( 'thread_comments' ) ) {
        wp_enqueue_script( 'comment-reply' );
    }
}
add_action( 'wp_enqueue_scripts', 'mytheme_scripts' );

// Widget areas
function mytheme_widgets_init(): void {
    register_sidebar( array(
        'name'          => __( 'Sidebar', 'mytheme' ),
        'id'            => 'sidebar-1',
        'before_widget' => '<section id="%1$s" class="widget %2$s">',
        'after_widget'  => '</section>',
        'before_title'  => '<h3 class="widget-title">',
        'after_title'   => '</h3>',
    ) );
}
add_action( 'widgets_init', 'mytheme_widgets_init' );
```

## 7. theme.json (Block Themes)

```json
{
    "$schema": "https://schemas.wp.org/trunk/theme.json",
    "version": 2,
    "settings": {
        "color": {
            "palette": [
                { "slug": "primary", "color": "#1a1a2e", "name": "Primary" },
                { "slug": "secondary", "color": "#eaeaea", "name": "Secondary" }
            ]
        },
        "typography": {
            "fontFamilies": [
                { "slug": "inter", "name": "Inter", "fontFamily": "Inter, sans-serif" }
            ],
            "fontSizes": [
                { "slug": "small", "size": "14px", "name": "Small" },
                { "slug": "medium", "size": "18px", "name": "Medium" }
            ]
        },
        "layout": {
            "contentSize": "800px",
            "wideSize": "1200px"
        }
    },
    "styles": {
        "blocks": {
            "core/post-title": {
                "typography": { "fontSize": "var(--wp--preset--font-size--large)" }
            }
        }
    }
}
```

## 8. Enqueuing Assets

```php
// Style
wp_enqueue_style( 'handle', get_template_directory_uri() . '/css/file.css',
    array( 'dependency' ), '1.0', 'all' );

// Script
wp_enqueue_script( 'handle', get_template_directory_uri() . '/js/file.js',
    array( 'jquery' ), '1.0', array( 'strategy' => 'defer' ) );

// Block-style (block themes — per-block)
wp_enqueue_block_style( "core/latest-comments", array(
    'handle' => 'mytheme-latest-comments',
    'src'    => get_theme_file_uri( "assets/css/blocks/latest-comments.css" ),
    'path'   => get_theme_file_path( "assets/css/blocks/latest-comments.css" ),
) );
```

## 9. Child Themes

```css
/*
Theme Name: My Child Theme
Template: parent-theme-folder   ← REQUIRED
*/
```

Child theme `functions.php` loads **before** parent's. Use `parent_theme_slug` to enqueue parent style.

## 10. Related Skills

- [Hooks API](skill_view(name='wordpress-documentation', file_path='references/hooks.md'))
- [Database API](skill_view(name='wordpress-documentation', file_path='references/database.md'))
- [Security API](skill_view(name='wordpress-documentation', file_path='references/security.md'))
- [Coding Standards](skill_view(name='wordpress-documentation', file_path='references/coding-standards.md'))
- [Block Editor](skill_view(name='wordpress-documentation', file_path='references/block-editor.md'))
