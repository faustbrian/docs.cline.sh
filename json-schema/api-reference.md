# API Reference

Complete API documentation for the JSON Schema validation library.

## JsonSchemaManager

The main entry point for validation operations.

### Methods

#### `validate(mixed $data, array $schema, ?Draft $draft = null): ValidationResult`

Validates data against a JSON schema.

**Parameters:**
- `$data` (mixed) - The data to validate (any JSON-compatible type)
- `$schema` (array) - The JSON schema definition
- `$draft` (?Draft) - Optional explicit draft version (auto-detected if null)

**Returns:** `ValidationResult` containing validation status and errors

**Example:**
```php
use Cline\JsonSchema\JsonSchemaManager;

$manager = new JsonSchemaManager(new ValidatorFactory());
$result = $manager->validate(['name' => 'John'], ['type' => 'object']);
```

## JsonSchema Facade

Laravel facade providing static access to `JsonSchemaManager`.

### Methods

#### `JsonSchema::validate(mixed $data, array $schema, ?Draft $draft = null): ValidationResult`

Static validation method.

**Example:**
```php
use Cline\JsonSchema\Facades\JsonSchema;

$result = JsonSchema::validate($data, $schema);
```

## ValidationResult

Immutable value object representing validation results.

### Properties

- `bool $valid` - Whether validation passed
- `array $errors` - Array of `ValidationError` objects

### Methods

#### `isValid(): bool`

Check if validation passed.

**Returns:** `bool`

**Example:**
```php
if ($result->isValid()) {
    // Data is valid
}
```

#### `isInvalid(): bool`

Check if validation failed.

**Returns:** `bool`

**Example:**
```php
if ($result->isInvalid()) {
    // Handle validation errors
}
```

#### `errors(): array`

Get all validation errors.

**Returns:** `array<ValidationError>`

**Example:**
```php
foreach ($result->errors() as $error) {
    echo $error->message();
}
```

#### `getErrors(): array`

Alias for `errors()`.

**Returns:** `array<ValidationError>`

#### `toArray(): array`

Convert result to array representation.

**Returns:** `array{valid: bool, errors: array}`

**Example:**
```php
$array = $result->toArray();
// ['valid' => false, 'errors' => [...]]
```

### Factory Methods

#### `ValidationResult::success(): ValidationResult`

Create a successful validation result.

**Example:**
```php
$result = ValidationResult::success();
```

#### `ValidationResult::failure(array $errors): ValidationResult`

Create a failed validation result.

**Parameters:**
- `$errors` (array<ValidationError>) - Validation errors

**Example:**
```php
$result = ValidationResult::failure($errors);
```

## ValidationError

Represents a single validation error.

### Properties

- `string $path` - JSON pointer to error location
- `string $message` - Human-readable error message
- `string $keyword` - Schema keyword that failed
- `array $context` - Additional error context

### Methods

#### `path(): string`

Get the JSON pointer to the error location.

**Returns:** `string`

**Example:**
```php
$path = $error->path();  // "/users/0/email"
```

#### `message(): string`

Get the error message.

**Returns:** `string`

**Example:**
```php
$message = $error->message();  // "Value must be a valid email"
```

#### `keyword(): string`

Get the schema keyword that failed.

**Returns:** `string`

**Example:**
```php
$keyword = $error->keyword();  // "format"
```

#### `context(): array`

Get additional error context.

**Returns:** `array`

**Example:**
```php
$context = $error->context();
```

#### `toArray(): array`

Convert error to array representation.

**Returns:** `array{path: string, message: string, keyword: string}`

## SchemaBuilder

Fluent builder for constructing JSON schemas programmatically.

### Factory Methods

#### `SchemaBuilder::create(): SchemaBuilder`

Create a new schema builder instance.

**Example:**
```php
$builder = SchemaBuilder::create();
```

### Type Methods

#### `type(SchemaType $type): self`

Set the schema type.

