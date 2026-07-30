---
name: php-generators
description: PHP Generators — lazy iteration, yield/yield from, memory efficiency, two-way communication with send(), cooperative multitasking, and production patterns for PHP 8.4
tags: [php, generators, yield, lazy-iteration, memory-efficiency, coroutines, strict-types]
---

# PHP Generators

> **PHP 8.4** · `declare(strict_types=1)` · Lazy iteration, memory efficiency, coroutines

---

## 1. Overview

Generators provide an easy way to implement iterators without the overhead of implementing the `Iterator` interface. A generator function uses `yield` to return a stream of values one at a time. Generators are lazy — they compute values on demand and consume O(1) memory.

### Key Characteristics

| Feature | Description |
|---------|-------------|
| **Lazy evaluation** | Values produced only when requested |
| **Low memory** | Only current value + state kept in memory |
| **Pause/resume** | Execution suspended at `yield`, resumed on `next()` |
| **One-direction** | Values flow from generator ← consumer (unless using send()) |

---

## 2. Basic Generator

```php
<?php
declare(strict_types=1);

function rangeGenerator(int $start, int $end, int $step = 1): \Generator
{
    for ($i = $start; $i <= $end; $i += $step) {
        yield $i;
    }
}

foreach (rangeGenerator(1, 5) as $value) {
    echo $value . "\n";
}
// Output: 1 2 3 4 5
```

### 2.1 Memory Efficiency

```
                    Array                          Generator
Memory (1M items)   ~80 MB                         ~0.001 MB
Time to create      20-50 ms                       ~0.001 ms
```

---

## 3. The `yield` Keyword

### 3.1 Yielding Key/Value Pairs

```php
<?php
declare(strict_types=1);

function configValues(): \Generator
{
    yield 'db.host' => 'localhost';
    yield 'db.port' => 5432;
    yield 'app.debug' => true;
}
```

### 3.2 Generator Delegation (`yield from`)

```php
<?php
declare(strict_types=1);

function inner(): \Generator
{
    yield 2;
    yield 3;
}

function outer(): \Generator
{
    yield 1;
    yield from inner();
    yield 4;
}

foreach (outer() as $value) {
    echo $value . ' ';
}
// Output: 1 2 3 4
```

### 3.3 `yield from` with Return Values (PHP 7.0+)

```php
<?php
declare(strict_types=1);

function subGen(): \Generator
{
    yield 1;
    yield 2;
    return 'done';
}

function mainGen(): \Generator
{
    $result = yield from subGen();
    yield "subGen returned: $result";
}
```

---

## 4. Generator Methods

```php
<?php
declare(strict_types=1);

$gen = (function () {
    yield 1;
    yield 2;
    yield 3;
    return 'final';
})();

// Iterator interface
$gen->rewind();
$gen->valid();
$gen->current();
$gen->key();
$gen->next();

// Generator-specific
$gen->send($value);        // Two-way communication
$gen->throw($exception);   // Throw exception at yield point
$gen->getReturn();         // Get return value after iteration
```

---

## 5. Two-Way Communication with `send()`

```php
<?php
declare(strict_types=1);

function echoGenerator(): \Generator
{
    $received = yield 'ready';
    echo "Received: $received\n";
    $received = yield 'again';
    echo "Received: $received\n";
    return 'done';
}

$gen = echoGenerator();
$gen->rewind();
$value = $gen->current();   // "ready"
$gen->send('hello');         // "Received: hello"
$value2 = $gen->current();  // "again"
$gen->send('world');         // "Received: world"
echo $gen->getReturn();      // "done"
```

---

## 6. Infinite Generators

```php
<?php
declare(strict_types=1);

function fibonacci(): \Generator
{
    $a = 0; $b = 1;
    while (true) {
        yield $a;
        [$a, $b] = [$b, $a + $b];
    }
}

$fib = fibonacci();
foreach ($fib as $i => $value) {
    if ($i > 20) break;
    echo "$value ";
}
```

---

## 7. Real-World: CSV Reader

```php
<?php
declare(strict_types=1);

function readCsv(string $path, string $separator = ','): \Generator
{
    $handle = fopen($path, 'r');
    if ($handle === false) {
        throw new \RuntimeException("Cannot open $path");
    }

    $headers = fgetcsv($handle, 0, $separator);
    if ($headers === false || $headers === null) {
        fclose($handle);
        return;
    }
    yield $headers;

    try {
        while (($row = fgetcsv($handle, 0, $separator)) !== false) {
            yield count($row) === count($headers)
                ? array_combine($headers, $row)
                : $row;
        }
    } finally {
        fclose($handle);
    }
}
```

---

## 8. Common Pitfalls

1. **Generator Exhaustion**: Generators can only be iterated once — recreate for another pass
2. **Returning vs Yielding**: `yield [1,2,3]` yields one array; `yield from [1,2,3]` yields three values
3. **No `rewind()` Before `send()`**: Call `$gen->rewind()` before first `send()`
4. **Memory defeat**: `iterator_to_array(largeGenerator())` loads everything into memory

---

## Summary

| Feature | Syntax | PHP Version |
|---------|--------|-------------|
| Basic generator | `yield $value` | 5.5 |
| Key/value yield | `yield $key => $value` | 5.5 |
| Generator delegation | `yield from $iterable` | 7.0 |
| `getReturn()` | `$gen->getReturn()` | 7.0 |
| `send()` | `$received = yield;` | 5.5 |
| `throw()` | `$gen->throw($e)` | 5.5 |

---

## References

- [PHP Manual — Generators](https://www.php.net/manual/en/language.generators.php)