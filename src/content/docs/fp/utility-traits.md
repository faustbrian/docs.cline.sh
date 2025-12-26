---
title: Utility Traits
description: PHP traits that enable functional programming patterns with objects, including immutable evolution and pipeable construction.
---

`Cline/fp` provides two traits to aid usage of objects in a functional environment.

## Evolvable

The `Evolvable` trait provides a single method, `with()`, which accepts a variadic list of named arguments. It will produce a new copy of the same object, but with the provided values assigned to properties of the same name. Note that because it runs in private scope, visibility is ignored. This may be undesirable in some cases.

This trait is most useful with `readonly` classes:

```php
use Cline\fp\Evolvable;

readonly class Person
{
    use Evolvable;

    public function __construct(
        public string $name,
        public int $age,
        public string $jobTitle,
    ) {}
}

$p = new Person("Larry", 25, "Engineer");
$p2 = $p->with(age: 18, jobTitle: "Developer");
```

## Newable

Constructor calls in PHP are screwy, and cannot be easily chained or piped. The `Newable` trait provides a simple standard static method that wraps the constructor, making it possible to chain and pipe. Its arguments are variadic and passed to the constructor verbatim.

```php
use Cline\fp\Newable;

readonly class Person
{
    use Newable;

    public function __construct(
        public string $name,
        public int $age,
        public string $jobTitle,
    ) {}
}

$p = Person::new("Larry", 18, "Developer");
```

You can also use it within a pipe:

```php
use function Cline\fp\pipe;
use Cline\fp\Newable;

readonly class Person
{
    use Newable;

    public function __construct(
        public string $name,
    ) {}
}

$person = pipe("Alice",
    Person::new(...)
);
// $person is a Person with name "Alice"
```
