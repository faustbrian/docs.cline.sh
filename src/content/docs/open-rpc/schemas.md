---
title: Schemas
description: Reusable JSON Schema definitions for OpenRPC specifications
---

Schemas provide reusable JSON Schema definitions that can be referenced throughout your OpenRPC specification using `$ref` pointers. This package includes pre-built schemas for common patterns.

## CursorPaginatorSchema

Defines the structure for cursor-based pagination parameters. Used in conjunction with `CursorPaginatorContentDescriptor`.

```php
use Cline\OpenRpc\Schema\CursorPaginatorSchema;

$schema = CursorPaginatorSchema::create();

// Result:
// [
//     'name' => 'CursorPaginator',
//     'data' => [
//         'type' => 'object',
//         'required' => ['cursor'],
//         'properties' => [
//             'cursor' => [
//                 'type' => 'string',
//                 'description' => 'The cursor to start from...',
//             ],
//             'size' => [
//                 'type' => 'integer',
//                 'description' => 'The number of items to return per page...',
//             ],
//         ],
//     ],
// ]
```

### Adding to Components

```php
use Cline\OpenRpc\ValueObject\ComponentsValue;
use Cline\OpenRpc\ValueObject\SchemaValue;
use Cline\OpenRpc\Schema\CursorPaginatorSchema;

$cursorSchema = CursorPaginatorSchema::create();

$components = new ComponentsValue(
    contentDescriptors: null,
    schemas: [
        'CursorPaginator' => new SchemaValue(
            name: $cursorSchema['name'],
            data: $cursorSchema['data'],
        ),
    ],
    examples: null,
    links: null,
    errors: null,
    examplePairingObjects: null,
    tags: null,
);
```

### Referencing the Schema

Once added to components, reference it using `$ref`:

```php
use Cline\OpenRpc\ContentDescriptor\CursorPaginatorContentDescriptor;

$descriptor = CursorPaginatorContentDescriptor::create();
// Schema is: ['$ref' => '#/components/schemas/CursorPaginator']
```

## Custom Schemas

Create your own schemas using `SchemaValue`:

### User Schema

```php
use Cline\OpenRpc\ValueObject\SchemaValue;

$userSchema = new SchemaValue(
    name: 'User',
    data: [
        'type' => 'object',
        'required' => ['id', 'email'],
        'properties' => [
            'id' => [
                'type' => 'integer',
                'description' => 'Unique user identifier',
                'minimum' => 1,
            ],
            'email' => [
                'type' => 'string',
                'format' => 'email',
                'description' => 'User email address',
            ],
            'name' => [
                'type' => 'string',
                'description' => 'Display name',
                'minLength' => 1,
                'maxLength' => 255,
            ],
            'role' => [
                'type' => 'string',
                'enum' => ['user', 'admin', 'moderator'],
                'default' => 'user',
            ],
            'createdAt' => [
                'type' => 'string',
                'format' => 'date-time',
            ],
            'updatedAt' => [
                'type' => 'string',
                'format' => 'date-time',
            ],
        ],
    ],
);
```

### Pagination Response Schema

```php
$paginatedSchema = new SchemaValue(
    name: 'PaginatedResponse',
    data: [
        'type' => 'object',
        'required' => ['data', 'meta'],
        'properties' => [
            'data' => [
                'type' => 'array',
                'items' => [
                    '$ref' => '#/components/schemas/User',
                ],
            ],
            'meta' => [
                'type' => 'object',
                'properties' => [
                    'cursor' => [
                        'type' => 'object',
                        'properties' => [
                            'current' => ['type' => 'string', 'nullable' => true],
                            'prev' => ['type' => 'string', 'nullable' => true],
                            'next' => ['type' => 'string', 'nullable' => true],
                        ],
                    ],
                    'perPage' => ['type' => 'integer'],
                    'hasMore' => ['type' => 'boolean'],
                ],
            ],
        ],
    ],
);
```

### Error Response Schema

```php
$errorSchema = new SchemaValue(
    name: 'ValidationError',
    data: [
        'type' => 'object',
        'required' => ['field', 'message'],
        'properties' => [
            'field' => [
                'type' => 'string',
                'description' => 'Field that failed validation',
            ],
            'message' => [
                'type' => 'string',
                'description' => 'Human-readable error message',
            ],
            'code' => [
                'type' => 'string',
                'description' => 'Machine-readable error code',
            ],
        ],
    ],
);
```

## Complete Components Example

