---
title: Testing Strategies
description: Patterns and strategies for testing time-dependent code with deterministic results.
---

Testing time-dependent code can be challenging. The clock package provides multiple strategies to make your tests deterministic, fast, and reliable.

## Basic Testing with FrozenClock

The simplest approach is fixing time at a specific point:

```php
use Cline\Clock\Clocks\FrozenClock;
use DateTimeImmutable;

test('order expires after 24 hours', function () {
    $fixedTime = new DateTimeImmutable('2025-01-15 12:00:00');
    $clock = new FrozenClock($fixedTime);

    $order = new Order($clock);
    $order->setExpiresAt(new DateTimeImmutable('2025-01-15 13:00:00'));

    expect($order->isExpired())->toBeFalse();
});
```

## Dependency Injection Pattern

Always inject the clock as a dependency:

```php
class OrderService
{
    public function __construct(
        private readonly ClockInterface $clock
    ) {}

    public function createOrder(array $data): Order
    {
        return new Order(
            data: $data,
            createdAt: $this->clock->now(),
        );
    }
}

// Test
test('creates order with current timestamp', function () {
    $fixedTime = new DateTimeImmutable('2025-01-15 12:00:00');
    $clock = new FrozenClock($fixedTime);
    $service = new OrderService($clock);

    $order = $service->createOrder(['id' => 1]);

    expect($order->createdAt)->toEqual($fixedTime);
});
```

## Testing Time Progression

Use `MockClock` for scenarios involving time advancement:

```php
use Cline\Clock\Clocks\MockClock;

test('session expires after timeout', function () {
    $clock = new MockClock(new DateTimeImmutable('2025-01-15 12:00:00'));
    $session = new Session($clock, timeoutSeconds: 3600);

    $session->start();
    expect($session->isActive())->toBeTrue();

    // Advance 30 minutes
    $clock->advance('+30 minutes');
    expect($session->isActive())->toBeTrue();

    // Advance another 31 minutes (total 61 minutes)
    $clock->advance('+31 minutes');
    expect($session->isActive())->toBeFalse();
});
```

## Testing Ordered Operations

Use `SequenceClock` when testing operations in sequence:

```php
use Cline\Clock\Clocks\SequenceClock;

test('processes batch with incremental timestamps', function () {
    $times = [
        new DateTimeImmutable('2025-01-15 10:00:00'),
        new DateTimeImmutable('2025-01-15 10:01:00'),
        new DateTimeImmutable('2025-01-15 10:02:00'),
    ];

    $clock = new SequenceClock($times);
    $processor = new BatchProcessor($clock);

    $results = $processor->processBatch([
        ['id' => 1],
        ['id' => 2],
        ['id' => 3],
    ]);

    expect($results[0]->processedAt)->toEqual($times[0]);
    expect($results[1]->processedAt)->toEqual($times[1]);
    expect($results[2]->processedAt)->toEqual($times[2]);
});
```

## Testing with TickClock

For regular interval testing:

```php
use Cline\Clock\Clocks\TickClock;

test('scheduler runs tasks at intervals', function () {
    $clock = new TickClock(new DateTimeImmutable('2025-01-15 12:00:00'));
    $scheduler = new TaskScheduler($clock);

    $scheduler->schedule('cleanup', fn() => null, intervalSeconds: 900);

    // First run - should execute
    expect($scheduler->run())->toContain('cleanup');

    // 10 minutes later - should not run (interval is 15 min)
    $clock->tick('+10 minutes');
    expect($scheduler->run())->not()->toContain('cleanup');

    // 5 more minutes - should run
    $clock->tick('+5 minutes');
    expect($scheduler->run())->toContain('cleanup');
});
```

## Testing Future/Past Scenarios

Use `OffsetClock` without complex date math:

```php
use Cline\Clock\Clocks\OffsetClock;
use Cline\Clock\Clocks\FrozenClock;

test('coupon expires in the future', function () {
    $baseClock = new FrozenClock(new DateTimeImmutable('2025-01-15 12:00:00'));
    $futureClock = new OffsetClock($baseClock, '+7 days');

    $coupon = new Coupon(
        expiresAt: new DateTimeImmutable('2025-01-20 12:00:00')
    );

    expect($coupon->isValidAt($baseClock->now()))->toBeTrue();
    expect($coupon->isValidAt($futureClock->now()))->toBeFalse();
});
```

## Laravel Integration Testing

Override the clock binding in Laravel tests:

