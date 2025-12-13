---
title: Laravel Integration
description: Use Postal Code with Laravel applications.
---

Use Postal Code with Laravel applications.

## Installation

```bash
composer require cline/postal-code
```

The service provider is auto-discovered in Laravel.

## Validation Rules

### Basic Rule

```php
use Cline\PostalCode\Rules\PostalCodeRule;

// In form request
public function rules(): array
{
    return [
        'postal_code' => ['required', new PostalCodeRule('US')],
    ];
}
```

### Dynamic Country

```php
public function rules(): array
{
    return [
        'country' => ['required', 'string', 'size:2'],
        'postal_code' => [
            'required',
            new PostalCodeRule($this->input('country')),
        ],
    ];
}
```

### Custom Messages

```php
public function rules(): array
{
    return [
        'postal_code' => ['required', new PostalCodeRule('US')],
    ];
}

public function messages(): array
{
    return [
        'postal_code' => 'Please enter a valid ZIP code.',
    ];
}
```

## Validation Rule String

```php
// Using rule string (if registered)
public function rules(): array
{
    return [
        'postal_code' => 'required|postal_code:US',
    ];
}

// Register in AppServiceProvider
use Cline\PostalCode\PostalCode;
use Illuminate\Support\Facades\Validator;

public function boot(): void
{
    Validator::extend('postal_code', function ($attribute, $value, $parameters) {
        $country = $parameters[0] ?? 'US';
        return PostalCode::isValid($value, $country);
    });
}
```

## Facade

```php
use Cline\PostalCode\Facades\PostalCode;

// Validate
$result = PostalCode::validate('90210', 'US');

// Quick check
PostalCode::isValid('90210', 'US');

// Format
PostalCode::format('sw1a1aa', 'GB'); // "SW1A 1AA"
```

## Model Casting

```php
use Cline\PostalCode\Casts\PostalCodeCast;

class Address extends Model
{
    protected $casts = [
        'postal_code' => PostalCodeCast::class . ':US',
    ];
}

// Usage
$address = new Address();
$address->postal_code = '90210';
$address->postal_code; // Returns formatted postal code
```

## Eloquent Accessor

```php
use Cline\PostalCode\PostalCode;

class Address extends Model
{
    public function getFormattedPostalCodeAttribute(): string
    {
        $result = PostalCode::validate(
            $this->postal_code,
            $this->country_code
        );

        return $result->isValid()
            ? $result->formatted()
            : $this->postal_code;
    }
}
```

## Blade Directive

```php
// Register in AppServiceProvider
Blade::directive('postalcode', function ($expression) {
    return "<?php echo \Cline\PostalCode\PostalCode::format($expression); ?>";
});

// Usage in Blade
@postalcode($address->postal_code, $address->country)
```

## API Resource

```php
use Cline\PostalCode\PostalCode;
use Illuminate\Http\Resources\Json\JsonResource;

class AddressResource extends JsonResource
{
    public function toArray($request): array
    {
        $result = PostalCode::validate(
            $this->postal_code,
            $this->country_code
        );

        return [
            'street' => $this->street,
            'city' => $this->city,
            'postal_code' => $result->formatted(),
            'postal_code_valid' => $result->isValid(),
            'country' => $this->country_code,
        ];
    }
}
```

## Configuration

```php
// config/postal-code.php
return [
    // Default country for validation
    'default_country' => env('POSTAL_CODE_COUNTRY', 'US'),

    // Custom handlers
    'handlers' => [
        // 'XX' => App\PostalCode\CustomHandler::class,
    ],
];
```

```bash
php artisan vendor:publish --tag=postal-code-config
```
