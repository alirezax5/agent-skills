---
name: "wordpress-documentation"
title: "WordPress Documentation — Complete Developer Reference"
description: "Complete practical reference from official WordPress docs (wordpress.org/documentation, developer.wordpress.org). 14 focused reference files covering: User Admin, Glossary, Theme Dev, Plugin Dev, Block Editor, REST API, WP-CLI, Hooks, Database, Security, Coding Standards, APIs, Advanced Admin. Production-quality PHP 8.2+ examples. Cross-index included."
version: "4.0"
author: "Hermes Agent"
last_updated: "2026-07-30"
category: "software-development"
tags: ["wordpress", "documentation", "theme-dev", "plugin-dev", "block-editor", "rest-api", "security", "wp-cli", "coding-standards", "hooks", "database"]
---

# WordPress Documentation — Complete Developer Reference

## Quick Fetch Trick

Every page on `wordpress.org` and `developer.wordpress.org` supports `?output_format=md`:

```bash
# Get any page as clean Markdown (no HTML scraping needed!)
curl -sL "https://developer.wordpress.org/reference/functions/wp_enqueue_script/?output_format=md"
curl -sL "https://wordpress.org/documentation/article/wordpress-glossary/?output_format=md"
curl -sL "https://developer.wordpress.org/themes/?output_format=md"
curl -sL "https://developer.wordpress.org/plugins/?output_format=md"
curl -sL "https://developer.wordpress.org/rest-api/?output_format=md"
```

## Reference Files (14 Total)

| # | File | Content | Load Command |
|---|---|---|---|
| — | **SKILL.md** (this file) | Index, URLs, quick fetch | `skill_view(name='wordpress-documentation')` |
| — | **references/index.md** | Cross-reference matrix, dependency chains | `file_path='references/index.md'` |
| 1 | **references/glossary.md** | 50+ key WP terms & definitions | `file_path='references/glossary.md'` |
| 2 | **references/user-admin.md** | Dashboard structure, settings API reference | `file_path='references/user-admin.md'` |
| 3 | **references/hooks.md** | Hooks (Actions & Filters) — full API reference | `file_path='references/hooks.md'` |
| 4 | **references/database.md** | WP_Query, $wpdb, Metadata API, Options API | `file_path='references/database.md'` |
| 5 | **references/security.md** | Escaping, sanitizing, validation, nonces, capabilities | `file_path='references/security.md'` |
| 6 | **references/coding-standards.md** | WP PHP Coding Standards (naming, Yoda, braces, OOP) | `file_path='references/coding-standards.md'` |
| 7 | **references/theme-dev.md** | Template hierarchy, functions.php, theme.json, enqueuing | `file_path='references/theme-dev.md'` |
| 8 | **references/plugin-dev.md** | Plugin structure, CPT, taxonomies, hooks, security | `file_path='references/plugin-dev.md'` |
| 9 | **references/block-editor.md** | Block Editor/Gutenberg — creating blocks, APIs | `file_path='references/block-editor.md'` |
| 10 | **references/rest-api.md** | REST API — routes, auth, custom endpoints | `file_path='references/rest-api.md'` |
| 11 | **references/wp-cli.md** | WP-CLI — all commands reference | `file_path='references/wp-cli.md'` |
| 12 | **references/apis.md** | Options, Settings, Rewrite, Shortcode, Transients, HTTP, Cron, Widgets, Filesystem | `file_path='references/apis.md'` |
| 13 | **references/advanced-admin.md** | Debugging, server config, performance, migrations | `file_path='references/advanced-admin.md'` |

## Developer Documentation Map

```
developer.wordpress.org/
├── /reference/                # Code reference (functions, classes, hooks)
├── /block-editor/             # Block Editor handbook
├── /themes/                   # Theme development handbook
├── /plugins/                  # Plugin development handbook
├── /rest-api/                 # REST API handbook
├── /apis/                     # Common APIs (Options, Rewrite, Shortcodes, etc.)
├── /cli/commands/             # WP-CLI commands
├── /coding-standards/         # PHP/JS/CSS/HTML standards
├── /advanced-administration/  # Debug, server config, security
├── /playground/               # WordPress in-browser (WebAssembly)
├── /news/                     # Developer Blog
```

## Key URL Quick Reference

```
# User Docs (wordpress.org/documentation)
/overview/                  # Intro, FAQs, first steps
/technical-guides/          # Installation, maintenance, security
/support-guides/            # Dashboard, publishing, media
/customization/             # Appearance, blocks, site editor
/article/wordpress-glossary/ # Full glossary

# Developer Docs (developer.wordpress.org)
/reference/                 # Code reference (functions, classes, hooks)
/reference/functions/       # All WP functions
/reference/hooks/           # All action/filter hooks
/block-editor/              # Block development
/themes/                    # Theme development
/plugins/                   # Plugin development
/rest-api/                  # REST API
/cli/commands/              # WP-CLI reference
/coding-standards/          # Coding standards
/advanced-administration/   # Server, debug, optimization
/playground/                # WP in browser

# Learn & Community
learn.wordpress.org         # Tutorials, courses
wordpress.org/support/forums # Support
make.wordpress.org          # Contributing
wordpress.org/themes/       # Theme directory
wordpress.org/plugins/      # Plugin directory
```
