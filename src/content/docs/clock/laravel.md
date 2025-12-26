---
title: Laravel Integration
description: Service provider, facade, dependency injection, and testing patterns for Laravel applications.
---

The clock package integrates seamlessly with Laravel through automatic service provider registration and facade support.

## Installation

The service provider and facade are automatically registered via Laravel's package auto-discovery:

```bash
composer require cline/clock
```

No manual configuration required.

## Dependency Injection

The preferred method is injecting `ClockInterface` into your classes:

```php
use Cline\Clock\Contracts\ClockInterface;

class OrderController extends Controller
{
    public function __construct(
        private readonly ClockInterface $clock
    ) {}

    public function store(Request $request)
    {
        $order = Order::create([
            'user_id' => $request->user()->id,
            'total' => $request->total,
            'created_at' => $this->clock->now(),
        ]);

        return response()->json($order, 201);
    }
}
```

## Facade

For convenience, use the `Clock` facade:

```php
use Cline\Clock\Facades\Clock;

class ReportService
{
    public function generateReport(): array
    {
        return [
            'generated_at' => Clock::now(),
            'data' => $this->fetchData(),
        ];
    }
}
```

## Service Container

Access the clock through the service container:

```php
// Using interface
$clock = app(ClockInterface::class);

// Using alias
$clock = app('clock');
```

## Customizing the Default Clock

Override the binding in a service provider:

```php
use Cline\Clock\Clocks\UtcClock;
use Cline\Clock\Contracts\ClockInterface;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Use UTC clock instead of CarbonImmutableClock
        $this->app->singleton(
            ClockInterface::class,
            fn() => new UtcClock()
        );
    }
}
```

## Testing

Override the clock binding in your tests:

```php
use Cline\Clock\Clocks\FrozenClock;
use Cline\Clock\Contracts\ClockInterface;

class OrderTest extends TestCase
{
    protected function setUp(): void
    {
        parent::setUp();

        $this->app->singleton(
            ClockInterface::class,
            fn() => new FrozenClock(new DateTimeImmutable('2025-01-15 12:00:00'))
        );
    }

    public function test_creates_order_with_fixed_timestamp(): void
    {
        $response = $this->postJson('/api/orders', [
            'item' => 'widget',
        ]);

        $response->assertStatus(201)
            ->assertJson(['created_at' => '2025-01-15T12:00:00+00:00']);
    }
}
```

## Middleware Example

```php
use Cline\Clock\Contracts\ClockInterface;
use Closure;
use Illuminate\Http\Request;

class RateLimitMiddleware
{
    public function __construct(
        private readonly ClockInterface $clock,
        private readonly RateLimiter $limiter,
    ) {}

    public function handle(Request $request, Closure $next)
    {
        $key = $request->user()?->id ?? $request->ip();

        if ($this->limiter->tooManyAttempts($key)) {
            return response()->json([
                'error' => 'Too many requests',
                'retry_after' => $this->limiter->availableAt($key),
            ], 429);
        }

        $this->limiter->hit($key, $this->clock->now());

        return $next($request);
    }
}
```

## Service Class Example

```php
use Cline\Clock\Contracts\ClockInterface;

class SubscriptionService
{
    public function __construct(
        private readonly ClockInterface $clock
    ) {}

    public function isActive(Subscription $subscription): bool
    {
        $now = $this->clock->now();

        return $subscription->starts_at <= $now
            && $subscription->ends_at >= $now;
    }

    public function daysRemaining(Subscription $subscription): int
    {
        $now = $this->clock->now();

        if ($subscription->ends_at < $now) {
            return 0;
        }

        return $now->diff($subscription->ends_at)->days;
    }

    public function renew(Subscription $subscription): Subscription
    {
        $subscription->update([
            'ends_at' => $this->clock->now()->modify('+1 year'),
            'renewed_at' => $this->clock->now(),
        ]);

        return $subscription->fresh();
    }
}
```

## Artisan Command Example

```php
use Cline\Clock\Contracts\ClockInterface;
use Illuminate\Console\Command;

class CleanupExpiredOrdersCommand extends Command
{
    protected $signature = 'orders:cleanup';
    protected $description = 'Delete expired orders';

    public function __construct(
        private readonly ClockInterface $clock
    ) {
        parent::__construct();
    }

    public function handle(): int
    {
        $deleted = Order::query()
            ->where('expires_at', '<', $this->clock->now())
            ->delete();

        $this->info("Deleted {$deleted} expired orders");

        return self::SUCCESS;
    }
}
```

## Queued Job Example

```php
use Cline\Clock\Contracts\ClockInterface;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;

class ProcessSubscriptionRenewalJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable;

    public function __construct(
        private readonly int $subscriptionId
    ) {}

    public function handle(ClockInterface $clock): void
    {
        $subscription = Subscription::findOrFail($this->subscriptionId);

        if ($subscription->ends_at > $clock->now()) {
            // Not yet expired, reschedule
            $this->release($subscription->ends_at->diffInSeconds($clock->now()));
            return;
        }

        $subscription->update([
            'starts_at' => $clock->now(),
            'ends_at' => $clock->now()->modify('+1 year'),
        ]);
    }
}
```

## Event Listener Example

```php
use Cline\Clock\Contracts\ClockInterface;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendOrderConfirmationEmail implements ShouldQueue
{
    public function __construct(
        private readonly ClockInterface $clock
    ) {}

    public function handle(OrderCreated $event): void
    {
        Mail::to($event->order->user)->send(
            new OrderConfirmation(
                order: $event->order,
                sentAt: $this->clock->now()
            )
        );

        Log::info("Order confirmation sent", [
            'order_id' => $event->order->id,
            'sent_at' => $this->clock->now()->format('Y-m-d H:i:s'),
        ]);
    }
}
```

## Testing Queued Jobs

```php
use Cline\Clock\Clocks\FrozenClock;
use Cline\Clock\Contracts\ClockInterface;

test('processes renewal at correct time', function () {
    $fixedTime = new DateTimeImmutable('2025-01-15 12:00:00');

    $this->app->singleton(
        ClockInterface::class,
        fn() => new FrozenClock($fixedTime)
    );

    $subscription = Subscription::factory()->create([
        'ends_at' => new DateTimeImmutable('2025-01-14 12:00:00'),
    ]);

    ProcessSubscriptionRenewalJob::dispatch($subscription->id);
    $this->artisan('queue:work --once');

    $subscription->refresh();

    expect($subscription->starts_at)->toEqual($fixedTime);
    expect($subscription->ends_at)->toEqual($fixedTime->modify('+1 year'));
});
```

## Best Practices

1. **Always use dependency injection** - Don't call `app(ClockInterface::class)` in business logic
2. **Override in tests** - Use `FrozenClock` or `MockClock` in test setup
3. **Type hint the interface** - Use `ClockInterface`, not concrete implementations
4. **Use the facade sparingly** - Prefer constructor injection for better testability
5. **Test time-dependent logic** - Always write tests for code that depends on time
