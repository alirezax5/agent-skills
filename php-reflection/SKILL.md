---
name: php-reflection
description: PHP Reflection API — introspection of classes, methods, properties, parameters, attributes, enums; dynamic invocation, DI containers, serializers, and attribute reading for PHP 8.4
tags: [php, reflection, introspection, attributes, dependency-injection, strict-types]
---

# PHP Reflection API

> **PHP 8.4** · `declare(strict_types=1)` · Introspection, attributes, dynamic invocation

---

## 1. Overview

The PHP Reflection API provides the ability to introspect classes, interfaces, functions, methods, parameters, properties, and type information at runtime. It enables tools like dependency injection containers, ORMs, serializers, mocks, and documentation generators.

---

## 2. ReflectionClass

### 2.1 Basic Introspection

```php
<?php
declare(strict_types=1);

use App\Service\PaymentService;

$ref = new \ReflectionClass(PaymentService::class);

echo $ref->getName();           // "App\Service\PaymentService"
echo $ref->getShortName();      // "PaymentService"
echo $ref->getNamespaceName();  // "App\Service"

var_dump($ref->isAbstract());
var_dump($ref->isFinal());
var_dump($ref->isReadonly());
var_dump($ref->isInterface());
var_dump($ref->isTrait());
var_dump($ref->isEnum());

$instance = $ref->newInstance($constructorArg);
$instance = $ref->newInstanceWithoutConstructor();
```

### 2.2 Method & Property Discovery

```php
<?php
declare(strict_types=1);

$methods = $ref->getMethods();
$publicMethods = $ref->getMethods(\ReflectionMethod::IS_PUBLIC);

$properties = $ref->getProperties();
$promoted = $ref->getProperties(\ReflectionProperty::IS_PROMOTED);

$prop = $ref->getProperty('name');
$prop->setAccessible(true);
$value = $prop->getValue($instance);
$prop->setValue($instance, 'new value');
```

---

## 3. ReflectionMethod

```php
<?php
declare(strict_types=1);

$method = new \ReflectionMethod(PaymentService::class, 'processPayment');

echo $method->getName();
echo $method->getNumberOfParameters();
echo $method->getReturnType();

// Dynamic invocation
$method->setAccessible(true);
$result = $method->invoke($service, 'arg1');
$result = $method->invokeArgs($service, ['arg1', true]);
```

---

## 4. ReflectionParameter

```php
<?php
declare(strict_types=1);

$params = $method->getParameters();
foreach ($params as $param) {
    echo $param->getName() . ': ' . $param->getType() . "\n";
    echo "  Optional: " . ($param->isOptional() ? 'yes' : 'no') . "\n";
    if ($param->isDefaultValueAvailable()) {
        var_dump($param->getDefaultValue());
    }
}
```

---

## 5. ReflectionAttribute (PHP 8.0+)

```php
<?php
declare(strict_types=1);

use App\Attribute\Route;

$ref = new \ReflectionMethod(UserController::class, 'index');
$attributes = $ref->getAttributes(Route::class);

foreach ($attributes as $attribute) {
    $route = $attribute->newInstance();
    echo "Path: " . $route->path . "\n";
}
```

---

## 6. ReflectionFunction & Closures

```php
<?php
declare(strict_types=1);

$fn = new \ReflectionFunction('helper');
echo $fn->getNumberOfParameters();
echo $fn->getReturnType();

// Closures
$closure = function (int $x): int { return $x * 2; };
$ref = new \ReflectionFunction($closure);
```

---

## 7. ReflectionEnum (PHP 8.1+)

```php
<?php
declare(strict_types=1);

$ref = new \ReflectionEnum(Currency::class);
var_dump($ref->isBacked());
echo $ref->getBackingType();

$cases = $ref->getCases();
foreach ($cases as $case) {
    if ($case instanceof \ReflectionEnumBackedCase) {
        echo "  Value: " . $case->getBackingValue() . "\n";
    }
}
```

---

## 8. Real-World: Dependency Injection Container

```php
<?php
declare(strict_types=1);

final class Container
{
    private array $instances = [];

    public function get(string $class): object
    {
        if (isset($this->instances[$class])) {
            return $this->instances[$class];
        }

        $ref = new \ReflectionClass($class);

        if (!$ref->isInstantiable()) {
            throw new \RuntimeException("$class is not instantiable");
        }

        $constructor = $ref->getConstructor();
        if ($constructor === null) {
            return $this->instances[$class] = $ref->newInstance();
        }

        $params = array_map(
            fn(\ReflectionParameter $param) => $this->resolveParameter($param),
            $constructor->getParameters()
        );

        return $this->instances[$class] = $ref->newInstanceArgs($params);
    }

    private function resolveParameter(\ReflectionParameter $param): mixed
    {
        $type = $param->getType();
        if (!$type instanceof \ReflectionNamedType || $type->isBuiltin()) {
            if ($param->isDefaultValueAvailable()) {
                return $param->getDefaultValue();
            }
            throw new \RuntimeException("Cannot resolve \${$param->getName()}");
        }
        return $this->get($type->getName());
    }
}
```

---

## 9. Best Practices

1. **Cache reflection objects** — methods iteration is expensive
2. **Use `setAccessible(true)` sparingly** — breaks encapsulation
3. **Check type with `instanceof`** before calling type-specific methods
4. **Never serialize reflection objects** — cannot be unserialized
5. **Use `ReflectionAttribute::newInstance()` lazily**

---

## References

- [PHP Manual — Reflection](https://www.php.net/manual/en/book.reflection.php)