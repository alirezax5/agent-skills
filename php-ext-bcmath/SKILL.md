---
name: php-bcmath
description: BCMath — arbitrary precision decimal arithmetic, bcadd, bcsub, bcmul, bcdiv, bcpow, bcmod, bcscale, financial math, comparison with GMP
php_version: 8.4
tags:
  - php
  - bcmath
  - arbitrary-precision
  - decimal
  - financial
  - big-numbers
---

# BCMath — Arbitrary Precision Mathematics

## Overview

BCMath (Binary Calculator) provides arbitrary precision decimal arithmetic for PHP. Unlike GMP which handles only integers, BCMath works with decimal numbers at a user-defined scale (number of decimal places). It's the standard choice for financial calculations, currency operations, and any context requiring exact decimal representation without floating-point rounding errors.

## Core Functions

All BCMath functions operate on **strings** and return **strings** (since PHP 8.0, passing non-string scalars also works).

### Basic Arithmetic

```php
<?php
declare(strict_types=1);

// bcadd — addition
$sum = bcadd('1.234', '5.678', 3);    // '6.912'

// bcsub — subtraction
$diff = bcsub('10.5', '3.2', 2);      // '7.30'

// bcmul — multiplication
$product = bcmul('2.5', '3.5', 4);    // '8.7500'

// bcdiv — division
$quotient = bcdiv('10', '3', 6);       // '3.333333'

// bcmod — modulus
$mod = bcmod('17', '5');               // '2'

// bcpow — power
$pow = bcpow('2.5', '3', 4);           // '15.6250'

// bcsqrt — square root
$sqrt = bcsqrt('2', 20);               // '1.41421356237309504880'

// bccomp — comparison (returns -1, 0, 1)
$cmp = bccomp('1.5', '2.0', 2);        // -1
$cmp = bccomp('100', '100');           // 0
```

### Scale Management

```php
<?php
declare(strict_types=1);

// Global scale
bcscale(4);
echo bcadd('1.23456', '2.34567');       // '3.5802'

// Per-operation scale overrides global
echo bcdiv('1', '3', 10);               // '0.3333333333'

// Get current scale
$scale = bcscale();                     // 4
```

## Financial Calculations

```php
<?php
declare(strict_types=1);

bcscale(2);

// Invoice line totals
$items = [
    ['price' => '29.99', 'qty' => '2'],
    ['price' => '4.50', 'qty' => '3'],
    ['price' => '99.99', 'qty' => '1'],
];

$subtotal = '0.00';
foreach ($items as $item) {
    $subtotal = bcadd($subtotal, bcmul($item['price'], $item['qty']));
}
// $subtotal = '173.47'

// Tax calculation (8.25%)
$tax = bcmul($subtotal, '0.0825', 2);     // '14.31'
$total = bcadd($subtotal, $tax, 2);       // '187.78'
```

### Compound Interest

```php
<?php
declare(strict_types=1);

bcscale(10);

function compoundInterest(string $principal, string $rate, int $compoundsPerYear, int $years): string
{
    $factor = bcpow(bcadd('1', bcdiv($rate, (string)$compoundsPerYear)), (string)($compoundsPerYear * $years));
    return bcmul($principal, $factor, 2);
}

// $1000 at 5% compounded monthly for 10 years
echo compoundInterest('1000.00', '0.05', 12, 10); // '1647.01'
```

## Comparison: BCMath vs GMP vs float

| Feature | Native (float) | BCMath | GMP |
|---------|---------------|--------|-----|
| **Precision** | ~15-17 digits | Arbitrary | Arbitrary (integers only) |
| **Decimals** | Yes (binary) | Yes (decimal) | No |
| **Speed** | Fastest | Slowest | ~5-30x faster than BC |
| **Type** | float | string | GMP object |
| **0.1 + 0.2** | `0.30000000000000004` | `'0.3'` | N/A |

## Precise Rounding

```php
<?php
declare(strict_types=1);

function bcround(string $value, int $precision = 0): string
{
    $multiplier = bcpow('10', (string)$precision);
    $truncated = bcadd($value, '0', $precision + 1);
    $truncated_bc = bcmul($truncated, $multiplier, 0);
    $remainder = bcmod(bcmul($truncated, bcpow('10', (string)($precision + 1))), '10');
    
    if (bccomp($remainder, '5') >= 0) {
        $truncated_bc = bcadd($truncated_bc, '1');
    }
    
    return bcdiv($truncated_bc, $multiplier, $precision);
}

echo bcround('3.3333', 2);   // '3.33'
echo bcround('3.3367', 2);   // '3.34'
```

## Common Pitfalls

1. **Truncation, not rounding** — BCMath truncates by default. Always apply explicit rounding for financial values.
2. **Performance** — BCMath is slow for large integers. Use GMP for integer-only operations.
3. **`bcpowmod()` only works with integers** — All parameters must be integer strings.
4. **Comparison with `bccomp()`** — Don't use `==` to compare BC results. `'1.50' != '1.5'` but `bccomp('1.50', '1.5') === 0`.
5. **Locale independence** — BCMath always uses `.` as decimal separator. Not affected by locale settings.

## References

- [PHP: BCMath](https://www.php.net/manual/en/book.bc.php)
- [PHP: BCMath Functions](https://www.php.net/manual/en/ref.bc.php)
- [PHP: bcscale](https://www.php.net/manual/en/function.bcscale.php)