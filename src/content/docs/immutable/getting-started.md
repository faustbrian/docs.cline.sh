---
title: Getting Started
description: Immutable object mutation utilities for PHP - create modified copies without changing originals
---

## Installation

Install via Composer:

```bash
composer require cline/immutable
```

The service provider will be automatically registered via Laravel's package discovery.

## What is Immutable?

Immutable provides utilities for working with immutable objects and arrays in PHP. Instead of modifying objects directly, you create new copies with your desired changes - the original remains untouched.

### Why Immutability?

- **Predictable State** - Objects don't change unexpectedly
- **Thread Safety** - No race conditions from shared mutable state
- **Debugging** - Trace exactly when and where state changes occur
- **Functional Patterns** - Enable pure functions that don't mutate inputs

## Quick Start

### Using the Immutable Trait

Add the `Immutable` trait to your data objects:

```php
<?php

use Cline\Immutable\Concerns\Immutable;
use Cline\Immutable\Contracts\Mutable;

final class UserData implements Mutable
{
    use Immutable;

    public function __construct(
        public readonly string $name,
        public readonly string $email,
        public readonly int $age,
    ) {}
}

$user = new UserData('John', 'john@example.com', 30);

// Create a new instance with modified age
$updated = $user->mutate(['age' => 31]);

$user->age;     // 30 (unchanged)
$updated->age;  // 31 (new instance)
```

### Using the Mutator Class

The `Mutator` class works with any object or array:

```php
<?php

use Cline\Immutable\Mutator;

// Mutate arrays
$original = ['name' => 'John', 'age' => 30];
$updated = Mutator::mutate($original, ['age' => 31]);

// Mutate any object
$user = new UserData('John', 'john@example.com', 30);
$updated = Mutator::mutate($user, ['name' => 'Jane']);

// Fluent API
$updated = Mutator::for($user)
    ->with(['name' => 'Jane'])
    ->set('age', 25)
    ->get();
```

## Two Approaches

Immutable offers two complementary approaches:

### 1. Trait-Based (Recommended for Your Classes)

Use the `Immutable` trait when you control the class definition. This provides a clean `mutate()` method directly on your objects.

```php
$updated = $user->mutate(['age' => 31]);
```

### 2. Static Mutator (Works with Any Object)

Use the `Mutator` class when working with third-party objects, stdClass, or arrays.

```php
$updated = Mutator::mutate($anyObject, ['property' => 'value']);
```

## Type Safety

Both approaches validate types at runtime:

```php
// Throws InvalidTypeException - age must be int
$user->mutate(['age' => 'thirty']);

// Throws PropertyDoesNotExistException - property doesn't exist
$user->mutate(['nonexistent' => 'value']);
```

## Next Steps

- **[Trait Usage](trait-usage)** - Detailed guide to using the Immutable trait
- **[Static Mutations](static-mutations)** - Using the Mutator class for arrays and objects
- **[Exceptions](exceptions)** - Understanding error handling
