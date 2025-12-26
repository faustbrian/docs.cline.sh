---
title: Error Context
description: Add structured debugging information to exceptions using context, tags, and metadata. This makes error tracking, logging, and monitoring significantly more effective.
---

Add structured debugging information to exceptions using context, tags, and metadata. This makes error tracking, logging, and monitoring significantly more effective.

## Overview

The `HasErrorContext` trait provides three methods for attaching debugging information:

- `withContext()` - Structured contextual data about the error
- `withTags()` - Categorical labels for filtering and grouping
- `withMetadata()` - Detailed technical debugging information

## Adding Context

Use `withContext()` to attach relevant data about the circumstances of the error.

### Basic Usage

```php
use App\Exceptions\PaymentFailedException;

PaymentFailedException::insufficientFunds()
    ->withContext([
        'user_id' => $user->id,
        'amount' => $amount,
        'balance' => $account->balance,
    ])
    ->throwIf($account->balance < $amount);
```

### Request Context

```php
UnauthorizedException::invalidToken()
    ->withContext([
        'ip_address' => request()->ip(),
        'user_agent' => request()->userAgent(),
        'session_id' => session()->getId(),
        'attempted_action' => 'access_admin_panel',
    ])
    ->throwIf(!$token->isValid());
```

### Business Context

```php
OrderCannotBeCancelledException::alreadyShipped()
    ->withContext([
        'order_id' => $order->id,
        'status' => $order->status,
        'shipped_at' => $order->shipped_at,
        'customer_id' => $order->customer_id,
    ])
    ->throwIf($order->status === 'shipped');
```

### Multiple Context Calls

Context data merges across multiple calls:

```php
$exception = DatabaseException::queryFailed()
    ->withContext(['query' => $sql])
    ->withContext(['duration_ms' => $executionTime]);

// Result: ['query' => $sql, 'duration_ms' => $executionTime]
```

Later values override earlier ones:

```php
$exception = PaymentException::failed()
    ->withContext(['status' => 'pending'])
    ->withContext(['status' => 'failed']);

// Result: ['status' => 'failed']
```

## Adding Tags

Use `withTags()` to categorize exceptions for filtering in logging and monitoring systems.

### Basic Tagging

```php
PaymentException::gatewayTimeout()
    ->withTags(['payment', 'stripe', 'critical'])
    ->throwIf($timeout);
```

### Severity Tags

```php
// Critical errors
DatabaseException::connectionLost()
    ->withTags(['critical', 'database', 'infrastructure'])
    ->throwIf($connectionLost);

// Warning-level errors
RateLimitException::approaching()
    ->withTags(['warning', 'rate-limit'])
    ->throwIf($usage > 0.9);
```

### System Tags

```php
// Tag by affected system
ApiException::timeout()
    ->withTags(['external-api', 'payment-gateway', 'stripe'])
    ->throwIf($timedOut);

// Tag by error category
ValidationException::invalidInput()
    ->withTags(['validation', 'user-input', 'form-submission'])
    ->throwIf(!$valid);
```

### Multiple Tag Calls

Tags accumulate across calls:

```php
$exception = PaymentException::failed()
    ->withTags(['payment'])
    ->withTags(['stripe', 'critical']);

// Result: ['payment', 'stripe', 'critical']
```

## Adding Metadata

Use `withMetadata()` for detailed technical debugging information that's too verbose for regular context.

### API Response Metadata

```php
try {
    $response = $client->post('/charge', $data);
} catch (RequestException $e) {
    throw PaymentException::gatewayError()
        ->wrap($e)
        ->withContext(['user_id' => $user->id, 'amount' => $amount])
        ->withTags(['payment', 'stripe'])
        ->withMetadata([
            'request_body' => $data,
            'response_status' => $e->getResponse()?->getStatusCode(),
            'response_body' => $e->getResponse()?->getBody()?->getContents(),
            'response_headers' => $e->getResponse()?->getHeaders(),
            'duration_ms' => $duration,
        ]);
}
```

### Database Query Metadata

```php
try {
    DB::statement($sql, $bindings);
} catch (QueryException $e) {
    throw DatabaseException::queryFailed()
        ->wrap($e)
        ->withContext(['table' => 'orders'])
        ->withMetadata([
            'query' => $sql,
            'bindings' => $bindings,
            'execution_time' => $time,
            'connection' => config('database.default'),
        ]);
}
```

