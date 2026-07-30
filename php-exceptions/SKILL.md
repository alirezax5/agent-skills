---
name: php-exceptions
description: PHP Exceptions & Error Handling — Throwable interface, try/catch/finally, custom exceptions, error-to-exception bridging, and modern error handling patterns for PHP 8.4 with strict_types and PSR-12
tags: [php, exceptions, error-handling, throwable, try-catch-finally, strict-types]
---

# PHP Exceptions & Error Handling

> **PHP 8.4** · `declare(strict_types=1)` · Modern error handling patterns

---

## 1. Overview

PHP's error handling model evolved from procedural error reporting (`trigger_error()`, `die()`) to a full object-oriented exception model with `Throwable` interface, try/catch/finally blocks, and structured error-to-exception bridging. As of PHP 8.4, the engine uses exceptions for almost everything — including type errors, division by zero, argument count mismatches, and internal engine errors.

```
┌───────────────┐
│   Throwable   │  (interface — all errors/exceptions)
└───────┬───────┘
        │
   ┌────┴────────────┐
   │                 │
┌──▼───────┐  ┌─────▼──────────┐
│ Exception │  │  Error         │  (engine-level, not catchable by traditional catch(Exception))
└──┬──┬──┬──┘  └─────┬──────────┘
   │  │  │            │
   │  │  └─ ...       └─ TypeError, ValueError, ...
   │  │
   │  └─ RuntimeException, LogicException, ...
   │
   └─ Custom exceptions
```

### 1.1 Key Concepts

| Concept | Description |
|---------|-------------|
| `Throwable` | Base interface, implemented by `Exception` and `Error` |
| `Exception` | Base class for application & runtime exceptions |
| `Error` | Base class for internal PHP engine errors (catchable since PHP 7) |
| `try` | Executes risky code |
| `catch` | Handles thrown exceptions |
| `finally` | Always executes, regardless of exception |
| `throw` | Triggers an exception |

---

## 2. The Throwable Interface

All errors and exceptions implement `Throwable`:

```php
interface Throwable
{
    public function getMessage(): string;
    public function getCode(): int;
    public function getFile(): string;
    public function getLine(): int;
    public function getTrace(): array;
    public function getTraceAsString(): string;
    public function getPrevious(): ?Throwable;
    public function __toString(): string;
}
```

### 2.1 Exception vs. Error

```php
try {
    // Something that might go wrong
} catch (\Throwable $e) {
    // Catches EVERYTHING (both Exception and Error)
}

try {
    // ...
} catch (\Exception $e) {
    // Catches only Exception subclasses — NOT Engine Errors!
    // Prefer catching Throwable or specific types
}
```

---

## 3. Exception Hierarchy

PHP provides a rich hierarchy in SPL:

```
Exception
├── LogicException
│   ├── BadFunctionCallException
│   │   └── BadMethodCallException
│   ├── DomainException
│   ├── InvalidArgumentException
│   ├── LengthException
│   └── OutOfRangeException
├── RuntimeException
│   ├── OutOfBoundsException
│   ├── OverflowException
│   ├── RangeException
│   ├── UnderflowException
│   ├── UnexpectedValueException
│   └── PDOException
├── JsonException          (PHP 7.3+)
├── TimeoutException       (PHP 7.3+ — used by Fibers)
└── SodiumException
```

```php
Error
├── TypeError                   (type mismatch)
├── ValueError                  (invalid value, PHP 8.0+)
├── ArithmeticError
│   └── DivisionByZeroError     (PHP 8.0+)
├── CompileError
├── ParseError
└── AssertionError
```

---

## 4. Basic try/catch/finally

```php
<?php
declare(strict_types=1);

try {
    $result = riskyOperation();
    echo "Success: $result\n";
} catch (\InvalidArgumentException $e) {
    echo "Bad argument: " . $e->getMessage() . "\n";
} catch (\RuntimeException $e) {
    echo "Runtime issue: " . $e->getMessage() . "\n";
} catch (\Throwable $e) {
    echo "Unexpected error: " . $e->getMessage() . "\n";
} finally {
    echo "Cleanup completed.\n";
}
```

### 4.1 Multiple Catch Types (PHP 7.1+)

```php
try {
    // ...
} catch (\InvalidArgumentException | \OutOfBoundsException $e) {
    // Handle both identically
    echo "Argument or bounds error: " . $e->getMessage() . "\n";
}
```

### 4.2 Finally — Always Executes

`finally` runs whether the try block completes normally, a catch block executes, or a throw re-propagates. Use it for:

- Closing database connections
- Releasing file handles
- Restoring global state
- Unlocking mutexes

```php
function readFileSafe(string $path): string
{
    $handle = fopen($path, 'r');
    if ($handle === false) {
        throw new \RuntimeException("Cannot open $path");
    }

    try {
        $content = fread($handle, filesize($path));
        if ($content === false) {
            throw new \RuntimeException("Cannot read $path");
        }
        return $content;
    } finally {
        fclose($handle); // Always closes, even if re-thrown
    }
}
```

---

## 5. Throwing Exceptions

