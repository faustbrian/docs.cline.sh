---
title: Object Functions
description: Functions for working with objects in a functional pipeline, including property access, method invocation, and type checking.
---

All functions below are in the `Cline\fp` namespace.

## Available Functions

### prop(string $prop)

Returns the `$prop` public property of a piped object.

```php
use function Cline\fp\prop;

class Person {
    public function __construct(public string $name) {}
}

$name = pipe(new Person('Alice'),
  prop('name')
);
// $name is 'Alice'
```

### method(string $method, ...$args)

Invokes `$method` on a piped object using `$args` as arguments. Both positional and named arguments are supported.

```php
use function Cline\fp\method;

class Calculator {
    public function add(int $a, int $b): int {
        return $a + $b;
    }
}

$result = pipe(new Calculator(),
  method('add', 5, 3)
);
// $result is 8
```

### typeIs(string $type)

Returns `true` if a piped value is of the specified type, `false` otherwise. Legal types are `int`, `string`, `float`, `bool`, `array`, `resource`, or a class/interface name. This will usually be the last function in a pipe.

```php
use function Cline\fp\typeIs;

$isString = pipe("Hello",
  typeIs('string')
);
// $isString is true

$isPerson = pipe(new Person('Alice'),
  typeIs(Person::class)
);
// $isPerson is true
```
