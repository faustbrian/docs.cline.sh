---
title: Chaining Operations
description: Discover how to chain Numerus operations for fluent and readable numeric computations.
---

One of Numerus's most powerful features is the ability to chain operations together, creating fluent and readable numeric computations.

## Basic Chaining

Every operation returns a new Numerus instance, allowing method chaining:

```php
use function Cline\Numerus\numerus;

$result = numerus(10)
    ->plus(5)
    ->minus(3)
    ->multiplyBy(2)
    ->divideBy(4);

// $result->value() === 6.0
```

## Complex Calculations

### Calculate Percentage Increase

```php
function calculatePercentageIncrease(float $original, float $new): float
{
    return numerus($new)
        ->minus($original)
        ->divideBy($original)
        ->multiplyBy(100)
        ->round(2)
        ->value();
}

$increase = calculatePercentageIncrease(50, 75);
// $increase === 50.0 (50% increase)
```

### Calculate Average with Constraints

```php
function constrainedAverage(array $values, int $min, int $max): float
{
    $sum = array_reduce(
        $values,
        fn($carry, $value) => $carry->plus($value),
        numerus(0)
    );

    return $sum
        ->divideBy(count($values))
        ->clamp($min, $max)
        ->round(2)
        ->value();
}
```

### Distance Formula

```php
function distance(float $x1, float $y1, float $x2, float $y2): float
{
    $dx = numerus($x2)->minus($x1)->power(2);
    $dy = numerus($y2)->minus($y1)->power(2);

    return $dx->plus($dy->value())->sqrt()->value();
}

$dist = distance(0, 0, 3, 4);
// $dist === 5.0
```

## Conditional Chaining

### Apply Operation Based on Condition

```php
function applyDiscount(float $price, bool $isPremium): float
{
    $num = numerus($price);

    if ($isPremium) {
        return $num
            ->multiplyBy(0.8)  // 20% discount
            ->round(2)
            ->value();
    }

    return $num
        ->multiplyBy(0.95)  // 5% discount
        ->round(2)
        ->value();
}
```

### Normalize and Transform

```php
function normalizeAndScale(float $value, float $min, float $max, float $scale = 1.0): float
{
    return numerus($value)
        ->clamp($min, $max)
        ->minus($min)
        ->divideBy($max - $min)
        ->multiplyBy($scale)
        ->value();
}

$normalized = normalizeAndScale(75, 0, 100, 10);
// $normalized === 7.5
```

## Working with Collections

### Transform Array of Values

```php
$prices = [10.99, 25.50, 99.99];

$discounted = array_map(
    fn($price) => numerus($price)
        ->multiplyBy(0.9)
        ->round(2)
        ->value(),
    $prices
);

// $discounted === [9.89, 22.95, 89.99]
```

### Calculate Statistics

```php
function calculateStats(array $values): array
{
    $count = count($values);
    $sum = array_reduce(
        $values,
        fn($carry, $val) => $carry->plus($val),
        numerus(0)
    );

    $mean = $sum->divideBy($count);

    $variance = array_reduce(
        $values,
        fn($carry, $val) => $carry->plus(
            numerus($val)->minus($mean->value())->power(2)->value()
        ),
        numerus(0)
    )->divideBy($count);

    return [
        'sum' => $sum->value(),
        'mean' => $mean->round(2)->value(),
        'variance' => $variance->round(2)->value(),
        'stddev' => $variance->sqrt()->round(2)->value(),
    ];
}

$stats = calculateStats([2, 4, 4, 4, 5, 5, 7, 9]);
```

## Practical Examples

### Currency Conversion with Rounding

```php
use RoundingMode;

function convertCurrency(float $amount, float $rate): float
{
    return numerus($amount)
        ->multiplyBy($rate)
        ->round(2)
        ->value();
}

$euros = convertCurrency(100, 0.85);
// $euros === 85.0

// Using banker's rounding for financial accuracy
function convertCurrencyBankers(float $amount, float $rate): float
{
    return numerus($amount)
        ->multiplyBy($rate)
        ->round(2, RoundingMode::HalfEven)
        ->value();
}

// Or use the convenience method
function convertCurrencyBankers2(float $amount, float $rate): float
{
    return numerus($amount)
        ->multiplyBy($rate)
        ->roundHalfEven(2)
        ->value();
}
```

### Calculate Tax

```php
function calculateTotalWithTax(float $subtotal, float $taxRate): float
{
    return numerus($subtotal)
        ->multiplyBy(numerus(1)->plus($taxRate)->value())
        ->round(2)
        ->value();
}

$total = calculateTotalWithTax(100, 0.20);
// $total === 120.0
```

### Compound Interest

```php
function compoundInterest(
    float $principal,
    float $rate,
    int $years,
    int $compoundsPerYear = 1
): float {
    return numerus(1)
        ->plus($rate / $compoundsPerYear)
        ->power($compoundsPerYear * $years)
        ->multiplyBy($principal)
        ->round(2)
        ->value();
}

$amount = compoundInterest(1000, 0.05, 10, 12);
// Monthly compound interest over 10 years
```

## Immutability Benefits

The immutability of Numerus ensures that chained operations don't affect earlier values:

```php
$base = numerus(100);
$withTax = $base->multiplyBy(1.20);
$discounted = $base->multiplyBy(0.90);

// $base->value() === 100 (unchanged)
// $withTax->value() === 120.0
// $discounted->value() === 90.0
```

This allows you to derive multiple calculations from the same base value safely.
