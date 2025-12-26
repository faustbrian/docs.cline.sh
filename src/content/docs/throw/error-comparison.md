---
title: Error Comparison
description: Go-inspired utilities for checking and casting exceptions through the exception chain.
---

Go-inspired utilities for checking and casting exceptions through the exception chain.

## Overview

The `Errors` utility class provides two static methods inspired by Go's error handling:

- `Errors::is()` - Check if an exception matches a specific type
- `Errors::as()` - Cast an exception to a specific type

Both methods traverse the entire exception chain (via `getPrevious()`), making them useful for working with wrapped exceptions.

## Errors::is()

Check if an exception or any exception in its chain matches the given type.

### Basic Usage

```php
use Cline\Throw\Support\Errors;

try {
    $user->charge($amount);
} catch (Throwable $e) {
    if (Errors::is($e, PaymentException::class)) {
        // Handle payment error
    }
}
```

### Checking Wrapped Exceptions

```php
use Cline\Throw\Exceptions\DatabaseException;

try {
    $db->query($sql);
} catch (Throwable $e) {
    // Will match even if PDOException is wrapped in DatabaseException
    if (Errors::is($e, PDOException::class)) {
        logger()->error('Database connection failed');
        return response()->json(['error' => 'Database error'], 500);
    }
}
```

### Multiple Exception Checks

```php
try {
    $this->externalApi->call();
} catch (Throwable $e) {
    if (Errors::is($e, TimeoutException::class)) {
        return $this->handleTimeout($e);
    }

    if (Errors::is($e, RateLimitException::class)) {
        return $this->handleRateLimit($e);
    }

    if (Errors::is($e, NetworkException::class)) {
        return $this->handleNetworkError($e);
    }

    throw $e; // Re-throw if not handled
}
```

## Errors::as()

Cast an exception to a specific type if it matches, returning the typed exception or null.

### Basic Usage

```php
use Cline\Throw\Support\Errors;

try {
    $user->charge($amount);
} catch (Throwable $e) {
    $paymentError = Errors::as($e, PaymentException::class);

    if ($paymentError !== null) {
        logger()->error('Payment failed', [
            'transaction_id' => $paymentError->getTransactionId(),
            'amount' => $paymentError->getAmount(),
        ]);
    }
}
```

### Accessing Wrapped Exception Details

```php
try {
    $db->query($sql);
} catch (Throwable $e) {
    $pdoError = Errors::as($e, PDOException::class);

    if ($pdoError !== null) {
        logger()->error('Query failed', [
            'error_code' => $pdoError->getCode(),
            'sql_state' => $pdoError->errorInfo[0] ?? null,
        ]);
    }
}
```

### Using Exception Context

```php
try {
    $payment->process();
} catch (Throwable $e) {
    $domainError = Errors::as($e, DomainException::class);

    if ($domainError !== null) {
        // Access context from HasErrorContext trait
        $context = $domainError->getContext();
        $tags = $domainError->getTags();

        sentry()->captureException($domainError, [
            'context' => $context,
            'tags' => $tags,
        ]);
    }
}
```

## Comparison with Traditional Instanceof

### Before (using instanceof)

```php
try {
    $api->call();
} catch (Throwable $e) {
    // Only checks the top-level exception
    if ($e instanceof TimeoutException) {
        // Won't match if TimeoutException is wrapped
    }

    // To check wrapped exceptions, you need manual traversal
    $current = $e;
    while ($current !== null) {
        if ($current instanceof TimeoutException) {
            // Handle timeout
            break;
        }
        $current = $current->getPrevious();
    }
}
```

### After (using Errors::is)

```php
use Cline\Throw\Support\Errors;

try {
    $api->call();
} catch (Throwable $e) {
    // Automatically checks the entire exception chain
    if (Errors::is($e, TimeoutException::class)) {
        // Matches even if wrapped
    }
}
```

## Real-World Examples

### Exception Handler Integration

```php
use Cline\Throw\Support\Errors;
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

class Handler extends ExceptionHandler
{
    public function register(): void
    {
        $this->reportable(function (Throwable $e) {
            // Handle domain exceptions differently
            if (Errors::is($e, DomainException::class)) {
                $domainError = Errors::as($e, DomainException::class);

                logger()->warning('Domain error', [
                    'message' => $domainError->getMessage(),
                    'context' => $domainError->getContext(),
                ]);

                return; // Don't report to error tracking
            }

            // Report infrastructure errors to Sentry
            if (Errors::is($e, InfrastructureException::class)) {
                app('sentry')->captureException($e);
            }
        });
    }
}
```

