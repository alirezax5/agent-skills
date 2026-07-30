---
name: php-json
description: Master PHP JSON handling — encoding/decoding, flags, JsonSerializable, JSON_THROW_ON_ERROR, JSON Lines, streaming, Schema validation, and production patterns with strict_types and PSR-12
tags: [php, json, serialization, api, json-schema, strict-types]
---

# PHP JSON Mastery

## 1. Overview

JSON is the primary data interchange format for PHP applications — REST APIs, configuration files, structured logging, inter-process communication, and persistent cache. PHP's `json_encode` and `json_decode` provide C-level serialization that is fast, safe, and extensible via `JsonSerializable`.

## 2. Key Functions

### 2.1 Encoding

```php
declare(strict_types=1);

// Basic
$json = json_encode(['name' => 'Alice', 'age' => 30]);

// Pretty print with unescaped Unicode
$pretty = json_encode($data, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);

// Safe for HTML embedding
$safe = json_encode($data, JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE);

// Force object for empty arrays
$obj = json_encode([], JSON_FORCE_OBJECT); // '{}'

// Preserve zero fractional part
$float = json_encode(3.0, JSON_PRESERVE_ZERO_FRACTION); // '3.0'

// Invalid UTF-8 replacement
$utf8Safe = json_encode($data, JSON_INVALID_UTF8_SUBSTITUTE);
```

### 2.2 Decoding

```php
declare(strict_types=1);

// Basic — stdClass objects
$data = json_decode($json);
echo $data->name;

// Associative arrays
$data = json_decode($json, true);
echo $data['name'];

// With flags (PHP 7.3+)
$data = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
$data = json_decode($json, true, 512, JSON_BIGINT_AS_STRING | JSON_THROW_ON_ERROR);
```

### 2.3 Error Handling

```php
declare(strict_types=1);

// Method 1: Return code
$data = json_decode($json);
if (json_last_error() !== JSON_ERROR_NONE) {
    throw new RuntimeException('JSON decode error: ' . json_last_error_msg());
}

// Method 2: THROW_ON_ERROR (RECOMMENDED — PHP 7.3+)
try {
    $data = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
} catch (JsonException $e) {
    throw new RuntimeException('Invalid JSON: ' . $e->getMessage(), previous: $e);
}
```

## 3. Custom Serialization

### 3.1 JsonSerializable Interface

```php
declare(strict_types=1);

class User implements JsonSerializable
{
    public function __construct(
        private readonly int $id,
        private readonly string $name,
        private readonly ?string $email = null,
    ) {}

    public function jsonSerialize(): array
    {
        return [
            'id'    => $this->id,
            'name'  => $this->name,
            'email' => $this->email,
        ];
    }
}

$json = json_encode($user, JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE);
```

## 4. Common Patterns

### 4.1 API response envelope

```php
declare(strict_types=1);

class ApiResponse implements JsonSerializable
{
    private function __construct(
        private readonly bool $success,
        private readonly mixed $data = null,
        private readonly ?string $error = null,
        private readonly int $statusCode = 200,
    ) {}

    public static function success(mixed $data, int $status = 200): self
    {
        return new self(true, $data, null, $status);
    }

    public static function error(string $message, int $status = 400): self
    {
        return new self(false, null, $message, $status);
    }

    public function jsonSerialize(): array
    {
        $payload = ['success' => $this->success];
        if ($this->data !== null) $payload['data'] = $this->data;
        if ($this->error !== null) $payload['error'] = $this->error;
        return $payload;
    }

    public function getStatusCode(): int { return $this->statusCode; }
}
```

### 4.2 JSON Lines reader (NDJSON)

```php
declare(strict_types=1);

function readJsonLines(string $path): \Generator
{
    $fp = fopen($path, 'rb');
    try {
        while (($line = fgets($fp)) !== false) {
            $line = rtrim($line, "\r\n");
            if ($line === '') continue;
            try {
                yield json_decode($line, true, 512, JSON_THROW_ON_ERROR);
            } catch (JsonException $e) {
                error_log("JSONL parse error: {$e->getMessage()}");
            }
        }
    } finally {
        fclose($fp);
    }
}
```

### 4.3 JSON config file loader

```php
declare(strict_types=1);

function loadJsonConfig(string $path): array
{
    if (! file_exists($path)) {
        throw new RuntimeException("Config not found: $path");
    }
    $content = file_get_contents($path);
    if ($content === false) {
        throw new RuntimeException("Cannot read: $path");
    }
    try {
        $config = json_decode($content, true, 32, JSON_THROW_ON_ERROR);
    } catch (JsonException $e) {
        throw new RuntimeException("Invalid JSON: " . $e->getMessage(), previous: $e);
    }
    if (! is_array($config)) {
        throw new RuntimeException("Config must be a JSON object");
    }
    return $config;
}
```

## 5. Performance

| Operation | Relative Cost | Notes |
|-----------|--------------|-------|
| `json_encode` (small) | Very fast | C-level, ~100ns per scalar |
| `json_decode` (small) | Very fast | Similar profile |
| `JSON_PRETTY_PRINT` | ~2-3x slower | Extra whitespace |
| `JSON_UNESCAPED_UNICODE` | Slightly faster | Avoids escaping non-ASCII |

## References

- [PHP Manual: JSON Functions](https://www.php.net/manual/en/ref.json.php)
- [PHP Manual: JsonSerializable](https://www.php.net/manual/en/class.jsonserializable.php)
- [RFC 7159: JSON](https://datatracker.ietf.org/doc/html/rfc7159)
- [PSR-12: Extended Coding Style](https://www.php-fig.org/psr/psr-12/)