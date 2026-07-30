---
name: php-fibers
description: PHP Fibers — cooperative multitasking, suspend/resume, async I/O patterns, fiber scheduler, fiber vs generator comparison for PHP 8.1+
tags: [php, fibers, cooperative-multitasking, async, suspend, resume, strict-types]
---

# PHP Fibers

> **PHP 8.1+** · `declare(strict_types=1)` · Cooperative multitasking, async I/O patterns, green threads

---

## 1. Overview

Fibers (introduced in PHP 8.1) are primitives for cooperative multitasking. Unlike traditional callbacks or generators, fibers allow a function to be paused (suspended) and resumed at any point in its call stack, enabling non-blocking I/O patterns without the overhead of OS threads.

### Key Concepts

| Term | Description |
|------|-------------|
| `Fiber` | A suspendable execution unit |
| `Fiber::suspend()` | Pause current fiber, return control to caller |
| `Fiber::resume()` | Continue a suspended fiber |
| `Fiber::start()` | Begin executing a fiber |
| `Fiber::isStarted()` | Has the fiber been started? |
| `Fiber::isSuspended()` | Is the fiber currently paused? |
| `Fiber::isRunning()` | Is the fiber currently executing? |
| `Fiber::isTerminated()` | Has the fiber finished? |

---

## 2. Basic Fiber Usage

### 2.1 Simple Fiber Lifecycle

```php
<?php
declare(strict_types=1);

$fiber = new Fiber(function (): void {
    $value = Fiber::suspend('first suspend');
    echo "Resumed with: $value\n";
    Fiber::suspend('second suspend');
    echo "Second resume\n";
});

$value = $fiber->start();
echo "Main got: $value\n"; // "first suspend"

$fiber->resume('hello from main');
$fiber->resume(null);
```

### 2.2 Fiber Return Values

```php
<?php
declare(strict_types=1);

$fiber = new Fiber(function (int $a, int $b): int {
    $c = Fiber::suspend($a + $b);
    $d = Fiber::suspend($c + 100);
    return $d;
});

$result = $fiber->start(10, 20);        // 30
$result = $fiber->resume($result);      // 130
$result = $fiber->resume($result);      // 130 (return value)
```

---

## 3. Full Call Stack Suspension

Fibers can suspend from **any depth** in the call stack:

```php
<?php
declare(strict_types=1);

function deepFunction(int $depth): string
{
    if ($depth > 0) {
        $input = Fiber::suspend("at depth $depth");
        return deepFunction($depth - 1) . " -> $input";
    }
    return 'bottom';
}

$fiber = new Fiber(function (): string {
    return deepFunction(3);
});

echo $fiber->start();        // "at depth 3"
echo $fiber->resume('A');    // "at depth 2"
echo $fiber->resume('B');    // "at depth 1"
echo $fiber->resume('C');    // "bottom -> C -> B -> A"
```

---

## 4. Exception Handling

### 4.1 Throwing into a Fiber

```php
<?php
declare(strict_types=1);

$fiber = new Fiber(function (): void {
    try {
        $value = Fiber::suspend('waiting');
    } catch (\RuntimeException $e) {
        echo "Caught: " . $e->getMessage() . "\n";
    }
});

$fiber->start();
$fiber->throw(new \RuntimeException('Injected error'));
```

---

## 5. Cooperative Task Scheduler

```php
<?php
declare(strict_types=1);

final class Scheduler
{
    /** @var \SplQueue<Fiber> */
    private \SplQueue $queue;

    public function __construct()
    {
        $this->queue = new \SplQueue();
    }

    public function addFiber(callable $callback): void
    {
        $this->queue->enqueue(new Fiber($callback));
    }

    public function run(): void
    {
        while (!$this->queue->isEmpty()) {
            $fiber = $this->queue->dequeue();

            if (!$fiber->isStarted()) {
                $fiber->start();
            } elseif ($fiber->isSuspended()) {
                $fiber->resume(null);
            }

            if ($fiber->isSuspended()) {
                $this->queue->enqueue($fiber);
            }
        }
    }
}
```

---

## 6. Fiber vs Generator

| Feature | Generator | Fiber |
|---------|-----------|-------|
| Introduced | PHP 5.5 | PHP 8.1 |
| Suspend level | Top-level only | Any call stack depth |
| Use case | Lazy iteration | Cooperative multitasking |
| Stack depth | Single frame | Full call stack |

---

## 7. Limitations

- Cannot suspend in destructors, shutdown functions, or output buffer callbacks
- `Fiber::suspend()` outside a fiber throws `\FiberError`
- Global state is shared between fibers (not thread-safe)

---

## References

- [PHP Manual — Fibers](https://www.php.net/manual/en/language.fibers.php)
- [RFC: Fibers](https://wiki.php.net/rfc/fibers)