### Service Class Error Handling

```php
use Cline\Throw\Support\Errors;

class PaymentService
{
    public function charge(User $user, int $amount): Payment
    {
        try {
            return $this->gateway->charge($user->payment_method, $amount);
        } catch (Throwable $e) {
            // Check for specific gateway errors
            $cardError = Errors::as($e, CardException::class);
            if ($cardError !== null) {
                throw PaymentException::cardDeclined()
                    ->wrap($cardError)
                    ->withContext([
                        'decline_code' => $cardError->getDeclineCode(),
                        'user_id' => $user->id,
                    ]);
            }

            // Check for rate limiting
            if (Errors::is($e, RateLimitException::class)) {
                throw PaymentException::rateLimited()
                    ->wrap($e)
                    ->withTags(['payment', 'rate-limit']);
            }

            // Generic error
            throw PaymentException::failed()->wrap($e);
        }
    }
}
```

### API Client Error Mapping

```php
use Cline\Throw\Support\Errors;
use GuzzleHttp\Exception\RequestException;

class GitHubClient
{
    public function getUser(string $username): array
    {
        try {
            $response = $this->client->get("users/{$username}");
            return json_decode($response->getBody(), true);
        } catch (Throwable $e) {
            $requestError = Errors::as($e, RequestException::class);

            if ($requestError !== null) {
                $status = $requestError->getResponse()?->getStatusCode();

                return match ($status) {
                    404 => throw ExternalServiceException::userNotFound($username)->wrap($e),
                    429 => throw ExternalServiceException::rateLimitExceeded()->wrap($e),
                    default => throw ExternalServiceException::requestFailed()->wrap($e),
                };
            }

            throw $e;
        }
    }
}
```

### Job Failure Handling

```php
use Cline\Throw\Support\Errors;
use Illuminate\Contracts\Queue\ShouldQueue;

class ProcessWebhook implements ShouldQueue
{
    public function handle(): void
    {
        try {
            $this->processPayload();
        } catch (Throwable $e) {
            // Log different error types with appropriate severity
            if (Errors::is($e, ValidationException::class)) {
                logger()->warning('Webhook validation failed', [
                    'exception' => $e->getMessage(),
                ]);
            } elseif (Errors::is($e, InfrastructureException::class)) {
                logger()->error('Infrastructure error processing webhook', [
                    'exception' => $e->getMessage(),
                ]);
            }

            throw $e; // Re-throw to trigger job failure
        }
    }

    public function failed(Throwable $exception): void
    {
        // Extract domain-specific error details
        $domainError = Errors::as($exception, DomainException::class);

        if ($domainError !== null) {
            // Send notification with business context
            notification()->send(new WebhookFailedNotification(
                $domainError->getMessage(),
                $domainError->getContext()
            ));
        }
    }
}
```

## Testing

```php
use Cline\Throw\Support\Errors;

test('Errors::is identifies exception type in chain', function () {
    $root = new PDOException('Connection failed');
    $wrapped = new RuntimeException('Database error', 0, $root);

    expect(Errors::is($wrapped, PDOException::class))->toBeTrue();
    expect(Errors::is($wrapped, RuntimeException::class))->toBeTrue();
    expect(Errors::is($wrapped, InvalidArgumentException::class))->toBeFalse();
});

test('Errors::as casts exception from chain', function () {
    $root = new PDOException('Connection failed');
    $wrapped = new RuntimeException('Database error', 0, $root);

    $pdo = Errors::as($wrapped, PDOException::class);

    expect($pdo)->toBeInstanceOf(PDOException::class)
        ->and($pdo->getMessage())->toBe('Connection failed');
});

test('Errors::as returns null when type not found', function () {
    $exception = new RuntimeException('Error');

    $result = Errors::as($exception, PDOException::class);

    expect($result)->toBeNull();
});
```

## When to Use

### Use Errors::is() when:
- Checking if an exception type exists anywhere in the chain
- Implementing error-specific handling logic
- Working with wrapped exceptions from third-party code

### Use Errors::as() when:
- You need to access exception-specific methods or properties
- Extracting context from domain exceptions
- Type-safe exception handling with IDE autocomplete

### Use traditional instanceof when:
- You only care about the top-level exception type
- Not dealing with wrapped exceptions
- Performance is critical (Errors methods traverse the chain)
