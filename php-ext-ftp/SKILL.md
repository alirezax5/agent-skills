---
name: php-ftp
description: FTP — ftp_connect, login, get, put, passive/active mode, FTP over SSL, recursive operations, FTP stream wrapper
php_version: 8.4
tags:
  - php
  - ftp
  - file-transfer
  - sftp
  - network
  - ssl
---

# FTP — File Transfer Protocol

## Overview

PHP's FTP extension provides a comprehensive client implementation of the File Transfer Protocol (FTP and FTPS). It supports both active and passive modes, SSL/TLS encryption, recursive operations, and an `ftp://` stream wrapper. For SFTP (SSH File Transfer Protocol), use the SSH2 extension instead.

```mermaid
flowchart LR
    subgraph "FTP Connection Modes"
        PASSIVE[Passive Mode<br>Client connects to server port]
        ACTIVE[Active Mode<br>Server connects to client port]
    end
    subgraph "Security"
        PLAIN[FTP — plaintext]
        FTPS[FTPS — FTP over SSL/TLS]
        SFTP[SFTP — SSH File Transfer<br>(NOT this extension)]
    end
```

## Connection & Authentication

### Basic Connection

```php
<?php
declare(strict_types=1);

// Connect to FTP server
$ftp = ftp_connect('ftp.example.com', 21, 90);  // host, port, timeout
if ($ftp === false) {
    throw new \RuntimeException('Could not connect to FTP server');
}

// Login
$loggedIn = ftp_login($ftp, 'username', 'password');
if (!$loggedIn) {
    throw new \RuntimeException('FTP login failed');
}

// Now connected — do operations...

// Close connection
ftp_close($ftp);
```

### FTP over SSL (FTPS)

```php
<?php
declare(strict_types=1);

// Explicit FTPS (FTP over TLS) — connect on standard port then upgrade
$ftp = ftp_ssl_connect('ftp.example.com', 21, 90);
if ($ftp === false) {
    throw new \RuntimeException('FTPS connection failed');
}

ftp_login($ftp, 'username', 'password');

// Enable passive mode with TLS
ftp_pasv($ftp, true);

// Now operations are encrypted

// Implicit FTPS (legacy — port 990)
// $ftp = ftp_ssl_connect('ftp.example.com', 990, 90);
```

### Connection Options

```php
<?php
declare(strict_types=1);

$ftp = ftp_connect('ftp.example.com');

// Set timeout after connect
ftp_set_option($ftp, FTP_TIMEOUT_SEC, 120);

// Auto-seek for resuming downloads (default: true)
ftp_set_option($ftp, FTP_AUTOSEEK, true);

// Set transfer mode (FTP_ASCII or FTP_BINARY — use autoselect)
// FTP_AUTOSELECT — auto-detect based on file extension (PHP 8.0+)
ftp_set_option($ftp, FTP_USEPASVADDRESS, false); // Don't use PASV IP (some NAT setups)

// Get current timeout
$timeout = ftp_get_option($ftp, FTP_TIMEOUT_SEC);
```

## File Operations

### Downloading

```php
<?php
declare(strict_types=1);

// Download a file (binary mode)
$downloaded = ftp_get($ftp, '/local/path/file.zip', '/remote/path/file.zip', FTP_BINARY);
if (!$downloaded) {
    throw new \RuntimeException('Download failed');
}

// Download in ASCII mode (for text files)
ftp_get($ftp, '/local/readme.txt', '/remote/readme.txt', FTP_ASCII);

// Or use autoselect (PHP 8.0+)
ftp_get($ftp, '/local/file.zip', '/remote/file.zip', FTP_AUTOSELECT);

// Resume download (if FTP_AUTOSEEK is on)
$ret = ftp_nb_get($ftp, '/local/large.zip', '/remote/large.zip', FTP_BINARY, ftp_size($ftp, '/remote/large.zip'));

// Non-blocking download with progress
$localFile = '/tmp/big-file.iso';
$remoteFile = '/backups/big-file.iso';
$remoteSize = ftp_size($ftp, $remoteFile);

$ret = ftp_nb_get($ftp, $localFile, $remoteFile, FTP_BINARY);
while ($ret === FTP_MOREDATA) {
    // Show progress
    $localSize = file_exists($localFile) ? filesize($localFile) : 0;
    $percent = $remoteSize > 0 ? round($localSize / $remoteSize * 100) : 0;
    echo "Downloaded: $percent%\n";
    
    $ret = ftp_nb_continue($ftp);
}

if ($ret === FTP_FINISHED) {
    echo "Download complete\n";
} else {
    echo "Download failed\n";
}
```

