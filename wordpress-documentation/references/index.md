# WordPress Documentation — Cross-Reference Index

> Use this to navigate between skill reference files.

## Topic → File Mapping

| Topic | File | Load Command |
|---|---|---|
| Glossary (50+ terms) | glossary.md | `file_path='references/glossary.md'` |
| User Admin & Settings | user-admin.md | `file_path='references/user-admin.md'` |
| Theme Development | theme-dev.md | `file_path='references/theme-dev.md'` |
| Plugin Development | plugin-dev.md | `file_path='references/plugin-dev.md'` |
| Block Editor / Gutenberg | block-editor.md | `file_path='references/block-editor.md'` |
| REST API | rest-api.md | `file_path='references/rest-api.md'` |
| WP-CLI | wp-cli.md | `file_path='references/wp-cli.md'` |
| Hooks (Actions & Filters) | hooks.md | `file_path='references/hooks.md'` |
| Database & WP_Query | database.md | `file_path='references/database.md'` |
| Security (escaping, sanitization, nonce) | security.md | `file_path='references/security.md'` |
| Coding Standards (PHP) | coding-standards.md | `file_path='references/coding-standards.md'` |
| APIs (Options, Settings, Rewrite, Shortcode, Transients, HTTP, Cron, Widgets) | apis.md | `file_path='references/apis.md'` |
| Advanced Admin (Debug, Server, Performance) | advanced-admin.md | `file_path='references/advanced-admin.md'` |

## Cross-Reference Matrix

```
                    glossary  user-admin  theme  plugin  block-editor  rest-api  wp-cli  hooks  database  security  coding-stds  apis  advanced
glossary           ──────────     ✓        ✓       ✓         ✓           ✓        ✓       ✓       ✓         ✓           ✓         ✓       ✓
user-admin             ✓       ──────────   ✓       ✓         ✓           ✓                ✓                       ✓                   ✓
theme                  ✓           ✓     ────────── ✓                   ✓        ✓       ✓       ✓         ✓           ✓         ✓       ✓
plugin                 ✓           ✓        ✓    ──────────              ✓        ✓       ✓       ✓         ✓           ✓         ✓       ✓
block-editor           ✓           ✓        ✓       ✓      ──────────    ✓                ✓       ✓                                         ✓
rest-api               ✓           ✓        ✓       ✓         ✓        ──────────        ✓               ✓           ✓
wp-cli                 ✓           ✓        ✓       ✓         ✓          ✓      ──────────                 ✓                   ✓       ✓
hooks                  ✓           ✓        ✓       ✓         ✓                             ──────────     ✓           ✓         ✓
database               ✓           ✓        ✓       ✓                                     ✓   ──────────   ✓           ✓
security               ✓           ✓        ✓       ✓         ✓          ✓                          ✓   ──────────     ✓
coding-stds            ✓           ✓        ✓       ✓                             ✓         ✓     ✓     ──────────     ✓
apis                   ✓           ✓        ✓       ✓         ✓                     ✓       ✓     ✓                   ──────────
advanced               ✓           ✓        ✓       ✓         ✓                     ✓             ✓     ✓             ✓     ──────────
```

## Dependency Chains

When working on a specific task, load these files:

```
Building a Custom Block:
  block-editor.md → hooks.md → security.md → coding-standards.md

Creating a Plugin:
  plugin-dev.md → hooks.md → database.md → security.md → apis.md → coding-standards.md

Building a Theme:
  theme-dev.md → hooks.md → security.md → coding-standards.md

REST API Endpoint:
  rest-api.md → security.md → database.md → hooks.md

Performance Tuning:
  advanced-admin.md → database.md → apis.md (transients)

Security Audit:
  security.md → coding-standards.md → hooks.md (capabilities)
```

## All Load Commands

```bash
# Load main index
skill_view(name='wordpress-documentation')

# Load specific reference
skill_view(name='wordpress-documentation', file_path='references/hooks.md')
skill_view(name='wordpress-documentation', file_path='references/database.md')
skill_view(name='wordpress-documentation', file_path='references/security.md')
skill_view(name='wordpress-documentation', file_path='references/coding-standards.md')
skill_view(name='wordpress-documentation', file_path='references/block-editor.md')
skill_view(name='wordpress-documentation', file_path='references/rest-api.md')
skill_view(name='wordpress-documentation', file_path='references/wp-cli.md')
skill_view(name='wordpress-documentation', file_path='references/apis.md')
skill_view(name='wordpress-documentation', file_path='references/theme-dev.md')
skill_view(name='wordpress-documentation', file_path='references/plugin-dev.md')
skill_view(name='wordpress-documentation', file_path='references/advanced-admin.md')
skill_view(name='wordpress-documentation', file_path='references/glossary.md')
skill_view(name='wordpress-documentation', file_path='references/user-admin.md')
```
