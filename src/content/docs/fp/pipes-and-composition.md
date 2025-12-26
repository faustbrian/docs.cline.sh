---
title: Pipes and Composition
description: Understanding the core pipe() and compose() functions that enable functional programming in PHP.
---

The most important function in this library is `pipe()`. It takes an arbitrary number of arguments. The first is whatever starting value you want to send through a sequence of functions. The rest are any unary callable (single-argument callable) that returns a value. `pipe()` will pass the first value to the first callable, then pass the result of that to the second callable, then pass the result of that to the third callable, and so on until the pipe ends. The final result will then be returned.

## Basic Usage

For a trivial example:

```php
use function Cline\fp\pipe;

$result = pipe(5,
  static fn ($in) => $in ** 4,     // Returns 625
  static fn ($in) => $in / 4,     // Returns 156.25
  static fn ($in) => (string)$in,  // Coerces the number to a string
  strlen(...),                    // Returns the length of the string
);
// $result is now 6, because "156.25" has 6 characters in it.
```

## Composition

There is also a similar method `compose()`, which takes only an arbitrary number of callables and produces a function that will take one argument and pass it through all of them the same way. The difference is that `compose()` returns the resulting callable, while `pipe()` executes immediately. Technically it is trivial to implement either one in terms of the other, but for performance reasons they are separate.
