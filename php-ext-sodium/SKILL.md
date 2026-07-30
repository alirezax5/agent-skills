---
name: php-sodium
description: Sodium (libsodium) — authenticated encryption, secret-key and public-key cryptography, hashing, key exchange, signatures, password hashing, key derivation, PES
php_version: 8.4
tags:
  - php
  - sodium
  - cryptography
  - encryption
  - libsodium
  - security
  - aead
  - ed25519
  - x25519
---

# Sodium (libsodium) — Modern Cryptography in PHP

## Overview

Sodium is a modern, easy-to-use cryptographic library that provides authenticated encryption, public-key cryptography, hashing, digital signatures, and password hashing. It's designed to be fast, secure, and hard to misuse. Sodium has been bundled with PHP since 7.2 as a core extension.

**Core design principle:** High-level operations with safe defaults. You don't choose ciphers, modes, or key sizes — you choose the **operation** and Sodium handles the rest.

## Secret-Key Cryptography (Symmetric)

### Authenticated Encryption (`secretbox`)

XSalsa20-Poly1305 — encrypts AND authenticates in one operation.

```php
<?php
declare(strict_types=1);

// Key generation
$key = sodium_crypto_secretbox_keygen();   // 32 random bytes

// Encryption
$message = 'This is a secret message';
$nonce = random_bytes(SODIUM_CRYPTO_SECRETBOX_NONCEBYTES); // 24 bytes
$ciphertext = sodium_crypto_secretbox($message, $nonce, $key);

// Decryption
$decrypted = sodium_crypto_secretbox_open($ciphertext, $nonce, $key);
if ($decrypted === false) {
    throw new \RuntimeException('Decryption failed — data tampered or wrong key');
}

// Always zero sensitive data when done
sodium_memzero($key);
```

## Public-Key Cryptography (Asymmetric)

### Key Generation

```php
<?php
declare(strict_types=1);

$keypair = sodium_crypto_box_keypair();
$secret_key = sodium_crypto_box_secretkey($keypair);  // 32 bytes
$public_key = sodium_crypto_box_publickey($keypair);  // 32 bytes
```

### Anonymous Encryption (Sealed Box)

```php
<?php
declare(strict_types=1);

$bob_keypair = sodium_crypto_box_keypair();
$bob_public = sodium_crypto_box_publickey($bob_keypair);

$sealed = sodium_crypto_box_seal('Hello Bob', $bob_public);
$decrypted = sodium_crypto_box_seal_open($sealed, $bob_keypair);
// Sealed box does NOT prove sender identity
```

### Authenticated Public-Key Encryption (Box)

```php
<?php
declare(strict_types=1);

$alice_kp = sodium_crypto_box_keypair();
$bob_kp = sodium_crypto_box_keypair();

$nonce = random_bytes(SODIUM_CRYPTO_BOX_NONCEBYTES);
$encrypted = sodium_crypto_box(
    'Hello Bob',
    $nonce,
    sodium_crypto_box_publickey($bob_kp),
    sodium_crypto_box_secretkey($alice_kp)
);

$decrypted = sodium_crypto_box_open(
    $encrypted, $nonce,
    sodium_crypto_box_publickey($alice_kp),
    sodium_crypto_box_secretkey($bob_kp)
);
```

## Digital Signatures (Ed25519)

```php
<?php
declare(strict_types=1);

$kp = sodium_crypto_sign_keypair();
$secret_key = sodium_crypto_sign_secretkey($kp);
$public_key = sodium_crypto_sign_publickey($kp);

// Detached signature
$signature = sodium_crypto_sign_detached('message', $secret_key);
$valid = sodium_crypto_sign_verify_detached($signature, 'message', $public_key);

// Combined signature
$signed = sodium_crypto_sign('message', $secret_key);
$original = sodium_crypto_sign_open($signed, $public_key);
```

## Password Hashing (Argon2)

```php
<?php
declare(strict_types=1);

$hash = sodium_crypto_pwhash_str(
    'user-password',
    SODIUM_CRYPTO_PWHASH_OPSLIMIT_MODERATE,   // 2 iterations
    SODIUM_CRYPTO_PWHASH_MEMLIMIT_MODERATE    // 256 MB
);

$valid = sodium_crypto_pwhash_str_verify($hash, 'user-password');

// Rehash needed?
if (sodium_crypto_pwhash_str_needs_rehash($hash, SODIUM_CRYPTO_PWHASH_OPSLIMIT_INTERACTIVE, SODIUM_CRYPTO_PWHASH_MEMLIMIT_INTERACTIVE)) {
    $newHash = sodium_crypto_pwhash_str('user-password', SODIUM_CRYPTO_PWHASH_OPSLIMIT_INTERACTIVE, SODIUM_CRYPTO_PWHASH_MEMLIMIT_INTERACTIVE);
}
```

## AEAD (Authenticated Encryption with Associated Data)

### XChaCha20-Poly1305 (Recommended)

```php
<?php
declare(strict_types=1);

$key = sodium_crypto_aead_xchacha20poly1305_ietf_keygen();
$nonce = random_bytes(SODIUM_CRYPTO_AEAD_XCHACHA20POLY1305_IETF_NPUBBYTES); // 24 bytes

$ciphertext = sodium_crypto_aead_xchacha20poly1305_ietf_encrypt(
    'Secret message', 'public-header-data', $nonce, $key
);

$decrypted = sodium_crypto_aead_xchacha20poly1305_ietf_decrypt(
    $ciphertext, 'public-header-data', $nonce, $key
);
```

### AES-256-GCM (Hardware-accelerated)

```php
<?php
declare(strict_types=1);

if (sodium_crypto_aead_aes256gcm_is_available()) {
    $key = sodium_crypto_aead_aes256gcm_keygen();
    $nonce = random_bytes(SODIUM_CRYPTO_AEAD_AES256GCM_NPUBBYTES); // 12 bytes
    $ciphertext = sodium_crypto_aead_aes256gcm_encrypt($message, $ad, $nonce, $key);
    $decrypted = sodium_crypto_aead_aes256gcm_decrypt($ciphertext, $ad, $nonce, $key);
}
```

## Common Pitfalls

1. **`sodium_memzero()` doesn't work on function return values** — Store in a variable first.
2. **Nonce reuse is catastrophic** — Never reuse a nonce with the same key.
3. **`sodium_crypto_box_seal()` provides no sender authentication** — Use `sodium_crypto_box()` for authenticated communication.
4. **Missing `catch` on `sodium_crypto_secretbox_open()` returning `false`** — Always check `=== false`.
5. **Storing keys in source code** — Use environment variables or key management services.
6. **Comparing MACs manually** — Never use `===` for authentication tags. Use `sodium_crypto_auth_verify()`.
7. **AES-256-GCM not available** — Check `sodium_crypto_aead_aes256gcm_is_available()` first.

## References

- [PHP: Sodium](https://www.php.net/manual/en/book.sodium.php)
- [Libsodium Documentation](https://doc.libsodium.org/)
- [Libsodium Quick Reference](https://doc.libsodium.org/quickstart)
- [Sodium Compat](https://github.com/paragonie/sodium_compat) — Pure PHP polyfill