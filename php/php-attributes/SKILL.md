---
name: php-attributes
description: "Complete guide to PHP 8.0+ attributes — native attribute declarations, reflection, validation, routing, serialization, and custom attribute patterns."
---

# PHP Attributes

## 1. Overview

PHP 8.0 introduced **attributes** (also called annotations), a native language feature for adding structured metadata to classes, methods, properties, constants, and parameters. Unlike docblock annotations (`@param`, `@return`), attributes are first-class PHP syntax that can be typed, validated, and introspected via reflection at runtime.

```mermaid
flowchart TB
    subgraph Sources[Attribute Targets]
        C[Class]
        M[Method]
        P[Property]
        PM[Parameter]
        CO[Constant]
        FN[Function]
        CO2[Closure]
    end

    subgraph Usage[Common Uses]
        V[Validation]
        R[Routing]
        S[Serialization]
        D[DI / Autowire]
        A[Auth / Middleware]
        E[Event Mapping]
    end

    Sources -->|#[...]| Usage
    Usage -->|Reflection| Runtime[Runtime Processing]
```

## 2. Core Concepts

### 2.1 Attribute Declaration

```php
#[Attribute]
class ReadOnly
{
}
```

### 2.2 Attribute Usage

```php
#[ReadOnly]
class UserDTO
{
    #[ReadOnly]
    public string $name;
}
```

### 2.3 Built-in Attributes

```php
#[Attribute]          // Marks a class as an attribute
#[AllowDynamicProperties]  // Allows dynamic properties (PHP 8.2+)
#[Override]           // Method must override parent (PHP 8.3+)
#[\ReturnTypeWillChange]   // Suppress return type mismatch warning (PHP 8.1+)
#[Deprecated]         // Marks function/class as deprecated (PHP 8.4+)
```

## 3. Architecture & Design

### 3.1 Attribute Targets

```php
// Target classes, methods, properties, etc.
#[Attribute(Attribute::TARGET_CLASS | Attribute::TARGET_METHOD)]
class Route
{
    public function __construct(
        public string $path,
        public string $method = 'GET',
    ) {}
}

// Constraining: only valid on methods
#[Attribute(Attribute::TARGET_METHOD)]
class Middleware
{
    public function __construct(
        public string ...$names,
    ) {}
}
```

| Target Constant | Valid On |
|-----------------|----------|
| `TARGET_CLASS` | Classes, enums, interfaces, traits |
| `TARGET_METHOD` | Methods |
| `TARGET_PROPERTY` | Properties |
| `TARGET_PARAMETER` | Function/method parameters |
| `TARGET_FUNCTION` | Functions |
| `TARGET_CONSTANT` | Constants (PHP 8.3+) |
| `TARGET_ALL` | Everything |

### 3.2 Attribute Inheritance

**Key rule:** Attributes on methods are NOT inherited by overriding methods. You must read parent class attributes explicitly via `getParentClass()` → `getMethod()`.

## 4. Syntax & Usage

### 4.1 Basic Attribute Class

```php
declare(strict_types=1);

#[Attribute]
class NotBlank
{
    public function __construct(
        public string $message = 'This value should not be blank',
    ) {}
}
```

### 4.2 Repeated Attributes

```php
#[Attribute(Attribute::TARGET_CLASS | Attribute::TARGET_METHOD | Attribute::IS_REPEATABLE)]
class Middleware
{
    public function __construct(
        public string $name,
    ) {}
}

#[Middleware('auth')]
#[Middleware('csrf')]
#[Middleware('rate_limit')]
class DashboardController
{
    // ...
}
```

### 4.3 Attribute with Arrays and Named Arguments

```php
#[Attribute]
class ValidationRule
{
    public function __construct(
        public string $rule,
        public array $options = [],
    ) {}
}

#[ValidationRule('email')]
#[ValidationRule('unique', options: ['table' => 'users', 'column' => 'email'])]
public string $email;
```

### 4.4 Reading Attributes via Reflection