**Parameters:**
- `$type` (SchemaType) - The schema type

**Example:**
```php
$builder->type(SchemaType::String);
```

#### `types(array $types): self`

Set multiple allowed types.

**Parameters:**
- `$types` (array<SchemaType>) - Array of allowed types

**Example:**
```php
$builder->types([SchemaType::String, SchemaType::Null]);
```

### String Methods

#### `minLength(int $minLength): self`

Set minimum string length.

#### `maxLength(int $maxLength): self`

Set maximum string length.

#### `pattern(string $pattern): self`

Set regex pattern constraint.

**Parameters:**
- `$pattern` (string) - Regex pattern (without delimiters)

#### `format(Format $format): self`

Set format constraint.

**Parameters:**
- `$format` (Format) - Format enum value

**Example:**
```php
$builder->format(Format::Email);
```

### Number Methods

#### `minimum(float|int $minimum): self`

Set inclusive minimum value.

#### `maximum(float|int $maximum): self`

Set inclusive maximum value.

#### `exclusiveMinimum(float|int $exclusiveMinimum): self`

Set exclusive minimum value.

#### `exclusiveMaximum(float|int $exclusiveMaximum): self`

Set exclusive maximum value.

#### `multipleOf(float|int $multipleOf): self`

Set multipleOf constraint.

### Object Methods

#### `properties(array $properties): self`

Set object properties.

**Parameters:**
- `$properties` (array<string, array|Schema>) - Property schemas

**Example:**
```php
$builder->properties([
    'name' => ['type' => 'string'],
    'age' => ['type' => 'integer'],
]);
```

#### `required(array $required): self`

Set required properties.

**Parameters:**
- `$required` (array<string>) - Required property names

#### `additionalProperties(array|bool $additionalProperties): self`

Set additional properties constraint.

**Parameters:**
- `$additionalProperties` (array|bool) - Schema or boolean

#### `minProperties(int $minProperties): self`

Set minimum property count.

#### `maxProperties(int $maxProperties): self`

Set maximum property count.

### Array Methods

#### `items(array|Schema $items): self`

Set array items schema.

**Parameters:**
- `$items` (array|Schema) - Items schema

#### `minItems(int $minItems): self`

Set minimum array length.

#### `maxItems(int $maxItems): self`

Set maximum array length.

#### `uniqueItems(bool $uniqueItems = true): self`

Set unique items constraint.

### Metadata Methods

#### `title(string $title): self`

Set schema title.

#### `description(string $description): self`

Set schema description.

#### `draft(Draft $draft): self`

Set JSON Schema draft version.

**Example:**
```php
$builder->draft(Draft::Draft202012);
```

### Constraint Methods

#### `enum(array $values): self`

Set allowed enum values.

**Parameters:**
- `$values` (array<mixed>) - Allowed values

#### `const(mixed $value): self`

Set constant value.

**Parameters:**
- `$value` (mixed) - Required constant value

#### `ref(string $ref): self`

Set schema reference.

**Parameters:**
- `$ref` (string) - JSON pointer or URI reference

### Build Methods

#### `build(): Schema`

Build and return Schema instance.

**Returns:** `Schema`

**Example:**
```php
$schema = $builder->build();
```

#### `toArray(): array`

Get schema as array without building.

**Returns:** `array<string, mixed>`

**Example:**
```php
$array = $builder->toArray();
```

## Schema

Immutable value object representing a JSON schema.

### Methods

#### `toArray(): array`

Get schema as array.

**Returns:** `array<string, mixed>`

**Example:**
```php
$schemaArray = $schema->toArray();
```

## Draft Enum

Represents JSON Schema draft versions.

### Cases

- `Draft::Draft04` - JSON Schema Draft 04
- `Draft::Draft06` - JSON Schema Draft 06
- `Draft::Draft07` - JSON Schema Draft 07
- `Draft::Draft201909` - JSON Schema 2019-09
- `Draft::Draft202012` - JSON Schema 2020-12 (latest)

