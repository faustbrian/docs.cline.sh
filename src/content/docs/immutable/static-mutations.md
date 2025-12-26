---
title: Static Mutations
description: Using the Mutator class for arrays and any object type
---

## Overview

The `Mutator` class provides static methods to create modified copies of any object or array. Use it when working with third-party classes, `stdClass`, or when you need a standalone mutation utility.

## Static API

### mutate()

The simplest way to create a modified copy:

```php
<?php

use Cline\Immutable\Mutator;

// Mutate an array
$original = ['name' => 'John', 'age' => 30];
$updated = Mutator::mutate($original, ['age' => 31]);

// Original unchanged
$original; // ['name' => 'John', 'age' => 30]
$updated;  // ['name' => 'John', 'age' => 31]

// Mutate an object
$user = new UserData('John', 'john@example.com', 30);
$updated = Mutator::mutate($user, ['age' => 31]);
```

## Fluent API

For more complex mutations, use the fluent interface:

### for()

Start a fluent chain:

```php
$mutator = Mutator::for($original);
```

### with()

Add multiple properties to modify:

```php
$updated = Mutator::for($user)
    ->with(['name' => 'Jane', 'age' => 25])
    ->get();
```

### set()

Add a single property:

```php
$updated = Mutator::for($user)
    ->set('name', 'Jane')
    ->get();
```

### Chaining

Chain multiple calls together:

```php
$updated = Mutator::for($user)
    ->set('name', 'Jane')
    ->with(['email' => 'jane@example.com'])
    ->set('age', 25)
    ->get();
```

Later mutations override earlier ones:

```php
$updated = Mutator::for(['name' => 'John'])
    ->set('name', 'Jane')
    ->set('name', 'Bob')
    ->get();

$updated['name']; // 'Bob'
```

### get()

Execute the mutation and return the result:

```php
$result = Mutator::for($data)->with($changes)->get();
```

## Array Mutations

### Replace Values

```php
$original = ['a' => 1, 'b' => 2];
$updated = Mutator::mutate($original, ['a' => 10]);
// ['a' => 10, 'b' => 2]
```

### Add New Keys

```php
$original = ['a' => 1, 'b' => 2];
$updated = Mutator::mutate($original, ['c' => 3]);
// ['a' => 1, 'b' => 2, 'c' => 3]
```

### Numeric Keys

```php
$original = [1, 2, 3];
$updated = Mutator::mutate($original, [0 => 10]);
// [10, 2, 3]
```

### Nested Arrays

Replace entire nested structures:

```php
$original = ['user' => ['name' => 'John', 'age' => 30]];
$updated = Mutator::mutate($original, [
    'user' => ['name' => 'Jane', 'age' => 25]
]);
// ['user' => ['name' => 'Jane', 'age' => 25]]
```

## Object Mutations

### Objects with Immutable Trait

The Mutator automatically delegates to the object's `mutate()` method:

```php
$user = new UserData('John', 'john@example.com', 30);
$updated = Mutator::mutate($user, ['age' => 31]);

// Equivalent to:
$updated = $user->mutate(['age' => 31]);
```

### Generic Objects

Works with any object using reflection:

```php
class GenericObject
{
    public function __construct(
        public string $name,
        public int $value,
    ) {}
}

$obj = new GenericObject('test', 42);
$updated = Mutator::mutate($obj, ['value' => 100]);

$obj->value;     // 42
$updated->value; // 100
```

### stdClass Objects

Full support for dynamic objects:

```php
$obj = new stdClass();
$obj->name = 'John';
$obj->age = 30;

$updated = Mutator::mutate($obj, ['age' => 31]);
$updated->age; // 31

// Can add new properties
$updated = Mutator::mutate($obj, ['email' => 'john@example.com']);
$updated->email; // 'john@example.com'
```

## Type Validation

For objects with type hints, the Mutator validates values:

```php
use Cline\Immutable\Exceptions\InvalidTypeException;

$obj = new GenericObject('test', 42);

// Throws InvalidTypeException
Mutator::mutate($obj, ['value' => 'not an int']);
```

### Property Validation

```php
use Cline\Immutable\Exceptions\PropertyDoesNotExistException;

$obj = new GenericObject('test', 42);

// Throws PropertyDoesNotExistException
Mutator::mutate($obj, ['nonexistent' => 'value']);
```

Note: `stdClass` allows any property name without validation.

## Complex Example

Real-world usage with operation state transitions:

```php
$operation = new OperationData(
    id: 'op-123',
    function: 'process',
    version: '1.0.0',
    status: OperationStatus::Running,
    progress: 50,
    result: null,
    errors: [],
    startedAt: $now,
    completedAt: null,
    cancelledAt: null,
    metadata: ['key' => 'value'],
);

// Without immutable mutation (verbose):
$cancelledOperation = new OperationData(
    id: $operation->id,
    function: $operation->function,
    version: $operation->version,
    status: OperationStatus::Cancelled,
    progress: $operation->progress,
    result: $operation->result,
    errors: $operation->errors,
    startedAt: $operation->startedAt,
    completedAt: $operation->completedAt,
    cancelledAt: new DateTimeImmutable(),
    metadata: $operation->metadata,
);

// With Mutator (clean):
$cancelledOperation = Mutator::mutate($operation, [
    'status' => OperationStatus::Cancelled,
    'cancelledAt' => new DateTimeImmutable(),
]);
```

## Laravel Facade

For Laravel applications, use the facade:

```php
<?php

use Cline\Immutable\Facades\Mutator;

$updated = Mutator::mutate($user, ['age' => 31]);

$updated = Mutator::for($user)
    ->with(['age' => 31])
    ->get();
```
