# Getting Started

A fast, spec-compliant PHP implementation of JSON Schema validation supporting Draft 04, 06, 07, 2019-09, and 2020-12.

## Installation

Install via Composer:

```bash
composer require cline/json-schema
```

The package auto-registers with Laravel 11+. The facade and service provider are automatically configured.

## Requirements

- PHP 8.4+
- Laravel 11+ or 12+

## Quick Example

```php
use Cline\JsonSchema\Facades\JsonSchema;

$schema = [
    'type' => 'object',
    'properties' => [
        'name' => ['type' => 'string'],
        'age' => ['type' => 'integer', 'minimum' => 0],
    ],
    'required' => ['name'],
];

$data = ['name' => 'John', 'age' => 30];

$result = JsonSchema::validate($data, $schema);

if ($result->isValid()) {
    // Data is valid
} else {
    // Handle errors
    foreach ($result->errors() as $error) {
        echo $error->message();
    }
}
```

## Basic Validation

### Using the Facade

The simplest way to validate data:

```php
use Cline\JsonSchema\Facades\JsonSchema;

$result = JsonSchema::validate($data, $schema);
```

### Using Dependency Injection

Inject the manager into your classes:

```php
use Cline\JsonSchema\JsonSchemaManager;

class UserValidator
{
    public function __construct(
        private JsonSchemaManager $jsonSchema
    ) {}

    public function validate(array $data): bool
    {
        $schema = ['type' => 'object', /* ... */];
        $result = $this->jsonSchema->validate($data, $schema);

        return $result->isValid();
    }
}
```

## Draft Versions

The library automatically detects the JSON Schema draft version from the `$schema` keyword:

```php
$schema = [
    '$schema' => 'https://json-schema.org/draft/2020-12/schema',
    'type' => 'string',
];

// Automatically uses Draft 2020-12 validator
$result = JsonSchema::validate('hello', $schema);
```

### Explicit Draft Selection

Force a specific draft version:

```php
use Cline\JsonSchema\Enums\Draft;

$result = JsonSchema::validate($data, $schema, Draft::Draft07);
```

Available draft versions:

- `Draft::Draft04`
- `Draft::Draft06`
- `Draft::Draft07`
- `Draft::Draft201909`
- `Draft::Draft202012`

## Validation Results

Every validation returns a `ValidationResult` object:

```php
$result = JsonSchema::validate($data, $schema);

// Check validity
if ($result->isValid()) {
    // Valid data
}

if ($result->isInvalid()) {
    // Invalid data
}

// Access errors
foreach ($result->errors() as $error) {
    echo $error->message();    // Error message
    echo $error->path();       // JSON pointer to error location
    echo $error->keyword();    // Schema keyword that failed
}
```

## Common Validation Examples

### String Validation

```php
$schema = [
    'type' => 'string',
    'minLength' => 3,
    'maxLength' => 50,
    'pattern' => '^[A-Za-z]+$',  // Letters only
];

JsonSchema::validate('John', $schema);  // Valid
JsonSchema::validate('Jo', $schema);    // Invalid (too short)
JsonSchema::validate('John123', $schema);  // Invalid (pattern mismatch)
```

### Number Validation

```php
$schema = [
    'type' => 'number',
    'minimum' => 0,
    'maximum' => 100,
    'multipleOf' => 5,
];

JsonSchema::validate(50, $schema);   // Valid
JsonSchema::validate(53, $schema);   // Invalid (not multiple of 5)
JsonSchema::validate(-5, $schema);   // Invalid (below minimum)
```

### Array Validation

```php
$schema = [
    'type' => 'array',
    'items' => ['type' => 'string'],
    'minItems' => 1,
    'maxItems' => 5,
    'uniqueItems' => true,
];

JsonSchema::validate(['a', 'b', 'c'], $schema);  // Valid
JsonSchema::validate(['a', 'a'], $schema);       // Invalid (not unique)
JsonSchema::validate([], $schema);               // Invalid (too few items)
```

### Object Validation

```php
$schema = [
    'type' => 'object',
    'properties' => [
        'username' => [
            'type' => 'string',
            'minLength' => 3,
        ],
        'email' => [
            'type' => 'string',
            'format' => 'email',
        ],
        'age' => [
            'type' => 'integer',
            'minimum' => 18,
        ],
    ],
    'required' => ['username', 'email'],
    'additionalProperties' => false,
];

$validData = [
    'username' => 'john_doe',
    'email' => 'john@example.com',
    'age' => 25,
];

JsonSchema::validate($validData, $schema);  // Valid
```

### Enum Validation

```php
$schema = [
    'type' => 'string',
    'enum' => ['red', 'green', 'blue'],
];

JsonSchema::validate('red', $schema);     // Valid
JsonSchema::validate('yellow', $schema);  // Invalid
```

## Format Validation

Common string formats are supported:

```php
$schema = ['type' => 'string', 'format' => 'email'];
JsonSchema::validate('user@example.com', $schema);  // Valid

$schema = ['type' => 'string', 'format' => 'uuid'];
JsonSchema::validate('550e8400-e29b-41d4-a716-446655440000', $schema);  // Valid

$schema = ['type' => 'string', 'format' => 'uri'];
JsonSchema::validate('https://example.com', $schema);  // Valid
```

Supported formats:
- `email`
- `uri`
- `uri-reference`
- `uri-template`
- `uuid`
- `hostname`
- `ipv4`
- `ipv6`
- `date-time`
- `json-pointer`

## Next Steps

- [Validation Guide](/json-schema/validation) - Advanced validation techniques
- [Schema Builder](/json-schema/schema-builder) - Programmatic schema creation
- [API Reference](/json-schema/api-reference) - Complete API documentation