### Uploading

```php
<?php
declare(strict_types=1);

// Upload a file
$uploaded = ftp_put($ftp, '/remote/path/file.zip', '/local/path/file.zip', FTP_BINARY);
if (!$uploaded) {
    throw new \RuntimeException('Upload failed');
}

// Upload from string (create a remote file from content)
$tempHandle = fopen('php://temp', 'r+');
fwrite($tempHandle, 'This is file content');
rewind($tempHandle);
ftp_fput($ftp, '/remote/newfile.txt', $tempHandle, FTP_ASCII);
fclose($tempHandle);

// Non-blocking upload with progress
$localPath = '/tmp/large-file.mp4';
$remotePath = '/uploads/large-file.mp4';
$localSize = filesize($localPath);

$ret = ftp_nb_put($ftp, $remotePath, $localPath, FTP_BINARY);
while ($ret === FTP_MOREDATA) {
    $percent = round(ftp_size($ftp, $remotePath) / $localSize * 100);
    echo "Uploaded: $percent%\n";
    
    usleep(500000); // Check every 0.5s
    $ret = ftp_nb_continue($ftp);
}

if ($ret === FTP_FINISHED) {
    echo "Upload complete\n";
}
```

## Directory Operations

```php
<?php
declare(strict_types=1);

// Get current directory
$cwd = ftp_pwd($ftp);
echo "Current directory: $cwd\n";

// Change directory
ftp_chdir($ftp, '/var/www/html');

// Change to parent
ftp_cdup($ftp);

// List files (returns array of file names)
$files = ftp_nlist($ftp, '/var/www/html');
// ['index.php', 'style.css', 'script.js', ...]

// Raw listing (detailed)
$raw = ftp_rawlist($ftp, '/var/www/html');
// Each entry: "drwxr-xr-x  2 user group 4096 Dec 25 12:00 directory"
foreach ($raw as $line) {
    echo $line . "\n";
}

// Parse raw listing into structured data (manual)
function parseRawList(array $rawList): array
{
    $items = [];
    foreach ($rawList as $line) {
        if (preg_match(
            '/^([drwxlst-]{10})\\s+\\d+\\s+\\S+\\s+\\S+\\s+(\\d+)\\s+'
            . '(\\w{3}\\s+\\d{1,2}\\s+(?:\\d{1,2}:\\d{2}|\\d{4}))\\s+(.+)$/',
            $line,
            $m
        )) {
            $items[] = [
                'permissions' => $m[1],
                'size' => (int)$m[2],
                'date' => $m[3],
                'name' => $m[4],
                'is_dir' => $m[1][0] === 'd',
            ];
        }
    }
    return $items;
}

// Create directory
ftp_mkdir($ftp, '/var/www/html/new-folder');

// Remove directory (must be empty)
ftp_rmdir($ftp, '/var/www/html/empty-folder');

// Delete file
ftp_delete($ftp, '/var/www/html/old-file.txt');

// Rename / move
ftp_rename($ftp, '/var/www/html/old-name.txt', '/var/www/html/new-name.txt');

// Check if file exists
$exists = ftp_size($ftp, '/var/www/html/index.php');
if ($exists !== -1) {
    echo "File exists\n";
}
```

## Passive vs Active Mode

```php
<?php
declare(strict_types=1);

// Passive mode — client connects to data port (default, recommended)
// Works through most firewalls
ftp_pasv($ftp, true);

// Active mode — server connects back to client
// May be blocked by client-side firewalls
ftp_pasv($ftp, false);

// When to use each:
// Passive: Most environments, through NAT, cloud servers
// Active: When server firewall blocks passive range, legacy setups

// Check current mode
// No direct getter; track it yourself
$isPassive = ...; // Store from ftp_pasv() calls
```

## Recursive Operations

### Recursive Download

