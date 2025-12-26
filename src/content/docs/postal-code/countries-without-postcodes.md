---
title: Countries Without Postal Codes
description: Handling countries that do not have postal code systems.
---

Some countries do not have postal code systems. When shipping to these destinations or validating addresses, you may need to detect this and provide a fallback value.

## Checking Postal Code Support

Use the `Country` helper to check if a country uses postal codes:

```php
use Cline\PostalCode\Support\Country;

// Countries with postal codes
Country::hasPostalCode('US'); // true
Country::hasPostalCode('GB'); // true
Country::hasPostalCode('DE'); // true

// Countries without postal codes
Country::hasPostalCode('HK'); // false (Hong Kong)
Country::hasPostalCode('AE'); // false (United Arab Emirates)
Country::hasPostalCode('QA'); // false (Qatar)
```

The method is case-insensitive:

```php
Country::hasPostalCode('hk'); // false
Country::hasPostalCode('Hk'); // false
```

## Fallback Value

When a country doesn't use postal codes but your system requires one (e.g., for shipping carriers), use the fallback:

```php
use Cline\PostalCode\Support\Country;

$postalCode = Country::hasPostalCode($countryCode)
    ? $userProvidedPostalCode
    : Country::fallbackPostalCode(); // Returns '00000'
```

## Practical Example

Here's how to normalize postal codes in a shipping application:

```php
use Cline\PostalCode\Facades\PostalCode;
use Cline\PostalCode\Support\Country;

function normalizePostalCode(string $postalCode, string $countryCode): string
{
    // Countries without postal codes get the fallback
    if (!Country::hasPostalCode($countryCode)) {
        return Country::fallbackPostalCode();
    }

    // Format valid postal codes, return original if invalid
    return PostalCode::for($postalCode, $countryCode)->formatOr($postalCode);
}

// Usage
normalizePostalCode('90210', 'US');      // '90210'
normalizePostalCode('WC2E9RZ', 'GB');    // 'WC2E 9RZ'
normalizePostalCode('anything', 'HK');   // '00000'
normalizePostalCode('anything', 'QA');   // '00000'
```

## Complete List

Get all countries without postal code systems:

```php
use Cline\PostalCode\Support\Country;

$countries = Country::countriesWithoutPostalCodes();
// ['AE', 'AG', 'AN', 'AO', 'AW', 'BF', 'BI', ...]
```

### Countries Without Postal Codes

| Code | Country |
|------|---------|
| AE | United Arab Emirates |
| AG | Antigua and Barbuda |
| AN | Netherlands Antilles (former) |
| AO | Angola |
| AW | Aruba |
| BF | Burkina Faso |
| BI | Burundi |
| BJ | Benin |
| BO | Bolivia |
| BS | Bahamas |
| BW | Botswana |
| BZ | Belize |
| CD | Democratic Republic of the Congo |
| CF | Central African Republic |
| CG | Republic of the Congo |
| CI | Ivory Coast |
| CK | Cook Islands |
| CM | Cameroon |
| CW | Curacao |
| DJ | Djibouti |
| DM | Dominica |
| ER | Eritrea |
| FJ | Fiji |
| GA | Gabon |
| GD | Grenada |
| GH | Ghana |
| GM | Gambia |
| GQ | Equatorial Guinea |
| GY | Guyana |
| HK | Hong Kong |
| JM | Jamaica |
| KI | Kiribati |
| KM | Comoros |
| KN | Saint Kitts and Nevis |
| KP | North Korea |
| ML | Mali |
| MO | Macau |
| MR | Mauritania |
| MW | Malawi |
| NR | Nauru |
| NU | Niue |
| QA | Qatar |
| RW | Rwanda |
| SB | Solomon Islands |
| SC | Seychelles |
| SL | Sierra Leone |
| SO | Somalia |
| SR | Suriname |
| SS | South Sudan |
| ST | Sao Tome and Principe |
| SX | Sint Maarten |
| SY | Syria |
| TD | Chad |
| TG | Togo |
| TK | Tokelau |
| TL | Timor-Leste |
| TO | Tonga |
| TV | Tuvalu |
| UG | Uganda |
| VU | Vanuatu |
| YE | Yemen |
| ZW | Zimbabwe |
