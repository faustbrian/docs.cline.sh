---
title: Pipeable Functions
description: Learn about partial application and how to create pipe-friendly versions of multi-argument PHP functions.
---

As stated, `pipe()` works only with unary functions. PHP has numerous functions that are not unary, however, including many of the most useful array and string functions. For that reason, this library provides alternate, pipe-friendly versions of most common operations. All of them will take some number of arguments and return a Closure that has those arguments partially applied; that is, the provided arguments get "saved" and used when the returned function is invoked. Normally that would be within a `pipe()` chain, but they may be directly invoked as well if desired.

For example, the `explode()` function (which is namespaced to not conflict with the global function), takes a single argument, the delimiter. Its return value is a callable that will, when called with a string, call the built-in `\explode()` function with the provided string and the saved delimiter.

```php
use function Cline\fp\explode;

$result = pipe("Hello world",
  explode(' '),  // Produces ['Hello', 'world']
  count(...),    // Returns the number of array elements, which is 2
);
// $result is now 2

// or

$words = explode(' ')("Hello World");
// $words is now ['Hello', 'world']
```

The upshot of this approach is that _nearly all needle/haystack questions go away_, as either the value to operate on is subsumed into the pipe itself or very clearly provided as a secondary argument list.

Most functions will simply wrap and fall back to standard-lib functions where possible.
