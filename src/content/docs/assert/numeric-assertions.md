---
title: Numeric Assertions
description: Validate numbers and perform comparison operations.
---

Numeric assertions validate numbers and perform comparison operations.

## Available Assertions

### lessThan()

Assert that a value is less than a limit.

```php
use Cline\Assert\Assertions\Assertion;

Assertion::lessThan($age, 18);
Assertion::lessThan($score, 100, 'Score must be less than 100');
```

### lessOrEqualThan()

Assert that a value is less than or equal to a limit.

```php
Assertion::lessOrEqualThan($percentage, 100);
Assertion::lessOrEqualThan($quantity, 10, 'Maximum quantity is 10');
```

### greaterThan()

Assert that a value is greater than a limit.

```php
Assertion::greaterThan($age, 17);
Assertion::greaterThan($price, 0, 'Price must be positive');
```

### greaterOrEqualThan()

Assert that a value is greater than or equal to a limit.

```php
Assertion::greaterOrEqualThan($age, 18);
Assertion::greaterOrEqualThan($rating, 1, 'Rating must be at least 1');
```

### between()

Assert that a value is between two limits (inclusive).

```php
Assertion::between($age, 18, 65);
Assertion::between($score, 0, 100, 'Score must be between 0 and 100');
```

### betweenExclusive()

Assert that a value is between two limits (exclusive).

```php
Assertion::betweenExclusive($temperature, 0, 100);
```

### range()

Alias for `between()`.

```php
Assertion::range($month, 1, 12);
Assertion::range($hour, 0, 23);
```

### min()

Assert minimum value.

```php
Assertion::min($quantity, 1);
Assertion::min($price, 0.01, 'Price must be at least $0.01');
```

### max()

Assert maximum value.

```php
Assertion::max($discount, 100);
Assertion::max($items, 50, 'Maximum 50 items allowed');
```

## Chaining Numeric Assertions

```php
use Cline\Assert\Assert;

Assert::that($age)
    ->integer()
    ->greaterOrEqualThan(18)
    ->lessThan(100);

Assert::that($price)
    ->float()
    ->greaterThan(0)
    ->max(9999.99);

Assert::that($percentage)
    ->numeric()
    ->between(0, 100);
```

## Common Patterns

### Age Validation

```php
Assert::that($age)
    ->integer()
    ->greaterOrEqualThan(0)
    ->lessThan(150);
```

### Price Validation

```php
Assert::that($price)
    ->float()
    ->greaterThan(0)
    ->max(1000000);
```

### Rating System (1-5)

```php
Assert::that($rating)
    ->integer()
    ->between(1, 5, 'Rating must be between 1 and 5');
```

### Percentage Validation

```php
Assert::that($percentage)
    ->numeric()
    ->greaterOrEqualThan(0)
    ->lessOrEqualThan(100);
```

### Quantity Validation

```php
Assert::that($quantity)
    ->integer()
    ->greaterThan(0, 'Quantity must be positive')
    ->max(999, 'Maximum quantity is 999');
```

## Working with Different Numeric Types

### Integer Validation

```php
Assert::that($count)
    ->integer()
    ->greaterOrEqualThan(0);
```

### Float Validation

```php
Assert::that($latitude)
    ->float()
    ->between(-90, 90);
```

### Numeric Strings

```php
Assert::that($value)
    ->numeric()
    ->greaterThan(0);
```

## Next Steps

- [Type Assertions](/assert/type-assertions/) - Type checking for integers, floats
- [Comparison Assertions](/assert/comparison-assertions/) - Equality operations
- [Lazy Assertions](/assert/lazy-assertions/) - Validate multiple fields
