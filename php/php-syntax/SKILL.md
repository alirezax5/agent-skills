---
name: php-syntax
description: "PHP syntax reference covering all fundamental language constructs, tags, structure, namespaces, strict typing, and modern features."
---

# PHP Syntax Reference (8.4)

PHP syntax reference covering all fundamental language constructs, tags, structure, namespaces, strict typing, attributes, and modern features.

---

## 1. Overview & Purpose

This skill provides a comprehensive reference for PHP syntax as of PHP 8.4, including opening/closing tags, file structure, namespaces, use declarations, strict types, attributes, and fundamental language constructs. Use this as both a learning guide and a day-to-day reference when writing PHP code.

PHP is a server-side scripting language whose syntax is inspired by C, Java, and Perl. It supports both procedural and object-oriented paradigms.

---

## 2. PHP Tags

PHP code is delimited by special opening and closing tags that allow entering and exiting "PHP mode."

### Standard Tags (Always Available)

```php
<?php
// PHP code here
?>
```

### Short Echo Tag (Always Available)

```php
<?= $variable ?>
// Equivalent to <?php echo $variable; ?>
```

### Short Tags (Deprecated, removed in 8.4)

```php
<? /* Deprecated — do not use */ ?>
```

> **⚠️ PHP 8.4:** Short open tags `<?` have been **deprecated and removed**. Only `<?php` and `<?=` are valid. Set `short_open_tag = Off` in php.ini.

### ASP Tags (Removed in PHP 7.0)

```php
<% /* Removed */ %>
<%= /* Removed */ %>
```

### Standalone Files

For pure PHP files, **omit the closing `?>` tag** to prevent accidental whitespace output:

```php
<?php
// Good — no trailing ?>
```

---

## 3. File Structure & Encoding

### Basic File Template (strict_types)

```php
<?php

declare(strict_types=1);

namespace App\Service;

use App\Contract\SomeInterface;
use App\Trait\SomeTrait;

/**
 * Class documentation.
 */
final readonly class SomeService implements SomeInterface
{
    use SomeTrait;

    public function __construct(
        private string $name,
    ) {}

    public function greet(): string
    {
        return "Hello, {$this->name}";
    }
}
```

### Encoding

- PHP files should be saved as **UTF-8 without BOM**.
- The BOM (Byte Order Mark) `\xEF\xBB\xBF` causes output before headers, breaking HTTP responses.
- All string and character encoding in PHP 8+ uses Unicode where possible.

### File naming conventions

- Classes: PascalCase → `MyClassName.php`
- Interfaces: PascalCase → `MyInterface.php`
- Traits: PascalCase → `MyTrait.php`
- Enums: PascalCase → `MyEnum.php`
- Functions: snake_case or camelCase (PSR-12 recommends camelCase)
- Configuration: lowercase with underscores

---

## 4. Namespaces

Namespaces encapsulate classes, interfaces, functions, and constants to prevent name collisions.

### Declaration

```php
<?php

namespace App\Http\Controller;
// Must be the first statement after `declare()`
```

### Sub-namespaces

```php
namespace App\Http\Controller\Admin;
```

### Multiple namespaces in one file (discouraged)

```php
<?php

namespace Foo {
    class Bar {}
}

namespace Baz {
    class Qux {}
}
```

### Global namespace

```php
namespace {
    // Global namespace code
}
```

```php
// Referencing global class from within a namespace
$std = new \stdClass();
```

### `__NAMESPACE__` constant

```php
echo __NAMESPACE__; // "App\Http\Controller"
```

---

## 5. Use Declarations

Import classes, functions, constants, and enums from other namespaces.

### Import a class

```php
use App\Entity\User;
```

### Import with alias

```php
use App\Entity\User as UserEntity;
```

### Import a function

```php
use function App\Helpers\formatDate;
```

### Import a constant

```php
use const App\Config\MAX_RETRIES;
```

### Multiple imports, grouped (PSR-12 recommends one per line)

```php
use App\Entity\{User, Post, Comment};
// Equivalent to three separate use statements
```

### Class, function, and constant imports together

```php
use App\Helpers\{Utility, function formatDate, const VERSION};
```

### Rules

- One `use` declaration per line per PSR-12
- Place after namespace declaration
- No leading backslash on imported names
- Import resolution is compile-time

---

## 6. Strict Types

`declare(strict_types=1)` enables strict type checking for **function/method calls** within that file only.

```php
<?php

declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

add(1, 2);   // OK: 3
add("1", 2); // TypeError: add(): Argument #1 ($a) must be of type int, string given
```

### How it works

- **`strict_types=0` (default):** PHP coerces scalars automatically ("type juggling")
- **`strict_types=1`:** PHP raises `\TypeError` on type mismatch at call time
- **Per-file scope:** Only the calling file's setting matters, not the callee's

### Decision tree

```
Is declare(strict_types=1) in the CALLER's file?
├── YES → Strict mode: TypeError on type mismatch
└── NO  → Coercive mode: PHP auto-converts scalars
          ├── string "123" → int 123
          ├── int 1 → string "1"
          └── int 0 → false (bool) — careful!
```

