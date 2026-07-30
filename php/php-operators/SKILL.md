---
name: php-operators
description: "Complete reference for all PHP operators: arithmetic, assignment, comparison, logical, bitwise, string, array, type, error control, execution, increment/decrement, nullsafe, and spaceship operators."
---

# PHP Operators Reference (8.4)

Complete reference for all PHP operators: arithmetic, assignment, comparison, logical, bitwise, string, array, type, error control, execution, increment/decrement, nullsafe, and spaceship operators.

---

## 1. Overview & Purpose

Operators in PHP produce a value from one or more operands. PHP supports a wide range of operators including C-like arithmetic and bitwise operators, unique operators like the spaceship `<=>`, null coalescing `??`, and the nullsafe `?->`. Understanding operator precedence and associativity is critical to writing correct, predictable code.

---

## 2. Operator Precedence & Associativity

Operators with higher precedence are evaluated first. Associativity determines evaluation order for operators of equal precedence.

### Full precedence table (highest to lowest)

| Precedence | Operators | Associativity |
|------------|-----------|---------------|
| 1 | `new`, `clone` | None |
| 2 | `**` | Right |
| 3 | `++`, `--`, `~`, `(int)`, `(float)`, `(string)`, `(bool)`, `(array)`, `(object)`, `@` | Right |
| 4 | `instanceof` | None |
| 5 | `!` | Right |
| 6 | `*`, `/`, `%` | Left |
| 7 | `+`, `-`, `.` | Left |
| 8 | `<<`, `>>` | Left |
| 9 | `<`, `<=`, `>`, `>=` | None |
| 10 | `==`, `!=`, `===`, `!==`, `<>`, `<=>` | None |
| 11 | `&` | Left |
| 12 | `^` | Left |
| 13 | `\|` | Left |
| 14 | `&&` | Left |
| 15 | `\|\|` | Left |
| 16 | `? :` (ternary) | Right |
| 17 | `??`, `?->` | Right |
| 18 | `=`, `+=`, `-=`, `*=`, `**=`, `/=`, `.=`, `%=`, `&=`, `\|=`, `^=`, `<<=`, `>>=`, `??=` | Right |
| 19 | `and`| `xor` | `or` | Left |
| 20 | `yield from`, `yield` | None |
| 21 | `=>` (in array) | None |

### Decision tree: When to use parentheses

```
Are you mixing operators from different precedence levels?
├── NO  → Probably fine without parens
└── YES → Do you know the precedence order by heart?
           ├── YES → Probably fine (but parens still improve readability)
           └── NO  → USE PARENTHESES
```

> **⚠️ Pitfall:** `&&`/`||` have **higher precedence** than `and`/`or`/`xor`. This leads to subtle bugs:
> ```php
> $result = true && false;  // $result = (true && false) → false
> $result = true and false; // $result = true; (then) and false
> ```

---

## 3. Arithmetic Operators

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `+` | Addition | `$a + $b` | Sum |
| `-` | Subtraction | `$a - $b` | Difference |
| `*` | Multiplication | `$a * $b` | Product |
| `/` | Division | `$a / $b` | Float quotient (always) |
| `%` | Modulo | `$a % $b` | Remainder (sign matches `$a`) |
| `**` | Exponentiation | `$a ** $b` | `$a` raised to power `$b` (right-associative) |

### Division always returns float

```php
var_dump(10 / 3);   // float(3.3333333333333)
var_dump(10 / 2);   // float(5) — note float, not int!
var_dump(0 / 0);    // float(NAN)
var_dump(1 / 0);    // DivisionByZeroError (PHP 8.0+) — not false/INF!
```

### Integer division

```php
intdiv(10, 3);      // int(3) — integer division
intdiv(10, 0);      // DivisionByZeroError
```

### Modulo with negative

```php
var_dump(-5 % 3);   // int(-2) — sign follows dividend
var_dump(5 % -3);   // int(2)  — divisor sign is ignored
```

### Fmod (float modulo)

```php
fmod(5.7, 1.3);     // float(0.5) — works with floats
```

### Exponentiation edge cases

