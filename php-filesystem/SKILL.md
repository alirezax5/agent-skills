---
name: php-filesystem
description: Master PHP filesystem functions — directory traversal, file reading/writing, permissions, atomic operations, lock-based writes, upload handling, and production patterns with strict_types and PSR-12
tags: [php, filesystem, io, streams, permissions, strict-types]
---

# PHP Filesystem Mastery

## 1. Overview

PHP's filesystem layer provides ~100 functions for reading, writing, traversing, and managing the filesystem. It wraps POSIX and Windows APIs with a unified interface, supporting local files, stream wrappers, and network protocols.

## 2. Key Functions

### 2.1 Reading Files

```php
declare(strict_types=1);

// Whole file
$content = file_get_contents('/path/to/file.txt');
if ($content === false) {
    throw new RuntimeException('Failed to read file');
}

// Lines into array
$lines = file('/path/to/file.txt', FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);

// Stream-based
$handle = fopen('/path/to/file.txt', 'rb');
try {
    while (($line = fgets($handle)) !== false) {
        // process $line
    }
} finally {
    fclose($handle);
}

// Object-oriented
$file = new SplFileObject('/path/to/file.txt', 'rb');
$file->setFlags(SplFileObject::DROP_NEW_LINE | SplFileObject::READ_AHEAD);
foreach ($file as $lineNum => $line) {
    // process $line
}

// CSV reading
$csv = new SplFileObject('/path/to/data.csv', 'rb');
$csv->setFlags(SplFileObject::READ_CSV);
foreach ($csv as $row) {
    // $row is array of fields
}
```

### 2.2 Writing Files

```php
declare(strict_types=1);

// Simple write
$written = file_put_contents('/path/to/file.txt', $data, LOCK_EX);

// Append
file_put_contents('/path/to/log.txt', $line . "\n", FILE_APPEND | LOCK_EX);

// Stream-based
$handle = fopen('/path/to/file.txt', 'wb');
try {
    fwrite($handle, $data);
    fflush($handle);
} finally {
    fclose($handle);
}
```

### 2.3 Directory Operations

```php
declare(strict_types=1);

$items = scandir('/path/to/dir');
mkdir('/path/to/new/dir', 0755, true);

// Recursive remove
function rmdirRecursive(string $dir): void {
    $it = new RecursiveDirectoryIterator($dir, RecursiveDirectoryIterator::SKIP_DOTS);
    $files = new RecursiveIteratorIterator($it, RecursiveIteratorIterator::CHILD_FIRST);
    foreach ($files as $file) {
        $file->isDir() ? rmdir($file->getRealPath()) : unlink($file->getRealPath());
    }
    rmdir($dir);
}

// DirectoryIterator
$dir = new DirectoryIterator('/path/to/dir');
foreach ($dir as $item) {
    if ($item->isDot()) continue;
    echo $item->getFilename() . "\n";
}
```

### 2.4 File Metadata

```php
declare(strict_types=1);

$stat = stat($path);
filesize($path);
filemtime($path);
filetype($path);
fileperms($path);

file_exists($path);
is_file($path);
is_dir($path);
is_link($path);
is_readable($path);
is_writable($path);

realpath($path);
clearstatcache();
```

## 3. Common Patterns

### 3.1 Atomic file write

```php
declare(strict_types=1);

function atomicWrite(string $path, string $content): void
{
    $tmp = $path . '.' . bin2hex(random_bytes(8)) . '.tmp';
    $written = file_put_contents($tmp, $content, LOCK_EX);
    if ($written === false) {
        throw new RuntimeException("Failed to write temp file: $tmp");
    }
    if (rename($tmp, $path) === false) {
        @unlink($tmp);
        throw new RuntimeException("Failed to rename $tmp to $path");
    }
}
```

### 3.2 Locked read/write

```php
declare(strict_types=1);

function lockedRead(string $path): string
{
    $handle = fopen($path, 'rb');
    flock($handle, LOCK_SH);
    try {
        return stream_get_contents($handle);
    } finally {
        flock($handle, LOCK_UN);
        fclose($handle);
    }
}
```

### 3.3 Safe file upload

```php
declare(strict_types=1);

function handleUpload(array $file, string $destDir): string
{
    if ($file['error'] !== UPLOAD_ERR_OK) {
        throw new RuntimeException('Upload error: ' . $file['error']);
    }
    $finfo = new finfo(FILEINFO_MIME_TYPE);
    $mime = $finfo->file($file['tmp_name']);
    $allowed = ['image/jpeg', 'image/png', 'application/pdf'];
    if (! in_array($mime, $allowed, true)) {
        throw new RuntimeException("Disallowed MIME: $mime");
    }
    $ext = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
    $destName = bin2hex(random_bytes(16)) . '.' . $ext;
    $destPath = $destDir . '/' . $destName;
    if (move_uploaded_file($file['tmp_name'], $destPath) === false) {
        throw new RuntimeException('Failed to move uploaded file');
    }
    return $destPath;
}
```

### 3.4 Ensure directory exists

```php
declare(strict_types=1);

function ensureDir(string $path, int $mode = 0755): string
{
    if (! is_dir($path)) {
        $parent = dirname($path);
        if (! is_dir($parent)) {
            ensureDir($parent, $mode);
        }
        $created = mkdir($path, $mode, false);
        if ($created === false && ! is_dir($path)) {
            throw new RuntimeException("Cannot create dir: $path");
        }
    }
    return realpath($path) ?: $path;
}
```

## 4. Performance

| Operation | Cache | Notes |
|-----------|-------|-------|
| `file_exists` | Yes (stat cache) | Returns immediately for cached |
| `filesize` | Yes | |
| `file_get_contents` small file | No | Reads into memory wholly |
| `fopen` + streaming | No | Process in chunks |
| `rename` (same FS) | No | Atomic metadata op |

## References

- [PHP Manual: Filesystem Functions](https://www.php.net/manual/en/ref.filesystem.php)
- [PHP Manual: Directory Functions](https://www.php.net/manual/en/ref.dir.php)
- [PHP Manual: SplFileObject](https://www.php.net/manual/en/class.splfileobject.php)
- [PSR-12: Extended Coding Style](https://www.php-fig.org/psr/psr-12/)