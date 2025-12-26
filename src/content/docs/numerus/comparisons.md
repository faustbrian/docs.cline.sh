---
title: Comparisons
description: Explore Numerus comparison operations for equality, relational checks, min/max operations, and type checking.
---

Numerus provides rich comparison operations for numeric values.

## Equality

```php
use function Cline\Numerus\numerus;

$num = numerus(42);

$num->equals(42);        // true
$num->equals(43);        // false
$num->equals(numerus(42)); // true

$num->notEquals(43);     // true
$num->notEquals(42);     // false
```

## Negated Assertions

Use `not()` for inverted comparison logic:

```php
$num = numerus(10);

// Negated equality
$num->not()->equals(5);              // true (10 != 5)
$num->not()->equals(10);             // false (10 == 10)

// Negated comparisons
$num->not()->greaterThan(20);        // true (10 <= 20)
$num->not()->lessThan(5);            // true (10 >= 5)

// Negated range checks
$num->not()->between(5, 15);         // false (10 is within range)
$num->not()->between(20, 30);        // true (10 is outside range)
```

## Relational Comparisons

### Greater Than

```php
$num = numerus(42);

$num->greaterThan(41);  // true
$num->greaterThan(42);  // false
$num->greaterThan(43);  // false
```

### Greater Than or Equal

```php
$num = numerus(42);

$num->greaterThanOrEqual(41);  // true
$num->greaterThanOrEqual(42);  // true
$num->greaterThanOrEqual(43);  // false
```

### Less Than

```php
$num = numerus(42);

$num->lessThan(43);  // true
$num->lessThan(42);  // false
$num->lessThan(41);  // false
```

### Less Than or Equal

```php
$num = numerus(42);

$num->lessThanOrEqual(43);  // true
$num->lessThanOrEqual(42);  // true
$num->lessThanOrEqual(41);  // false
```

## Min/Max Operations

### Minimum

```php
$num = numerus(42);
$result = $num->min(30);
// $result->value() === 30

$result = $num->min(50);
// $result->value() === 42
```

### Maximum

```php
$num = numerus(42);
$result = $num->max(50);
// $result->value() === 50

$result = $num->max(30);
// $result->value() === 42
```

### Clamp

Clamp constrains a value between a minimum and maximum:

```php
$num = numerus(100);
$result = $num->clamp(0, 50);
// $result->value() === 50 (clamped to max)

$num = numerus(-10);
$result = $num->clamp(0, 50);
// $result->value() === 0 (clamped to min)

$num = numerus(25);
$result = $num->clamp(0, 50);
// $result->value() === 25 (within range)

// Using Numerus instances
$result = $num->clamp(numerus(0), numerus(50));
```

## Type Checking

### Positive/Negative/Zero

```php
numerus(42)->isPositive();   // true
numerus(-42)->isPositive();  // false
numerus(0)->isPositive();    // false

numerus(-42)->isNegative();  // true
numerus(42)->isNegative();   // false
numerus(0)->isNegative();    // false

numerus(0)->isZero();      // true
numerus(0.0)->isZero();    // true
numerus(42)->isZero();     // false
```

### Even/Odd

Note: These checks only work with integer values.

```php
numerus(42)->isEven();  // true
numerus(43)->isEven();  // false
numerus(0)->isEven();   // true

numerus(43)->isOdd();   // true
numerus(42)->isOdd();   // false
numerus(0)->isOdd();    // false
```

## Practical Examples

### Validate Range

```php
function validateScore(int $score): bool
{
    return numerus($score)
        ->greaterThanOrEqual(0)
        && numerus($score)->lessThanOrEqual(100);
}

// Using negated assertions
function isInvalidScore(int $score): bool
{
    return numerus($score)->not()->between(0, 100);
}
```

### Ensure Non-Negative

```php
function ensurePositive(int|float $value): Numerus
{
    $num = numerus($value);

    return $num->isNegative() ? $num->abs() : $num;
}
```

### Constrain Value

```php
function normalizePercentage(float $value): float
{
    return numerus($value)
        ->clamp(0, 100)
        ->value();
}
```