```php
var_dump(2 ** 3 ** 2); // 512 — right-associative: 2 ** (3 ** 2)
```

---

## 4. Assignment Operators

| Operator | Example | Equivalent to |
|----------|---------|---------------|
| `=` | `$a = $b` | Assign `$b` to `$a` |
| `+=` | `$a += $b` | `$a = $a + $b` |
| `-=` | `$a -= $b` | `$a = $a - $b` |
| `*=` | `$a *= $b` | `$a = $a * $b` |
| `**=` | `$a **= $b` | `$a = $a ** $b` |
| `/=` | `$a /= $b` | `$a = $a / $b` |
| `.=` | `$a .= $b` | `$a = $a . $b` |
| `%=` | `$a %= $b` | `$a = $a % $b` |
| `&=` | `$a &= $b` | `$a = $a & $b` |
| `\|=` | `$a \|= $b` | `$a = $a \| $b` |
| `^=` | `$a ^= $b` | `$a = $a ^ $b` |
| `<<=` | `$a <<= $b` | `$a = $a << $b` |
| `>>=` | `$a >>= $b` | `$a = $a >> $b` |
| `??=` | `$a ??= $b` | `$a = $a ?? $b` |

### Null coalescing assignment (??=, PHP 7.4+)

```php
$data['key'] ??= 'default';
// Equivalent to: $data['key'] = $data['key'] ?? 'default';
// Only assigns if $data['key'] is null or not set
```

### Assignment by reference

```php
$a = 1;
$b = &$a;  // $b is a reference to $a
$a = 2;
echo $b;   // 2 — $b references the same variable
```

> **⚠️ Pitfall:** References can cause hard-to-track bugs. Use `unset($ref)` to break reference.

---

## 5. Bitwise Operators

Operate on the binary representation of integers (and only strings in specific contexts).

| Operator | Name | Example | Description |
|----------|------|---------|-------------|
| `&` | AND | `$a & $b` | Bits set in both |
| `\|` | OR (inclusive) | `$a \| $b` | Bits set in either |
| `^` | XOR (exclusive) | `$a ^ $b` | Bits set in one but not both |
| `~` | NOT | `~$a` | Flip all bits |
| `<<` | Shift left | `$a << $b` | Shift left by `$b` bits (multiply by `2^$b`) |
| `>>` | Shift right | `$a >> $b` | Shift right by `$b` bits (divide by `2^$b`) |

### Bitwise examples

```php
// Permission flags
const READ = 0b001;   // 1
const WRITE = 0b010;  // 2
const EXEC = 0b100;   // 4

$permissions = READ | WRITE;  // 0b011 (3)
var_dump($permissions & READ); // 1 (truthy) — has read
var_dump($permissions & EXEC); // 0 (falsy)  — no exec
```

### Shifting

```php
8 << 2;  // 32  (8 * 2^2)
8 >> 2;  // 2   (8 / 2^2)
```

> **⚠️ Pitfall:** Bitwise NOT on `int` includes the sign bit: `~5` on a 64-bit system is `-6`.

---

## 6. Comparison Operators

| Operator | Name | Example | True when |
|----------|------|---------|-----------|
| `==` | Equal | `$a == $b` | Type-juggled equal |
| `===` | Identical | `$a === $b` | Same value AND same type |
| `!=` | Not equal | `$a != $b` | Type-juggled not equal |
| `<>` | Not equal | `$a <> $b` | Same as `!=` |
| `!==` | Not identical | `$a !== $b` | Not same type or value |
| `<` | Less than | `$a < $b` | `$a` strictly less |
| `>` | Greater than | `$a > $b` | `$a` strictly greater |
| `<=` | Less than or equal | `$a <= $b` | `$a` ≤ `$b` |
| `>=` | Greater than or equal | `$a >= $b` | `$a` ≥ `$b` |
| `<=>` | Spaceship | `$a <=> $b` | `-1` if `$a < $b`, `0` if equal, `1` if `$a > $b` |

### String comparison quirks (PHP 8.0+)

PHP 8.0+ changed string-to-number comparison to be more predictable:

