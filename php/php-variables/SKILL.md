---
name: php-variables
description: "Complete reference for PHP variables, constants, scope, references, variable variables, superglobals, type checking, and modern best practices."
---

# PHP Variables & Constants Reference (8.4)

Complete reference for PHP variables, constants, scope, references, variable variables, superglobals, type checking, and modern best practices.

---

## 1. Overview & Purpose

Variables in PHP are containers for storing data values. PHP is a dynamically typed language — variables can hold values of any type and change type over their lifetime (unless declared with a type in PHP 7.4+). Constants provide immutable named values. This skill covers variables, constants, scope rules, and all related constructs.

---

## 2. Variable Basics

### Naming rules

- Start with `$` followed by a letter or underscore
- Remaining characters: letters, digits, or underscores
- Case-sensitive (`$Name` ≠ `$name`)
- Must be a legal PHP identifier

```php
$name = 'Alice';     // Valid
$_count = 42;        // Valid (convention: "private-ish")
$user2 = 'Bob';      // Valid
$2user = 'Charlie';  // ❌ Invalid — starts with digit
$my-var = 'test';    // ❌ Invalid — hyphen not allowed
```

### Conventions (PSR-12)

- Variables: `$camelCase`
- Constants: `UPPER_SNAKE_CASE`
- We'll cover constants in detail later.

### Assignment

```php
$greeting = 'Hello';
$copy = $greeting;       // Copy (default — copy-on-write)
$reference = &$greeting; // Reference (alias)
```

### Variable lifecycle

```php
$var = 'initial';        // Created
echo $var;               // Accessed
unset($var);             // Destroyed
echo $var;               // Undefined — emits E_WARNING (PHP 8.0+)
```

### Undefined variable behavior

```php
// PHP 8.0+ emits E_WARNING for undefined variables
echo $undefined;  // Warning: Undefined variable $undefined
                  // (Was E_NOTICE in PHP 7.x)
```

---

## 3. Typed Properties (PHP 7.4+)

Variables inside classes can have explicit types.

```php
class User
{
    // Typed properties
    public string $name;
    protected int $age;
    private ?string $email = null;

    // Readonly (PHP 8.1+)
    public readonly string $uuid;

    // PHP 8.4: Property hooks
    public string $displayName {
        get => $this->displayName ?? $this->name;
        set (string $value) {
            if (strlen($value) === 0) {
                throw new \InvalidArgumentException();
            }
            $this->displayName = $value;
        }
    }
}
```

### Typed property rules

- Must be initialized before access (can have a default value)
- Uninitialized typed property access throws `\Error`
- Readonly can only be set once, typically in constructor
- Property hooks (8.4) can intercept get/set operations

```php
$user = new User();
$user->name = 'Alice';  // OK
echo $user->name;        // "Alice"
echo $user->age;         // Error: Typed property uninitialized
```

### Typed property defaults

```php
class Config
{
    public string $name = 'default';   // ✅ Initialized
    public string $path;               // ⚠️ Must be set before read
    public ?string $label = null;      // ✅ Nullable with default
    public readonly string $id;        // Set in constructor
}
```

---

## 4. Variable Scope

### Local scope

Variables defined inside a function are local to that function:

```php
function test(): void
{
    $local = 'inside';
    echo $local; // "inside"
}

test();
echo $local; // Warning: Undefined variable
```

### Global scope

Variables defined outside functions are global:

```php
$global = 'outside';

function test(): void
{
    // $global is NOT accessible here
    // echo $global; // Warning: Undefined
}
```

### The `global` keyword (avoid if possible)

```php
$counter = 0;

function increment(): void
{
    global $counter; // Pulls global into local scope
    $counter++;
}

increment();
echo $counter; // 1
```

> **⚠️ Pitfall:** `global` creates hidden dependencies. Prefer dependency injection:
> ```php
> function increment(int &$counter): void { $counter++; }
> ```

### Static variables

Persist between function calls:

```php
function counter(): int
{
    static $count = 0;
    return ++$count;
}

echo counter(); // 1
echo counter(); // 2
echo counter(); // 3
```

