---
name: php-types
description: "Complete reference for PHP's type system including scalar types, compound types, union/intersection types, generics, type declarations, type coercion, and modern PHP 8.x additions."
---

# PHP Type System Reference (8.4)

Complete reference for PHP's type system including scalar types, compound types, union/intersection types, generics (PHPDoc), type declarations, type coercion, and modern PHP 8.x additions.

---

## 1. Overview & Purpose

PHP's type system has evolved dramatically from a "mostly dynamically typed" language to one with robust static typing support. PHP 8.4 strengthens this with property hooks, enhanced type safety, and more consistent union types. This skill covers everything from basic scalars to advanced type system features.

PHP uses a combination of:
- **Declaration types** — explicit type hints on parameters, return values, and properties
- **Runtime types** — the actual type of a value at runtime
- **Type coercion** — automatic conversion in coercive mode (default)
- **Variance** — covariance of return types, contravariance of parameter types

---

## 2. Scalar Types (Primitives)

### `int`

Signed integer, platform-dependent size (64-bit on most modern systems).

```php
<?php

declare(strict_types=1);

function increment(int $value): int
{
    return $value + 1;
}

$number = 42;           // decimal
$number = 0b101010;     // binary
$number = 0o52;         // octal (PHP 8.1+)
$number = 0x2A;         // hexadecimal
$number = 1_000_000;    // numeric separator
```

Range: `PHP_INT_MIN` (–9223372036854775808) to `PHP_INT_MAX` (9223372036854775807) on 64-bit.

### `float` (double)

Double-precision IEEE 754 floating point.

```php
$price = 19.99;
$scientific = 1.5e3;   // 1500.0
$tiny = 1.5e-3;        // 0.0015
```

> **⚠️ Pitfall:** Never compare floats for equality directly. Use epsilon comparison:
> ```php
> $epsilon = 1.0e-10;
> if (abs($a - $b) < $epsilon) { /* approximately equal */ }
> ```

### `bool`

```php
$isActive = true;
$isAdmin = false;
```

Logical values. When coerced to int: `true` → 1, `false` → 0.

### `string`

Sequence of bytes (binary-safe). PHP strings are not Unicode objects but can contain Unicode via UTF-8 encoding.

```php
$name = 'Alice';
$greeting = "Hello, $name!";
$heredoc = <<<TEXT
Multi-line string
TEXT;
$nowdoc = <<<'TEXT'
No interpolation here
TEXT;
```

---

## 3. Compound Types

### `array`

Ordered map that maps keys to values. PHP arrays are hybrid structures (hash table + ordered list).

```php
// Indexed array
$list = [1, 2, 3, 4, 5];

// Associative array
$map = [
    'name' => 'Alice',
    'age' => 30,
    'role' => 'admin',
];

// Mixed key types
$mixed = [
    0 => 'zero',
    'key' => 'value',
    1 => 'one',
];

// Array unpacking (PHP 7.4+)
$merged = [...$list1, ...$list2];
```

### `object`

An instance of a class. In PHP 8.4, objects can use property hooks for computed properties.

```php
$user = new User();
$user->name = 'Alice';

// Anonymous class
$obj = new class {
    public string $greeting = 'Hello';
};
```

### `callable`

Anything that can be called: function names, closure objects, invocable objects, static methods.

```php
$callable = 'strtolower';
$callable = fn(string $s) => strtoupper($s);
$callable = [MyClass::class, 'staticMethod'];
$callable = [$object, 'methodName'];
```

### `iterable`

Accepts any `array` or `Traversable` (implementations of `Iterator` or `IteratorAggregate`).

```php
function process(iterable $items): void
{
    foreach ($items as $item) {
        // ...
    }
}
```

---

## 4. Special Types

### `void`

No return value. Function body must not contain a `return` with a value.

```php
function log(string $message): void
{
    file_put_contents('log.txt', $message, FILE_APPEND);
    // No return statement needed (or allowed with value)
}
```

### `never`

Function never returns — either throws an exception or calls `exit()`/`die()`.

```php
function abort(string $message): never
{
    throw new \RuntimeException($message);
}

function redirect(string $url): never
{
    header("Location: $url");
    exit;
}
```

### `mixed`

Union of all possible types: `string|int|float|bool|null|array|object|callable|iterable|resource`.

```php
function dump(mixed $value): void
{
    var_dump($value);
}
```

