---
title: Content Descriptors
description: Pre-built OpenRPC content descriptors for pagination, filtering, sorting, and relationships
---

Content descriptors define the structure and metadata for method parameters and results. This package provides pre-built descriptors for common API patterns that follow JSON:API conventions.

## Overview

| Descriptor | Purpose |
|------------|---------|
| `CursorPaginatorContentDescriptor` | Cursor-based pagination parameters |
| `FieldsContentDescriptor` | Sparse fieldset selection |
| `FiltersContentDescriptor` | Query filtering parameters |
| `SortsContentDescriptor` | Sort order specification |
| `RelationshipsContentDescriptor` | Relationship inclusion |
| `MethodDataContentDescriptor` | Generic method data input |

## CursorPaginatorContentDescriptor

Generates content descriptor for cursor-based pagination, ideal for real-time data feeds where traditional offset pagination becomes inconsistent.

```php
use Cline\OpenRpc\ContentDescriptor\CursorPaginatorContentDescriptor;

$descriptor = CursorPaginatorContentDescriptor::create();

// Result:
// [
//     'name' => 'page',
//     'description' => 'The page to return. If not specified, the first page is returned.',
//     'schema' => [
//         '$ref' => '#/components/schemas/CursorPaginator',
//     ],
// ]
```

**Usage in methods:**

```php
$method = new MethodValue(
    name: 'users.list',
    params: new DataCollection(ContentDescriptorValue::class, [
        ContentDescriptorValue::from(CursorPaginatorContentDescriptor::create()),
    ]),
    // ...
);
```

**Client request example:**

```json
{
    "jsonrpc": "2.0",
    "method": "users.list",
    "params": {
        "page": {
            "cursor": "eyJpZCI6MTAwfQ==",
            "size": 25
        }
    },
    "id": 1
}
```

## FieldsContentDescriptor

Enables sparse fieldsets following JSON:API patterns. Clients specify exactly which fields they want, reducing payload size.

```php
use Cline\OpenRpc\ContentDescriptor\FieldsContentDescriptor;

$descriptor = FieldsContentDescriptor::create([
    'users' => ['id', 'name', 'email', 'created_at'],
    'posts' => ['id', 'title', 'content', 'published_at'],
]);

// Result:
// [
//     'name' => 'fields',
//     'description' => 'The fields to return for each resource...',
//     'schema' => [
//         'type' => 'object',
//         'properties' => [
//             'users' => [
//                 'name' => 'users',
//                 'type' => 'array',
//                 'items' => [
//                     'type' => 'string',
//                     'enum' => ['id', 'name', 'email', 'created_at'],
//                 ],
//             ],
//             'posts' => [
//                 'name' => 'posts',
//                 'type' => 'array',
//                 'items' => [
//                     'type' => 'string',
//                     'enum' => ['id', 'title', 'content', 'published_at'],
//                 ],
//             ],
//         ],
//     ],
// ]
```

**Client request example:**

```json
{
    "jsonrpc": "2.0",
    "method": "users.list",
    "params": {
        "fields": {
            "users": ["id", "name", "email"],
            "posts": ["id", "title"]
        }
    },
    "id": 1
}
```

Returns `null` if no fields are provided:

```php
$descriptor = FieldsContentDescriptor::create([]);
// Returns null
```

## FiltersContentDescriptor

Defines available filter fields for server-side data filtering. Ensures clients can only filter on explicitly allowed fields.

```php
use Cline\OpenRpc\ContentDescriptor\FiltersContentDescriptor;

$descriptor = FiltersContentDescriptor::create([
    'users' => ['name', 'email', 'active', 'role'],
    'posts' => ['title', 'published', 'author_id', 'category'],
]);

// Result:
// [
//     'name' => 'filters',
//     'description' => 'The filters to apply to the resources...',
//     'schema' => [
//         'type' => 'object',
//         'properties' => [
//             'users' => [
//                 'name' => 'users',
//                 'type' => 'array',
//                 'items' => [
//                     'type' => 'string',
//                     'enum' => ['name', 'email', 'active', 'role'],
//                 ],
//             ],
//             // ...
//         ],
//     ],
// ]
```

**Client request example:**

```json
{
    "jsonrpc": "2.0",
    "method": "users.list",
    "params": {
        "filters": {
            "users": {
                "active": true,
                "role": "admin"
            }
        }
    },
    "id": 1
}
```

## SortsContentDescriptor

Specifies sortable fields with automatic ascending/descending variants. Follows JSON:API convention where `-` prefix indicates descending order.

```php
use Cline\OpenRpc\ContentDescriptor\SortsContentDescriptor;

$descriptor = SortsContentDescriptor::create([
    'users' => ['name', 'created_at', 'email'],
    'posts' => ['title', 'published_at', 'views'],
]);

// Result includes both ascending and descending variants:
// [
//     'name' => 'sorts',
//     'description' => 'The sort order of the resources...',
//     'schema' => [
//         'type' => 'object',
//         'properties' => [
//             'users' => [
//                 'type' => 'array',
//                 'items' => [
//                     'type' => 'string',
//                     'enum' => [
//                         'name', 'created_at', 'email',        // ascending
//                         '-name', '-created_at', '-email',      // descending
//                     ],
//                 ],
//             ],
//             // ...
//         ],
//     ],
// ]
```

