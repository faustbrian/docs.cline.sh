---
title: Boolean Assertions
description: Validate boolean values and truthiness.
---

Boolean assertions validate boolean values and truthiness.

## Available Assertions

### true()

Assert that a value is boolean true.

```php
use Cline\Assert\Assertions\Assertion;

Assertion::true($value);
Assertion::true($isActive, 'Must be active');
```

### false()

Assert that a value is boolean false.

```php
Assertion::false($value);
Assertion::false($isDeleted, 'Must not be deleted');
```

### boolean()

Assert that a value is a boolean (true or false).

```php
Assertion::boolean($value);
Assertion::boolean($flag, 'Value must be boolean');
```

## Strict Type Checking

These assertions check for **actual boolean values only**, not truthy/falsy values:

```php
// Pass
Assertion::true(true);
Assertion::false(false);
Assertion::boolean(true);

// Fail - These are NOT booleans
Assertion::true(1);           // integer, not boolean
Assertion::false(0);          // integer, not boolean
Assertion::false(null);       // null, not boolean
```

## Chaining Boolean Assertions

```php
use Cline\Assert\Assert;

Assert::that($isActive)
    ->boolean()
    ->true('User must be active');

Assert::that($isDeleted)
    ->boolean()
    ->false('Record must not be deleted');
```

## Common Patterns

### Feature Flag Validation

```php
Assert::that($config['feature_enabled'])
    ->boolean()
    ->true('Feature must be enabled');
```

### State Validation

```php
Assert::that($user->is_active)
    ->boolean('is_active must be boolean')
    ->true('User account must be active');
```

### Configuration Validation

```php
Assert::lazy()
    ->that($config['debug'], 'debug')->boolean()
    ->that($config['cache_enabled'], 'cache_enabled')->boolean()
    ->verifyNow();
```

### Access Control

```php
Assert::that($user->hasPermission('admin'))
    ->boolean('Permission check must return boolean')
    ->true('User must have admin permission');
```

## Working with Truthy/Falsy Values

If you need to accept truthy/falsy values, convert them first:

```php
$isActive = (bool) $value;
Assertion::boolean($isActive);
```

## Database Boolean Fields

Many databases store booleans as integers:

```php
// Database returns 1/0
$row['is_active'] = 1;

// Convert first
$isActive = (bool) $row['is_active'];
Assertion::boolean($isActive);
```

## Best Practices

### Type Safety First

```php
function setActive(bool $value) {
    Assertion::boolean($value);
    $this->isActive = $value;
}
```

### Clear Error Messages

```php
Assertion::true($isVerified, 'Email address must be verified before proceeding');
```

## Next Steps

- [Type Assertions](/assert/type-assertions/) - Type checking
- [Comparison Assertions](/assert/comparison-assertions/) - Comparing values
- [Custom Assertions](/assert/custom-assertions/) - Custom validators
