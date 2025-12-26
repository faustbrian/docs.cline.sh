---
title: Instantiation
description: Enhanced methods for creating enum instances from values and names.
---

Enhanced methods for creating enum instances from values and names.

## Overview

The `From` trait adds:
- `from()` and `tryFrom()` methods to pure enums
- `fromName()` and `tryFromName()` methods to all enums

> **Note:** BackedEnum instances already have `from()` and `tryFrom()` methods which won't be overridden.

## Setup

```php
use Cline\Enums\Concerns\From;

enum TaskStatus: int
{
    use From;

    case Incomplete = 0;
    case Completed = 1;
    case Canceled = 2;
}

enum Role
{
    use From;

    case Administrator;
    case Subscriber;
    case Guest;
}
```

## Pure Enums

For pure enums, `from()` and `tryFrom()` work with case names:

### from()

Throws `ValueError` if the case doesn't exist:

```php
Role::from('Administrator'); // Role::Administrator
Role::from('Nobody'); // Error: ValueError
```

### tryFrom()

Returns `null` if the case doesn't exist:

```php
Role::tryFrom('Guest'); // Role::Guest
Role::tryFrom('Never'); // null
```

## All Enums

Both backed and pure enums can use name-based instantiation:

### fromName()

Throws `ValueError` if the case doesn't exist:

```php
// Backed enum
TaskStatus::fromName('Incomplete'); // TaskStatus::Incomplete
TaskStatus::fromName('Missing'); // Error: ValueError

// Pure enum
Role::fromName('Subscriber'); // Role::Subscriber
Role::fromName('Hacker'); // Error: ValueError
```

### tryFromName()

Returns `null` if the case doesn't exist:

```php
// Backed enum
TaskStatus::tryFromName('Completed'); // TaskStatus::Completed
TaskStatus::tryFromName('Nothing'); // null

// Pure enum
Role::tryFromName('Guest'); // Role::Guest
Role::tryFromName('Tester'); // null
```

## Use Cases

### Safe Instantiation

When dealing with user input or external data:

```php
$status = TaskStatus::tryFromName($request->get('status'));

if ($status === null) {
    return response()->json(['error' => 'Invalid status'], 400);
}
```

### Flexible APIs

Accept both values and names:

```php
public function setStatus(int|string $status): void
{
    $this->status = is_int($status)
        ? TaskStatus::from($status)
        : TaskStatus::fromName($status);
}
```

### Migration Support

When transitioning from string constants to enums:

```php
// Old code used strings
const STATUS_INCOMPLETE = 'Incomplete';

// New code can still instantiate from those strings
$status = TaskStatus::fromName(self::STATUS_INCOMPLETE);
```

### Dynamic Case Resolution

```php
$caseNames = ['Administrator', 'Subscriber', 'Guest'];

$roles = array_map(
    fn($name) => Role::fromName($name),
    $caseNames
);
```
