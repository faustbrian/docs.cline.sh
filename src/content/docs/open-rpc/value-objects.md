---
title: Value Objects
description: Complete API reference for OpenRPC specification value objects
---

All OpenRPC specification components are represented as immutable value objects extending `Spatie\LaravelData\Data`. This ensures type safety, automatic serialization, and integration with Laravel's ecosystem.

## DocumentValue

The root container for an OpenRPC specification document.

```php
use Cline\OpenRpc\ValueObject\DocumentValue;

$document = new DocumentValue(
    openrpc: '1.2.6',                    // Required: OpenRPC spec version
    info: $info,                          // Required: API metadata
    servers: $servers,                    // Optional: Server endpoints
    methods: $methods,                    // Required: RPC methods
    components: $components,              // Optional: Reusable components
    externalDocs: $externalDocs,          // Optional: External documentation
);
```

**Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `openrpc` | `string` | Yes | Semantic version of OpenRPC specification |
| `info` | `InfoValue` | Yes | API metadata |
| `servers` | `DataCollection<ServerValue>` | No | Available server endpoints |
| `methods` | `DataCollection<MethodValue>` | Yes | RPC method definitions |
| `components` | `ComponentsValue` | No | Reusable specification components |
| `externalDocs` | `ExternalDocumentationValue` | No | External documentation links |

## InfoValue

Metadata about the API including title, version, and contact information.

```php
use Cline\OpenRpc\ValueObject\InfoValue;
use Cline\OpenRpc\ValueObject\ContactValue;
use Cline\OpenRpc\ValueObject\LicenseValue;

$info = new InfoValue(
    title: 'My API',
    description: 'A JSON-RPC 2.0 API',
    termsOfService: 'https://example.com/tos',
    contact: new ContactValue(
        name: 'API Support',
        url: 'https://example.com/support',
        email: 'support@example.com',
    ),
    license: new LicenseValue(
        name: 'MIT',
        url: 'https://opensource.org/licenses/MIT',
    ),
    version: '1.0.0',
);
```

**Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `title` | `string` | Yes | API title |
| `description` | `?string` | No | Detailed description (CommonMark) |
| `termsOfService` | `?string` | No | URL to terms of service |
| `contact` | `?ContactValue` | No | Contact information |
| `license` | `?LicenseValue` | No | License information |
| `version` | `string` | Yes | API version |

## MethodValue

Defines an RPC method with its parameters, result, and metadata.

```php
use Cline\OpenRpc\ValueObject\MethodValue;
use Cline\OpenRpc\ValueObject\ContentDescriptorValue;
use Spatie\LaravelData\DataCollection;

$method = new MethodValue(
    name: 'user.create',
    tags: null,
    summary: 'Create a new user',
    description: 'Creates a user with the provided data',
    externalDocs: null,
    params: new DataCollection(ContentDescriptorValue::class, [
        new ContentDescriptorValue(
            name: 'email',
            summary: 'User email address',
            description: null,
            required: true,
            schema: ['type' => 'string', 'format' => 'email'],
            deprecated: false,
        ),
        new ContentDescriptorValue(
            name: 'name',
            summary: 'User display name',
            description: null,
            required: true,
            schema: ['type' => 'string', 'minLength' => 1],
            deprecated: false,
        ),
    ]),
    result: new ContentDescriptorValue(
        name: 'user',
        summary: 'Created user object',
        description: null,
        required: true,
        schema: ['$ref' => '#/components/schemas/User'],
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

**Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | `string` | Yes | Unique method identifier |
| `tags` | `DataCollection<TagValue>` | No | Categorization tags |
| `summary` | `?string` | No | Brief description |
| `description` | `?string` | No | Detailed description |
| `externalDocs` | `?ExternalDocumentationValue` | No | External documentation |
| `params` | `DataCollection<ContentDescriptorValue>` | Yes | Method parameters |
| `result` | `?ContentDescriptorValue` | No | Return value descriptor |
| `deprecated` | `bool` | No | Deprecation flag |
| `servers` | `DataCollection<ServerValue>` | No | Method-specific servers |
| `errors` | `DataCollection<ErrorValue>` | No | Possible error responses |
| `links` | `DataCollection<LinkValue>` | No | Related resources |
| `paramStructure` | `?string` | No | `by-position` or `by-name` |
| `examples` | `DataCollection<ExamplePairingValue>` | No | Example request/response pairs |

## ContentDescriptorValue

Describes a parameter, result, or error with its schema and metadata.

```php
use Cline\OpenRpc\ValueObject\ContentDescriptorValue;

$descriptor = new ContentDescriptorValue(
    name: 'userId',
    summary: 'User identifier',
    description: 'The unique identifier for a user in the system',
    required: true,
    schema: [
        'type' => 'integer',
        'minimum' => 1,
        'description' => 'Positive integer ID',
    ],
    deprecated: false,
);
```

**Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | `string` | Yes | Unique identifier |
| `summary` | `?string` | No | Brief description |
| `description` | `?string` | No | Detailed description |
| `required` | `bool` | No | Whether parameter is required |
| `schema` | `?array` | No | JSON Schema definition |
| `deprecated` | `bool` | No | Deprecation flag |

## ServerValue

Defines an API server endpoint.

```php
use Cline\OpenRpc\ValueObject\ServerValue;
use Cline\OpenRpc\ValueObject\ServerVariableValue;