```php
function processRoutes(string $controllerClass): void
{
    $reflection = new \ReflectionClass($controllerClass);

    // Read class-level attributes
    $classAttrs = $reflection->getAttributes(Route::class);
    foreach ($classAttrs as $attr) {
        $route = $attr->newInstance();
        echo "Base path: {$route->path}\n";
    }

    // Read method-level attributes
    foreach ($reflection->getMethods() as $method) {
        $methodAttrs = $method->getAttributes(Route::class);
        foreach ($methodAttrs as $attr) {
            $route = $attr->newInstance();
            echo "{$route->method} {$route->path}\n";
        }
    }
}
```

## 5. Best Practices

1. **Always use `declare(strict_types=1)`** in attribute classes
2. **Name attributes as adjectives or nouns** — `#[NotBlank]`, `#[Route]`, `#[Transactional]`
3. **Keep attributes stateless** — they're metadata, not services
4. **Use typed constructor parameters** — validation happens automatically at instantiation
5. **Make attributes repeatable** when multiple of the same kind make sense
6. **Use `#[Attribute]` with explicit targets** — document intent
7. **Validate attribute arguments in constructor** — `throw \InvalidArgumentException`
8. **Prefer attributes over docblock annotations** — type-safe, explicit
9. **Create domain-specific attribute libraries** — `#[ApiProperty]`, `#[CommandHandler]`
10. **Cache attribute reflection results** — reflection is expensive in tight loops

## 6. Anti-Patterns

### 6.1 Mixing Attributes and Docblocks

```php
// ❌ BAD — mixed metadata sources
class Product
{
    #[NotNull]
    /** @var string */
    public string $name;

    /** @Assert\NotBlank */
    public string $description;
}

// ✅ GOOD — pick one (attributes preferred)
class Product
{
    #[NotBlank]
    public string $name;

    #[NotBlank]
    public string $description;
}
```

### 6.2 Attribute with Heavy Logic

```php
// ❌ BAD — attribute does work
#[Attribute]
class LogExecutionTime
{
    public function __construct()
    {
        // Starts a timer? No! Attributes are just metadata.
    }
}

// ✅ GOOD — attribute is metadata
#[Attribute]
class LogExecutionTime
{
    public function __construct(
        public string $channel = 'performance',
    ) {}
}
```

### 6.3 Attribute as DI Container

```php
// ❌ BAD — attribute tries to be a service locator
#[Attribute]
class Inject
{
    public function __construct(
        public string $serviceClass,
    ) {
        $this->instance = Container::get($serviceClass);
    }
}

// ✅ GOOD — attribute only describes, doesn't instantiate
#[Attribute]
class Inject
{
    public function __construct(
        public string $serviceClass,
    ) {}
}
```

### 6.4 Overusing Non-Standard Attributes

```php
// ❌ BAD — every line is an attribute
#[Transactional]
#[Cache(duration: 3600)]
#[Log(channel: 'audit')]
#[Retry(times: 3)]
#[Authorize(role: 'admin')]
#[RateLimit(max: 100, window: 60)]
class CreateOrderHandler
{
    // ...
}

// ✅ GOOD — group related concerns, use thoughtfully
class CreateOrderHandler
{
    #[Transactional]
    public function __invoke(#[Validated] CreateOrderCommand $command): void
    {
        // ...
    }
}
```

## 7. Trade-offs

| Aspect | Pros | Cons |
|--------|------|------|
| vs docblocks | Type-safe, IDE validated, modern | Breaks backwards compatibility with tools reading docblocks |
| Runtime cost | Nil (just metadata) | Reflection parsing can be slow without caching |
| Serialization | Structured, typed data | Can't serialize attributes to JSON natively |
| Composability | Repeatable, combinable | Multiple attributes can be verbose |
| Debugging | Clear compilation errors | `newInstance()` can throw at runtime |

## 8. AI Reasoning Guide

### When generating attributes:

1. **Replace `@param` / `@return` with PHP 8 types** — attributes don't replace type hints
2. **Replace `@see` / `@link` with attributes** — structured metadata
3. **Replace validation annotations** — `#[NotBlank]`, `#[Email]`, `#[Unique]`
4. **Replace route annotations** — `#[Route('/path', method: 'POST')]`
5. **Add event/handler mapping** — `#[EventHandler('user.created')]`
6. **Add serialization metadata** — `#[Exclude]`, `#[Groups(['admin'])]`

