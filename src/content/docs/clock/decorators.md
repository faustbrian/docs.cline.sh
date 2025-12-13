---
title: Decorators
description: Caching and logging decorators for clock implementations.
---

The clock package includes decorator classes that wrap any `ClockInterface` implementation to add caching or logging capabilities.

## CachingClock

Caches time values for a configurable TTL period, reducing overhead from repeated time calculations.

```php
use Cline\Clock\Decorators\CachingClock;
use Cline\Clock\Clocks\CarbonImmutableClock;

$baseClock = new CarbonImmutableClock();
$clock = new CachingClock($baseClock, ttlSeconds: 5);

$first = $clock->now();  // Fetches from base clock
$second = $clock->now(); // Returns cached value
$third = $clock->now();  // Returns cached value

// After 5 seconds...
$fourth = $clock->now(); // Fetches fresh value
```

### Configuration

```php
// 1 second TTL (default)
$clock = new CachingClock($baseClock);

// Custom TTL
$clock = new CachingClock($baseClock, ttlSeconds: 10);
```

### Clear Cache

Force the next call to fetch a fresh value:

```php
$clock->clear();
$fresh = $clock->now(); // Fetches from base clock
```

### Use Cases

- High-frequency time checks where microsecond precision isn't required
- Reducing system calls in performance-critical code
- Batch operations that should use the same timestamp

## LoggingClock

Logs every time retrieval to a PSR-3 logger with detailed context.

```php
use Cline\Clock\Decorators\LoggingClock;
use Cline\Clock\Clocks\CarbonImmutableClock;
use Psr\Log\LoggerInterface;

$baseClock = new CarbonImmutableClock();
$logger = app(LoggerInterface::class);

$clock = new LoggingClock($baseClock, $logger);

$clock->now();
// Logs: "Clock returned time" with context:
// - timestamp: "2025-01-15 12:00:00.123456"
// - timezone: "UTC"
// - clock_class: "Cline\Clock\Clocks\CarbonImmutableClock"
```

### Log Level

Configure the PSR-3 log level:

```php
// Default: debug
$clock = new LoggingClock($baseClock, $logger);

// Custom level
$clock = new LoggingClock($baseClock, $logger, level: 'info');
```

Available levels: `emergency`, `alert`, `critical`, `error`, `warning`, `notice`, `info`, `debug`

### Log Output

Each call to `now()` produces a log entry:

```json
{
    "message": "Clock returned time",
    "context": {
        "timestamp": "2025-01-15 12:00:00.123456",
        "timezone": "America/New_York",
        "clock_class": "Cline\\Clock\\Clocks\\CarbonImmutableClock"
    },
    "level": "debug"
}
```

### Use Cases

- Debugging time-related issues
- Auditing time access patterns
- Monitoring clock usage in production
- Tracing time flow through complex operations

## Combining Decorators

Decorators can be stacked:

```php
use Cline\Clock\Decorators\CachingClock;
use Cline\Clock\Decorators\LoggingClock;
use Cline\Clock\Clocks\CarbonImmutableClock;

$baseClock = new CarbonImmutableClock();

// Log first, then cache
$logged = new LoggingClock($baseClock, $logger);
$cached = new CachingClock($logged, ttlSeconds: 5);

// Or cache first, then log (logs cache hits too)
$cached = new CachingClock($baseClock, ttlSeconds: 5);
$logged = new LoggingClock($cached, $logger);
```

The order matters:
- **Log then Cache**: Only logs cache misses (when base clock is called)
- **Cache then Log**: Logs every access including cache hits

## Creating Custom Decorators

Implement `ClockInterface` and wrap another clock:

```php
use Cline\Clock\Contracts\ClockInterface;
use DateTimeImmutable;

final readonly class MetricsClock implements ClockInterface
{
    public function __construct(
        private ClockInterface $clock,
        private MetricsCollector $metrics,
    ) {}

    public function now(): DateTimeImmutable
    {
        $this->metrics->increment('clock.calls');

        $start = microtime(true);
        $result = $this->clock->now();
        $duration = microtime(true) - $start;

        $this->metrics->timing('clock.duration', $duration);

        return $result;
    }
}
```

## Laravel Service Provider Example

Register decorated clocks:

```php
use Cline\Clock\Clocks\CarbonImmutableClock;
use Cline\Clock\Contracts\ClockInterface;
use Cline\Clock\Decorators\CachingClock;
use Cline\Clock\Decorators\LoggingClock;
use Illuminate\Support\ServiceProvider;
use Psr\Log\LoggerInterface;

class ClockServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton(ClockInterface::class, function ($app) {
            $clock = new CarbonImmutableClock();

            if ($app->environment('production')) {
                // Cache in production
                $clock = new CachingClock($clock, ttlSeconds: 1);
            }

            if (config('clock.logging', false)) {
                // Add logging when enabled
                $clock = new LoggingClock(
                    $clock,
                    $app->make(LoggerInterface::class),
                    level: 'debug'
                );
            }

            return $clock;
        });
    }
}
```
