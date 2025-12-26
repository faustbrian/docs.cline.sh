# JSON Schema Integration

Using ECMA-262 Regex for JSON Schema pattern validation.

## Why Use This Package for JSON Schema?

JSON Schema uses ECMA-262 (JavaScript) regular expressions, while PHP natively uses PCRE. These engines have subtle differences that can cause validation inconsistencies between JavaScript and PHP validators.

This package provides 100% ECMA-262 compliance, ensuring your PHP JSON Schema validators behave identically to JavaScript validators.

## Quick Example

```php
use Cline\EcmaRegex\Facades\EcmaRegex;

// JSON Schema pattern validation
$schema = [
    'type' => 'string',
    'pattern' => '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
];

$email = 'user@example.com';
$isValid = EcmaRegex::test('/' . $schema['pattern'] . '/', $email);
// true - pattern matches
```

## Common JSON Schema Patterns

### Email Validation

```php
$pattern = '/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/';
EcmaRegex::test($pattern, 'user@example.com');  // true
EcmaRegex::test($pattern, 'invalid@');          // false
```

### URL/URI Validation

```php
// Basic HTTP(S) URL
$pattern = '/^https?:\/\/.+/';
EcmaRegex::test($pattern, 'https://example.com');  // true

// More strict URI validation
$pattern = '/^https?:\/\/[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(\/.*)?$/';
EcmaRegex::test($pattern, 'https://api.example.com/v1/users');  // true
```

### Date and Time Patterns

**Date (YYYY-MM-DD):**
```php
$pattern = '/^\d{4}-\d{2}-\d{2}$/';
EcmaRegex::test($pattern, '2024-12-24');  // true
```

**Time (HH:MM:SS):**
```php
$pattern = '/^([01]\d|2[0-3]):[0-5]\d:[0-5]\d$/';
EcmaRegex::test($pattern, '14:30:00');  // true
```

**DateTime (ISO 8601):**
```php
$pattern = '/^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d{3})?Z?$/';
EcmaRegex::test($pattern, '2024-12-24T14:30:00.000Z');  // true
```

### Numeric Patterns

**Integer:**
```php
$pattern = '/^-?\d+$/';
EcmaRegex::test($pattern, '123');   // true
EcmaRegex::test($pattern, '-456');  // true
```

**Decimal:**
```php
$pattern = '/^-?\d+\.\d+$/';
EcmaRegex::test($pattern, '123.45');   // true
EcmaRegex::test($pattern, '-67.89');   // true
```

**Phone Numbers:**
```php
// US format
$pattern = '/^\d{3}-\d{3}-\d{4}$/';
EcmaRegex::test($pattern, '555-123-4567');  // true

// International format
$pattern = '/^\+\d{1,3}\s?\d{4,14}$/';
EcmaRegex::test($pattern, '+1 5551234567');  // true
```

### String Format Patterns

**Alphanumeric:**
```php
$pattern = '/^[a-zA-Z0-9]+$/';
EcmaRegex::test($pattern, 'Test123');  // true
```

**Slug:**
```php
$pattern = '/^[a-z0-9]+(?:-[a-z0-9]+)*$/';
EcmaRegex::test($pattern, 'my-blog-post');  // true
```

**Username:**
```php
$pattern = '/^[a-zA-Z0-9_-]{3,16}$/';
EcmaRegex::test($pattern, 'user_name-123');  // true
```

**Hexadecimal Color:**
```php
$pattern = '/^#[0-9A-Fa-f]{6}$/';
EcmaRegex::test($pattern, '#FF5733');  // true
```

## Integration with JSON Schema Validators

### Custom Validator Implementation

```php
use Cline\EcmaRegex\Facades\EcmaRegex;

class JsonSchemaValidator
{
    public function validatePattern(string $pattern, mixed $value): bool
    {
        if (!is_string($value)) {
            return false;
        }

        // Wrap pattern with delimiters if not present
        $regexPattern = $this->ensureDelimiters($pattern);

        return EcmaRegex::test($regexPattern, $value);
    }

    private function ensureDelimiters(string $pattern): string
    {
        if (!str_starts_with($pattern, '/')) {
            $pattern = '/' . $pattern . '/';
        }

        return $pattern;
    }
}
```

### Usage in Validation

```php
$validator = new JsonSchemaValidator();

$schemas = [
    'email' => '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$',
    'phone' => '^\d{3}-\d{3}-\d{4}$',
    'date' => '^\d{4}-\d{2}-\d{2}$',
];

$data = [
    'email' => 'user@example.com',
    'phone' => '555-123-4567',
    'date' => '2024-12-24',
];

foreach ($schemas as $field => $pattern) {
    $isValid = $validator->validatePattern($pattern, $data[$field]);
    echo "$field: " . ($isValid ? 'valid' : 'invalid') . "\n";
}
```

## Working with JSON Schema Objects

### Validating Nested Patterns

```php
$schema = [
    'type' => 'object',
    'properties' => [
        'username' => [
            'type' => 'string',
            'pattern' => '^[a-zA-Z0-9_-]{3,16}$'
        ],
        'email' => [
            'type' => 'string',
            'pattern' => '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        ],
    ],
];

$data = [
    'username' => 'john_doe',
    'email' => 'john@example.com',
];

foreach ($schema['properties'] as $field => $rules) {
    if (isset($rules['pattern'])) {
        $pattern = '/' . $rules['pattern'] . '/';
        $isValid = EcmaRegex::test($pattern, $data[$field]);
        echo "$field: " . ($isValid ? '✓' : '✗') . "\n";
    }
}
```

