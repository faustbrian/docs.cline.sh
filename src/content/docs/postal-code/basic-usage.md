---
title: Basic Usage
description: Validating and formatting postal codes for different countries.
---

Validating and formatting postal codes for different countries.

## Validation

```php
use Cline\PostalCode\PostalCode;

// Basic validation
$result = PostalCode::validate('90210', 'US');

if ($result->isValid()) {
    echo "Valid: " . $result->formatted();
} else {
    echo "Invalid: " . $result->error();
}
```

## Country-Specific Examples

### United States

```php
// 5-digit ZIP
PostalCode::isValid('90210', 'US'); // true

// ZIP+4
PostalCode::isValid('90210-1234', 'US'); // true

// Invalid
PostalCode::isValid('9021', 'US'); // false (too short)
PostalCode::isValid('902101', 'US'); // false (too long)
```

### United Kingdom

```php
// Various UK formats
PostalCode::isValid('SW1A 1AA', 'GB'); // true
PostalCode::isValid('M1 1AE', 'GB'); // true
PostalCode::isValid('B33 8TH', 'GB'); // true
PostalCode::isValid('CR2 6XH', 'GB'); // true
PostalCode::isValid('DN55 1PT', 'GB'); // true

// Case insensitive
PostalCode::isValid('sw1a 1aa', 'GB'); // true
```

### Germany

```php
// 5-digit postal codes
PostalCode::isValid('10115', 'DE'); // true (Berlin)
PostalCode::isValid('80331', 'DE'); // true (Munich)

// Invalid
PostalCode::isValid('1011', 'DE'); // false
PostalCode::isValid('101155', 'DE'); // false
```

### Canada

```php
// Format: A1A 1A1
PostalCode::isValid('K1A 0B1', 'CA'); // true
PostalCode::isValid('M5V 2T6', 'CA'); // true

// Without space
PostalCode::isValid('K1A0B1', 'CA'); // true
```

### Other Countries

```php
// France (5 digits)
PostalCode::isValid('75001', 'FR'); // true

// Netherlands (4 digits + 2 letters)
PostalCode::isValid('1012 AB', 'NL'); // true

// Japan (7 digits with hyphen)
PostalCode::isValid('100-0001', 'JP'); // true

// Australia (4 digits)
PostalCode::isValid('2000', 'AU'); // true

// Brazil (8 digits with hyphen)
PostalCode::isValid('01310-100', 'BR'); // true
```

## Formatting

```php
// Auto-format based on country rules
$result = PostalCode::validate('sw1a1aa', 'GB');
$result->formatted(); // "SW1A 1AA"

$result = PostalCode::validate('k1a0b1', 'CA');
$result->formatted(); // "K1A 0B1"

// Get without formatting
$result->input(); // Original input
$result->normalized(); // Cleaned but not formatted
```

## Validation Result

```php
$result = PostalCode::validate('90210', 'US');

// Check validity
$result->isValid(); // true

// Get formatted value
$result->formatted(); // "90210"

// Get country
$result->country(); // "US"

// For invalid codes
$result = PostalCode::validate('invalid', 'US');
$result->isValid(); // false
$result->error(); // "Invalid postal code format for US"
```

## Batch Validation

```php
$postalCodes = ['90210', '10001', 'invalid', '94102'];

$results = array_map(
    fn($code) => [
        'code' => $code,
        'valid' => PostalCode::isValid($code, 'US'),
    ],
    $postalCodes
);
```
