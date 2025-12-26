---
title: Getting Started
description: Installation and basic usage of the Cline Intl PHP package for Laravel applications
---

Cline Intl provides type-safe internationalization value objects and validation rules for Laravel applications. Built on top of Symfony Intl components and Spatie Laravel Data, it offers a modern, elegant approach to handling countries, currencies, locales, timezones, phone numbers, postal codes, and addresses.

## Requirements

- **PHP 8.5+**
- **Laravel 12+**
- **Spatie Laravel Data 4.18+**

## Installation

Install the package via Composer:

```bash
composer require cline/intl
```

No configuration or service provider registration is required. The package is automatically discovered by Laravel.

## Key Features

### Type-Safe Value Objects

All internationalization data is wrapped in immutable value objects that extend Spatie's Laravel Data:

```php
use Cline\Intl\ValueObjects\Country;
use Cline\Intl\ValueObjects\Currency;
use Cline\Intl\ValueObjects\Language;
use Cline\Intl\ValueObjects\Locale;
use Cline\Intl\ValueObjects\TimeZone;
use Cline\Intl\ValueObjects\PhoneNumber;
use Cline\Intl\ValueObjects\PostalCode;
use Cline\Intl\ValueObjects\Address;
```

### Eloquent Model Casts

Seamlessly integrate value objects with your Eloquent models using built-in casts:

```php
use Cline\Intl\Data\Cast\CountryCast;
use Cline\Intl\Data\Cast\CurrencyCast;
use Cline\Intl\Data\Cast\LanguageCast;
use Cline\Intl\Data\Cast\LocaleCast;
use Cline\Intl\Data\Cast\TimeZoneCast;
use Cline\Intl\Data\Cast\PhoneNumberCast;
use Cline\Intl\Data\Cast\PostalCodeCast;

class User extends Model
{
    protected function casts(): array
    {
        return [
            'country' => CountryCast::class,
            'preferred_currency' => CurrencyCast::class,
            'language' => LanguageCast::class,
            'timezone' => TimeZoneCast::class,
        ];
    }
}
```

### Validation Rules

Validate user input with dedicated validation rules:

```php
use Cline\Intl\Rules\CountryRule;
use Cline\Intl\Rules\CurrencyRule;
use Cline\Intl\Rules\LanguageRule;
use Cline\Intl\Rules\LocaleRule;
use Cline\Intl\Rules\PhoneNumberRule;
use Cline\Intl\Rules\TimeZoneRule;

$request->validate([
    'country' => ['required', new CountryRule()],
    'currency' => ['required', new CurrencyRule()],
    'language' => ['required', new LanguageRule()],
    'phone' => ['required', new PhoneNumberRule()],
]);
```

## Quick Example

Here's a complete example showing the basic workflow:

```php
use Cline\Intl\ValueObjects\Country;
use Cline\Intl\ValueObjects\Currency;

// Create value objects from strings
$country = Country::createFromString('US');
$currency = Currency::createFromString('USD');

// Access properties
echo $country->localized;  // "United States"
echo $country->alpha2;     // "US"
echo $country->alpha3;     // "USA"

echo $currency->name;             // "US Dollar"
echo $currency->symbol;           // "$"
echo $currency->fractionDigits;   // 2

// Use in Eloquent models
$user = new User();
$user->country = 'US';  // Automatically cast to Country object
$user->save();

// Access as object
echo $user->country->localized;  // "United States"

// Compare instances
$other = Country::createFromString('US');
$country->isEqualTo($other);  // true

// Convert to string
(string) $country;  // "US"
```

## Underlying Libraries

Cline Intl builds on solid foundation libraries:

- **[Symfony Intl](https://symfony.com/doc/current/components/intl.html)** - Country, Currency, Language, Locale, and TimeZone data
- **[Brick PhoneNumber](https://github.com/brick/phonenumber)** - Phone number parsing and formatting
- **[Brick Postcode](https://github.com/brick/postcode)** - Postal code validation and formatting
- **[CommerceGuys Addressing](https://github.com/commerceguys/addressing)** - Address handling based on xAL standard
- **[Spatie Laravel Data](https://spatie.be/docs/laravel-data)** - Data transfer objects with casting support

## Next Steps

Explore the detailed documentation for each value object:

- **[Country](/intl/country)** - ISO 3166-1 country codes with alpha-2 and alpha-3 support
- **[Currency](/intl/currency)** - ISO 4217 currency codes with symbols and fraction digits
- **[Language & Locale](/intl/language-locale)** - Language and locale handling
- **[TimeZone](/intl/timezone)** - Timezone support with localized names
- **[Casts & Rules](/intl/casts-and-rules)** - Complete guide to model casts and validation rules
