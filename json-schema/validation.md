# Validation Guide

Advanced validation techniques and patterns for JSON Schema validation.

## Complex Schemas

### Nested Objects

Validate deeply nested data structures:

```php
$schema = [
    'type' => 'object',
    'properties' => [
        'user' => [
            'type' => 'object',
            'properties' => [
                'name' => ['type' => 'string'],
                'address' => [
                    'type' => 'object',
                    'properties' => [
                        'street' => ['type' => 'string'],
                        'city' => ['type' => 'string'],
                        'zipcode' => [
                            'type' => 'string',
                            'pattern' => '^\d{5}$',
                        ],
                    ],
                    'required' => ['street', 'city'],
                ],
            ],
            'required' => ['name'],
        ],
    ],
];

$data = [
    'user' => [
        'name' => 'John Doe',
        'address' => [
            'street' => '123 Main St',
            'city' => 'Springfield',
            'zipcode' => '12345',
        ],
    ],
];

$result = JsonSchema::validate($data, $schema);
```

### Arrays of Objects

Validate collections of structured data:

```php
$schema = [
    'type' => 'array',
    'items' => [
        'type' => 'object',
        'properties' => [
            'id' => ['type' => 'integer'],
            'name' => ['type' => 'string'],
            'active' => ['type' => 'boolean'],
        ],
        'required' => ['id', 'name'],
    ],
    'minItems' => 1,
];

$data = [
    ['id' => 1, 'name' => 'Product A', 'active' => true],
    ['id' => 2, 'name' => 'Product B', 'active' => false],
];

$result = JsonSchema::validate($data, $schema);
```

## Schema Composition

### AllOf (AND Logic)

Data must satisfy all subschemas:

```php
$schema = [
    'allOf' => [
        ['type' => 'object', 'properties' => ['name' => ['type' => 'string']]],
        ['properties' => ['age' => ['type' => 'integer', 'minimum' => 0]]],
        ['required' => ['name']],
    ],
];

// Valid - satisfies all schemas
$result = JsonSchema::validate(['name' => 'John', 'age' => 30], $schema);
```

### AnyOf (OR Logic)

Data must satisfy at least one subschema:

```php
$schema = [
    'anyOf' => [
        ['type' => 'string', 'minLength' => 5],
        ['type' => 'number', 'minimum' => 0],
    ],
];

$result = JsonSchema::validate('hello', $schema);  // Valid (first schema)
$result = JsonSchema::validate(42, $schema);       // Valid (second schema)
$result = JsonSchema::validate('hi', $schema);     // Invalid (neither schema)
```

### OneOf (XOR Logic)

Data must satisfy exactly one subschema:

```php
$schema = [
    'oneOf' => [
        ['type' => 'number', 'multipleOf' => 5],
        ['type' => 'number', 'multipleOf' => 3],
    ],
];

$result = JsonSchema::validate(10, $schema);  // Valid (only first schema)
$result = JsonSchema::validate(9, $schema);   // Valid (only second schema)
$result = JsonSchema::validate(15, $schema);  // Invalid (both schemas match)
$result = JsonSchema::validate(7, $schema);   // Invalid (no schemas match)
```

### Not

Data must NOT satisfy the subschema:

```php
$schema = [
    'type' => 'number',
    'not' => ['multipleOf' => 2],  // Not even
];

$result = JsonSchema::validate(5, $schema);   // Valid (odd number)
$result = JsonSchema::validate(4, $schema);   // Invalid (even number)
```

## Conditional Validation

### If-Then-Else

Apply schemas conditionally based on data:

```php
$schema = [
    'type' => 'object',
    'properties' => [
        'type' => ['type' => 'string'],
        'value' => [],
    ],
    'if' => [
        'properties' => ['type' => ['const' => 'number']],
    ],
    'then' => [
        'properties' => ['value' => ['type' => 'number']],
    ],
    'else' => [
        'properties' => ['value' => ['type' => 'string']],
    ],
];

// Valid - type is 'number', value is numeric
$result = JsonSchema::validate(['type' => 'number', 'value' => 42], $schema);

// Valid - type is not 'number', value is string
$result = JsonSchema::validate(['type' => 'text', 'value' => 'hello'], $schema);
```

## Schema References

### Internal References

Reuse schema definitions:

```php
$schema = [
    '$defs' => [
        'positiveInteger' => [
            'type' => 'integer',
            'minimum' => 1,
        ],
    ],
    'type' => 'object',
    'properties' => [
        'width' => ['$ref' => '#/$defs/positiveInteger'],
        'height' => ['$ref' => '#/$defs/positiveInteger'],
    ],
];

$result = JsonSchema::validate(['width' => 100, 'height' => 200], $schema);
```

### Recursive Schemas

Define self-referential structures:

```php
$schema = [
    '$defs' => [
        'node' => [
            'type' => 'object',
            'properties' => [
                'value' => ['type' => 'string'],
                'children' => [
                    'type' => 'array',
                    'items' => ['$ref' => '#/$defs/node'],
                ],
            ],
        ],
    ],
    '$ref' => '#/$defs/node',
];

$tree = [
    'value' => 'root',
    'children' => [
        ['value' => 'child1', 'children' => []],
        ['value' => 'child2', 'children' => [
            ['value' => 'grandchild', 'children' => []],
        ]],
    ],
];

$result = JsonSchema::validate($tree, $schema);
```

## Error Handling

### Accessing Validation Errors

