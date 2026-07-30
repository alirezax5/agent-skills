---
name: php-functions
description: "Complete reference for PHP functions: user-defined functions, parameters, return types, arrow functions, closures, generators, variadic functions, named arguments, first-class callables, and function internals."
---

# PHP Functions Reference (8.4)

Complete reference for PHP functions: user-defined functions, parameters, return types, arrow functions, closures, generators, variadic functions, named arguments, first-class callables, and function internals.

---

## 1. Overview & Purpose

Functions are the fundamental unit of behavior in PHP. They encapsulate reusable logic, accept parameters, return values, and can be passed around as first-class citizens. PHP supports both procedural functions and anonymous functions (closures/arrow functions) with lexical scoping.

---

## 2. User-Defined Functions

### Basic function declaration

```php
<?php

declare(strict_types=1);

function greet(string $name): string
{
    return "Hello, {$name}!";
}

echo greet('Alice'); // "Hello, Alice!"
```

### Function naming rules

- Must start with a letter or underscore
- Followed by any number of letters, digits, or underscores
- Function names are **case-insensitive** in PHP (but always call as declared)
- Should follow PSR-12 naming: `camelCase` for functions

### Function scope

```php
// Functions are globally scoped (can be called before declaration)
foo(); // Works — functions are hoisted

function foo(): void
{
    echo 'Foo called';
}
```

### Conditional function declaration

```php
// Functions can be defined conditionally (not recommended)
if (true) {
    function conditional(): void
    {
        echo 'Conditional';
    }
}
// But cannot be re-declared at runtime — causes Fatal Error
```

---

## 3. Function Parameters

### Required parameters

```php
function createUser(string $name, string $email): User
{
    // Both parameters are required
}
```

### Default parameters

```php
function createUser(
    string $name,
    string $email,
    bool $isAdmin = false,      // Default value
    string $locale = 'en_US',
): User {
    // ...
}

createUser('Alice', 'alice@example.com'); // isAdmin defaults to false
```

### Default value rules

- Defaults must be constant expressions (not variables or function calls)
- Required parameters must precede optional ones

```php
// ❌ Invalid
function bad(string $name = 'Guest', string $email): void {}

// ✅ Valid
function good(string $name, string $email = null): void {}
```

### Nullable defaults & implicit nullable deprecation

```php
// PHP 8.4 — explicitly nullable
function foo(?string $name = null): void {} // ✅

// PHP 8.4 — DEPRECATED implicit nullable
function bar(string $name = null): void {} // ❌ Deprecated
```

---

## 4. Variadic Parameters

Accept a variable number of arguments via the spread (`...`) operator.

```php
function sum(int ...$numbers): int
{
    return array_sum($numbers);
}

echo sum(1, 2, 3, 4, 5); // 15
```

### Variadic with typed parameters

```php
function concatenate(string $separator, string ...$parts): string
{
    return implode($separator, $parts);
}

echo concatenate(', ', 'apple', 'banana', 'cherry');
// "apple, banana, cherry"
```

### Variadic with union types

```php
function process(mixed ...$args): void
{
    foreach ($args as $arg) {
        echo get_debug_type($arg) . ': ' . json_encode($arg) . PHP_EOL;
    }
}
```

### Passing arrays as individual arguments

```php
$numbers = [1, 2, 3, 4, 5];
echo sum(...$numbers); // Argument unpacking
```

---

## 5. Return Types

### Basic return type

```php
function add(int $a, int $b): int
{
    return $a + $b;
}
```

### Void return

```php
function log(string $message): void
{
    // No return statement needed
    // Cannot return a value
}
```

### Never return (PHP 8.1+)

```php
function redirect(string $url): never
{
    header("Location: $url");
    exit; // or exit() — function never returns to caller
}
```

### Nullable return

```php
function findUser(int $id): ?User
{
    return $this->repository->find($id); // User or null
}
```

### Union return types

```php
function parse(string $input): int|float|false
{
    if (is_numeric($input)) {
        return str_contains($input, '.') ? (float) $input : (int) $input;
    }
    return false;
}
```

### Intersection return types (PHP 8.1+)

```php
function getProcessable(): SomeInterface & AnotherInterface
{
    // Must return an object implementing both interfaces
}
```

---

## 6. Named Arguments (PHP 8.0+)

Pass arguments by parameter name instead of position.

```php
function createUser(
    string $name,
    string $email,
    bool $isAdmin = false,
    string $locale = 'en_US',
    ?array $roles = null,
): User
```

### Calling with named arguments

```php
createUser(
    name: 'Alice',
    email: 'alice@example.com',
    locale: 'de_DE',
    isAdmin: true,
);

// Skip default parameters
createUser(
    name: 'Bob',
    email: 'bob@example.com',
    // isAdmin defaults to false
    // locale defaults to 'en_US'
);
```

### Mixing positional and named

```php
// Positional first, then named
createUser('Alice', 'alice@example.com', locale: 'fr_FR');
```