```php
use Cline\Clock\Clocks\FrozenClock;
use Cline\Clock\Contracts\ClockInterface;

test('creates order with frozen time', function () {
    $fixedTime = new DateTimeImmutable('2025-01-15 12:00:00');

    $this->app->singleton(
        ClockInterface::class,
        fn() => new FrozenClock($fixedTime)
    );

    $response = $this->postJson('/api/orders', ['item' => 'widget']);

    $response->assertStatus(201)
        ->assertJson([
            'created_at' => '2025-01-15T12:00:00+00:00'
        ]);
});
```

## Feature Tests with Time Progression

```php
use Cline\Clock\Clocks\MockClock;
use Cline\Clock\Contracts\ClockInterface;

test('session expires after timeout', function () {
    $clock = new MockClock(new DateTimeImmutable('2025-01-15 12:00:00'));
    $this->app->singleton(ClockInterface::class, fn() => $clock);

    $this->post('/login', [
        'email' => 'user@example.com',
        'password' => 'password',
    ])->assertOk();

    expect($this->isAuthenticated())->toBeTrue();

    // Advance past session timeout
    $clock->advance('+2 hours');

    $this->get('/dashboard')->assertRedirect('/login');
});
```

## Testing Timezone Behavior

```php
use Cline\Clock\Clocks\CarbonImmutableClock;

test('handles timezone conversion', function () {
    $nyClock = new CarbonImmutableClock(new DateTimeZone('America/New_York'));
    $utcClock = new Cline\Clock\Clocks\UtcClock();

    $nyTime = $nyClock->now();
    $utcTime = $utcClock->now();

    // Same moment, different representation
    expect($nyTime->getTimestamp())->toBe($utcTime->getTimestamp());
    expect($nyTime->getTimezone()->getName())->not()->toBe('UTC');
});
```

## Complete Workflow Test

```php
use Cline\Clock\Clocks\MockClock;

test('complete subscription workflow', function () {
    $clock = new MockClock(new DateTimeImmutable('2025-01-01 00:00:00'));

    $subscription = new Subscription(
        clock: $clock,
        startsAt: new DateTimeImmutable('2025-01-15 00:00:00'),
        endsAt: new DateTimeImmutable('2026-01-15 00:00:00'),
    );

    // Before start
    expect($subscription->isPending())->toBeTrue();
    expect($subscription->isActive())->toBeFalse();

    // Fast forward to start
    $clock->freezeAt('2025-01-15 00:00:00');
    expect($subscription->isActive())->toBeTrue();
    expect($subscription->daysRemaining())->toBe(365);

    // Fast forward 6 months
    $clock->advance('+180 days');
    expect($subscription->daysRemaining())->toBe(185);

    // Near expiration
    $clock->advance('+178 days');
    expect($subscription->daysRemaining())->toBe(7);

    // Renew
    $renewed = $subscription->renew();
    expect($renewed->isActive())->toBeTrue();
});
```

## Best Practices

### Always Use Dependency Injection

```php
// Good
class OrderService
{
    public function __construct(
        private readonly ClockInterface $clock
    ) {}
}

// Bad - Hard to test
class OrderService
{
    public function getCurrentTime(): DateTimeImmutable
    {
        return new DateTimeImmutable();
    }
}
```

### Type Hint the Interface

```php
public function __construct(
    private readonly ClockInterface $clock // PSR-20 interface
) {}
```

### Choose the Right Clock

| Test Type | Clock |
|-----------|-------|
| Fixed timestamp | `FrozenClock` |
| Time progression | `MockClock` |
| Ordered operations | `SequenceClock` |
| Regular intervals | `TickClock` |
| Future/past | `OffsetClock` |

### Test Edge Cases

```php
test('handles leap year', function () {
    $clock = new FrozenClock(new DateTimeImmutable('2024-02-29 12:00:00'));
    expect($clock->now()->format('Y-m-d'))->toBe('2024-02-29');
});

test('handles year boundary', function () {
    $clock = new FrozenClock(new DateTimeImmutable('2024-12-31 23:59:59'));
    expect($clock->now()->format('Y'))->toBe('2024');
});

test('handles daylight saving transition', function () {
    $clock = new CarbonImmutableClock(new DateTimeZone('America/New_York'));
    // Spring forward - 2:00 AM becomes 3:00 AM
    $frozen = new FrozenClock(new DateTimeImmutable('2025-03-09 01:59:59', new DateTimeZone('America/New_York')));
    // Test your DST-sensitive logic here
});
```
