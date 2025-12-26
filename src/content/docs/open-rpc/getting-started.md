---
title: Getting Started
description: Build OpenRPC specification documents for JSON-RPC 2.0 APIs in Laravel
---

## Installation

Install via Composer:

```bash
composer require cline/open-rpc
```

## What is OpenRPC?

OpenRPC is a specification for describing JSON-RPC 2.0 APIs in a machine-readable format. Think of it as OpenAPI/Swagger for JSON-RPC. This package provides:

- **Complete value objects** for all OpenRPC specification components
- **Content descriptors** for common API patterns (pagination, filtering, sorting)
- **JSON Schema integration** for parameter and result validation
- **Spatie Laravel Data** integration for type-safe serialization

## Quick Start

### Create an OpenRPC Document

```php
<?php

use Cline\OpenRpc\ValueObject\DocumentValue;
use Cline\OpenRpc\ValueObject\InfoValue;
use Cline\OpenRpc\ValueObject\MethodValue;
use Cline\OpenRpc\ValueObject\ContentDescriptorValue;
use Spatie\LaravelData\DataCollection;

// Create API info
$info = new InfoValue(
    title: 'My API',
    description: 'A sample JSON-RPC API',
    termsOfService: 'https://example.com/tos',
    contact: null,
    license: null,
    version: '1.0.0',
);

// Define a method
$method = new MethodValue(
    name: 'user.get',
    tags: null,
    summary: 'Get a user by ID',
    description: 'Returns a user object for the given ID',
    externalDocs: null,
    params: new DataCollection(ContentDescriptorValue::class, [
        new ContentDescriptorValue(
            name: 'userId',
            summary: 'The user ID',
            description: null,
            required: true,
            schema: ['type' => 'integer', 'minimum' => 1],
            deprecated: false,
        ),
    ]),
    result: new ContentDescriptorValue(
        name: 'user',
        summary: 'The user object',
        description: null,
        required: true,
        schema: [
            'type' => 'object',
            'properties' => [
                'id' => ['type' => 'integer'],
                'name' => ['type' => 'string'],
                'email' => ['type' => 'string', 'format' => 'email'],
            ],
        ],
        deprecated: false,
    ),
    deprecated: false,
    servers: null,
    errors: null,
    links: null,
    paramStructure: null,
    examples: null,
);

// Create the document
$document = new DocumentValue(
    openrpc: '1.2.6',
    info: $info,
    servers: null,
    methods: new DataCollection(MethodValue::class, [$method]),
    components: null,
    externalDocs: null,
);

// Serialize to array/JSON
$spec = $document->toArray();
```

### Add Server Configuration

```php
use Cline\OpenRpc\ValueObject\ServerValue;
use Cline\OpenRpc\ValueObject\ServerVariableValue;
use Spatie\LaravelData\DataCollection;

$servers = new DataCollection(ServerValue::class, [
    new ServerValue(
        name: 'Production',
        url: 'https://api.example.com/rpc',
        summary: 'Production API server',
        description: null,
        variables: null,
    ),
    new ServerValue(
        name: 'Development',
        url: 'http://localhost:8080/rpc',
        summary: 'Local development server',
        description: null,
        variables: null,
    ),
]);

$document = new DocumentValue(
    openrpc: '1.2.6',
    info: $info,
    servers: $servers,
    methods: $methods,
    components: null,
    externalDocs: null,
);
```

## Specification Version

The `openrpc` field specifies which version of the OpenRPC specification your document conforms to. Current versions:

- `1.2.6` - Latest stable version
- `1.2.5`, `1.2.4`, etc. - Previous versions

## Value Objects

All OpenRPC specification components are represented as immutable value objects:

| Value Object | Description |
|-------------|-------------|
| `DocumentValue` | Root document container |
| `InfoValue` | API metadata (title, version, description) |
| `MethodValue` | RPC method definition |
| `ContentDescriptorValue` | Parameter/result schema |
| `ServerValue` | Server endpoint configuration |
| `ErrorValue` | Error response definition |
| `ComponentsValue` | Reusable components container |
| `ExampleValue` | Sample values |
| `ExamplePairingValue` | Request/response pairs |
| `TagValue` | Method categorization |
| `LinkValue` | Related resources |

## Next Steps

- **[Value Objects](value-objects)** - Complete API reference for all value objects
- **[Content Descriptors](content-descriptors)** - Pre-built descriptors for pagination, filtering, and sorting
- **[Schemas](schemas)** - Reusable JSON Schema definitions
