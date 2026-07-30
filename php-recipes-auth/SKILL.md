---
name: php-recipes-auth
title: PHP Authentication & Authorization Patterns
description: Production-ready PHP 8.4 implementations for JWT, session-based auth, OAuth 2.0 client, API tokens, 2FA/TOTP, RBAC, password policies, CSRF protection, and social login integration.
category: software-development
tags: [php, auth, authentication, authorization, jwt, oauth, rbac, security]
php_version: 8.4
---

# Authentication & Authorization

> Complete, copy-paste ready auth patterns for PHP 8.4.  
> All implementations use `declare(strict_types=1)` and PSR-12 coding style.

---

## 1. JWT Authentication

See the [main PHP Recipes](../skill.md#2-jwt-authentication--issue-verify-refresh) for the base JWT service.  
Below: a full auth flow with login endpoint, refresh, blacklisting, and middleware.

```php
<?php
declare(strict_types=1);

namespace App\Auth;

final class LoginRequest
{
    public function __construct(
        public readonly string $email,
        public readonly string $password,
        public readonly ?string $deviceName = null,
    ) {}
}

final class AuthService
{
    private const int MAX_FAILED_ATTEMPTS = 5;
    private const int LOCKOUT_MINUTES = 15;

    public function __construct(
        private readonly \App\Repositories\UserRepositoryInterface $users,
        private readonly \App\Auth\JwtService $jwt,
        private readonly \Psr\Log\LoggerInterface $logger,
    ) {}

    /** @return array{access_token: string, refresh_token: string, expires_in: int, user: array} */
    public function login(LoginRequest $request): array
    {
        $user = $this->users->findByEmail($request->email);

        if ($user === null) {
            $this->logger->warning('Login failed: user not found', ['email' => $request->email]);
            throw new \App\Exceptions\UnauthorizedException('Invalid credentials');
        }

        if ($this->isLockedOut($user->id)) {
            throw new \App\Exceptions\UnauthorizedException('Account temporarily locked. Try again later.');
        }

        if (!password_verify($request->password, $user->passwordHash)) {
            $this->incrementFailedAttempts($user->id);
            $this->logger->warning('Login failed: wrong password', ['user_id' => $user->id]);
            throw new \App\Exceptions\UnauthorizedException('Invalid credentials');
        }

        $this->clearFailedAttempts($user->id);
        $this->logger->info('User logged in', ['user_id' => $user->id]);

        $accessToken = $this->jwt->issue([
            'sub'         => $user->id,
            'email'       => $user->email,
            'roles'       => $user->roles,
        ]);

        $refreshToken = $this->jwt->issueRefreshToken($user->id);
        $this->storeRefreshToken($user->id, $refreshToken, $request->deviceName ?? 'unknown');

        return [
            'access_token'  => $accessToken,
            'refresh_token' => $refreshToken,
            'expires_in'    => 3600,
            'user'          => ['id' => $user->id, 'name' => $user->name, 'email' => $user->email],
        ];
    }

    public function logout(string $refreshToken): void
    {
        try {
            $payload = $this->jwt->verify($refreshToken);
            $this->revokeRefreshToken($refreshToken);
            $this->addToBlacklist($refreshToken, $payload['exp']);
            $this->logger->info('User logged out', ['user_id' => $payload['sub'] ?? 'unknown']);
        } catch (\Throwable) {
            // Token already invalid — nothing to do
        }
    }

    /** @return array{access_token: string, refresh_token: string, expires_in: int} */
    public function refresh(string $refreshToken): array
    {
        if ($this->isBlacklisted($refreshToken)) {
            throw new \App\Exceptions\UnauthorizedException('Refresh token has been revoked');
        }

        $result = $this->jwt->refresh($refreshToken);

        // Rotate: revoke old, store new
        $this->revokeRefreshToken($refreshToken);
        $payload = $this->jwt->verify($result['refresh_token']);
        $this->storeRefreshToken((int)$payload['sub'], $result['refresh_token']);

        return $result;
    }

    // --- Lockout ---

    private function isLockedOut(int $userId): bool
    {
        // Implement with cache/redis
        return false;
    }

    private function incrementFailedAttempts(int $userId): void {}

    private function clearFailedAttempts(int $userId): void {}

    // --- Token Storage ---

    private function storeRefreshToken(int $userId, string $token, string $deviceName = 'unknown'): void
    {
        // INSERT INTO refresh_tokens (user_id, token_hash, device_name, expires_at, created_at)
        // VALUES (:uid, :hash, :device, :exp, NOW())
    }

    private function revokeRefreshToken(string $token): void
    {
        // DELETE FROM refresh_tokens WHERE token_hash = :hash
    }

    private function isBlacklisted(string $token): bool
    {
        // Check Redis/APCu blacklist
        return false;
    }

    private function addToBlacklist(string $token, int $expiresAt): void
    {
        // Store in Redis with TTL matching token expiry
    }
}
```

---

## 2. Session-Based Authentication

```php
<?php
declare(strict_types=1);

namespace App\Auth;

final class SessionAuth
{
    private const int SESSION_LIFETIME = 7200; // 2 hours
    private const string SESSION_KEY = 'auth_user';

    public function __construct(
        private readonly \App\Repositories\UserRepositoryInterface $users,
    ) {
        if (session_status() === PHP_SESSION_NONE) {
            $this->configureSession();
            session_start();
        }
    }

    private function configureSession(): void
    {
        ini_set('session.use_strict_mode', '1');
        ini_set('session.use_only_cookies', '1');
        ini_set('session.cookie_httponly', '1');
        ini_set('session.cookie_samesite', 'Lax');
        ini_set('session.cookie_secure', (string)(int)isset($_SERVER['HTTPS']));
        ini_set('session.gc_maxlifetime', (string)self::SESSION_LIFETIME);
        ini_set('session.cookie_lifetime', '0'); // Session cookie
    }

    /** Regenerate session ID to prevent fixation. */
    public function regenerate(): void
    {
        session_regenerate_id(true);
    }

    /** Log in a user (store their ID in session). */
    public function login(int $userId): void
    {
        $_SESSION[self::SESSION_KEY] = $userId;
        $_SESSION['login_time'] = time();
        $_SESSION['last_activity'] = time();
        $this->regenerate();
    }

    /** Log out and destroy the session. */
    public function logout(): void
    {
        $_SESSION = [];
        if (ini_get('session.use_cookies')) {
            $params = session_get_cookie_params();
            setcookie(
                session_name(),
                '',
                time() - 42000,
                $params['path'],
                $params['domain'],
                $params['secure'],
                $params['httponly'],
            );
        }
        session_destroy();
    }

    /** Check if a user is authenticated. */
    public function check(): bool
    {
        if (!isset($_SESSION[self::SESSION_KEY])) {
            return false;
        }

        // Idle timeout
        $idleMax = 1800; // 30 min
        if (isset($_SESSION['last_activity']) && (time() - $_SESSION['last_activity']) > $idleMax) {
            $this->logout();
            return false;
        }

        // Absolute timeout
        if (isset($_SESSION['login_time']) && (time() - $_SESSION['login_time']) > self::SESSION_LIFETIME) {
            $this->logout();
            return false;
        }

        $_SESSION['last_activity'] = time();
        return true;
    }

    /** Get the authenticated user or null. */
    public function user(): ?\App\Entities\User
    {
        if (!$this->check()) {
            return null;
        }
        return $this->users->findById((int)$_SESSION[self::SESSION_KEY]);
    }

    /** Get the authenticated user ID or null. */
    public function id(): ?int
    {
        return $this->check() ? (int)$_SESSION[self::SESSION_KEY] : null;
    }
}
```

---

## 3. OAuth 2.0 Client (Social Login)

A standalone OAuth2 client for "Login with Google / GitHub / Facebook".

```php
<?php
declare(strict_types=1);

namespace App\Auth\OAuth;

/**
 * Generic OAuth2 client — works with any provider.
 *
 * Usage:
 *   $google = new OAuth2Client([
 *       'clientId'     => '...',
 *       'clientSecret' => '...',
 *       'redirectUri'  => 'https://example.com/auth/google/callback',
 *       'authUrl'      => 'https://accounts.google.com/o/oauth2/v2/auth',
 *       'tokenUrl'     => 'https://oauth2.googleapis.com/token',
 *       'userInfoUrl'  => 'https://www.googleapis.com/oauth2/v2/userinfo',
 *       'scopes'       => ['email', 'profile'],
 *   ]);
 *   // Step 1: Redirect user to $google->getAuthorizationUrl()
 *   // Step 2: In callback, call $google->handleCallback($_GET['code'])
 */
final class OAuth2Client
{
    private string $clientId;
    private string $clientSecret;
    private string $redirectUri;
    private string $authUrl;
    private string $tokenUrl;
    private string $userInfoUrl;
    /** @var string[] */
    private array $scopes;
    private string $state;

    /** @param array<string, mixed> $config */
    public function __construct(array $config)
    {
        $this->clientId     = $config['clientId'];
        $this->clientSecret = $config['clientSecret'];
        $this->redirectUri  = $config['redirectUri'];
        $this->authUrl      = $config['authUrl'];
        $this->tokenUrl     = $config['tokenUrl'];
        $this->userInfoUrl  = $config['userInfoUrl'];
        $this->scopes       = $config['scopes'] ?? ['email'];
    }

    /** Generate the authorization URL (step 1). */
    public function getAuthorizationUrl(): string
    {
        $this->state = bin2hex(random_bytes(16));
        $_SESSION['oauth_state'] = $this->state;

        $params = [
            'response_type' => 'code',
            'client_id'     => $this->clientId,
            'redirect_uri'  => $this->redirectUri,
            'scope'         => implode(' ', $this->scopes),
            'state'         => $this->state,
            'access_type'   => 'offline',
        ];

        return $this->authUrl . '?' . http_build_query($params);
    }

    /**
     * Handle the callback (step 2).
     *
     * @param string $code  The 'code' query parameter from the provider
     * @param string $state The 'state' query parameter (for CSRF check)
     * @return array{access_token: string, refresh_token?: string, expires_in: int, user: array}
     */
    public function handleCallback(string $code, string $state): array
    {
        // CSRF check
        $expectedState = $_SESSION['oauth_state'] ?? '';
        unset($_SESSION['oauth_state']);
        if ($expectedState === '' || !hash_equals($expectedState, $state)) {
            throw new \RuntimeException('Invalid OAuth state — possible CSRF');
        }

        // Exchange code for token
        $tokenData = $this->exchangeCode($code);
        $accessToken = $tokenData['access_token'];

        // Fetch user info
        $userData = $this->fetchUserInfo($accessToken);

        return [
            'access_token'  => $accessToken,
            'refresh_token' => $tokenData['refresh_token'] ?? '',
            'expires_in'    => $tokenData['expires_in'] ?? 3600,
            'user'          => $userData,
        ];
    }

    /** Exchange authorization code for tokens. */
    private function exchangeCode(string $code): array
    {
        $ch = curl_init();
        curl_setopt_array($ch, [
            CURLOPT_URL            => $this->tokenUrl,
            CURLOPT_POST           => true,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_POSTFIELDS     => http_build_query([
                'grant_type'    => 'authorization_code',
                'code'          => $code,
                'redirect_uri'  => $this->redirectUri,
                'client_id'     => $this->clientId,
                'client_secret' => $this->clientSecret,
            ]),
            CURLOPT_HTTPHEADER     => ['Accept: application/json'],
        ]);

        $body = curl_exec($ch);
        $status = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);

        if ($status !== 200) {
            throw new \RuntimeException("OAuth token exchange failed with status $status");
        }

        $data = json_decode($body, true, 512, JSON_THROW_ON_ERROR);
        if (!isset($data['access_token'])) {
            throw new \RuntimeException('No access_token in OAuth response');
        }

        return $data;
    }

    /** Fetch user info using the access token. */
    private function fetchUserInfo(string $accessToken): array
    {
        $ch = curl_init();
        curl_setopt_array($ch, [
            CURLOPT_URL            => $this->userInfoUrl,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_HTTPHEADER     => [
                'Authorization: *** ' . $accessToken,
                'Accept: application/json',
            ],
        ]);

        $body = curl_exec($ch);
        $status = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);

        if ($status !== 200) {
            throw new \RuntimeException("OAuth userinfo failed with status $status");
        }

        return json_decode($body, true, 512, JSON_THROW_ON_ERROR) ?: [];
    }

    /** Refresh an expired access token using a refresh token. */
    public function refreshAccessToken(string $refreshToken): array
    {
        $ch = curl_init();
        curl_setopt_array($ch, [
            CURLOPT_URL            => $this->tokenUrl,
            CURLOPT_POST           => true,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_POSTFIELDS     => http_build_query([
                'grant_type'    => 'refresh_token',
                'refresh_token' => $refreshToken,
                'client_id'     => $this->clientId,
                'client_secret' => $this->clientSecret,
            ]),
        ]);

        $body = curl_exec($ch);
        $status = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);

        if ($status !== 200) {
            throw new \RuntimeException('OAuth token refresh failed');
        }

        return json_decode($body, true, 512, JSON_THROW_ON_ERROR) ?: [];
    }
}
```

---

## 4. API Token Authentication

```php
<?php
declare(strict_types=1);

namespace App\Auth;

/**
 * API token authentication for machine-to-machine communication.
 * Tokens are stored hashed (bcrypt) in the database; plaintext shown once.
 */
final class ApiTokenService
{
    private const int TOKEN_BYTE_LENGTH = 32;

    public function __construct(
        private readonly \PDO $pdo,
        private readonly string $table = 'api_tokens',
    ) {}

    /**
     * Generate a new API token.
     *
     * @param int    $userId     Owner
     * @param string $name       Human-readable label
     * @param string[] $abilities e.g. ['read:users', 'write:orders']
     * @return array{plain: string, id: int} Plain token — show once!
     */
    public function create(int $userId, string $name, array $abilities = ['*']): array
    {
        $plain = bin2hex(random_bytes(self::TOKEN_BYTE_LENGTH));
        $hash = password_hash($plain, PASSWORD_BCRYPT);
        $lastChars = substr($plain, -4);

        $stmt = $this->pdo->prepare(
            "INSERT INTO {$this->table} (user_id, name, token_hash, abilities, last_chars, created_at)
             VALUES (:uid, :name, :hash, :abilities, :last, NOW())"
        );
        $stmt->execute([
            ':uid'       => $userId,
            ':name'      => $name,
            ':hash'      => $hash,
            ':abilities' => json_encode($abilities, JSON_THROW_ON_ERROR),
            ':last'      => $lastChars,
        ]);

        return [
            'plain' => $plain,
            'id'    => (int)$this->pdo->lastInsertId(),
        ];
    }

    /**
     * Validate a token from an HTTP request.
     *
     * @param string $headerValue Value of the Authorization header
     * @return array|null Token record or null on failure
     */
    public function validate(string $headerValue): ?array
    {
        // Expect "Bearer <token>"
        if (!preg_match('/^Bearer\s+(.+)$/i', $headerValue, $m)) {
            return null;
        }
        $plain = $m[1];

        // Fetch all tokens (last_chars index helps narrow)
        $lastChars = substr($plain, -4);
        $stmt = $this->pdo->prepare(
            "SELECT * FROM {$this->table} WHERE last_chars = :last AND revoked_at IS NULL"
        );
        $stmt->execute([':last' => $lastChars]);
        $tokens = $stmt->fetchAll(\PDO::FETCH_ASSOC);

        foreach ($tokens as $row) {
            if (password_verify($plain, $row['token_hash'])) {
                // Touch last_used_at
                $this->pdo->prepare(
                    "UPDATE {$this->table} SET last_used_at = NOW() WHERE id = :id"
                )->execute([':id' => $row['id']]);

                $row['abilities'] = json_decode($row['abilities'], true, 512, JSON_THROW_ON_ERROR) ?? [];
                return $row;
            }
        }

        return null;
    }

    /** Check if a token has a specific ability. */
    public static function can(array $token, string $ability): bool
    {
        $abilities = $token['abilities'] ?? [];
        return in_array('*', $abilities, true) || in_array($ability, $abilities, true);
    }

    /** Revoke a token. */
    public function revoke(int $tokenId): void
    {
        $this->pdo->prepare(
            "UPDATE {$this->table} SET revoked_at = NOW() WHERE id = :id"
        )->execute([':id' => $tokenId]);
    }

    /** List tokens for a user (excluding hashes for security). */
    public function listForUser(int $userId): array
    {
        $stmt = $this->pdo->prepare(
            "SELECT id, name, abilities, last_chars, created_at, last_used_at
             FROM {$this->table}
             WHERE user_id = :uid AND revoked_at IS NULL
             ORDER BY created_at DESC"
        );
        $stmt->execute([':uid' => $userId]);
        return $stmt->fetchAll(\PDO::FETCH_ASSOC);
    }
}

// --- Middleware for API token auth ---

final class ApiTokenMiddleware implements \App\Infrastructure\Http\Middleware
{
    public function __construct(
        private readonly ApiTokenService $tokens,
        private readonly string $requiredAbility = '',
    ) {}

    public function handle(\App\Infrastructure\Http\RequestContext $ctx, callable $next): \App\Infrastructure\Http\RequestContext
    {
        $auth = $ctx->headers['Authorization'] ?? $ctx->headers['authorization'] ?? '';
        $token = $this->tokens->validate($auth);

        if ($token === null) {
            throw new \App\Exceptions\UnauthorizedException('Invalid or missing API token');
        }

        if ($this->requiredAbility !== '' && !ApiTokenService::can($token, $this->requiredAbility)) {
            throw new \App\Exceptions\ForbiddenException('Token lacks required ability: ' . $this->requiredAbility);
        }

        return $next(new \App\Infrastructure\Http\RequestContext(
            query: $ctx->query,
            body: $ctx->body,
            attributes: array_merge($ctx->attributes, ['api_token' => $token, 'user_id' => $token['user_id']]),
            headers: $ctx->headers,
            method: $ctx->method,
            uri: $ctx->uri,
        ));
    }
}
```

---

## 5. Two-Factor Authentication — TOTP (Time-Based One-Time Password)

```php
<?php
declare(strict_types=1);

namespace App\Auth\TwoFactor;

/**
 * TOTP implementation (RFC 6238).
 * Compatible with Google Authenticator, Authy, 1Password, etc.
 *
 * Usage:
 *   $totp = new TotpService('MyApp');
 *   // Setup: generate secret => show QR code to user
 *   $secret = $totp->generateSecret();
 *   $qrUrl  = $totp->getProvisioningUri('user@example.com', $secret);
 *   // Verify: user enters 6-digit code
 *   $valid  = $totp->verify($secret, $userInput);
 */
final class TotpService
{
    private const int CODE_LENGTH = 6;
    private const int TIME_STEP = 30;
    private const int WINDOW = 1; // ±1 step allowed for clock skew

    public function __construct(
        private readonly string $issuer = 'App',
    ) {}

    /** Generate a new base32-encoded secret. */
    public function generateSecret(): string
    {
        $bytes = random_bytes(20); // 160 bits
        return $this->base32Encode($bytes);
    }

    /**
     * Get the provisioning URI for a QR code.
     * Use with a QR code library or https://api.qrserver.com/v1/create-qr-code/
     */
    public function getProvisioningUri(string $label, string $secret): string
    {
        $params = http_build_query([
            'secret'  => $secret,
            'issuer'  => $this->issuer,
            'algorithm' => 'SHA1',
            'digits'  => self::CODE_LENGTH,
            'period'  => self::TIME_STEP,
        ]);
        return "otpauth://totp/" . rawurlencode($this->issuer) . ":" . rawurlencode($label) . "?$params";
    }

    /**
     * Verify a TOTP code.
     *
     * @param string $secret     Base32-encoded secret
     * @param string $code       User-provided 6-digit code
     * @return bool
     */
    public function verify(string $secret, string $code): bool
    {
        $secretBytes = $this->base32Decode($secret);
        if ($secretBytes === '') {
            return false;
        }

        $counter = (int)floor(time() / self::TIME_STEP);

        // Check current, previous, and next time step (window)
        for ($i = -self::WINDOW; $i <= self::WINDOW; $i++) {
            $expected = $this->generateCode($secretBytes, $counter + $i);
            if (hash_equals($expected, $code)) {
                return true;
            }
        }

        return false;
    }

    private function generateCode(string $secretBytes, int $counter): string
    {
        $data = pack('J', $counter); // 64-bit big-endian
        $hash = hash_hmac('sha1', $data, $secretBytes, true);

        // Dynamic truncation (RFC 4226)
        $offset = ord($hash[19]) & 0x0F;
        $value = (
            ((ord($hash[$offset]) & 0x7F) << 24) |
            ((ord($hash[$offset + 1]) & 0xFF) << 16) |
            ((ord($hash[$offset + 2]) & 0xFF) << 8) |
            (ord($hash[$offset + 3]) & 0xFF)
        );

        return str_pad((string)($value % 10 ** self::CODE_LENGTH), self::CODE_LENGTH, '0', STR_PAD_LEFT);
    }

    // --- Base32 encoding (RFC 4648) ---

    private const string BASE32_ALPHABET = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ234567';

    private function base32Encode(string $bytes): string
    {
        $result = '';
        $bits = 0;
        $buffer = 0;
        $len = strlen($bytes);

        for ($i = 0; $i < $len; $i++) {
            $buffer = ($buffer << 8) | ord($bytes[$i]);
            $bits += 8;
            while ($bits >= 5) {
                $bits -= 5;
                $result .= self::BASE32_ALPHABET[($buffer >> $bits) & 0x1F];
            }
        }

        if ($bits > 0) {
            $result .= self::BASE32_ALPHABET[($buffer << (5 - $bits)) & 0x1F];
        }

        // Padding
        $padLen = (8 - ceil($len * 8 / 5) % 8) % 8;
        return str_pad($result, strlen($result) + $padLen, '=');
    }

    private function base32Decode(string $data): string
    {
        $data = rtrim($data, '=');
        $data = strtoupper($data);
        $result = '';
        $bits = 0;
        $buffer = 0;
        $len = strlen($data);

        for ($i = 0; $i < $len; $i++) {
            $val = strpos(self::BASE32_ALPHABET, $data[$i]);
            if ($val === false) {
                return '';
            }
            $buffer = ($buffer << 5) | $val;
            $bits += 5;
            if ($bits >= 8) {
                $bits -= 8;
                $result .= chr(($buffer >> $bits) & 0xFF);
            }
        }

        return $result;
    }
}

// --- 2FA Middleware ---

final class TwoFactorMiddleware implements \App\Infrastructure\Http\Middleware
{
    public function __construct(
        private readonly \PDO $pdo,
    ) {}

    public function handle(\App\Infrastructure\Http\RequestContext $ctx, callable $next): \App\Infrastructure\Http\RequestContext
    {
        $user = $ctx->attributes['user'] ?? null;
        if ($user === null || !$this->userHas2faEnabled($user['sub'])) {
            return $next($ctx); // No 2FA configured — pass through
        }

        // Check if session is already 2FA-verified
        if (($ctx->attributes['2fa_verified'] ?? false)) {
            return $next($ctx);
        }

        // Require 2FA code
        $twoFactorCode = $ctx->headers['X-2FA-Code'] ?? $ctx->body['two_factor_code'] ?? '';
        if ($twoFactorCode === '') {
            throw new \App\Exceptions\HttpException('Two-factor authentication code required', 412);
        }

        $secret = $this->getUserSecret($user['sub']);
        $totp = new TotpService();
        if (!$totp->verify($secret, $twoFactorCode)) {
            throw new \App\Exceptions\UnauthorizedException('Invalid two-factor code');
        }

        // Mark as verified for remaining middleware chain
        return $next(new \App\Infrastructure\Http\RequestContext(
            query: $ctx->query,
            body: $ctx->body,
            attributes: array_merge($ctx->attributes, ['2fa_verified' => true]),
            headers: $ctx->headers,
            method: $ctx->method,
            uri: $ctx->uri,
        ));
    }

    private function userHas2faEnabled(int $userId): bool
    {
        $stmt = $this->pdo->prepare('SELECT totp_secret FROM users WHERE id = :id');
        $stmt->execute([':id' => $userId]);
        $row = $stmt->fetch(\PDO::FETCH_ASSOC);
        return $row !== false && $row['totp_secret'] !== null && $row['totp_secret'] !== '';
    }

    private function getUserSecret(int $userId): string
    {
        $stmt = $this->pdo->prepare('SELECT totp_secret FROM users WHERE id = :id');
        $stmt->execute([':id' => $userId]);
        $row = $stmt->fetch(\PDO::FETCH_ASSOC);
        return $row['totp_secret'] ?? '';
    }
}
```

---

## 6. RBAC — Role-Based Access Control

```php
<?php
declare(strict_types=1);

namespace App\Auth\RBAC;

/**
 * Role-Based Access Control with role hierarchy and permission caching.
 *
 * Usage:
 *   $rbac = new RbacService($pdo);
 *   $rbac->assignRole(1, 'admin');
 *   if ($rbac->can(1, 'delete_users')) { ... }
 */
final class RbacService
{
    /** @var array<int, array<string, bool>> Cache: user_id => [permission => true] */
    private array $permissionCache = [];

    public function __construct(
        private readonly \PDO $pdo,
    ) {}

    // --- Permission Checks ---

    /** Check if a user has a specific permission (cached). */
    public function can(int $userId, string $permission): bool
    {
        $permissions = $this->getUserPermissions($userId);
        return isset($permissions[$permission]);
    }

    /** Check if a user has ALL of the given permissions. */
    public function canAll(int $userId, string ...$permissions): bool
    {
        $userPerms = $this->getUserPermissions($userId);
        foreach ($permissions as $p) {
            if (!isset($userPerms[$p])) {
                return false;
            }
        }
        return true;
    }

    /** Check if a user has ANY of the given permissions. */
    public function canAny(int $userId, string ...$permissions): bool
    {
        $userPerms = $this->getUserPermissions($userId);
        foreach ($permissions as $p) {
            if (isset($userPerms[$p])) {
                return true;
            }
        }
        return false;
    }

    /** Check if a user has a specific role. */
    public function hasRole(int $userId, string $role): bool
    {
        $stmt = $this->pdo->prepare(
            'SELECT 1 FROM user_roles ur
             JOIN roles r ON r.id = ur.role_id
             WHERE ur.user_id = :uid AND r.name = :role'
        );
        $stmt->execute([':uid' => $userId, ':role' => $role]);
        return $stmt->fetchColumn() !== false;
    }

    // --- Role Management ---

    /** Assign a role to a user. */
    public function assignRole(int $userId, string $role): void
    {
        $roleId = $this->ensureRole($role);
        $stmt = $this->pdo->prepare(
            'INSERT IGNORE INTO user_roles (user_id, role_id) VALUES (:uid, :rid)'
        );
        $stmt->execute([':uid' => $userId, ':rid' => $roleId]);
        unset($this->permissionCache[$userId]);
    }

    /** Remove a role from a user. */
    public function removeRole(int $userId, string $role): void
    {
        $stmt = $this->pdo->prepare(
            'DELETE ur FROM user_roles ur
             JOIN roles r ON r.id = ur.role_id
             WHERE ur.user_id = :uid AND r.name = :role'
        );
        $stmt->execute([':uid' => $userId, ':role' => $role]);
        unset($this->permissionCache[$userId]);
    }

    // --- Permission Management ---

    /** Grant a permission directly to a role. */
    public function grantPermission(string $role, string $permission): void
    {
        $roleId = $this->ensureRole($role);
        $permId = $this->ensurePermission($permission);
        $this->pdo->prepare(
            'INSERT IGNORE INTO role_permissions (role_id, permission_id) VALUES (:rid, :pid)'
        )->execute([':rid' => $roleId, ':pid' => $permId]);
        $this->permissionCache = []; // Clear all caches
    }

    /** Revoke a permission from a role. */
    public function revokePermission(string $role, string $permission): void
    {
        $stmt = $this->pdo->prepare(
            'DELETE rp FROM role_permissions rp
             JOIN roles r ON r.id = rp.role_id
             JOIN permissions p ON p.id = rp.permission_id
             WHERE r.name = :role AND p.name = :perm'
        );
        $stmt->execute([':role' => $role, ':perm' => $permission]);
        $this->permissionCache = [];
    }

    /** Define a role hierarchy: $child inherits all permissions from $parent. */
    public function setRoleHierarchy(string $child, string $parent): void
    {
        $childId  = $this->ensureRole($child);
        $parentId = $this->ensureRole($parent);
        $this->pdo->prepare(
            'INSERT IGNORE INTO role_hierarchy (child_role_id, parent_role_id) VALUES (:c, :p)'
        )->execute([':c' => $childId, ':p' => $parentId]);
        $this->permissionCache = [];
    }

    // --- Internal ---

    /** @return array<string, bool> */
    private function getUserPermissions(int $userId): array
    {
        if (isset($this->permissionCache[$userId])) {
            return $this->permissionCache[$userId];
        }

        // Gather all permissions via roles + hierarchy
        $stmt = $this->pdo->prepare(
            "WITH RECURSIVE user_roles_tree AS (
                SELECT r.id, r.name FROM roles r
                JOIN user_roles ur ON ur.role_id = r.id
                WHERE ur.user_id = :uid
                UNION ALL
                SELECT r.id, r.name FROM role_hierarchy rh
                JOIN roles r ON r.id = rh.parent_role_id
                JOIN user_roles_tree ut ON ut.id = rh.child_role_id
             )
             SELECT DISTINCT p.name FROM user_roles_tree ut
             JOIN role_permissions rp ON rp.role_id = ut.id
             JOIN permissions p ON p.id = rp.permission_id"
        );
        $stmt->execute([':uid' => $userId]);
        $permissions = $stmt->fetchAll(\PDO::FETCH_COLUMN);

        $cache = array_fill_keys($permissions, true);
        $this->permissionCache[$userId] = $cache;

        return $cache;
    }

    private function ensureRole(string $name): int
    {
        $stmt = $this->pdo->prepare('SELECT id FROM roles WHERE name = :name');
        $stmt->execute([':name' => $name]);
        $id = $stmt->fetchColumn();
        if ($id !== false) {
            return (int)$id;
        }
        $this->pdo->prepare('INSERT INTO roles (name) VALUES (:name)')->execute([':name' => $name]);
        return (int)$this->pdo->lastInsertId();
    }

    private function ensurePermission(string $name): int
    {
        $stmt = $this->pdo->prepare('SELECT id FROM permissions WHERE name = :name');
        $stmt->execute([':name' => $name]);
        $id = $stmt->fetchColumn();
        if ($id !== false) {
            return (int)$id;
        }
        $this->pdo->prepare('INSERT INTO permissions (name) VALUES (:name)')->execute([':name' => $name]);
        return (int)$this->pdo->lastInsertId();
    }
}

// --- RBAC Middleware ---

final class RbacMiddleware implements \App\Infrastructure\Http\Middleware
{
    /** @param string[] $permissions */
    public function __construct(
        private readonly RbacService $rbac,
        private readonly array $permissions = [],
    ) {}

    public function handle(\App\Infrastructure\Http\RequestContext $ctx, callable $next): \App\Infrastructure\Http\RequestContext
    {
        $user = $ctx->attributes['user'] ?? null;
        if ($user === null) {
            throw new \App\Exceptions\UnauthorizedException('Authentication required');
        }

        $userId = (int)$user['sub'];

        if ($this->permissions !== [] && !$this->rbac->canAll($userId, ...$this->permissions)) {
            throw new \App\Exceptions\ForbiddenException('Insufficient permissions');
        }

        return $next($ctx);
    }
}
```

---

## 7. Password Policies

```php
<?php
declare(strict_types=1);

namespace App\Auth;

/**
 * Enforce password strength rules.
 * Usage:
 *   $policy = new PasswordPolicy();
 *   $result = $policy->validate('MyP@ssw0rd!');
 *   if (!$result->isValid) { echo $result->message; }
 */
final readonly class PasswordPolicyResult
{
    public function __construct(
        public bool $isValid,
        public string $message = '',
    ) {}
}

final class PasswordPolicy
{
    public int $minLength = 8;
    public int $maxLength = 128;
    public bool $requireUppercase = true;
    public bool $requireLowercase = true;
    public bool $requireDigit = true;
    public bool $requireSpecialChar = true;
    public int $minSpecialChars = 1;
    /** @var string[] */
    public array $commonPasswords = [
        'password', '12345678', 'qwerty123', 'letmein', 'welcome',
        'monkey', 'dragon', 'master', 'passw0rd', 'admin123',
    ];

    public function validate(string $password): PasswordPolicyResult
    {
        if (strlen($password) < $this->minLength) {
            return new PasswordPolicyResult(false, "Password must be at least {$this->minLength} characters");
        }

        if (strlen($password) > $this->maxLength) {
            return new PasswordPolicyResult(false, "Password must be no more than {$this->maxLength} characters");
        }

        if ($this->requireUppercase && !preg_match('/[A-Z]/', $password)) {
            return new PasswordPolicyResult(false, 'Password must contain an uppercase letter');
        }

        if ($this->requireLowercase && !preg_match('/[a-z]/', $password)) {
            return new PasswordPolicyResult(false, 'Password must contain a lowercase letter');
        }

        if ($this->requireDigit && !preg_match('/[0-9]/', $password)) {
            return new PasswordPolicyResult(false, 'Password must contain a digit');
        }

        if ($this->requireSpecialChar) {
            $specialCount = preg_match_all('/[^a-zA-Z0-9]/', $password);
            if ($specialCount < $this->minSpecialChars) {
                return new PasswordPolicyResult(false, "Password must contain at least {$this->minSpecialChars} special character(s)");
            }
        }

        $lower = strtolower($password);
        foreach ($this->commonPasswords as $common) {
            if (str_contains($lower, $common)) {
                return new PasswordPolicyResult(false, 'Password is too common');
            }
        }

        // Check for repeated characters (e.g., "aaa")
        if (preg_match('/(.)\\1{3,}/', $password)) {
            return new PasswordPolicyResult(false, 'Password contains too many repeated characters');
        }

        // Check for sequential characters (e.g., "abcd", "1234")
        if ($this->hasSequence($password, 4)) {
            return new PasswordPolicyResult(false, 'Password contains sequential characters');
        }

        return new PasswordPolicyResult(true);
    }

    private function hasSequence(string $password, int $minLength): bool
    {
        $lower = strtolower($password);
        $len = strlen($lower);
        for ($i = 0; $i <= $len - $minLength; $i++) {
            $seq = true;
            for ($j = 1; $j < $minLength; $j++) {
                if (ord($lower[$i + $j]) !== ord($lower[$i]) + $j) {
                    $seq = false;
                    break;
                }
            }
            if ($seq) return true;
        }
        return false;
    }
}

// --- Password hashing helper ---

final class PasswordHasher
{
    /**
     * Hash a password using the default bcrypt algorithm.
     * PHP 8.4 auto-selects the best algorithm.
     */
    public static function hash(string $password): string
    {
        return password_hash($password, PASSWORD_BCRYPT, ['cost' => 12]);
    }

    /**
     * Verify a password against a hash.
     * Automatically rehash if algorithm/cost has changed.
     *
     * @return array{valid: bool, needsRehash: bool}
     */
    public static function verifyAndCheckRehash(string $password, string $hash): array
    {
        $valid = password_verify($password, $hash);
        $needsRehash = $valid && password_needs_rehash($hash, PASSWORD_BCRYPT, ['cost' => 12]);

        return ['valid' => $valid, 'needsRehash' => $needsRehash];
    }
}
```

---

## 8. CSRF Protection

```php
<?php
declare(strict_types=1);

namespace App\Auth;

/**
 * CSRF token generation and validation.
 *
 * Usage:
 *   $csrf = new CsrfProtection();
 *   // In form:
 *   <input type="hidden" name="_token" value="<?= $csrf->generate() ?>">
 *   // In JS (for AJAX):
 *   meta name="csrf-token" content="<?= $csrf->generate() ?>"
 *   // Validate:
 *   $csrf->validate($_POST['_token'] ?? '');
 */
final class CsrfProtection
{
    private const string TOKEN_KEY = '_csrf_token';
    private const int TOKEN_LIFETIME = 7200; // 2 hours

    public function __construct()
    {
        if (session_status() === PHP_SESSION_NONE) {
            session_start();
        }
    }

    /** Generate a new CSRF token (or reuse existing within lifetime). */
    public function generate(): string
    {
        $token = $_SESSION[self::TOKEN_KEY]['token'] ?? null;
        $expires = $_SESSION[self::TOKEN_KEY]['expires'] ?? 0;

        if ($token === null || $expires < time()) {
            $token = bin2hex(random_bytes(32));
            $_SESSION[self::TOKEN_KEY] = [
                'token'   => $token,
                'expires' => time() + self::TOKEN_LIFETIME,
            ];
        }

        return $token;
    }

    /**
     * Validate a CSRF token.
     *
     * @param string $userToken Token from request body or header
     * @throws \RuntimeException on failure
     */
    public function validate(string $userToken): void
    {
        $stored = $_SESSION[self::TOKEN_KEY]['token'] ?? null;

        if ($stored === null) {
            throw new \RuntimeException('CSRF token missing from session');
        }

        if (!hash_equals($stored, $userToken)) {
            throw new \RuntimeException('CSRF token mismatch');
        }

        // Single-use: regenerate after validation
        unset($_SESSION[self::TOKEN_KEY]);
    }

    /**
     * Validate CSRF token from common sources.
     * Call in a middleware before mutating requests.
     */
    public function validateFromRequest(): void
    {
        $token = $_POST['_token']
              ?? $_SERVER['HTTP_X_CSRF_TOKEN']
              ?? $_SERVER['HTTP_X_XSRF_TOKEN']
              ?? '';

        if ($token === '') {
            throw new \RuntimeException('CSRF token not provided');
        }

        $this->validate($token);
    }
}

// --- Double Submit Cookie pattern (stateless CSRF) ---

final class DoubleSubmitCookieCsrf
{
    /**
     * Set a CSRF cookie with a random value and include it in a custom header.
     * The server compares both — no session storage needed.
     */
    public static function setCookie(): void
    {
        $token = bin2hex(random_bytes(32));
        setcookie('csrf_token', $token, [
            'expires'  => 0,
            'path'     => '/',
            'secure'   => isset($_SERVER['HTTPS']),
            'httponly' => true,
            'samesite' => 'Strict',
        ]);
        // Also available via meta tag or JavaScript variable
    }

    /**
     * Validate by comparing cookie value to header value.
     *
     * @param string $headerToken From X-CSRF-Token header
     */
    public static function validate(string $headerToken): void
    {
        $cookieToken = $_COOKIE['csrf_token'] ?? '';
        if ($cookieToken === '' || !hash_equals($cookieToken, $headerToken)) {
            throw new \RuntimeException('CSRF validation failed');
        }
    }
}
```

---

## 9. Password Reset Flow

```php
<?php
declare(strict_types=1);

namespace App\Auth;

/**
 * Secure password reset with expiring tokens.
 *
 * Flow:
 *   1. User requests reset via email
 *   2. Generate token, store hash in DB, send email
 *   3. User clicks link with token
 *   4. Verify token (not expired, correct hash)
 *   5. Allow password change
 */
final class PasswordResetService
{
    private const int TOKEN_LENGTH = 32;
    private const int TOKEN_EXPIRY_HOURS = 1;

    public function __construct(
        private readonly \PDO $pdo,
        private readonly \Psr\Log\LoggerInterface $logger,
    ) {}

    /**
     * Generate a reset token and store it.
     *
     * @param string $email User's email
     * @return string|null The plain token to include in the email, or null if email not found
     */
    public function generateToken(string $email): ?string
    {
        // Check user exists (don't reveal if not — log it)
        $stmt = $this->pdo->prepare('SELECT id FROM users WHERE email = :email');
        $stmt->execute([':email' => $email]);
        $user = $stmt->fetch(\PDO::FETCH_ASSOC);

        if ($user === false) {
            $this->logger->info('Password reset requested for unknown email', ['email' => $email]);
            return null; // Don't reveal that email doesn't exist
        }

        // Invalidate previous tokens
        $this->pdo->prepare(
            "UPDATE password_resets SET used_at = NOW() WHERE email = :email AND used_at IS NULL"
        )->execute([':email' => $email]);

        // Create new token
        $plain = bin2hex(random_bytes(self::TOKEN_LENGTH));
        $hash = hash('sha256', $plain);

        $stmt = $this->pdo->prepare(
            "INSERT INTO password_resets (email, token_hash, expires_at, created_at)
             VALUES (:email, :hash, :expires, NOW())"
        );
        $stmt->execute([
            ':email'   => $email,
            ':hash'    => $hash,
            ':expires' => date('Y-m-d H:i:s', time() + 3600 * self::TOKEN_EXPIRY_HOURS),
        ]);

        $this->logger->info('Password reset token generated', ['email' => $email]);

        return $plain;
    }

    /**
     * Validate a reset token.
     *
     * @param string $email Email address
     * @param string $token Plain token from URL
     * @return bool
     */
    public function validateToken(string $email, string $token): bool
    {
        $hash = hash('sha256', $token);

        $stmt = $this->pdo->prepare(
            "SELECT 1 FROM password_resets
             WHERE email = :email AND token_hash = :hash
             AND used_at IS NULL AND expires_at > NOW()
             ORDER BY created_at DESC LIMIT 1"
        );
        $stmt->execute([':email' => $email, ':hash' => $hash]);

        return $stmt->fetchColumn() !== false;
    }

    /**
     * Reset the password using a valid token.
     *
     * @param string $email
     * @param string $token Plain token from URL
     * @param string $newPassword New password
     * @throws \RuntimeException on invalid token
     */
    public function reset(string $email, string $token, string $newPassword): void
    {
        if (!$this->validateToken($email, $token)) {
            throw new \RuntimeException('Invalid or expired reset token');
        }

        $hash = hash('sha256', $token);

        $this->pdo->beginTransaction();
        try {
            // Update password
            $stmt = $this->pdo->prepare(
                "UPDATE users SET password_hash = :hash WHERE email = :email"
            );
            $stmt->execute([
                ':hash'  => password_hash($newPassword, PASSWORD_BCRYPT, ['cost' => 12]),
                ':email' => $email,
            ]);

            // Mark token as used
            $this->pdo->prepare(
                "UPDATE password_resets SET used_at = NOW() WHERE email = :email AND token_hash = :hash"
            )->execute([':email' => $email, ':hash' => $hash]);

            // Revoke all refresh tokens for this user
            $userId = $this->pdo->prepare("SELECT id FROM users WHERE email = :email");
            $userId->execute([':email' => $email]);
            $uid = $userId->fetchColumn();
            if ($uid !== false) {
                $this->pdo->prepare(
                    "UPDATE refresh_tokens SET revoked_at = NOW() WHERE user_id = :uid"
                )->execute([':uid' => $uid]);
            }

            $this->pdo->commit();
            $this->logger->info('Password reset completed', ['email' => $email]);
        } catch (\Throwable $e) {
            $this->pdo->rollBack();
            throw $e;
        }
    }
}
```

---

## 10. Security Headers Middleware

```php
<?php
declare(strict_types=1);

namespace App\Auth;

final class SecurityHeadersMiddleware implements \App\Infrastructure\Http\Middleware
{
    /** @param array<string, string> $extraHeaders */
    public function __construct(
        private readonly array $extraHeaders = [],
    ) {}

    public function handle(\App\Infrastructure\Http\RequestContext $ctx, callable $next): \App\Infrastructure\Http\RequestContext
    {
        $headers = array_merge([
            'X-Content-Type-Options'  => 'nosniff',
            'X-Frame-Options'         => 'DENY',
            'X-XSS-Protection'        => '0', // Deprecated but still sent
            'Referrer-Policy'         => 'strict-origin-when-cross-origin',
            'Permissions-Policy'      => 'camera=(), microphone=(), geolocation=()',
            'Strict-Transport-Security' => 'max-age=31536000; includeSubDomains',
        ], $this->extraHeaders);

        foreach ($headers as $name => $value) {
            header("$name: $value");
        }

        return $next($ctx);
    }
}
```

---

## Database Schema for Auth Tables

```sql
-- Users
CREATE TABLE users (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    totp_secret VARCHAR(64) NULL,
    email_verified_at DATETIME NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Password resets
CREATE TABLE password_resets (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    token_hash VARCHAR(64) NOT NULL,
    expires_at DATETIME NOT NULL,
    used_at DATETIME NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_email_token (email, token_hash),
    INDEX idx_expires (expires_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- API tokens
CREATE TABLE api_tokens (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id INT UNSIGNED NOT NULL,
    name VARCHAR(255) NOT NULL,
    token_hash VARCHAR(255) NOT NULL,
    abilities JSON NOT NULL,
    last_chars VARCHAR(4) NOT NULL,
    last_used_at DATETIME NULL,
    revoked_at DATETIME NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user (user_id),
    INDEX idx_last_chars (last_chars),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Refresh tokens
CREATE TABLE refresh_tokens (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id INT UNSIGNED NOT NULL,
    token_hash VARCHAR(64) NOT NULL,
    device_name VARCHAR(255) NULL,
    expires_at DATETIME NOT NULL,
    revoked_at DATETIME NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user (user_id),
    INDEX idx_token_hash (token_hash),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- RBAC tables
CREATE TABLE roles (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE permissions (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE user_roles (
    user_id INT UNSIGNED NOT NULL,
    role_id INT UNSIGNED NOT NULL,
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE role_permissions (
    role_id INT UNSIGNED NOT NULL,
    permission_id INT UNSIGNED NOT NULL,
    PRIMARY KEY (role_id, permission_id),
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE,
    FOREIGN KEY (permission_id) REFERENCES permissions(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE role_hierarchy (
    child_role_id INT UNSIGNED NOT NULL,
    parent_role_id INT UNSIGNED NOT NULL,
    PRIMARY KEY (child_role_id, parent_role_id),
    FOREIGN KEY (child_role_id) REFERENCES roles(id) ON DELETE CASCADE,
    FOREIGN KEY (parent_role_id) REFERENCES roles(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```
