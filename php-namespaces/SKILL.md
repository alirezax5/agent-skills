---
name: php-namespaces
description: PHP Namespaces & Autoloading — PSR-4, Composer autoloading, use statements, name resolution, and modern namespace patterns for PHP 8.4 with strict_types
tags: [php, namespaces, autoloading, psr-4, composer, strict-types]
---

# PHP Namespaces & Autoloading

> **PHP 8.4** · `declare(strict_types=1)` · PSR-4, Composer, and modern namespace patterns

---

## 1. Overview

Namespaces in PHP solve the problem of name collisions between code from different sources. Combined with autoloading — especially PSR-4 via Composer — they form the foundation of modern PHP application architecture.

---

## 2. Declaring Namespaces

### 2.1 File-Level Declaration

Every PHP file that uses namespaces must start with a `namespace` declaration:

```php
<?php
declare(strict_types=1);

namespace App\Controller;
```

> **Rule**: One namespace per file. The namespace declaration must be on the first line after `<?php` and `declare()`.

### 2.2 Sub-Namespaces

```php
<?php
declare(strict_types=1);

namespace App\Domain\User\ValueObject;
```

---

## 3. Importing (Use Statements)

### 3.1 `use` Keyword

```php
<?php
declare(strict_types=1);

namespace App\Controller;

use App\Service\UserService;
use App\Exception\UserNotFoundException;
```

### 3.2 Use Function and Constant (PHP 5.6+)

```php
use function array_map;
use function strtoupper;
use const PHP_VERSION;
```

### 3.3 Grouped Use (PHP 7.0+)

```php
use App\Exception\{
    UserNotFoundException,
    PaymentFailedException,
    OrderNotFoundException,
};
```

### 3.4 Aliasing

```php
use App\Service\UserService as UserManager;
```

---

## 4. Name Resolution Rules

```
Unqualified:  UserService       → Current namespace prefix
Qualified:    Domain\User\...   → Current namespace + suffix
Fully Qualified: \App\Domain\... → Exactly as written
```

| Declaration | Current NS | Resolves To |
|-------------|-----------|-------------|
| `new Foo()` | `App` | `App\Foo` |
| `new \App\Foo()` | any | `\App\Foo` |
| `use App\Foo; new Foo()` | any | `\App\Foo` |

---

## 5. Autoloading

### 5.1 `spl_autoload_register`

```php
<?php
declare(strict_types=1);

spl_autoload_register(function (string $class): void {
    $prefix = 'App\\';
    $baseDir = __DIR__ . '/src/';
    if (strncmp($prefix, $class, strlen($prefix)) !== 0) {
        return;
    }
    $relativeClass = substr($class, strlen($prefix));
    $file = $baseDir . str_replace('\\', '/', $relativeClass) . '.php';
    if (file_exists($file)) {
        require $file;
    }
});
```

### 5.2 Composer Autoloading

```bash
composer install
```

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

$user = new \App\Entity\User();
```

---

## 6. PSR-4 Autoloading Standard

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

| Namespace Prefix | Base Directory |
|-----------------|---------------|
| `App\` | `src/` |
| `Vendor\Package\` | `vendor/package/src/` |

---

## 7. Composer Autoload Optimization

```bash
# Development
composer dump-autoload

# Classmap (production)
composer dump-autoload -o

# Authoritative (no filesystem fallback)
composer dump-autoload -a

# APCu cache (PHP-FPM)
composer dump-autoload --apcu
```

### Production Install

```bash
composer install \
    --no-dev \
    --optimize-autoloader \
    --classmap-authoritative \
    --no-interaction \
    --prefer-dist
```

---

## 8. Common Pitfalls

1. **Class Not Found**: Verify namespace matches directory path; run `composer dump-autoload`
2. **Wrong Backslash**: Use `\` not `/`
3. **Leading Backslash in `use`**: `use App\Foo;` not `use \App\Foo;`
4. **Namespace After Code**: Namespace must be the first statement after `<?php`
5. **Function Collisions**: Namespaced functions shadow globals

---

## References

- [PHP Manual — Namespaces](https://www.php.net/manual/en/language.namespaces.php)
- [PSR-4 Specification](https://www.php-fig.org/psr/psr-4/)
- [Composer Autoloading](https://getcomposer.org/doc/01-basic-usage.md#autoloading)