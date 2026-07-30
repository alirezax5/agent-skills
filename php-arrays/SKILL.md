---
name: php-arrays
description: Master PHP array functions — creation, access, mutation, sorting, map/filter/reduce, SPL data structures, spread operator, and production patterns with strict_types and PSR-12
tags: [php, arrays, spl, data-structures, sorting, map-filter-reduce, strict-types]
---

# PHP Array Mastery

## 1. Overview

Arrays are PHP's universal data structure — ordered map, list, hash table, stack, queue, and set all in one. PHP 8.4 provides ~80 native array functions alongside a rich SPL collection layer.

## 2. Key Functions Reference

### 2.1 Creation & Initialization

```php
declare(strict_types=1);

$nums = range(1, 10);                 // [1, 2, …, 10]
$filled = array_fill(0, 5, null);     // [null, null, null, null, null]
$keyed  = array_fill_keys(['a','b','c'], 0);  // ['a'=>0, 'b'=>0, 'c'=>0]
$padded = array_pad([1,2,3], 5, 0);  // [1,2,3,0,0]
$chunked  = array_chunk([1,2,3,4], 2); // [[1,2], [3,4]]
$slice    = array_slice([1,2,3,4,5], 1, 2); // [2,3]
```

### 2.2 Access & Mutation

```php
declare(strict_types=1);

$arr = ['a' => 1, 'b' => 2, 'c' => 3];

$val = $arr['d'] ?? null;           // null — no warning
$first = reset($arr);               // 1
$last  = end($arr);                  // 3

array_push($arr, 4);                 // O(1) append
$popped = array_pop($arr);           // O(1)

$has = array_key_exists('a', $arr);  // true
$has = array_search(2, $arr, true);  // 'b' — strict mode
```

### 2.3 Sorting

```php
declare(strict_types=1);

$arr = [3, 1, 4, 1, 5, 9];

asort($arr);     // asc, preserve keys
arsort($arr);    // desc, preserve keys
sort($arr);      // asc, re-index
rsort($arr);     // desc, re-index
ksort($arr);     // asc by key
usort($arr, fn(int $a, int $b): int => $a <=> $b);
natsort($files); // natural order
```

### 2.4 Transformations

```php
declare(strict_types=1);

$items = [1, 2, 3, 4, 5];

$squared = array_map(fn(int $v): int => $v ** 2, $items);
$evens   = array_filter($items, fn(int $v): bool => $v % 2 === 0);
$product = array_reduce($items, fn(int $c, int $v): int => $c * $v, 1);

$names = array_column($users, 'name');
$keyed = array_column($users, 'name', 'id');
```

### 2.5 Set Operations

```php
declare(strict_types=1);

$a = [1, 2, 3, 4];
$b = [3, 4, 5, 6];

$diff  = array_diff($a, $b);          // [1, 2]
$inter = array_intersect($a, $b);     // [3, 4]
$union = array_unique([...$a, ...$b]); // [1, 2, 3, 4, 5, 6]
```

### 2.6 Spread Operator (PHP 7.4+)

```php
declare(strict_types=1);

$merged  = [...$a, ...$b];
$copy    = [...$original];
[$first, $second] = [1, 2, 3];
```

## 3. SPL Data Structures

```php
declare(strict_types=1);

// ArrayObject — object with array syntax
$ao = new ArrayObject(['a' => 1]);
$ao['b'] = 2;
$ao->asort();

// SplStack / SplQueue
$stack = new SplStack();
$stack->push('a');
$stack->push('b');
$top = $stack->pop(); // 'b'

$queue = new SplQueue();
$queue->enqueue('a');
$first = $queue->dequeue(); // 'a'

// SplPriorityQueue
$pq = new SplPriorityQueue();
$pq->insert('low', 1);
$pq->insert('high', 100);
echo $pq->extract(); // 'high'

// SplFixedArray — fixed-size, memory efficient
$fixed = new SplFixedArray(3);
$fixed[0] = 'a';
```

## 4. Common Patterns

### 4.1 Paginate

```php
declare(strict_types=1);

function paginate(array $items, int $page = 1, int $perPage = 20): array
{
    $total  = count($items);
    $pages  = (int) ceil($total / $perPage);
    $offset = ($page - 1) * $perPage;
    return [
        'items'   => array_slice($items, $offset, $perPage),
        'page'    => $page,
        'perPage' => $perPage,
        'total'   => $total,
        'pages'   => $pages,
    ];
}
```

### 4.2 Group-by

```php
declare(strict_types=1);

function groupBy(array $items, callable $grouper): array
{
    $groups = [];
    foreach ($items as $item) {
        $groups[$grouper($item)][] = $item;
    }
    return $groups;
}
```

## 5. References

- [PHP Manual: Array Functions](https://www.php.net/manual/en/ref.array.php)
- [PHP Manual: SPL Data Structures](https://www.php.net/manual/en/spl.datastructures.php)
- [PSR-12: Extended Coding Style](https://www.php-fig.org/psr/psr-12/)