```php
<?php
declare(strict_types=1);

class PaymentProcessor
{
    /**
     * @throws \RuntimeException
     */
    public function charge(float $amount, string $currency): string
    {
        if ($amount <= 0) {
            throw new \InvalidArgumentException(
                'Amount must be positive',
                1001
            );
        }

        if (!in_array($currency, ['USD', 'EUR', 'GBP'], true)) {
            throw new \InvalidArgumentException(
                "Unsupported currency: $currency",
                1002
            );
        }

        $result = $this->gateway->charge($amount, $currency);
        if ($result === false) {
            throw new \RuntimeException(
                'Payment gateway rejected transaction',
                2001
            );
        }

        return $result;
    }
}
```

### 5.1 Exception Chaining

```php
<?php
declare(strict_types=1);

try {
    performDatabaseQuery();
} catch (\PDOException $e) {
    throw new \RuntimeException(
        'Database query failed',
        0,
        $e
    );
}
```

---

## 6. Custom Exception Classes

```php
<?php
declare(strict_types=1);

namespace App\Exception;

class OrderException extends \RuntimeException
{
    private string $orderId;

    public function __construct(
        string $message,
        string $orderId,
        int $code = 0,
        ?\Throwable $previous = null
    ) {
        $this->orderId = $orderId;
        parent::__construct($message, $code, $previous);
    }

    public function getOrderId(): string
    {
        return $this->orderId;
    }
}
```

### 6.1 Domain-Specific Exceptions Pattern

```
App\Exception\
├── OrderException
│   ├── OrderNotFoundException
│   ├── OrderAlreadyShippedException
│   └── OrderPaymentFailedException
├── UserException
│   ├── UserNotFoundException
│   └── AuthenticationException
└── PaymentException
    ├── InsufficientFundsException
    └── GatewayTimeoutException
```

---

## 7. Error-to-Exception Bridging

PHP allows converting traditional PHP errors (warnings, notices) into exceptions via `set_error_handler()`:

```php
<?php
declare(strict_types=1);

set_error_handler(function (
    int $severity,
    string $message,
    string $file,
    int $line
): bool {
    if (!(error_reporting() & $severity)) {
        return false;
    }
    throw new \ErrorException($message, 0, $severity, $file, $line);
});

try {
    $arr = [];
    $value = $arr['undefined_key'];
} catch (\ErrorException $e) {
    echo "Caught: " . $e->getMessage() . "\n";
}
```

---

## 8. Engine Errors (Error Hierarchy)

Since PHP 7, engine errors are catchable. They do NOT extend `Exception` — they extend `Error`.

```php
<?php
declare(strict_types=1);

class UserManager
{
    public function __construct(
        private string $name
    ) {}
}

try {
    $mgr = new UserManager(42);
} catch (\TypeError $e) {
    echo $e->getMessage() . "\n";
}

try {
    $result = intdiv(PHP_INT_MIN, -1);
} catch (\ArithmeticError $e) {
    echo $e->getMessage() . "\n";
}

try {
    $result = 1 / 0;
} catch (\DivisionByZeroError $e) {
    echo "Cannot divide by zero\n";
}
```

---

## 9. Catch Without Variable (PHP 8.0+)

```php
<?php
declare(strict_types=1);

try {
    maybeThrow();
} catch (\SpecificException) {
    echo "Specific exception was caught (details unimportant)\n";
}
```

---

## 10. `finally` and Return Values

```php
<?php
declare(strict_types=1);

function test(): string
{
    try {
        return 'from try';
    } finally {
        return 'from finally';
    }
}

echo test(); // outputs: "from finally"
```

> **Pitfall**: Never return from `finally` unless you intend to suppress the previous return value.

---

## 11. Global Exception Handler

```php
<?php
declare(strict_types=1);

set_exception_handler(function (\Throwable $e): void {
    http_response_code(500);
    header('Content-Type: application/json');
    echo json_encode([
        'error' => true,
        'message' => 'Internal server error',
        'id' => uniqid('err_', true),
    ]);
    error_log(sprintf(
        "[UNHANDLED] %s in %s:%d\n%s",
        $e->getMessage(),
        $e->getFile(),
        $e->getLine(),
        $e->getTraceAsString()
    ));
});
```

---

## 12. Best Practices

| Rule | Explanation |
|------|-------------|
| **Be specific** | Catch the exact exception type, not `\Throwable` |
| **Don't swallow** | Empty catch blocks hide bugs — at least log |
| **Preserve chain** | Pass `$previous` when wrapping exceptions |
| **Fail fast** | Throw `\InvalidArgumentException` at input boundaries |
| **Domain exceptions** | Create custom classes for your domain |
| **finally for cleanup** | Never for flow control or return values |
| **Catch at boundaries** | HTTP controllers, CLI entry points, queue workers |
| **Use set_error_handler** | Convert warnings to exceptions for consistency |

---

## References

- [PHP Manual — Language Exceptions](https://www.php.net/manual/en/language.exceptions.php)
- [PHP Manual — Error](https://www.php.net/manual/en/class.error.php)