---
title: InvokableCases
description: Get the value of a backed enum, or the name of a pure enum, by invoking it statically or as an instance.
---

Get the value of a backed enum, or the name of a pure enum, by invoking it statically (`MyEnum::FOO()`) or as an instance (`$enum()`).

## Benefits

Eliminates the need to append `->value` when working with backed enums:

```php
// Without InvokableCases
'statuses' => [
    TaskStatus::Incomplete->value => ['some configuration'],
    TaskStatus::Completed->value => ['some configuration'],
];

// With InvokableCases
'statuses' => [
    TaskStatus::Incomplete() => ['some configuration'],
    TaskStatus::Completed() => ['some configuration'],
];
```

## Setup

```php
use Cline\Enums\Concerns\InvokableCases;

enum TaskStatus: int
{
    use InvokableCases;

    case Incomplete = 0;
    case Completed = 1;
    case Canceled = 2;
}

enum Role
{
    use InvokableCases;

    case Administrator;
    case Subscriber;
    case Guest;
}
```

## Static Invocation

Get the primitive value by invoking cases statically:

```php
// Backed enum returns the value
TaskStatus::Incomplete(); // 0
TaskStatus::Completed(); // 1
TaskStatus::Canceled(); // 2

// Pure enum returns the name
Role::Administrator(); // 'Administrator'
Role::Subscriber(); // 'Subscriber'
Role::Guest(); // 'Guest'
```

## Instance Invocation

Invoke enum instances to get their primitive value:

```php
public function updateStatus(TaskStatus $status, Role $role)
{
    // Invoke the instances to get their values
    $this->record->setStatus($status(), $role());
}
```

## Use Cases

### Array Keys

Configure mappings and associative arrays:

```php
$config = [
    TaskStatus::Incomplete() => ['color' => 'yellow', 'icon' => 'clock'],
    TaskStatus::Completed() => ['color' => 'green', 'icon' => 'check'],
    TaskStatus::Canceled() => ['color' => 'red', 'icon' => 'x'],
];
```

### Method Parameters

Pass primitive values to methods expecting them:

```php
public function updateStatus(int $status): void;

// Clean invocation without ->value
$task->updateStatus(TaskStatus::Completed());
```

### Database Queries

Use in database queries without appending `->value`:

```php
$tasks = Task::where('status', TaskStatus::Incomplete())->get();
```

## IDE Support

InvokableCases provides full IDE autocompletion. Add parentheses to convert suggestions to primitive values:

```php
MyEnum::FOO;   // MyEnum instance (IDE autocompletes)
MyEnum::FOO(); // Primitive value (add parentheses)
```
