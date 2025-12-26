---
title: Clock Implementations
description: Comprehensive guide to all available clock implementations and when to use each one.
---

The clock package provides multiple implementations of the PSR-20 `ClockInterface`, each designed for specific use cases.

## Production Clocks

### CarbonImmutableClock

The default clock for Laravel applications. Uses Carbon's immutable datetime instances.

```php
use Cline\Clock\Clocks\CarbonImmutableClock;
use DateTimeZone;

$clock = new CarbonImmutableClock();
$now = $clock->now();

// With timezone
$clock = new CarbonImmutableClock(new DateTimeZone('America/New_York'));
```

**Best for:** Laravel applications, when you need Carbon's rich API.

### CarbonClock

Uses Laravel's Date facade with mutable Carbon instances.

```php
use Cline\Clock\Clocks\CarbonClock;

$clock = new CarbonClock();
$now = $clock->now(); // Converted to DateTimeImmutable
```

**Best for:** Legacy code requiring mutable Carbon instances.

### DateTimeImmutableClock

Uses PHP's native `DateTimeImmutable` without third-party dependencies.

```php
use Cline\Clock\Clocks\DateTimeImmutableClock;

$clock = new DateTimeImmutableClock();
$now = $clock->now();
```

**Best for:** Lightweight applications, maximum compatibility.

### DateTimeClock

Uses PHP's native mutable `DateTime` class.

```php
use Cline\Clock\Clocks\DateTimeClock;

$clock = new DateTimeClock();
$now = $clock->now(); // Converted to DateTimeImmutable
```

**Best for:** Legacy code requiring mutable DateTime.

### UtcClock

Always returns time in UTC timezone regardless of system configuration.

```php
use Cline\Clock\Clocks\UtcClock;

$clock = new UtcClock();
$now = $clock->now(); // Always UTC
```

**Best for:** Distributed systems, API servers, database timestamps, timezone-independent operations.

## Testing Clocks

### FrozenClock

Returns a fixed point in time. Perfect for deterministic testing.

```php
use Cline\Clock\Clocks\FrozenClock;
use DateTimeImmutable;

$fixedTime = new DateTimeImmutable('2025-01-15 12:00:00');
$clock = new FrozenClock($fixedTime);

$clock->now(); // Always 2025-01-15 12:00:00
$clock->now(); // Still 2025-01-15 12:00:00

// From string
$clock = FrozenClock::fromString('2025-01-15 12:00:00');
```

**Best for:** Unit tests requiring fixed timestamps, reproducible test scenarios.

### MockClock

Combines frozen time with manual advancement and sequencing capabilities.

```php
use Cline\Clock\Clocks\MockClock;
use DateTimeImmutable;

$clock = new MockClock(new DateTimeImmutable('2025-01-15 12:00:00'));

// Get current time
$clock->now(); // 2025-01-15 12:00:00

// Freeze at specific time
$clock->freezeAt('2025-06-01 00:00:00');
$clock->now(); // 2025-06-01 00:00:00

// Advance time
$clock->advance('+2 hours');
$clock->now(); // 2025-06-01 02:00:00

// Use DateInterval
$clock->advance(new DateInterval('P1D'));
$clock->now(); // 2025-06-02 02:00:00

// Sequence mode
$clock->useSequence([
    new DateTimeImmutable('2025-01-15 10:00:00'),
    new DateTimeImmutable('2025-01-15 11:00:00'),
    new DateTimeImmutable('2025-01-15 12:00:00'),
]);
$clock->now(); // 10:00:00
$clock->now(); // 11:00:00
$clock->now(); // 12:00:00

// Reset
$clock->reset();
```

**Best for:** Complex testing scenarios, time progression tests, integration tests.

### SequenceClock

Returns a predetermined sequence of datetime values. Throws when exhausted.

```php
use Cline\Clock\Clocks\SequenceClock;
use DateTimeImmutable;

$times = [
    new DateTimeImmutable('2025-01-15 10:00:00'),
    new DateTimeImmutable('2025-01-15 11:00:00'),
    new DateTimeImmutable('2025-01-15 12:00:00'),
];

$clock = new SequenceClock($times);

$clock->now(); // 10:00:00
$clock->now(); // 11:00:00
$clock->now(); // 12:00:00
$clock->now(); // Throws RuntimeException

// Check if more times available
$clock->hasNext(); // false

// Reset to beginning
$clock->reset();
$clock->hasNext(); // true
```

**Best for:** Testing ordered time-dependent operations, batch processing tests.

### TickClock

Allows manual time advancement by specific intervals.

```php
use Cline\Clock\Clocks\TickClock;
use DateInterval;
use DateTimeImmutable;

$clock = new TickClock(new DateTimeImmutable('2025-01-15 12:00:00'));

$clock->now(); // 12:00:00

// Advance with string modifier
$clock->tick('+1 hour');
$clock->now(); // 13:00:00

// Advance with DateInterval
$clock->tick(new DateInterval('PT30M'));
$clock->now(); // 13:30:00

// Jump to specific time
$clock->setTo(new DateTimeImmutable('2025-02-01 00:00:00'));
$clock->now(); // 2025-02-01 00:00:00

// Reset
$clock->reset(new DateTimeImmutable('2025-01-01 00:00:00'));
```

**Best for:** Step-by-step time advancement, simulating scheduled tasks.

### OffsetClock

Wraps another clock and applies a fixed time offset.

```php
use Cline\Clock\Clocks\OffsetClock;
use Cline\Clock\Clocks\CarbonImmutableClock;
use DateInterval;

$baseClock = new CarbonImmutableClock();

// Offset with string
$futureClock = new OffsetClock($baseClock, '+7 days');
$futureClock->now(); // 7 days from now

// Offset with DateInterval
$pastClock = new OffsetClock($baseClock, DateInterval::createFromDateString('-1 month'));
$pastClock->now(); // 1 month ago

// Stack offsets
$innerOffset = new OffsetClock($baseClock, '+1 day');
$outerOffset = new OffsetClock($innerOffset, '+1 hour');
$outerOffset->now(); // 1 day and 1 hour from now
```

**Best for:** Testing future/past scenarios, simulating time zones, time-shift testing.

## Freezable Interface

Production clocks implement `FreezableInterface` for creating frozen snapshots:

```php
use Cline\Clock\Clocks\CarbonImmutableClock;

$clock = new CarbonImmutableClock();
$frozen = $clock->freeze(); // FrozenClock at current time

$clock->now();  // Continues advancing
$frozen->now(); // Fixed at freeze moment
```

## Choosing the Right Clock

### Production

| Scenario | Recommended Clock |
|----------|-------------------|
| Laravel applications | `CarbonImmutableClock` |
| Standalone PHP | `DateTimeImmutableClock` |
| Distributed systems | `UtcClock` |
| UTC database storage | `UtcClock` |

### Testing

| Scenario | Recommended Clock |
|----------|-------------------|
| Fixed timestamp | `FrozenClock` |
| Time progression | `MockClock` or `TickClock` |
| Ordered operations | `SequenceClock` |
| Future/past scenarios | `OffsetClock` |
| Complex scenarios | `MockClock` |
