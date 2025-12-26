# Getting Started

ECMA-262 Regex for PHP provides a 100% ECMA-262 compliant regular expression engine designed for JSON Schema validation and full JavaScript regex compatibility.

## Why This Package?

PHP uses PCRE (Perl Compatible Regular Expressions), while JSON Schema and JavaScript use ECMA-262 regex syntax. These engines have subtle differences that can cause validation inconsistencies. This package provides a pure PHP implementation of ECMA-262 regex, ensuring perfect compatibility with JSON Schema patterns.

## Installation

Install via Composer:

```bash
composer require cline/ecma-regex
```

The package will auto-register with Laravel's service container.

### Requirements

- PHP ^8.5.0

## Configuration

Publish the configuration file (optional):

```bash
php artisan vendor:publish --tag="ecma-regex-config"
```

This creates `config/ecma-regex.php`:

```php
return [
    // Enable pattern caching for performance
    'cache_enabled' => env('ECMA_REGEX_CACHE_ENABLED', true),

    // Cache TTL in seconds (default: 1 hour)
    'cache_ttl' => env('ECMA_REGEX_CACHE_TTL', 3600),
];
```

## Basic Usage

### Quick Pattern Testing

The simplest way to test a pattern:

```php
use Cline\EcmaRegex\Facades\EcmaRegex;

// Test if pattern matches
$isValid = EcmaRegex::test('/^[a-z]+$/', 'hello');  // true
$isValid = EcmaRegex::test('/^[a-z]+$/', 'Hello');  // false
```

### Getting Match Details

Get full match information:

```php
$result = EcmaRegex::match('/h(ello)/', 'hello world');

if ($result->matched) {
    echo $result->match;      // 'hello'
    echo $result->index;      // 0
    print_r($result->captures); // ['ello']
    echo $result->input;      // 'hello world'
}
```

### Finding All Matches

Find all pattern occurrences:

```php
$matches = EcmaRegex::matchAll('/\d+/', '123 abc 456');

foreach ($matches as $match) {
    echo "{$match->match} at position {$match->index}\n";
}
// Output:
// 123 at position 0
// 456 at position 8
```

### Compiling Patterns for Reuse

Compile patterns once and reuse them:

```php
$pattern = EcmaRegex::compile('/[a-z]+/i');

$pattern->test('Hello');      // true
$pattern->test('WORLD');      // true

$result = $pattern->exec('Hello World');
$matches = $pattern->matchAll('Hello World');
```

## Common Patterns

### Email Validation

```php
$emailPattern = '/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/';
EcmaRegex::test($emailPattern, 'user@example.com');  // true
```

### URL Validation

```php
$urlPattern = '/^https?:\/\/.+/';
EcmaRegex::test($urlPattern, 'https://example.com');  // true
```

### Date Validation (YYYY-MM-DD)

```php
$datePattern = '/^\d{4}-\d{2}-\d{2}$/';
EcmaRegex::test($datePattern, '2024-12-24');  // true
```

### Phone Number

```php
$phonePattern = '/^\d{3}-\d{3}-\d{4}$/';
EcmaRegex::test($phonePattern, '555-123-4567');  // true
```

## Next Steps

- [Pattern Syntax](/ecma-regex/pattern-syntax/) - Learn all supported pattern features
- [API Reference](/ecma-regex/api-reference/) - Complete API documentation
- [JSON Schema Integration](/ecma-regex/json-schema/) - Using with JSON Schema validators
- [Performance](/ecma-regex/performance/) - Caching and optimization tips