### `null`

Represents a variable with no value. In strict mode, only `null` satisfies `?type` or `type|null`.

---

## 5. Union Types (PHP 8.0+)

Type declarations can combine multiple types with `|`.

```php
function parseId(int|string $id): User
{
    // $id can be int or string
}

function format(money): int|float
{
    return $money->amount;
}
```

### Nullable shorthand

```php
// These are equivalent:
function foo(?string $name): void {}
function foo(string|null $name): void {}
```

### `false` and `null` in unions

```php
// Common in legacy APIs
function findUser(int $id): User|null
{
    // returns User or null
}

function strpos(string $haystack, string $needle): int|false
{
    // returns int position or false
}
```

> ⚠️ `true` is also a valid type in union declarations since PHP 8.2.

### `true` and `false` as standalone types (PHP 8.2+)

```php
function isAdmin(): true
{
    return true; // must always return true
}
```

---

## 6. Intersection Types (PHP 8.1+)

Require a value to satisfy **all** types simultaneously (for interfaces/classes only).

```php
function process(NamedEntity & AgeAware $entity): void
{
    echo $entity->getName() . ' is ' . $entity->getAge();
}
```

> ⚠️ Intersection types **cannot** include scalars, `void`, `never`, `mixed`, `null`, `callable`, `iterable`, `object`, `resource`, or `array`.

---

## 7. DNF Types (Disjunctive Normal Form, PHP 8.2+)

Combine union and intersection types:

```php
function process(
    (NamedEntity & AgeAware) | (NamedEntity & RoleAware) $entity
): void
{
}
```

- Intersections are grouped in parentheses
- Unions separate the groups
- Allows complex type constraints while remaining parseable

---

## 8. Type Declarations (Typed Properties & Parameters)

### Property types (PHP 7.4+)

```php
class User
{
    public string $name;           // Required (uninitialized)
    public int $age = 0;           // Optional (with default)
    public ?string $email = null;  // Nullable
    public readonly string $uuid;  // Readonly (PHP 8.1+)
}
```

### PHP 8.4: Property Hooks

```php
class User
{
    public string $name {
        set(string $value) {
            if (strlen($value) === 0) {
                throw new \InvalidArgumentException('Name cannot be empty');
            }
            $this->name = $value;
        }
        get => $this->name;
    }

    public string $fullName {
        get => $this->firstName . ' ' . $this->lastName;
    }
}
```

### Parameter types

```php
function createUser(
    string $name,
    int $age = 18,
    ?string $email = null,
    string|int $id = 'temp',
): User
```

### Return types

```php
function getUser(int $id): ?User {}     // nullable return
function getUsers(): array {}           // array return
function first(): int|false {}          // union return
```

### Typed class constants (PHP 8.3+)

```php
class Foo
{
    const string NAME = 'Foo';
    final public const int VERSION = 1;
}
```

---

## 9. Type Coercion & Juggling

In coercive mode (`declare(strict_types=0)`, the default), PHP automatically converts between types.

### Coercion rules

| Context | Input | Coerced to | Example |
|---------|-------|-----------|---------|
| `(int)` argument | `"123"` | `int(123)` | `foo("123")` |
| `(string)` argument | `123` | `string("123")` | `foo(123)` |
| `(bool)` argument | `0` | `false` | `if ($num)` |
| `(float)` argument | `"1.5"` | `1.5` | `foo("1.5")` |
| `(int)` argument | `"hello"` | `int(0)` | — careful! |

### Explicit casting

```php
(int) $var;       // cast to int
(float) $var;     // cast to float
(string) $var;    // cast to string
(bool) $var;      // cast to bool
(array) $var;     // cast to array
(object) $var;    // cast to stdClass
(unset) $var;     // deprecated in 8.4 — results in null
```

### String-to-int edge cases (PHP 8.0+)

From PHP 8.0 onwards, string-to-number comparisons follow saner rules, but explicit cast is preferred:

```php
var_dump((int) "123abc");  // int(123) — leading digits extracted
var_dump((int) "abc");     // int(0)   — no leading digits
```

---

## 10. Type Variance

### Covariance (return types)

A method in a child class can return a **more specific type** than the parent.

```php
class Animal {}
class Dog extends Animal {}

class PetOwner {
    public function getPet(): Animal { return new Animal(); }
}
class DogOwner extends PetOwner {
    public function getPet(): Dog { return new Dog(); } // Covariant
}
```

