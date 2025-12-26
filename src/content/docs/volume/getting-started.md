---
title: Getting Started
description: Volume and dimensional calculation utilities for PHP 8.4+ with chainable unit conversions and logistics calculations.
---

Volume and dimensional calculation utilities for PHP 8.4+ with chainable unit conversions and logistics calculations.

## Requirements

> **Requires [PHP 8.4+](https://php.net/releases/)**

## Installation

```bash
composer require cline/volume
```

## Quick Example

```php
use function Cline\Volume\volume;

// Create from dimensions (default: centimeters)
$vol = volume([120, 80, 100]);

// Chainable unit conversions
$vol->centimeters()->value(); // 960000 cm³
$vol->meters()->value();      // 0.96 m³
$vol->decimeters()->value();  // 960 dm³

// Logistics calculations
$vol->floorMeters()->value();   // 0.96 m² (floor area)
$vol->loadingMeters()->value(); // 0.4 LDM (truck space)
```

## Input Formats

The `volume()` function accepts multiple input formats:

```php
// Indexed array [length, width, height]
volume([120, 80, 100]);

// Associative array (order doesn't matter)
volume(['height' => 100, 'length' => 120, 'width' => 80]);

// Named parameters
volume(length: 120, width: 80, height: 100);

// With explicit unit
volume([1.2, 0.8, 1.0], Unit::Meters);
volume(length: 1.2, width: 0.8, height: 1.0, unit: Unit::Meters);
```

## Available Units

```php
use Cline\Volume\Enums\Unit;

Unit::Centimeters; // Default
Unit::Decimeters;
Unit::Meters;
```

## Value Objects

All conversions return immutable value objects:

```php
$cm3 = $vol->centimeters();

$cm3->value();     // 960000.0 (raw value)
$cm3->format(2);   // "960,000.00" (formatted)
(string) $cm3;     // "960000" (string cast)

// Access original dimensions
$cm3->length;  // 120.0
$cm3->width;   // 80.0
$cm3->height;  // 100.0
```

## Features

- **[Basic Usage](/volume/basic-usage/)** - Creating volumes and unit conversions
- **[Loading Meters](/volume/loading-meters/)** - Freight logistics calculations
- **[Floor Meters](/volume/floor-meters/)** - Floor area calculations
- **[Examples](/volume/examples/)** - Real-world usage patterns
- **[Validation](/volume/validation/)** - Input validation rules