### Methods

#### `fromSchemaUri(string $schemaUri): ?self`

Get draft from schema URI.

**Parameters:**
- `$schemaUri` (string) - The $schema URI

**Returns:** `?Draft` (null if not recognized)

**Example:**
```php
$draft = Draft::fromSchemaUri('https://json-schema.org/draft/2020-12/schema');
```

#### `label(): string`

Get human-readable label.

**Returns:** `string`

**Example:**
```php
$label = Draft::Draft202012->label();  // "Draft 2020-12"
```

## SchemaType Enum

Represents JSON Schema primitive types.

### Cases

- `SchemaType::String` - String type
- `SchemaType::Number` - Number type (integer or float)
- `SchemaType::Integer` - Integer type
- `SchemaType::Object` - Object type
- `SchemaType::Array` - Array type
- `SchemaType::Boolean` - Boolean type
- `SchemaType::Null` - Null type

## Format Enum

Represents JSON Schema string formats.

### Cases

- `Format::Email` - Email address
- `Format::Uri` - URI
- `Format::UriReference` - URI reference
- `Format::UriTemplate` - URI template
- `Format::Uuid` - UUID
- `Format::Hostname` - Hostname
- `Format::Ipv4` - IPv4 address
- `Format::Ipv6` - IPv6 address
- `Format::DateTime` - ISO 8601 date-time
- `Format::JsonPointer` - JSON Pointer

## ValidatorFactory

Factory for creating draft-specific validators.

### Methods

#### `create(Draft $draft): ValidatorInterface`

Create validator for specific draft.

**Parameters:**
- `$draft` (Draft) - Draft version

**Returns:** `ValidatorInterface`

**Example:**
```php
$factory = new ValidatorFactory();
$validator = $factory->create(Draft::Draft07);
```

## Exceptions

### JsonSchemaException

Base exception for all package exceptions.

### ValidationException

Thrown when validation fails.

**Methods:**
- `getResult(): ValidationResult` - Get validation result

**Example:**
```php
use Cline\JsonSchema\Exceptions\ValidationException;

try {
    $result = JsonSchema::validate($data, $schema);
    if ($result->isInvalid()) {
        throw new ValidationException($result);
    }
} catch (ValidationException $e) {
    $errors = $e->getResult()->errors();
}
```

### InvalidSchemaException

Thrown when schema is malformed.

### InvalidJsonSchemaException

Thrown when JSON schema syntax is invalid.

### DraftNotSupportedException

Thrown when draft version is unsupported.

### CannotResolveReferenceException

Thrown when schema reference cannot be resolved.

### UnresolvedReferenceException

Thrown when reference resolution fails.

### MissingSchemaKeywordException

Thrown when required schema keyword is missing.

### InvalidJsonPointerException

Thrown when JSON pointer is malformed.

### DraftCannotBeDetectedException

Thrown when draft version cannot be detected.

## Service Provider

### JsonSchemaServiceProvider

Registers package services in Laravel.

**Registered Services:**
- `ValidatorFactory` (singleton)
- `JsonSchemaManager` (singleton)

**Facade:**
- `JsonSchema` → `JsonSchemaManager`

## Testing Support

The package is tested against the official JSON Schema Test Suite for 100% spec compliance.

**Available Test Scripts:**
```bash
composer test                # Run all tests
composer test:unit          # Unit tests
composer test:draft-04      # Draft 04 compliance
composer test:draft-06      # Draft 06 compliance
composer test:draft-07      # Draft 07 compliance
composer test:draft-2019-09 # Draft 2019-09 compliance
composer test:draft-2020-12 # Draft 2020-12 compliance
```

## Next Steps

- [Getting Started](/json-schema/getting-started) - Installation and basic usage
- [Validation Guide](/json-schema/validation) - Advanced validation techniques
- [Schema Builder](/json-schema/schema-builder) - Programmatic schema construction
