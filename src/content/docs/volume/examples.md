---
title: Examples
description: Real-world usage patterns for volume calculations in e-commerce, logistics, and warehousing.
---

Practical examples demonstrating real-world usage of the volume library.

## E-Commerce Shipping Calculator

Calculate shipping costs based on package volume and dimensions:

```php
use function Cline\Volume\volume;

function calculateShippingCost(array $dimensions): float
{
    $vol = volume($dimensions); // dimensions in centimeters

    // Cubic meter pricing
    $cubicMeters = $vol->meters()->value();
    $volumeRate = $cubicMeters * 25.50; // €25.50 per m³

    // Loading meter pricing for logistics
    $loadingMeters = $vol->loadingMeters()->value();
    $ldmRate = $loadingMeters * 15.00; // €15.00 per LDM

    // Use whichever is higher (dimensional weight pricing)
    return max($volumeRate, $ldmRate);
}

$cost = calculateShippingCost([120, 80, 100]);
echo "Shipping cost: €" . number_format($cost, 2);
```

## Warehouse Inventory Management

Track storage space utilization:

```php
class WarehouseInventory
{
    private array $items = [];

    public function addItem(string $sku, array $dimensions, int $quantity): void
    {
        $vol = volume($dimensions);

        $this->items[$sku] = [
            'dimensions' => $dimensions,
            'quantity' => $quantity,
            'floor_space' => $vol->floorMeters()->value() * $quantity,
            'cubic_meters' => $vol->meters()->value() * $quantity,
        ];
    }

    public function getTotalFloorSpace(): float
    {
        return array_sum(array_column($this->items, 'floor_space'));
    }

    public function getTotalVolume(): float
    {
        return array_sum(array_column($this->items, 'cubic_meters'));
    }
}

$inventory = new WarehouseInventory();
$inventory->addItem('PALLET-001', [120, 80, 100], 50);
$inventory->addItem('BOX-LARGE', [60, 40, 40], 200);

echo "Total floor space: {$inventory->getTotalFloorSpace()} m²\n";
echo "Total volume: {$inventory->getTotalVolume()} m³\n";
```

## Pallet Loading Optimizer

Calculate optimal truck loading with stacking:

```php
class PalletLoadingOptimizer
{
    public function calculateTruckCapacity(
        array $palletDimensions,
        int $quantity,
        float $stackingFactor,
        float $truckLength = 13.6 // Standard truck length in meters
    ): array {
        $vol = volume($palletDimensions);

        $singlePalletLdm = $vol->loadingMeters()->value();
        $totalLdm = $vol->loadingMeters(
            quantity: $quantity,
            stackingFactor: $stackingFactor
        )->value();

        $trucksNeeded = ceil($totalLdm / $truckLength);
        $utilizationPercent = ($totalLdm / ($trucksNeeded * $truckLength)) * 100;

        return [
            'total_ldm' => $totalLdm,
            'trucks_needed' => (int) $trucksNeeded,
            'utilization' => round($utilizationPercent, 1),
            'ldm_per_truck' => round($totalLdm / $trucksNeeded, 2),
        ];
    }
}

$optimizer = new PalletLoadingOptimizer();
$result = $optimizer->calculateTruckCapacity(
    palletDimensions: [120, 80, 145], // Euro pallet
    quantity: 33,
    stackingFactor: 2.0 // Can stack 2 high
);

print_r($result);
// [
//     'total_ldm' => 6.6,
//     'trucks_needed' => 1,
//     'utilization' => 48.5,
//     'ldm_per_truck' => 6.6,
// ]
```

## Multi-Unit Conversion Dashboard

Display volume in multiple units for comparison:

```php
class VolumeConverter
{
    public function __construct(
        private readonly Volume $volume
    ) {}

    public static function fromDimensions(array $dimensions): self
    {
        return new self(volume($dimensions));
    }

    public function getAllMeasurements(): array
    {
        return [
            'cubic_centimeters' => $this->volume->centimeters()->format(0),
            'cubic_decimeters' => $this->volume->decimeters()->format(2),
            'cubic_meters' => $this->volume->meters()->format(4),
            'floor_meters' => $this->volume->floorMeters()->format(2),
            'loading_meters' => $this->volume->loadingMeters()->format(2),
        ];
    }

    public function displayTable(): void
    {
        $measurements = $this->getAllMeasurements();

        echo "Volume Measurements:\n";
        echo str_repeat('-', 40) . "\n";
        foreach ($measurements as $unit => $value) {
            $label = str_pad(ucwords(str_replace('_', ' ', $unit)), 20);
            echo "{$label}: {$value}\n";
        }
    }
}

$converter = VolumeConverter::fromDimensions([120, 80, 100]);
$converter->displayTable();
// Volume Measurements:
// ----------------------------------------
// Cubic Centimeters   : 960,000
// Cubic Decimeters    : 960.00
// Cubic Meters        : 0.9600
// Floor Meters        : 0.96
// Loading Meters      : 0.40
```

## Batch Processing with Different Units

Process multiple items with varying unit inputs:

```php
use Cline\Volume\Enums\Unit;

$shipment = [
    ['name' => 'Pallet A', 'dims' => [120, 80, 145], 'unit' => Unit::Centimeters],
    ['name' => 'Crate B', 'dims' => [1.5, 1.2, 2.0], 'unit' => Unit::Meters],
    ['name' => 'Box C', 'dims' => [50, 40, 30], 'unit' => Unit::Centimeters],
];

$totalCubicMeters = 0;
$totalLoadingMeters = 0;

foreach ($shipment as $item) {
    $vol = volume($item['dims'], $item['unit']);
    $cubicMeters = $vol->meters()->value();
    $loadingMeters = $vol->loadingMeters()->value();

    $totalCubicMeters += $cubicMeters;
    $totalLoadingMeters += $loadingMeters;

    printf(
        "%s: %.3f m³, %.2f LDM\n",
        $item['name'],
        $cubicMeters,
        $loadingMeters
    );
}

echo "\nTotals:\n";
echo "Cubic Meters: {$totalCubicMeters} m³\n";
echo "Loading Meters: {$totalLoadingMeters} LDM\n";
```

## Container Packing Estimation

Estimate how many items fit in a shipping container:

```php
class ContainerPacker
{
    // Standard 40ft container internal dimensions
    private const CONTAINER_40FT = [
        'length' => 1203, // cm
        'width' => 235,   // cm
        'height' => 269,  // cm
    ];

    public function estimateFit(array $itemDimensions, float $utilizationFactor = 0.85): int
    {
        $container = volume(self::CONTAINER_40FT);
        $item = volume($itemDimensions);

        $containerVolume = $container->meters()->value();
        $itemVolume = $item->meters()->value();

        // Apply utilization factor for realistic packing
        $usableVolume = $containerVolume * $utilizationFactor;

        return (int) floor($usableVolume / $itemVolume);
    }

    public function estimateByLoadingMeters(array $itemDimensions): int
    {
        $containerLdm = self::CONTAINER_40FT['length'] / 100 / 2.4;
        $itemLdm = volume($itemDimensions)->loadingMeters()->value();

        return (int) floor($containerLdm / $itemLdm);
    }
}

$packer = new ContainerPacker();
$boxDimensions = [60, 40, 40]; // Standard shipping box

echo "By volume: " . $packer->estimateFit($boxDimensions) . " boxes\n";
echo "By LDM: " . $packer->estimateByLoadingMeters($boxDimensions) . " boxes\n";
```
