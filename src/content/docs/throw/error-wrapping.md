---
title: Error Wrapping
description: Wrap lower-level exceptions with domain-specific exceptions while preserving the original error for debugging. This pattern maintains clean error boundaries between application layers.
---

Wrap lower-level exceptions with domain-specific exceptions while preserving the original error for debugging. This pattern maintains clean error boundaries between application layers.

## Overview

The `WrapsErrors` trait provides the `wrap()` method to:

- Catch low-level exceptions (PDOException, RequestException, etc.)
- Re-throw as domain-specific exceptions
- Preserve the original exception in the exception chain
- Maintain all context, tags, and metadata

## Basic Usage

### Wrapping Database Exceptions

```php
use Cline\Throw\Exceptions\InfrastructureException;

final class DatabaseException extends InfrastructureException
{
    public static function queryFailed(): self
    {
        return new self('Database query failed');
    }
}

try {
    $db->query($sql);
} catch (PDOException $e) {
    throw DatabaseException::queryFailed()->wrap($e);
}
```

### Wrapping API Exceptions

```php
use Cline\Throw\Exceptions\InfrastructureException;

final class PaymentGatewayException extends InfrastructureException
{
    public static function requestFailed(): self
    {
        return new self('Payment gateway request failed');
    }
}

try {
    $response = $client->post('/charge', $data);
} catch (RequestException $e) {
    throw PaymentGatewayException::requestFailed()->wrap($e);
}
```

## Why Wrap Exceptions?

### Clean Layer Boundaries

```php
// ❌ Bad - Exposes infrastructure details to application layer
public function chargeCustomer(Customer $customer, Money $amount): void
{
    try {
        $this->stripe->charges->create([/* ... */]);
    } catch (ApiErrorException $e) {
        // Application layer now depends on Stripe SDK exception
        throw $e;
    }
}

// ✅ Good - Application layer sees domain exception
public function chargeCustomer(Customer $customer, Money $amount): void
{
    try {
        $this->stripe->charges->create([/* ... */]);
    } catch (ApiErrorException $e) {
        throw PaymentFailedException::gatewayError()->wrap($e);
    }
}
```

### Preserve Original Exception

The wrapped exception is preserved in two ways:

```php
try {
    $db->query($sql);
} catch (PDOException $e) {
    $wrapped = DatabaseException::queryFailed()->wrap($e);

    $wrapped->getWrapped();   // Returns the PDOException
    $wrapped->getPrevious();  // Also returns the PDOException
}
```

## Combining with Context

Wrap exceptions and add context in one fluent chain:

```php
try {
    $db->query($sql);
} catch (PDOException $e) {
    throw DatabaseException::queryFailed()
        ->wrap($e)
        ->withContext([
            'query' => $sql,
            'bindings' => $bindings,
        ])
        ->withTags(['database', 'critical'])
        ->withMetadata([
            'connection' => config('database.default'),
            'execution_time' => $executionTime,
        ]);
}
```

## Real-World Patterns

### Database Layer

```php
namespace App\Exceptions;

use Cline\Throw\Exceptions\InfrastructureException;

final class DatabaseException extends InfrastructureException
{
    public static function queryFailed(): self
    {
        return new self('Database query failed');
    }

    public static function connectionFailed(): self
    {
        return new self('Failed to connect to database');
    }

    public static function transactionFailed(): self
    {
        return new self('Database transaction failed');
    }
}

// Usage in repository
class OrderRepository
{
    public function save(Order $order): void
    {
        try {
            DB::table('orders')->insert($order->toArray());
        } catch (QueryException $e) {
            throw DatabaseException::queryFailed()
                ->wrap($e)
                ->withContext(['order_id' => $order->id])
                ->withTags(['database', 'orders']);
        }
    }
}
```

### HTTP Client Layer

```php
namespace App\Exceptions;

use Cline\Throw\Exceptions\InfrastructureException;

final class ExternalApiException extends InfrastructureException
{
    public static function requestFailed(string $service): self
    {
        return new self("Request to {$service} failed");
    }

    public static function timeout(string $service): self
    {
        return new self("{$service} request timed out");
    }
}

// Usage in service
class PaymentGatewayService
{
    public function charge(Money $amount, string $token): PaymentIntent
    {
        try {
            $response = $this->client->post('/charge', [
                'amount' => $amount->getAmount(),
                'token' => $token,
            ]);
        } catch (ConnectException $e) {
            throw ExternalApiException::timeout('Stripe')
                ->wrap($e)
                ->withTags(['payment', 'stripe', 'timeout']);
        } catch (RequestException $e) {
            throw ExternalApiException::requestFailed('Stripe')
                ->wrap($e)
                ->withContext(['amount' => $amount->getAmount()])
                ->withMetadata([
                    'response_status' => $e->getResponse()?->getStatusCode(),
                    'response_body' => $e->getResponse()?->getBody()?->getContents(),
                ]);
        }
    }
}
```

