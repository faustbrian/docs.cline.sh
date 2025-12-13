---
title: Exception Handling
description: Handle postal code validation errors gracefully.
---

Handle postal code validation errors gracefully.

## Validation Exceptions

```php
use Cline\PostalCode\PostalCode;
use Cline\PostalCode\Exceptions\InvalidPostalCodeException;

try {
    $result = PostalCode::validateOrFail('invalid', 'US');
} catch (InvalidPostalCodeException $e) {
    echo $e->getMessage();
    // "The postal code 'invalid' is not valid for US"

    echo $e->getPostalCode(); // "invalid"
    echo $e->getCountry(); // "US"
}
```

## Country Exceptions

```php
use Cline\PostalCode\Exceptions\UnsupportedCountryException;

try {
    PostalCode::validate('12345', 'XX');
} catch (UnsupportedCountryException $e) {
    echo $e->getMessage();
    // "The country 'XX' is not supported"

    echo $e->getCountry(); // "XX"
}
```

## Safe Validation

```php
// Returns result object instead of throwing
$result = PostalCode::validate('invalid', 'US');

if (!$result->isValid()) {
    $error = $result->error();
    // Handle error without exception
}

// Boolean check (never throws)
$isValid = PostalCode::isValid('90210', 'US');
```

## Custom Error Messages

```php
use Cline\PostalCode\PostalCode;
use Cline\PostalCode\Exceptions\InvalidPostalCodeException;

InvalidPostalCodeException::$messageFormat =
    'ZIP code ":code" is invalid for :country';

try {
    PostalCode::validateOrFail('bad', 'US');
} catch (InvalidPostalCodeException $e) {
    echo $e->getMessage();
    // "ZIP code 'bad' is invalid for US"
}
```

## Batch Validation with Error Collection

```php
use Cline\PostalCode\PostalCode;

$codes = ['90210', 'invalid', '10001', 'bad'];
$errors = [];
$valid = [];

foreach ($codes as $code) {
    $result = PostalCode::validate($code, 'US');

    if ($result->isValid()) {
        $valid[] = $result->formatted();
    } else {
        $errors[] = [
            'code' => $code,
            'error' => $result->error(),
        ];
    }
}

// $valid = ['90210', '10001']
// $errors = [
//     ['code' => 'invalid', 'error' => '...'],
//     ['code' => 'bad', 'error' => '...'],
// ]
```

## Laravel Validation

```php
use Cline\PostalCode\Rules\PostalCodeRule;

// In form request
public function rules(): array
{
    return [
        'postal_code' => ['required', new PostalCodeRule('US')],
    ];
}

// Dynamic country from request
public function rules(): array
{
    return [
        'country' => ['required', 'string', 'size:2'],
        'postal_code' => ['required', new PostalCodeRule($this->country)],
    ];
}

// Custom error message
public function messages(): array
{
    return [
        'postal_code' => 'Please enter a valid ZIP code.',
    ];
}
```

## Error Logging

```php
use Cline\PostalCode\PostalCode;
use Cline\PostalCode\Exceptions\InvalidPostalCodeException;
use Illuminate\Support\Facades\Log;

function validatePostalCode(string $code, string $country): ?string
{
    try {
        $result = PostalCode::validateOrFail($code, $country);
        return $result->formatted();
    } catch (InvalidPostalCodeException $e) {
        Log::warning('Invalid postal code submitted', [
            'code' => $code,
            'country' => $country,
            'error' => $e->getMessage(),
            'user_id' => auth()->id(),
        ]);
        return null;
    }
}
```

## Exception Hierarchy

```php
// Base exception
Cline\PostalCode\Exceptions\PostalCodeException

// Specific exceptions
├── InvalidPostalCodeException  // Invalid format
├── UnsupportedCountryException // Country not supported
└── HandlerException            // Handler error
```