```php
var_dump('abc' == 0);   // false (PHP 8.0+, was true in PHP 7.x)
var_dump('123abc' == 123); // true (numeric string wins)
var_dump(0 == 'abc');   // false (PHP 8.0+)
```

### Strict comparison is safer

```php
// ALWAYS prefer === and !==
if ($result === false) { /* exact false */ }
if ($result !== null) { /* not null */ }
```

### Spaceship operator (<=>)

Returns `-1`, `0`, or `1`. Ideal for sort callbacks:

```php
usort($users, fn(User $a, User $b) => $a->age <=> $b->age);
usort($users, fn(User $a, User $b) => $a->name <=> $b->name);

// Multi-criteria sort
usort($users, fn(User $a, User $b) =>
    $a->age <=> $b->age
    ?: $a->name <=> $b->name
);
```

### String comparison with spaceship

```php
'apple' <=> 'banana'; // -1 (alphabetical)
'banana' <=> 'apple'; // 1
'apple' <=> 'apple';  // 0
```

---

## 7. Logical Operators

| Operator | Name | Example | True when |
|----------|------|---------|-----------|
| `and` | AND | `$a and $b` | Both true |
| `or` | OR | `$a or $b` | One or both true |
| `xor` | XOR | `$a xor $b` | Exactly one true |
| `!` | NOT | `!$a` | `$a` is falsy |
| `&&` | AND | `$a && $b` | Both true |
| `\|\|` | OR | `$a \|\| $b` | One or both true |

### Short-circuit evaluation

PHP short-circuits logical operations:

```php
// If connect() returns false, query() is never called
$result = connect() or throw new \RuntimeException('Failed');

// If is_array() returns false, count() is never called
if (is_array($data) && count($data) > 0) { /* safe */ }

// Null check short-circuit
if ($user !== null && $user->isActive()) { /* safe */ }
```

### Priority difference: &&/|| vs and/or

```php
$x = true;
$y = false;

$result = $x && $y;   // $result = false (&& has high precedence)
$result = $x and $y;  // $result = true — see below!

// Explanation:
$result = ($x && $y);    // $result = false
$result = ($x) and ($y); // $result = true; and evaluates after assignment
```

---

## 8. String Operators

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `.` | Concatenation | `$a . $b` | Joined strings |
| `.=` | Concatenating assignment | `$a .= $b` | Append `$b` to `$a` |

### Concatenation examples

```php
$greeting = 'Hello' . ', ' . 'World!'; // "Hello, World!"

$name = 'Alice';
$message = 'Welcome, ' . $name . '!';  // "Welcome, Alice!"

// Often clearer with double-quoted interpolation:
$message = "Welcome, {$name}!";
```

> **⚠️ Pitfall:** Concatenation in loops creates many intermediate strings. Use `implode()` for arrays or a `StringBuilder` pattern when building large strings.

---

## 9. Array Operators

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `+` | Union | `$a + $b` | Union: left keys take precedence |
| `==` | Equality | `$a == $b` | Same key/value pairs (type-juggled) |
| `===` | Identity | `$a === $b` | Same pairs, same order, same types |
| `!=` | Inequality | `$a != $b` | Not `==` |
| `<>` | Inequality | `$a <> $b` | Same as `!=` |
| `!==` | Non-identity | `$a !== $b` | Not `===` |

### Array union (+) vs array_merge

```php
$a = ['a' => 1, 'b' => 2];
$b = ['b' => 3, 'c' => 4];

$union = $a + $b;
// ['a' => 1, 'b' => 2, 'c' => 4] — left wins on conflict

$merged = array_merge($a, $b);
// ['a' => 1, 'b' => 3, 'c' => 4] — later wins on conflict
```

### Array identity (===)

```php
$a = [1, 2, 3];
$b = [1, 2, 3];
var_dump($a === $b); // true — same values, same order, same types

$c = [1 => 1, 2 => 2]; // key order matters for ===
$d = [2 => 2, 1 => 1];
var_dump($c === $d);    // false — different key order
```

---