```php
<?php
declare(strict_types=1);

function ftpDownloadDir(
    \FTP\Connection $ftp,
    string $remoteDir,
    string $localDir
): void {
    if (!is_dir($localDir)) {
        mkdir($localDir, 0755, true);
    }
    
    $items = ftp_rawlist($ftp, $remoteDir);
    $parsed = parseRawList($items);  // See above
    
    foreach ($parsed as $item) {
        $remotePath = "$remoteDir/{$item['name']}";
        $localPath = "$localDir/{$item['name']}";
        
        if ($item['is_dir']) {
            if ($item['name'] !== '.' && $item['name'] !== '..') {
                ftpDownloadDir($ftp, $remotePath, $localPath);
            }
        } else {
            echo "Downloading: $remotePath\n";
            ftp_get($ftp, $localPath, $remotePath, FTP_BINARY);
        }
    }
}

ftpDownloadDir($ftp, '/remote/site', '/local/backup');
```

### Recursive Upload

```php
<?php
declare(strict_types=1);

function ftpUploadDir(
    \FTP\Connection $ftp,
    string $localDir,
    string $remoteDir
): void {
    // Ensure remote directory exists
    @ftp_mkdir($ftp, $remoteDir);
    
    $iterator = new RecursiveIteratorIterator(
        new RecursiveDirectoryIterator($localDir, RecursiveDirectoryIterator::SKIP_DOTS),
        RecursiveIteratorIterator::SELF_FIRST
    );
    
    foreach ($iterator as $file) {
        /** @var SplFileInfo $file */
        $relativePath = $iterator->getSubPathname();
        $remotePath = "$remoteDir/$relativePath";
        
        if ($file->isDir()) {
            @ftp_mkdir($ftp, $remotePath);
        } else {
            ftp_put($ftp, $remotePath, $file->getPathname(), FTP_BINARY);
        }
    }
}

ftpUploadDir($ftp, '/local/site', '/remote/backup');
```

### Recursive Delete

```php
<?php
declare(strict_types=1);

function ftpDeleteDir(\FTP\Connection $ftp, string $remoteDir): void
{
    $items = ftp_nlist($ftp, $remoteDir);
    
    foreach ($items as $item) {
        if (in_array(basename($item), ['.', '..'])) continue;
        
        // Try to delete as file; if it's a directory, recurse
        if (!ftp_delete($ftp, $item)) {
            ftpDeleteDir($ftp, $item);
        }
    }
    
    ftp_rmdir($ftp, $remoteDir);
}

// Usage
ftpDeleteDir($ftp, '/remote/old-backup');
```

## FTP Stream Wrapper

```php
<?php
declare(strict_types=1);

// ftp:// stream wrapper — read/write files without FTP functions
// Format: ftp://user:pass@host/path

// Read remote file
$content = file_get_contents('ftp://user:pass@ftp.example.com/remote/file.txt');

// Write to remote file
file_put_contents(
    'ftp://user:pass@ftp.example.com/remote/new-file.txt',
    'Content to upload'
);

// With passive mode
$opts = [
    'ftp' => [
        'overwrite' => true,
        'passive' => true,
    ],
];
$context = stream_context_create($opts);
$content = file_get_contents(
    'ftp://user:pass@ftp.example.com/remote/file.txt',
    false,
    $context
);

// FTPS stream wrapper (PHP 8.0+)
// Use ftps:// scheme
$content = file_get_contents('ftps://user:pass@ftp.example.com/remote/file.txt');

// Directory listing via stream wrapper
$files = scandir('ftp://user:pass@ftp.example.com/remote/');

// Include a remote PHP file (discouraged for security)
// include 'ftp://user:pass@ftp.example.com/remote/lib.php';
```

## Advanced Operations

### File Permissions & Timestamps

```php
<?php
declare(strict_types=1);

// Change permissions (chmod)
ftp_chmod($ftp, 0644, '/remote/file.txt');

// Get file modification time
$mtime = ftp_mdtm($ftp, '/remote/file.txt');
if ($mtime !== -1) {
    echo "Last modified: " . date('Y-m-d H:i:s', $mtime);
}

// Get file size
$size = ftp_size($ftp, '/remote/large-file.zip');
if ($size !== -1) {
    echo "Size: " . number_format($size) . " bytes\n";
}

// Site-specific commands
ftp_site($ftp, 'CHMOD 644 file.txt');
ftp_site($ftp, 'UMASK 022');
```