### Array Item Validation

```php
$schema = [
    'type' => 'array',
    'items' => [
        'type' => 'string',
        'pattern' => '^#[0-9A-Fa-f]{6}$'  // Hex colors
    ],
];

$colors = ['#FF5733', '#C70039', '#900C3F'];

foreach ($colors as $color) {
    $pattern = '/' . $schema['items']['pattern'] . '/';
    $isValid = EcmaRegex::test($pattern, $color);

    if (!$isValid) {
        echo "Invalid color: $color\n";
    }
}
```

## Pattern Caching for Performance

When validating large datasets, compile patterns once:

```php
use Cline\EcmaRegex\Facades\EcmaRegex;

$schema = [
    'email' => '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$',
    'phone' => '^\d{3}-\d{3}-\d{4}$',
];

// Compile patterns once
$patterns = [];
foreach ($schema as $field => $pattern) {
    $patterns[$field] = EcmaRegex::compile('/' . $pattern . '/');
}

// Validate multiple records efficiently
$records = [
    ['email' => 'user1@example.com', 'phone' => '555-111-2222'],
    ['email' => 'user2@example.com', 'phone' => '555-333-4444'],
    // ... thousands more
];

foreach ($records as $record) {
    foreach ($patterns as $field => $pattern) {
        if (!$pattern->test($record[$field])) {
            echo "Invalid $field: {$record[$field]}\n";
        }
    }
}
```

## ECMA-262 vs PCRE Differences

Key differences this package handles for you:

### Lookahead/Lookbehind Syntax

**ECMA-262:**
```php
EcmaRegex::test('/(?<=@)\w+/', '@test');  // positive lookbehind
EcmaRegex::test('/(?<!@)\w+/', 'test');   // negative lookbehind
```

**PCRE:**
```php
// Uses same syntax but different engine behavior
```

### Unicode Handling

**ECMA-262:**
```php
$pattern = EcmaRegex::compile('/\w+/', 'u');
$pattern->test('café');  // true
```

**PCRE:**
```php
// Requires /u modifier but handles differently
```

### Character Classes

Both engines support character classes, but edge cases differ:

```php
// Negated class at start
EcmaRegex::test('/[^a-z]/', 'A');  // true - ECMA-262 compliant
```

## Best Practices

### 1. Always Use ECMA Regex for Schema Validation

```php
// ✓ Good - ECMA-262 compliant
use Cline\EcmaRegex\Facades\EcmaRegex;

EcmaRegex::test('/^[a-z]+$/', 'hello');

// ✗ Avoid - PCRE (may differ from JavaScript)
preg_match('/^[a-z]+$/', 'hello');
```

### 2. Wrap Patterns in Delimiters

```php
// JSON Schema pattern (no delimiters)
$schemaPattern = '^[a-z]+$';

// Add delimiters for this package
$regexPattern = '/' . $schemaPattern . '/';
EcmaRegex::test($regexPattern, 'hello');
```

### 3. Cache Compiled Patterns

```php
// Compile once for repeated use
$pattern = EcmaRegex::compile('/^[a-z]+$/');

foreach ($values as $value) {
    if ($pattern->test($value)) {
        // Process valid value
    }
}
```

### 4. Handle Exceptions Gracefully

```php
use Cline\EcmaRegex\Exceptions\EcmaRegexException;

try {
    $isValid = EcmaRegex::test($pattern, $value);
} catch (EcmaRegexException $e) {
    // Invalid pattern in schema
    throw new SchemaValidationException(
        "Invalid regex pattern: {$e->getMessage()}"
    );
}
```

## Real-World Example

Complete JSON Schema validator:

```php
use Cline\EcmaRegex\Facades\EcmaRegex;
use Cline\EcmaRegex\Exceptions\EcmaRegexException;

class SchemaValidator
{
    private array $compiledPatterns = [];

    public function validate(array $schema, array $data): array
    {
        $errors = [];

        foreach ($schema['properties'] ?? [] as $field => $rules) {
            if (!isset($data[$field])) {
                if ($rules['required'] ?? false) {
                    $errors[$field] = 'Field is required';
                }
                continue;
            }

            if (isset($rules['pattern'])) {
                $pattern = $this->getCompiledPattern($rules['pattern']);

                try {
                    if (!$pattern->test($data[$field])) {
                        $errors[$field] = 'Does not match required pattern';
                    }
                } catch (EcmaRegexException $e) {
                    $errors[$field] = "Invalid pattern: {$e->getMessage()}";
                }
            }
        }

        return $errors;
    }

    private function getCompiledPattern(string $pattern)
    {
        if (!isset($this->compiledPatterns[$pattern])) {
            $regexPattern = '/' . $pattern . '/';
            $this->compiledPatterns[$pattern] = EcmaRegex::compile($regexPattern);
        }

        return $this->compiledPatterns[$pattern];
    }
}

// Usage
$validator = new SchemaValidator();

$schema = [
    'properties' => [
        'email' => [
            'pattern' => '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$',
            'required' => true,
        ],
        'age' => [
            'pattern' => '^\d{1,3}$',
        ],
    ],
];

$data = [
    'email' => 'user@example.com',
    'age' => '25',
];

$errors = $validator->validate($schema, $data);

if (empty($errors)) {
    echo "Validation passed!\n";
} else {
    print_r($errors);
}
```