### Static variable with expression (PHP 8.3+)

```php
function getConfig(): array
{
    static $config = loadConfig(); // PHP 8.3+ allows expressions
    return $config;
}
```

---

## 5. Variable Variables ($$)

Use the value of one variable as the name of another:

```php
$name = 'greeting';
$greeting = 'Hello!';

echo $$name; // "Hello!" — evaluates $greeting
```

### Array access with variable variables

```php
$varName = 'options';
$$varName['key'] = 'value';
// Equivalent to: $options['key'] = 'value';
```

> **⚠️ Pitfall:** Variable variables make code hard to read and refactor. Use arrays or Data Transfer Objects instead:
> ```php
> // Instead of $${$prefix}Name, use:
> $data[$prefix . 'Name'];
> // Or better: typed objects
> ```

---

## 6. Superglobals

Built-in associative arrays accessible from any scope.

| Superglobal | Description |
|-------------|-------------|
| `$_SERVER` | Server and execution environment info |
| `$_GET` | URL query parameters |
| `$_POST` | HTTP POST body (form-encoded) |
| `$_REQUEST` | Combined GET/POST/COOKIE data |
| `$_FILES` | Uploaded files |
| `$_COOKIE` | HTTP Cookies |
| `$_SESSION` | Session variables |
| `$_ENV` | Environment variables |
| `$GLOBALS` | All global variables |

### $_SERVER (common keys)

```php
$_SERVER['REQUEST_METHOD'];  // 'GET', 'POST', etc.
$_SERVER['REQUEST_URI'];     // '/api/users?id=1'
$_SERVER['HTTP_HOST'];       // 'example.com'
$_SERVER['HTTP_REFERER'];    // Previous page URL
$_SERVER['REMOTE_ADDR'];     // Client IP
$_SERVER['SERVER_NAME'];     // Server hostname
$_SERVER['CONTENT_TYPE'];    // Request content type
$_SERVER['QUERY_STRING'];    // 'id=1'
```

### $_GET and $_POST

```php
// URL: /page.php?name=Alice&age=30
$name = $_GET['name'] ?? 'Guest'; // 'Alice'
$age = (int) ($_GET['age'] ?? 0); // 30

// Always sanitize/filter input
$safeName = filter_input(INPUT_GET, 'name', FILTER_SANITIZE_STRING);
```

### $_FILES structure

```php
// HTML: <input type="file" name="avatar">
$_FILES['avatar'] = [
    'name' => 'photo.jpg',
    'type' => 'image/jpeg',
    'size' => 12345,
    'tmp_name' => '/tmp/phpXXXXXX',
    'error' => UPLOAD_ERR_OK,
];
```

### $GLOBALS

```php
$myVar = 'value';
echo $GLOBALS['myVar']; // 'value' — accesses global scope
```

---

## 7. References

References allow two variables to point to the same value.

### Creating references

```php
$a = 1;
$b = &$a;   // $b is a reference to $a

$a = 2;
echo $b;    // 2 — $b reflects the change

$b = 3;
echo $a;    // 3 — change propagates
```

### Reference in functions

```php
// Pass by reference
function addItem(array &$list, mixed $item): void
{
    $list[] = $item;
}

$items = ['apple', 'banana'];
addItem($items, 'cherry');
// $items is now ['apple', 'banana', 'cherry']
```

### Return by reference

```php
class Container
{
    private array $data = [];

    public function &getData(): array
    {
        return $this->data;
    }
}

$container = new Container();
$ref = &$container->getData();
$ref['key'] = 'value';
// Modifies $container->data directly
```

### Unsetting references

```php
$a = 1;
$b = &$a;
unset($b);  // Breaks reference — $a remains
// $b is now undefined
```

### Reference counting & copy-on-write

```php
$a = 'long string';  // refcount=1
$b = $a;             // refcount=2 — still same memory
$b .= ' more';       // refcount drops to 1 for $a — copy happens
```

---

## 8. Constants

Named immutable values.

### Defining constants with `define()`

```php
define('MAX_RETRIES', 3);
define('API_VERSION', 'v2.1');
define('DB_CONFIG', [
    'host' => 'localhost',
    'port' => 3306,
]);
```

