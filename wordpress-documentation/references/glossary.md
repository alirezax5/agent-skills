# WordPress Glossary — 50+ Key Terms

> Source: https://wordpress.org/documentation/article/wordpress-glossary/
> WordPress Version: 6.7+

## Core Concepts

| Term | Definition |
|---|---|
| **Post** | Chronological content entry. Stored in `wp_posts` with `post_type='post'` |
| **Page** | Static, timeless content. `post_type='page'` |
| **Custom Post Type (CPT)** | User-defined content type via `register_post_type()` |
| **Taxonomy** | Mechanism to group content (Categories, Tags, custom) |
| **Term** | An item within a taxonomy (e.g., "News" in Categories) |
| **Meta** | Key-value data attached to posts, users, terms, or comments |
| **Hook** | Extension point where code can be added/modified |
| **Action** | Hook type that adds/removes functionality (`do_action`) |
| **Filter** | Hook type that modifies content (`apply_filters`, must return) |

## Architecture

| Term | Definition |
|---|---|
| **Theme** | Controls site appearance. Lives in `wp-content/themes/` |
| **Plugin** | Adds features. Lives in `wp-content/plugins/` |
| **Must-Use Plugin** | Auto-activated plugin in `wp-content/mu-plugins/` |
| **Block Theme** | Theme using HTML block markup, enables Full Site Editing |
| **Classic Theme** | Theme using PHP template files with template tags |
| **Child Theme** | Inherits parent theme, safe to modify |
| **theme.json** | Configuration file for block themes (settings + styles) |
| **WordPress Loop** | `while(have_posts()): the_post();` — standard post iteration |

## Block Editor / Gutenberg

| Term | Definition |
|---|---|
| **Block** | Unit of content markup (paragraph, image, heading, etc.) |
| **Block Type** | Blueprint defining block behavior (e.g., `core/paragraph`) |
| **Block Pattern** | Pre-designed block layouts |
| **Block Template** | Predefined block arrangement (for CPTs or pages) |
| **Template Part** | Reusable block (header, footer) in block themes |
| **Full Site Editing (FSE)** | Edit all site parts via Block Editor |
| **Global Styles** | Design settings applied site-wide via theme.json |
| **InnerBlocks** | Component allowing nested blocks inside a block |

## Users & Security

| Term | Definition |
|---|---|
| **Role** | Set of capabilities (Administrator, Editor, Author, etc.) |
| **Capability** | Specific permission (e.g., `edit_posts`, `publish_posts`) |
| **Nonce** | One-time security token (CSRF protection) |
| **Transient** | Cached data with expiration time |
| **Cron** | WordPress pseudo-cron for scheduled tasks |
| **Revision** | Saved version of a post/page |

## Database

| Term | Definition |
|---|---|
| **WP_Query** | Class for fetching posts from database |
| **$wpdb** | Global database access class for direct SQL |
| **Options** | Key-value settings in `wp_options` table |
| **Transients** | Expiring cached data in `wp_options` table |
| **Metadata** | Key-value data specific to posts/users/terms/comments |
| **wp-config.php** | Core configuration file (DB credentials, debug settings) |

## Development

| Term | Definition |
|---|---|
| **Shortcode** | `[shortcode]` macro that expands to HTML via `add_shortcode()` |
| **REST API** | JSON API for external interaction (`/wp-json/wp/v2/`) |
| **WP-CLI** | Command-line management tool |
| **Localization (l10n)** | Translating strings to a locale |
| **Internationalization (i18n)** | Making code translatable (`__()`, `_e()`, etc.) |
| **Enqueue** | Properly loading scripts/styles via `wp_enqueue_script()` |
| **Sanitization** | Cleaning input data (removing dangerous content) |
| **Escaping** | Securing output data for safe display (`esc_html()`) |
| **Validation** | Checking data against a pattern (reject if invalid) |

## Server / Environment

| Term | Definition |
|---|---|
| **Permalink** | Pretty URL structure (`/%postname%/`) |
| **Rewrite Rules** | URL-to-query mapping rules |
| **.htaccess** | Apache config file for rewrites and security |
| **Multisite** | Single WP install running multiple sites |
| **Object Cache** | In-memory caching (Redis, Memcached) |
| **OPcache** | PHP bytecode cache |
| **CDN** | Content Delivery Network for static assets |
