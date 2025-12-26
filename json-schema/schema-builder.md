# Schema Builder

Build JSON schemas programmatically using a fluent, chainable API.

## Introduction

The `SchemaBuilder` provides a convenient way to construct JSON schemas without manually building arrays. It offers type safety, IDE autocompletion, and prevents common mistakes.

## Basic Usage

### Creating a Schema

```php
use Cline\JsonSchema\Builders\SchemaBuilder;

$schema = SchemaBuilder::create()
    ->type(SchemaType::String)
    ->minLength(3)
    ->maxLength(50)
    ->build();

// Returns a Schema instance
// Use ->toArray() to get raw array
$schemaArray = $schema->toArray();
```

### Using Built Schemas

```php
use Cline\JsonSchema\Facades\JsonSchema;

$schema = SchemaBuilder::create()
    ->type(SchemaType::String)
    ->format(Format::Email)
    ->build();

$result = JsonSchema::validate('user@example.com', $schema->toArray());
```

## Type Constraints

### Single Type

```php
use Cline\JsonSchema\Enums\SchemaType;

$schema = SchemaBuilder::create()
    ->type(SchemaType::String)
    ->build();
```

Available types:
- `SchemaType::String`
- `SchemaType::Number`
- `SchemaType::Integer`
- `SchemaType::Object`
- `SchemaType::Array`
- `SchemaType::Boolean`
- `SchemaType::Null`

### Multiple Types

```php
$schema = SchemaBuilder::create()
    ->types([SchemaType::String, SchemaType::Null])
    ->build();
```

## String Validation

### Length Constraints

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::String)
    ->minLength(3)
    ->maxLength(50)
    ->build();
```

### Pattern Matching

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::String)
    ->pattern('^[A-Za-z]+$')  // Letters only
    ->build();
```

### Format Validation

```php
use Cline\JsonSchema\Enums\Format;

$schema = SchemaBuilder::create()
    ->type(SchemaType::String)
    ->format(Format::Email)
    ->build();
```

Available formats:
- `Format::Email`
- `Format::Uri`
- `Format::UriReference`
- `Format::UriTemplate`
- `Format::Uuid`
- `Format::Hostname`
- `Format::Ipv4`
- `Format::Ipv6`
- `Format::DateTime`
- `Format::JsonPointer`

## Number Validation

### Range Constraints

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::Number)
    ->minimum(0)
    ->maximum(100)
    ->build();
```

### Exclusive Ranges

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::Number)
    ->exclusiveMinimum(0)      // Greater than 0
    ->exclusiveMaximum(100)    // Less than 100
    ->build();
```

### Multiple Of

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::Number)
    ->multipleOf(5)  // Must be divisible by 5
    ->build();
```

## Object Validation

### Properties

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::Object)
    ->properties([
        'name' => ['type' => 'string'],
        'age' => ['type' => 'integer', 'minimum' => 0],
        'email' => ['type' => 'string', 'format' => 'email'],
    ])
    ->required(['name', 'email'])
    ->build();
```

### Nested Schema Builders

```php
$addressSchema = SchemaBuilder::create()
    ->type(SchemaType::Object)
    ->properties([
        'street' => ['type' => 'string'],
        'city' => ['type' => 'string'],
    ])
    ->build();

$userSchema = SchemaBuilder::create()
    ->type(SchemaType::Object)
    ->properties([
        'name' => ['type' => 'string'],
        'address' => $addressSchema,  // Pass Schema instance
    ])
    ->build();
```

### Additional Properties

```php
// Disallow additional properties
$schema = SchemaBuilder::create()
    ->type(SchemaType::Object)
    ->properties(['name' => ['type' => 'string']])
    ->additionalProperties(false)
    ->build();

// Allow any additional properties
$schema = SchemaBuilder::create()
    ->type(SchemaType::Object)
    ->properties(['name' => ['type' => 'string']])
    ->additionalProperties(true)
    ->build();

// Additional properties must match schema
$schema = SchemaBuilder::create()
    ->type(SchemaType::Object)
    ->properties(['name' => ['type' => 'string']])
    ->additionalProperties(['type' => 'string'])
    ->build();
```

### Property Count

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::Object)
    ->minProperties(1)
    ->maxProperties(10)
    ->build();
```

## Array Validation

### Items Schema

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::Array)
    ->items(['type' => 'string'])
    ->build();
```

### Length Constraints

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::Array)
    ->minItems(1)
    ->maxItems(10)
    ->build();
```

### Unique Items

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::Array)
    ->uniqueItems()
    ->build();
```

### Array of Objects

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::Array)
    ->items([
        'type' => 'object',
        'properties' => [
            'id' => ['type' => 'integer'],
            'name' => ['type' => 'string'],
        ],
        'required' => ['id', 'name'],
    ])
    ->minItems(1)
    ->build();
```

## Enum and Const

### Enum Values

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::String)
    ->enum(['red', 'green', 'blue'])
    ->build();
```

### Constant Value

```php
$schema = SchemaBuilder::create()
    ->const('fixed-value')
    ->build();
```

## Schema Metadata

### Title and Description

```php
$schema = SchemaBuilder::create()
    ->title('User Schema')
    ->description('Validates user registration data')
    ->type(SchemaType::Object)
    ->properties([
        'username' => ['type' => 'string'],
    ])
    ->build();
```

### Draft Version

