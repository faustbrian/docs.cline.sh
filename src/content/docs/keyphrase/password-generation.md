---
title: Password Generation
description: The PasswordGenerator creates secure random passwords with customizable character sets, lengths, and exclusions.
---

The `PasswordGenerator` creates secure random passwords with customizable character sets, lengths, and exclusions.

## Basic Usage

```php
use Cline\Keyphrase\Generators\PasswordGenerator;

// Using static factory method
$password = PasswordGenerator::create()->generate();

// Using constructor
$generator = new PasswordGenerator();
$password = $generator->generate();
```

## Setting Length

```php
// Default is 16 characters
$password = PasswordGenerator::create()->generate();

// Custom length
$password = PasswordGenerator::create()
    ->length(32)
    ->generate();
```

## Character Sets

### Default Character Sets

By default, passwords include:
- Lowercase letters (a-z)
- Uppercase letters (A-Z)
- Digits (0-9)

Symbols are **not** included by default.

```php
$generator = PasswordGenerator::create();

// Toggle character sets
$password = $generator
    ->withLowercase(true)    // Include a-z (default: true)
    ->withUppercase(true)    // Include A-Z (default: true)
    ->withDigits(true)       // Include 0-9 (default: true)
    ->withSymbols(true)      // Include !@#$%^&*... (default: false)
    ->generate();
```

### Disabling Character Sets

```php
// Uppercase only
$password = PasswordGenerator::create()
    ->withLowercase(false)
    ->withDigits(false)
    ->generate();

// Digits only
$password = PasswordGenerator::create()
    ->withLowercase(false)
    ->withUppercase(false)
    ->generate();
```

## Presets

Common configurations are available as presets:

### Alphanumeric

Letters and digits only (no symbols):

```php
$password = PasswordGenerator::create()
    ->alphanumeric()
    ->length(20)
    ->generate();

// Example: "Kj8mNpQ2xR5vLwY9sT3z"
```

### PIN

Digits only:

```php
$pin = PasswordGenerator::create()
    ->pin()
    ->length(6)
    ->generate();

// Example: "847291"
```

### Hexadecimal

Lowercase hex characters (0-9, a-f):

```php
$hex = PasswordGenerator::create()
    ->hex()
    ->length(32)
    ->generate();

// Example: "a1b2c3d4e5f67890abcdef1234567890"
```

## Customization

### Excluding Ambiguous Characters

Remove visually similar characters (0, O, 1, l, I):

```php
$password = PasswordGenerator::create()
    ->excludeAmbiguous()
    ->generate();

// Never contains: 0, O, 1, l, I
```

### Excluding Specific Characters

Remove any characters you want to exclude:

```php
$password = PasswordGenerator::create()
    ->exclude('aeiou')
    ->generate();

// Never contains vowels
```

### Adding Custom Characters

Add your own characters to the pool:

```php
$password = PasswordGenerator::create()
    ->withLowercase(false)
    ->withUppercase(false)
    ->withDigits(false)
    ->withCustomCharacters('ABC123!@#')
    ->length(10)
    ->generate();

// Only uses: A, B, C, 1, 2, 3, !, @, #
```

## Generating Multiple Passwords

```php
$passwords = PasswordGenerator::create()
    ->length(20)
    ->withSymbols()
    ->generateMany(10);

// Returns array of 10 unique passwords
```

## Entropy Calculation

Calculate the entropy (bits of randomness) for the current configuration:

```php
$generator = PasswordGenerator::create()
    ->length(16)
    ->alphanumeric();

$entropy = $generator->entropy();
// Returns: 95.27 bits

// More characters = more entropy
$entropy = PasswordGenerator::create()
    ->length(16)
    ->withSymbols()
    ->entropy();
// Returns: ~105 bits
```

## Full Example

```php
use Cline\Keyphrase\Generators\PasswordGenerator;

// Create a secure password generator with common settings
$generator = PasswordGenerator::create()
    ->length(24)
    ->withSymbols()
    ->excludeAmbiguous();

// Generate passwords
$password1 = $generator->generate();
$password2 = $generator->generate();

// Check entropy
$entropy = $generator->entropy();
echo "Password entropy: {$entropy} bits";

// Generate multiple
$passwords = $generator->generateMany(5);
```

## Security Recommendations

| Use Case | Minimum Length | Recommended Settings |
|----------|---------------|---------------------|
| PIN | 6 | `->pin()->length(6)` |
| Web Account | 16 | `->length(16)->alphanumeric()` |
| High Security | 24+ | `->length(24)->withSymbols()->excludeAmbiguous()` |
| API Key | 32+ | `->length(32)->alphanumeric()` |
| Master Password | 20+ | `->length(20)->withSymbols()` |
