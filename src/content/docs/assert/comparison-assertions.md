---
title: Comparison Assertions
description: Validate equality and identity between values.
---

Comparison assertions validate equality and identity between values.

## Available Assertions

### eq()

Assert equality using loose comparison (==).

```php
use Cline\Assert\Assertions\Assertion;

Assertion::eq($actual, $expected);
Assertion::eq(123, '123');  // Passes (loose comparison)
Assertion::eq($result, 42, 'Result must equal 42');
```

### same()

Assert identity using strict comparison (===).

```php
Assertion::same($actual, $expected);
Assertion::same(123, 123);   // Passes
Assertion::same(123, '123'); // Fails (different types)
```

### notEq()

Assert values are NOT equal (loose comparison).

```php
Assertion::notEq($actual, $unwanted);
Assertion::notEq($status, 'deleted', 'Status cannot be deleted');
```

### notSame()

Assert values are NOT identical (strict comparison).

```php
Assertion::notSame($actual, $unwanted);
Assertion::notSame($newPassword, $oldPassword, 'New password must be different');
```

## Loose vs Strict Comparison

### Loose Comparison (eq/notEq)

Uses `==` operator with type coercion:

```php
Assertion::eq(123, '123');     // Pass
Assertion::eq(1, true);        // Pass
Assertion::eq(0, false);       // Pass
```

### Strict Comparison (same/notSame)

Uses `===` operator without type coercion:

```php
Assertion::same(123, '123');   // Fail
Assertion::same(1, true);      // Fail
Assertion::same(0, false);     // Fail
```

## Chaining Comparison Assertions

```php
use Cline\Assert\Assert;

Assert::that($result)
    ->integer()
    ->same(42);

Assert::that($status)
    ->string()
    ->notEq('deleted')
    ->notEq('archived');
```

## Common Patterns

### Expected Value Validation

```php
Assert::that($response->status)
    ->integer()
    ->same(200, 'Expected HTTP 200 status');
```

### State Validation

```php
Assert::that($order->status)
    ->string()
    ->notEq('cancelled', 'Order is cancelled')
    ->notEq('refunded', 'Order is refunded');
```

### Preventing Duplicate Values

```php
Assert::that($newEmail)
    ->email()
    ->notSame($currentEmail, 'New email must be different');
```

## When to Use Which

### Use eq() when:
- Comparing form input (strings) with expected values
- Type flexibility is acceptable

### Use same() when:
- Type safety is important
- Comparing computed values
- Validating configuration values

### Use notSame() when:
- Ensuring different passwords or tokens
- Type-safe blacklisting

## Best Practices

### Prefer Strict Comparison

```php
// Loose comparison can hide bugs
Assertion::eq($count, '0');

// Strict comparison catches type issues
Assertion::same($count, 0);
```

### Combine with Type Checks

```php
Assert::that($value)
    ->integer()
    ->same(42);
```

## Next Steps

- [Numeric Assertions](/assert/numeric-assertions/) - Range comparisons
- [Type Assertions](/assert/type-assertions/) - Type validation
- [Array Assertions](/assert/array-assertions/) - Array comparisons
