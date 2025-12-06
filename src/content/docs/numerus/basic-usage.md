---
title: Basic Usage
description: Learn how to create and use Numerus instances for immutable numeric operations with chainable arithmetic.
---

Numerus is an immutable numeric value object that provides chainable arithmetic operations.

## Creating Numerus Instances

There are two ways to create Numerus instances:

### Using the Static Factory

```php
use Cline\Numerus\Numerus;

$num = Numerus::create(42);
$decimal = Numerus::create(42.5);
$negative = Numerus::create(-10);
```

### Using the Helper Function (Recommended)

The `numerus()` helper function provides a more concise syntax:

```php
use function Cline\Numerus\numerus;

$num = numerus(42);
$decimal = numerus(42.5);
$negative = numerus(-10);
```

Both methods create identical instances. The helper function is recommended for cleaner, more readable code.

## Arithmetic Operations

All arithmetic operations return a new Numerus instance, preserving immutability.

### Addition

```php
use function Cline\Numerus\numerus;

$num = numerus(10);
$result = $num->plus(5);
// $result->value() === 15
// $num->value() === 10 (original unchanged)

// Can also add another Numerus instance
$num1 = numerus(10);
$num2 = numerus(5);
$result = $num1->plus($num2);
// $result->value() === 15
```

### Subtraction

```php
$num = numerus(10);
$result = $num->minus(3);
// $result->value() === 7
```

### Multiplication

```php
$num = numerus(10);
$result = $num->multiplyBy(3);
// $result->value() === 30
```

### Division

```php
$num = numerus(10);
$result = $num->divideBy(2);
// $result->value() === 5.0

// Division by zero throws InvalidArgumentException
try {
    $num->divideBy(0);
} catch (InvalidArgumentException $e) {
    // Handle division by zero
}
```

### Modulo

```php
$num = numerus(10);
$result = $num->mod(3);
// $result->value() === 1
```

### Power

```php
$num = numerus(2);
$result = $num->power(3);
// $result->value() === 8
```

### Square Root

```php
$num = numerus(16);
$result = $num->sqrt();
// $result->value() === 4.0

// Square root of negative throws InvalidArgumentException
```

## Rounding Operations

### Absolute Value

```php
$num = numerus(-42);
$result = $num->abs();
// $result->value() === 42
```

### Ceiling

```php
$num = numerus(42.3);
$result = $num->ceil();
// $result->value() === 43
```

### Floor

```php
$num = numerus(42.9);
$result = $num->floor();
// $result->value() === 42
```

### Round

```php
use RoundingMode;

$num = numerus(42.6);
$result = $num->round();
// $result->value() === 43.0

// With precision
$num = numerus(42.456);
$result = $num->round(2);
// $result->value() === 42.46

// With RoundingMode enum
$num = numerus(2.5);
$result = $num->round(0, RoundingMode::HalfEven); // Banker's rounding
// $result->value() === 2.0

// Convenience methods for specific rounding modes
$num = numerus(2.5);
$num->roundAwayFromZero();       // 3
$num->roundTowardsZero();        // 2
$num->roundPositiveInfinity();   // 3
$num->roundNegativeInfinity();   // 2
$num->roundHalfAwayFromZero();   // 3 (default behavior)
$num->roundHalfTowardsZero();    // 2
$num->roundHalfEven();           // 2 (banker's rounding)
$num->roundHalfOdd();            // 3
```

### Negate

```php
$num = numerus(42);
$result = $num->negate();
// $result->value() === -42
```

## Retrieving the Value

```php
$num = numerus(42);

// Get raw value
$value = $num->value(); // 42

// Convert to specific types
$int = $num->toInt();     // 42
$float = $num->toFloat(); // 42.0
$string = $num->toString(); // "42"
$string = (string) $num;    // "42"
```

## JSON Serialization

Numerus instances implement `JsonSerializable` and automatically convert to their numeric values when JSON encoded:

