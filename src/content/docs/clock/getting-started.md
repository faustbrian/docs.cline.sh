---
title: Getting Started
description: Clock abstraction for PHP 8.4+ with multiple implementations including Carbon, DateTime, and frozen time for testing.
---

Clock is a PSR-20 compliant clock abstraction for PHP that provides multiple implementations for different use cases, including Carbon, DateTime, and specialized testing clocks.

## Requirements

> **Requires [PHP 8.4+](https://php.net/releases/)**

## Installation

```bash
composer require cline/clock
```

The package auto-registers with Laravel via package discovery.

## Quick Example

```php
use Cline\Clock\Clocks\CarbonImmutableClock;
use Cline\Clock\Clocks\FrozenClock;
use DateTimeImmutable;

// Production: Get current time
$clock = new CarbonImmutableClock();
$now = $clock->now(); // DateTimeImmutable

// Testing: Fixed time for deterministic tests
$frozen = new FrozenClock(new DateTimeImmutable('2025-01-15 12:00:00'));
$frozen->now(); // Always returns 2025-01-15 12:00:00
```

## Helper Function

Use the `clock()` helper for quick access:

```php
use function Cline\Clock\clock;

$now = clock()->now(); // Uses CarbonImmutableClock by default

// With timezone
$clock = clock(timezone: new DateTimeZone('America/New_York'));
```

## Laravel Facade

```php
use Cline\Clock\Facades\Clock;

$now = Clock::now();
```

## Available Implementations

| Clock | Use Case |
|-------|----------|
| `CarbonImmutableClock` | Default for Laravel apps |
| `CarbonClock` | Mutable Carbon instances |
| `DateTimeImmutableClock` | Native PHP without dependencies |
| `DateTimeClock` | Native mutable DateTime |
| `UtcClock` | Always UTC timezone |
| `FrozenClock` | Fixed time for testing |
| `MockClock` | Mutable testing clock |
| `SequenceClock` | Predetermined time sequence |
| `TickClock` | Manual time advancement |
| `OffsetClock` | Time offset decorator |

## Next Steps

- [Clock Implementations](/clock/implementations/) - Detailed guide to all clock types
- [Testing Strategies](/clock/testing/) - Patterns for testing time-dependent code
- [Laravel Integration](/clock/laravel/) - Service provider, facade, and DI
- [Decorators](/clock/decorators/) - Caching and logging decorators
- [Examples](/clock/examples/) - Real-world usage patterns
