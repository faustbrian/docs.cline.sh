---
title: Metadata
description: Add custom properties and behaviors to enum cases using attributes.
---

Add custom properties and behaviors to enum cases using attributes.

## Overview

The Metadata trait allows you to:
- Attach custom properties to enum cases
- Access properties via dynamic methods
- Find cases by metadata values
- Transform property values automatically

## Basic Setup

```php
use Cline\Enums\Concerns\Metadata;
use Cline\Enums\Meta\Meta;
use Cline\Enums\Meta\MetaProperty;

// Define your meta properties
#[Attribute]
class Description extends MetaProperty {}

#[Attribute]
class Color extends MetaProperty {}

// Apply to your enum
#[Meta(Description::class, Color::class)]
enum TaskStatus: int
{
    use Metadata;

    #[Description('Incomplete Task')] #[Color('red')]
    case Incomplete = 0;

    #[Description('Completed Task')] #[Color('green')]
    case Completed = 1;

    #[Description('Canceled Task')] #[Color('gray')]
    case Canceled = 2;
}
```

## Accessing Metadata

```php
TaskStatus::Incomplete->description(); // 'Incomplete Task'
TaskStatus::Completed->color(); // 'green'
```

## Creating Meta Properties

Each meta property is a class extending `MetaProperty`:

```php
#[Attribute]
class Color extends MetaProperty {}

#[Attribute]
class Description extends MetaProperty {}

#[Attribute]
class Icon extends MetaProperty {}
```

## Advanced Features

### Custom Method Names

Override the default method name:

```php
#[Attribute]
class Description extends MetaProperty
{
    public static function method(): string
    {
        return 'note'; // Use ->note() instead of ->description()
    }
}
```

### Value Transformation

Transform values before they're returned:

```php
#[Attribute]
class Color extends MetaProperty
{
    protected function transform(mixed $value): mixed
    {
        return "text-{$value}-500"; // 'green' becomes 'text-green-500'
    }
}
```

### Default Values

Provide defaults for cases without the attribute:

```php
#[Attribute]
class Priority extends MetaProperty
{
    public function defaultValue(): mixed
    {
        return 'normal'; // Cases without Priority attribute return 'normal'
    }
}
```

## Finding Cases by Metadata

### fromMeta()

Find a case by its metadata value (throws `ValueError` if not found):

```php
TaskStatus::fromMeta(Color::make('green')); // TaskStatus::COMPLETED
TaskStatus::fromMeta(Color::make('blue')); // Error: ValueError
```

### tryFromMeta()

Find a case by its metadata value (returns `null` if not found):

```php
TaskStatus::tryFromMeta(Color::make('green')); // TaskStatus::COMPLETED
TaskStatus::tryFromMeta(Color::make('blue')); // null
```

## Real-World Examples

### UI Configuration

```php
#[Attribute]
class Color extends MetaProperty {}

#[Attribute]
class Icon extends MetaProperty {}

#[Attribute]
class Label extends MetaProperty {}

#[Meta(Color::class, Icon::class, Label::class)]
enum OrderStatus: string
{
    use Metadata;

    #[Color('yellow')] #[Icon('clock')] #[Label('Pending')]
    case Pending = 'pending';

    #[Color('blue')] #[Icon('truck')] #[Label('Shipped')]
    case Shipped = 'shipped';

    #[Color('green')] #[Icon('check')] #[Label('Delivered')]
    case Delivered = 'delivered';
}

// In your view
<span class="text-<?= $order->status->color() ?>-500">
    <i class="icon-<?= $order->status->icon() ?>"></i>
    <?= $order->status->label() ?>
</span>
```

### Permissions & Roles

```php
#[Attribute]
class Permissions extends MetaProperty
{
    protected function transform(mixed $value): array
    {
        return is_string($value) ? explode(',', $value) : $value;
    }
}

#[Meta(Permissions::class)]
enum Role: string
{
    use Metadata;

    #[Permissions('read')]
    case GUEST = 'guest';

    #[Permissions('read,write')]
    case USER = 'user';

    #[Permissions('read,write,delete,admin')]
    case ADMIN = 'admin';
}

if (in_array('write', $user->role->permissions())) {
    // User can write
}
```

### State Machine Transitions

```php
#[Attribute]
class AllowedTransitions extends MetaProperty {}

#[Meta(AllowedTransitions::class)]
enum OrderStatus: string
{
    use Metadata;

    #[AllowedTransitions(['shipped', 'canceled'])]
    case Pending = 'pending';

    #[AllowedTransitions(['delivered', 'returned'])]
    case Shipped = 'shipped';

    #[AllowedTransitions(['returned'])]
    case Delivered = 'delivered';
}

$canTransitionTo = in_array(
    $newStatus->value,
    $currentStatus->allowedTransitions()
);
```

## IDE Support

Add `@method` annotations for better IDE support:

```php
/**
 * @method string description()
 * @method string color()
 * @method string icon()
 */
#[Meta(Description::class, Color::class, Icon::class)]
enum TaskStatus: int
{
    use Metadata;
    // ...
}
```

Or create a trait for reusable metadata:

```php
/**
 * @method string color()
 * @method string icon()
 */
trait HasUIMetadata
{
    use Metadata;
}

#[Meta(Color::class, Icon::class)]
enum TaskStatus: int
{
    use HasUIMetadata;
    // ...
}
```
