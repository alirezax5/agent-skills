---
name: php-streams
description: Master PHP streams — stream wrappers, contexts, filters, non-blocking I/O, stream_select, socket programming, pipes, and production patterns with strict_types and PSR-12
tags: [php, streams, io, sockets, pipes, stream-filters, non-blocking, strict-types]
---

# PHP Streams Mastery

## 1. Overview

PHP's stream abstraction layer unifies file I/O, network sockets, compression, process pipes, and in-memory buffers under a single interface: `fopen()`, `fread()`, `fwrite()`, `fclose()` work identically regardless of the underlying transport.

## 2. Key Functions

### 2.1 Opening Streams

```php
declare(strict_types=1);

// Standard file
$fp = fopen('/path/to/file.txt', 'rb');

// HTTP with context
$ctx = stream_context_create(['http' => [
    'method'  => 'GET',
    'header'  => "User-Agent: MyApp/1.0\r\nAccept: application/json\r\n",
    'timeout' => 5.0,
]]);
$fp = fopen('https://api.example.com/data', 'rb', false, $ctx);

// Compression
$fp = fopen('compress.zlib:///path/to/file.gz', 'rb');

// In-memory streams
$fp = fopen('php://memory', 'wb+');  // Full in memory
$fp = fopen('php://temp', 'wb+');    // Spills to disk at 2 MB

// Process pipes
$desc = [
    0 => ['pipe', 'r'],  // stdin
    1 => ['pipe', 'w'],  // stdout
    2 => ['pipe', 'w'],  // stderr
];
$proc = proc_open('cat', $desc, $pipes);
```

### 2.2 Reading & Writing

```php
declare(strict_types=1);

$fp = fopen('/path/to/file', 'rb');

$chunk = fread($fp, 8192);
while (($line = fgets($fp)) !== false) { }
$all = stream_get_contents($fp);

fwrite($fp, $data);
fflush($fp);
```

### 2.3 Stream Positioning & Metadata

```php
declare(strict_types=1);

$pos = ftell($fp);
fseek($fp, 0, SEEK_SET);
rewind($fp);
feof($fp);

$meta = stream_get_meta_data($fp);
// ['timed_out', 'blocked', 'eof', 'stream_type', 'wrapper_type', 'mode', 'seekable', 'uri']
```

### 2.4 Stream Filters

```php
declare(strict_types=1);

$fp = fopen('/path/to/file.txt', 'rb');

$filter = stream_filter_append($fp, 'string.toupper');
$content = stream_get_contents($fp);  // All uppercase
stream_filter_remove($filter);

// Base64 decode on read
stream_filter_append($fp, 'convert.base64-decode');
```

### 2.5 Non-blocking I/O & Multiplexing

```php
declare(strict_types=1);

$fp = fopen('/path/to/pipe', 'rb');
stream_set_blocking($fp, false);

$read   = [$fp];
$write  = [];
$except = [];

if (stream_select($read, $write, $except, 5) > 0) {
    foreach ($read as $stream) {
        echo stream_get_contents($stream);
    }
}
```

### 2.6 Socket Programming

```php
declare(strict_types=1);

// TCP server
$server = stream_socket_server('tcp://0.0.0.0:8080', $errno, $errstr);
while ($conn = stream_socket_accept($server, -1)) {
    $request = fread($conn, 4096);
    fwrite($conn, "HTTP/1.1 200 OK\r\nContent-Length: 2\r\n\r\nOK");
    fclose($conn);
}

// TCP client
$client = stream_socket_client('tcp://api.example.com:80', $errno, $errstr, 5.0);
```

## 3. Common Patterns

### 3.1 Reliable HTTP GET with retry

```php
declare(strict_types=1);

function httpGet(string $url, int $maxRetries = 3, float $timeout = 5.0): string
{
    for ($attempt = 1; $attempt <= $maxRetries; $attempt++) {
        $ctx = stream_context_create(['http' => [
            'method' => 'GET', 'timeout' => $timeout,
            'header' => "User-Agent: MyApp/1.0\r\n",
            'ignore_errors' => true,
        ]]);
        $result = @file_get_contents($url, false, $ctx);
        if ($result !== false) {
            return $result;
        }
        usleep(100_000 * $attempt);
    }
    throw new RuntimeException("HTTP GET failed after $maxRetries attempts");
}
```

### 3.2 Generator from stream

```php
declare(strict_types=1);

function streamLines(string $path): \Generator
{
    $fp = fopen($path, 'rb');
    try {
        while (($line = fgets($fp)) !== false) {
            yield rtrim($line, "\r\n");
        }
    } finally {
        fclose($fp);
    }
}
```

## 4. Performance

- Read in chunks of 4-64 KB for optimal throughput
- Default buffer is 8 KB; tune with `stream_set_read_buffer()`
- `stream_select()` is a single kernel `poll()`/`select()` call

## References

- [PHP Manual: Streams](https://www.php.net/manual/en/book.stream.php)
- [PHP Manual: Stream Wrappers](https://www.php.net/manual/en/wrappers.php)
- [PHP Manual: Stream Filters](https://www.php.net/manual/en/filters.php)
- [PHP Manual: Stream Contexts](https://www.php.net/manual/en/context.php)
- [PSR-12: Extended Coding Style](https://www.php-fig.org/psr/psr-12/)