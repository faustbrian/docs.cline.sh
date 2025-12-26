---
title: String Assertions
description: Validate string values for length, patterns, and content.
---

String assertions validate and check string values for various conditions including length, patterns, and content.

## Available Assertions

### regex()

Assert that a string matches a regular expression pattern.

```php
use Cline\Assert\Assertions\Assertion;

Assertion::regex($value, '/^[A-Z][a-z]+$/');
Assertion::regex($code, '/^[A-Z]{3}\d{3}$/', 'Code must be 3 letters followed by 3 digits');
```

### notRegex()

Assert that a string does NOT match a pattern.

```php
Assertion::notRegex($username, '/[^a-zA-Z0-9_]/', 'Username contains invalid characters');
```

### length()

Assert that a string has an exact length.

```php
Assertion::length($zipCode, 5);
Assertion::length($value, 10, null, null, 'utf8');
```

### minLength()

Assert minimum string length.

```php
Assertion::minLength($password, 8);
Assertion::minLength($username, 3, 'Username must be at least 3 characters');
```

### maxLength()

Assert maximum string length.

```php
Assertion::maxLength($username, 20);
Assertion::maxLength($title, 100, 'Title cannot exceed 100 characters');
```

### betweenLength()

Assert string length is within a range.

```php
Assertion::betweenLength($password, 8, 100);
Assertion::betweenLength($name, 2, 50, 'Name must be between 2 and 50 characters');
```

### startsWith()

Assert that a string starts with a substring.

```php
Assertion::startsWith($url, 'https://');
Assertion::startsWith($phoneNumber, '+1', 'Phone number must start with +1');
```

### endsWith()

Assert that a string ends with a substring.

```php
Assertion::endsWith($filename, '.pdf');
Assertion::endsWith($email, '@example.com', 'Email must be from example.com');
```

### contains()

Assert that a string contains a substring.

```php
Assertion::contains($content, 'keyword');
Assertion::contains($url, '://secure.', 'URL must contain secure subdomain');
```

### notContains()

Assert that a string does NOT contain a substring.

```php
Assertion::notContains($password, $username, 'Password cannot contain username');
Assertion::notContains($content, '<script', 'Content cannot contain script tags');
```

### alnum()

Assert that a string is alphanumeric.

```php
Assertion::alnum($identifier);
Assertion::alnum($code, 'Code must be alphanumeric');
```

## Chaining String Assertions

```php
use Cline\Assert\Assert;

Assert::that($password)
    ->string()
    ->notEmpty()
    ->minLength(8)
    ->maxLength(100)
    ->notContains($username);

Assert::that($slug)
    ->string()
    ->regex('/^[a-z0-9-]+$/')
    ->minLength(3);
```

## Common Patterns

### Username Validation

```php
Assert::that($username)
    ->string()
    ->betweenLength(3, 20)
    ->regex('/^[a-zA-Z0-9_]+$/', 'Letters, numbers, and underscores only');
```

### Password Strength

```php
Assert::that($password)
    ->string()
    ->minLength(8)
    ->regex('/[A-Z]/', 'Must contain uppercase letter')
    ->regex('/[a-z]/', 'Must contain lowercase letter')
    ->regex('/[0-9]/', 'Must contain number')
    ->notContains($username, 'Password cannot contain username');
```

### URL Validation

```php
Assert::that($url)
    ->string()
    ->notEmpty()
    ->startsWith('https://')
    ->url();
```

## Encoding Support

String assertions support different character encodings:

```php
Assertion::length($japanese, 5, null, null, 'utf8');
Assertion::minLength($text, 10, null, null, 'ISO-8859-1');
```

## Next Steps

- [Validation Assertions](/assert/validation-assertions/) - Email, URL, UUID validation
- [Type Assertions](/assert/type-assertions/) - Type checking
- [Assertion Chains](/assert/assertion-chains/) - Fluent API
