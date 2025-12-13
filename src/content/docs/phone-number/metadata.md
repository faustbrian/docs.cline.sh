---
title: Metadata
description: Access phone number metadata and carrier information.
---

Access phone number metadata and carrier information.

## Region Information

```php
use Cline\PhoneNumber\PhoneNumber;

$phone = PhoneNumber::parse('+44 20 7946 0958');

// Region code
$phone->getRegionCode(); // "GB"

// Country code
$phone->getCountryCode(); // 44

// Region name
$phone->getRegionName(); // "United Kingdom"

// Is geographic number
$phone->isGeographic(); // true
```

## Carrier Information

```php
use Cline\PhoneNumber\PhoneNumber;

$phone = PhoneNumber::parse('+1 555 123 4567');

// Carrier name (when available)
$phone->getCarrierName(); // "Example Carrier"
$phone->getCarrierName('en'); // English
$phone->getCarrierName('de'); // German

// Note: Carrier info may not be available for all numbers
```

## Timezone Information

```php
$phone = PhoneNumber::parse('+1 212 555 1234'); // New York

// Get timezones
$timezones = $phone->getTimezones();
// ["America/New_York"]

// Multiple timezones possible for some numbers
$phone = PhoneNumber::parse('+1 555 123 4567');
$timezones = $phone->getTimezones();
// May return multiple if number spans timezone boundaries
```

## Geocoding

```php
use Cline\PhoneNumber\PhoneNumber;

$phone = PhoneNumber::parse('+1 212 555 1234');

// Geographic description
$phone->getGeoDescription(); // "New York, NY"
$phone->getGeoDescription('en'); // "New York, NY"
$phone->getGeoDescription('de'); // "New York, NY"
$phone->getGeoDescription('fr'); // "New York, NY"
```

## Example Numbers

```php
use Cline\PhoneNumber\PhoneNumber;

// Get example number for region
$example = PhoneNumber::getExampleNumber('US');
$example->formatNational(); // "(201) 555-0123"

// Get example for specific type
$mobile = PhoneNumber::getExampleNumber('US', 'MOBILE');
$tollFree = PhoneNumber::getExampleNumber('US', 'TOLL_FREE');
```

## Metadata Access

```php
use Cline\PhoneNumber\PhoneNumber;

// Get all supported regions
$regions = PhoneNumber::getSupportedRegions();
// ["AC", "AD", "AE", "AF", ...]

// Get country code for region
$code = PhoneNumber::getCountryCodeForRegion('US'); // 1
$code = PhoneNumber::getCountryCodeForRegion('GB'); // 44

// Get region for country code
$region = PhoneNumber::getRegionForCountryCode(1); // "US"
$region = PhoneNumber::getRegionForCountryCode(44); // "GB"

// Note: Country code 1 has multiple regions (US, CA, etc.)
$regions = PhoneNumber::getRegionsForCountryCode(1);
// ["US", "CA", "AG", "AI", ...]
```

## Number Length

```php
$phone = PhoneNumber::parse('+1 555 123 4567');

// Get length information for region
$phone->getNationalNumberLength(); // 10

// Check possible lengths
$phone->isPossibleLength(); // true
```

## Formatting Metadata

```php
use Cline\PhoneNumber\PhoneNumber;

// Get format patterns for region
$patterns = PhoneNumber::getFormatPatterns('US');

// Get national prefix
$prefix = PhoneNumber::getNationalPrefix('US'); // "1"
$prefix = PhoneNumber::getNationalPrefix('GB'); // "0"
```