### Attribute vs annotation decision:

```
Do you need unstructured text (description/docs)?
├─ Yes → Keep docblock: /** Description */
Do you need structured metadata for runtime processing?
├─ Yes → Use attribute: #[Route('/path')]
Need both?
├─ Docblock for descriptions, attribute for structured metadata
```

### Reflection processing patterns:

```
Simple read: getAttributes(MyAttr::class) → newInstance()
Read with validation: check getArguments() before instantiation
Repeatable: getAttributes() always returns array, check IS_REPEATABLE
Inherited: manually traverse getParentClass() for method attributes
```

## 9. Examples

### 9.1 Minimal Validation Framework

```php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_PROPERTY | Attribute::IS_REPEATABLE)]
interface ValidationAttribute
{
    public function validate(mixed $value): void;
}

#[Attribute(Attribute::TARGET_PROPERTY)]
class NotBlank implements ValidationAttribute
{
    public function __construct(
        public string $message = 'Value must not be blank',
    ) {}

    public function validate(mixed $value): void
    {
        if ($value === null || $value === '' || $value === []) {
            throw new \ValidationException($this->message);
        }
    }
}

#[Attribute(Attribute::TARGET_PROPERTY)]
class Email implements ValidationAttribute
{
    public function __construct(
        public string $message = 'Invalid email address',
    ) {}

    public function validate(mixed $value): void
    {
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            throw new \ValidationException($this->message);
        }
    }
}

// Validator engine
class Validator
{
    public function validate(object $object): void
    {
        $reflection = new \ReflectionClass($object);
        foreach ($reflection->getProperties() as $property) {
            $attrs = $property->getAttributes(ValidationAttribute::class);
            foreach ($attrs as $attr) {
                $validator = $attr->newInstance();
                $validator->validate($property->getValue($object));
            }
        }
    }
}

// Usage
class CreateUserRequest
{
    public function __construct(
        #[NotBlank]
        #[Email]
        public string $email,

        #[NotBlank(message: 'Name is required')]
        public string $name,
    ) {}
}
```

### 9.2 Route Attribute Framework

```php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_CLASS)]
class Controller
{
}

#[Attribute(Attribute::TARGET_METHOD | Attribute::IS_REPEATABLE)]
class Route
{
    public function __construct(
        public string $path,
        public string $method = 'GET',
        public ?string $name = null,
    ) {}
}

#[Attribute(Attribute::TARGET_METHOD)]
class Middleware
{
    public function __construct(
        public string $name,
    ) {}
}

// Router processor
class Router
{
    /** @var array<string, RouteHandler> */
    private array $routes = [];

    public function registerController(string $class): void
    {
        $reflection = new \ReflectionClass($class);
        $basePath = '';

        // Read class-level attribute
        $routeAttr = $reflection->getAttributes(Route::class);
        if (!empty($routeAttr)) {
            $basePath = $routeAttr[0]->newInstance()->path;
        }

        foreach ($reflection->getMethods(\ReflectionMethod::IS_PUBLIC) as $method) {
            foreach ($method->getAttributes(Route::class) as $routeAttr) {
                $route = $routeAttr->newInstance();
                $middlewareAttrs = $method->getAttributes(Middleware::class);
                $middleware = array_map(
                    fn($attr) => $attr->newInstance()->name,
                    $middlewareAttrs,
                );

                $this->routes[$route->method . ':' . $basePath . $route->path] = new RouteHandler(
                    controller: $class,
                    method: $method->getName(),
                    middlewares: $middleware,
                );
            }
        }
    }
}
```

### 9.3 JSON Serialization with Attributes

