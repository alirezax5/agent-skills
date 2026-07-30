---
name: php-gmp
description: GNU Multiple Precision (GMP) — arbitrary precision arithmetic, gmp_* functions, big integer math, number theory, performance vs BCMath
php_version: 8.4
tags:
  - php
  - gmp
  - arbitrary-precision
  - big-integers
  - number-theory
  - math
---

# GMP — GNU Multiple Precision Arithmetic

## Overview

The GMP extension provides arbitrary-precision integer arithmetic using the GNU MP library. It supports integers of unlimited size, number-theoretic functions (GCD, modular exponentiation, primality testing), and is significantly faster than BCMath for integer operations. GMP is the preferred choice for cryptographic operations, large integer computation, and number theory.

## Core Functions

### Creating GMP Numbers

```php
<?php
declare(strict_types=1);

$a = gmp_init(42);             // From integer
$b = gmp_init('12345678901234567890');  // From string (arbitrary size)
$c = gmp_init('ff', 16);       // From hex string: 255

$bytes = hex2bin('deadbeef');
$d = gmp_import($bytes);       // 0xdeadbeef = 3735928559

echo gmp_export($d);           // binary string
echo gmp_strval($d);           // '3735928559'
echo gmp_strval($d, 16);       // 'deadbeef'
```

### Arithmetic Operations

```php
<?php
declare(strict_types=1);

$a = gmp_init('12345678901234567890');
$b = gmp_init('98765432109876543210');

$sum   = gmp_add($a, $b);
$diff  = gmp_sub($a, $b);
$prod  = gmp_mul($a, $b);
$div   = gmp_div_q($a, $b);
$mod   = gmp_mod($a, $b);
$pow   = gmp_pow($a, 3);
$sqrt  = gmp_sqrt($a);
$neg   = gmp_neg($a);
$abs   = gmp_abs($neg);

// Division with remainder
[$q, $r] = gmp_div_qr($a, $b);
```

### Number Theory Functions

```php
<?php
declare(strict_types=1);

$gcd = gmp_gcd(gmp_init('1234'), gmp_init('5678'));       // 2
$lcm = gmp_lcm(gmp_init('1234'), gmp_init('5678'));      // 3503326
$fact = gmp_fact(100);           // 100!
$binomial = gmp_binomial(50, 25); // C(50,25)

// Extended GCD: returns [g, x, y] where g = a*x + b*y
[$g, $x, $y] = gmp_gcdext(gmp_init('1234'), gmp_init('5678'));
```

### Modular Arithmetic (Cryptographic)

```php
<?php
declare(strict_types=1);

$result = gmp_powm(gmp_init('5'), gmp_init('3'), gmp_init('13'));
// 5^3 mod 13 = 8

$inv = gmp_invert(gmp_init('5'), gmp_init('13'));
// 5^-1 mod 13
```

### Primality Testing

```php
<?php
declare(strict_types=1);

$probable_prime = gmp_prob_prime(gmp_init('17'));
// 2 = definitely prime, 1 = probably prime, 0 = composite

$prime = gmp_random_prime(256);  // 256-bit prime
$random = gmp_random_bits(128);  // 128-bit random number
```

### Bitwise Operations

```php
<?php
declare(strict_types=1);

$a = gmp_init('0b1100');   // 12
$b = gmp_init('0b1010');   // 10

$and  = gmp_and($a, $b);    // 8 (0b1000)
$or   = gmp_or($a, $b);     // 14 (0b1110)
$xor  = gmp_xor($a, $b);    // 6 (0b0110)
$not  = gmp_com($a);        // Bitwise NOT (-13)

$popcount = gmp_popcount($a);     // 2
```

### Comparison

```php
<?php
declare(strict_types=1);

$cmp = gmp_cmp(gmp_init('100'), gmp_init('200'));
// -1: a < b, 0: a == b, 1: a > b
```

## Performance: GMP vs BCMath

| Operation | Native (int) | BCMath | GMP |
|-----------|-------------|--------|-----|
| 100-digit addition | 0.5 µs | 20 µs | 3 µs |
| 100-digit multiplication | — | 150 µs | 15 µs |
| 1000-digit multiplication | — | 15 ms | 0.5 ms |
| Modular exponentiation (RSA 2048) | — | 60 ms | 15 ms |
| 1000-digit GCD | — | 120 µs | 20 µs |

**Key takeaway**: GMP is **5–30× faster** than BCMath for integer operations.

## Real-World Use Cases

### RSA Key Generation

```php
<?php
declare(strict_types=1);

function generateRsaKeyPair(int $bits = 2048): array
{
    do {
        $p = gmp_random_prime($bits / 2);
        $q = gmp_random_prime($bits / 2);
    } while (gmp_cmp($p, $q) === 0);
    
    $n = gmp_mul($p, $q);
    $phi = gmp_mul(gmp_sub($p, 1), gmp_sub($q, 1));
    $e = gmp_init(65537);
    $d = gmp_invert($e, $phi);
    
    return [
        'public'  => ['n' => gmp_strval($n), 'e' => 65537],
        'private' => ['n' => gmp_strval($n), 'd' => gmp_strval($d)],
    ];
}
```

## Common Pitfalls

1. **GMP objects are not integers** — `gmp_init(5) + gmp_init(3)` gives 2 (object-to-int conversion). Use `gmp_add()`.
2. **Negative modulus** — `gmp_mod()` always returns non-negative result (unlike `%` operator).
3. **GMP vs BCMath for decimals** — GMP only does integers. Use BCMath for decimal/financial calculations.
4. **`gmp_random_seed()` reliability** — Use `random_bytes()` + `gmp_import()` for cryptographic randomness.

## References

- [PHP: GMP](https://www.php.net/manual/en/book.gmp.php)
- [PHP: GMP Functions](https://www.php.net/manual/en/ref.gmp.php)
- [GMP Library Manual](https://gmplib.org/manual/)