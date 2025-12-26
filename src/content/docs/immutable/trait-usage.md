---
title: Trait Usage
description: Using the Immutable trait for clean, type-safe object mutations
---

## Overview

The `Immutable` trait provides a `mutate()` method that creates modified copies of your objects while keeping the originals unchanged. This is the recommended approach when you control the class definition.

## Basic Setup

Add the trait and implement the `Mutable` contract:

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
```

The `Mutable` contract is optional but recommended - it documents that your class supports immutable mutation and enables IDE autocompletion.

## Mutation Patterns

### Single Property

```php
$user = new UserData('John', 'john@example.com', 30);
$updated = $user->mutate(['age' => 31]);

// Original unchanged
$user->age;     // 30
$updated->age;  // 31
```

### Multiple Properties

```php
$updated = $user->mutate([
    'name' => 'Jane',
    'email' => 'jane@example.com',
]);
```

### All Properties

```php
$updated = $user->mutate([
    'name' => 'Jane',
    'email' => 'jane@example.com',
    'age' => 25,
]);
```

### Clone Without Changes

Pass an empty array to create an exact clone:

```php
$clone = $user->mutate([]);

$clone !== $user;          // true (different instance)
$clone->name === $user->name;  // true (same values)
```

## Complex Objects

The trait handles complex property types including enums, DateTimeImmutable, and nested objects.

### With Enums

```php
<?php

enum OperationStatus: string
{
    case Pending = 'pending';
    case Running = 'running';
    case Completed = 'completed';
    case Cancelled = 'cancelled';
}

final class OperationData implements Mutable
{
    use Immutable;

    public function __construct(
        public readonly string $id,
        public readonly OperationStatus $status,
        public readonly int $progress,
        public readonly ?DateTimeImmutable $completedAt,
    ) {}
}

$operation = new OperationData(
    id: 'op-123',
    status: OperationStatus::Running,
    progress: 50,
    completedAt: null,
);

$completed = $operation->mutate([
    'status' => OperationStatus::Completed,
    'progress' => 100,
    'completedAt' => new DateTimeImmutable(),
]);
```

### Nullable Properties

```php
// Set nullable to value
$withDate = $operation->mutate(['completedAt' => new DateTimeImmutable()]);

// Set back to null
$withoutDate = $withDate->mutate(['completedAt' => null]);
```

## Type Validation

The trait validates property types at runtime:

```php
use Cline\Immutable\Exceptions\InvalidTypeException;

// Throws InvalidTypeException - age expects int
$user->mutate(['age' => 'thirty']);
// Message: Property "age" expects type "int", got "string".

// Throws InvalidTypeException - status expects enum
$operation->mutate(['status' => 'running']);
// Message: Property "status" expects type "OperationStatus", got "string".
```

### Supported Types

- Scalar types: `int`, `float`, `string`, `bool`
- `array`
- `object`
- `mixed`
- Class types (including enums)
- Nullable types (`?string`, `?int`, etc.)

Union and intersection types skip validation.

## Property Validation

Non-existent properties throw an exception:

```php
use Cline\Immutable\Exceptions\PropertyDoesNotExistException;

$user->mutate(['nonexistent' => 'value']);
// Throws: Property "nonexistent" does not exist on class "UserData".
```

## Best Practices

### Use Readonly Properties

Combine with PHP's `readonly` keyword for maximum safety:

```php
public function __construct(
    public readonly string $name,  // Cannot be modified after construction
    public readonly string $email,
    public readonly int $age,
) {}
```

### Final Classes

Make classes `final` to prevent inheritance issues:

```php
final class UserData implements Mutable
{
    use Immutable;
    // ...
}
```

### Descriptive DTOs

Use Data Transfer Objects (DTOs) for complex state:

```php
final class OrderData implements Mutable
{
    use Immutable;

    public function __construct(
        public readonly string $id,
        public readonly string $customerId,
        public readonly OrderStatus $status,
        public readonly Money $total,
        public readonly DateTimeImmutable $createdAt,
        public readonly ?DateTimeImmutable $shippedAt,
        public readonly ?string $trackingNumber,
    ) {}
}

// Clean state transitions
$shipped = $order->mutate([
    'status' => OrderStatus::Shipped,
    'shippedAt' => new DateTimeImmutable(),
    'trackingNumber' => 'TRACK123',
]);
```
