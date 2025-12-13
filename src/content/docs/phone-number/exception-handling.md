---
title: Exception Handling
description: Handling phone number parsing and validation errors.
---

Handling phone number parsing and validation errors.

## Parse Exceptions

```php
use Cline\PhoneNumber\PhoneNumber;
use Cline\PhoneNumber\Exceptions\PhoneNumberParseException;

try {
    $phone = PhoneNumber::parse('not a phone number');
} catch (PhoneNumberParseException $e) {
    echo $e->getMessage();
    // "The string supplied did not seem to be a phone number"

    echo $e->getErrorType();
    // "NOT_A_NUMBER"
}
```

## Error Types

```php
use Cline\PhoneNumber\Exceptions\PhoneNumberParseException;

// NOT_A_NUMBER - Input doesn't look like a phone number
try {
    PhoneNumber::parse('hello world');
} catch (PhoneNumberParseException $e) {
    $e->getErrorType(); // "NOT_A_NUMBER"
}

// INVALID_COUNTRY_CODE - Country code not recognized
try {
    PhoneNumber::parse('+999 123 456');
} catch (PhoneNumberParseException $e) {
    $e->getErrorType(); // "INVALID_COUNTRY_CODE"
}

// TOO_SHORT_AFTER_IDD - Number too short after country code
try {
    PhoneNumber::parse('+1 12');
} catch (PhoneNumberParseException $e) {
    $e->getErrorType(); // "TOO_SHORT_AFTER_IDD"
}

// TOO_SHORT_NSN - National number too short
try {
    PhoneNumber::parse('123', 'US');
} catch (PhoneNumberParseException $e) {
    $e->getErrorType(); // "TOO_SHORT_NSN"
}

// TOO_LONG - Number is too long
try {
    PhoneNumber::parse('+1 55512345678901234567890');
} catch (PhoneNumberParseException $e) {
    $e->getErrorType(); // "TOO_LONG"
}
```

## Safe Parsing

```php
use Cline\PhoneNumber\PhoneNumber;

// Returns null instead of throwing
$phone = PhoneNumber::tryParse('invalid');
if ($phone === null) {
    // Handle invalid input
}

// With region
$phone = PhoneNumber::tryParse('555-123-4567', 'US');
```

## Validation Without Exceptions

```php
// Check validity without parsing
$isValid = PhoneNumber::isValid('+1 555 123 4567'); // true
$isValid = PhoneNumber::isValid('invalid'); // false

// Check if possible (less strict)
$isPossible = PhoneNumber::isPossible('+1 555 123 4567'); // true
```

## Custom Exception Handling

```php
use Cline\PhoneNumber\PhoneNumber;
use Cline\PhoneNumber\Exceptions\PhoneNumberParseException;
use Cline\PhoneNumber\Exceptions\InvalidRegionException;

function parseUserPhone(string $input, string $region): ?PhoneNumber
{
    try {
        $phone = PhoneNumber::parse($input, $region);

        if (!$phone->isValid()) {
            return null;
        }

        return $phone;
    } catch (PhoneNumberParseException $e) {
        logger()->warning('Invalid phone number', [
            'input' => $input,
            'error' => $e->getErrorType(),
        ]);
        return null;
    } catch (InvalidRegionException $e) {
        logger()->error('Invalid region code', [
            'region' => $region,
        ]);
        return null;
    }
}
```

## Laravel Validation

```php
use Cline\PhoneNumber\Rules\PhoneNumberRule;

// In form request
public function rules(): array
{
    return [
        'phone' => ['required', new PhoneNumberRule()],
    ];
}

// With region
public function rules(): array
{
    return [
        'phone' => ['required', new PhoneNumberRule('US')],
    ];
}

// With type restriction
public function rules(): array
{
    return [
        'phone' => ['required', new PhoneNumberRule(type: 'MOBILE')],
    ];
}
```

## Batch Processing

```php
use Cline\PhoneNumber\PhoneNumber;

$inputs = ['+1 555 123 4567', 'invalid', '+44 20 7946 0958'];
$results = [];

foreach ($inputs as $input) {
    $phone = PhoneNumber::tryParse($input);
    $results[] = [
        'input' => $input,
        'valid' => $phone !== null,
        'formatted' => $phone?->formatE164(),
    ];
}
```