```php
$result = JsonSchema::validate($data, $schema);

if ($result->isInvalid()) {
    foreach ($result->errors() as $error) {
        // Error message
        echo $error->message();

        // JSON pointer to error location
        echo $error->path();

        // Schema keyword that failed
        echo $error->keyword();

        // Additional context
        var_dump($error->context());
    }
}
```

### Error Paths

Errors include JSON pointers indicating the exact location:

```php
$schema = [
    'type' => 'object',
    'properties' => [
        'users' => [
            'type' => 'array',
            'items' => [
                'type' => 'object',
                'properties' => [
                    'age' => ['type' => 'integer', 'minimum' => 0],
                ],
            ],
        ],
    ],
];

$data = [
    'users' => [
        ['age' => 25],
        ['age' => -5],  // Invalid
    ],
];

$result = JsonSchema::validate($data, $schema);
// Error path: /users/1/age
```

### Custom Error Messages

Catch validation exceptions:

```php
use Cline\JsonSchema\Exceptions\ValidationException;

try {
    $result = JsonSchema::validate($data, $schema);

    if ($result->isInvalid()) {
        throw new ValidationException($result);
    }
} catch (ValidationException $e) {
    $errors = $e->getResult()->errors();
    // Handle errors
}
```

## Advanced Object Validation

### Property Dependencies

Require properties based on others:

```php
$schema = [
    'type' => 'object',
    'properties' => [
        'name' => ['type' => 'string'],
        'credit_card' => ['type' => 'string'],
        'billing_address' => ['type' => 'string'],
    ],
    'dependentRequired' => [
        'credit_card' => ['billing_address'],  // If credit_card exists, require billing_address
    ],
];
```

### Pattern Properties

Validate properties matching patterns:

```php
$schema = [
    'type' => 'object',
    'patternProperties' => [
        '^S_' => ['type' => 'string'],
        '^I_' => ['type' => 'integer'],
    ],
];

// Valid
$data = [
    'S_name' => 'John',
    'I_age' => 30,
];

$result = JsonSchema::validate($data, $schema);
```

### Property Names Validation

Validate property key names:

```php
$schema = [
    'type' => 'object',
    'propertyNames' => [
        'pattern' => '^[A-Za-z_][A-Za-z0-9_]*$',  // Valid identifier names
    ],
];

// Valid
$result = JsonSchema::validate(['valid_name' => 1, '_another' => 2], $schema);

// Invalid
$result = JsonSchema::validate(['invalid-name' => 1], $schema);
```

## Advanced Array Validation

### Prefix Items (Tuple Validation)

Validate array positions individually:

```php
$schema = [
    'type' => 'array',
    'prefixItems' => [
        ['type' => 'number'],    // First item must be number
        ['type' => 'string'],    // Second item must be string
        ['type' => 'boolean'],   // Third item must be boolean
    ],
];

// Valid
$result = JsonSchema::validate([42, 'hello', true], $schema);

// Invalid - wrong types
$result = JsonSchema::validate(['hello', 42, true], $schema);
```

### Contains

Require at least one array item to match:

```php
$schema = [
    'type' => 'array',
    'contains' => ['type' => 'number', 'minimum' => 10],
    'minContains' => 2,  // At least 2 items must match
];

// Valid - has 2 numbers >= 10
$result = JsonSchema::validate([5, 10, 15, 20], $schema);

// Invalid - only 1 number >= 10
$result = JsonSchema::validate([5, 10], $schema);
```

## String Format Validation

### Built-in Formats

```php
// Email validation
$schema = ['type' => 'string', 'format' => 'email'];
JsonSchema::validate('user@example.com', $schema);

// URI validation
$schema = ['type' => 'string', 'format' => 'uri'];
JsonSchema::validate('https://example.com/path', $schema);

// UUID validation
$schema = ['type' => 'string', 'format' => 'uuid'];
JsonSchema::validate('550e8400-e29b-41d4-a716-446655440000', $schema);

// IPv4 validation
$schema = ['type' => 'string', 'format' => 'ipv4'];
JsonSchema::validate('192.168.1.1', $schema);

// IPv6 validation
$schema = ['type' => 'string', 'format' => 'ipv6'];
JsonSchema::validate('2001:0db8:85a3:0000:0000:8a2e:0370:7334', $schema);

// Hostname validation
$schema = ['type' => 'string', 'format' => 'hostname'];
JsonSchema::validate('example.com', $schema);

// Date-time validation
$schema = ['type' => 'string', 'format' => 'date-time'];
JsonSchema::validate('2024-01-15T10:30:00Z', $schema);

// JSON Pointer validation
$schema = ['type' => 'string', 'format' => 'json-pointer'];
JsonSchema::validate('/path/to/property', $schema);
```

## Performance Tips

### Schema Reuse

Cache and reuse schema definitions:

```php
class SchemaRepository
{
    private array $schemas = [];

    public function getUserSchema(): array
    {
        return $this->schemas['user'] ??= [
            'type' => 'object',
            'properties' => [
                'name' => ['type' => 'string'],
                'email' => ['type' => 'string', 'format' => 'email'],
            ],
        ];
    }
}
```

### Early Validation

Validate early to fail fast:

```php
// Validate required fields first
$basicSchema = ['type' => 'object', 'required' => ['id', 'name']];
$result = JsonSchema::validate($data, $basicSchema);

if ($result->isInvalid()) {
    return $result;  // Fail fast
}

// Then validate full schema
$fullSchema = [/* complex schema */];
return JsonSchema::validate($data, $fullSchema);
```

## Next Steps

- [Schema Builder](/json-schema/schema-builder) - Build schemas programmatically
- [API Reference](/json-schema/api-reference) - Complete API documentation