```php
$num = numerus(42);
echo json_encode($num); // 42

$decimal = numerus(42.5);
echo json_encode($decimal); // 42.5

// Works seamlessly in arrays and objects
$data = [
    'price' => numerus(99.99),
    'quantity' => numerus(5),
    'total' => numerus(99.99)->multiplyBy(5)
];
echo json_encode($data);
// {"price":99.99,"quantity":5,"total":499.95}

// In API responses
return response()->json([
    'amount' => numerus(100)->addPercent(20),
    'tax' => numerus(20)
]);
// {"amount":120,"tax":20}
```

## Extending with Macros

Both `Numerus` and `Percentage` support Laravel's Macroable trait, allowing you to add custom methods:

### Numerus Macros

```php
use Cline\Numerus\Numerus;
use function Cline\Numerus\numerus;

// Add instance macros
Numerus::macro('squared', fn () => $this->multiplyBy($this));
Numerus::macro('cubed', fn () => $this->power(3));
Numerus::macro('doubled', fn () => $this->multiplyBy(2));

$result = numerus(5)->squared(); // 25
$result = numerus(3)->cubed();   // 27
$result = numerus(10)->doubled(); // 20

// Add static macros
Numerus::macro('fromHundred', fn (int|float $value) =>
    Numerus::create(100)->minus($value)
);

$result = Numerus::fromHundred(30); // 70

// Macros with parameters
Numerus::macro('multiplyAndAdd', fn (int|float $multiplier, int|float $addend) =>
    $this->multiplyBy($multiplier)->plus($addend)
);

$result = numerus(10)->multiplyAndAdd(3, 5); // 35

// Chain macros with native methods
Numerus::macro('percentage', fn (int|float $percent) =>
    $this->multiplyBy($percent)->divideBy(100)
);

$result = numerus(200)->percentage(15)->round(2); // 30.00
```

### Percentage Macros

```php
use Cline\Numerus\Percentage;

// Add convenience methods
Percentage::macro('isHigh', fn (float $value, float $threshold = 50) =>
    $value >= $threshold
);

Percentage::isHigh(75); // true
Percentage::isHigh(30); // false
Percentage::isHigh(45, 40); // true

// Add domain-specific calculations
Percentage::macro('taxAmount', fn (float $taxRate, int|float $price) =>
    Percentage::calculate($taxRate, $price)
);

Percentage::macro('discountPrice', fn (float $discount, float $price) =>
    Percentage::subtract($discount, $price)
);

$tax = Percentage::taxAmount(10, 100);      // 10.0
$final = Percentage::discountPrice(20, 100); // 80.0

// Check if macro exists
if (Numerus::hasMacro('squared')) {
    // Macro is available
}

// Remove all macros (useful for testing)
Numerus::flushMacros();
Percentage::flushMacros();
```

### Practical Macro Examples

```php
// Financial calculations
Numerus::macro('withTax', fn (float $taxRate) =>
    $this->addPercent($taxRate)
);

Numerus::macro('afterDiscount', fn (float $discountRate) =>
    $this->subtractPercent($discountRate)
);

$price = numerus(100)
    ->afterDiscount(10)  // 90
    ->withTax(20);       // 108

// Unit conversions
Numerus::macro('toKilobytes', fn () => $this->divideBy(1024));
Numerus::macro('toMegabytes', fn () => $this->divideBy(1024 * 1024));

$bytes = numerus(1_048_576);
$mb = $bytes->toMegabytes(); // 1.0

// Mathematical helpers
Numerus::macro('isEvenNumber', fn () => $this->isInteger() && $this->isEven());
Numerus::macro('isPrime', function () {
    if ($this->lessThanOrEqual(1)) return false;
    if ($this->equals(2)) return true;
    if ($this->isEven()) return false;

    $sqrt = $this->sqrt()->toInt();
    for ($i = 3; $i <= $sqrt; $i += 2) {
        if ($this->mod($i)->isZero()) return false;
    }
    return true;
});

numerus(17)->isPrime(); // true
numerus(20)->isPrime(); // false
```