### Defining constants with `const`

```php
// At top level (outside class)
const PI = 3.14159;
const APP_NAME = 'MyApp';

// Inside a class
class Config
{
    public const VERSION = '1.0.0';
    protected const INTERNAL = 'secret';
    private const SALT = 'xyz';
}
```

### `const` vs `define()`

| Feature | `const` | `define()` |
|---------|---------|------------|
| Compile-time | ✅ | ❌ (runtime) |
| Class member | ✅ | ❌ |
| Scalar expressions | ✅ (PHP 5.6+) | ✅ |
| Arrays | ✅ (PHP 5.6+) | ✅ |
| Objects | ❌ | ❌ |
| Case-insensitive | ❌ | ✅ (3rd param) |
| Conditional definition | ❌ | ✅ |

### Typed constants (PHP 8.3+)

```php
class Api
{
    public const string BASE_URL = 'https://api.example.com';
    final public const int TIMEOUT = 30;
}

// Type is enforced
// Api::BASE_URL = 42; // TypeError
```

### Magic constants

| Constant | Description |
|----------|-------------|
| `__LINE__` | Current line number |
| `__FILE__` | Full file path |
| `__DIR__` | Directory of current file |
| `__FUNCTION__` | Current function name |
| `__CLASS__` | Current class (with namespace) |
| `__METHOD__` | Current class method |
| `__NAMESPACE__` | Current namespace |
| `__TRAIT__` | Current trait name |
| `__PROPERTY__` (PHP 8.4+) | Current property in `__set` hook |

### Class constant visibility (PHP 7.1+)

```php
class Status
{
    public const PUBLIC = 'public';
    protected const PROTECTED = 'protected';
    private const PRIVATE = 'private';
}
```

---

## 9. Variable Information & Type Checking

### `isset()` — Check if variable exists and is not null

```php
$var = null;
$name = 'Alice';

isset($name); // true — exists, not null
isset($var);  // false — exists but is null
isset($other); // false — doesn't exist

// Works on array keys and object properties
isset($array['key']);
isset($obj->prop);
```

### `empty()` — Check if variable is falsy or non-existent

```php
$zero = 0;
$empty = '';
$null = null;

empty($zero);  // true
empty($empty); // true
empty($null);  // true
empty($undef); // true (no warning)

// Equivalent to: !isset($var) || !$var
```

### `unset()` — Destroy variable

```php
$name = 'Alice';
unset($name);
isset($name); // false — variable gone
```

### `is_*` functions

```php
is_int($var);
is_float($var);
is_string($var);
is_bool($var);
is_array($var);
is_object($var);
is_null($var);
is_numeric($var); // true for int, float, or numeric strings
is_scalar($var);  // true for int, float, string, bool
```

### `get_debug_type()` (PHP 8.0+)

```php
get_debug_type(42);        // "int"
get_debug_type(new User()); // "App\Entity\User"
get_debug_type('hello');    // "string"

// Better than gettype() which returns "integer", "object", etc.
```

---

## 10. Type Juggling (Coercion)

### Automatic coercion in expressions

```php
$result = 10 + "5";     // 15 — string "5" cast to int
$result = 10 + "5.5";   // 15.5 — string cast to float
$result = "3" . 4;      // "34" — int cast to string
```

### Explicit casting

```php
(int) $var;        // Cast to int (integer)
(bool) $var;       // Cast to bool (boolean)
(float) $var;      // Cast to float (double, real)
(string) $var;     // Cast to string
(array) $var;      // Cast to array
(object) $var;     // Cast to stdClass
(unset) $var;      // Cast to null (deprecated in PHP 8.4)
```

### Falsy values (evaluate to false in boolean context)

```php
false          // boolean false
0              // integer zero
0.0            // float zero
''             // empty string
'0'            // string "0"
[]             // empty array
null           // null
SimpleXMLElement (empty) // special case
```

---

## 11. Predefined Variables & Constants

### PHP version & configuration

