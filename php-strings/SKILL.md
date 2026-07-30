---
name: php-strings
description: Master PHP string functions — encoding, manipulation, regex, interpolation, multibyte safety, PCRE2, and production patterns with strict_types and PSR-12
tags: [php, strings, encoding, multibyte, regex, pcre, strict-types]
---

# PHP String Mastery

## 1. Overview

PHP provides one of the richest string-handling toolkits of any language: ~100 native string functions, support for multiple encodings via the `mb_*` extension, PCRE2 regex, and low-level binary manipulation.

## 2. Key Functions Reference

### 2.1 Length & Substring

```php
declare(strict_types=1);

$str = 'Hëllo 🌍';

strlen($str);       // 11 bytes
mb_strlen($str);    // 7 characters
mb_strlen($str, 'UTF-8');  // 7

substr($str, 0, 5);       // 'Hëllo' — byte-safe
mb_substr($str, 0, 5);    // 'Hëllo ' — character-safe
```

### 2.2 Search & Replace

```php
declare(strict_types=1);

$haystack = 'The quick brown fox';

strpos($haystack, 'quick');          // 4 (byte offset)
str_contains($haystack, 'quick');    // true (PHP 8.0+)
str_starts_with($haystack, 'The');   // true (PHP 8.0+)
str_ends_with($haystack, 'fox');     // true (PHP 8.0+)

str_replace('quick', 'slow', $haystack);
str_ireplace('QUICK', 'slow', $haystack);
substr_replace($haystack, 'Lazy', 4, 5);

trim('  hello  ');
ltrim('  hello  ');
rtrim('  hello  ');
```

### 2.3 Case Conversion

```php
declare(strict_types=1);

// ASCII-safe
strtolower($str);
strtoupper($str);
ucfirst($str);
ucwords($str);

// Multibyte safe
mb_strtolower($str, 'UTF-8');
mb_strtoupper($str, 'UTF-8');
mb_convert_case($str, MB_CASE_TITLE, 'UTF-8');
```

### 2.4 Formatting & Building

```php
declare(strict_types=1);

$msg = sprintf('User %s is %d years old', $name, $age);

// Heredoc
$html = <<<HTML
<div class="user">
    <h2>$name</h2>
</div>
HTML;

// Nowdoc (no interpolation)
$sql = <<<'SQL'
SELECT * FROM users WHERE name = ?
SQL;

str_pad('hello', 10, '-', STR_PAD_BOTH);
number_format(1234.567, 2);           // '1,234.57'
```

### 2.5 Split & Join

```php
declare(strict_types=1);

$parts = explode(',', $csv);             // ['a', 'b', 'c']
$joined = implode(',', ['a', 'b', 'c']); // 'a,b,c'
$chars = mb_str_split('Hëllo 🌍');       // ['H', 'ë', 'l', 'l', 'o', ' ', '🌍']
```

### 2.6 Regex (PCRE2)

```php
declare(strict_types=1);

$str = 'Contact: alice@example.com';

preg_match_all('/[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/', $str, $matches);

preg_match('/(?P<name>\w+)@(?P<domain>[\w.]+)/', $str, $m);
// $m['name'] = 'alice'

$clean = preg_replace('/[^a-zA-Z0-9]/', '-', $str);
```

### 2.7 Encoding & Conversion

```php
declare(strict_types=1);

$enc = mb_detect_encoding($utf8, ['UTF-8', 'ISO-8859-1', 'Windows-1252'], true);
$latin1 = mb_convert_encoding($utf8, 'ISO-8859-1', 'UTF-8');
mb_check_encoding($utf8, 'UTF-8');

$cp = mb_ord('🌍');       // 127757
$char = mb_chr(127757);   // '🌍'
```

## 3. Common Patterns

### 3.1 Slug generation

```php
declare(strict_types=1);

function slugify(string $text, string $separator = '-'): string
{
    $text = normalizer_normalize($text, Normalizer::FORM_KD);
    $text = preg_replace('/[^\x00-\x7F]/', '', $text);
    $text = preg_replace('/[^a-zA-Z0-9]+/', $separator, $text);
    return trim(mb_strtolower($text), $separator);
}
```

### 3.2 Secure random string

```php
declare(strict_types=1);

function randomString(int $length = 32): string
{
    $chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    $result = '';
    for ($i = 0; $i < $length; $i++) {
        $result .= $chars[random_int(0, 61)];
    }
    return $result;
}
```

## 4. Performance

- `strlen()` is O(1) — length stored in zend_string
- For large strings in loops, collect parts in array and `implode` once
- Repeated `.=` on long strings is O(n²)

## References

- [PHP Manual: Strings](https://www.php.net/manual/en/language.types.string.php)
- [PHP Manual: Multibyte String Functions](https://www.php.net/manual/en/ref.mbstring.php)
- [PHP Manual: PCRE Functions](https://www.php.net/manual/en/ref.pcre.php)