**Client request example:**

```json
{
    "jsonrpc": "2.0",
    "method": "users.list",
    "params": {
        "sorts": {
            "users": ["name", "-created_at"]
        }
    },
    "id": 1
}
```

This sorts users by name ascending, then by created_at descending.

## RelationshipsContentDescriptor

Defines which relationships can be eagerly loaded with each resource type, following JSON:API include patterns.

```php
use Cline\OpenRpc\ContentDescriptor\RelationshipsContentDescriptor;

$descriptor = RelationshipsContentDescriptor::create([
    'users' => ['profile', 'posts', 'comments', 'roles'],
    'posts' => ['author', 'comments', 'tags', 'category'],
]);

// Result:
// [
//     'name' => 'relationships',
//     'description' => 'The relationships to return for each resource...',
//     'schema' => [
//         'type' => 'object',
//         'properties' => [
//             'users' => [
//                 'type' => 'array',
//                 'items' => [
//                     'type' => 'string',
//                     'enum' => ['profile', 'posts', 'comments', 'roles'],
//                 ],
//             ],
//             // ...
//         ],
//     ],
// ]
```

**Client request example:**

```json
{
    "jsonrpc": "2.0",
    "method": "users.get",
    "params": {
        "userId": 123,
        "relationships": {
            "users": ["profile", "posts"]
        }
    },
    "id": 1
}
```

## MethodDataContentDescriptor

Creates content descriptors for method input data. Supports both direct schema definition and extraction from data classes.

### From Schema

```php
use Cline\OpenRpc\ContentDescriptor\MethodDataContentDescriptor;

$descriptor = MethodDataContentDescriptor::create([
    'type' => 'object',
    'required' => ['email', 'password'],
    'properties' => [
        'email' => [
            'type' => 'string',
            'format' => 'email',
        ],
        'password' => [
            'type' => 'string',
            'minLength' => 8,
        ],
        'remember' => [
            'type' => 'boolean',
            'default' => false,
        ],
    ],
]);

// Result:
// [
//     'name' => 'data',
//     'description' => 'The data that will be passed to the method.',
//     'schema' => [
//         'type' => 'object',
//         'required' => ['email', 'password'],
//         'properties' => [...],
//     ],
// ]
```

### From Data Class

```php
use Cline\OpenRpc\ContentDescriptor\MethodDataContentDescriptor;

// Data class must implement getValidationRules() method
class LoginData
{
    public static function getValidationRules(array $context): array
    {
        return [
            'type' => 'object',
            'required' => ['email', 'password'],
            'properties' => [
                'email' => ['type' => 'string', 'format' => 'email'],
                'password' => ['type' => 'string', 'minLength' => 8],
            ],
        ];
    }
}

$descriptor = MethodDataContentDescriptor::createFromData(LoginData::class);
```

## Complete Method Example

Combining all content descriptors in a list method:

```php
use Cline\OpenRpc\ValueObject\MethodValue;
use Cline\OpenRpc\ValueObject\ContentDescriptorValue;
use Cline\OpenRpc\ContentDescriptor\CursorPaginatorContentDescriptor;
use Cline\OpenRpc\ContentDescriptor\FieldsContentDescriptor;
use Cline\OpenRpc\ContentDescriptor\FiltersContentDescriptor;
use Cline\OpenRpc\ContentDescriptor\SortsContentDescriptor;
use Cline\OpenRpc\ContentDescriptor\RelationshipsContentDescriptor;
use Spatie\LaravelData\DataCollection;

$params = [];

// Add pagination
$params[] = ContentDescriptorValue::from(
    CursorPaginatorContentDescriptor::create()
);

// Add fields (if any)
if ($fields = FieldsContentDescriptor::create(['users' => ['id', 'name', 'email']])) {
    $params[] = ContentDescriptorValue::from($fields);
}

// Add filters
if ($filters = FiltersContentDescriptor::create(['users' => ['name', 'email', 'active']])) {
    $params[] = ContentDescriptorValue::from($filters);
}

// Add sorts
if ($sorts = SortsContentDescriptor::create(['users' => ['name', 'created_at']])) {
    $params[] = ContentDescriptorValue::from($sorts);
}

// Add relationships
if ($rels = RelationshipsContentDescriptor::create(['users' => ['profile', 'posts']])) {
    $params[] = ContentDescriptorValue::from($rels);
}

$method = new MethodValue(
    name: 'users.list',
    tags: null,
    summary: 'List users with pagination and filtering',
    description: 'Returns a paginated list of users with optional filtering, sorting, and relationship loading.',
    externalDocs: null,
    params: new DataCollection(ContentDescriptorValue::class, $params),
    result: new ContentDescriptorValue(
        name: 'users',
        summary: 'Paginated user list',
        description: null,
        required: true,
        schema: ['$ref' => '#/components/schemas/PaginatedUsers'],
        deprecated: false,
    ),
    deprecated: false,
    servers: null,
    errors: null,
    links: null,
    paramStructure: 'by-name',
    examples: null,
);
```
