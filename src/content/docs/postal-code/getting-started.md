---
title: Getting Started
description: Install and start using Postal Code for validating postal codes in PHP.
---

Postal Code is a PHP library for validating postal/ZIP codes for countries worldwide.

## Installation

```bash
composer require cline/postal-code
```

## Basic Usage

```php
use Cline\PostalCode\PostalCode;

// Validate a postal code
$result = PostalCode::validate('90210', 'US');
$result->isValid(); // true

// UK postal code
$result = PostalCode::validate('SW1A 1AA', 'GB');
$result->isValid(); // true

// German postal code
$result = PostalCode::validate('10115', 'DE');
$result->isValid(); // true
```

## Quick Validation

```php
use Cline\PostalCode\PostalCode;

// Returns boolean
PostalCode::isValid('90210', 'US'); // true
PostalCode::isValid('invalid', 'US'); // false

// With exception on invalid
PostalCode::validateOrFail('90210', 'US'); // Returns result
PostalCode::validateOrFail('invalid', 'US'); // Throws exception
```

## Supported Countries

```php
use Cline\PostalCode\PostalCode;

// Get all supported country codes
$countries = PostalCode::getSupportedCountries();
// ["AD", "AE", "AF", "AG", ...]

// Check if country is supported
PostalCode::isCountrySupported('US'); // true
PostalCode::isCountrySupported('XX'); // false
```

## Formatting

```php
$result = PostalCode::validate('sw1a1aa', 'GB');

// Get formatted version
$result->formatted(); // "SW1A 1AA"

// Original input
$result->input(); // "sw1a1aa"
```

## Next Steps

- [Basic Usage](/postal-code/basic-usage/) - Validation patterns
- [Custom Handlers](/postal-code/custom-handlers/) - Add custom validators
- [Exception Handling](/postal-code/exception-handling/) - Handle validation errors
- [Laravel Integration](/postal-code/laravel-integration/) - Use with Laravel