> **⚠️ Pitfall:** If `caller.php` has `strict_types=1` and `callee.php` does not, a `string` passed to an `int` parameter still throws because the caller's setting controls.

---

## 7. Attributes (PHP 8.0–8.4)

Attributes provide structured metadata for classes, methods, properties, constants, closures, parameters, and enum cases.

### Syntax

```php
<?php

#[Route('/api/users', methods: ['GET'])]
#[Middleware('auth')]
class UserController
{
    #[Column(type: 'integer')]
    private int $id;

    #[Assert\NotBlank]
    #[Assert\Length(min: 3, max: 255)]
    public string $name;

    #[\Override]
    public function execute(): void
    {
        // ...
    }
}
```

### Target types

- Classes: `#[Attribute]`
- Methods: `#[Override]`
- Properties: `#[Column]`
- Parameters: `#[MapQuery]` string `$name`
- Constants (PHP 8.3+): `#[Deprecated]`
- Closures (PHP 8.3+): `#[Attribute]`
- Enum cases: `#[Animal]`

### Built-in attributes

| Attribute | Since | Description |
|-----------|-------|-------------|
| `#[Attribute]` | 8.0 | Marks a class as an attribute |
| `#[\Override]` | 8.3 | Method must override a parent or interface method |
| `#[\Deprecated]` | 8.4 | Marks entity as deprecated (emits deprecation notice) |
| `#[\ReturnTypeWillChange]` | 8.1 | Suppress deprecation for absent return type (temporary) |
| `#[\SensitiveParameter]` | 8.2 | Redacts parameter value in stack traces |
| `#[\AllowDynamicProperties]` | 8.2 | Allows dynamic properties on class (suppresses deprecation) |

### Creating a custom attribute

```php
<?php

#[Attribute(Attribute::TARGET_CLASS | Attribute::TARGET_METHOD)]
class Route
{
    public function __construct(
        public readonly string $path,
        public readonly array $methods = ['GET'],
    ) {}
}
```

### Reading attributes via Reflection

```php
$reflection = new ReflectionClass(UserController::class);
$attributes = $reflection->getAttributes(Route::class);
$route = $attributes[0]->newInstance();
echo $route->path; // '/api/users'
```

---

## 8. Comments

### Single-line

```php
// This is a comment
# This is also a comment (discouraged per PSR-12)
```

### Multi-line

```php
/*
 * This is a multi-line comment.
 * PSR-12 recommends the left-aligned asterisk style.
 */
```

### PHPDoc / DocBlock

```php
/**
 * Calculate the total price including tax.
 *
 * @param float $price The base price
 * @param float $taxRate The tax rate (0.0 – 1.0)
 * @return float The total price
 *
 * @throws \InvalidArgumentException If price is negative
 */
function calculateTotal(float $price, float $taxRate): float
{
    // ...
}
```

---

## 9. Heredoc & Nowdoc

### Heredoc (variable interpolation)

```php
$name = 'Alice';
$html = <<<HTML
    <div>
        <p>Hello, $name!</p>
        <p>Today is {date('Y-m-d')}.</p>
    </div>
    HTML;

// Closing identifier must NOT be indented (before PHP 7.3)
// PHP 7.3+ allows indented closing identifier with flexible heredoc
```

### Nowdoc (no interpolation — like single-quoted)

```php
$sql = <<<'SQL'
    SELECT u.id, u.name
    FROM users u
    WHERE u.active = 1
    SQL;
```

### PHP 7.3+: Flexible Heredoc/Nowdoc

Closing identifier can be indented, and leading whitespace is stripped:

```php
if ($condition) {
    $json = <<<'JSON'
        {
            "name": "Alice",
            "role": "admin"
        }
        JSON;  // Indented closing marker is allowed
}
```

---

## 10. Keywords & Reserved Words

PHP reserves a set of keywords for its syntax. They cannot be used as identifiers for classes, functions, constants, etc.

### Reserved keywords (full partial list — see PHP manual for complete)

| Category | Keywords |
|----------|----------|
| Class-related | `class`, `interface`, `trait`, `enum`, `extends`, `implements`, `abstract`, `final`, `readonly` |
| Visibility | `public`, `protected`, `private`, `var` (deprecated) |
| Static/Scope | `static`, `self`, `parent`, `new`, `clone`, `instanceof` |
| Control flow | `if`, `else`, `elseif`, `switch`, `match`, `for`, `foreach`, `while`, `do`, `break`, `continue` |
| Return/Error | `return`, `throw`, `try`, `catch`, `finally`, `die`, `exit` |
| Type system | `int`, `float`, `bool`, `string`, `void`, `never`, `mixed`, `null`, `true`, `false`, `array`, `object`, `callable`, `iterable`, `resource` (soft) |
| Misc | `declare`, `include`, `require`, `include_once`, `require_once`, `goto`, `yield`, `from`, `fn`, `match`, `namespace`, `use`, `const`, `function`, `and`, `or`, `xor`, `print`, `echo`, `list`, `unset`, `isset`, `empty`, `eval`, `global`, `as` |