```php
use Cline\JsonSchema\Enums\Draft;

$schema = SchemaBuilder::create()
    ->draft(Draft::Draft202012)
    ->type(SchemaType::String)
    ->build();
```

## Schema References

### Internal References

```php
$schema = SchemaBuilder::create()
    ->ref('#/$defs/user')
    ->build();

// Use in larger schema
$fullSchema = [
    '$defs' => [
        'user' => [
            'type' => 'object',
            'properties' => ['name' => ['type' => 'string']],
        ],
    ],
    'type' => 'object',
    'properties' => [
        'author' => $schema->toArray(),
    ],
];
```

## Complex Examples

### User Registration Schema

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::Object)
    ->title('User Registration')
    ->description('Validates new user registration data')
    ->properties([
        'username' => [
            'type' => 'string',
            'minLength' => 3,
            'maxLength' => 20,
            'pattern' => '^[a-zA-Z0-9_]+$',
        ],
        'email' => [
            'type' => 'string',
            'format' => 'email',
        ],
        'password' => [
            'type' => 'string',
            'minLength' => 8,
        ],
        'age' => [
            'type' => 'integer',
            'minimum' => 18,
            'maximum' => 120,
        ],
        'terms' => [
            'type' => 'boolean',
            'const' => true,  // Must accept terms
        ],
    ])
    ->required(['username', 'email', 'password', 'terms'])
    ->additionalProperties(false)
    ->build();
```

### Product Catalog Schema

```php
$productSchema = SchemaBuilder::create()
    ->type(SchemaType::Object)
    ->properties([
        'id' => ['type' => 'integer', 'minimum' => 1],
        'name' => ['type' => 'string', 'minLength' => 1],
        'price' => [
            'type' => 'number',
            'minimum' => 0,
            'multipleOf' => 0.01,  // Cents precision
        ],
        'tags' => [
            'type' => 'array',
            'items' => ['type' => 'string'],
            'uniqueItems' => true,
        ],
        'inStock' => ['type' => 'boolean'],
    ])
    ->required(['id', 'name', 'price'])
    ->build();

$catalogSchema = SchemaBuilder::create()
    ->type(SchemaType::Array)
    ->items($productSchema)
    ->minItems(1)
    ->build();
```

### API Response Schema

```php
$successSchema = SchemaBuilder::create()
    ->type(SchemaType::Object)
    ->properties([
        'status' => ['type' => 'string', 'const' => 'success'],
        'data' => ['type' => 'object'],
        'timestamp' => ['type' => 'string', 'format' => 'date-time'],
    ])
    ->required(['status', 'data'])
    ->build();

$errorSchema = SchemaBuilder::create()
    ->type(SchemaType::Object)
    ->properties([
        'status' => ['type' => 'string', 'const' => 'error'],
        'error' => [
            'type' => 'object',
            'properties' => [
                'code' => ['type' => 'integer'],
                'message' => ['type' => 'string'],
            ],
            'required' => ['code', 'message'],
        ],
    ])
    ->required(['status', 'error'])
    ->build();
```

### Configuration Schema

```php
$configSchema = SchemaBuilder::create()
    ->type(SchemaType::Object)
    ->title('Application Configuration')
    ->properties([
        'app' => [
            'type' => 'object',
            'properties' => [
                'name' => ['type' => 'string'],
                'debug' => ['type' => 'boolean'],
                'url' => ['type' => 'string', 'format' => 'uri'],
            ],
            'required' => ['name', 'url'],
        ],
        'database' => [
            'type' => 'object',
            'properties' => [
                'host' => ['type' => 'string', 'format' => 'hostname'],
                'port' => ['type' => 'integer', 'minimum' => 1, 'maximum' => 65535],
                'name' => ['type' => 'string'],
            ],
            'required' => ['host', 'name'],
        ],
        'cache' => [
            'type' => 'object',
            'properties' => [
                'driver' => ['type' => 'string', 'enum' => ['redis', 'memcached', 'file']],
                'ttl' => ['type' => 'integer', 'minimum' => 0],
            ],
        ],
    ])
    ->required(['app', 'database'])
    ->build();
```

## Best Practices

### Reusable Schema Components

```php
class SchemaFactory
{
    public static function emailField(): array
    {
        return SchemaBuilder::create()
            ->type(SchemaType::String)
            ->format(Format::Email)
            ->toArray();
    }

    public static function positiveInteger(): array
    {
        return SchemaBuilder::create()
            ->type(SchemaType::Integer)
            ->minimum(1)
            ->toArray();
    }

    public static function userSchema(): Schema
    {
        return SchemaBuilder::create()
            ->type(SchemaType::Object)
            ->properties([
                'email' => self::emailField(),
                'id' => self::positiveInteger(),
            ])
            ->build();
    }
}
```

### Method Chaining

```php
$schema = SchemaBuilder::create()
    ->type(SchemaType::String)
    ->minLength(1)
    ->maxLength(100)
    ->pattern('^[A-Z]')  // Must start with capital
    ->build();
```

### Getting Raw Arrays

```php
// Build and get array directly
$array = SchemaBuilder::create()
    ->type(SchemaType::String)
    ->toArray();  // Don't call build()

// Or build then convert
$schema = SchemaBuilder::create()
    ->type(SchemaType::String)
    ->build();
$array = $schema->toArray();
```

## Next Steps

- [Getting Started](/json-schema/getting-started) - Basic usage
- [Validation Guide](/json-schema/validation) - Advanced validation
- [API Reference](/json-schema/api-reference) - Complete API documentation