### Multiple Metadata Calls

Metadata merges like context:

```php
$exception = ApiException::failed()
    ->withMetadata(['request' => $requestData])
    ->withMetadata(['response' => $responseData]);

// Result: ['request' => $requestData, 'response' => $responseData]
```

## Retrieving Context Data

Access attached data using getter methods:

```php
try {
    PaymentException::failed()
        ->withContext(['user_id' => 123])
        ->withTags(['payment'])
        ->withMetadata(['debug' => 'info'])
        ->throwIf(true);
} catch (PaymentException $e) {
    $context = $e->getContext();   // ['user_id' => 123]
    $tags = $e->getTags();         // ['payment']
    $metadata = $e->getMetadata(); // ['debug' => 'info']
}
```

## Using Context in Exception Handlers

### Laravel Exception Handler

```php
namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Cline\Throw\Concerns\HasErrorContext;

class Handler extends ExceptionHandler
{
    public function report(Throwable $exception)
    {
        if ($this->hasErrorContext($exception)) {
            // Log context data
            Log::error($exception->getMessage(), [
                'context' => $exception->getContext(),
                'tags' => $exception->getTags(),
                'metadata' => $exception->getMetadata(),
            ]);

            // Send to monitoring service
            if (in_array('critical', $exception->getTags())) {
                $this->reportCriticalError($exception);
            }
        }

        parent::report($exception);
    }

    private function hasErrorContext(Throwable $exception): bool
    {
        return method_exists($exception, 'getContext');
    }

    private function reportCriticalError(Throwable $exception): void
    {
        // Send to Sentry, Bugsnag, etc.
        app('sentry')->captureException($exception, [
            'extra' => [
                'context' => $exception->getContext(),
                'tags' => $exception->getTags(),
                'metadata' => $exception->getMetadata(),
            ],
        ]);
    }
}
```

### Filtering by Tags

```php
public function report(Throwable $exception)
{
    if (!method_exists($exception, 'getTags')) {
        return parent::report($exception);
    }

    $tags = $exception->getTags();

    // Route to different handlers based on tags
    if (in_array('payment', $tags)) {
        $this->notifyFinanceTeam($exception);
    }

    if (in_array('database', $tags)) {
        $this->notifyDatabaseTeam($exception);
    }

    if (in_array('critical', $tags)) {
        $this->sendUrgentAlert($exception);
    }

    parent::report($exception);
}
```

## Real-World Patterns

### Complete Error Context

```php
// Comprehensive error reporting
PaymentFailedException::gatewayTimeout()
    ->withContext([
        'user_id' => auth()->id(),
        'order_id' => $order->id,
        'amount' => $amount,
        'currency' => 'USD',
    ])
    ->withTags(['payment', 'stripe', 'critical', 'timeout'])
    ->withMetadata([
        'gateway_request' => $requestData,
        'gateway_response' => $responseData,
        'attempt_number' => 3,
        'total_duration_ms' => 15000,
    ])
    ->throwIf($timeout);
```

### Multi-Tenant Context

```php
TenantException::quotaExceeded()
    ->withContext([
        'tenant_id' => $tenant->id,
        'tenant_name' => $tenant->name,
        'current_usage' => $usage,
        'quota_limit' => $limit,
    ])
    ->withTags(['multi-tenant', 'quota', 'billing'])
    ->throwIf($usage >= $limit);
```

### Feature Flag Context

```php
FeatureNotAvailableException::disabled()
    ->withContext([
        'feature' => 'advanced_analytics',
        'user_id' => $user->id,
        'user_plan' => $user->plan,
    ])
    ->withTags(['feature-flag', 'access-control'])
    ->throwUnless(Feature::enabled('advanced_analytics'));
```

## Best Practices

1. **Context for what happened** - User IDs, resource IDs, relevant values
2. **Tags for categorization** - System, severity, error type
3. **Metadata for debugging** - Full request/response data, detailed technical info
4. **Sanitize sensitive data** - Don't include passwords, tokens, or PII
5. **Be consistent** - Use similar context keys across your application
6. **Tag strategically** - Use tags that help you filter in monitoring tools

## Next Steps

- Learn about [Error Wrapping](error-wrapping.md) for exception chains
- See [Base Exceptions](base-exceptions.md) for categorizing errors
- Explore [Assertions](assertions.md) for the `ensure()` helper