$server = new ServerValue(
    name: 'Production',
    url: 'https://api.example.com/{version}/rpc',
    summary: 'Production API endpoint',
    description: 'Main production server with regional routing',
    variables: [
        'version' => new ServerVariableValue(
            default: 'v1',
            description: 'API version',
            enum: ['v1', 'v2'],
        ),
    ],
);
```

**Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | `string` | Yes | Server identifier |
| `url` | `string` | Yes | Server URL (supports variables) |
| `summary` | `?string` | No | Brief description |
| `description` | `?string` | No | Detailed description |
| `variables` | `?array<string, ServerVariableValue>` | No | URL template variables |

## ErrorValue

Defines an error response following JSON-RPC 2.0 error format.

```php
use Cline\OpenRpc\ValueObject\ErrorValue;

$error = new ErrorValue(
    code: -32602,
    message: 'Invalid params',
    data: [
        'field' => 'email',
        'reason' => 'Invalid email format',
    ],
);
```

**Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `code` | `int` | Yes | Numeric error code |
| `message` | `string` | Yes | Human-readable message |
| `data` | `mixed` | No | Additional error data |

**Standard JSON-RPC 2.0 Error Codes:**

| Code | Message | Description |
|------|---------|-------------|
| -32700 | Parse error | Invalid JSON |
| -32600 | Invalid Request | Invalid JSON-RPC request |
| -32601 | Method not found | Method does not exist |
| -32602 | Invalid params | Invalid method parameters |
| -32603 | Internal error | Server error |
| -32000 to -32099 | Server error | Reserved for implementation |

## ComponentsValue

Container for reusable specification components.

```php
use Cline\OpenRpc\ValueObject\ComponentsValue;
use Cline\OpenRpc\ValueObject\SchemaValue;

$components = new ComponentsValue(
    contentDescriptors: [
        'UserId' => new ContentDescriptorValue(...),
    ],
    schemas: [
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
    ],
    examples: null,
    links: null,
    errors: [
        'NotFound' => new ErrorValue(
            code: 404,
            message: 'Resource not found',
            data: null,
        ),
    ],
    examplePairingObjects: null,
    tags: null,
);
```

**Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `contentDescriptors` | `?array<string, ContentDescriptorValue>` | Reusable parameter/result descriptors |
| `schemas` | `?array<string, SchemaValue>` | JSON Schema definitions |
| `examples` | `?array<string, ExampleValue>` | Reusable examples |
| `links` | `?array<string, LinkValue>` | Reusable links |
| `errors` | `?array<string, ErrorValue>` | Reusable error definitions |
| `examplePairingObjects` | `?array<string, ExamplePairingValue>` | Reusable example pairs |
| `tags` | `?array<string, TagValue>` | Reusable tags |

## SchemaValue

Named JSON Schema definition for use in components.

```php
use Cline\OpenRpc\ValueObject\SchemaValue;

$schema = new SchemaValue(
    name: 'Pagination',
    data: [
        'type' => 'object',
        'properties' => [
            'page' => ['type' => 'integer', 'minimum' => 1],
            'perPage' => ['type' => 'integer', 'minimum' => 1, 'maximum' => 100],
        ],
    ],
);

// Converts to: { "Pagination": { "type": "object", ... } }
$array = $schema->toArray();
```

## ExampleValue

Sample values for documentation and testing.

```php
use Cline\OpenRpc\ValueObject\ExampleValue;

$example = new ExampleValue(
    name: 'valid-user',
    summary: 'A valid user object',
    description: 'Example of a complete user with all fields',
    value: json_encode([
        'id' => 123,
        'email' => 'user@example.com',
        'name' => 'John Doe',
    ]),
    externalValue: null,
);
```

## ExamplePairingValue

Groups parameter examples with their corresponding results.

```php
use Cline\OpenRpc\ValueObject\ExamplePairingValue;
use Cline\OpenRpc\ValueObject\ExampleValue;
use Spatie\LaravelData\DataCollection;

$pairing = new ExamplePairingValue(
    name: 'get-user-by-id',
    description: 'Get an existing user by their ID',
    summary: 'Successful user retrieval',
    params: new DataCollection(ExampleValue::class, [
        new ExampleValue(
            name: 'userId',
            summary: null,
            description: null,
            value: '123',
            externalValue: null,
        ),
    ]),
    result: new DataCollection(ExampleValue::class, [
        new ExampleValue(
            name: 'user',
            summary: null,
            description: null,
            value: json_encode(['id' => 123, 'name' => 'John']),
            externalValue: null,
        ),
    ]),
);
```

## ReferenceValue

Creates `$ref` pointers to reusable components.

```php
use Cline\OpenRpc\ValueObject\ReferenceValue;

$ref = new ReferenceValue(
    ref: '#/components/schemas/User',
);
```

## Supporting Value Objects

### ContactValue

```php
$contact = new ContactValue(
    name: 'API Support',
    url: 'https://example.com/support',
    email: 'api@example.com',
);
```

### LicenseValue

```php
$license = new LicenseValue(
    name: 'MIT',
    url: 'https://opensource.org/licenses/MIT',
);
```

### TagValue

```php
$tag = new TagValue(
    name: 'users',
    summary: 'User management methods',
    description: 'Methods for creating, reading, updating, and deleting users',
    externalDocs: null,
);
```

### LinkValue

```php
$link = new LinkValue(
    name: 'GetUserById',
    summary: 'Get the user by ID',
    description: null,
    method: 'user.get',
    params: ['userId' => '$response.id'],
    server: null,
);
```

### ExternalDocumentationValue

```php
$docs = new ExternalDocumentationValue(
    description: 'Full API documentation',
    url: 'https://docs.example.com/api',
);
```

### ServerVariableValue

```php
$variable = new ServerVariableValue(
    default: 'v1',
    description: 'API version',
    enum: ['v1', 'v2', 'beta'],
);
```
