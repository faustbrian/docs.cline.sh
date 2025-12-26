---
title: Array Functions
description: Comprehensive collection of functional array manipulation functions for PHP, including mapping, filtering, reducing, and more.
---

All functions below are in the `Cline\fp` namespace.

In many cases below, there are multiple versions of a function. There are two axes on which they vary: Whether they return an array or an iterable, and whether they operate on array keys as well.

PHP's built-in array functions do not accept iterables; Nearly all the functions below do. Those that begin with `a` will return an array, even if what was passed in is an iterator. Those that begin with `it` will return a Generator, which will produce values lazily.

Unless otherwise specified, functions will operate only on array values. Array keys will be explicitly ignored and not passed to the provided callback, but preserved. If a function has the suffix `withKeys`, then the key will be made available to the provided callback.

These must be separated due to the combination of three ill-interacting PHP features:

1. All arrays are associative, but some are short-circuited to have list-like keys, rather than having lists and maps be two separate constructs.
2. PHP supports optional arguments, which means some functions will misbehave if passed an array key as an optional second argument.
3. PHP user-space functions will silently ignore excess arguments but bundled functions will fail if called with excess arguments.

The upshot of these design choices is that it is not possible to reliably build a function that applies a callable to an array without knowing if the keys are important. That distinction must be made by the developer. The non-key versions require a callback with a single argument only (the array value), while the `withKeys` version will pass the value and key as two separate arguments to the callback.

Deciding whether to use the greedy (array) or lazy (iterable) versions of functions depends on the tradeoffs appropriate to your use case. As a general rule, the greedy version will be faster but may use more memory, while the lazy version will use less memory but may be slower. How much the difference is will vary widely with the specific use case.

## Mapping

Applies a provided callable to each entry in an iterable, producing a new iterable with the same keys as the source, but with the values replaced with the corresponding callback result.

* `amap(callable $c)`
* `amapWithKeys(callable $c)`
* `itmap(callable $c)`
* `itmapWithKeys(callable $c)`

```php
use function Cline\fp\{pipe, amap};

$doubled = pipe([1, 2, 3],
  amap(fn($x) => $x * 2)
);
// $doubled is [2, 4, 6]
```

## Filtering

Produces a new array containing only those array entries where the callable returns true. Array keys are preserved. If no callback is provided, a default of "is truthy" is used, just like PHP's native [`array_filter()`](https://www.php.net/array_filter).

* `afilter(?callable $c = null)`
* `afilterWithKeys(?callable $c = null)`
* `itfilter(?callable $c = null)`
* `itfilterWithKeys(?callable $c = null)`

```php
use function Cline\fp\{pipe, afilter};

$evens = pipe([1, 2, 3, 4],
  afilter(fn($x) => $x % 2 === 0)
);
// $evens is [2, 4]
```

## Collecting

The `collect()` function will accept a piped iterable or array, and produce an array. It's really just a wrapper around `iterator_to_array()` that guards against passing it an array, which is not supported in PHP 8.1. In PHP 8.2 and later, this function is equivalent to just using `iterator_to_array(...)` directly in a pipe, as it now accepts arrays as well.

## Reducing

Reducing, also known as `fold` or `foldl` in some languages, involves iteratively applying an operation across an array to produce a single final result. See [`array_reduce()`](https://www.php.net/array_reduce) for more details.

* `reduce(mixed $init, callable $c)` - Starting with `$init`, `$c` will be called with `$init` and each element in a piped iterable, and the result used as `$init` for the next entry. The callable signature is `($runningValue, $valueFromTheArray)`. The return from the last callable invocation is returned.
* `reduceWithKeys(mixed $init, callable $c)` - Same as `reduce()`, but the callback signature is `($runningValue, $valueFromTheArray, $keyFromTheArray)`.
* `reduceUntil(mixed $init, callable $c, callable $stop)` - Same as `reduce()`, but after each iteration `$stop($runningValue)` is called. If that returns true, the process stops early and whatever the current running value is will be returned.

```php
use function Cline\fp\{pipe, reduce};

$sum = pipe([1, 2, 3, 4],
  reduce(0, fn($carry, $x) => $carry + $x)
);
// $sum is 10
```

## First, conditionally

Several functions provide a way to obtain the first value in a sequence that meets some criteria. In all cases, they return null if nothing does.

* `first(callable $c)` - Returns the first value in a piped iterable for which `$c` returns `true`.
* `firstWithKeys(callable $c)` - Same as `first()`, but the callback is passed the value and key of each entry rather than just the value.
* `firstValue(callable $c)` - Invokes the provided callable on each item in a piped iterable, and returns the first result that is truthy, according to PHP.
* `firstValueWithKeys(callable $c)` - Same as `firstValue()`, but the callback is passed the value and key of each entry rather than just the value.

```php
use function Cline\fp\{pipe, first};

$firstEven = pipe([1, 3, 4, 6],
  first(fn($x) => $x % 2 === 0)
);
// $firstEven is 4
```

## Miscellaneous functions

* `indexBy(callable $keyMaker)` - Takes a piped array and returns a new array with the same values, but the key for each value is the result of calling `$keyMaker` with the value.
* `keyedMap(callable $values, ?callable $keys = null)` - Produces a new array from a piped array, in which the keys are the result of calling `$keys($key, $value)` and the values are the result of calling `$values($key, $value)`. If no `$keys` callback is specified, a default is provided that just indexes the entries numerically.
* `any(callable $c)` - Returns true if `$c` returns `true` for any value in a piped iterable. It may not be invoked on all items.
* `anyWithKeys(callable $c)` - Same as `any()`, but the callback is passed the value and key of each entry rather than just the value.
* `all(callable $c)` - Returns true if `$c` returns `true` for all value in a piped iterable. It may not be invoked on all items.
* `allWithKeys(callable $c)` - Same as `all()`, but the callback is passed the value and key of each entry rather than just the value.
* `flatten(array $arr)` - Accepts a multidimensional piped array and returns all the same values, but flattened into a single-dimensional sequential array.
* `append(mixed $value, mixed $key = null)` - Returns a piped array, but with the provided value added. If `$key` is provided, the value is assigned to that key regardless of whether it already exists. If not, the value is appended with `[]` and PHP's normal array handling applies.
* `atake(int $count)` - Accepts a piped iterable and returns an array, consisting of the first `$count` items from the iterable/array, or all the items if there are fewer than `$count`.
* `ittake(int $count)` - Accepts a piped iterable and returns an iterable, consisting of the first `$count` items from the iterable/array, or all the items if there are fewer than `$count`.
* `headtail(mixed $init, callable $first, callable $rest)` - Similar to `reduce()`, but uses a different reducing function for the first item.

## Utility functions

The following functions are not designed to be used with `pipe()`, but are more "traditional" functions. That said, they may be referenced as a first-class-closure.

* `iterate(mixed $init, callable $mapper)` - Produces an infinite list Generator. The first item is `$init`, the second is the result of calling `$mapper($init)`, the third is the result of calling `$mapper` on that result, etc. Note: This generator produces an infinite list! Make sure you have some termination check when calling it to avoid iterating forever.
* `nth(int $count, mixed $init, callable $mapper)` - Similar to `iterate()`, but returns the `$count`th item from the sequence and then stops.
* `head(array $a)` - Returns the first item from an array, or `null` if the array is empty.
* `tail(array $a)` - Returns all but the first item from an array.