```php
use Cline\OpenRpc\ValueObject\ComponentsValue;
use Cline\OpenRpc\ValueObject\SchemaValue;
use Cline\OpenRpc\ValueObject\ErrorValue;
use Cline\OpenRpc\ValueObject\ContentDescriptorValue;
use Cline\OpenRpc\Schema\CursorPaginatorSchema;

$cursorSchema = CursorPaginatorSchema::create();

$components = new ComponentsValue(
    contentDescriptors: [
        'UserId' => new ContentDescriptorValue(
            name: 'userId',
            summary: 'User identifier',
            description: null,
            required: true,
            schema: ['type' => 'integer', 'minimum' => 1],
            deprecated: false,
        ),
    ],
    schemas: [
        'CursorPaginator' => new SchemaValue(
            name: $cursorSchema['name'],
            data: $cursorSchema['data'],
        ),
        'User' => new SchemaValue(
            name: 'User',
            data: [
                'type' => 'object',
                'required' => ['id', 'email'],
                'properties' => [
                    'id' => ['type' => 'integer'],
                    'email' => ['type' => 'string', 'format' => 'email'],
                    'name' => ['type' => 'string'],
                ],
            ],
        ),
        'PaginatedUsers' => new SchemaValue(
            name: 'PaginatedUsers',
            data: [
                'type' => 'object',
                'properties' => [
                    'data' => [
                        'type' => 'array',
                        'items' => ['$ref' => '#/components/schemas/User'],
                    ],
                    'meta' => ['$ref' => '#/components/schemas/PaginationMeta'],
                ],
            ],
        ),
        'PaginationMeta' => new SchemaValue(
            name: 'PaginationMeta',
            data: [
                'type' => 'object',
                'properties' => [
                    'cursor' => ['type' => 'string', 'nullable' => true],
                    'nextCursor' => ['type' => 'string', 'nullable' => true],
                    'prevCursor' => ['type' => 'string', 'nullable' => true],
                    'hasMore' => ['type' => 'boolean'],
                    'perPage' => ['type' => 'integer'],
                ],
            ],
        ),
    ],
    examples: null,
    links: null,
    errors: [
        'NotFound' => new ErrorValue(
            code: 404,
            message: 'Resource not found',
            data: null,
        ),
        'Unauthorized' => new ErrorValue(
            code: 401,
            message: 'Authentication required',
            data: null,
        ),
        'ValidationFailed' => new ErrorValue(
            code: -32602,
            message: 'Validation failed',
            data: [
                'type' => 'array',
                'items' => ['$ref' => '#/components/schemas/ValidationError'],
            ],
        ),
    ],
    examplePairingObjects: null,
    tags: null,
);
```

## Using References

Reference components throughout your specification:

```php
// In method parameters
$params = new DataCollection(ContentDescriptorValue::class, [
    new ContentDescriptorValue(
        name: 'userId',
        summary: null,
        description: null,
        required: true,
        schema: ['$ref' => '#/components/contentDescriptors/UserId'],
        deprecated: false,
    ),
]);

// In method results
$result = new ContentDescriptorValue(
    name: 'users',
    summary: 'List of users',
    description: null,
    required: true,
    schema: ['$ref' => '#/components/schemas/PaginatedUsers'],
    deprecated: false,
);

// In method errors
$errors = new DataCollection(ErrorValue::class, [
    ErrorValue::from(['$ref' => '#/components/errors/NotFound']),
    ErrorValue::from(['$ref' => '#/components/errors/Unauthorized']),
]);
```

## SchemaValue Serialization

`SchemaValue` has a custom `toArray()` method that outputs the schema in the format expected by OpenRPC:

```php
$schema = new SchemaValue(
    name: 'User',
    data: ['type' => 'object', 'properties' => [...]],
);

$array = $schema->toArray();
// Result: ['User' => ['type' => 'object', 'properties' => [...]]]
```

This format is required when placing schemas in the `components/schemas` section of an OpenRPC document.

## JSON Schema Features

The schema `data` array supports all JSON Schema Draft 7 features:

### Basic Types

```php
['type' => 'string']
['type' => 'integer']
['type' => 'number']
['type' => 'boolean']
['type' => 'array']
['type' => 'object']
['type' => 'null']
```

### String Constraints

```php
[
    'type' => 'string',
    'minLength' => 1,
    'maxLength' => 255,
    'pattern' => '^[a-z]+$',
    'format' => 'email', // email, uri, date-time, uuid, etc.
]
```

### Numeric Constraints

```php
[
    'type' => 'integer',
    'minimum' => 0,
    'maximum' => 100,
    'exclusiveMinimum' => 0,
    'multipleOf' => 5,
]
```

### Array Constraints

```php
[
    'type' => 'array',
    'items' => ['type' => 'string'],
    'minItems' => 1,
    'maxItems' => 10,
    'uniqueItems' => true,
]
```

### Object Constraints

```php
[
    'type' => 'object',
    'required' => ['id', 'name'],
    'properties' => [...],
    'additionalProperties' => false,
    'minProperties' => 1,
]
```

### Composition

```php
// anyOf
['anyOf' => [['type' => 'string'], ['type' => 'integer']]]

// oneOf
['oneOf' => [['$ref' => '#/components/schemas/Cat'], ['$ref' => '#/components/schemas/Dog']]]

// allOf (inheritance)
['allOf' => [['$ref' => '#/components/schemas/Base'], ['properties' => [...]]]]
```
