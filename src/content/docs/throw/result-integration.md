---
title: Result Integration
description: Convert between `Attempt` (Try monad) and `Result` type for explicit error handling without exceptions.
---

Convert between `Attempt` (Try monad) and `Result` type for explicit error handling without exceptions.

## Overview

The `toResult()` method bridges the gap between exception-based and value-based error handling, allowing you to work with Rust-inspired Result types.

```php
use function Cline\Throwttempt;

// Convert Attempt to Result
$result = attempt(fn() => User::findOrFail($id))->toResult();
// Returns Ok<User> or Err<Throwable>
```

## Basic Usage

### Success Case (Ok)

```php
use function Cline\Throwttempt;

$result = attempt(fn() => 42)->toResult();

// Check if Ok
if ($result->isOk()) {
    echo $result->unwrap(); // 42
}
```

### Failure Case (Err)

```php
use function Cline\Throwttempt;

$result = attempt(fn() => throw new RuntimeException('Error'))
    ->toResult();

// Check if Err
if ($result->isErr()) {
    $exception = $result->unwrapErr();
    echo $exception->getMessage(); // "Error"
}
```

## Chaining with Result Methods

### Transform Success Values

```php
$name = attempt(fn() => User::find($id))
    ->toResult()
    ->map(fn($user) => $user->name)
    ->unwrapOr('Guest');
```

### Transform Errors

```php
$result = attempt(fn() => processPayment())
    ->toResult()
    ->mapErr(fn($e) => "Payment failed: {$e->getMessage()}");

if ($result->isErr()) {
    logger()->error($result->unwrapErr());
}
```

### Provide Defaults

```php
$config = attempt(fn() => loadConfig())
    ->toResult()
    ->unwrapOr(['debug' => false]);
```

## Real-World Examples

### API Request Handling

```php
class ApiClient
{
    public function fetchUser(int $id): Result
    {
        return attempt(function () use ($id) {
            $response = Http::get("/users/{$id}");

            if ($response->failed()) {
                throw new ApiException('Request failed');
            }

            return $response->json();
        })->toResult();
    }
}

// Usage
$result = $apiClient->fetchUser(123);

$user = $result
    ->map(fn($data) => new User($data))
    ->mapErr(fn($e) => logger()->error($e))
    ->unwrapOr(null);
```

### Database Operations

```php
class UserRepository
{
    public function findByEmail(string $email): Result
    {
        return attempt(fn() => User::where('email', $email)
            ->firstOrFail()
        )->toResult();
    }
}

// Usage
$repository->findByEmail('user@example.com')
    ->map(fn($user) => $user->notify(new WelcomeNotification()))
    ->mapErr(fn($e) => Log::info("User not found: {$e->getMessage()}"));
```

### File Operations

```php
function readConfigFile(string $path): Result
{
    return attempt(function () use ($path) {
        if (!file_exists($path)) {
            throw new FileException("File not found: {$path}");
        }

        $contents = file_get_contents($path);

        return json_decode($contents, true, 512, JSON_THROW_ON_ERROR);
    })->toResult();
}

// Usage with chaining
$config = readConfigFile('config.json')
    ->map(fn($data) => new Config($data))
    ->unwrapOrElse(fn($e) => Config::defaults());
```

### Service Layer

```php
class PaymentService
{
    public function charge(Order $order): Result
    {
        return attempt(function () use ($order) {
            $this->validateOrder($order);

            $payment = $this->gateway->charge(
                $order->total,
                $order->paymentMethod
            );

            $order->markAsPaid($payment);

            return $payment;
        })->toResult();
    }
}

// Usage in controller
public function processOrder(Request $request, int $orderId)
{
    $result = $this->paymentService
        ->charge(Order::find($orderId))
        ->toResult();

    return $result->match(
        ok: fn($payment) => response()->json([
            'success' => true,
            'payment_id' => $payment->id,
        ]),
        err: fn($e) => response()->json([
            'success' => false,
            'error' => $e->getMessage(),
        ], 422)
    );
}
```

## Converting to Option

You can convert Result back to Option if needed:

```php
$maybeUser = attempt(fn() => User::find($id))
    ->toResult()
    ->ok(); // Some<User> or None

$userName = $maybeUser
    ->map(fn($user) => $user->name)
    ->unwrapOr('Anonymous');
```

## Error Handling Patterns

### Explicit Error Handling

```php
$result = attempt(fn() => dangerousOperation())->toResult();

if ($result->isErr()) {
    $error = $result->unwrapErr();

    match (true) {
        $error instanceof ValidationException => handleValidation($error),
        $error instanceof DatabaseException => handleDatabase($error),
        default => handleUnknown($error),
    };
}
```

### Railway-Oriented Programming

```php
$result = attempt(fn() => validateInput($input))
    ->toResult()
    ->andThen(fn($validated) => attempt(fn() => processData($validated))->toResult())
    ->andThen(fn($processed) => attempt(fn() => saveToDatabase($processed))->toResult())
    ->map(fn($saved) => new SuccessResponse($saved))
    ->unwrapOrElse(fn($e) => new ErrorResponse($e));
```

### Collect Multiple Results

```php
$results = collect([1, 2, 3])
    ->map(fn($id) => attempt(fn() => User::findOrFail($id))->toResult());

$failures = $results->filter(fn($r) => $r->isErr());
$successes = $results->filter(fn($r) => $r->isOk());

if ($failures->isNotEmpty()) {
    logger()->warning("Failed to load users", [
        'count' => $failures->count(),
        'errors' => $failures->map(fn($r) => $r->unwrapErr()->getMessage()),
    ]);
}
```

## Best Practices

1. **Use Result for explicit error handling** - When callers need to handle errors explicitly
2. **Use Attempt for try-catch replacement** - When you want exception handling without try-catch
3. **Chain transformations** - Leverage `map()` and `mapErr()` for clean pipelines
4. **Avoid unwrap() without checks** - Always use `unwrapOr()` or check `isOk()` first
5. **Log errors in mapErr** - Use `mapErr()` for side effects like logging

## See Also

- [Attempt Monad](attempt-monad.md) - Try monad for exception handling
- [Error Wrapping](error-wrapping.md) - Wrapping lower-level exceptions
- [Integration Patterns](integration-patterns.md) - Using with Laravel
