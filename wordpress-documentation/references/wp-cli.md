# WP-CLI — Complete Command Reference

> Source: https://developer.wordpress.org/cli/commands/
> WordPress Version: 6.7+

## 1. Overview

WP-CLI is the official command-line tool for managing WordPress installations. It enables rapid management without the admin interface.

### Common Global Flags
```bash
--path=<path>       # Path to WordPress files
--url=<url>         # Site URL (multisite)
--user=<id|login>   # Run as specific user
--skip-plugins      # Skip all plugins
--skip-themes       # Skip all themes
--format=<format>   # table, json, csv, yaml, count
```

## 2. Essential Commands

### Core Management
```bash
wp core download               # Download WP
wp core install                # Install WP (requires DB)
wp core version                # Check WP version
wp core update                 # Update WP
wp core update-db              # Update database
wp core check-update           # Check updates
```

### Database
```bash
wp db create                   # Create database
wp db drop                     # Drop database
wp db reset                    # Reset database
wp db export [file.sql]        # Export database
wp db import [file.sql]        # Import database
wp db optimize                 # Optimize tables
wp db search 'old' 'new'      # Search & replace dry-run
wp db query 'SELECT * FROM ...' # Run SQL directly
```

### Search & Replace (Critical for Migrations)
```bash
wp search-replace 'http://old.com' 'http://new.com' --all-tables
wp search-replace 'old' 'new' --precise  # Exact match
wp search-replace 'old' 'new' --dry-run  # Preview
```

### Posts & Content
```bash
wp post list                    # List posts
wp post create --post_title='Title' --post_content='Content' --post_status=publish
wp post update 123 --post_title='New Title'
wp post delete 123 456          # Delete multiple
wp post meta list 123           # List post meta
wp post meta get 123 key_name   # Get meta
wp post meta update 123 key 'value'
```

### Post Types & Taxonomies
```bash
wp post-type list              # List registered post types
wp taxonomy list               # List registered taxonomies
wp term list category          # List terms in taxonomy
wp term create category 'News' --description='News category'
```

### Plugins
```bash
wp plugin list                 # List all plugins
wp plugin status               # Show active/inactive
wp plugin activate hello       # Activate plugin
wp plugin deactivate hello     # Deactivate
wp plugin install woocommerce  # Install (from WP repo)
wp plugin install plugin.zip   # Upload zip
wp plugin update --all         # Update all
wp plugin delete hello         # Delete
wp plugin auto-updates enable hello
```

### Themes
```bash
wp theme list                  # List themes
wp theme activate twentytwenty # Activate
wp theme install twentytwenty --activate
wp theme update --all
wp theme delete twentynineteen
```

### Users
```bash
wp user list                   # List all users
wp user create bob bob@example.com --role=editor
wp user update bob --display_name='Robert'
wp user delete 123             # Delete user (reassign posts to 1)
wp user meta list 123
wp user reset-password bob     # Send reset email
wp user session list 123       # Active sessions
wp user session destroy 123    # Destroy all sessions
```

### Options & Settings
```bash
wp option list                 # List all options
wp option get siteurl          # Get option
wp option update blogname 'My Site'
wp option delete myplugin_option
wp option get home             # Site URL
wp option get siteurl          # WP URL
```

### Cache & Transients
```bash
wp cache flush                 # Flush object cache
wp transient delete-all        # Delete all transients
wp transient list              # List transients
```

### Cron
```bash
wp cron event list             # List scheduled events
wp cron event schedule my_hook  # Schedule event
wp cron event run my_hook      # Run immediately
wp cron event delete my_hook   # Remove event
wp cron test                   # Test cron system
```

### Rewrite Rules
```bash
wp rewrite flush               # Flush rewrite rules
wp rewrite list                # List rules
wp rewrite structure '/%postname%/'
```

### Scaffolding (Code Generation)
```bash
wp scaffold plugin my-plugin         # Plugin skeleton
wp scaffold theme my-theme           # Theme skeleton
wp scaffold child-theme my-child     # Child theme from parent
wp scaffold post-type movie          # CPT code
wp scaffold taxonomy genre --post_types=movie
wp scaffold block my-block           # Block plugin
wp scaffold _sass-based-theme mytheme # WP-sass theme
```

### Maintenance
```bash
wp maintenance-mode activate        # Enable maintenance
wp maintenance-mode deactivate      # Disable
wp maintenance-mode status          # Check status
```

### Site Health
```bash
wp site health check                # Check site health
wp site health status               # Show issues
```

### Multisite
```bash
wp site create --slug=newsite       # Create site
wp site list                        # List sites
wp site archive 2                   # Archive site
wp site activate 2                  # Activate site
wp site delete 2                    # Delete site
wp super-admin list
wp super-admin add bob
```

### Language
```bash
wp language core install fa_IR      # Install Persian
wp language core activate fa_IR     # Activate
wp language plugin install woocommerce fa_IR
wp language theme install twentytwenty fa_IR
```

## 3. Development Utilities
```bash
wp shell                    # Interactive PHP console
wp eval 'echo "hello";'     # Run PHP code
wp eval-file script.php     # Run PHP file
wp server                   # Built-in PHP server (port 8080)
wp help                     # Help system
wp cli info                 # WP-CLI version & environment
wp cli update               # Update WP-CLI itself
```

## 4. Security Tips
- Never run `search-replace` without `--dry-run` first on production
- Always back up database before destructive operations
- Use `--user=<id>` when running privileged commands via automation
- WP-CLI commands respect WordPress permissions (capabilities)

## 5. AI Reasoning Notes
- `wp search-replace` is the most critical command for site migrations
- Always pair with `--dry-run` first
- Use `wp db export` before any destructive action
- Scaffold (`wp scaffold`) saves hours of boilerplate
- Use `wp eval` for quick one-off PHP tests