## 10. Type Operators

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `instanceof` | Type test | `$a instanceof MyClass` | `true` if `$a` is instance of class/interface/trait |

### instanceof examples

```php
if ($entity instanceof User) {
    // $entity is User
}

// Works with interfaces
if ($service instanceof LoggerInterface) {
    $service->log('...');
}

// Works with inheritance
if ($object instanceof ParentClass) {
    // also true for child classes
}

// instanceof with variable class name
$className = 'App\Entity\User';
if ($entity instanceof $className) {
    // dynamic class check
}
```

---

## 11. Error Control Operator (@)

Prefix an expression to suppress errors:

```php
$content = @file_get_contents('missing.txt');
// Returns false, no warning emitted
```

> **⚠️ Pitfall:**
> - `@` suppresses **all** errors for that expression
> - `@` sets error_reporting to 0 temporarily — can hide legitimate bugs
> - From PHP 8.0+, `@` no longer silences fatal errors
> - **Best practice:** Handle errors explicitly with try/catch or error checking instead of `@`

---

## 12. Nullsafe Operator (?->) (PHP 8.0+)

Short-circuit on `null` when accessing properties or methods:

```php
// Without nullsafe:
$country = null;
if ($user !== null) {
    $address = $user->getAddress();
    if ($address !== null) {
        $country = $address->getCountry();
    }
}

// With nullsafe:
$country = $user?->getAddress()?->getCountry();
// Returns null if any step is null
```

### Nullsafe with method calls

```php
// Only calls getAddress() if $user is not null
$address = $user?->getAddress();

// Works with static methods too
$result = $class?::method();
```

### Null coalescing (??) vs nullsafe

```php
// Null coalescing: fallback on null
$name = $data['name'] ?? 'default';

// Nullsafe chaining: short-circuit
$city = $user?->address?->city;

// Can combine both:
$city = $user?->address?->city ?? 'Unknown';
```

---

## 13. Troubleshooting & Common Pitfalls

### Pitfall: Precedence surprises with &&/|| vs and/or

```php
// Bug:
$result = checkAccess() or throw new \Exception('Denied');
// Only triggers if checkAccess() returns false in a weird way
// because PHP reads: ($result = checkAccess()) or throw ...

// Fix — use || for high precedence, or wrap:
$result = checkAccess() || throw new \Exception('Denied');
// Or use early return / guard clause
```

### Pitfall: String concatenation in loops

```php
// Slow O(n²):
$result = '';
for ($i = 0; $i < 10000; $i++) {
    $result .= $data[$i];
}

// Fast O(n):
$parts = [];
for ($i = 0; $i < 10000; $i++) {
    $parts[] = $data[$i];
}
$result = implode('', $parts);
```

### Pitfall: == vs === confusion

```php
var_dump(false == 0);    // true — type juggling!
var_dump(false === 0);   // false — type-safe

var_dump('' == false);   // true
var_dump('' === false);  // false

var_dump(null == false); // true
var_dump(null === false);// false
```

**Rule:** Always use `===` and `!==` except when you explicitly need type-juggling.

### Pitfall: Array + vs array_merge

```php
// + is not commutative!
$result = [1, 2] + [3, 4]; // [1, 2] — left wins on numeric keys too
```

### Pitfall: Integer overflow

```php
$large = PHP_INT_MAX;       // 9223372036854775807
$overflow = $large + 1;     // float(9.223372036854776E+18) — precision loss!
```

**Fix:** Use `GMP` or `BCMath` for large integer arithmetic.

### Pitfall: Division by zero

```php
// PHP 8.0+ throws DivisionByZeroError for intdiv and / operator
// For float, it returns NAN or INF:
var_dump(0.0 / 0.0); // NAN
var_dump(1.0 / 0.0); // INF
```

### Pitfall: @ operator hiding real errors

```php
// Never do this:
@$result = riskyOperation();

// Instead:
try {
    $result = riskyOperation();
} catch (\Throwable $e) {
    // Handle gracefully
}
```

> **Reference:** [PHP Manual — Operators](https://www.php.net/manual/en/language.operators.php) | Version: 8.4 | Updated: 2026
