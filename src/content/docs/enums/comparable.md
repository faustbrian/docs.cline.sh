---
title: Comparable
description: Compare enum instances using convenient methods.
---

Compare enum instances using convenient methods.

## Overview

The Comparable trait provides methods for comparing enum instances:
- `is()` - Check if equal to another instance
- `isNot()` - Check if not equal to another instance
- `in()` - Check if in a list of instances
- `notIn()` - Check if not in a list of instances

## Setup

```php
use Cline\Enums\Concerns\Comparable;

enum TaskStatus: int
{
    use Comparable;

    case Incomplete = 0;
    case Completed = 1;
    case Canceled = 2;
}

enum Role
{
    use Comparable;

    case Administrator;
    case Subscriber;
    case Guest;
}
```

## Methods

### is()

Check if an enum instance equals another:

```php
$status = TaskStatus::Incomplete;

$status->is(TaskStatus::Incomplete); // true
$status->is(TaskStatus::Completed); // false

$role = Role::Administrator;
$role->is(Role::Administrator); // true
$role->is(Role::Subscriber); // false
```

### isNot()

Check if an enum instance does not equal another:

```php
$status = TaskStatus::Incomplete;

$status->isNot(TaskStatus::Incomplete); // false
$status->isNot(TaskStatus::Completed); // true

$role = Role::Administrator;
$role->isNot(Role::Administrator); // false
$role->isNot(Role::Subscriber); // true
```

### in()

Check if an enum instance is in a list:

```php
$status = TaskStatus::Incomplete;

$status->in([TaskStatus::Incomplete, TaskStatus::Completed]); // true
$status->in([TaskStatus::Completed, TaskStatus::Canceled]); // false

$role = Role::Administrator;
$role->in([Role::Administrator, Role::Guest]); // true
$role->in([Role::Subscriber, Role::Guest]); // false
```

### notIn()

Check if an enum instance is not in a list:

```php
$status = TaskStatus::Incomplete;

$status->notIn([TaskStatus::Incomplete, TaskStatus::Completed]); // false
$status->notIn([TaskStatus::Completed, TaskStatus::Canceled]); // true

$role = Role::Administrator;
$role->notIn([Role::Administrator, Role::Guest]); // false
$role->notIn([Role::Subscriber, Role::Guest]); // true
```

## Use Cases

### State Machine Logic

```php
class Task
{
    public function canTransition(TaskStatus $to): bool
    {
        return match($this->status) {
            TaskStatus::Incomplete => $to->in([
                TaskStatus::Completed,
                TaskStatus::Canceled
            ]),
            TaskStatus::Completed => $to->is(TaskStatus::Incomplete),
            TaskStatus::Canceled => false,
        };
    }
}
```

### Permission Checks

```php
class UserController
{
    public function edit(User $user)
    {
        if ($user->role->isNot(Role::Administrator)) {
            abort(403);
        }

        // Edit logic...
    }
}
```

### Filtering Collections

```php
$activeTasks = $tasks->filter(function ($task) {
    return $task->status->notIn([
        TaskStatus::Completed,
        TaskStatus::Canceled
    ]);
});
```

### Conditional Rendering

```php
@if($order->status->in([OrderStatus::Pending, OrderStatus::Processing]))
    <button>Cancel Order</button>
@endif

@if($user->role->is(Role::Administrator))
    <a href="/admin">Admin Panel</a>
@endif
```

### Validation Rules

```php
public function rules(): array
{
    return [
        'status' => [
            'required',
            function ($attribute, $value, $fail) {
                $status = TaskStatus::tryFrom($value);

                if (!$status || $status->in([TaskStatus::Archived, TaskStatus::Deleted])) {
                    $fail('The selected status is invalid.');
                }
            }
        ]
    ];
}
```

## Benefits

The Comparable trait provides cleaner syntax than native PHP comparisons:

1. **Improved readability** - `$status->is(TaskStatus::Completed)` vs `$status === TaskStatus::Completed`
2. **Simplified array operations** - `in()` and `notIn()` replace verbose `in_array()` calls
3. **Chainable conditions** - Compose complex logic flows efficiently
4. **Consistent API** - Aligns with patterns from other libraries and frameworks
