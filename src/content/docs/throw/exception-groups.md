---
title: Exception Groups
description: Handle multiple exceptions as a single unit, inspired by Python 3.11's exception groups. Perfect for validation scenarios where multiple errors can occur simultaneously.
---

Handle multiple exceptions as a single unit, inspired by Python 3.11's exception groups. Perfect for validation scenarios where multiple errors can occur simultaneously.

## Overview

```php
use function Cline\Throwaise;

// Collect multiple errors
$errors = [];
if (!$email) $errors[] = new RequiredFieldException('Email required');
if (!$password) $errors[] = new RequiredFieldException('Password required');

// Throw all at once
raise($errors, 'Validation failed');
```

## Basic Usage

### Creating Exception Groups

```php
use Cline\Throw\Exceptions\ExceptionGroup;

// Direct instantiation
throw new ExceptionGroup('Validation failed', [
    new InvalidEmailException('Invalid email format'),
    new WeakPasswordException('Password too weak'),
    new RequiredFieldException('Name is required'),
]);

// Using static factory
throw ExceptionGroup::from([
    new ValidationException('Error 1'),
    new ValidationException('Error 2'),
], 'Multiple validation errors');
```

### Using the raise() Helper

```php
use function Cline\Throwaise;

$errors = [];

// Collect errors
if (!validateEmail($email)) {
    $errors[] = new InvalidEmailException();
}

if (!validatePassword($password)) {
    $errors[] = new WeakPasswordException();
}

// Raise only if errors exist
raise($errors, 'Validation failed');
```

## Real-World Examples

### Form Validation

```php
class UserRegistrationValidator
{
    public function validate(array $data): void
    {
        $errors = [];

        if (empty($data['email'])) {
            $errors[] = new RequiredFieldException('Email is required');
        } elseif (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
            $errors[] = new InvalidEmailException('Invalid email format');
        }

        if (empty($data['password'])) {
            $errors[] = new RequiredFieldException('Password is required');
        } elseif (strlen($data['password']) < 8) {
            $errors[] = new WeakPasswordException('Password must be at least 8 characters');
        }

        if (empty($data['name'])) {
            $errors[] = new RequiredFieldException('Name is required');
        }

        if (!empty($data['age']) && $data['age'] < 18) {
            $errors[] = new ValidationException('Must be 18 or older');
        }

        raise($errors, 'User registration validation failed');
    }
}

// Usage in controller
try {
    $validator->validate($request->all());
    $user = User::create($request->validated());
} catch (ExceptionGroup $eg) {
    return response()->json([
        'message' => $eg->getMessage(),
        'errors' => collect($eg->getExceptions())->map(fn($e) => $e->getMessage()),
    ], 422);
}
```

### API Request Validation

```php
class ApiRequestValidator
{
    public function validateBulkCreate(array $items): void
    {
        $errors = [];

        foreach ($items as $index => $item) {
            if (empty($item['name'])) {
                $errors[] = new ValidationException("Item {$index}: name required");
            }

            if (!isset($item['price']) || $item['price'] <= 0) {
                $errors[] = new ValidationException("Item {$index}: invalid price");
            }

            if (empty($item['category'])) {
                $errors[] = new ValidationException("Item {$index}: category required");
            }
        }

        raise($errors, 'Bulk validation failed');
    }
}
```

### Database Constraints

```php
class OrderValidator
{
    public function validateBeforeCheckout(Order $order): void
    {
        $errors = [];

        if ($order->items->isEmpty()) {
            $errors[] = new ValidationException('Order must contain at least one item');
        }

        foreach ($order->items as $item) {
            if ($item->quantity > $item->product->stock) {
                $errors[] = new OutOfStockException(
                    "Product {$item->product->name} is out of stock"
                );
            }

            if (!$item->product->isAvailable()) {
                $errors[] = new ProductUnavailableException(
                    "Product {$item->product->name} is no longer available"
                );
            }
        }

        if (!$order->hasValidShippingAddress()) {
            $errors[] = new ValidationException('Invalid shipping address');
        }

        if (!$order->hasValidPaymentMethod()) {
            $errors[] = new ValidationException('Invalid payment method');
        }

        raise($errors, 'Order validation failed');
    }
}
```

## Handling Exception Groups

### Catch and Process All Errors

```php
try {
    $validator->validate($data);
} catch (ExceptionGroup $eg) {
    foreach ($eg->getExceptions() as $exception) {
        logger()->error($exception->getMessage());
    }

    return response()->json([
        'message' => 'Validation failed',
        'errors' => array_map(fn($e) => $e->getMessage(), $eg->getExceptions()),
    ], 422);
}
```

### Filter by Exception Type

```php
try {
    $processor->processItems($items);
} catch (ExceptionGroup $eg) {
    // Handle only validation errors
    $validationErrors = $eg->filter(ValidationException::class);

    foreach ($validationErrors as $error) {
        Log::warning("Validation error: {$error->getMessage()}");
    }

    // Handle system errors differently
    $systemErrors = $eg->filter(SystemException::class);

    foreach ($systemErrors as $error) {
        Log::critical("System error: {$error->getMessage()}");
        alert_team($error);
    }
}
```

