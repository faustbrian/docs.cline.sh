---
title: Custom Assertions
description: Create custom validation rules using callbacks and custom assertion classes.
---

Create custom validation rules using the `satisfy()` assertion and custom assertion classes.

## The satisfy() Assertion

The `satisfy()` assertion allows you to define custom validation logic using callbacks.

### Basic Usage

```php
use Cline\Assert\Assertions\Assertion;

Assertion::satisfy($value, function($v) {
    return $v % 2 === 0;
}, 'Value must be even');
```

### With Type Checking

```php
use Cline\Assert\Assert;

Assert::that($age)
    ->integer()
    ->satisfy(fn($v) => $v >= 18 && $v <= 65, 'Age must be between 18 and 65');
```

## Common Custom Validation Patterns

### Custom String Validation

```php
Assert::that($username)
    ->string()
    ->satisfy(function($v) {
        return preg_match('/^[a-z0-9_]{3,20}$/', $v) === 1;
    }, 'Username: 3-20 chars, lowercase letters, numbers, underscores only');
```

### Password Strength

```php
Assertion::satisfy($password, function($pass) {
    return strlen($pass) >= 8
        && preg_match('/[A-Z]/', $pass)
        && preg_match('/[a-z]/', $pass)
        && preg_match('/[0-9]/', $pass)
        && preg_match('/[^A-Za-z0-9]/', $pass);
}, 'Password must contain uppercase, lowercase, number, and special character');
```

### Custom Date Range

```php
Assertion::satisfy($date, function($d) {
    $timestamp = strtotime($d);
    $now = time();
    $oneYearAgo = strtotime('-1 year');
    return $timestamp >= $oneYearAgo && $timestamp <= $now;
}, 'Date must be within the last year');
```

### Business Rule Validation

```php
Assertion::satisfy($order, function($order) {
    $calculatedTotal = array_sum(array_column($order['items'], 'price'));
    return abs($order['total'] - $calculatedTotal) < 0.01;
}, 'Order total must match sum of items');
```

## Creating Reusable Custom Validators

### Standalone Functions

```php
function assertEvenNumber($value, ?string $message = null): void
{
    Assertion::satisfy($value, fn($v) => $v % 2 === 0, $message ?? 'Value must be even');
}

assertEvenNumber($quantity);
```

### Helper Class

```php
class CustomAssertions
{
    public static function strongPassword($value, ?string $message = null): void
    {
        Assertion::satisfy($value, function($pass) {
            return strlen($pass) >= 8
                && preg_match('/[A-Z]/', $pass)
                && preg_match('/[a-z]/', $pass)
                && preg_match('/[0-9]/', $pass);
        }, $message ?? 'Password does not meet strength requirements');
    }

    public static function slug($value, ?string $message = null): void
    {
        Assertion::satisfy($value, function($v) {
            return preg_match('/^[a-z0-9-]+$/', $v) === 1;
        }, $message ?? 'Invalid slug format');
    }

    public static function hexColor($value, ?string $message = null): void
    {
        Assertion::satisfy($value, function($v) {
            return preg_match('/^#[0-9A-Fa-f]{6}$/', $v) === 1;
        }, $message ?? 'Invalid hex color format');
    }
}

CustomAssertions::strongPassword($password);
CustomAssertions::slug($urlSlug);
CustomAssertions::hexColor($brandColor);
```

## Complex Custom Validations

### Credit Card Validation

```php
function validateCreditCard(string $number): bool
{
    $number = preg_replace('/[\s-]/', '', $number);

    // Luhn algorithm
    $sum = 0;
    $numDigits = strlen($number);
    $parity = $numDigits % 2;

    for ($i = 0; $i < $numDigits; $i++) {
        $digit = (int) $number[$i];
        if ($i % 2 == $parity) {
            $digit *= 2;
        }
        if ($digit > 9) {
            $digit -= 9;
        }
        $sum += $digit;
    }

    return $sum % 10 === 0;
}

Assertion::satisfy($cardNumber, 'validateCreditCard', 'Invalid credit card number');
```

### ISBN-13 Validation

```php
Assertion::satisfy($isbn, function($v) {
    $isbn = preg_replace('/[^0-9]/', '', $v);

    if (strlen($isbn) !== 13) {
        return false;
    }

    $sum = 0;
    for ($i = 0; $i < 12; $i++) {
        $sum += (int) $isbn[$i] * ($i % 2 === 0 ? 1 : 3);
    }

    $checkDigit = (10 - ($sum % 10)) % 10;
    return $checkDigit === (int) $isbn[12];
}, 'Invalid ISBN-13');
```

## Combining Custom Assertions

### Chainable Custom Rules

```php
class UserValidator
{
    public static function assertValid(array $user): void
    {
        Assert::lazy()
            ->that($user['username'], 'username')
                ->notEmpty()
                ->satisfy(fn($v) => preg_match('/^[a-z0-9_]+$/', $v), 'Invalid username format')
            ->that($user['email'], 'email')
                ->notEmpty()
                ->email()
            ->that($user['age'], 'age')
                ->integer()
                ->satisfy(fn($v) => $v >= 13, 'Must be at least 13 years old')
            ->verifyNow();
    }
}
```

## Best Practices

### Keep Callbacks Simple

```php
// Extract complex logic to function
function validateComplexData($data): bool {
    // Complex validation logic...
}

Assertion::satisfy($data, 'validateComplexData', 'Invalid data');
```

### Provide Clear Messages

```php
Assertion::satisfy($value, $callback, 'Value must be between 1 and 100 and divisible by 5');
```

### Type Check First

```php
Assert::that($quantity)
    ->integer()
    ->satisfy(fn($v) => $v % 12 === 0, 'Quantity must be multiple of 12');
```

### Reuse Common Patterns

```php
function assertUsername($value) {
    Assertion::satisfy($value, fn($v) => preg_match('/^[a-z0-9_]+$/', $v), 'Invalid username');
}
```

## Next Steps

- [Lazy Assertions](/assert/lazy-assertions/) - Combine multiple validations
- [Assertion Chains](/assert/assertion-chains/) - Chain custom rules
- [Getting Started](/assert/getting-started/) - Core concepts