### Soft keywords

Some keywords are context-dependent and may be used as identifiers in some contexts:

- `readonly` — valid as function name but not class name (as of 8.4, restricted)
- `enum` — cannot be used as class/interface/trait name
- `false`, `null`, `true` — already reserved in PHP 8.0+

### Reserved for future use

`int`, `float`, `bool`, `string`, `true`, `false`, `null` cannot be used as class names (as of PHP 8.0+).

---

## 11. Literals & Constants

### Integer literals

```php
42          // decimal
0b101010    // binary
0o52        // octal (PHP 8.1+)
052         // octal (before 8.1 — ambiguous)
0x2A        // hexadecimal
1_000_000   // numeric separator (PHP 7.4+)
```

### Float literals

```php
3.14
0.314e1
314e-2
```

### String literals

```php
'single quoted'           // no variable interpolation
"double quoted $var"      // variable interpolation
"curly syntax: {$var}"    // explicit variable interpolation
```

### Array literals

```php
$array = [1, 2, 3];           // short syntax
$array = array(1, 2, 3);      // long syntax (discouraged)
$map = ['key' => 'value'];    // associative
```

### Magic constants

| Constant | Description |
|----------|-------------|
| `__LINE__` | Current line number |
| `__FILE__` | Full path and filename |
| `__DIR__` | Directory of the file |
| `__FUNCTION__` | Current function name |
| `__CLASS__` | Current class name (with namespace) |
| `__METHOD__` | Current class method name |
| `__NAMESPACE__` | Current namespace |
| `__TRAIT__` | Current trait name |
| `__PROPERTY__` (PHP 8.4+) | Current property being written in `__set` |

### Predefined constants

```php
PHP_VERSION       // e.g., "8.4.0"
PHP_INT_MAX       // 9223372036854775807 (64-bit)
PHP_INT_MIN       // -9223372036854775808
PHP_INT_SIZE      // 8
PHP_FLOAT_MAX     // 1.7976931348623E+308
PHP_FLOAT_EPSILON // 2.2204460492503E-16
PHP_EOL           // "\n" on Linux, "\r\n" on Windows
PHP_OS            // Operating system
```

---

## 12. Expressions & Statements

### Expression

Anything that has a value:

```php
$a = 5;          // Assignment expression
$b = $a + 3;     // Arithmetic expression
$c = $a > 0;     // Boolean expression
$d = match(true) { ... }; // Match expression
$result = fn($x) => $x * 2; // Arrow function expression
```

### Statements

Statements are units that perform an action:

```php
// Expression statement
echo "Hello";

// Control flow
if ($a > $b) {
    return $a;
}

// Loop
foreach ($items as $item) {
    process($item);
}

// Declaration
class Foo {}

// Return
return $value;
```

### Declare statement

```php
declare(strict_types=1);
declare(ticks=1);
declare(encoding='UTF-8');
```

Block form:

```php
declare(ticks=1):
    // code here
enddeclare;
```

---

## 13. Troubleshooting & Common Pitfalls

### Pitfall: Unexpected whitespace output

```php
<?php
// everything before <?php is output!
?>

<?php
// Trailing whitespace after ?> is output!
```

**Fix:** Omit `?>` in pure PHP files.

### Pitfall: strict_types scope confusion

```php
// caller.php — strict
declare(strict_types=1);
require 'callee.php';
foo('string'); // TypeError even if callee.php is not strict
```

**Fix:** Place `declare(strict_types=1)` at the top of every file where you want strict checking.

### Pitfall: BOM in files

UTF-8 BOM at file start causes "headers already sent" errors.

**Fix:** Save files as UTF-8 without BOM. Check with `hexdump` or `xxd`.

### Pitfall: Namespace + use confusion

```php
namespace App\Service;

use App\Entity\User; // Correct — no leading backslash
use \App\Entity\User; // Syntax error in PHP 8+
```

### Pitfall: Heredoc closing identifier

```php
$text = <<<EOT
Hello
  EOT; // Error: closing marker must be at line start (pre-7.3)
```

**Fix (PHP 7.3+):** Indented closers are valid. Pre-7.3: no whitespace before closer.

### Pitfall: Reserved words as identifiers

```php
class int {} // Fatal error in PHP 8.0+
```

**Fix:** Use descriptive alternatives: `class IntegerValue {}`

### PHP 8.4 specific deprecations

- `#` comments inside PHPDoc tags are deprecated
- Passing `null` to non-nullable parameter without default is deprecated
- `E_STRICT` constant deprecated
- Implicitly nullable parameter types like `function foo(?Foo $f = null)` should use explicit `?Foo`

---

> **Reference:** [PHP Manual — Language Reference](https://www.php.net/manual/en/langref.php) | Version: 8.4 | Updated: 2026
