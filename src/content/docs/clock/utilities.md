---
title: Utilities
description: Helper function, clock registry, and comparison utilities.
---

The clock package includes several utilities to simplify common operations.

## Helper Function

The `clock()` function provides quick access to clock instances:

```php
use function Cline\Clock\clock;
use Cline\Clock\Clocks\FrozenClock;
use DateTimeImmutable;
use DateTimeZone;

// Default CarbonImmutableClock
$clock = clock();
$now = $clock->now();

// With timezone
$clock = clock(timezone: new DateTimeZone('America/New_York'));

// Pass existing clock (returned as-is)
$frozen = new FrozenClock(new DateTimeImmutable('2025-01-15'));
$same = clock($frozen); // Returns $frozen

// Create specific clock type
$clock = clock(FrozenClock::class, frozenTime: new DateTimeImmutable('2025-01-15'));
```

## Clock Registry

Global registry for managing named clock instances:

```php
use Cline\Clock\Support\ClockRegistry;
use Cline\Clock\Clocks\CarbonImmutableClock;
use Cline\Clock\Clocks\UtcClock;
use Cline\Clock\Clocks\FrozenClock;

// Register clocks
ClockRegistry::set('local', new CarbonImmutableClock());
ClockRegistry::set('utc', new UtcClock());

// Set default
ClockRegistry::setDefault('local');

// Retrieve
$local = ClockRegistry::get('local');
$default = ClockRegistry::getDefault();

// Check existence
ClockRegistry::has('utc'); // true
ClockRegistry::hasDefault(); // true

// List all registered
ClockRegistry::registered(); // ['local', 'utc']

// Remove
ClockRegistry::remove('utc');

// Clear all (useful in tests)
ClockRegistry::clear();
```

### Registry Exceptions

```php
// Throws RuntimeException
ClockRegistry::get('nonexistent');

// Throws RuntimeException
ClockRegistry::setDefault('nonexistent');

// Throws RuntimeException
ClockRegistry::getDefault(); // When no default set
```

### Multi-Clock Application

```php
use Cline\Clock\Support\ClockRegistry;
use Cline\Clock\Clocks\CarbonImmutableClock;
use Cline\Clock\Clocks\UtcClock;
use DateTimeZone;

// Bootstrap in service provider
ClockRegistry::set('app', new CarbonImmutableClock());
ClockRegistry::set('utc', new UtcClock());
ClockRegistry::set('tokyo', new CarbonImmutableClock(new DateTimeZone('Asia/Tokyo')));
ClockRegistry::setDefault('app');

// Use throughout application
class OrderService
{
    public function createOrder(array $data): Order
    {
        return new Order(
            data: $data,
            createdAt: ClockRegistry::getDefault()->now(),
            createdAtUtc: ClockRegistry::get('utc')->now(),
        );
    }
}
```

## Clock Comparison Trait

The `ClockComparison` trait adds comparison methods to clock implementations:

```php
use Cline\Clock\Contracts\ClockInterface;
use Cline\Clock\Support\ClockComparison;
use DateTimeImmutable;

final class MyCustomClock implements ClockInterface
{
    use ClockComparison;

    public function now(): DateTimeImmutable
    {
        return new DateTimeImmutable();
    }
}

$clock = new MyCustomClock();
$reference = new DateTimeImmutable('2025-01-15 12:00:00');

// Comparisons
$clock->isAfter($reference);  // true if now > reference
$clock->isBefore($reference); // true if now < reference
$clock->isSameAs($reference); // true if same timestamp
$clock->isBetween($start, $end); // true if within range (inclusive)

// Differences
$clock->diffInSeconds($reference);
$clock->diffInMinutes($reference);
$clock->diffInHours($reference);
$clock->diffInDays($reference);
```

### Comparison Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `isAfter($time)` | `bool` | Clock time is after given time |
| `isBefore($time)` | `bool` | Clock time is before given time |
| `isSameAs($time)` | `bool` | Same Unix timestamp |
| `isBetween($start, $end)` | `bool` | Within range (inclusive) |
| `diffInSeconds($time)` | `int` | Absolute difference in seconds |
| `diffInMinutes($time)` | `int` | Absolute difference in minutes |
| `diffInHours($time)` | `int` | Absolute difference in hours |
| `diffInDays($time)` | `int` | Absolute difference in days |

### Example Usage

```php
use Cline\Clock\Clocks\FrozenClock;
use Cline\Clock\Support\ClockComparison;
use DateTimeImmutable;

// Create a clock that uses the comparison trait
final class ComparableFrozenClock extends FrozenClock
{
    use ClockComparison;
}

$clock = new ComparableFrozenClock(new DateTimeImmutable('2025-01-15 12:00:00'));

$past = new DateTimeImmutable('2025-01-10 12:00:00');
$future = new DateTimeImmutable('2025-01-20 12:00:00');

// Time comparisons
$clock->isAfter($past);   // true
$clock->isBefore($future); // true
$clock->isBetween($past, $future); // true

// Differences
$clock->diffInDays($past);   // 5
$clock->diffInDays($future); // 5 (absolute)
```

## Freezable Interface

Production clocks implement `FreezableInterface` for creating frozen snapshots:

```php
use Cline\Clock\Clocks\CarbonImmutableClock;

$clock = new CarbonImmutableClock();

// Create frozen snapshot
$frozen = $clock->freeze();

// Original continues
sleep(1);
$clock->now();  // Current time
$frozen->now(); // Time when freeze() was called
```

### Implementing Freezable

```php
use Cline\Clock\Contracts\ClockInterface;
use Cline\Clock\Contracts\FreezableInterface;
use Cline\Clock\Clocks\FrozenClock;
use DateTimeImmutable;

final readonly class MyCustomClock implements ClockInterface, FreezableInterface
{
    public function now(): DateTimeImmutable
    {
        return new DateTimeImmutable();
    }

    public function freeze(): FrozenClock
    {
        return new FrozenClock($this->now());
    }
}
```