### Benefits

- Self-documenting calls
- Skip optional parameters without passing `null`
- Reorder parameters (as long as required ones are covered)
- Better DX with IDE autocompletion

### Pitfalls

```php
// ❌ Positional after named — syntax error
createUser(name: 'Alice', 'alice@example.com');

// ❌ Cannot use named with variadic arguments before PHP 8.4
// PHP 8.4: named args work with variadics
```

### Named arguments with variadic (PHP 8.4 example)

```php
function render(string $template, mixed ...$data): string
{
    // ...
}

render('profile', title: 'User Profile', user: $user);
```

---

## 7. Arrow Functions (fn)

Short closure syntax (PHP 7.4+). Single-expression, automatically captures variables by-value.

```php
// Traditional closure
$double = function (int $x): int {
    return $x * 2;
};

// Arrow function
$double = fn(int $x): int => $x * 2;
```

### Auto-capture of outer scope

```php
$factor = 3;
$multiply = fn(int $x): int => $x * $factor;
// $factor is captured automatically — no need for "use"

echo $multiply(5); // 15
```

### Restrictions

| Feature | Arrow function | Closure |
|---------|---------------|---------|
| Body | Single expression only | Multi-statement |
| Return type | Automatically inferred (or explicit) | Optional |
| `use` by-reference | Not possible (always by-value) | Supports `&$var` |
| `$this` binding | Same as parent scope | Same as parent scope unless rebound |
| Recursion | Not possible | Possible |
| `yield` / generators | Not possible | Possible |

### Return type hint in arrow functions

```php
$safeDivide = fn(int $a, int $b): int|false => $b !== 0 ? intdiv($a, $b) : false;
```

---

## 8. Closures & Anonymous Functions

Full anonymous functions with multi-statement bodies.

```php
$greeter = function (string $name): string {
    $prefix = 'Hello';
    return "{$prefix}, {$name}!";
};

echo $greeter('Alice'); // "Hello, Alice!"
```

### Closures: capturing variables with `use`

```php
$prefix = 'Welcome';
$greeter = function (string $name) use ($prefix): string {
    return "{$prefix}, {$name}!";
};
```

### By-reference capture

```php
$counter = 0;
$increment = function () use (&$counter): void {
    $counter++;
};

$increment();
echo $counter; // 1
```

### Rebinding `$this`

```php
class Container {
    private string $value = 'from container';
}

$closure = function (): string {
    return $this->value;
};

$bound = $closure->bindTo(new Container());
echo $bound(); // "from container"
```

### Callable syntax for closures

```php
$closure = function (int $x): int { return $x * 2; };

$result = $closure(5);            // Direct call
$result = call_user_func($closure, 5); // Via call_user_func
$result = array_map($closure, [1, 2, 3]); // As callback
```

---

## 9. First-Class Callable Syntax (PHP 8.1+)

Create callable references without resorting to strings or closures.

```php
// Before (PHP 8.0)
$callback = [$this, 'method'];
$callback = 'strtolower';
$callback = fn(string $s) => MyClass::staticMethod($s);

// PHP 8.1+: First-class callable syntax
$callback = strtolower(...);
$callback = $this->method(...);
$callback = MyClass::staticMethod(...);
// These create a Closure object directly
```

### Examples

```php
// Array map with first-class callable
$strings = ['Hello', 'World'];
$lower = array_map(strtolower(...), $strings);

// Method reference
$processor = fn(string $s) => $this->process($s);
$processor = $this->process(...); // Same thing, cleaner
```

### Type information preserved

```php
$closure = strlen(...);
var_dump($closure); // class Closure { ... }
// IDE knows it expects string and returns int
```

---

## 10. Generators

Memory-efficient iteration using `yield`.

```php
function rangeGenerator(int $start, int $end): \Generator
{
    for ($i = $start; $i <= $end; $i++) {
        yield $i;
    }
}

foreach (rangeGenerator(1, 1_000_000) as $number) {
    // Processes one number at a time — no array in memory
}
```

### Key-value yielding

```php
function mapGenerator(array $items): \Generator
{
    foreach ($items as $key => $item) {
        yield $key => strtoupper($item);
    }
}
```

### Generator delegation with `yield from`

```php
function concat(string ...$parts): \Generator
{
    foreach ($parts as $part) {
        yield from str_split($part);
    }
}

foreach (concat('ab', 'cd') as $char) {
    echo $char; // a b c d
}
```

### Generator return values

```php
function counter(int $max): \Generator
{
    for ($i = 1; $i <= $max; $i++) {
        yield $i;
    }
    return 'done';
}

$gen = counter(3);
foreach ($gen as $value) {
    echo $value; // 1, 2, 3
}
echo $gen->getReturn(); // 'done'
```

### Sending values into generators

```php
function interact(): \Generator
{
    $name = yield "What is your name?";
    $age = yield "Hello, {$name}. How old are you?";
    yield "{$name} is {$age} years old.";
}

$gen = interact();
echo $gen->current(); // "What is your name?"
echo $gen->send('Alice'); // "Hello, Alice. How old are you?"
echo $gen->send(30); // "Alice is 30 years old."
```

