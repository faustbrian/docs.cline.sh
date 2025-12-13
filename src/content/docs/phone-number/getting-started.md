---
title: Getting Started
description: Install and start using Phone Number for parsing and validating phone numbers in PHP.
---

Phone Number is a PHP library for parsing, validating, and formatting phone numbers using libphonenumber.

## Installation

```bash
composer require cline/phone-number
```

## Basic Usage

```php
use Cline\PhoneNumber\PhoneNumber;

// Parse a phone number
$phone = PhoneNumber::parse('+1 (555) 123-4567');

// Get formatted versions
$phone->format();           // "+15551234567"
$phone->formatNational();   // "(555) 123-4567"
$phone->formatInternational(); // "+1 555-123-4567"
$phone->formatE164();       // "+15551234567"

// Get components
$phone->getCountryCode();   // 1
$phone->getNationalNumber(); // "5551234567"
$phone->getRegionCode();    // "US"
```

## Validation

```php
use Cline\PhoneNumber\PhoneNumber;

// Check if valid
$phone = PhoneNumber::parse('+1 555 123 4567');
$phone->isValid(); // true

// Invalid number
$phone = PhoneNumber::parse('not a number');
$phone->isValid(); // false

// Validate for specific region
$phone->isValidForRegion('US'); // true
$phone->isValidForRegion('GB'); // false
```

## Parsing with Region

```php
// Parse with default region
$phone = PhoneNumber::parse('555-123-4567', 'US');

// Useful for numbers without country code
$phone = PhoneNumber::parse('020 7946 0958', 'GB');
$phone->formatInternational(); // "+44 20 7946 0958"
```

## Phone Number Types

```php
$phone = PhoneNumber::parse('+1 555 123 4567');

$phone->getType(); // "FIXED_LINE_OR_MOBILE"
$phone->isMobile(); // true/false
$phone->isFixedLine(); // true/false
$phone->isTollFree(); // false
```

## Next Steps

- [Basic Usage](/phone-number/basic-usage/) - Detailed parsing and formatting
- [Exception Handling](/phone-number/exception-handling/) - Handle parsing errors
- [Metadata](/phone-number/metadata/) - Access phone number metadata
