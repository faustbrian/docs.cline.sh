---
title: String Functions
description: Pipe-friendly string manipulation functions for exploding, imploding, and replacing text.
---

All functions below are in the `Cline\fp` namespace.

## Available Functions

### explode(string $delimiter)

Explode a piped string using `$delimiter`.

```php
use function Cline\fp\explode;

$words = pipe("Hello World",
  explode(' ')
);
// $words is ['Hello', 'World']
```

### implode(string $glue)

Implode a piped array using `$glue`.

```php
use function Cline\fp\implode;

$sentence = pipe(['Hello', 'World'],
  implode(' ')
);
// $sentence is 'Hello World'
```

### replace(array|string $find, array|string $replace)

Does a find/replace in a piped string, using [`str_replace()`](https://www.php.net/str_replace).

```php
use function Cline\fp\replace;

$result = pipe("Hello World",
  replace('World', 'PHP')
);
// $result is 'Hello PHP'
```