---

## 11. Function Metadata & Reflection

### `func_get_args()`, `func_get_arg()`, `func_num_args()`

```php
function sum(): int
{
    $args = func_get_args(); // All passed arguments
    $count = func_num_args();
    return array_sum($args);
}

echo sum(1, 2, 3, 4, 5); // 15
```

> ⚠️ These cannot be used as function parameter type declarations. Prefer explicit variadic `...` instead.

### `get_defined_functions()`

```php
$functions = get_defined_functions();
$userFunctions = $functions['user'];    // User-defined
$internalFunctions = $functions['internal']; // Built-in
```

### ReflectionFunction

```php
$reflection = new ReflectionFunction('str_replace');
echo $reflection->getNumberOfParameters(); // 3
echo $reflection->getReturnType(); // string|string[]
```

### `is_callable()` and `function_exists()`

```php
var_dump(function_exists('array_map'));  // true
var_dump(is_callable('undefined_func')); // false
```

---

## 12. Built-in Function Categories

PHP provides thousands of built-in functions. Major categories:

| Category | Prefix Examples | Description |
|----------|----------------|-------------|
| String | `str_*`, `substr_*`, `trim`, `explode`, `implode` | String manipulation |
| Array | `array_*`, `sort`, `current`, `next`, `reset` | Array operations |
| DateTime | `date`, `time`, `strtotime`, `DateTime` class | Date/time handling |
| Filesystem | `file_*`, `fopen`, `fread`, `fwrite`, `mkdir` | File operations |
| Database | `pdo_*`, `mysqli_*` | Database access |
| JSON | `json_encode`, `json_decode` | JSON serialization |
| Math | `abs`, `round`, `ceil`, `floor`, `sqrt`, `sin` | Math functions |
| Network | `curl_*`, `stream_*`, `http_response_code` | Network communication |
| Output | `echo`, `print`, `printf`, `var_dump` | Output |
| Type | `is_*`, `gettype`, `get_debug_type`, `var_dump` | Type checking |
| Error | `trigger_error`, `set_error_handler`, `error_log` | Error handling |

### Notable PHP 8.x additions

```php
str_contains($haystack, $needle);     // PHP 8.0
str_starts_with($haystack, $needle);  // PHP 8.0
str_ends_with($haystack, $needle);    // PHP 8.0
array_is_list($array);                // PHP 8.1
fdiv(1.0, 0.0);                       // PHP 8.0 — returns INF/NAN, no error
```

---

## 13. Troubleshooting & Common Pitfalls

### Pitfall: Function re-declaration

```php
// ❌ Cannot redeclare function
if (false) {
    function foo(): void {}
}
foo(); // Fatal error: function foo not found
```

**Fix:** Use conditional loading sparingly, or wrap in namespace checks.

### Pitfall: Default parameter evaluation

```php
function process(array $items = []): void
{
    $items[] = 'new';
    // $items is always fresh — default is evaluated once at declaration
    // but the array is COW (copy-on-write)
}
```

### Pitfall: Argument unpacking with string keys

```php
function foo(int $a, int $b): void {}
$args = ['a' => 1, 'b' => 2];
foo(...$args); // ✅ Works (PHP 8.0+)

$args = ['x' => 1, 'y' => 2];
foo(...$args); // ❌ Unpacking with unknown keys throws
```

### Pitfall: Closures and `$this` scope

```php
class Foo {
    private string $value = 'secret';

    public function getPrinter(): Closure
    {
        return function () {
            return $this->value; // Binds to Foo automatically
        };
    }
}
```

**Fix:** If you need to detach, use `Closure::bind` or `Closure::bindTo`.

### Pitfall: By-reference side effects

```php
$items = [1, 2, 3];
$closure = function () use (&$items) {
    $items[] = 4;
};
$closure();
// $items now has 4 elements — side effect!
```

### Pitfall: Arrow functions can't modify outer scope

```php
$count = 0;
$increment = fn() => $count++; // ❌ Arrow functions cannot modify by-reference
```

**Fix:** Use a closure with `use (&$count)`.

### Pitfall: func_get_args() with type hints

```php
// ❌ func_get_args() doesn't check types
function sum(int ...$numbers): int {
    return array_sum($numbers); // ✅ Type-safe
}
```

### Performance comparison

```
Operation                | Relative cost
-------------------------|--------------
Function call overhead   | ~50ns (trivial)
Closure creation         | ~100ns
Arrow fn creation        | ~80ns (slightly lighter)
Variadic ...$args        | ~10ns overhead per call
Named arguments          | ~5ns overhead per call
ReflectionFunction       | ~500ns (cachable)
```

> **Reference:** [PHP Manual — Functions](https://www.php.net/manual/en/language.functions.php) | Version: 8.4 | Updated: 2026