### Check for Specific Error Types

```php
try {
    $service->performOperation();
} catch (ExceptionGroup $eg) {
    if ($eg->has(CriticalException::class)) {
        // Alert team about critical errors
        alert_team($eg);
    }

    if ($eg->has(ValidationException::class)) {
        // Return user-friendly validation errors
        return redirect()->back()->withErrors(
            $eg->filter(ValidationException::class)
        );
    }
}
```

### Format for Logging

```php
try {
    $batch->process();
} catch (ExceptionGroup $eg) {
    // Log formatted exception group
    logger()->error($eg->format());

    // Or custom formatting
    logger()->error('Batch processing failed', [
        'total_errors' => $eg->count(),
        'error_types' => collect($eg->getExceptions())
            ->map(fn($e) => get_class($e))
            ->unique()
            ->values()
            ->all(),
        'messages' => collect($eg->getExceptions())
            ->map(fn($e) => $e->getMessage())
            ->all(),
    ]);
}
```

## Advanced Patterns

### Nested Validation with Context

```php
class ProfileUpdateValidator
{
    public function validate(User $user, array $data): void
    {
        $errors = [];

        // Personal info validation
        $personalErrors = $this->validatePersonalInfo($data);
        foreach ($personalErrors as $error) {
            $errors[] = $error->withContext(['section' => 'personal']);
        }

        // Contact info validation
        $contactErrors = $this->validateContactInfo($data);
        foreach ($contactErrors as $error) {
            $errors[] = $error->withContext(['section' => 'contact']);
        }

        // Preferences validation
        $prefErrors = $this->validatePreferences($data);
        foreach ($prefErrors as $error) {
            $errors[] = $error->withContext(['section' => 'preferences']);
        }

        if (!empty($errors)) {
            $group = ExceptionGroup::from($errors, 'Profile validation failed');
            $group->withContext(['user_id' => $user->id]);
            $group->withTags(['validation', 'profile-update']);

            throw $group;
        }
    }
}
```

### Aggregating Async Operation Errors

```php
class BulkEmailSender
{
    public function sendToMultipleRecipients(array $recipients): void
    {
        $errors = [];

        foreach ($recipients as $recipient) {
            try {
                $this->sendEmail($recipient);
            } catch (Exception $e) {
                $errors[] = new EmailSendException(
                    "Failed to send email to {$recipient}: {$e->getMessage()}"
                );
            }
        }

        raise($errors, 'Bulk email sending encountered errors');
    }
}
```

### Conditional Error Handling

```php
try {
    $importer->import($file);
} catch (ExceptionGroup $eg) {
    // If only warnings, log and continue
    if (!$eg->has(CriticalException::class)) {
        foreach ($eg->getExceptions() as $warning) {
            Log::warning($warning->getMessage());
        }
        return;
    }

    // If critical errors exist, abort
    throw $eg;
}
```

### API Response Formatting

```php
class ExceptionGroupTransformer
{
    public function toApiResponse(ExceptionGroup $group): array
    {
        return [
            'error' => $group->getMessage(),
            'code' => 'VALIDATION_ERROR',
            'details' => collect($group->getExceptions())->map(function ($exception) {
                return [
                    'type' => class_basename($exception),
                    'message' => $exception->getMessage(),
                    'context' => method_exists($exception, 'getContext')
                        ? $exception->getContext()
                        : null,
                ];
            })->all(),
            'count' => $group->count(),
        ];
    }
}

// Usage
try {
    $validator->validate($data);
} catch (ExceptionGroup $eg) {
    return response()->json(
        $transformer->toApiResponse($eg),
        422
    );
}
```

## Best Practices

1. **Collect before raising** - Gather all errors first, then raise once
2. **Add context** - Use `withContext()` to add debugging information
3. **Filter intelligently** - Use `filter()` to handle different error types appropriately
4. **Don't raise empty groups** - The `raise()` helper automatically skips empty arrays
5. **Log formatted output** - Use `format()` for readable log entries
6. **Type-specific handling** - Use `has()` and `filter()` for targeted error handling

## Common Pitfalls

❌ **Don't raise inside loops**
```php
// Bad
foreach ($items as $item) {
    $errors = [];
    if (!valid($item)) $errors[] = new Exception();
    raise($errors); // Throws on first error
}

// Good
$errors = [];
foreach ($items as $item) {
    if (!valid($item)) $errors[] = new Exception();
}
raise($errors); // Throws once with all errors
```

❌ **Don't check isEmpty() before raise**
```php
// Bad
if (!empty($errors)) {
    raise($errors);
}

// Good
raise($errors); // Automatically handles empty arrays
```

✅ **Do add context for debugging**
```php
$group = ExceptionGroup::from($errors, 'Validation failed');
$group->withContext(['user_id' => $user->id, 'ip' => request()->ip()]);
$group->withTags(['validation', 'registration']);

throw $group;
```

## See Also

- [Assertions](assertions.md) - Single exception throwing
- [Error Context](error-context.md) - Adding context to exceptions
- [Basic Usage](basic-usage.md) - Conditional exception throwing
