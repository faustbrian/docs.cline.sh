---
title: Basic Usage
description: Parsing, formatting, and validating phone numbers.
---

Parsing, formatting, and validating phone numbers.

## Parsing Phone Numbers

```php
use Cline\PhoneNumber\PhoneNumber;

// International format
$phone = PhoneNumber::parse('+44 20 7946 0958');

// With region hint
$phone = PhoneNumber::parse('020 7946 0958', 'GB');

// Various input formats
PhoneNumber::parse('+1-555-123-4567');
PhoneNumber::parse('(555) 123-4567', 'US');
PhoneNumber::parse('555.123.4567', 'US');
PhoneNumber::parse('5551234567', 'US');
```

## Formatting

### Standard Formats

```php
$phone = PhoneNumber::parse('+1 555 123 4567');

// E.164 (international standard)
$phone->formatE164(); // "+15551234567"

// International
$phone->formatInternational(); // "+1 555-123-4567"

// National
$phone->formatNational(); // "(555) 123-4567"

// RFC 3966 (tel: URI)
$phone->formatRFC3966(); // "tel:+1-555-123-4567"
```

### Custom Formatting

```php
// Format for specific region
$phone->formatForRegion('US'); // "(555) 123-4567"
$phone->formatForRegion('GB'); // "001 555 123 4567"

// Out-of-country format
$phone->formatOutOfCountry('GB'); // "00 1 555-123-4567"
```

## Validation

```php
$phone = PhoneNumber::parse('+1 555 123 4567');

// Basic validation
$phone->isValid(); // true

// Region-specific validation
$phone->isValidForRegion('US'); // true
$phone->isValidForRegion('CA'); // true (same country code)
$phone->isValidForRegion('GB'); // false

// Possible number (less strict)
$phone->isPossible(); // true
```

## Phone Number Components

```php
$phone = PhoneNumber::parse('+44 20 7946 0958');

// Country code
$phone->getCountryCode(); // 44

// National number
$phone->getNationalNumber(); // "2079460958"

// Region code
$phone->getRegionCode(); // "GB"

// Extension (if present)
$phone = PhoneNumber::parse('+1 555 123 4567 ext. 890');
$phone->getExtension(); // "890"
```

## Phone Number Types

```php
$phone = PhoneNumber::parse('+1 555 123 4567');

// Get type as string
$phone->getType();
// Possible values:
// - FIXED_LINE
// - MOBILE
// - FIXED_LINE_OR_MOBILE
// - TOLL_FREE
// - PREMIUM_RATE
// - SHARED_COST
// - VOIP
// - PERSONAL_NUMBER
// - PAGER
// - UAN
// - VOICEMAIL
// - UNKNOWN

// Type checks
$phone->isMobile();
$phone->isFixedLine();
$phone->isTollFree();
$phone->isPremiumRate();
$phone->isVoip();
```

## Comparison

```php
$phone1 = PhoneNumber::parse('+1 555 123 4567');
$phone2 = PhoneNumber::parse('(555) 123-4567', 'US');
$phone3 = PhoneNumber::parse('+44 20 7946 0958');

$phone1->equals($phone2); // true (same number)
$phone1->equals($phone3); // false

// Compare as string
(string) $phone1 === (string) $phone2; // true
```

## Static Helpers

```php
use Cline\PhoneNumber\PhoneNumber;

// Quick validation
PhoneNumber::isValid('+1 555 123 4567'); // true
PhoneNumber::isValid('invalid'); // false

// Quick formatting
PhoneNumber::formatE164('+1 (555) 123-4567'); // "+15551234567"
```
