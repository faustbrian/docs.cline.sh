---
title: Base Exceptions
description: Throw provides three base exception classes that categorize errors by their nature, making error handling more semantic and maintainable.
---

Throw provides three base exception classes that categorize errors by their nature, making error handling more semantic and maintainable.

## Overview

The base exceptions help you organize errors into clear categories:

- **DomainException** - Business logic violations
- **InfrastructureException** - External system failures
- **ValidationException** - Input validation errors

All base exceptions include the `ConditionallyThrowable`, `HasErrorContext`, and `WrapsErrors` traits.

## DomainException

Use `DomainException` for business rule violations and domain-specific errors.

### Creating Domain Exceptions

```php
use Cline\Throw\Exceptions\DomainException;

final class OrderCannotBeCancelledException extends DomainException
{
    public static function alreadyShipped(): self
    {
        return new self('Order cannot be cancelled after shipping');
    }

    public static function alreadyDelivered(): self
    {
        return new self('Order has already been delivered');
    }
}
```

### Usage Examples

```php
// Guard against invalid state transitions
OrderCannotBeCancelledException::alreadyShipped()
    ->throwIf($order->status === 'shipped');

// Business rule enforcement
InsufficientFundsException::forAmount($amount)
    ->withContext(['balance' => $account->balance, 'required' => $amount])
    ->throwIf($account->balance < $amount);

// Invalid operations
SubscriptionNotActiveException::cannotAccess()
    ->throwUnless($subscription->isActive());
```

### Common Domain Exception Patterns

```php
// State machine violations
final class InvalidStateTransitionException extends DomainException
{
    public static function from(string $from, string $to): self
    {
        return new self("Cannot transition from {$from} to {$to}");
    }
}

// Business constraints
final class MaximumRetriesExceededException extends DomainException
{
    public static function forOperation(string $operation, int $max): self
    {
        return new self("Maximum {$max} retries exceeded for {$operation}");
    }
}

// Invariant violations
final class AccountOverdraftException extends DomainException
{
    public static function amount(Money $overdraft): self
    {
        return new self("Account would be overdrawn by {$overdraft->format()}");
    }
}
```

## InfrastructureException

Use `InfrastructureException` for failures in external dependencies like databases, APIs, file systems, or caches.

### Creating Infrastructure Exceptions

```php
use Cline\Throw\Exceptions\InfrastructureException;

final class DatabaseException extends InfrastructureException
{
    public static function connectionFailed(string $host): self
    {
        return new self("Failed to connect to database at {$host}");
    }

    public static function queryFailed(): self
    {
        return new self('Database query failed');
    }
}
```

### Usage Examples

```php
// Database operations
try {
    $db->query($sql);
} catch (PDOException $e) {
    throw DatabaseException::queryFailed()
        ->wrap($e)
        ->withContext(['query' => $sql, 'bindings' => $bindings]);
}

// External API calls
try {
    $response = $client->post('/payments', $data);
} catch (RequestException $e) {
    throw PaymentGatewayException::requestFailed()
        ->wrap($e)
        ->withTags(['payment', 'stripe', 'critical'])
        ->withMetadata(['data' => $data, 'response' => $e->getResponse()]);
}

// File system operations
FileSystemException::cannotWrite($path)
    ->throwIf(!is_writable($path));
```

### Common Infrastructure Exception Patterns

```php
// API failures
final class ApiException extends InfrastructureException
{
    public static function timeout(string $endpoint): self
    {
        return new self("API request to {$endpoint} timed out");
    }

    public static function rateLimited(): self
    {
        return new self('API rate limit exceeded');
    }
}

// Cache failures
final class CacheException extends InfrastructureException
{
    public static function connectionFailed(string $driver): self
    {
        return new self("Failed to connect to {$driver} cache");
    }
}

// Queue failures
final class QueueException extends InfrastructureException
{
    public static function jobFailed(string $job): self
    {
        return new self("Queue job {$job} failed to process");
    }
}
```

## ValidationException

Use `ValidationException` for input validation failures and data constraint violations.

### Creating Validation Exceptions

```php
use Cline\Throw\Exceptions\ValidationException;

final class InvalidEmailException extends ValidationException
{
    public static function format(string $email): self
    {
        return new self("Invalid email format: {$email}");
    }
}

final class RequiredFieldException extends ValidationException
{
    public static function missing(string $field): self
    {
        return new self("Required field missing: {$field}");
    }
}
```

### Usage Examples

```php
// Email validation
InvalidEmailException::format($email)
    ->throwIf(!filter_var($email, FILTER_VALIDATE_EMAIL));

// Required fields
RequiredFieldException::missing('email')
    ->throwIf(empty($data['email']));

// Type validation
InvalidTypeException::expectedArray('settings')
    ->withContext(['actual_type' => get_debug_type($value)])
    ->throwIf(!is_array($value));

// Range validation
OutOfRangeException::forField('age', 0, 120)
    ->throwIf($age < 0 || $age > 120);
```

### Common Validation Exception Patterns

```php
// Format validation
final class InvalidFormatException extends ValidationException
{
    public static function forField(string $field, string $expectedFormat): self
    {
        return new self("{$field} must be in {$expectedFormat} format");
    }
}

// Length validation
final class StringTooLongException extends ValidationException
{
    public static function forField(string $field, int $max, int $actual): self
    {
        return new self("{$field} exceeds maximum length of {$max} (got {$actual})");
    }
}

// Constraint validation
final class UniqueConstraintException extends ValidationException
{
    public static function forField(string $field, mixed $value): self
    {
        return new self("{$field} value '{$value}' already exists");
    }
}
```

## Choosing the Right Base Exception

| Exception Type | Use When | Examples |
|---------------|----------|----------|
| **DomainException** | Violating business rules or domain constraints | Order already shipped, insufficient funds, subscription expired |
| **InfrastructureException** | External systems fail | Database down, API timeout, file not writable |
| **ValidationException** | Input data is invalid | Invalid email, missing required field, number out of range |

## Combined Features

All base exceptions support the full Throw feature set:

```php
// Combine all features
PaymentFailedException::insufficientFunds()
    ->withContext([
        'user_id' => $user->id,
        'amount' => $amount,
        'balance' => $account->balance,
    ])
    ->withTags(['payment', 'critical', 'stripe'])
    ->withMetadata([
        'gateway_response' => $response,
        'attempt_number' => 3,
    ])
    ->throwIf($account->balance < $amount);

// Wrap lower-level exceptions
try {
    $db->query($sql);
} catch (PDOException $e) {
    throw DatabaseException::queryFailed()
        ->wrap($e)
        ->withContext(['query' => $sql])
        ->withTags(['database', 'critical']);
}
```

## Best Practices

1. **Choose the right base** - Use the exception type that best describes the error's nature
2. **Create specific exceptions** - Extend base exceptions for each distinct error case
3. **Use static factories** - Provide named constructors for common scenarios
4. **Add context** - Include relevant data to aid debugging
5. **Tag appropriately** - Use tags for filtering in monitoring systems
6. **Wrap low-level errors** - Preserve original exceptions while providing domain context

## Next Steps

- Learn about [Error Context](error-context.md) for adding debugging information
- Explore [Error Wrapping](error-wrapping.md) for preserving exception chains
- See [Assertions](assertions.md) for the `ensure()` helper pattern