### Contravariance (parameter types)

A method in a child class can accept a **more general type** than the parent.

```php
class PetOwner {
    public function feed(Dog $dog): void {}
}
class AnimalLover extends PetOwner {
    public function feed(Animal $animal): void {} // Contravariant
}
```

### Invariance (property types)

Property types in child classes must match exactly (cannot be covariant or contravariant) as of PHP 8.4 — **property types are invariant**.

---

## 11. Type Introspection & Checking

### `gettype()`

Returns the type as a string. Rarely useful in modern code — prefer `is_*` functions or `instanceof`.

```php
var_dump(gettype(42)); // "integer"
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
is_iterable($var);
is_callable($var);
is_numeric($var);   // int or float or numeric string
is_scalar($var);    // int, float, string, bool
```

### `instanceof` operator

```php
if ($user instanceof User) {
    // $user is a User
}

if ($object instanceof SomeInterface) {
    // $object implements SomeInterface
}

// PHP 8.0+: type alias via class-string
function create(string $class): object
{
    return new $class();
}
```

### `get_debug_type()` (PHP 8.0+)

Better error message output than `gettype()`:

```php
echo get_debug_type($var);
// "int" instead of "integer"
// "App\Entity\User" instead of "object"
```

---

## 12. Generics (PHPDoc Only — No Native Generics)

PHP does not have native runtime generics. Generics are provided via PHPDoc annotations for IDE/static analysis support.

### Template annotation

```php
/**
 * @template T
 */
class Collection
{
    /** @var array<int, T> */
    private array $items = [];

    /**
     * @param T $item
     */
    public function add(mixed $item): void
    {
        $this->items[] = $item;
    }

    /**
     * @return T
     */
    public function get(int $index): mixed
    {
        return $this->items[$index] ?? throw new \OutOfBoundsException();
    }
}
```

### Using generic collections

```php
/** @var Collection<User> $users */
$users = new Collection();
$users->add(new User());
$user = $users->get(0); // IDE knows this is User
```

### Template constraints

```php
/**
 * @template T of SomeInterface
 */
class Processor {}
```

### `@template-covariant`

```php
/**
 * @template-covariant T
 */
class ReadOnlyWrapper {}
```

---

## 13. Troubleshooting & Type System Pitfalls

### Pitfall: Implicit nullable parameter removed

```php
// PHP 7.1: Valid, ?string is explicit
function foo(?string $name = null): void {}

// PHP 8.4+: Deprecated implicit nullable
function bar(string $name = null): void {} // Deprecated!
```

**Fix:** Always use `?string` or `string|null` when you mean nullable.

### Pitfall: strict_types leaking

```php
// file_a.php
declare(strict_types=1);
require 'file_b.php';
foo('123'); // TypeError if foo expects int — caller's strict_types applies
```

### Pitfall: Mixed and void at same time

```php
function foo(): mixed|void {} // Syntax error — void cannot be in union
```

`void` cannot be part of a union type. Use `null` instead.

### Pitfall: never is not void

```php
function bar(): never {}
bar();
echo "Unreachable"; // Unreachable, but PHP doesn't detect statically
```

`never` implies termination (exit/throw/die). Code after a `never` call is dead but not checked.

### Pitfall: String-to-int coercion surprises

```php
// Coercive mode
$sum = 10 + "5 apples"; // 15 in PHP 8.0+ (was 15 before too, but E_NOTICE)
$sum = 10 + "apples";   // 10 (with PHP 8.0+ E_WARNING, before E_NOTICE)
```

**Fix:** Use strict_types=1 and explicit casting.

### Pitfall: Array keys lose type

```php
$arr = [0 => 'a', '0' => 'b'];
// Only key 'b' exists — PHP casts "0" to 0
```

### Decision tree: Which type declaration to use

```
Need a value or no value?
├── Function never returns?          → never
├── No meaningful return?            → void
├── Multiple types possible?         → union (int|string)
├── Multiple interfaces required?    → intersection (A & B)
├── Mixed DNF?                       → DNF ((A & B) | C)
├── Anything goes?                   → mixed
└── Exact type known?                → int|float|string|bool|array|object
```

> **Reference:** [PHP Manual — Types](https://www.php.net/manual/en/language.types.php) | Version: 8.4 | Updated: 2026
