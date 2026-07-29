# WordPress Coding Standards — PHP — Complete Reference

> Source: https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/
> WordPress Version: 6.7+

## 1. Overview

The WordPress Coding Standards are **mandatory for WordPress Core** and **recommended for all themes/plugins**. They cover not just code style but also interoperability, translatability, and security best practices.

## 2. Naming Conventions

```php
// ✅ Functions & Variables: lowercase, underscore-separated
function myprefix_get_data( $post_id ) {}
$my_variable;

// ❌ No camelCase
function myPrefixGetData() {}  // Wrong

// ✅ Classes: Capitalized_Words_Separated_By_Underscores
class WP_HTTP {};
class Walker_Category {};

// ✅ Interfaces: Capitalized + _Interface suffix
interface Logger_Interface {};

// ✅ Traits: Capitalized + _Trait suffix (or descriptive)
trait Forbid_Dynamic_Properties {};

// ✅ Enums: Capitalized_Words
enum Post_Status {};

// ✅ Constants: UPPERCASE with underscores
define( 'MYPLUGIN_VERSION', '1.0' );

// ✅ Files: lowercase, hyphen-separated
// my-plugin-name.php
// class class-wp-error.php (classes: class-{class-name}.php)
```

## 3. PHP Tags

```php
// ✅ Always full PHP tags
<?php ... ?>

// ❌ Never shorthand
<? ... ?>
<?= ... ?>
```

## 4. Quotes

```php
// ✅ Single quotes for strings without variables
echo '<a href="/static">Link</a>';

// ✅ Double quotes for strings with variables
echo "<a href='{$url}'>Link</a>";

// ✅ Alternate quotes to avoid escaping
// Instead of: echo '<a href="http://example.com">Link</a>';
// Those are fine — single quotes with double inside
```

## 5. Indentation & Whitespace

```php
// ✅ Tabs for code indentation, spaces for alignment
function demo( $param1,    $param2 ) {  // Spaces for alignment
    $result = $param1;                   // Tab for indentation
    return $result;
}

// ✅ Space after keywords, before/after operators
if ( $a === $b ) {
    foreach ( $items as $item ) {
        $total += $item;
    }
}
```

## 6. Brace Style

```php
// ✅ K&R style for control structures
if ( condition ) {
    // code
} elseif ( other_condition ) {
    // code
} else {
    // code
}

// ✅ Braces on new line for functions/classes
function my_function( $param ) {
    // code
}

class My_Class {
    // code
}
```

## 7. Yoda Conditions

```php
// ✅ Constant/ literal on LEFT
if ( true === $variable ) {}
if ( 10 === $count ) {}
if ( 'publish' === get_post_status() ) {}

// ❌ Variable on left (less safe, can miss typos)
if ( $variable = true ) {}  // Assignment, not comparison!
```

## 8. Database Queries

```php
// ✅ Always use $wpdb->prepare()
$results = $wpdb->get_results( $wpdb->prepare(
    "SELECT * FROM {$wpdb->posts} WHERE ID = %d AND post_type = %s",
    $id,
    $type
) );
```

## 9. require / include

```php
// ✅ No parentheses around path
require_once ABSPATH . 'file.php';

// ❌ Wrong
require_once ( ABSPATH . 'file.php' );

// ✅ Prefer require[_once] for critical files (fatal error vs warning)
```

## 10. Control Structures

```php
// ✅ Use elseif (one word), not else if
if ( condition ) {
    // ...
} elseif ( other ) {
    // ...
}

// ✅ Always use braces, even for single lines
if ( condition ) {
    $result = true;
}

// ❌ No braceless
if ( condition ) $result = true;
```

## 11. Ternary Operator

```php
// ✅ Use ternary for simple conditions
$type = $is_premium ? 'premium' : 'free';

// ✅ Use long form for complex conditions
$name = ! empty( $user_name ) ? $user_name : 'Guest';

// ❌ Avoid nested ternaries
// $x = $a ? ($b ? 'c' : 'd') : 'e';  // Hard to read
```

## 12. OOP Standards

```php
// ✅ One class/interface/trait/enum per file
// ✅ Always declare visibility (public/protected/private)
class My_Class {
    public $public_prop;
    protected $protected_prop;
    private $private_prop;

    public function my_method() {}
    protected function helper() {}
}

// ✅ Visibility and modifier order: abstract/final, visibility, static
abstract public static function my_method();
final public function my_method();
```

## 13. PHP 8.0+ Specific

- Avoid reserved keywords as parameter names (named arguments may break)
- Function parameter renaming is a **breaking change** since PHP 8.0
- Use union types, match expressions, named arguments where appropriate

```php
function process_item( int|string $id, bool $force = false ): array {
    return match ( true ) {
        is_int( $id ) => $this->get_by_id( $id ),
        is_string( $id ) => $this->get_by_slug( $id ),
    };
}
```

## 14. Auto-Check with PHPCS

```bash
# Install WordPress Coding Standards
composer create-project wp-coding-standards/wpcs --no-dev

# Run PHPCS
vendor/bin/phpcs --standard=WordPress my-file.php
vendor/bin/phpcs --standard=WordPress-Extra my-plugin/
vendor/bin/phpcs --standard=WordPress-Docs my-theme/
```

## 15. AI Reasoning Notes

When writing WordPress PHP code:
1. **Functions/variables**: `snake_case` — always
2. **Classes**: `Capitalized_Separated` — always
3. **Files**: `hyphen-separated.php` — always; class files: `class-name.php`
4. **Yoda conditions**: constant first
5. **Sanitize input**, **escape output**, **prepare SQL**
6. **elseif** = one word
7. **Braces**: K&R for control, new line for functions/classes
8. **Quotes**: single unless evaluating variables
9. Run `phpcs --standard=WordPress` before committing