```php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_PROPERTY)]
class SerializedName
{
    public function __construct(
        public string $name,
    ) {}
}

#[Attribute(Attribute::TARGET_PROPERTY)]
class Exclude
{
}

#[Attribute(Attribute::TARGET_CLASS)]
class JsonSerializableWithAttributes
{
}

class JsonSerializer
{
    /** @return array<string, mixed> */
    public function serialize(object $object): array
    {
        $reflection = new \ReflectionClass($object);
        $result = [];

        foreach ($reflection->getProperties() as $property) {
            // Skip excluded
            if (!empty($property->getAttributes(Exclude::class))) {
                continue;
            }

            $value = $property->getValue($object);

            // Check for serialized name override
            $serializedName = $property->getAttributes(SerializedName::class);
            $key = $serializedName
                ? $serializedName[0]->newInstance()->name
                : $property->getName();

            // Recursively serialize nested objects
            if (is_object($value) && !$value instanceof \DateTimeInterface) {
                $value = $this->serialize($value);
            }

            $result[$key] = $value;
        }

        return $result;
    }
}

// Usage
class Address
{
    public function __construct(
        #[SerializedName('street_line')]
        public string $street,
        public string $city,
        #[Exclude]
        public string $internalNote = '',
    ) {}
}
```

### 9.4 Combined: Controller with Validation + Routing

```php
declare(strict_types=1);

#[Controller]
#[Route('/api/users')]
class UserController
{
    public function __construct(
        private UserRepository $repository,
        private Validator $validator,
    ) {}

    #[Route('/{id}', method: 'GET', name: 'user.get')]
    #[Middleware('auth')]
    public function get(string $id): UserResponse
    {
        return new UserResponse(
            user: $this->repository->findById($id),
        );
    }

    #[Route('/', method: 'POST', name: 'user.create')]
    #[Middleware('auth')]
    #[Middleware('csrf')]
    public function create(
        #[NotBlank]
        #[Email]
        public string $email,

        #[NotBlank(message: 'Name is required')]
        public string $name,
    ): UserResponse {
        $user = $this->repository->create($email, $name);
        return new UserResponse(user: $user);
    }
}
```

## 10. Common Pitfalls

1. **Forgetting `#[Attribute]` on attribute classes** — won't be recognized as attributes
2. **Attribute constructor throws** — `newInstance()` throws if validation fails, catch it gracefully
3. **`getAttributes()` without filter returns all attributes** — always filter by class name
4. **Attributes on methods are NOT inherited** — must traverse parent class manually
5. **`IS_REPEATABLE` needed for multiple same attributes** — without it, duplicate throws error
6. **Attribute instance == constructor arguments** — they're not singletons, every `newInstance()` creates a new object
7. **Can't type-check attribute constructor at definition time** — errors happen at reflection time
8. **Performance** — reflection + `newInstance()` is slow; cache attribute metadata
9. **Serialization** — attributes don't serialize; design separate serialization
10. **PHP 8.0 vs 8.1+ differences** — nested attributes in arguments changed syntax between 8.0 and 8.1

## 11. Related Features

| Feature | Relationship |
|---------|--------------|
| Enums | Attributes can accept enum values as arguments |
| Named arguments | `#[Route(path: '/', method: 'POST')]` |
| Union types | Attribute constructor can accept union types |
| Constructor promotion | Attributes use promotion for concise definitions |
| readonly classes | Attribute parameters are naturally readonly |
| `ReflectionAttribute` | API for reading attributes at runtime |

## 12. Version Compatibility

| Feature | PHP Version |
|---------|-------------|
| Basic attributes | 8.0+ |
| Named argument syntax in attributes | 8.0+ |
| `TARGET_CLASS_CONSTANT` | 8.3+ |
| `#[Override]` | 8.3+ |
| `#[Deprecated]` | 8.4+ |
| `#[AllowDynamicProperties]` | 8.2+ |

## 13. Reference

- [PHP Manual: Attributes](https://www.php.net/manual/en/language.attributes.php)
- [PHP RFC: Attributes v2](https://wiki.php.net/rfc/attributes_v2)
- [PHP RFC: Attributes v3 (named arguments)](https://wiki.php.net/rfc/attribute_named_arguments)
- [PHP RFC: Override attribute](https://wiki.php.net/rfc/override_attribute)
- [PHP RFC: Deprecated attribute](https://wiki.php.net/rfc/deprecated_attribute)
- [Doctrine Annotations vs Attributes Migration](https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/attributes-reference.html)
