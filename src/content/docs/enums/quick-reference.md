---
title: Quick Reference
description: A quick overview of the main features available in the enums package.
---

This page provides a quick overview of the main features available in the enums package.

## Core Features

- **InvokableCases**: Use `MyEnum::FOO()` instead of `MyEnum::FOO->value`
- **Names & Values**: Get all case names or values as arrays
- **Options**: Generate form-ready arrays and HTML options
- **Metadata**: Attach custom properties to cases with attributes
- **Comparable**: Use `->is()`, `->in()` for readable comparisons
- **From**: Create instances from names with `fromName()`

## Common Patterns

```php
use Cline\Enums\Concerns\{InvokableCases, Names, Values, Comparable};

enum Status: int
{
    use InvokableCases, Names, Values, Comparable;

    case Draft = 0;
    case Published = 1;
    case Archived = 2;
}

// Get value without ->value
Status::Published(); // 1

// Collections
Status::names(); // ['Draft', 'Published', 'Archived']
Status::values(); // [0, 1, 2]

// Comparisons
$status->is(Status::Published); // true
$status->in([Status::Draft, Status::Published]); // true
```

## See Also

- **[Getting Started](/enums/getting-started)** - Installation and basic usage
- **[InvokableCases](/enums/invokable-cases)** - Get enum values without ->value
- **[Collections](/enums/collections)** - Names, values, and options methods
- **[Instantiation](/enums/instantiation)** - Enhanced from() and fromName() methods
- **[Metadata](/enums/metadata)** - Add custom properties to enum cases
- **[Comparable](/enums/comparable)** - Compare enum instances with is(), in()
