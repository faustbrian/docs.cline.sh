---
title: Floor Meters
description: Calculate floor area (footprint) for warehouse and storage planning.
---

Floor meters calculate the footprint area (length × width) of cargo in square meters, providing the ground space required for storage or stacking.

## Basic Calculation

```php
use function Cline\Volume\volume;

$vol = volume([200, 150, 100]); // 200cm × 150cm × 100cm
$fm = $vol->floorMeters();

$fm->value();    // 3.0 m²
$fm->format(2);  // "3.00"
```

## Accessing Dimensions

```php
$fm = $vol->floorMeters();

// Original dimensions in meters
$fm->length;  // 2.0
$fm->width;   // 1.5

// Get the calculated area
$fm->value(); // 3.0 (square meters)

// Format for display
$fm->format(1); // "3.0"
(string) $fm;   // "3"
```

## Direct Instantiation

```php
use Cline\Volume\ValueObjects\FloorMeter;

// From meters
$fm = FloorMeter::fromMeters(2.0, 1.5);
$fm->value(); // 3.0 m²

// From centimeters
$fm = FloorMeter::fromCentimeters(200, 150);
$fm->value(); // 3.0 m²
```

## Practical Examples

### Warehouse Space Planning

```php
// Calculate floor space for multiple pallets
$pallet = volume([120, 80, 100]); // Standard Euro pallet
$singlePalletArea = $pallet->floorMeters()->value(); // 0.96 m²

// 50 pallets
$totalArea = $singlePalletArea * 50; // 48 m²
```

### Storage Optimization

```php
// Different package sizes
$small = volume([60, 40, 30])->floorMeters()->value();   // 0.24 m²
$medium = volume([100, 60, 50])->floorMeters()->value(); // 0.6 m²
$large = volume([150, 100, 80])->floorMeters()->value(); // 1.5 m²

// Calculate total warehouse footprint
$totalFootprint = ($small * 100) + ($medium * 50) + ($large * 25);
// = 24 + 30 + 37.5 = 91.5 m²
```

### Comparing Storage Efficiency

```php
$packages = [
    ['name' => 'Box A', 'dims' => [40, 30, 20]],
    ['name' => 'Box B', 'dims' => [50, 40, 30]],
    ['name' => 'Box C', 'dims' => [60, 45, 35]],
];

foreach ($packages as $pkg) {
    $vol = volume($pkg['dims']);
    $floor = $vol->floorMeters()->value();
    $cubic = $vol->meters()->value();
    $efficiency = $cubic / $floor; // Height utilization

    echo "{$pkg['name']}: {$floor} m² floor, {$cubic} m³ volume\n";
}
```

## API Reference

### FloorMeter::fromMeters()

```php
public static function fromMeters(float $length, float $width): FloorMeter
```

### FloorMeter::fromCentimeters()

```php
public static function fromCentimeters(float $length, float $width): FloorMeter
```

### Instance Methods

| Method | Return | Description |
|--------|--------|-------------|
| `value()` | `float` | Raw floor area in m² |
| `format(int $decimals = 2)` | `string` | Formatted with number_format |
| `__toString()` | `string` | String representation |

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$length` | `float` | Length dimension in meters |
| `$width` | `float` | Width dimension in meters |
| `$value` | `float` | Calculated area in m² |
