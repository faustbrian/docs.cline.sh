---
title: Getting Started
description: Throw provides a fluent, readable API for conditionally throwing exceptions in Laravel applications.
---

Throw provides a fluent, readable API for conditionally throwing exceptions in Laravel applications.

## Requirements

Throw requires PHP 8.5+.

## Installation

Install Throw with composer:

```bash
composer require cline/throw
```

## Add the Trait

Add Throw's trait to your custom exception classes:

```php
use Cline\Throw\Concerns\ConditionallyThrowable;
use RuntimeException;

class InvalidTokenException extends RuntimeException
{
    use ConditionallyThrowable;

    public static function expired(): self
    {
        return new self('Token has expired');
    }
}
```

## Basic Usage

Now you can use fluent conditional throwing:

```php
// Throw if condition is true
InvalidTokenException::expired()->throwIf($token->isExpired());

// Throw unless condition is true
InvalidTokenException::expired()->throwUnless($token->isValid());
```

## Next Steps

- Learn about [Basic Usage](basic-usage.md) patterns
- Explore [HTTP Responses](http-responses.md) for aborting requests
- See [Integration Patterns](integration-patterns.md) for Laravel conventions