### File System Layer

```php
namespace App\Exceptions;

use Cline\Throw\Exceptions\InfrastructureException;

final class FileSystemException extends InfrastructureException
{
    public static function cannotRead(string $path): self
    {
        return new self("Cannot read file: {$path}");
    }

    public static function cannotWrite(string $path): self
    {
        return new self("Cannot write to file: {$path}");
    }
}

// Usage
class FileStorage
{
    public function read(string $path): string
    {
        try {
            return file_get_contents($path);
        } catch (ErrorException $e) {
            throw FileSystemException::cannotRead($path)
                ->wrap($e)
                ->withContext(['path' => $path, 'permissions' => fileperms($path)]);
        }
    }
}
```

### Cache Layer

```php
namespace App\Exceptions;

use Cline\Throw\Exceptions\InfrastructureException;

final class CacheException extends InfrastructureException
{
    public static function connectionFailed(string $driver): self
    {
        return new self("Failed to connect to {$driver} cache");
    }

    public static function operationFailed(string $operation): self
    {
        return new self("Cache {$operation} operation failed");
    }
}

// Usage
class CacheService
{
    public function remember(string $key, callable $callback, int $ttl): mixed
    {
        try {
            return Cache::remember($key, $ttl, $callback);
        } catch (RedisException $e) {
            throw CacheException::operationFailed('remember')
                ->wrap($e)
                ->withContext(['key' => $key, 'ttl' => $ttl])
                ->withTags(['cache', 'redis']);
        }
    }
}
```

## Accessing Wrapped Exceptions

### Check if Exception is Wrapped

```php
if ($exception->hasWrapped()) {
    $original = $exception->getWrapped();
    // Handle original exception
}
```

### Type Checking Wrapped Exceptions

```php
try {
    // ... some operation
} catch (DatabaseException $e) {
    if ($e->getWrapped() instanceof PDOException) {
        // Handle PDO-specific errors
        $pdoError = $e->getWrapped();
        $errorCode = $pdoError->getCode();
    }
}
```

### Full Exception Chain

```php
try {
    // ... some operation
} catch (DatabaseException $e) {
    $current = $e;

    // Walk the entire exception chain
    while ($current !== null) {
        echo $current->getMessage() . PHP_EOL;
        $current = $current->getPrevious();
    }
}
```

## Exception Handler Integration

### Laravel Exception Handler

```php
namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Cline\Throw\Concerns\WrapsErrors;

class Handler extends ExceptionHandler
{
    public function report(Throwable $exception)
    {
        // Log wrapped exception details
        if (method_exists($exception, 'getWrapped') && $exception->hasWrapped()) {
            $wrapped = $exception->getWrapped();

            Log::error('Wrapped exception detected', [
                'domain_exception' => get_class($exception),
                'domain_message' => $exception->getMessage(),
                'original_exception' => get_class($wrapped),
                'original_message' => $wrapped->getMessage(),
                'context' => method_exists($exception, 'getContext')
                    ? $exception->getContext()
                    : [],
            ]);
        }

        parent::report($exception);
    }
}
```

### Sentry/Bugsnag Integration

```php
public function report(Throwable $exception)
{
    if (method_exists($exception, 'getWrapped') && $exception->hasWrapped()) {
        // Report both exceptions to Sentry
        app('sentry')->captureException($exception->getWrapped(), [
            'extra' => [
                'wrapped_by' => get_class($exception),
                'context' => $exception->getContext() ?? [],
            ],
        ]);
    }

    parent::report($exception);
}
```

## Best Practices

1. **Wrap at boundaries** - Catch low-level exceptions at layer boundaries (repository, service, etc.)
2. **Add context** - Include relevant data about the operation that failed
3. **Use domain exceptions** - Wrap with exceptions that make sense in your domain
4. **Preserve original** - Always wrap rather than replacing the original exception
5. **Tag for routing** - Use tags to route wrapped exceptions to appropriate handlers
6. **Check wrapped type** - Use type checking on wrapped exceptions for specific handling

## Testing Wrapped Exceptions

```php
use Tests\Fixtures\TestInfrastructureException;

test('wraps PDOException', function () {
    $pdo = new PDOException('Connection failed');
    $wrapped = TestInfrastructureException::databaseFailure()->wrap($pdo);

    expect($wrapped->getWrapped())
        ->toBeInstanceOf(PDOException::class)
        ->and($wrapped->getWrapped()->getMessage())
        ->toBe('Connection failed')
        ->and($wrapped->hasWrapped())
        ->toBeTrue();
});
```

## Next Steps

- Learn about [Error Context](error-context.md) for adding debugging data
- See [Base Exceptions](base-exceptions.md) for choosing the right exception type
- Explore [Assertions](assertions.md) for the `ensure()` helper pattern
