---
title: Validation
description: Input validation rules and error handling for volume calculations.
---

The library enforces strict validation to ensure all volume calculations are based on valid dimensional data.

## Required Dimensions

All three dimensions (length, width, height) are required for volume calculations:

```php
use function Cline\Volume\volume;

// Valid - all three dimensions provided
volume([120, 80, 100]);
volume(length: 120, width: 80, height: 100);

// Invalid - throws InvalidDimensionsException
volume([120, 80]);           // Only 2 dimensions
volume(length: 120, width: 80); // Missing height
```

## Positive Values Only

All dimensions must be positive numbers (> 0):

```php
// Valid - all positive
volume([120, 80, 100]);
volume([0.1, 0.1, 0.1]);

// Invalid - throws InvalidDimensionsException
volume([120, 0, 100]);    // Zero dimension
volume([120, -80, 100]);  // Negative dimension
volume([-1, -1, -1]);     // All negative
```

## Array Validation

### Indexed Arrays

Indexed arrays must contain exactly 3 values:

```php
// Valid
volume([120, 80, 100]);

// Invalid - throws InvalidDimensionsException
volume([120]);              // Too few
volume([120, 80]);          // Too few
volume([120, 80, 100, 50]); // Too many
```

### Associative Arrays

Associative arrays must use the exact keys: `length`, `width`, `height`:

```php
// Valid
volume(['length' => 120, 'width' => 80, 'height' => 100]);
volume(['height' => 100, 'length' => 120, 'width' => 80]); // Order doesn't matter

// Invalid - throws InvalidDimensionsException
volume(['l' => 120, 'w' => 80, 'h' => 100]);          // Wrong keys
volume(['depth' => 100, 'width' => 80, 'height' => 100]); // 'depth' not 'length'
```

## Loading Meter Validation

Loading meter calculations have additional parameter validation:

```php
use Cline\Volume\ValueObjects\LoadingMeter;

// Valid
LoadingMeter::fromCentimeters(120, 80, quantity: 1, stackingFactor: 1.0, truckWidth: 2.4);

// Invalid - throws InvalidLoadingMeterParameterException
LoadingMeter::fromCentimeters(120, 80, quantity: 0);     // quantity < 1
LoadingMeter::fromCentimeters(120, 80, quantity: -1);    // quantity < 1
LoadingMeter::fromCentimeters(120, 80, stackingFactor: 0);   // stackingFactor <= 0
LoadingMeter::fromCentimeters(120, 80, stackingFactor: -1);  // stackingFactor <= 0
LoadingMeter::fromCentimeters(120, 80, truckWidth: 0);   // truckWidth <= 0
```

## Floor Meter Validation

Floor meters require positive length and width:

```php
use Cline\Volume\ValueObjects\FloorMeter;

// Valid
FloorMeter::fromMeters(1.2, 0.8);
FloorMeter::fromCentimeters(120, 80);

// Invalid - throws InvalidFloorDimensionsException
FloorMeter::fromMeters(0, 0.8);    // Zero length
FloorMeter::fromMeters(1.2, -0.8); // Negative width
```

## Exception Types

| Exception | Thrown When |
|-----------|-------------|
| `InvalidDimensionsException` | Volume dimensions invalid (count, keys, values) |
| `InvalidFloorDimensionsException` | Floor dimensions non-positive |
| `InvalidLoadingMeterParameterException` | LDM parameters invalid |

## Error Messages

The library provides clear error messages for validation failures:

```php
try {
    volume([120, 80]); // Missing height
} catch (InvalidDimensionsException $e) {
    echo $e->getMessage();
    // "3 dimensions required, 2 provided"
}

try {
    volume([120, 0, 100]); // Zero width
} catch (InvalidDimensionsException $e) {
    echo $e->getMessage();
    // "All dimensions must be positive numbers"
}

try {
    volume(['length' => 120, 'width' => 80]); // Missing height
} catch (InvalidDimensionsException $e) {
    echo $e->getMessage();
    // "Missing required dimension: height"
}

try {
    LoadingMeter::fromCentimeters(120, 80, quantity: 0);
} catch (InvalidLoadingMeterParameterException $e) {
    echo $e->getMessage();
    // "Quantity must be at least 1"
}
```

## Safe Handling Pattern

```php
use Cline\Volume\Exceptions\InvalidDimensionsException;

function safeVolumeCalculation(array $dimensions): ?float
{
    try {
        return volume($dimensions)->meters()->value();
    } catch (InvalidDimensionsException $e) {
        log_error("Invalid dimensions: " . $e->getMessage());
        return null;
    }
}

// Usage
$result = safeVolumeCalculation($userInput);
if ($result === null) {
    echo "Please provide valid dimensions (length, width, height > 0)";
}
```