### Multiple File Operations

```php
<?php
declare(strict_types=1);

// Sync local files to remote (mirror)
function ftpSyncDir(
    \FTP\Connection $ftp,
    string $localDir,
    string $remoteDir
): array {
    $synced = 0;
    $skipped = 0;
    
    $iterator = new RecursiveIteratorIterator(
        new RecursiveDirectoryIterator($localDir, FilesystemIterator::SKIP_DOTS)
    );
    
    foreach ($iterator as $file) {
        /** @var SplFileInfo $file */
        $relative = $iterator->getSubPathname();
        $remotePath = "$remoteDir/$relative";
        $localMod = $file->getMTime();
        
        // Create subdirectories
        $remoteSubdir = dirname($remotePath);
        @ftp_mkdir($ftp, $remoteSubdir);
        
        // Check if file needs updating
        $remoteMod = ftp_mdtm($ftp, $remotePath);
        
        if ($remoteMod !== -1 && abs($remoteMod - $localMod) < 60) {
            $skipped++;
            continue; // Already up-to-date
        }
        
        ftp_put($ftp, $remotePath, $file->getPathname(), FTP_BINARY);
        $synced++;
    }
    
    return ['synced' => $synced, 'skipped' => $skipped];
}
```

## Error Handling

```php
<?php
declare(strict_types=1);

// Get last error message
$error = ftp_get_option($ftp, FTP_TIMEOUT_SEC); // Not an error function

// Manual error tracking
$errors = [];
$ftp = @ftp_connect('ftp.example.com');
if ($ftp === false) {
    $error = error_get_last();
    throw new \RuntimeException(
        "FTP connect failed: " . ($error['message'] ?? 'Unknown error')
    );
}

// Suppress warnings with @ and use return value checks
$result = @ftp_get($ftp, $local, $remote, FTP_BINARY);
if ($result === false) {
    $error = error_get_last();
    echo "FTP error: " . ($error['message'] ?? 'Operation failed') . "\n";
}

// FTP connection type hint (PHP 8.1+)
// \FTP\Connection is the type for ftp connections
function getConnectionInfo(\FTP\Connection $ftp): array {
    return [
        'timeout' => ftp_get_option($ftp, FTP_TIMEOUT_SEC),
    ];
}
```

## Common Pitfalls

1. **Passive mode required behind NAT** — Most servers behind firewalls need `ftp_pasv($ftp, true)`. Active mode fails when the server can't connect back.
2. **Timeout on large transfers** — `ftp_get_option($ftp, FTP_TIMEOUT_SEC)` defaults to 90s. Increase for large files.
3. **`ftp_nlist()` returns basename only vs full path** — Behavior varies by FTP server. Use `ftp_rawlist()` and parse it for reliable results.
4. **SSL certificate validation** — FTPS (`ftp_ssl_connect()`) may fail on self-signed certs. PHP 8.2+ allows `ftp_set_option($ftp, FTP_SSL_VERIFY, false)` to disable verification (not recommended for production).
5. **`ftp_size()` returns -1 for directories** — Can't use to check existence of directories. Use `ftp_chdir()` / `ftp_pwd()` instead.
6. **ASCII vs Binary mode** — Use `FTP_BINARY` for everything except plain text files you want server-side line-ending conversion. `FTP_AUTOSELECT` (PHP 8.0+) handles it automatically.
7. **`ftp://` stream wrapper limitations** — No passive mode control without stream context. Less reliable than using the FTP functions directly.
8. **PORT command (active mode) with load balancers** — The server's data connection IP may differ from the control connection IP. Set `ftp_set_option($ftp, FTP_USEPASVADDRESS, false)` when behind a NAT.

## References

- [PHP: FTP](https://www.php.net/manual/en/book.ftp.php)
- [PHP: FTP Functions](https://www.php.net/manual/en/ref.ftp.php)
- [PHP: ftp:// Wrapper](https://www.php.net/manual/en/wrappers.ftp.php)
- [RFC 959: File Transfer Protocol](https://datatracker.ietf.org/doc/html/rfc959)
- [RFC 4217: FTP over TLS](https://datatracker.ietf.org/doc/html/rfc4217)