```php
PHP_VERSION;        // "8.4.0"
PHP_VERSION_ID;     // 80400
PHP_OS;             // "Linux", "WINNT", etc.
PHP_INT_MAX;        // 9223372036854775807
PHP_INT_MIN;        // -9223372036854775808
PHP_INT_SIZE;       // 8
PHP_FLOAT_MAX;      // 1.7976931348623E+308
PHP_EOL;            // "\n" or "\r\n"
PHP_SAPI;           // "cli", "fpm-fcgi", etc.
```

### Environment variables

```php
// Access with getenv() or $_ENV
$dbHost = getenv('DB_HOST') ?: 'localhost';
$appEnv = $_ENV['APP_ENV'] ?? 'prod';

// Setting environment
putenv('APP_ENV=production');
```

---

## 12. Memory & Performance

### Variable size

```php
// Integers are 8 bytes on 64-bit
// Strings are 8 bytes + string length + overhead (~40 bytes)
// Arrays are ~56 bytes + elements * (key_size + value_size + overhead)
```

### Copy-on-write (COW)

PHP uses COW to optimize memory:

```php
$big = str_repeat('x', 1000000); // 1MB string
$copy = $big;                     // Memory isn't duplicated
$copy .= 'y';                     // Now copy happens (1MB + 1 byte)
```

### Memory limit

```php
ini_get('memory_limit');      // "128M"
memory_get_usage();           // Current usage in bytes
memory_get_peak_usage(true);  // Peak usage (including system)
```

### Garbage collection

```php
gc_enable();         // Enable cyclic reference GC
gc_collect_cycles(); // Force collection
gc_status();         // PHP 7.3+: array with GC stats
```

---

## 13. Troubleshooting & Common Pitfalls

### Pitfall: Undefined variable access

```php
// PHP 8.0+ emits E_WARNING
echo $undefined; // Warning: Undefined variable
```

**Fix:** Always initialize variables: `$var = null;` or use the null coalescing operator `??`.

### Pitfall: Reference confusion

```php
$a = [1];
$b = &$a[0];
$a[0] = 2;
echo $b; // 2 — reference still points to the same value

// But:
unset($a);
echo $b; // 2 — $b still exists (unset only removes the name)
```

### Pitfall: Global scope not accessible in functions

```php
$config = ['debug' => true];

function isDebug(): bool
{
    return $config['debug']; // ❌ Warning: Undefined variable
}
```

**Fix:** Pass explicitly, use a container, or use `global` (last resort).

### Pitfall: Constants and class names

```php
define('User', 'some value');
new User(); // ❌ Parse error: User is a constant, use \User
```

**Fix:** Avoid constants with class names.

### Pitfall: isset() vs empty()

```php
$count = 0;

if (isset($count))  { /* true — $count is set */ }
if (empty($count))  { /* true — 0 is empty */ }
if ($count)         { /* false — 0 is falsy */ }
```

### Pitfall: Variable variables with superglobals

```php
// Does NOT work
$name = '_SERVER';
$$name['HTTP_HOST']; // ❌ Not parsed as expected

// Use $GLOBALS instead
$GLOBALS['_SERVER']['HTTP_HOST'];
```

### Pitfall: PHP 8.4 deprecations

```php
// Deprecated: implicit nullable
function foo(string $name = null): void {}

// Deprecated: (unset) cast
$result = (unset) $value;

// Deprecated: E_STRICT constant
```

### Decision tree: Which variable tool to use

```
Need to check if a variable exists?
├── isset($var) — exists and not null
├── empty($var) — exists but is falsy, or doesn't exist
└── array_key_exists($key, $arr) — array key exists (even if null)

Need to handle possibly missing variables?
├── $var ?? 'default' — null coalescing
├── $var ??= 'default' — null coalescing assignment (PHP 7.4+)
└── match(true) { ... } — for complex conditions

Need immutable values?
├── define('NAME', value) — global constant
├── const NAME = value — class constant or compile-time
└── readonly properties — per-instance immutability
```

> **Reference:** [PHP Manual — Variables](https://www.php.net/manual/en/language.variables.php) | [PHP Manual — Constants](https://www.php.net/manual/en/language.constants.php) | Version: 8.4 | Updated: 2026
